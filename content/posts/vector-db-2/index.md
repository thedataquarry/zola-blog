+++
title = "Vector databases (2): Understanding their internals"
description = "A primer on embeddings, semantic similarity and search"
date = 2023-07-09
updated = 2023-11-28
draft = false

[taxonomies]
tags = ["vector-db"]
categories = ["Databases"]

[extra]
lang = "en"
toc = true
comment = true
math = true
+++

## Why is everybody talking about vector DBs these days?

This is the second post in a series on vector databases. As mentioned in [part 1](../vector-db-1/) of this series, there's been a lot of marketing (and unfortunately, hype) related to vector databases in the first half of 2023, and if you're reading this, you're likely curious what vector databases are actually doing under the hood, and how search functionality is built on top of efficient vector storage.

Before going deeper into what vector DBs are, what can explain this frenzy of activity and investment in this space?

#### The age of Large Language Models (LLMs)

In November 2022, an early demo of ChatGPT (which is OpenAI's interface to GPT 3.5 and above) was released, following which it quickly became the [fastest growing application in history](https://www.forbes.com/sites/cindygordon/2023/02/02/chatgpt-is-the-fastest-growing-ap-in-the-history-of-web-applications/), gaining a million users in just 5 days 🤯! Indeed, if you look at the ✨ history on GitHub for some of the major open-source vector database repos, it's clear that there were sharp spikes in the star counts for some of them _after_ the release of ChatGPT in November 2022 and the subsequent release of [ChatGPT plugins](https://openai.com/blog/chatgpt-plugins) in March 2023. These factors, and their associated posts in websites like Hacker News and popular media[^1], are a large part of why there's been such a lot of activity in this space.

{{ figure(src="vector-db-stars.png" alt="Made with ❤️ by <a href='https://star-history.com/#qdrant/qdrant&weaviate/weaviate&milvus-io/milvus&chroma-core/chroma&Date'>star-history.com</a>") }}

#### The problem with relying on LLMs

LLMs are _generative_, meaning that they produce meaningful, coherent text in a sequential manner based on a user prompt. However, when using LLMs to answer a human's questions, they often produce irrelevant or factually incorrect results.

- An LLM often _hallucinates_, i.e., it fabricates information, such as pointing users to URLs or making up numbers that don't exist
- LLMs learn/memorize a compressed version of their training data, and although they learn quite well, they don't do so _perfectly_ -- some information is always "lost" in a model's internal representation of the data
- An LLM cannot know facts that occurred after its training was completed

Vector databases help address these problems, by functioning as the underlying storage layer that can be efficiently queried by an LLM to retrieve facts. Unlike traditional databases, vector DBs specialize in natively representing data as vectors. As a result, we can now build applications with an LLM sitting on top of a vector storage layer that contains recent, up-to-date, factual data (well past the LLM's training date) and use them to "ground" the model, alleviating the hallucination problem.

Although vector databases (e.g., Vespa, Weaviate, Milvus) have existed well before LLMs, since the release of ChatGPT, the open-source community, as well as marketing teams at the vector DB vendors quickly realized their potential in mainstream use cases like search & retrieval in combination with high-quality text generation. This explains the absolute bonanza of VC funding in the world of vector databases!

## What are embeddings?

A vector database stores not only the original data (which could be images, audio or text), but also its encoded form: _embeddings_. These embeddings are essentially lists of numbers (i.e., vectors) that store contextual representations of the data. Intuitively, when we refer to an "embedding", we are talking about a **compressed**, low-dimensional representation of data (images, text, audio) that actually exists in higher dimensions.

Within the storage layer, the database stacks $m$ vectors, each representing a data point using $n$ dimensions, for a total size of $m \times n$. The stacks are typically partitioned via sharding for query performance reasons.

{{ figure(src="vector-embedding.png") }}

## How are embeddings generated?

The transformer revolution in NLP[^2] has provided engineers with ample means to generate these compressed representations, or embeddings very efficiently, and at scale.

- A popular method is via the open source library `sentence-transformers`, available via the [Hugging Face model hub](https://huggingface.co/models?library=sentence-transformers) or directly from [the source repo](https://www.sbert.net/)
- Another (more expensive) method is to use one of many API services:
  - [OpenAI embeddings API](https://platform.openai.com/docs/guides/embeddings)
  - [Cohere embeddings API](https://cohere.com/embed)

It's important to keep in mind that the lower the dimensionality of the underlying vectors, the more compact the representation is in embedding space, which can affect downstream task quality. Sentence Transformers (sbert) provides embedding models with a dimension $n$ in the range of 384, 512 and 768, and the models are completely free and open-source. OpenAI and Cohere embeddings, which require a paid API call to generate them, can be considered higher quality due to a dimensionality of a few thousand. One reason it makes sense to use a paid API to generate embeddings is if your data is multilingual (Cohere is known to possess high-quality multilingual embedding models that [are known to perform better](https://docs.cohere.com/docs/multilingual-language-models#model-performance) than open source variants).

{% warning(header="Information") %}
The choice of embedding model is typically a trade-off between quality and cost. In most cases, for textual data in English, the open-source `sentence-transformers` model can be utilized as is for text that isn't too long (~300-400 words for text sequences). It's possible to deal with text that's longer than the context length of `sentence-transformers`, which requires more external tools, but that topic's for another post!
{% end %}

## Storing the embeddings in vector databases

Because of their amenability to operating in embedding space, vector databases are proving to be very useful for _semantic or similarity-based search_ for multiple forms of data (text, image, audio). In semantic search, the input query sent by the user (typically in natural language) is translated into vector form, in the same embedding space as the data itself, so that the top-k results that are most similar to the input query are returned. A visualization of this is shown below.

{{ figure(src="embedding-pipeline.png") }}

## How is similarity computed?

The various vector databases offer different metrics to compute similarity, but for text, the following two metrics are most commonly used:

- **Dot product**: This produces a non-normalized value of an arbitrary magnitude
- **Cosine distance**: This produces a normalized value (between -1 and 1)

### An example of measuring cosine distance

Consider a simplified example where we vectorize the titles of red and white wines in 2-D space, where the horizontal axis represents red wines and the vertical axis represents white wines. In this space, points that are closer together would represent wines that share similar words or concepts, and points that are further apart do not have that much in common. The cosine distance is defined as the cosine of the angle between the lines connecting the position of each wine in the embedding space to the origin.

$$cos \theta = \frac{a^T \cdot b}{|a| \cdot{ |b|}}$$

A visualization will make this more intuitive.

{{ figure(src="wines-cosine.png") }}

On the left, the two wines (the Reserve White and the Toscana Red) have very little in common, both in terms of vocabulary and concepts, so they have a cosine distance approaching zero (they are orthogonal in the vector space). On the right, the two Zinfandels from Napa Valley have a lot more in common, so they have a much larger cosine similarity closer to 1. The limiting case here would be a cosine distance of 1; a sentence is always perfectly similar to itself.

Of course, in a real situation, the actual data exists in higher dimensional vector space (and has many more axes than just the variety of wine) and cannot be visualized on a 2-D plane, but the same principle of cosine similarity applies.

## Scalable nearest neighbour search

Once the vectors are generated and stored, when a user submits a search query, the goal of similarity search is to provide the top-k most similar vectors to the input query's own vector. Once again, we can visualize this in a simplified 2-D space.

{{ figure(src="knn.png") }}

The most naive way to do this would be to compare the query vector with each and every vector in the database, with the so-called k-nearest neighbour (kNN) method. However, **this quickly becomes way too expensive** as we scale to millions (or billions) of data points, as the number of comparisons required keeps increasing linearly with the data.

### Approximate nearest neighbours (ANN)

Every existing vector database focuses on making search highly efficient, regardless of the size of the dataset, via a class of algorithms called **Approximate Nearest Neighbour** (ANN) search. Instead of performing an exhaustive comparison between every vector in the database, an approximate search is performed for nearest neighbours, resulting in some loss of accuracy in the result (the _truly_ nearest neighbour may not always be returned), but a massive performance gain is possible using ANN algorithms.

## Indexing

Data is stored in a vector database via _indexing_, which refers to the act of creating data structures called indexes that allow efficient lookup for vectors by rapidly narrowing down on the search space. The embedding models used typically stores vectors with a dimensionality of the order of $10^2$ or $10^3$, and ANN algorithms attempt to capture the actual complexity of the data as efficiently as possible in time and space.

There are many indexing algorithms used in the various vector DBs available, and their details are out of the scope of this post (I'll be studying these in future posts). But for reference, some of them are listed below.

- Inverted File Index (IVF)
- Hierarchical Navigable Small World (HNSW) graphs
- Vamana (utilized in the DiskANN implementation)

In a nutshell, the state-of-the-art in indexing is achieved by newer algorithms like HNSW and Vamana, but only a few database vendors offer the DiskANN implementation of Vamana (as of 2023):

- Milvus
- Weaviate (WIP)
- LanceDB (WIP)

## Putting it all together

We can combine all the above ideas to form a mental picture of what a vector database actually _is_.

{{ figure(src="vector-db-components.png") }}

The specifics of how each database vendor achieves scalability (via Kubernetes, sharding, streaming and so on) are not important for practitioners -- it's up to each vendor to architect the system considering the trade-offs between latency, cost and scalability.

### Storage layer and data ingestion

- The data that sits somewhere (locally or on the cloud) is passed to an embedding model, converted to vector form and ingested into the storage layer of a vector DB via the API gateway
- The data is indexed, during which it's partitioned/sharded for scalability and faster lookups
- The query engine is tightly integrated with the storage layer, to allow for rapid retrieval of nearest neighbours via the database's ANN implementation

### Application layer

- A user sends a query via an application's UI to the embedding model, which converts the input query to a vector that lies in the same embedding space as the data
- The vectorized query is sent to the query engine via the API gateway
  - Multiple incoming queries are handled asynchronously, and the top-k results are sent back to the user

## Extending vector databases to serve other functions

The use case above shows how vector DBs enable semantic search at a scale that was not possible several years ago, unless you were a big tech company with massive resources. However, this is just the tip of the iceberg: vector DBs are used to power a host of downstream functions.

### Hybrid search systems

In his excellent review post[^3], Colin Harman describes how a lot of companies, due to the plethora of vector DB marketing material out there today, experience "tunnel vision" when it comes to the search & retrieval landscape. As practitioners, we have to remember that vector databases are not the panacea of search -- they are very good at _semantic_ search, but in many cases, traditional keyword search can yield more relevant results and increased user satisfaction[^4]. Why is that? It's largely to do with the fact that ranking based on metrics like cosine similarity causes results that have a higher similarity score to appear above partial matches that may contain specific input keywords, reducing their relevance to the end user.

However, pure keyword search also has its own limitations -- in case the user enters a term that is semantically similar to the stored data (but is not exact), potentially useful and relevant results are not returned. As a result of this trade-off, real-world use cases for search & retrieval demand a combination of keyword and vector searches, **of which vector databases form a key component** (because they house the embeddings, enabling semantic similarity search and are able to scale to very large datasets).

To summarize the points above:

- **Keyword search**: Finds relevant, useful results when the user _knows_ what they're looking for and expects results that match exact phrases in their search terms. Does **not** require vector databases.
- **Vector search**: Finds relevant results when the user _doesn't_ know what exactly they're looking for. Requires a vector database.
- **Hybrid keyword + vector search**: Typically combines candidate results from full-text keyword and vector searches and re-ranks them using cross-encoder models[^5] (see below). Requires both a document database and a vector database.

This can be effectively visualized per the diagram below:

{{ figure(src="vector-hybrid-search.png" alt="Diagram inspired by <a href='https://qdrant.tech/articles/hybrid-search/'>Qdrant blog post</a>") }}

BM25[^6] is the most common indexing algorithm used for keyword search in certain databases (e.g., Elasticsearch, Opensearch, MongoDB). It produces a _sparse_ vector, by considering keyword term frequencies in relation to their inverse document frequency (IDF). In contrast, vector databases typically encode and store text in embeddings represented by _dense_ vectors (none of the terms in the vector are zero, unlike BM25), and this is typically done via a bi-encoder model like BERT, that produces a sentence embedding for a pair of documents, that can then be compared to produce a cosine similarity score.

#### Understanding the difference between bi-encoders and cross-encoders

To effectively perform hybrid search, it becomes necessary to combine search result candidates obtained via BM25 (keyword search) and cosine similarity (vector search), which requires a _cross-encoder_. This is a downstream step, as shown in the image below, that allows two sentences to be simultaneously passed to an encoder model like BERT. Unlike the bi-encoder that's used to produce sentence embeddings, a cross-encoder doesn't produce embeddings, but rather, it allows us to classify a pair of sentences by assigning them a score between 0 and 1, via a softmax layer. This is termed **_re-ranking_**, and is a very powerful approach to obtaining results that combine the best of both worlds (keyword + vector search).

{{ figure(src="vector-encoders.png" alt="Diagram inspired by <a href='https://www.sbert.net/examples/applications/cross-encoder/README.html'>Sentence transformers docs</a>") }}

{% important(header="Important") %}
It should be noted that re-ranking via cross-encoders is an expensive step, as it requires the use of a transformer model during query time. This approach is used when the quality of search is critical to a use case, and requires more compute resources (typically, GPUs) and tuning time to ensure that the application is performing as intended.
{% end %}

## Generative QA: "Chatting with your data"

With the advent of powerful LLMs like GPT-4, we can effectively integrate the user experience of an application with clean, factual information stored in a vector DB, allowing the user to query their data via natural language. Because question-answering can involve more than just information retrieval (it may require parts of the data to be analyzed, not just queried), including an agent-based framework like LangChain[^7] in between the application UI and the vector DB can be much more powerful than just plugging into the vector DB directly.

{{ figure(src="vector-db-qa.png") }}

Because vector DBs store the data to be queried as embeddings, and the LLM also encodes the knowledge within it as embeddings, they are a natural pairing when it comes to generative QA applications. The vector DB functions as a knowledge base, and the LLM can query a subset of the data directly in the embedding space. This can be done using the following approach:

1. Human asks a question in natural language via the UI
2. The question's text is passed to an embedding model (bi-encoder), which then returns a sentence embedding vector
3. The question vector is passed to the vector DB, which returns the top-k most similar results, obtained via an ANN search
    - This step is **crucial**, as it massively narrows down the search space for the LLM, used in the next step
4. An LLM prompt is constructed (based on the developer's predefined template), converted to an embedding, and passed to the LLM
    - A framework like LangChain makes it convenient to perform this step, as the prompt can be dynamically constructed and the LLM's native embedding module called without the developer having to write a lot of custom code for each workflow
5. The LLM searches the top-k results for the information, and produces an answer to the question
6. The answer is sent back to the human

The above workflow can be extended in many ways -- for example, if the user's question involved some arithmetic on numbers obtained from the database (which LLMs are notoriously bad at), a LangChain agent could determine, first, that arithmetic is required. It would then pass the numerical information extracted from the top-k results to a calculator API, perform the calculations, and then send the answer back to the user. With such composable workflows, it's easy to see how powerful, generative QA chat interfaces are enabled by vector DBs.

## Conclusions

There are many other useful applications that can be built combining the inherent power of LLMs and vector DBs. However, it's important to understand some the underlying limitations of vector DBs:

- They do not necessarily prioritize exact keyword phrase matches for relevance in search applications.
- The data being stored and queried in vector databases must fit inside the maximum sequence length of the embedding model used (for BERT-like models, this is no longer than a few hundred words). Currently, the best way to do so is to utilize frameworks like LangChain and LlamaIndex[^8] to chunk or squash the data into a fixed-sized vector that fits into the underlying model's context.

Vector databases are incredibly powerful, but the bottom line, per Harman[^3], applies to all of them:

> If you're not very knowledgeable about vector databases, and in general, the search and information retrieval space, the following materials **should NOT** be your primary sources of information for any kind of decision-making on choosing your tech stack:
>
> - Infrastructure stacks from super-smart VCs
> - Tutorials from popular LLM application frameworks

Just like in any other domain, clearly defining the business case and studying the tools at hand allows you to combine them effectively to solve real-world problems. In that regard, I hope this series on vector DBs was helpful so far!

**Other posts in this series**

- [Vector databases (Part 1): What makes each one different?](../vector-db-1/)
- [Vector databases (Part 3): Not all indexes are created equal](../vector-db-3/)
- [Vector databases (Part 4): Analyzing the trade-offs](../vector-db-4/)

---

[^1]: The rise of vector databases, [Forbes](https://www.forbes.com/sites/adrianbridgwater/2023/05/19/the-rise-of-vector-databases/?sh=7014b16514a6)

[^2]: Revolution in NLP is changing the way companies understand text, [TechCrunch](https://techcrunch.com/sponsor/nvidia/how-the-revolution-of-natural-language-processing-is-changing-the-way-companies-understand-text/)

[^3]: Beware tunnel vision in AI retrieval: Colin Harman on [Substack](https://colinharman.substack.com/p/beware-tunnel-vision-in-ai-retrieval)

[^4]: Vector search for clinical decisions, [Haystack US 2023](https://colinharman.substack.com/i/126262800/haystack-us-erica-lesyshyn-and-max-irwin-vector-search-for-clinical-decisions)

[^5]: On hybrid search, [Qdrant blog](https://qdrant.tech/articles/hybrid-search/)

[^6]: Okapi BM25, [Wikipedia](https://en.wikipedia.org/wiki/Okapi_BM25)

[^7]: LangChain [Python docs](https://python.langchain.com/docs/get_started/introduction.html)

[^8]: LlamaIndex [docs](https://gpt-index.readthedocs.io/en/latest/index.html)
