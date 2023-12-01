+++
title = "Obtain a 5x speedup for free by upgrading to Pydantic v2"
description = "Why Pydantic v2 being written in Rust matters, and how it's MUCH faster"
date = 2023-07-01
updated = 2023-11-29
draft = false

[taxonomies]
tags = ["pydantic", "rust"]
categories = ["Frameworks"]

[extra]
lang = "en"
toc = true
comment = true
+++

# Why it matters that Pydantic v2 is written in Rust

If you've worked with any form of data wrangling in Python, you've likely heard of [Pydantic](https://docs.pydantic.dev/latest/). If not, welcome! In this post, I'll show an example of why Pydantic is so integral to the workflows of data scientists and engineers, and how just a few small changes to your existing code written in v1 can be changed as per v2, to obtain a 5x speedup, largely due to Pydantic's internals being rewritten in Rust.

## What is Pydantic and why would you use it?

Yes, compiled language enthusiasts, we hear you. _Unlike_ compiled languages (C, C++, Rust), Python is an interpreted, dynamically typed language where variables can change types during run time. This means that when you write Python code, especially for large code bases involving ETL & data transformations, it becomes necessary to test for a _lot_ of edge cases. A whole class of errors (`TypeError`) gets introduced into your Python runtime, where data obtained downstream may not align with expectations based on logic you or other developers put in, further upstream.

Python 3.5 introduced [type hints](https://peps.python.org/pep-0484/), which allows developers to declare what type a variable should be, prior to runtime. However, type hints are only marginally effective because they **don't enforce** data types at runtime -- they're only a _guide_, and not a guarantee that the data you pass around remains a certain type. What's the solution to this? Enter Pydantic 😎.

As per [their docs](https://docs.pydantic.dev/latest/#why-use-pydantic), Pydantic is a data parsing, transformation and validation library for Python that leverages the underlying information in type hints and adds additional checks and constraints via a _schema_. Any untrusted data obtained is transformed and validated to conform to the known schema during runtime.

If the incoming data isn't of the expected type or quality, Pydantic very loudly and explicitly throws an informative error, helping developers more easily trace down bugs or missing data prior to major deployments in production. Using Pydantic reduces the number of tests required, eliminating entire classes of bugs, streamlining the development process and ensuring that complex Python workflows run more reliably in production.

Pydantic is a [highly used](https://pypistats.org/packages/pydantic) library and is downloaded >100 million times a month -- it is used by 20 of the 25 largest companies on the NASDAQ, as well as all the biggest tech companies in the world. Clearly, ensuring type stability and cleaner, more trustworthy data upstream, offers a TON of value and can save tremendous amounts of wasted compute time in production! When you think of the sheer volume of data being handled at the largest companies on earth (most of them involving Python, due to its heavy use in data science and ML), it makes sense why Pydantic has become so important in the PyData ecosystem of today.

## Pydantic v2: A new core 🦀

Normally, the release of a new major version of a library is just water under the bridge, and life goes on as normal. However, it's because of where exactly Pydantic sits in the value chain -- at the foundation of a host of data wrangling and ETL workflows -- and the fact that its core functionality is now written in a systems-language like Rust, that makes v2 so important.

If you think about it, modern software engineering and machine learning workflows rely on large batches or streams of data coming in, and it's a well-known fact that data quality can impact multiple outcomes of a software product[^2], all the way from customer engagement to insight generation and, ultimately, revenue.

### Why Rust?

I've already written about this in [my other blog post](../rust-is-supercharging-python/) on how Rust has been transforming the PyData ecosystem. Rust, being a close-to-bare-metal systems language, allows developers to build highly performant and memory-safe tooling for Python users with a lot more ease than they could in other languages. This is done via [PyO3](https://github.com/PyO3/pyo3), a tool that allows developer to create Rust bindings for the Python interpreter, fully leveraging the underlying power and expressivity of Rust.

Since 2022, PyO3 has been having more and more of an impact on the PyData ecosystem, and the lead developer of Pydantic, Samuel Colvin, has spoken[^3] about the importance of the interplay between Python 🐍 and Rust 🦀, and the kind of impact this can have on the PyData ecosystem in general.

In 2023, David Hewitt, a core maintainer of PyO3, a library that's empowering a whole new generation of Python developers to write faster, more efficient packages for Python in Rust, began working at Pydantic, which further solidifies the importance of Rust in the PyData ecosystem.

{{ figure(src="david-hewitt-pydantic.png" alt="Source: <a href='https://twitter.com/pydantic/status/1671233172867256320'>Pydantic Twitter</a>") }}


### How does Pydantic v2 do its magic?

As of v2, Pydantic is now divided into two packages:
- `pydantic-core`: Contains Rust bindings for the core validation and serialization logic, and doesn't have a user-facing interface
- `pydantic`: A pure Python, higher level, user-facing package

When you write your validation logic in Python via Pydantic v2, you're simply defining "instructions" that are pushed down to the `pydantic-core` Rust layer. Samuel Colvin, creator of Pydantic, explains[^4] how a large part of the performance gains are achieved:

- Compiled Rust bindings means all the validations are happening _outside_ of Python
- Recursive function calls are made in Rust, which have very little additional overhead
- Using a tree of small validators that call each other, making code easier to read and extend without harming performance

{{ figure(src="pydantic-core.png") }}

## Case study on Pydantic v1 vs v2

Okay, enough of background, let's see some code! 

{% warning(header="Information") %}
The aim of this section is to demonstrate how Pydantic v2, thanks to its core being written in Rust 🦀, is at least **5x faster** than v1 🤯. The performance gains come for _free_, just by upgrading to the newest version and changing a few lines of code 🚀.
{% end %}

### The data

The example dataset we'll be working with is the [wine reviews dataset](https://www.kaggle.com/datasets/zynicide/wine-reviews) from Kaggle. It consists of 130k wine reviews from the Wine Enthusiast magazine, including the variety, location, winery, price, description, and some other metadata for each wine. Refer to the Kaggle source for more detailed information on the data and how it was scraped. The original data was downloaded as a single JSON file. For the purposes of this blog post, the data was then converted to newline-delimited JSON (.jsonl) format where each line of the file contains a valid JSON object.

An example JSON line is shown below.

```json
{
    "id": 40825,
    "points": "90",
    "title": "Castello San Donato in Perano 2009 Riserva  (Chianti Classico)",
    "description": "Made from a blend of 85% Sangiovese and 15% Merlot, this ripe wine delivers soft plum, black currants, clove and cracked pepper sensations accented with coffee and espresso notes. A backbone of firm tannins give structure. Drink now through 2019.",
    "taster_name": "Kerin O'Keefe",
    "taster_twitter_handle": "@kerinokeefe",
    "price": "30.0",
    "designation": "Riserva",
    "variety": "Red Blend",
    "region_1": "null",
    "region_2": null,
    "province": "Tuscany",
    "country": "Italy",
    "winery": "Castello San Donato in Perano"
}
```

### The benchmark

This benchmark consists of the following data validation tasks:

- Ensure the `id` field always exists (this will be the primary key), and is an integer
- Ensure the `points` field is an integer
- Ensure the `price` field is a float
- Ensure the `country` field always has a non-null value -- if it's set as `null` or the `country` key doesn't exist in the raw data, it _must_ be set to `Unknown`. This is because the use case we defined will involve querying on `country` downstream
- Remove fields like `designation`, `province`, `region_1` and `region_2` if they have the value `null` in the raw data -- these fields will not be queried on and we do not want to unnecessarily store null values downstream

{% note(header="Note") %}
All tasks described in this post are run on a 2022 M2 Macbook Pro. To get a clearer estimate on performance differences, we will wrap the validation checks in a loop and run them repetitively, **over 10 cycles**. The run times for each cycle are reported, along with the total run time for both versions.
{% end %}

#### Note on timing

Using a convenience library like [codetiming](https://github.com/realpython/codetiming) makes it easier to wrap the portions of the code we want to time without having to add/remove `time.time()` commands over and over.


## Schema

The schema file is at the root of the validation logic in Pydantic. The general practice is to use [validators](https://docs.pydantic.dev/latest/usage/validators/), which are a kind of class method in Pydantic to transform or validate data and ensure it's of the right type and format for downstream tasks. Using validators is **far cleaner** than writing custom functions that manipulate data (they would be riddled with ugly `if-else` statements or `isinstance` clauses, not to mention that they'd need added tests). The schema below implements the validation logic defined above.

{% codeblock(name="Pydantic v1") %}
```python,linenos,hl_lines=19 28 36
from pydantic import BaseModel, root_validator

class Wine(BaseModel):
    id: int
    points: int
    title: str
    description: str | None
    price: float | None
    variety: str | None
    winery: str | None
    designation: str | None
    country: str | None
    province: str | None
    region_1: str | None
    region_2: str | None
    taster_name: str | None
    taster_twitter_handle: str | None

    @root_validator
    def _remove_unknowns(cls, values):
        "Set other fields that have the value 'null' as None so that we can throw it away"
        fields = ["designation", "province", "region_1", "region_2"]
        for field in fields:
            if not values.get(field) or values.get(field) == "null":
                values[field] = None
        return values

    @root_validator
    def _fill_country_unknowns(cls, values):
        "Fill in missing country values with 'Unknown', as we always want this field to be queryable"
        country = values.get("country")
        if not country or country == "null":
            values["country"] = "Unknown"
        return values

    @root_validator
    def _get_vineyard(cls, values):
        "Rename designation to vineyard"
        vineyard = values.pop("designation", None)
        if vineyard:
            values["vineyard"] = vineyard.strip()
        return values
```
{% end %}

The following code in Pydantic v2 performs the same actions as the v1 version. Note the changed lines highlighted in the code below.

{% codeblock(name="Pydantic v2") %}
```python,linenos,hl_lines=19 28 36
from pydantic import BaseModel, model_validator

class Wine(BaseModel):
    id: int
    points: int
    title: str
    description: str | None
    price: float | None
    variety: str | None
    winery: str | None
    designation: str | None
    country: str | None
    province: str | None
    region_1: str | None
    region_2: str | None
    taster_name: str | None
    taster_twitter_handle: str | None

    @model_validator(mode="before")
    def _remove_unknowns(cls, values):
        "Set other fields that have the value 'null' as None so that we can throw it away"
        fields = ["designation", "province", "region_1", "region_2"]
        for field in fields:
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

    @model_validator(mode="before")
    def _get_vineyard(cls, values):
        "Rename designation to vineyard"
        vineyard = values.pop("designation", None)
        if vineyard:
            values["vineyard"] = vineyard.strip()
        return values
```
{% end %}


## `root_validator` (v1) and `model_validator` (v2)

- In Pydantic v2, `root_validator` is deprecated, and we instead use `model_validator`
    - In a similar way, `validator` is deprecated and we instead use `field_validator` where we may need to validate specific fields (not used in this example)
- The keyword arguments to the `model_validator` differ from v1's `root_validator` (and in my view, are more intuitive and readable)
    - `mode="before"` in v2 means that we are performing data transformations on the fields that already exist in the raw data, prior to validation

That's it! With just these minor changes (**everything else is kept the same**) in the schema definition, we are now ready to test their performance!

## Timing comparison

Set up two separate Python virtual environments, one with Pydantic v2 installed, and another one with v1 installed. Each portion of the code base is explained in the following sections.

### Read in data as a list of dicts

The raw data is stored in line-delimited JSON (gzipped) format, so it is first read in as a list of dicts. The lightweight serialization library `srsly` is used for this task: `pip install srsly`

```python
from pathlib import Path
from typing import Any

import srsly

def get_json_data(data_dir: Path, filename: str) -> list[JsonBlob]:
    """Get all line-delimited files from a directory with a given prefix"""
    file_path = data_dir / filename
    data = srsly.read_gzip_jsonl(file_path)
    return data


if __name__ == "__main__":
    DATA_DIR = Path("../data")
    FILENAME = "winemag-data-130k-v2.jsonl.gz"
    data = list(get_json_data(DATA_DIR, FILENAME))
    # data is now a list of dicts
```

### Define validation function

- Define a function that imports the `Wine` schema defined above (separately for v1 and v2 of Pydantic).
- The list of dicts that contain the raw data are unwrapped in the `Wine` validator class, which performs validation and type coercion
- Note the change in syntax for dumping the validated data as a dict (in v1, the `dict()` method was used, but in v2, it's renamed to `model_dump`).
    - The `exclude_none=True` keyword argument to `model_dump` removes fields that have the value `None` -- this saves space and avoids us storing `None` values downstream

{% codeblock(name="Pydantic v1") %}
```python,linenos,hl_lines=4
from schemas import Wine

def validate(data: list[JsonBlob]) -> list[JsonBlob]:
    validated_data = [Wine(**item).dict(exclude_none=True) for item in data]
    return validated_data
```
{% end %}

Note how the code is almost identical for v2, except for the change from `dict()` to `model_dump()`.

{% codeblock(name="Pydantic v2") %}
```python,linenos,hl_lines=4
from schemas import Wine

def validate(data: list[JsonBlob]) -> list[JsonBlob]:
    validated_data = [Wine(**item).model_dump(exclude_none=True) for item in data]
    return validated_data
```
{% end %}

### Define a run function that wraps a timer

To time each validation cycle, we wrap the `run` function inside a `Timer` context manager as follows. Note that this requires that the `codetiming` library is installed: `pip install codetiming`.

```python
from codetiming import Timer

def run():
    """Wrapper function to time the validator over many runs"""
    with Timer(name="Single case", text="{name}: {seconds:.3f} sec"):
        validated_data = validate(data)
        print(f"Validated {len(validated_data)} records in cycle {i + 1} of {num}")
```

### Run in a loop

We can then call the `run()` method in a loop to perform validation 10 times, to see the average run time. To measure the total run time for all 10 cycles, wrap the loop itself inside another `Timer` block as follows:

```python
num = 10
with Timer(name="All cases", text="{name}: {seconds:.3f} sec"):
    for i in range(num):
        run()
```

## Results


{% note() %}
For a total of 10 cycles (where each cycle validates the same ~130k records), Pydantic v2 performs ~1.3 million validations in roughly 6 seconds, while the exact same workflow in v1 took almost 30 seconds: that's a **5x improvement**, for free!
{% end %}


Run      | Pydantic v1 | Pydantic v2
---------|------------|------------
Cycle 1  | 2.952 sec  |0.584 sec
Cycle 2  | 2.944 sec  | 0.580 sec
Cycle 3  | 2.952 sec  | 0.587 sec
Cycle 4  | 2.946 sec  | 0.576 sec
Cycle 5  | 2.925 sec  | 0.578 sec
Cycle 6  | 2.926 sec  | 0.582 sec
Cycle 7  | 2.927 sec  | 0.575 sec
Cycle 8  | 2.921 sec  | 0.582 sec
Cycle 9  | 2.950 sec  | 0.590 sec
Cycle 10 | 2.926 sec  | 0.582 sec
**Total** | **29.585 sec** | **6.032 sec**

### Why did v2 perform so much better?

- The v1 validation logic involved looping through dictionary key/value pairs and performing type checks and replacements in Python
- In v2 of Pydantic, all these operations, which are normally rather expensive in Python (due to its dynamic nature), are pushed down to the Rust level
    - None of the loops in the validator expressions are actually run in Python, explaining why without any major code changes, we see a large gain in performance

In many real world scenarios, the validation logic implemented in Python (due to business needs) can get rather complex, so the fact that library developers can leverage the power of Rust and reuse code this way, is a huge deal.

## Conclusions and broader implications

The timing numbers shown in this case study were for _validation only_. In a full-blown ETL workflow in production, it's likely that there are _many_ such scripts working on data far larger than 1 million records. It's easy to imagine that, when adding up the number of validations performed in Python at all the FAANG-scale companies (who really do use Pydantic in production), we would get a number in the _trillions_, purely due to the scale of data and the complexity of tasks being performed at these companies.

In 2019, Amazon Web Services (AWS) announced their official sponsorship for the Rust language. AWS is now among the leading contributors to the development of Rust, and for good reason: in 2022, they published a blog post highlighting how large-scale adoption of Rust at the core of compute-heavy operations can reduce energy usage in data centres worldwide[^5] by almost **50%**! 🤯

As per data available in 2020[^6], data centres consume about 200 terawatt hours per year globally, which amounts to around 1% of the total energy consumed on our planet. It's natural that with the frenzy of activity in AI and cloud computing these days, energy utilization in data centres will only grow with time, so any reduction in energy consumption via more efficient bare-metal operations has massive consequences.

{% tip() %}
Projects like Pydantic that perform core lower-level operations in Rust while allowing higher-level developers to build rapidly can have a **huge** impact on reducing energy consumption worldwide.
{% end %}

There are many other projects also going down the road of building faster, more efficient Python tooling on top of Rust, but projects like Pydantic, which were among the first to do so, will surely serve as an inspiration to the community at large. 💪

Hopefully this post will serve as an inspiration for you to begin exploring (and hopefully, learning) Rust! 🦀 💪🏽

## Code and data

The code and the data to reproduce these results are available [on GitHub](https://github.com/prrao87/pydantic-benchmarks). Depending on the CPU your machine uses, your mileage may vary. Have fun and happy Pydant-ing! 😎

---

[^2]: Data Quality in AI: Challenges, Importance & Best Practices, [AIMultiple](https://research.aimultiple.com/data-quality-ai/#:~:text=Data%20quality%20is%20crucial%20in,trust%20and%20confidence%20among%20users.)

[^3]: How Pydantic V2 leverages Rust's Superpowers, [FOSDEM '23](https://fosdem.org/2023/schedule/event/rust_how_pydantic_v2_leverages_rusts_superpowers/)

[^4]: The Talk Python Live Stream #376, [Pydantic v2 plan](https://www.youtube.com/watch?v=URrUBgOFl6U)

[^5]: Sustainability with Rust, [AWS Open Source Blog](https://aws.amazon.com/blogs/opensource/sustainability-with-rust/)

[^6]: Global data centre energy demand by data centre type, 2010-2022, [iea.org](https://www.iea.org/data-and-statistics/charts/global-data-centre-energy-demand-by-data-centre-type-2010-2022)