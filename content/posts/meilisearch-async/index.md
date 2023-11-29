+++
title = "Blazing fast async data loading in Meilisearch"
description = "An ETL case study comparing sync and async data loading with Meilisearch using Python and Pydantic"
date = 2023-07-28
updated = 2023-11-29
draft = false

[taxonomies]
tags = ["meilisearch", "async", "pydantic"]

[extra]
lang = "en"
toc = true
comment = true
+++

## Background

If you've ever visited the [Hugging Face model hub](https://huggingface.co/models), you've probably searched for your favourite NLP models by typing in their names, in part or in full. And, very quickly after you stop typing, you obtain a rather relevant set of results, in a matter of milliseconds. How is that possible? While it may seem like the 'search-as-you-type' functionality requires a lot of computing power, with the right underlying search index and database in place, it's actually *not* that expensive. Under the hood, these search functions are powered by Meilisearch, an open-source, lightning fast ⚡️, typo-tolerant search engine and database built in Rust 🦀.

In this article, I'll do a case study comparing an async ETL data loading workflow vs. the commonly used sync workflow used in most situations. I'll also highlight how a modern, lightweight database built in Rust can be used in conjunction with a well-designed async Python client[^1] to *very* rapidly load data and index it under the hood.

### Sync, multi-threaded and async

Before going into the code, let's review the difference between sync, async and multi-threaded programs.

* In a synchronous program, the program will wait for a task to complete before moving on to the next task. In other words, each task "blocks" the executing thread until it is finished.
* In a multi-threaded program, the program will spawn a new thread for each task, and the threads will process each task concurrently
* In an asynchronous program, the *same* thread that starts a task will then move on to the next task while the first task is running. The program will then return to the first task when it is finished with the next one. In other words, each task is "non-blocking", because it yields control to the thread before it's finished.

The following diagram visualizes this distinction -- note how even though the sync and async cases both use single threads (unlike the multi-threaded case), the async case is able to process more tasks in the same amount of time because it doesn't block the thread while waiting for a task to finish. The mechanism through which the async code achieves this is called the *event loop*, whose description is out of the scope of this post, but it's highly recommended to read up on event loops in more detail if you're interested in understanding async programming.

{{ figure(src="meilisearch-async-vs-multithread.png") }}

In most cases, async outperforms multi-threaded, because in spawning multiple threads on the same core, we introduce additional overhead in managing the communication between the threads. The async approach most effectively utilizes the full power of a single thread in a non-blocking manner. However, it also makes the program harder to reason about, due to potential race-conditions[^2] and deadlocks, so it must be used with care.

It is for this reason that most folks handling the lower-level async utilities in Python are library developers, who do the hard work of making sure that the higher-level async libraries are safe to use for end users like us.😅

{% warning() %}
The case study in this post shows ETL code that is either sync or async, but not multi-threaded. The async version uses the `meilisearch-python-sdk` [client](https://github.com/sanders41/meilisearch-python-sdk), maintained by Paul Sanders.
{% end %}

### The data

The dataset being loaded in this case study consists of 130k wine reviews from the Wine Enthusiast magazine, including the variety, location, winery, price, description, and some other metadata for each wine. Refer to the [Kaggle source](https://www.kaggle.com/datasets/zynicide/wine-reviews) for more detailed information on the data and how it was scraped.

An example JSON record is shown below.

```json
{
    "points": "90",
    "title": "Castello San Donato in Perano 2009 Riserva  (Chianti Classico)",
    "description": "Made from a blend of 85% Sangiovese and 15% Merlot, this ripe wine delivers soft plum, black currants, clove and cracked pepper sensations accented with coffee and espresso notes. A backbone of firm tannins give structure. Drink now through 2019.",
    "taster_name": "Kerin O'Keefe",
    "taster_twitter_handle": "@kerinokeefe",
    "price": 30,
    "designation": "Riserva",
    "variety": "Red Blend",
    "region_1": "Chianti Classico",
    "region_2": null,
    "province": "Tuscany",
    "country": "Italy",
    "winery": "Castello San Donato in Perano",
    "id": 40825
}
```

To test the performance of sync vs. async data loading in Meilisearch, we will run a benchmark where the same set of 130k records is loaded in a loop, either 1, 10 or 100 times. This will allow us to see how the performance of the sync and async versions scales with the size of the data.

### ETL steps

The workflow described in this post consists of the following steps:

* Read in a settings config file that describes the indexing settings for Meilisearch
* Read the raw data as a list of records (i.e., JSON blobs)
* Validate the data using Pydantic -- in this post, we use Pydantic v2, which is [5-10x faster than v1](../why-pydantic-v2-matters/).
* Send a list of validated records to Meilisearch for indexing
  * Use either the sync or async Python client for Meilisearch, whose results will be compared
* Verify that the index works as intended by testing search-as-you-type queries

### Meilisearch settings

A key step prior to loading the data into Meilisearch is to define the index settings. This is critical to ensure that the search performance is as expected, and that the index building time is reasonable regardless of the size of the data. The settings configuration is stored as a JSON file, and is read in as a Python dictionary. Some of the important attributes to set are:

* **Searchable attributes**: Even if we have a JSON blob with a lot of fields, it makes sense to ensure that only the fields that will be searched, should have their values indexed
* **Filterable and sortable attributes**: Depending on the user's needs during query time, certain fields should be marked as filterable, and others as sortable. In the wine reviews example here, `price` and `points` should be sortable, while `country` and `variety` should be filterable to offer the most relevant search experience.
* **Ranking rules**: Meilisearch defines rules to decide the relevance of search result, and certain parameters can be prioritized prior to indexing to offer a better search experience:
  * Words: Results are sorted by decreasing number of matched query terms
  * Typo: Results are sorted by increasing number of typos
  * Proximity: Results are sorted by increasing distance between matched query terms
  * Attribute: Results are sorted in the order of a specified attribute list (for example, `variety` matters more than `province`)
  * Sort: Results are sorted according to parameters decided at query time (for example, `price` or `points`)
  * Exactness: Results are sorted by the similarity between matched terms and query terms
  * Custom rules: Sorts results in ascending or descending order for a given attribute

A much more exhaustive guide to optimizing Meilisearch settings to speed up indexing while also ensuring a relevant search is described in their blog[^3]. If you're looking to index large documents with long-form text, it's highly recommended to read through this guide in detail.

### Meilisearch task queue

Just like the data is loaded asynchronously, Meilisearch itself performs tasks asynchronously under the hood[^8]. The data that's loaded into Meilisearch is not immediately available, because it's being indexed in the background. In creating the index, Meilisearch creates roughly 20 data structures, with each batch processed concurrently in the order they came in.

A caveat is, however, as more and more data gets ingested (for really huge datasets numbering in the hundreds of millions of records), the indexing takes progressively longer, especially if long-form text fields are indexed. In any case, searching through these large dumps of data isn't the primary use case for Meilisearch, as described in their blog[^4]. Before indexing any data in Meilisearch, it's always a good idea to understand what indexing involves, and to read the docs to optimize the indexing process. 😅

## Case 1: Sync

The synchronous ETL case uses the official Meilisearch Python client, in conjunction with Pydantic, to ensure that the data is of the right type and quality prior to loading into Meilisearch. I won't go into the details of the Pydantic schema here, as that has been covered in detail in my [earlier post on loading data into Neo4j](../neo4j-python-1/). In a nutshell, the JSON data is read in, and validated against a Pydantic schema. The validated data is then sent to Meilisearch for indexing.

The `main` function for sync ETL is shown below. Note that Meilisearch requires that every record being indexed has a unique identifier which serves as the primary key. In this case, the integer `id` field is used as the primary key. The `Timer` context manager from the `codetiming` package is used to time the indexing process.


```python
import srsly
from codetiming import codetiming
from meilisearch import Client
from meilisearch.index import Index
from tqdm import tqdm


def get_meili_settings(filename: str) -> dict[str, Any]:
    settings = dict(srsly.read_json(filename))
    return settings


def update_documents(filepath: Path, index: Index, primary_key: str, batch_size: int):
    data = list(get_json_data(filepath))
    if LIMIT > 0:
        data = data[:LIMIT]
    validated_data = validate(data)
    index.update_documents_in_batches(
        validated_data,
        batch_size=batch_size,
        primary_key=primary_key,
    )


def main(data_files: list[Path]) -> None:
    meili_settings = get_meili_settings(filename="settings/settings.json")
    config = Settings()
    URI = f"http://{config.meili_url}:{config.meili_port}"
    MASTER_KEY = config.meili_master_key
    index_name = "wines"
    primary_key = "id"

    client = Client(URI, MASTER_KEY)
    with Timer(name="Bulk Index", text="Bulk index took {:.4f} seconds"):
        # Create index
        index = client.index(index_name)
        # Update settings
        client.index(index_name).update_settings(meili_settings)
        print("Finished updating database index settings")
        try:
            # In a real case we'd be iterating through a list of files
            # For this example, it's just looping through the same file N times
            for filepath in tqdm(data_files):
                # Update index
                update_documents(filepath, index, primary_key=primary_key, batch_size=10000)
        except Exception as e:
            print(f"{e}: Error while indexing to db")
```

The `index.update_documents_in_batches()` method available in the Meilisearch client is used, so that we don't have to batch the data in Python -- passing the entire list of records to Meilisearch and letting it handle the batching is much more efficient, as all the underlying operations are done in Rust. The `batch_size` parameter for this method is set to 10k. Running the bulk indexing script on ~130k, 1.3M and 13M records respectively, produce the following timing numbers.

{% codeblock(name="bash") %}
```bash
$ python bulk_index_sync.py -b 1
Finished updating database index settings
100%|██████████████████████████████| 1/1 [00:01<00:00,  1.56s/it]
Bulk index took 2.2505 seconds

$ python bulk_index_sync.py -b 10
Finished updating database index settings
100%|████████████████████████████| 10/10 [00:15<00:00,  1.54s/it]
Bulk index took 16.1129 seconds

$ python bulk_index_sync.py -b 100
Finished updating database index settings
100%|██████████████████████████| 100/100 [02:43<00:00,  1.64s/it]
Bulk index took 164.3521 seconds
```
{% end %}

## Case 2: Async

The async version uses the `meilisearch-python-async` client[^1], whose API is remarkably similar to the sync client's.


```python
import srsly
from codetiming import codetiming
from meilisearch_python_async import Client
from meilisearch_python_async.index import Index
from meilisearch_python_async.models.settings import MeilisearchSettings
from tqdm.asyncio import tqdm_asyncio


def get_meili_settings(filename: str) -> MeilisearchSettings:
    settings = dict(srsly.read_json(filename))
    # Convert to MeilisearchSettings pydantic model object
    settings = MeilisearchSettings(**settings)
    return settings


async def update_documents(filepath: Path, index: Index, primary_key: str, batch_size: int):
    data = list(get_json_data(filepath))
    validated_data = validate(data)
    await index.update_documents_in_batches(
        validated_data,
        batch_size=batch_size,
        primary_key=primary_key,
    )


async def main(data_files: list[Path]) -> None:
    meili_settings = get_meili_settings(filename="settings/settings.json")
    config = Settings()
    URI = f"http://{config.meili_url}:{config.meili_port}"
    MASTER_KEY = config.meili_master_key
    index_name = "wines"
    primary_key = "id"
    async with Client(URI, MASTER_KEY) as client:
        with Timer(name="Bulk Index", text="Bulk index took {:.4f} seconds"):
            # Create index
            index = client.index(index_name)
            # Update settings
            await client.index(index_name).update_settings(meili_settings)
            print("Finished updating database index settings")
            file_chunks = chunk_files(data_files, file_chunksize=5)
            for chunk in tqdm(
                file_chunks, desc="Handling file chunks", total=len(data_files) // 5
            ):
                try:
                    tasks = [
                        # Update index
                        update_documents(
                            filepath,
                            index,
                            primary_key=primary_key,
                            batch_size=10000,
                        )
                        # In a real case we'd be iterating through a list of files
                        # For this example, it's just looping through the same file N times
                        for filepath in chunk
                    ]
                    await tqdm_asyncio.gather(*tasks)
                except Exception as e:
                    print(f"{e}: Error while indexing to db")
        print(f"Finished running benchmarks")
```

The key difference in the code in the async version is how we gather tasks. Each batch of records that needs to be loaded is stored in a list, and is then awaited via `asyncio.gather(*tasks)`. To observe the progress, we use the `tqdm_asyncio.gather(*tasks)` method that wraps a progress bar on top of the running `asyncio` event loop. Running the bulk indexing script on ~130k, 1.3M and 13M records respectively, produce the following timing numbers.

{% codeblock(name="bash") %}
```bash
$ python bulk_index_async.py -b 1
Finished updating database index settings
100%|██████████████████████████████| 1/1 [00:00<00:00,  1.25it/s]
Finished running benchmarks
Bulk index took 1.5184 seconds

$ python bulk_index_async.py -b 10
Finished updating database index settings
100%|████████████████████████████| 10/10 [00:09<00:00,  1.10it/s]
Finished running benchmarks
Bulk index took 9.7602 seconds

# Run 1 for 100 batches
$ python bulk_index_async.py -b 100 --chunksize 5000
Finished updating database index settings
100%|███████████████████████████████████| 100/100 [01:32<00:00,  1.08it/s]
Finished running benchmarks
Bulk index took 93.1814 seconds

# Run 2 for 100 batches
$ python bulk_index_async.py -b 100 --chunksize 2000
Finished updating database index settings
100%|████████████████████████| 100/100 [01:40<00:00,  1.01s/it]
Finished running benchmarks
Bulk index took 101.4677 seconds
```
{% end %}

It's clear that the async data loader is faster than the sync version for all cases.

## Results & discussion

The results of the benchmarking are summarized in the table below. The async version is consistently faster than the sync version, with the difference in performance remaining more or less the same as the number of records being loaded increases. The async version is  at least **~30%** faster than the sync version for all cases.

{{ figure(src="meilisearch-sync-vs-async-results.png") }}

However, in running the async code for 100 batches of data (~13 million records), the script [crashed due to a memory issue](https://github.com/prrao87/db-hub-fastapi/pull/41#issuecomment-1654907416) with the default `CHUNKSIZE` of 10k. I suspect this is because the async batch loader eagerly allocates a certain amount of memory for each batch, causing it to run out of buffer space when many batches are sent at once. 

As a best practice, for very large async data loads, it makes sense to do the following:

* Run smaller batch sizes (e.g. 2k) -- this number depends on your data and the memory available on the machine
* Divide the dataset into smaller chunks, and iterate through these chunks using a synchronous for loop -- this allows you to enforce the execution of each batch sequentially, while still taking advantage of async batch loading *within* each batch.

{% note(header="Note") %}
It's worth noting that the data ingestion is only the first step in the ETL pipeline, which gets data *into* Meilisearch. Once it's ingested, Meilisearch's task queue kicks in, and the indexing steps run asynchronously in the background for a few minutes, following which the data can be queried on the front end. The data is *eventually* available in Meilisearch, just not as soon as it's been loaded. Follow Meilisearch best practices to avoid very large indexing times.
{% end %}

### Search results on the front end

Meilisearch allows developers to test the load by querying the database via a simple front end. A demo of the search is shown below, which returns the most relevant results for the query `2010 cabernet sauvignon`. Note how the search is able to handle typos, which is one of the strengths of Meilisearch's indexing scheme. The results are returned almost as fast as terms are typed in, which is the best part!

{{ figure(src="meilisearch.gif") }}

## Conclusions

In this post, we've seen how to use Meilisearch to build a fast, full-text search engine to search through a sample dataset of wine reviews. We've also seen how to use the Meilisearch Python client (both sync and async) to load data into the database. In general, the async loader will be a fair bit faster than the sync loader, but it's worth testing them both out and tuning the batch sizes to get the best performance with the available memory in your situation, on your specific data.

### When is Meilisearch a great fit?

Meilisearch is an excellent choice for building search-as-you-type interfaces on end user-facing websites when keyword search is the primary search mechanism. In fact, a real-world case study showed that switching to Meilisearch from ElasticSearch improved the search experience on [bookshop.org](https://bookshop.org/), increasing overall conversion by **43%**[^5].

Another area where Meilisearch shines is in its ability to handle typos and misspellings in search queries. This is because Meilisearch implements a host of algorithms to match search terms to the closest terms in the index. This is a very useful feature for search-as-you-type interfaces, where users are highly likely to make spelling mistakes, and the aim is to have relevant results for terms that are off by one or two characters.

Lastly, *faceted* search - where you need to refine search results by broad categories - is also a great use case for Meilisearch. This greatly improves user experience by allowing the user to select filtering by facets, such as brand, size, or rating range as per the e-commerce example image below.

{{ figure(src="meilisearch-facets.png" alt="Image source: Faceted search in <a href='https://blog.meilisearch.com/postgres-full-text-search-limitations/'>Meilisearch blog</a>")}}

### Limitations of Meilisearch

Because Meilisearch was designed from the ground up to be a near-instant search data store, it does not have great support for aggregations or analytics, which are features we might be used to from other NoSQL databases like ElasticSearch and MongoDB. More info on this is provided in an excellent blog post[^4] by the Meilisearch creators themselves.

As they themselves state[^4]:

> Meilisearch is not made to search through billions of large text files or parse complex queries. This kind of searching power would require a higher degree of complexity and lead to slower search experiences, which runs against our instant search philosophy. For those purposes, look no further than Elasticsearch; it’s an excellent solution for companies with the necessary resources, whether that be the financial means to hire consultants or the time and money required to implement it themselves.


In summary, if your goal is to run analytics on your unstructured data, or more complex queries than string-based information retrieval, such as aggregation queries, then, maybe Meilisearch isn't the best choice -- stick to more established alternatives like MongoDB or ElasticSearch that were designed to store humongous amounts of data.


## Code & acknowledgements

As always, the code as well as the dataset used in this post is available on [GitHub](https://github.com/prrao87/db-hub-fastapi/tree/main/dbs/meilisearch/scripts).

Many, many thanks to [Paul Sanders](https://paulsanders.dev/), author of the Meilisearch async Python client[^1], from whom I learned a lot about combining multiprocessing and async workflows in Python[^6] and speeding up and debugging async workflows[^7]. I highly recommend trying out his Python SDK for your upcoming work with Meilisearch!

---


[^1]: Meilisearch async Python client, [Paul Sanders](https://github.com/sanders41/meilisearch-python-sdk)

[^2]: Asyncio race conditions, [Jason Brownlee](https://superfastpython.com/asyncio-race-conditions/)

[^3]: Are you indexing the smart way? Meilisearch blog, [Carolina Ferreira](https://blog.meilisearch.com/best-practices-for-faster-indexing/)

[^4]: Meilisearch vs Elasticsearch, Meilisearch blog, [Carolina Ferreira](https://blog.meilisearch.com/why-should-you-use-meilisearch-over-elasticsearch/)

[^5]: Bookshop.org increases search-based purchases by 43% with Meilisearch, [Maya Shin](https://blog.meilisearch.com/bookshop-increases-search-based-purchases/)

[^6]: Combining Python multi-processor with async, [GitHub PR #15](https://github.com/prrao87/db-hub-fastapi/pull/15)

[^7]: Understanding the memory implications of batch loading with async, [PR #41](https://github.com/prrao87/db-hub-fastapi/pull/41)

[^8]: Tasks and async operations, [Meilisearch docs](https://www.meilisearch.com/docs/learn/async/asynchronous_operations)


