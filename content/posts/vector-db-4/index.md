+++
title = "Vector databases (4): Analyzing the trade-offs"
description = "A deeper dive into some of the trade-offs involved when choosing a vector DB"
date = 2023-08-19
updated = 2023-11-29
draft = false

[taxonomies]
tags = ["vector-db"]
categories = ["Databases"]

[extra]
lang = "en"
toc = true
comment = true
+++


## Choosing the right vector DB solution

Welcome back! In the [previous post](../vector-db-3/) in this 4-part series, we looked at the different types of indexes typically used in vector DBs. However, indexing is just a small part of the bigger elephant in the room when it comes to vector databases. Recall that in [part 2](../vector-db-2/#putting-it-all-together), we described what a vector database *is*. To distinguish between the various vector DB offerings out there, we need to understand the relationships between the following components:

* Application layer, and where it sits
* Data layer, and where it sits in relation to the *database* and the application layer
* Indexing strategy, and how it relates to memory and CPU usage
* Storage layer design
* Scalability and cost considerations in relation to all these aspects

Each of these components involve their own trade-offs. I've broken them down into the following categories -- I'm sure there are more that I've missed, but this is a start. 🤓 Without further ado, let's dive in.

### 1. On-prem vs. cloud hosting

A lot of vendors stress on their "cloud-native" feature as though it's the best thing since sliced bread, from a scalability perspective. However, in deciding what solution works best from the *cost* perspective, it's important to consider how the hosting options are broken down in the first place. Embedded databases architectures are part of the options now! 🎉

Consider the following combinations:

* Cloud-native (managed) + client-server
* On-prem (self-hosted) + embedded
* Cloud-native (managed) + embedded

It's easier to compare these combinations visually.

{{ figure(src="vector-db-hosting-2.png" ) }}

It's clear that the most common kind of database architecture is the client-server architecture, and that's because it's has been battle-tested by various other databases over many years. The embedded/serverless hosting model, are described below, is particularly interesting for the reasons described below.

#### Client-server vs. embedded architectures

Ever since DuckDB monetized its open-source embedded SQL DB offering through MotherDuck[^1], it's served as an inspiration to other upcoming database vendors. In the vector world, one vendor stands out on this front: **LanceDB**[^2], a relatively new entrant -- it utilizes an embedded, serverless design, and its open source version is ready-to-use and scalable from the get go.

{{ figure(src="vector-db-lancedb.png" caption="[lancedb.com](https://lancedb.com/)") }}


Chroma, another well-known offering, also offers an embedded option, but it's purely in-memory and on a single machine, due to which they appear to be more inclined to move toward the tried-and-tested client-server model hosted on the cloud[^3]. Zilliz, which wraps a serverless version of Milvus, also offers an embedded DB for its free version on the cloud[^12]. However, LanceDB is the only one that's built as embedded serverless for all modes of operation.

For organizations that hold a lot of data spread across on-premise and cloud storage, embedded/serverless vector DBs offer a lot of freedom and flexibility to  developers when connecting the data layers to the application layer (where users interact with the data), especially when privacy and security are a concern.

Despite a lot of vendors offering fully-managed, cloud-hosted solutions with secure endpoints deployed within virtual private clouds (VPCs), organizations could be hesitant to move all of their data and vector compute to the cloud. This is especially true for large organizations with highly sensitive data that have to interface with legacy systems that are not cloud-native. In these cases, it's often easier to deploy an embedded vector database like LanceDB on-prem, and very easily connect it to the application layer.


#### Cost considerations

Databases that offer cloud-native, managed solutions typically charge based on the amount of data stored and the number of queries made. This is a great model for organizations that have a lot of data, but don't want to pay for the infrastructure to transport and store it. However, for organizations that have a lot of data, but already have established in-house data infrastructure teams ready to support on-prem or embedded hosting, sending data to the cloud, both for indexing and for querying, makes almost no sense. 

While it's hard to define a "critical mass" of an organization's size at which point this distinction can be made, it's important to know that the standard client-server database architectures deployed on the cloud that we have been used to for many years, are **NOT** the only option in 2023.

Questions to ask:

- Is my dataset growing fast enough that I need to scale  it elastically, on demand, on the cloud?
- Do I have an established enough data infrastructure team to support on-prem/embedded hosting?
- Is my dataset sensitive enough that I need to keep it on-prem?
- What might be the cost considerations of paying per query and per GB of data stored for a fully-managed cloud-hosted solution?

### 2. Purpose-built vendor vs. incumbent vendor

A lot of organizations are already invested in incumbent solutions (like Elasticsearch, Meilisearch, MongoDB, etc.). These solutions are not purpose-built for vector search, but they do add some vector search capabilities on top of their existing ones. If you're looking to add vector/semantic search capabilities on top of an existing application, it makes sense to first at least try out the vector search capabilities of your existing database, and consider the cost implications of these solutions before looking outward.

However, the vector search capabilities offered by incumbent vendors come with certain caveats. Elasticsearch, for example, has a lot of additional constraints placed on what sorts of clients can use their vector search offering, and may require specialized deployments and licensing terms to access their full suite of tools. MongoDB offers vector search *only* for their Atlas cloud deployment, and not on their self-hosted solution. These are important considerations!

The other key fact to keep in mind is this: purpose-built vector DB vendors like Qdrant, Weaviate and LanceDB and many others, have spent thousands of man-hours optimizing their storage, indexing and querying strategies for vector search. They have also had the luxury to build their systems from the ground up, often in modern programming languages like Go or Rust, and as such, are designed for massive scalability and performance. Pretty much every purpose-built vector database solution offers a Python or Javascript client, so it's easy to at least begin testing them on your own data to make a more informed decision.

Questions to ask:

- Do I already have an existing database that I can use for vector search?
- Can I run a simple benchmark on my own data to compare the performance of an incumbent solution vs. a purpose-built solution?
- Will this solution scale as my data grows?

In many cases, the reality is that an existing solution that adds vector search is not going to be as performant as a purpose-built vector database. This can be seen very clearly in this benchmark study[^4] comparing pgvector/PostgreSQL vs. Qdrant -- Spoiler: pgvector is *much* slower than Qdrant on all counts, and offers far fewer optimizations under the hood.

### 3. Insertion speed vs. query speed

Certain vendors like Milvus/Zilliz, have been around for long enough that they are looking at humongous-scale streaming use cases that require vectors. Inserting, and then indexing thousands of vectors per minute may be essential in real-time scenarios like video surveillance or financial transaction tracking, and Milvus/Zilliz are capable of throwing the kitchen sink at this problem[^5].

However, is insertion speed that important to the majority of use cases? For most organizations, querying speed is more important. This is because insertion and indexing are typically done infrequently, but the data might be queried much more frequently, typically in real-time at scale via user interfaces.

Qdrant, an open-source, purpose-built vector DB written in Rust 🦀, optimizes exactly for this use case. Because it's written in Rust, it performs admirably fast in indexing as well, but, as shown in the Qdrant demo on their own documentation page, semantic search-as-you-type, returning vector search solutions that produce relevant results in just a few milliseconds, is actually achievable in a live setting. 🤯

{{ figure(src="qdrant-search-as-you-type.png" alt="Search as you type, <a href='https://twitter.com/qdrant_engine/status/1691054852913418241'>Qdrant Twitter</a>") }}

Questions to ask:

- How often am I going to be inserting (and indexing) a large number of vectors?
- Do I meet the latency requirements of my application at query time?

### 4. Recall vs. latency

Recall is the percentage of relevant results returned by a query, and latency is the time taken to return the results. Different database vendor make different trade-offs when it comes to optimizing for recall vs. latency.

As summarized in [part 3](../vector-db-3/) of this series:

* A `Flat` index is one that stores vectors in their unmodified form, and is used for exact kNN search. It is the most accurate, but also the slowest.
* `IVF-Flat` indexes use inverted file indexes to rapidly narrow down on the search space, which are much faster than brute force search, but they sacrifice some accuracy in the form of recall
* `IVF-PQ` uses IVF in combination with Product Quantization to compress the vectors, reducing the memory footprint and speeding up search, while being better in recall than a pure `PQ` index
* `HNSW` is by far the most popular index, and can be combined with Product Quantization, in the form of `HNSW-PQ`, to produce better recall while improving memory efficiency compared to `IVF-PQ`
* `Vamana` is a relatively new index, designed and optimized for on-disk performance -- it offers the promise of storing larger-than-memory vector data while performing as well, and as fast, as `HNSW`
  * However, it's still early days and not many databases have made the leap towards implementing it due to the challenges of on-disk performance

Keeping all these trade-offs in mind, we can reference the image from earlier in this series.

{{ figure(src="../vector-db-1/vector-db-indexes.png" ) }}

IVF-PQ makes sense for early-stage use cases and POCs where perfect relevance isn't critical to the success of the application. However, for better quality and relevance, most purpose-built vendors use the HNSW index. Of late, vendors like Weaviate and Qdrant have begun combining product quantization (PQ) with HNSW to improve memory efficiency for really large datasets.

Questions to ask:

- How critical is search recall for my use case?
  - Typically, a benchmark study on your own data and queries will help you answer this question.
- How important is latency in my use case?
  - If the dataset really large, then it might make sense to use a product-quantized index like IVF-PQ or HNSW-PQ to reduce memory footprint.

### 5. In-memory vs. on-disk index and vector storage

Databases like Redis are completely in-memory, and are blazing fast. However, it's very plausible that your use case requires storing enough vectors that are larger than memory. A combination of tricks is required to scale vector storage to very large data (hundreds of millions, or even billions of vectors) while also maintaining ANN search speed. Databases like Qdrant and Weaviate provide the option of using memory-mapped files for vectors that utilize the page cache's virtual address space on disk, avoiding loading the entire data into RAM. This helps maintain almost the speed of in-memory databases without actually persisting the data to the disk.

From an indexing perspective, HNSW is known for being the most memory-hungry, and as vector datasets get larger and larger, the natural question that arises is, how well does it scale to out-of-memory indexes? Memory needs can be reduced by combining PQ with HNSW, as described in the previous section. Vamana, a relatively newer index, which is part of the DiskANN algorithm, is among the most promising methods out there (currently available only in LanceDB[^6] and Milvus[^7]), and is claimed to perform on par with HNSW while scaling to larger-than-memory indexes **purely on-disk**.

{% note(header="Note") %}
Of all the database vendors out there, [LanceDB](https://lancedb.github.io/lancedb/ann_indexes/) stands out in this aspect, because it's the **only** one in which *all* supported indexes are disk-based. When responding to a vector query, only the relevant pages from the index file are loaded from disk and cached in memory. It's able to do this largely because LanceDB builds on top of [Lance](https://github.com/lancedb/lance), a new open source columnar vector store that's optimized for fast on-disk vector retrieval.    
{% end %}

Questions to ask:

- If my dataset is really large (>10M vectors), how can I reduce memory consumption? In these cases, reducing the dimensionality of the vectors being stored, tuning the maximum number of graph connections (if using HNSW), or adding product quantization (if your DB supports it) can help.
- Does my database have an option to store vectors on disk, and if so, how does it affect query speed? As always, test on your own data and use case!

### 6. Sparse vs. dense vector storage

The vector embeddings generated by `sentence-transformers` or similar models are dense, meaning that they are composed entirely of non-zero floats. However, it's possible to also use sparse vectors that compute the relative word frequencies per document, in which most of the vector values are zero. Sparse vectors are typically generated by algorithms like BM25 and SPLADE (Sparse Lexical AnD Expansion). Elasticsearch offers its own proprietary pre-trained sparse model for English, ELSER (Elasticsearch Learned Sparse Encoder), that has roughly 30,000 dimensions, but because it's sparse, it's far cheaper to compute and store than an equivalent-length dense vector.

With the advent of `sentence-transformers` and many other transformer models, generating dense vectors from documents has never been easier (or more affordable). The main benefit of dense vectors are that they compress the semantics of language much better than sparse vectors do, due to the underlying embeddings that come from transformers — however, they are more expensive at indexing time, which is definitely a consideration when dealing with datasets of the order of 100M vectors.

Questions to ask:

- How important is semantic search in my use case? If it's very important, then dense vectors are absolutely the way to go
- How important is latency and speed of indexing? If these are very important constraints, then sparse vectors might be worth looking at (though they will never perform as well as dense vectors from a semantic search standpoint)

### 7. Full-text search vs. vector search hybrid strategy

It's well-known that vector search is no panacea. I'll again point the reader to Colin Harman's excellent blog post[^8], where he states this key fact:

> The unfortunate truth is that, for enterprise-level applications, plain vector search is frequently the wrong solution. Many practitioners understand this and often use vector search as a part of their information retrieval system.

The main reason for this is that vector search can rank semantically similar results above exact matches, and in many use cases, we may want exact matches to score higher than a semantically similar match. Also, as languages and terminologies evolve, the "meaning" of certain terms shift, and these may not be captured well enough by the underlying embeddings.  Re-ranking the results from a vector search, typically using scores from a full-text keyword-based search, or by combining indexing strategies (like using a combination of an inverted index and a vector index) can help improve the quality of results. Vespa, a hybrid search engine, that offers an `HNSW-IVF` index, is a good example of this[^9].

A few techniques to handle this trade-off via re-ranking can be considered, and are described below.

* **Naive fallback approach**: This approach uses a “fallback” strategy, where you start with keyword search-as-you-type, using a solution that implements BM25, like [Meilisearch](https://www.meilisearch.com/) or Elasticsearch, with the higher-latency vector search being used in a linear combination with the keyword search results. This is the simplest approach, and doesn't always yield the best results in terms of relevance.

* **Reciprocal Rank Fusion (RRF)**: This approach sums up the reciprocal ranks obtained from sparse and dense vectors to provide a fused ranking score. Databases like Elasticsearch and Weaviate[^10] offer hybrid search using one of many RRF methods.

* **Cross-encoder reranking**: This is the most advanced method for hybrid score re-ranking. It fuses the sparse/dense ranking scores using a neural bi-encoder with a cross-encoding loss function, which typically produces the best results, at the expense of higher latency due to re-ranking via a more expensive approach. These solutions aren't typically offered directly in vector databases -- custom search engines need to be built downstream of the vector DB to perform the re-ranking. Nils Reimers, creator of the `sentence-transformers` library, who is now at Cohere.ai, describes exactly such solutions in an episode of the Weaviate podcast linked below. 😄

{{ youtube(id="KITxQzV97jw" class="youtube") }}

Questions to ask:

- Does my database of choice implement a solid hybrid search strategy? Or will I have to hand-craft a solution downstream of the vector database?
- How is search latency affected by the re-ranking strategy?
- How much latency can I afford to add to my search results by improving accuracy (for e.g., through cross-encoder re-ranking)?

### 8. Filtering strategy

In search, real-world queries are rarely simple textual queries that ask for specific keywords. They typically involve filtering on other metadata attributes. Consider a retail example involving a clothing search (pants, jeans, etc.) of a particular size -- the way these filters are applied to the search result can have a huge impact on the quality of results.

- **Pre-filtered search**: This approach involves naïvely applying the size filter before performing vector search. Although this sounds like a natural thing to attempt, this can lead to a collapse of the HSNW graph into disjoint connected components (as per percolation theory[^11], which defines a threshold at which this happens). As an example, if the user was looking for "jeans" of size 28 which are not in the search catalog, the right way to go would be to at least show related items (like "pants") of the similar size. But, because pre-filtering aggressively prunes all the terms out of the search that don’t meet the size criteria, the full graph isn't traversed and we miss out on relevant results.

- **Post-filtered search**: This approach returns `top-k` nearest neighbours to the query vector ("jeans") and filters them down based on matching sizes after all "jeans" results are retrieved. However, this also has a problem -- we don’t always obtain the same number of results for every query, and if the filtered attribute, i.e., size 28, is a very small fraction of the entire dataset, we may obtain almost no results at all!

- **Custom-filtered search**: Numerous databases apply methods that are in between pre/post-filtering — Weaviate, for example, stores inverted index shards alongside the HNSW index shards, and uses the inverted index to pre-filter much more effectively — the “allow list” obtained from the inverted index, which is a quite large list, is then much more effectively searched via the HNSW index, which considers semantics. Qdrant uses its own filtering method in between pre/post-filtering that ensures a connected HNSW graph under a wide range of conditions by forming edges between categories at index time.

{% warning(header="Note") %}
There is no "one-size-fits-all" solution for filtering results in vector search. Multiple mitigation strategies exist, such as building an additional IVF index to use keywords to assist in the search, as in [Weaviate](https://weaviate.io/blog/hybrid-search-explained), or creating additional HNSW graph connections between *categories*, which helps consider all buckets in the search during filtering, as in [Qdrant](https://qdrant.tech/articles/filtrable-hnsw/). This area is continuously evolving, and highly relevant search & retrieval is an incredibly hard problem to solve, due to these numerous trade-offs.
{% end %}

Questions to ask:

- How does my database of choice handle pre/post filtering?
- For the classes of queries that I will be performing, how well does the pre/post filtering strategy work on my data?

## Conclusions

Whew! This has been a long road toward understanding the internals of vector databases. I've been thinking deeply about this topic for most of 2023, and have been down many rabbit-holes, having had a ton of interesting conversations with developers, CEOs and other folks experimenting with these tools and technologies. 🤓

As vector DBs continue to evolve in this rapidly changing space, I think the key takeaway from this series, at least for me, is that there is no one-size-fits-all solution. The best way to choose a vector database stack is to first understand the requirements and constraints of your use case, and then test out the different solutions on your own data.

In my experience, purpose-built solutions are the superior option, because they have a wider of suite of functions, are able to implement cutting-edge solutions due to starting from a clean-slate, and they also contain a host of optimizations that incumbent vendors just can't prioritize for.

Looking ahead, I'm particularly excited about two databases built in Rust 🦀, namely, Qdrant and LanceDB, which will be my go-to solutions for any POCs and experiments. They're both innovating at a new level in very different ways, and most importantly, they both have a **developer-first** philosophy, with a rapidly growing community. Regardless of which vendor you fancy, I urge you to join me and many others in exploring these tools further! 🤓

If you like listening to podcasts, I speak about these topics at length in the [Practical AI Podcast](https://practicalai.fm/234). Happy learning, and keep coding! 🚀

**Other posts in this series**

- [Vector databases (Part 1): What makes each one different?](../vector-db-1/)
- [Vector databases (Part 2): Understanding their internals](../vector-db-2/)
- [Vector databases (Part 3): Not all indexes are created equal](../vector-db-3/)

---

[^1]: Teach your DuckDB to fly, [MotherDuck](https://motherduck.com/)

[^2]: Developer-friendly, serverless vector database: [LanceDB](https://lancedb.com/)

[^3]: Deployment: [Chroma docs](https://docs.trychroma.com/deployment)

[^12]: Zilliz pricing, [zilliz.com](https://zilliz.com/pricing)

[^4]: pgvector vs. Qdrant, Nirant Kasliwal's [blog](https://nirantk.com/writing/pgvector-vs-qdrant/)

[^5]: Managing data in massive-scale vector search engine, Milvus [blog](https://milvus.io/blog/2019-11-08-data-management.md)

[^6]: ANN indexes FAQ, LanceDB [docs](https://lancedb.github.io/lancedb/ann_indexes/)

[^7]: On-disk index, Milvus [docs](https://milvus.io/docs/disk_index.md)

[^8]: Beware tunnel vision in AI retrieval: Colin Harman, [Substack](https://colinharman.substack.com/p/beware-tunnel-vision-in-ai-retrieval)

[^9]: Billion-scale vector search using hybrid HNSW, by [Jo Kristian Bergum](https://medium.com/vespa/billion-scale-vector-search-using-hybrid-hnsw-if-96d7058037d3)

[^10]: Hybrid search explained, Weaviate blog, by [Erika Cardenas](https://weaviate.io/blog/hybrid-search-explained)

[^11]: Filterable HNSW, Qdrant [blog](https://qdrant.tech/articles/filtrable-hnsw/)