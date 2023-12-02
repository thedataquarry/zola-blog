+++
title = "Why Pydantic just keeps getting better"
description = "Understanding the latest features and optimizations of Pydantic v2 that provide a 12x speedup over v1"
date = 2023-11-30
draft = true

[taxonomies]
tags = ["pydantic", "rust"]
categories = ["Frameworks"]

[extra]
lang = "en"
toc = true
comment = true
+++

## 12x possible speedup over v1, with more to come

[Pydantic](https://docs.pydantic.dev/latest/) is a data parsing, transformation and validation library for Python that's become integral to the PyData ecosystem. Version 2 of Pydantic, which came out in June 2023, had its core rewritten in Rust, which I described in some detail in my [previous post](../why-pydantic-v2-matters/). 

It's been really fascinating to watch each major release of Pydantic v2 lately, as they've been showing incremental performance improvements over prior releases.

{{ figure(src="pydantic-benchmark-results.png" alt="Performance comparison of each successive Pydantic v2 major release") }}

As can be seen, there's not only a **12x** speedup over v1.10 in the latest release (2.5.2 as of writing this post) -- there's also a **12%** improvement over the first v2 release. These improvements are likely due to a variety of optimizations and new features at the lower levels in Rust.

The goal of this post is to describe the details of the benchmark whose results are shown above, and also to uncover the underlying components of `pydantic-core` (the part that's written in Rust) that contribute to the overall performance improvement that we see at the Python level.

{% warning() %}
It's assumed you know what the purpose of Pydantic is, and have some familiarity with its API -- if not, check out the [first post](../why-pydantic-v2-matters/) in this series.
{% end %}

## Uncovering the Pydantic stack

Before we go into the benchmark and the reasons for the performance gain, it's worth spending a bit of time to really understand Pydantic's internals.

As we know, v2 has a new design that splits it up into two Python packages:

* `pydantic`: Written in Python and is user-facing
* `pydantic-core`: A base Python package that contains the core functionality for serialization and validation, whose core is based on Rust

The Python API we, the end users, are used to interfacing with, is built directly on top of `pydantic-core`. However, it's important to understand that `pydantic-core` itself is not a monolithic entity.

{{ figure(src="pydantic-stack.png" alt="The multi-layer stack of Pydantic v2") }}

The lowest layers of Pydantic are composed of a multitude of Rust crates that form `pydantic-core` that we're familiar with. The top (user-facing) layer is the Python API. The intermediate laters, PyO3 and `maturin`, are explained in more detail below.

### Rust crates

[`speedate`](https://github.com/pydantic/speedate) is a datetime and time duration parser that functions at the Rust level. [`jiter`](https://github.com/pydantic/jiter) is an iterable JSON parser that offers multiple interfaces for handling JSON data (enums, iterators and strings) in Rust. The [regex](https://github.com/rust-lang/regex) crate is the regex engine that's maintained by the Rust foundation. There are many other base crates as well, coming together to compose what we know as `pydantic-core`.

Earlier versions of Pydantic v2 used `orjson` (a Python library), which itself depended on the age-old [`serde_json`](https://github.com/serde-rs/json) Rust crate for JSON parsing. `serde_json` is among the most downloaded Rust crates, with almost 200 million downloads as of 2023, so the fact that the Pydantic team decided to move away from such a mature package in the Rust ecosystem to write their own JSON parser indicates how critical this part of the stack is to Pydantic's performance.

As Samuel Colvin, the creator of Pydantic, [explains](https://x.com/samuel_colvin/status/1720153791767429473?s=20):

> `orjson` uses serde-json, to deserialize JSON, then takes some serious liberties with the Python C API/PyO3 FFI interface for performance. `jiter`, an alternative to `serde_json`, is significantly faster, and also allows you to get the position of a value even if JSON parsing passes -- so Pydantic validation errors can show the JSON file positions.

With JSON being the key serialization and deserialization component in Pydantic workflows, and with Pydantic being used to validate ever larger amounts of data, every ounce of performance matters!

### PyO3

[PyO3](https://github.com/PyO3/pyo3) is a Rust crate that allows Rust code to be called from Python, via generated Rust bindings. It's a fork of `rust-cpython`, which aims to offer a Rust wrapper around the `libpython` C API so that Rust code can be called from Python. PyO3 is now a mature project with a large community, and is the most common way to write performant Python extensions in Rust.

Recent versions of Pydantic have begun applying **profile-guided optimization** (PGO) during `pydantic-core` compilation. As described in [the docs](https://doc.rust-lang.org/rustc/profile-guided-optimization.html), PGO collects data about the typical execution of a binary and uses this data to inform optimizations such as inlining, machine code layout, register allocation, and branch prediction. The result is that the `pydantic-core` bindings, even prior to being called in Python, are already optimized for the typical execution patterns of Pydantic.

### Maturin

[Maturin](https://github.com/PyO3/maturin) is a utility that's used to generate native Python modules (wheels) from Rust crates that were built with PyO3. The result of using Maturin is that the underlying Rust code in `pydantic-core` can be made available as a Python package that's itself imported into `pydantic`, to be available to end users.

### Pydantic

The user-facing part of Pydantic, written in Python, is essentially a wrapper around `pydantic-core`. None of the heavy-lifting is done at the Python level -- all the parsing, validation and serialization is done at the Rust level, so Python users can continue about their business as usual without worrying about Rust.

---
## Performance benchmark

This section covers the benchmark that was run to study the performance improvements in each major release of v2, with the reference version (for comparison) being v1.10. The benchmark code is available [here](https://github.com/prrao87/pydantic-benchmarks).

### Dataset

The dataset used is a familiar one if you've read my previous blogs: A wine reviews dataset from Kaggle. It consists of 129,971 wine reviews from the Wine Enthusiast magazine, made available in newline-delimited JSON as shown below. Refer to the [Kaggle source](https://www.kaggle.com/datasets/zynicide/wine-reviews) for more detailed information on the dataset and how it was scraped.

An example JSON line representing a single wine review, when read into Python, is shown below. Note how the comments in each line specify what we *want*, vs. what's actually present in the data.

```py
{
    "id": 40825,  # This field is compulsory (not nullable)
    "points": "90",  # Bad value: This should be an integer
    "title": "Castello San Donato in Perano 2009 Riserva  (Chianti Classico)",
    "description": "Made from a blend of 85% Sangiovese and 15% Merlot, this ripe wine delivers soft plum, black currants, clove and cracked pepper sensations accented with coffee and espresso notes. A backbone of firm tannins give structure. Drink now through 2019.",
    "taster_name": "Kerin O'Keefe",
    "taster_twitter_handle": "@kerinokeefe",
    "price": "30.0",  # Bad value: This should be a float
    "designation": "Riserva", # Rename this to "vineyard"
    "variety": "Red Blend",
    "region_1": "null", # Drop string 'null' fields
    "region_2": None, # Drop None fields
    "province": "Tuscany",
    "country": "Italy",  # Missing country is unacceptable: replace with "Unknown
    "winery": "Castello San Donato in Perano"
}
```

### Simple validator

The first benchmark makes use of a simple validator based on familiar Pydantic concepts -- fields, validators and models. The validator is shown below.

{% codeblock(name="schemas.py") %}
```py
from pydantic import BaseModel, ConfigDict, Field, model_validator

class Wine(BaseModel):
    model_config = ConfigDict(
        populate_by_name=True,
        str_strip_whitespace=True,
    )

    id: int
    points: int
    title: str
    description: str | None
    price: float | None
    variety: str | None
    winery: str | None
    designation: str | None = Field(None, alias="vineyard")
    country: str | None
    province: str | None
    region_1: str | None
    region_2: str | None
    taster_name: str | None
    taster_twitter_handle: str | None

    @model_validator(mode="before")
    def _remove_unknowns(cls, values):
        "Set other fields that have the value 'null' as None so that we can throw it away"
        for field in ["region_1", "region_2"]:
            if not values.get(field) or values.get(field) == "null":
                values[field] = None
        return values

    @model_validator(mode="before")
    def _fill_country_unknowns(cls, values):
        "Fill in missing country values with 'Unknown', as we always want this field to be queryable"
        country = values.get("country")
        if not country or country == "null":
            values["country"] = "Unknown"
        return values
```
{% end %}

The goal of the validator is to coerce bad data types to the types we want, and to drop fields that we don't want, or are missing. In addition, the `model_config` is used to strip whitespace from all string fields, and to populate specific fields by an alias. `model_validator` class methods are used to modify elements of a dict as they are parsed, before validation.


### Improved validator

The improved validator makes use of some new features in Pydantic v2, and is shown below.

{% codeblock(name="schemas_improved.py") %}
```py
from pydantic import BeforeValidator, TypeAdapter, constr, field_validator
from pydantic_core import PydanticOmit
from typing_extensions import Annotated, NotRequired, TypedDict

not_required_fields = ["region_1","region_2"]

def exclude_none(s: str | None) -> str:
    if s is None:
        # since we want `exclude_none=True` in the end,
        # just omit it if it's `None` during validation
        raise PydanticOmit
    else:
        return s

ExcludeNoneStr = Annotated[str, BeforeValidator(exclude_none)]

class Wine(TypedDict):
    id: int
    points: int
    title: str
    description: NotRequired[str]
    price: NotRequired[float]
    variety: NotRequired[str]
    winery: NotRequired[str]
    designation: NotRequired[constr(strip_whitespace=True)]
    country: NotRequired[str]
    province: NotRequired[str]
    region_1: NotRequired[str]
    region_2: NotRequired[str]
    taster_name: NotRequired[str]
    taster_twitter_handle: NotRequired[str]

    @field_validator(*not_required_fields, mode="before")
    def omit_null_none(cls, v):
        # type: ignore
        if v is None or v == "null":
            raise PydanticOmit
        else:
            return v

    @field_validator("country", mode="before")
    def country_unknown(cls, s: str | None) -> str:
        # type: ignore
        if s is None or s == "null":
            return "Unknown"
        else:
            return s

WinesTypeAdapter = TypeAdapter(list[Wine])
```
{% end %}

There are a number of differences between this validator and the previous one:

* The `Wine` model in the improved version is a [`TypedDict`](https://docs.python.org/3/library/typing.html#typing.TypedDict) instead of a `BaseModel`. This is a new feature in Pydantic v2 that allows us to define a schema using Python's native `TypedDict` type (which, at runtime, is a plain `dict`)

* `field_validator` is used instead of `model_validator`. These operate only on specific fields, and we apply the the `NotRequired` type to all fields that we want to be able to omit from the final output if their value is either `None` or `'null'`.
  
* A `TypeAdapter` is used, which exposes only [some of the functionality](https://docs.pydantic.dev/latest/api/type_adapter/) of `BaseModel`. This is much more performant, as it avoids the overhead of the `BaseModel` class when simply performing validation on a `dict`.

Because these features listed above are unique to Pydantic v2, the improved validator can only be run on v2.x and not on v1.

{% tip(header="Tip") %}
As I [learned from](https://github.com/prrao87/pydantic-benchmarks/pull/1) Samuel Colvin himself while making this benchmark, certain validation workflows such as this one, largely spend their time in serialization, i.e., converting a `dict` to a Pydantic model and back to a `dict` for downstream use. The model here is also rather simple, with no nested fields or complex validation logic. In such cases, `TypedDict` coupled with `TypeAdapter` is a better choice for performance reasons than `BaseModel`.
{% end %}

### Benchmark code

The benchmark is run using `pytest-benchmark`. First, the data is into a test fixture.

```py
from typing import Any
from pathlib import Path
from util import get_json_data  # A helper function to load data via srsly

import pytest

@pytest.fixture
def data() -> list[dict[str, Any]]:
    """Load the data once per session"""
    DATA_DIR = Path("/path/to/data")
    FILENAME = "winemag-data-130k-v2.jsonl.gz"
    data = list(get_json_data(DATA_DIR, FILENAME))
    return data
```

The from each JSON record in the dataset are unpacked as `kwargs` and passed to the validator to produce a Pydantic model, shown below. The model is then dumped back to a `dict` via `model_dump` for downstream use.

{% codeblock(name="basic_validator") %}
```py
```py
def validate(data):
    """Validate a list of JSON blobs against the Wine schema"""
    validated_data = [Wine(**item).model_dump(exclude_none=True, by_alias=True) for item in data]
    return validated_data

# Run validator
def test_validate(benchmark, data):
    """Validate the data"""
    result = benchmark(validate, data)
    assert len(result) == len(data)
```
{% end %}

The improved validator is run directly from the `TypeAdapter` we defined earlier. Note that we don't need to unpack the JSON items into a `dict` and then pass them to the validator, as `TypeAdapter` can operate directly on a list of `dict`s, which avoids unnecessary serialization overhead.

{% codeblock(name="improved_validator") %}
```py,linenos,hl_lines=3
def test_validate_improved(benchmark, data):
    """Validate a list of JSON blobs against the Wine schema"""
    result = benchmark(WinesTypeAdapter.validate_python, data)
    assert len(result) == len(data)
```
{% end %}

Note how the `validate_python` method is used instead of serializing a `dict` to a Pydantic model. This is *much* better for performance, and should be used when the only goal is to validate a `dict` against a schema.

The benchmark is then run via [`benchmark_validator.py`](https://github.com/prrao87/pydantic-benchmarks/blob/main/v2/benchmark_validator.py).

{% codeblock(name="bash") %}
```bash
pytest benchmark_validator.py --benchmark-sort=fullname --benchmark-warmup-iterations=5 --benchmark-min-rounds=10
```
{% end %}

## Reasons for performance improvement

The performance improvements can be divided into two categories:

### Improvements at the Rust level

* PGO applied at the PyO3 level during `pydantic-core` compilation
* Usage of a fast, custom JSON parser, `jiter`, which avoids the `serde_json` dependency via `orjson`, and [is tuned](https://x.com/samuel_colvin/status/1720153791767429473?s=20) to Pydantic's JSON parsing requirements

### Improvements at the Python level

* Using `TypedDict` and `TypeAdapter` instead of `BaseModel` to avoid serialization overhead (transferring the Python `dict` to be parsed and validated at the Rust level via PyO3 bindings)
* Using `field_validator` instead of `model_validator` to avoid unnecessary validation of all fields when not required
* Using `NotRequired` type extensions and a new `PydanticOmit` exception in v2 to avoid unnecessary serialization of fields that are `None` or `'null'`. This is done during the parsing stage, prior to validation, which is faster.

## Towards a nogil future

Going forward, there are even more improvements happening at the very low level, related to PyO3 and how to work around the Python Global Interpreter Lock (GIL).

{{ twitter(url="https://twitter.com/pydantic/status/1726939559680721200" class="twitter") }}

In the near term, an [upcoming release](https://github.com/PyO3/pyo3/issues/3382) of `pydantic-core` is attempting to port the upcoming improvements to PyO3's GIL handling. A further [20-30% performance improvement](https://github.com/pydantic/pydantic-core/pull/1085#issuecomment-1820573327) is expected from this, all available to end users for free, with zero code changes required. 🤯

Over the longer term, a future release of Python 3.13 that aims to make the GIL completely optional will allow the Python code at the top layer of Pydantic to perform operations within `pydantic-core` in a way that's completely free of the GIL. This is fondly referred to as the "nogil" future of Python, and has the potential to be a game-changer for Pydantic, as well as a host of other Python libraries that can benefit from true multi-threaded code execution.

## Conclusions

Because of these low level developments that are happening silently under the hood, largely at the Rust level, end users can expect to see more-than-subtle performance improvements in their Pydantic code over time. Considering the billions (or trillions) of times a day that Pydantic code executes on servers all over the world, every ounce of efficient computation is a win for companies that run the code, and for the planet as a whole. 🌎

I hope this post has gotten you as excited as I am about the future of Pydantic, and, in general, the future of Rust in the PyData ecosystem. A lot of this is enabled by the work of David Hewitt, who [recently joined Pydantic](https://twitter.com/pydantic/status/1671233172867256320), and is simultaneously pushing the boundaries of what's possible with PyO3 and Pydantic.

Have fun porting your Pydantic code to v2, and let's keep tracking and celebrating the performance gains! 🚀

## Code

All code for the benchmarks shown in this post is available [here](https://github.com/prrao87/pydantic-benchmarks).

### Acknowledgements

Special thanks to [Samuel Colvin](https://twitter.com/samuel_colvin?lang=en) for taking the time to create [a PR](https://github.com/prrao87/pydantic-benchmarks/pull/1) for the improved validation logic.

{% important() %}
Open source is truly a wonderful thing, so if you're a team that hugely benefits from Pydantic, consider [sponsoring](https://github.com/sponsors/samuelcolvin) the project to help it thrive. 🫶🏼
{% end %}