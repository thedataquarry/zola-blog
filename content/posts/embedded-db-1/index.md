+++
title = "Embedded databases (1): The harmony of DuckDB, KùzuDB and LanceDB"
description = "A look at how embedded databases enable easy navigation between relational, graph and vector paradigms"
date = 2023-08-27
draft = false

[taxonomies]
tags = ["embedded-db", "vector-db", "graph-db"]
categories = ["Tools"]

[extra]
toc = true
comment = true
+++

## Embedded DBs are here to stay

In the world of database systems, the client/server architecture has been by far the most widespread. Databases whose names you're likely very familiar with, like PostgreSQL, MySQL, MongoDB and many others, fall in this camp, whose foundations were laid in the early days of the web. Back then, the client/server architecture was a natural fit because it allowed for a separation of concerns between the client (the browser) and the server (the machine hosting the database).

Most databases that exist today are built using the client/server architecture, where the client is responsible for rendering the user/developer interface, and the server is responsible for storing and processing the data. It's not a stretch to say that the world has progressed a *long* way since the early days of the web. As computing power has grown exponentially, modern embedded databases are able to do a lot more, *with a lot less*.

Although the idea of embedded databases is not new -- SQLite has been around since 2000! -- there have been numerous research breakthroughs in the last decade, making it possible for vendors to build fast, lightweight, and easy to use alternatives to established solutions like Postgres or MySQL, especially for the massive analytical and ML workloads we're seeing today.

However, a lot of these enterprise solutions are designed for "big" data (whatever that means for your organization). But there's a whole ocean of use cases *in between* single-CPU, in-memory analytics and large-scale, distributed analytics. It's in this middle ground where embedded databases shine ✨, because they're designed to be lightweight, easy to use, and are extremely performant for analytics, from small-scale (a few million) to large-scale (billion-size) datasets.

The aim of this series is to first gain a birds-eye view of the embedded database landscape, and how their interplay with Apache Arrow ecosystem is allowing for greater flexibility in data modelling than was possible before. From an OLAP perspective, three particular vendors are really exciting: **DuckDB** in the relational world, **KùzuDB** for graphs, and **LanceDB** for vectors.

## Some background

Real-world data processing workloads can be broadly broken into two: **OLTP** (online transactional processing and) **OLAP** (online analytical processing). OLTP workloads involve a lot of small, fast transactions, like updating a user's profile, or adding a new item to a shopping cart. OLAP workloads, on the other hand, involve a lot of complex, long-running queries, like aggregating over a large dataset, or joining several large tables.

Over the years, the client/server architecture has been a great fit for OLTP workloads, but has been less than ideal for OLAP workloads, mainly because their underlying storage is row-oriented. This has led to a host of OLAP data warehouses being built specifically to process heavy analytical workloads, such as Snowflake, Clickhouse and BigQuery, which are designed to use columnar storage.

I'm focusing specifically on the 3 solutions shown below, because they're all open-source, and importantly, **they embrace unorthodoxy** at their core -- they do away with existing designs, and instead innovate by starting from a clean slate.

{{ figure(src="embedded-db-breakdown.png" alt="The unifying interface for modern data: Arrow" ) }}

The bottom part of the image is particularly interesting: [Arrow](https://arrow.apache.org/), a language-agnostic development ecosystem for in-memory data, is the common denominator between all three of these databases, allowing data from one paradigm to relatively easily be transformed into another (despite the databases themselves implementing different data models). The Arrow format was designed from the ground up to be a columnar, fast and in-memory format for flat and hierarchical data, to be efficient for analytical operations on modern CPUs and GPUs.

### What is an embedded database?

An embedded database is an **in-process** database management system that's tightly integrated with the application layer. The term "in-process" is important because the database compute runs within the same underlying process as the application (which could be written in any language, like Python, R, JavaScript, C++). In the case of RocksDB, an open-source embedded key-value store written in C++, the application it runs inside[^6] could *itself be another database*!

A key characteristic of embedded databases is how close the storage layer is to the application layer. Additionally, data that's larger than memory can be stored and queried on-disk, allowing them to scale to pretty huge amounts of data (TB) with relatively low query latencies and response times.

{% note() %}
In the database community, embedded databases are sometimes referred to as *serverless* databases, but they do not mean the same thing. In general, "embedded" refers to "in-process", and "serverless" refers to separation of storage and compute, so these terms are used in different contexts, especially to those coming from microservices backgrounds.
{%end%}

### A breakdown of the landscape

The three databases we will focus on are but part of a larger landscape of embedded DBs, many of which are out of the scope of this series. However, it makes sense to first gain a birds-eye view of the breakdown of the landscape.

{{ figure(src="embedded-db-landscape.png" alt="Embedded databases organized by data model paradigm" ) }}

Key-value embedded stores are quite popular, due to their speed and lightweight nature, allowing them to power a host of other applications/databases downstream. However, the key-value data model is quite simplistic, allowing for limited expressivity in data modelling. The other three data model paradigms -- relational, graph and vector -- are much more powerful and expressive, and are thus the focus of this series.

---

## DuckDB

[DuckDB](https://duckdb.org/) is a high-performance embedded, ACID-compliant relational database system (RDBMS) that can be queried via a rich SQL dialect that's very similar to Postgres. It's written in C++, and is designed to be fast and easy to use. It's designed to support large-scale OLAP query workloads, which are typically characterized by complex, relatively long-running queries that process significant portions of the stored data -- for example, aggregations over entire tables, or joins between several large tables.

### An interesting take on the "big data" narrative

In early 2023, the makers of DuckDB posted a blog with a click-baity title, "*Big data is dead*" [^1]. It was very smartly (and aptly) countered in another article that came shortly after, "*Big data is dead ... Long live big data!*"[^2]. Both articles are very well-written, and contain a lot of insights from folks who have spent years in the world of big data, so it's recommended you give them both a read.

* The thesis: [Big data is dead](https://motherduck.com/blog/big-data-is-dead/), by Jordan Tigani, a founding engineer at Google BigQuery, now founder and CEO of MotherDuck, DuckDB's commercial cloud offering.
* The antithesis: [Big data is dead, long live big data!](https://ponder.io/big-data-is-dead-long-live-big-data/), by Aditya Parameswaran, Associate Professor at UC Berkeley, and co-founder of Ponder, a data science startup.

The first blog post makes the (rather strong) claim that the age of big data is over, and that because a large fraction of organizations don't have Google-scale data, they are thus "overpaying" to enterprise data warehouse vendors for features or scale that they don't need, and a lot of their workloads could instead be run on single nodes.

While this is largely true, the second post rightly counters this viewpoint (while also acknowledging that DuckDB is an excellent solution), by stating that although a large fraction of organizations don't *yet* have big data, they will have big data *in the future*, because the relationship between having data and the ability to grow it (by better leveraging insights from existing data), is nonlinear. Organizations will *eventually* reach humongous scales of data, provided they have the right tools to organically arrive at that scale, and it all starts with having multiple tools that allow them to process data at all scales, from small to large.

I'm very much on board with both camps! Everything has to be taken in context, for each individual use case. 😎

### Key features of DuckDB

All jokes aside, DuckDB is a *seriously* powerful database, with a host of unique and amazing features that are straight from the world of academic database research. As they mention in their blog[^3], DuckDB stands on the shoulders of giants. A few of their key features for blazing fast OLAP query performance on large datasets (involving aggregations and joins on 100M+ rows on multiple tables), are listed below.

* Like other OLAP data warehouses, DuckDB uses columnar storage, which is a great fit for analytical workloads, because it allows for fast, efficient scans over large amounts of data
* Uses vectorized query execution, which is a technique that allows for processing large amounts of data in batches, which is a great fit for modern CPUs
* Utilizes recent advances in query optimization, with ideas from dynamic programming to unnesting arbitrary subqueries
* Uses concurrent execution via threads, which allows for faster execution of queries, and is a great fit for modern multi-threaded CPUs
* Utilizes a dialect of SQL that is very similar to Postgres (unlike Clickhouse, which deviates quite far from PostgreSQL)
* **Possibly the most important**: DuckDB has fast become a universal data connector[^5], due to the sheer number of data formats it's able to natively read data from
  * A lot of ETL processes involve expensive transformations that reshape data from one form to another
  * DuckDB natively reads from formats like CSV, JSON (including nested JSON), Parquet, and has scanners to directly read from Postgres and SQLite databases
  * Thanks to the Arrow format, data can very easily move from a DuckDB table to a Pandas or Polars DataFrame, and vice-versa
  * It also offers connectors that allow users to directly read Parquet data from S3, GCS and Azure storage

Because DuckDB **natively** supports a lot of these formats, it's able to perform efficient scans on-disk, without having to materialize them in-memory all the time (unlike Pandas).

{{ figure(src="embedded-db-duckdb.png" alt="A productive DuckDB setup for large (100M+ size) analytical workloads" ) }}

## KùzuDB

[KùzuDB](https://github.com/kuzudb/kuzu) is an open-source graph database management system (GDBMS), built for query speed and scalability, and is implemented in C++. Its origins and motivations are quite similar to DuckDB on two counts, in that it utilizes an embedded architecture, and that it came from an academic environment. KùzuDB is being actively developed at the University of Waterloo 🇨🇦, and builds on top of decades of research in the implementation and architectural design of graph database management systems. It utilizes the [openCypher](https://opencypher.org/) standard to implement a fully-functional, performant graph query language that allows users to access the full expressive power of graphs via the labelled property graph model.

Despite the term "graph", a graph database, at its core, expresses a relational model. The main difference in the internals of a GDBMS and a typical relational system is that the GDBMS is optimized for storing and querying specialized data structures that represent a graph data model, making it highly suited to datasets with a high degree of connectivity like social networks, recommendation engines, fraud detection, and many others.

### Key features of KùzuDB

KùzuDB, being an embedded database built from scratch, incorporates some cutting-edge features straight out of the world of academic database research. A few of their key features for blazing fast graph query performance are listed below, as adapted from their excellent blog post, titled "*What every competent GDBMS should do"*[^7].

* **Vectorized query processing**: A graph database exploits the underlying relational structure of the data and stores it via an efficient columnar format in blocks, so that queries and aggregations can be processed in a vectorized fashion, fully exploiting the power of multiple threads on modern CPUs.

* **Pre-defined, pointer-based joins**: A graph database stores the neighbours of a *node* (a "record", or a "row" in SQL terminology) as pre-defined relationships. While in a relational DB, a SQL query can make no prior assumptions about which tables are being joined with each other until query time, a graph database is all about exploiting the prior knowledge of existing relationships from the data, and instead uses a join index (i.e., and adjacency list index) to store these pre-defined relationships *at load time*.

* **Many-to-many growing joins**: A graph database is natively designed for many-to-many joins, on data that's ever growing. If on average, each of the nodes connects with many other nodes and there are also many relationships in the pattern being searched, we're basically asking the system to search through an exponentially growing number of combinations! Relational (OLTP) databases, because of their row-oriented design, cannot optimize for this use case.

* **Recursive joins**: Graph queries excel in performance for recursive join queries when compared with SQL, mainly because of the efficiency of the graph model in "hopping" over multiple levels of depth. Although SQL queries permit recursion, the idea of recursion was added as an afterthought in the relational data model, whereas in graphs, recursion is a first-class citizen.

* **Schema querying**: A special feature of graph querying that cannot be done in SQL is directly querying the *schema* of the database. A graph query in Cypher contains a subject, a predicate, and an object, and we are able to define the predicate directly on the schema, not just on the nodes/relations. This allows us to express a data modelling logic that would be a lot messier in SQL. To express the predicate (connective) logic between entities in relational tables, we would need to write a verbose query involving `UNION`s and other sub-queries, but this is just one line in Cypher (see the blog post[^7] for examples).

* **Semi-structured data handling**: In many cases, we may have data that contains deeply nested JSON, involving cases where one entity can have many types (a node representing Justin Trudeau can be both of type `Person`, and of type `Politician`, depending on the logic being expressed in the data model). A labelled property graph data model allows assigning multiple labels to these entities, providing added flexibility to the developer while designing the schema for the kinds of queries expected in the application.

### Goals of KùzuDB

The goals and vision of Kùzu are very well articulated in their blog post[^7], but the main summary is that it's designed to be a fast, scalable and easy-to-use solution for graph data science, graph machine learning (via frameworks like PyTorch Geometric) and analytics on very large graphs (upwards of 100M nodes and 1B edges). It's very well-integrated with the Python data science ecosystem, with client libraries in C++, Rust, Node.js, Java and of course, Python.

{{ figure(src="embedded-db-kuzudb.png" alt="Where Kùzu sits in the graph data science & ML Stack" ) }}

## LanceDB

[LanceDB](https://lancedb.com) is an open-source embedded database for vector search built with persistent storage, which greatly simplifies retrieval, filtering and management of embeddings. LanceDB's core is written in Rust 🦀 and is built using Lance, an open-source columnar format designed for performant ML workloads that's 100x faster than parquet[^9] for random access.

### Key features of LanceDB

Other than the fact that it uses an embedded architecture, what makes LanceDB different from the sea of other vector stores out there? I've spent some time thinking about this, and talked numerous folks in the industry, to come up with the list below.

* Built on top of a new, efficient columnar data format, Lance, that is aimed at becoming a modern alternative to parquet that's optimized for vector search
  * Building on top of highly efficient disk-based format like Lance allows LanceDB to proceed with confidence on its own version of the DiskANN algorithm, a modern and performant ANN index -- it's very likely that a lot of other vector DBs cannot implement DiskANN as efficiently on top of their own storage layers
* Can be queried in a number of ways, including via SQL, full-text search (via Tantivy[^8]), and vector search (IVF-PQ, or an upcoming DiskANN index).
* Zero-copy data access, which is a huge performance boost for disk-based indexes
* Automatic versioning of data via Lance[^10]
* Direct integrations with cloud storage providers like AWS S3 and Azure Blob Storage, making it possible to directly query data stored on the cloud, with no added ETL steps
* Truly multi-modal data access, where the vector embeddings are stored alongside the actual document, not just its metadata. This allows you to persist images, audio or text documents and their embeddings in the same storage location, unlike other vector DBs, where the raw data sits separately from the embeddings/metadata.

{{ figure(src="embedded-db-lancedb.png" alt="Using LanceDB to power <a href='https://github.com/lancedb/vectordb-recipes/tree/main'>data science, retrieval & ML workflows</a>" ) }}

---

## Will embedded databases be commercially successful?

The million-dollar question now is how commercially successful embedded databases will be in the long run. The client/server architecture has been around for a *very* long time, and has been proven to be a successful commercial model for numerous databases, which is why they are the norm in large-scale production use cases. The embedded architecture is still relatively new, at least for OLAP databases, and several vendors focusing on this section of the market are building them out as open-source technology, and are still figuring out their monetization strategies. However, the approach seems to be along the lines of the following:

1. Build a sound open-source offering with the full functionality of their client/server counterparts
1. Showcase blazing fast performance 🔥 and ease of use
1. Gain a sizeable user community that advocates for its use
1. Offer a managed cloud service that builds on top of the open-source version, with added convenience features -- from a pricing perspective, it would be essential that the cost is on par with (or lower than) tried-and-tested client/server databases

One can only speculate, and **time** will tell how successful the various monetization strategies of embedded databases will be. My personal take is that there is already ample opportunity in both small and large organizations to use embedded databases for a wide variety of use cases, and the three vendors I've mentioned in this post, in my view, seem the best-positioned in their domains to be 💰 commercially successful in the long run.

### DuckDB + MotherDuck

In early 2023, DuckDB made a significant announcement related to its monetization strategy, with its [MotherDuck](https://motherduck.com/company/) commercial entity spin-off entity. MotherDuck offers the following convenience features on top of open-source offering DuckDB[^4], designed specifically for serverless operation on the cloud.

* Convenient persistent storage via a managed service
* Hybrid execution mode, allowing seamlessly combining querying from in-memory, on-disk, or on-cloud, in a fully distributed fashion
* Secrets management to handle data stored on the cloud
* A notebook-like SQL IDE (similar to DataBricks Spark notebooks) to make data scientists and analysts more productive
* Sharing databases with teammates

### KùzuDB

Kùzu is the furthest ahead among graph DB vendors in the quest to provide an easy-to-use and scalable embeddable graph DBMS.

* [Kùzu](https://kuzudb.com/) is a powerful, open-source, ACID-compliant graph database **ready for production**, with great support for the openCypher query language
  * My [experiments](../embedded-db-2) with it show it to be blazing fast in comparison to existing solutions, and it's able to handle large-scale graph queries using a familiar query language.
* As they mention in their blog[^7], Kùzu is aiming to emulate in the graph database world what DuckDB has done in the SQL world, and gain widespread adoption through a sound core that's open-source and scalable
* As the use cases for graph data structures & algorithms proliferate into ML/AI and LLM applications, my take is that an as-yet unannounced commercial offering from KùzuDB could be hugely valuable in building analytics tools powered by graphs

### LanceDB + LanceDB Cloud

LanceDB has been making waves in the world of vector DBs, and it's fast differentiated itself from its competitors by offering innovative disk-based vector indexes and search capabilities directly on cloud storage[^8]. Due to the unique nature of [Lance](https://github.com/lancedb/lance), a new, columnar data format optimized for vector compute (and on top of which LanceDB is built), the management of vector storage and versioning on the cloud is a fascinating area that LanceDB is looking poised to revolutionize. Some of the convenience features on [LanceDB Cloud](https://lancedb.com/) are described below.

* Managed service to store and version Lance datasets in the cloud
* Distributed computing workloads via a managed service
* Auto re-indexing
* Caching layer
* Dashboard to more conveniently manage your data

## Conclusions

This post gave a rather detailed (and in my view, necessary) overview of the embedded DB landscape, specifically focusing on solutions in the relational, graph and vector paradigms. Although each of these tools is built in a different language (C++ or Rust) and caters to different data models, the interoperability features baked into underlying Arrow format effectively unify these otherwise distinct paradigms, allowing for seamless data transfer and a LOT more flexibility for complex use cases.

I'm firmly of the belief that these major developments will make embedded databases far more popular and accessible to data scientists and ML practitioners in the coming years. As a developer who's interested in one and all modelling paradigms, I can only speculate on the commercial angle of these solutions. In the upcoming posts in this series, I'll go deeper into examples with code on each of these solutions.

Onward and upward! 🚀

Other posts in this series:

- [Embedded databases (2): KùzuDB, an extremely fast OLAP graph database](../embedded-db-2)
- [Embedded databases (3): LanceDB and the modular data stack](../embedded-db-3)

---

[^6]: Building on top of RocksDB, [Cockroach Labs blog](https://www.cockroachlabs.com/blog/cockroachdb-on-rocksd/)

[^1]: *Big data is dead* by Jordan Tigani, [MotherDuck blog](https://motherduck.com/blog/big-data-is-dead/)

[^2]: *Big data is dead... Long live big data!* by Aditya Parameswaran, [ponder.io](https://ponder.io/big-data-is-dead-long-live-big-data/)

[^3]: Standing on the shoulders of giants, [DuckDB docs](https://duckdb.org/why_duckdb.html#standing-on-the-shoulders-of-giants)

[^5]: Why we built Rill with DuckDB, [Rill blog](https://www.rilldata.com/blog/why-we-built-rill-with-duckdb)

[^7]: What every competent graph database management system should do, [KùzuDB blog](https://kuzudb.com/docusaurus/blog/what-every-gdbms-should-do-and-vision/)

[^9]: Benchmarking random access in Lance, [LanceDB blog](https://blog.eto.ai/benchmarking-random-access-in-lance-ed690757a826)

[^8]: *S3 backed full-text search with Tantivy* by Rob Meng, [LanceDB blog](https://blog.lancedb.com/s3-backed-full-text-search-with-tantivy-part-1-ac653017068b)

[^10]: Building a time machine with Lance, [LanceDB blog](https://blog.lancedb.com/building-a-time-machine-with-lance-3b14ab536232)

[^4]: DuckDB vs. MotherDuck, [Kestra blog](https://kestra.io/blogs/2023-07-28-duckdb-vs-motherduck)