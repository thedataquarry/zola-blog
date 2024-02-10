+++
title = "Speeding up data analysis with Rayon and Rust"
description = "Rayon's makes data parallelism in Rust a breeze, and is blazing fast compared to Python multiprocessing. A simple walkthrough to get started."
date = 2024-02-10
draft = false

[taxonomies]
tags = ["rust"]
categories = ["Frameworks"]

[extra]
toc = true
comment = true
+++

## How Rayon + Rust can be a data scientist's secret weapon

Python is often the language used to analyze and explore data, and on many occasions, Python developers
encounter performance bottlenecks when processing large datasets. In this post,
I'll highlight how a Rust implementation that uses the [Rayon](https://github.com/rayon-rs/rayon) crate
makes parallelizing data processing *breathtakingly* simple in comparison to an equivalent Python
implementation. I'll also give a gentle introduction to the Rust concepts involved, and the type of
parallelism that Rayon implements.

The results from this post are quite remarkable. The Rust implementation using Rayon is about 4x faster
than Python, while also being more concise and readable than the Python version!

![](./rayon-vs-python-perf.png)

Rayon is a very powerful and flexible framework, and this post only scratches the surface of what it
can do. If you're coming from Python, I hope this walkthrough will inspire you to explore Rayon, and
the larger Rust ecosystem, much further. 🚀

{% note(header="Note") %}
In practice, the interop between Python and Rust is greatly enhanced by [PyO3](https://github.com/PyO3/pyo3), a
framework that enables you to call Rust bindings from your Python code. This approach is heavily used
by popular libraries like Pydantic, Polars and many others. Unifying Python and
Rust workflows via PyO3 is a larger topic for a future blog post, so stay tuned!
{% end%}

## Problem statement

Let's define a simple data analysis task that a Python data scientist might encounter. Say you want
to roughly estimate the proportion of men and women mentioned in a large dataset of news articles.
A simple heuristic is to count the number of times gendered
pronouns in English (*he, him, his, she, her, hers*) appear in each article's text. This is a very naive and crude
approach to analyzing gender balance -- it only works in English and potentially misses a lot of
mentions that don't use pronouns -- but it's at least a starting point to make sense of the data at hand.

We'll use the *[All the news](https://www.kaggle.com/datasets/snapcrack/all-the-news)* dataset from
Kaggle, which contains 143,000 news articles from 15 American publications. The dataset is made available
on Kaggle as three separate CSV files, `articles1.csv`, `articles2.csv` and `articles3.csv`.

As you can imagine, processing each article in a sequential fashion to extract and count pronouns is
not only slow -- it's also rather wasteful because it only exploits one CPU core to process the data when your
machine likely has multiple cores that are underutilized. In reality, each core (and its associated threads) can process a different
article at the same time. This can be viewed as an *embarrassingly parallel* problem, where each article
can be processed independently of the others.

It's typical in these scenarios to tokenize the text of each article and then count the
occurrences of specific tokens. However, because pronouns are the _only_ tokens we care about in
this case, it becomes a simple pattern matching problem and we can just use regular expressions
(skipping tokenization entirely).

This is best illustrated with an example.

> **She** says that **she's** going to be late to **his** place.

For the text above, we can apply a regex to match occurrences of gendered pronouns, replacing
word contractions like "she's" with the expanded form "she is", and then count the number of matches.
The result in this case is 1 male pronoun and 2 female pronouns. The goal is thus to apply this
method on the text of every article in the dataset, and to do so as efficiently and quickly as possible.

## Background

If you've been keeping up with the pace of innovation in ML and data science tooling lately, you've probably noticed that more
Python packages are powered by Rust, and for good reason. Rust is a systems programming language
that runs blazingly fast, offers memory safety without a garbage collector, and guarantees thread
safety at compile-time. CPU-hungry libraries like Hugging Face's [tokenizers](https://github.com/huggingface/tokenizers)
are a testament to this, having long understood these benefits, and are written almost
entirely in Rust.

Python users well know that their language is just *not* suited to tasks involving parallelism and concurrency. For CPU-intensive
tasks, you can use modules like `multiprocessing` and `concurrent.futures.ProcessPoolExecutor` that exploit
multiple cores to an extent, but building such pipelines can be surprisingly tedious. The onus is on the developer
to select the right number of CPU workers, manage batches of data and handle errors.

As mentioned above, Rust has a fantastic crate called Rayon for data parallelism[^1].
Rust is known for its relatively steep learning curve, and there's no denying that it's more
complex than Python to get started with. But Rayon is a great example of how a perfect level of abstraction
on top of a language's core features can make complex tasks like parallelism much more accessible to developers.

Even if you've not written a lot of Rust before, you can quickly become proficient enough with Rayon to massively speed up
certain data processing tasks without worrying about lower-level primitives like threads, locks, mutexes
and so on. If you read the [source code](https://github.com/huggingface/tokenizers/blob/c893204c45d7f2cd66958731dd7779548ca54ad5/tokenizers/src/utils/parallelism.rs)
of libraries like `tokenizers`, you'll see that it uses more complex Rust concepts like
lifetimes, traits and generics, but this post will highlight how you can get a LOT out of Rayon without
needing to get into those. As is common with most things in Rust, **start simply and build up from there!**
You do not need to be an expert in the language to use it productively. 😇

## Iterators and closures in Rust

This section will go over two important Rust concepts that are needed to follow along with
the code shown below. If you're already familiar with iterator and closure syntax in Rust, feel free to skip ahead to the next section.

In idiomatic Rust, for loops are not as common as in Python, and are often replaced by iterators,
which are used to process collections of data in a functional style. Consider the below example of squaring
each element of a vector of integers.

```rs
// 1. For loop
let nums = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
let mut nums_squared: Vec<i32> = Vec::new();
for num in nums {
    nums_squared.push(num.pow(2));
}
```

The for loop version will seem familiar to Python developers. Note that we need to declare `nums_squared` as mutable
because we're modifying it inside the loop. The `push` method is used to add elements to the vector.

```rs
// 2. Iterator
let nums = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
let nums_squared: Vec<i32> = nums
    .iter()
    .map(|&x| x.pow(2))
    .collect();
```

The iterator version first converts the input vector to an iterator via the `.iter()` method. It then
uses a common method in functional programming, `map`, to apply a squaring function to each element of the
`nums` vector, and then uses the `collect` method to collect the results into a new vector. The squaring
is done via a closure, which is an anonymous function in Rust, kind of analogous to lambda functions in Python.
The syntax `|&x| x.pow(2)` is a closure that takes a *reference* to an
integer `x` (i.e., it borrows the owned value of `x`) and returns the integer raised to a power of 2.

Rust has a strict borrow-checker and ownership system, so it's worth spending a bit of time to
understand how these work (but there's no need to go too deep when you're just getting started).

## Understanding Rayon's parallelism

Now that it's clear how to use iterators and closures in Rust, we can dig a little deeper into
how Rayon uses these constructs, and more, to achieve parallelism.

### Parallel iterators

The main user-facing abstraction in Rayon is the `par_iter` method,
which stands for parallel iterator. The idea is that a program that uses a sequential iterator
can be easily converted to use a parallel iterator by simply replacing `iter` with `par_iter`.

Under the hood, parallel iterators rely on a core primitive called `join` that allows Rayon to
potentially execute two closures in parallel and bring their results together once they both finish.
The remarkable feature is that Rayon *itself* (and not the user) decides whether or not parallelism
is beneficial, and if so, how to divide the work among threads.

This is an example of how simple it is to use a parallel iterator in Rayon.

```rs,hl_lines=1 6
use rayon::prelude::*;

// 3. Parallel iterator
let nums = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
let nums_squared: Vec<i32> = nums
    .par_iter()
    .map(|&x| x.pow(2))
    .collect();
```

All you do is replace the sequential `iter` with the parallel `par_iter` and Rayon takes care of
handling the lower level primitives. It's that simple!

Under the hood, parallel iterators apply a divide-and-conquer strategy to the task at hand. In the
example above, the parallel iterator initiated by the call to `par_iter` will spawn two threads that implement
a `join`, dividing the work into two halves. The worker thread in charge of either half
continues to divide the work further until a sufficiently small amount of work is left to be done,
which is then executed sequentially. This can be visualized as follows:

![](./rayon-divide-and-conquer.png)

Note that Rayon starts with roughly one worker thread per CPU core -- not all steps in the parallel
iterator pipeline will spawn new threads. The worker threads instead create *tasks* that are sent to the `join`
operation. A limited pool of worker threads shares a common pool of tasks, with a dynamic scheduling
strategy that exploits the available resources to the fullest.

### Work stealing

The idea of work stealing is not new, nor is it unique to Rayon. It's in fact from a body of research
in parallel program execution that goes back decades. Rayon's implementation is based on the method
described in a paper by Blumofe and Leiserson[^2].
In the paper, the authors compare and contrast work _stealing_ with work _sharing_, and show that work stealing
is provably more efficient.

In a nutshell, work sharing involves a scheduler that assigns tasks to new worker threads whenever
they are spawned, migrating the work to underutilized threads. In contrast, work stealing, which is implemented
by Rayon, is more proactive, where the underutilized threads themselves take
initiative and "steal" unfinished work from other threads. Intuitively, this results in less migration
of tasks and communication overhead leading to better performance.

Rayon implements an adaptive "thief-splitting" [algorithm](https://github.com/rayon-rs/rayon/pull/106)
when using `par_iter` that dynamically splits the work into enough jobs that fill every available thread.
Whenever a job is stolen, that job will again be split such that there is enough work for every thread.
The beauty of this approach is that it considers real-time factors like the number of threads, the
amount of work left to be done, how idle or busy the threads are, etc., and so the execution varies how
much parallelization is active at any given point. However, the user doesn't need to care about the lower level
details at any point.

{% tip() %}
In summary, a combination of **parallel iterators**, **work stealing** and the right level of user-facing
**abstraction** make Rayon such a fantastic tool for data parallelism.
{% end %}

## Safety guarantees in Rust

Rayon inherits all the safety guarantees that Rust provides.
From a memory safety perspective, Rust's ownership and borrowing system ensures that
no data races occur and that no two threads can modify the same data at the same time. When
coupled with Rust's powerful generics and type system, the result is that the framework properly
abstracts away the hard parts of parallelism, allowing the developer to focus on the program's logic.

Most importantly, the ever-watchful Rust compiler enforces these checks at compile time, meaning
that the kinds of errors that commonly plague parallel programs are caught before the program even runs.
This is in stark contrast to other systems languages, where subtle data access bugs can lead to
hard-to-debug errors during runtime.

## Data processing

We can now address the problem at hand -- capturing and counting gendered pronouns in the news articles dataset!

### Python and multiprocessing

In Python, we can begin by loading the CSV file's contents into a list of records. We first need to
clean the text by expanding word contractions (like "he's" to "he is") and then remove punctuation
and other non-alphabetic characters.

```py
import re

def clean_text(text: str) -> str:
    text_lower = text.lower()
    suffix_mapping = {
        "s": " is",
        "d": " had",
        "ll": " will",
    }
    # Replace contractions with full words
    formatted_text = re.sub(r"([’'])(s|d|ll)", lambda x: suffix_mapping[x.group(2)], text_lower)
    # Remove non-alphabetic characters
    result = re.sub(r"[^a-zA-Z\s]", "", formatted_text)
    return result
```

In the above function, we simultaneously match multiple patterns using a mapping via a lambda function.
The `x.group(2)` syntax denotes the second capture group in the regex match, which is the suffix of the contraction,
which is then looked up in the `suffix_mapping` dictionary via the lambda function.

Counting the pronouns is an easy job, where we check for the presence of each token in the list of pronouns
and increment the count by 1 each time it's found. The `calculate_counts` function is then applied
to each record in the dataset to calculate the counts at an article level, which we can then write
to a new CSV file.

```py
def count_gendered_pronouns(tokens: list[str]) -> tuple[int, int]:
    num_male_pronouns = sum(1 for token in tokens if token in ["he", "him", "his"])
    num_female_pronouns = sum(1 for token in tokens if token in ["she", "her", "hers"])
    return num_male_pronouns, num_female_pronouns

def calculate_counts(data: JsonBlob) -> JsonBlob:
    text = clean_text(data["content"])
    tokens = result.split()
    data["num_male_pronouns"], data["num_female_pronouns"] = count_gendered_pronouns(tokens)
    data.pop("content")
    return data
```

How is all this parallelized? We can design some multiprocessing logic that exploits multiple
CPU cores to process the data. Conceptually, it looks something like this:

![](./parallelism-python-approach.png)

In the figure above, only two worker processes
are shown, but in reality, the number of worker processes can be set to the number of CPU cores available.

The code to perform the operation via multiprocessing in Python is shown below.

```py
from typing import Iterator
from concurrent.futures import ProcessPoolExecutor

def create_batches(data: list[JsonBlob]) -> Iterator[list[JsonBlob]]:
    """Yield batches of data of the specified size"""
    for i in range(0, len(data), self.batch_size):
        yield data[i : i + self.batch_size]

def process_batches(data: list[JsonBlob]) -> list[JsonBlob]:
    with ProcessPoolExecutor(max_workers=NUM_WORKERS) as executor:
        # Batch up the data
        batches = list(create_batches(data))
        # Process batches in parallel
        results = []
        for batch in batches:
            batch_results = list(executor.map(self.calculate_counts, batch))
            results.extend(batch_results)
        return results

def main(files: list[Path]) -> None:
    for input_file in files:
        records = load_csv(input_file)
        results = process_batches(records)
        output_file = input_file.name.replace(".csv", "_processed.csv")
        write_results(results, file_path, output_file)
```

We manually divide the data into batches, and pass the batched data to the `ProcessPoolExecutor`,
which assigns each batch of data to a worker process.

The complete code for the Python implementation can be found [here](https://github.com/thedataquarry/rustinpieces/tree/main/src/parallelism/python).

### Rust and Rayon

The Rust code and logic is _really_ straightforward in comparison, and it's a testament to the
expressivity of the language as well as the design of the library.

```rs
fn clean_text(text: &str) -> String {
    let pattern1 = Regex::new(r"([’'])(s|d|ll)").unwrap();
    // Replace pattern with text
    let matched = pattern1.replace_all(text, |capture: &Captures| match &capture[2] {
        "s" => " is",
        "d" => " had",
        "ll" => " will",
        _ => "<unk>",
    });
    // Remove non-alphabetic characters
    let pattern2 = Regex::new(r"[^a-zA-Z\s]").unwrap();
    let clean_text = pattern2.replace_all(&matched, "");
    let result: String = clean_text.to_lowercase();
    result
}
```

The regex matching and replacement is done with the same patterns as in Python. The key difference
here is how we use a `match` statement to replace multiple patterns using a single closure.
The `&capture[2]` syntax is used to access the second capture group in the regex match, which is the
suffix of the contraction, and this is then passed to the match statement.

The parallelization logic is where the magic happens. Because Rayon implements parallel iterators, we
can actually perform *multiple* parallel computations at two levels -- first, at the level of the
articles, and then at the level of the tokens in each article. This is a very natural way to think
about divide-and-conquer problems, and shows how powerful the `par_iter` abstraction is.

```rs,hl_lines=3 7
fn count_gendered_pronouns(tokens: Vec<&str>) -> (usize, usize) {
    let num_male_pronouns = tokens
        .par_iter()
        .filter(|&x| *x == "he" || *x == "him" || *x == "his")
        .count();
    let num_female_pronouns = tokens
        .par_iter()
        .filter(|&x| *x == "she" || *x == "her" || *x == "hers")
        .count();
    (num_male_pronouns, num_female_pronouns)
}
```

The code above shows how just two lines of code changed from `iter()` to `par_iter()` can make the
computation parallel. The `filter` method is used to filter the tokens that match the pronouns, and
the `count` method is used to count the number of matches. This stage of the computation produces
the pronoun counts for each article.

```rs,hl_lines=18
fn process_record(record: &Record) -> RecordProcessed {
    let text = &record.content;
    let result: String = clean_text(text);
    let tokens: Vec<&str> = result.split_whitespace().collect();
    let (n_m, n_f) = count_gendered_pronouns(tokens);
    RecordProcessed {
        id: record.id,
        publication: record.publication.to_string(),
        author: record.author.to_string(),
        date: record.date.to_string(),
        num_male_pronouns: n_m,
        num_female_pronouns: n_f,
    }
}

fn main(input_path: &PathBuf) {
    let data = load_csv(Path::new(input_path)).unwrap();
    let records = data.par_iter().map(process_record).collect::<Vec<_>>();
    // ... Convert the output paths to strings prior to replacement
    let output_path = output_path.replace(".csv", "_processed.csv");
    let mut wtr = csv::Writer::from_path(Path::new(&output_path)).unwrap();
    _ = records.iter().map(|x| wtr.serialize(x)).collect::<Vec<_>>();
}
```

To collect all the article results prior to writing to CSV, we can apply another `par_iter` call,
where we process the entire set of articles in parallel. There's no need to manually divide the data
into batches, and no need to manage worker processes. Rayon takes care of all of this for us through
its work-stealing scheduler. 😎

The complete code for the Rust implementation can be found [here](https://github.com/thedataquarry/rustinpieces/tree/main/src/parallelism/rust).

## Comparing performance

The Python and Rust implementations were run on the full dataset of 143,000 records
to see how the performance scales to realistic sizes of data. The results are
shown in the table below.

numRecords | Python | Rust | Speedup factor
--- | --- | --- | ---
143,000 | 13.5 sec | 3.4 sec | **4x**

As mentioned at the start of this post, the Rust code using Rayon is about 4x faster than the Python code. This
speedup would likely increase as the number of records increases. The difference
can be explained by the fact that Rayon allows for easy parallelization at multiple stages of the
pipeline, and that it uses work-stealing to distribute the work across multiple CPU cores,
ensuring the most threads are kept utilized with minimum communication overhead.

Python's `ProcessPoolExecutor` does exploit multiple cores, but not as efficiently, and each
batch is processed sequentially. In addition, pure-Python objects are passed around at each stage,
which adds to the overhead while not utilizing the cores to their fullest potential.

## Analysis

It would be a shame to have done all this work without analyzing the results! Let's quickly
load the processed data into a Polars DataFrame and then calculate some statistics.

### Total articles

The results from the three CSV files can be combined into a single DataFrame in Polars. The Python
code is shown below (note that the same can also be done through Polars' Rust API).

```py
import polars as pl

def get_result() -> pl.DataFrame:
    articles1 = pl.read_csv("../data/articles1_processed.csv")
    articles2 = pl.read_csv("../data/articles2_processed.csv")
    articles3 = pl.read_csv("../data/articles3_processed.csv")
    # Combine the data into a single DataFrame
    result = (
        pl.concat([articles1, articles2, articles3])
        .unique(subset=["id"])
        .sort("id")
    )
    print(f"Number of articles: {result.height}")
    return result
```

The total number of records in the combined DataFrame after sorting and de-duplicating is shown below.

```sh
Number of records: 142570
```

### Articles per year

The dates exist as strings, and must first be parsed into a date format before extracting the year.
Null-handling has to be performed as ~7.5k records have null dates.

```py
def get_article_count_by_year(df: pl.DataFrame) -> pl.DataFrame:
    # Parse dates
    result = df.drop_nulls(subset="date").with_columns(
        pl.col("date").str.to_date("%Y-%m-%d", strict=False),
    )
    # Drop nulls and extract year
    result = (
        result.filter(pl.col("date").is_not_null())
        .with_columns(
            pl.col("date").dt.year().alias("year"),
        )
    )
    # Group by year and count
    result_by_year = (
        result.group_by("year")
        .len()
        .sort("year", descending=True)
        .head(5)
    )
    return result_by_year
```

As can be seen below, the most articles in the dataset were written between 2016 and 2017.

```sh
shape: (5, 2)
┌──────┬───────┐
│ year ┆ len   │
│ ---  ┆ ---   │
│ i32  ┆ u32   │
╞══════╪═══════╡
│ 2017 ┆ 48783 │
│ 2016 ┆ 82148 │
│ 2015 ┆ 3653  │
│ 2014 ┆ 108   │
│ 2013 ┆ 228   │
└──────┴───────┘
```

### Highest mean female pronoun count

Counting the raw number of pronouns is not very informative, because the dataset contains an unequal
distribution of articles across publications. We can thus display the *mean* value of male/female
pronouns per publication to get a better sense of the dataset.

```py
def get_pub_with_most_female_pronouns(df: pl.DataFrame) -> pl.DataFrame:
    result = (
        df.group_by("publication")
        .mean()
        .select("publication", "num_male_pronouns", "num_female_pronouns")
        .sort("num_female_pronouns", descending=True)
    )
    return result.head(5)
```

The mean per publication is calculated and sorted in descending order by nunber of female pronouns.

```sh
shape: (15, 3)
┌─────────────────────┬───────────────────┬─────────────────────┐
│ publication         ┆ num_male_pronouns ┆ num_female_pronouns │
│ ---                 ┆ ---               ┆ ---                 │
│ str                 ┆ f64               ┆ f64                 │
╞═════════════════════╪═══════════════════╪═════════════════════╡
│ New York Times      ┆ 18.716904         ┆ 7.383955            │
│ Washington Post     ┆ 15.860806         ┆ 5.739518            │
│ Atlantic            ┆ 16.061847         ┆ 5.564981            │
│ Guardian            ┆ 13.266214         ┆ 5.508467            │
│ Buzzfeed News       ┆ 10.669551         ┆ 5.358467            │
│ …                   ┆ …                 ┆ …                   │
│ Fox News            ┆ 7.71107           ┆ 3.123794            │
│ Breitbart           ┆ 6.774694          ┆ 2.336992            │
│ Talking Points Memo ┆ 6.334676          ┆ 1.247986            │
│ Business Insider    ┆ 5.295545          ┆ 1.206009            │
│ Reuters             ┆ 5.809991          ┆ 1.195238            │
└─────────────────────┴───────────────────┴─────────────────────┘
```

The top 5 publications with the most female pronouns are well-known news outlets like the New York Times,
Washington Post, Atlantic, Guardian and Buzzfeed News. The bottom of the list contains publications
like Reuters, Business Insider, Talking Points Memo, Breitbart and Fox News. Is there a trend? Possibly,
or possibly not. The difference is likely due to the fact that a lot news content covers politicians, businessmen and
sportspersons, who tend to be overwhelmingly male.

In any case, these results were easy to calculate and analyze using the methodology described above,
giving an initial sense of what's in the data.

## Conclusions

In my view as a developer, Rayon's design offers a **perfect** level of abstraction for data parallelism.
There's barely any boilerplate, and the Rust implementation of the code is 4x faster than the Python one.
The Rust implementation has around 10% fewer lines of code than the Python one, while also being more readable and expressive.
Although Rust was expected to be faster all along, it normally has a reputation for being
more complex and verbose than Python. When parallelism is involved, however, the opposite is true,
thanks to Rayon.

The parallelization approaches in Python and Rust could not be more different. Python's `ProcessPoolExecutor` requires the user to specify the maximum number of CPU worker processes to use, batch the data and handle errors. In Rust, Rayon provides parallel iterators and work-stealing to efficiently distribute the work across multiple threads. Because of Rayon's design, the most CPU cores (and associated threads) are kept active at all times without any user input, meaning that the developer can focus on the program's logic and let the library handle the parallelization.

This post showed a relatively simple use case in which we do not need to mutate shared state across threads, but Rayon is capable of far more complex parallelization logic.
It's highly recommended to read the Rayon [FAQ](https://github.com/rayon-rs/rayon/blob/master/FAQ.md) page to learn more about how it works under the hood and try it out for more advanced use cases.

While this post clearly separated Python and Rust into distinct workflows, it's worth noting that Rust can be used to write performance-critical extensions that can be exposed to Python via PyO3, and that Rayon can be used to power such extensions. This is a topic for a future blog post, so stay tuned! Till then, have fun coding in Rust! 🚀

## Code

The code for the Python and Rust implementations is [here](https://github.com/thedataquarry/rustinpieces/tree/main/src/parallelism).
It's part of a larger project called [Rust in Pieces](https://github.com/thedataquarry/rustinpieces),
in which I'm collaborating with [Paul Sanders](https://github.com/sanders41) to compare Python
and Rust for data processing tasks. If you found this post interesting, consider giving the GitHub repo a star ⭐️ and follow along!

---

[^1]: Rayon started off as a hobby project by Niko Matsakis, though it has since grown into a widely used
crate in the Rust ecosystem. See the talk *[Rayon: Data Parallelism for fun and profit](https://www.youtube.com/watch?v=gof_OEv71Aw)* for a great introduction.

[^2]: [Scheduling Multithreaded Computations by Work Stealing](http://supertech.csail.mit.edu/papers/steal.pdf), by Blumofe and Leiserson
