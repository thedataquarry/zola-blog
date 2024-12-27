+++
title = "Graph RAG (1): Build a simple system using Kùzu and ell"
description = "Understand the basics of Graph RAG and use Kùzu and ell, a language model prompting framework, to build a simple Graph RAG system."
date = 2024-10-24
updated = 2024-10-24
draft = true

[taxonomies]
tags = ["graph-db", "rag", "llm"]
categories = ["Tools"]

[extra]
toc = true
comment = true
+++

If you've been following the buzz around Graph RAG and how graphs can be used in conjunction with LLMs, you're in the right place!
In this blog post, we'll describe how to build a simple Graph RAG system using Kùzu, an embedded graph database,
and [ell](https://docs.ell.so/), a language model prompting framework that makes it incredibly
easy to use LLMs as part of your workflows. Before we get into the tutorial, it's worth understanding
some terminology about the terms "graph" and "RAG", as the meaning of these terms is often context-dependent,
depending on where you're coming from.

## What is a graph?

Graphs are an abstract representation of a set of objects (or nodes) and the relationships between them (or edges).
You could choose to model and work with graphs in-memory (as is done in [NetworkX](https://networkx.org/)),
or you could choose to store and persist your graphs in a database on disk, as we do in [Kùzu](https://github.com/kuzudb/kuzu).
As readers of this blog know, Kùzu is an embedded graph database that makes it simple to transform your existing data into a graph on disk,
and query it using Cypher, a declarative graph query language.

## Unpacking the term "RAG"

The term "RAG", or Retrieval Augmented Generation, broadly refers to a system that uses the retrieved
results from a search engine or database as part of the input to a large language model (LLM) to generate a response
in natural language. The term "RAG" was popularized by two papers in early 2020[^1] [^2], and has since been
increasingly used to power a variety of applications, including question answering agents, chatbots and domain-specific search engines.

With the arrival of large language models and vector databases, the term "RAG" has become synonymous with
the use of vector embeddings to power retrieval, and the use of LLMs to power generation -- today, these techniques
are called "traditional RAG". In traditional RAG, unstructured text data is divided into chunks, each of which is
fed to an embedding model to produce a vector embedding. A vector database is then used to store these embeddings,  
and a similarity search is used to retrieve the most relevant chunks for a given query. The retrieved chunks are then
fed as context to an LLM, which uses the context to generate a response.

## The emergence of Graph RAG

The term "Graph RAG" is used to describe a system that uses a graph database to power retrieval, and an LLM to power generation.
This does not mean that vector databases and embeddings from traditional RAG are unnecessary -- in fact, the two techniques
can be used in conjunction with each other. However, for the purposes of this blog post, we'll use the term "Graph RAG"
to refer to a system that retrieves from a graph database (Kùzu), and passes the retrieved results as context to an LLM for
generation.

The usage of Graph RAG is still in its nascent stages, and there are several interesting research directions that
are being explored. When thinking about building Graph RAG systems, it's important to utilize a framework of thinking
that answers the following key questions:

1. **What is the graph schema?** What are the key entities and relationships in the graph?
2. **How does the graph get populated?** How are the nodes and edges populated in the graph?
3. **How does the graph get queried?** How does the graph get queried to retrieve the most relevant information?

---

[^1]: REALM: Retrieval-Augmented Language Model Pre-Training. Guu, K., et al. (2020). [arXiv:2002.08909](https://arxiv.org/abs/2002.08909).
[^2]: Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks. Lewis P. et al. (2020). [arXiv:2005.11401](https://arxiv.org/abs/2005.11401).
