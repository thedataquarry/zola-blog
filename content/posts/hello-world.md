+++
title = "Hello, world!"
description = "Introductory post to the Data Quarry blog"
date = 2023-04-29
draft = false

[taxonomies]
tags = ["rust", "general"]
categories = ["General"]

[extra]
lang = "en"
toc = true
+++

## Welcome to my blog!

Hi there! 👋🏽 I'm an A.I. engineer with a passion for building intelligent, automated applications using a range of databases and software frameworks. You can always catch me nerding out on some cool new tool, library or framework (heaven knows, we have enough to keep track of in the tech world these days 🚀), so feel free to reach out on my socials if you're in Toronto 🇨🇦 and wanna chat. 😄

## What will I focus on in this blog?

I'm a Python programmer currently focusing on building data pipelines and A.I. applications, so I'll be writing about a variety of topics in this space. I'm also learning Rust 🦀, a programming language I genuinely believe is going to revolutionize computing this decade and beyond, so these are the general themes of what I'll be writing about.

## 1. Throwing a spotlight on specific open source tools/frameworks

We all lead busy professional and personal lives, and it can be challenging to spend our precious time contributing to open source projects. Here are my ways of giving back to this amazing community of unsung heroes, at least partially through my writing in this blog:

* Evangelize open source tools and stacks that I enjoy using, by writing about how to use them well
  * This part also falls under the banner of continuous learning and education
  * As I learn from open source tool creators and docs out there for free, I hope to contribute by helping people (no matter how few) learn as well, for free 🤓
* Make code contributions, file issues and fix documentation (no contribution is too small) with existing open source projects
* **Talk** about these tools and stacks, either online or in person! In the world of open source, spreading the word and educating the broader community can be as valuable as contributing code itself

## 2. End-to-end data pipelines and best practices

I regularly build data wrangling and analysis pipelines, and as a result, I come across a variety of really cool open source tools & technologies that are trailblazing a path ahead in the tech ecosystem. Not all of them are nearly well-known enough, in my opinion. My goal is to utilize this space to synergize my thoughts and learnings via conversations with industry colleagues and the amazing open source community out there. Geography and borders have no meaning when it comes to open source! 🌎

## 3. Can we get more people excited about Rust 🦀, already?

I've recently gained a huge fascination with the Rust ecosystem, specifically tools that utilize [PyO3](https://github.com/PyO3/pyo3) to expose the underlying power and speed of Rust to Python developers. This space is really exciting and is changing the way Python tooling is built! As the maintainers of PyO3 [describe](https://github.com/PyO3/pyo3#tools-and-libraries), there are ever more Python frameworks being built on top of Rust these days, all enabled by how easy it's become to build Python tooling on top of blazing fast 🚀 Rust bindings.

There are almost too many things going on in parallel in this space, but because I'm mostly focused on A.I., data engineering and backend tasks, I'll list just some of the many Rust projects that I'm excited about.

### Databases

* [Qdrant](https://qdrant.tech/): High-performant vector database for semantic search and information retrieval, of course built on Rust
* [Meilisearch](https://www.meilisearch.com/): Keyword-based search-as-you-type database written from the ground up in Rust
* [LanceDB](https://lancedb.com/): Serverless, low latency, zero-infrastructure vector search database, utilizing a Rust-powered ultra-fast columnar data structure, [`lance`](https://pypi.org/project/pylance/), that will be orders of magnitude faster than parquet format for nested data structure queries

### Frameworks

* [Pydantic](https://docs.pydantic.dev): Data validation and settings management via Python type annotations. [V2 of Pydantic](https://docs.pydantic.dev/latest/blog/pydantic-v2/) is built entirely on top of Rust 🦀
* [Robyn](https://robyn.tech/): An async web server with a Rust runtime, that [outperforms FastAPI](https://sansyrox.github.io/robyn/#/comparison) on initial benchmarks -- exciting to see where this is headed!

## 4. Other cool tech

Vector databases and graph databases are both raging in the NoSQL world these days, but in different ways, as they both shine in different areas. However, are the use cases for either kind of database truly orthogonal? I'll be exploring this in more detail by querying specific datasets in multiple ways: vector-based similarity search as well as graph-based structural search, to retrieve interesting insights and information. Some of these topics are listed below.

* [FastAPI](https://fastapi.tiangolo.com/): Building RESTful and GraphQL APIs in FastAPI (via [Strawberry](https://strawberry.rocks/))
  * It would be cool to see how the same questions can be answered via two different sets of endpoints (REST and GraphQL), especially on graph and vector databases!
* [Weaviate](https://weaviate.io/): One of the most popular and talked-about vector databases out there, written in [Go](https://go.dev/)
  * Comparing its API with that of Rust-based Qdrant (mentioned above) would be very interesting!
* NLP, vectorization, performance optimization on CPUs, and much more...

## And that's that...

I'll make it a point to try and write as often as possible, with whatever's fun and interesting in the tech world at the time. Onward and upward! 🪐 🔭