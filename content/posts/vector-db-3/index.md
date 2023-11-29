+++
title = "Vector databases (3): Not all indexes are created equal"
description = "Understanding Flat, Annoy, IVF, HNSW and Vamana vector indexes in vector databases"
date = 2023-07-24
updated = 2023-11-29
draft = false

[taxonomies]
tags = ["vector-db"]

[extra]
lang = "en"
toc = true
comment = true
math = true
+++

## Organizing vector indexes

This is the third post in a series on vector databases. [Part 1](../vector-db-1/) compared the offerings of various DB vendors and how they are different at a high level, while [Part 2](../vector-db-2/) focused on the basics of what vector DBs are and what they do.

You may have already come across the excellent post "*Not all vector databases are made equal*"[^1] by Dmitry Kan, which covered the differences between various vector DBs in the market back in 2021. The  vector DB landscape has been continuously evolving since then, and because each one is quite different from the others, I thought it made sense to do a deeper dive into indexing methods in this post.

Assuming that it's amply clear [what a vector database *is*](../vector-db-2/#putting-it-all-together), it's worth taking a step back to wonder, how does it all scale so wonderfully to be able to search millions, billions, or even trillions[^2] of vectors? As we know, the primary aim of a vector database is to provide a fast and efficient means to store and *semantically* query data, in a way that the vector data type is a first-class citizen. The similarity between two vectors is gauged by distance metrics like cosine distance or the dot product.

When working with vector databases, it's important to distinguish between the *search algorithm*, and the underlying *index* on which the Approximate Nearest Neighbour (ANN) search algorithm operates. As in most situations, choosing a vector index involves a tradeoff between accuracy (precision/recall) and speed/throughput.

Having scoured the literature through the course of 2023, I find that vector indexing methods can be organized in two levels: a) by their data structures, and b) by their level of compression. These classifications are by no means exhaustive, and many sources disagree on the right way to organize the various indexes, so, this is my best attempt at making sense of it all. Here goes! 😅

## Level 1: Data structures

It helps to start by organizing indexes based on the data structures that are used to construct them. This is best explored visually.

{{ figure(src="vector-db-indexing-types.png" alt="Breaking down vector indexes by their underlying data structures") }}

### Hash-based index

Hash-based indexes like LSH (Locally Sensitive Hashing) transform higher dimensional data into lower-dimensional hash codes that aim to keep the original similarity as much as possible. During indexing, the dataset is hashed multiple times to ensure that similar points are more likely to collide (which is the opposite of conventional hashing techniques, where the goal is to minimize collisions). During querying, the query point is also hashed using the same hash functions as used during indexing, and because the similar points are assigned to the same hash bucket, retrieval is very fast. The main advantage of hash-based indexes is that they are very fast while scaling to huge amounts of data, but the downside is that they are not very accurate.

### Tree-based index

Tree-based index structures allow for rapid searches in high-dimensional spaces, via binary search trees. The tree is constructed in a way that similar data points are more likely to end up in the same subtree, making it much faster to discover approximate nearest neighbours. **Annoy** (Approximate Nearest Neighbours Oh Yeah) was such a method that uses a forest of binary search trees, developed at Spotify. The downside of tree-based indexes is that they perform reasonably well only for low-dimensional data, and are not very accurate for high-dimensional data because they cannot adequately capture the complexity of the data.

### Graph-based index

Graph-based indexes are based on the idea that data points in vector space form a graph, where the nodes represent the data values, and edges connecting the nodes represent the similarity between the data points. The graph is constructed in a way that similar data points are more likely to be connected by edges, and the ANN search algorithm is designed to traverse the graph in an efficient manner. The main advantage of graph-based indexes, is that they are able to find approximate nearest neighbours in high-dimensional data, while also being memory efficient, increasing performance. HNSW and Vamana, explained below, are examples of graph-based indexes.

An extension of graph-based indexes to include concepts from tree-based indexes is NGT[^3] (Neighbourhood Graphs and Trees). Developed by Yahoo! Japan Corporation, it performs two constructions during indexing: one that transforms a dense kNN graph into a bidirectional graph, and another that incrementally constructs a navigable small world (NSW) graph. Where it differs from pure graph-based indexes is in its use of range search via a tree-like structure (Vantage-point, or 'VP' trees), a variant of greedy search during graph construction. Because both constructions result in nodes that have a high outward degree, to avoid combinatorial explosion, the seed vertex from which the search originates uses the range search to make the traversal more efficient. This makes NGT a hybrid graph and tree-based index.

### Inverted file index

Inverted file index (IVF) divides the vector space into a number of tessellated cells, called Voronoi diagrams -- these reduce the search space in the same way that clustering does. In order to find the nearest neighbours, the ANN algorithm must simply locate the centroid of the nearest Voronoi cell, and then search only within that cell. The benefit of IVF is that it helps design ANN algorithms that rapidly narrow down on the similarity region of interest, but the disadvantage in its raw form is that the quantization step involved in tessellating the vector space can be slow for very large amounts of data. As a result, IVF is commonly combined with quantization methods like product quantization (PQ) to improve performance, described below.

## Level 2: Compression

The second level on which indexes can be organized is their compression level: a "flat" or brute force index is one that stores vectors in their unmodified form. When a query vector is received, it is exhaustively compared against each and every vector in the database, as shown in the simplified example below in 3-D space. In essence, using such an index would be like doing a kNN search, where the returned results are exact matches with the $k$ nearest neighbouring vectors. As you can imagine, the time required to return results would increase linearly with the size of the data, making it impractical when applied on a dataset with more than a few hundred thousand vectors.

{{ figure(src="vector-db-flat-index.png" alt="Flat index for kNN (exhaustive) search: Image inspired by <a href='https://www.pinecone.io/learn/series/faiss/vector-indexes/'>Pinecone blog</a>") }}

The solution to improve search efficiency, at the cost of some accuracy in the retrieval, is compression. This process is called *quantization*, where the underlying vectors in the index are broken into chunks made up of fewer bytes (typically via converting floats to integers) to reduce memory consumption and computational cost during search.

{{ figure(src="vector-db-indexing-compression.png") }}

### Flat indexes

When using ANN (non-exhaustive) search, an existing index like IVF or HNSW is termed "flat" when it directly calculates the distance between the query vector and the database vectors in their raw form. To differentiate this from the quantized variants. When used this way, they are called IVF-Flat, HNSW-Flat, and so on.

### Quantized indexes

A quantized index is one that combines an existing index (IVF, HNSW, Vamana) with compression methods like quantization to reduce the memory footprint and to speed up search. The quantization is typically one of two types[^4]: Scalar Quantization (SQ), or Product Quantization (PQ). SQ converts the floating-point numbers in a vector to integers (which are much smaller in size in bytes) by symmetrically dividing the vector into bins that account for the minimum and maximum value in each dimension.

PQ is a more sophisticated method that considers the distribution of values along each vector dimension, performing *both* compression and data reduction[^4]: The idea behind PQ is to decompose a larger dimensional vector space into a cartesian product of smaller dimensional subspaces by quantizing each subspace into its own clusters -- vectors are represented by short codes, such that the distances between them can be efficiently estimated from their codes, termed *reproduction values*. An asymmetric binning procedure is used (unlike SQ), which increases precision[^5], as it incorporates the distribution of vectors within each subspace as part of the approximate distance estimation. However, there is a trade-off as it does reduce recall quite significantly[^6].

---
## Popular indexes

Among all the indexing methods listed so far, most purpose-built vector databases implement only a select few of them. This is a very rapidly evolving space 🔥, so a lot of information here may be out of date when you're reading this. It's highly recommended you check out the latest documentation of the databases you're interested in, to see what indexes they support. In this section, I'll focus on a few popular and upcoming indexes that multiple vendors are focusing on.

### IVF-PQ

IVF-PQ is a composite index available in databases like Milvus and LanceDB. The IVF part of the index is used to narrow down the search space, and the PQ part is used to speed up the distance calculation between the query vector and the database vectors, and to reduce the memory requirements by quantizing the vectors. The great part about combining the two is that the speed is massively improved due to the PQ component, and IVF component helps improve the recall (that is normally compromised) by the PQ component alone.

The PQ component can be broken down as per the diagram below. Each vector representing a data point consists of a fixed number of dimensions $d$ (of the order of hundreds or thousands, depending on the embedding model used upstream). Because storing these many 32 or 64-bit floating point numbers can be quite expensive on a large dataset, product quantization approaches this problem in two stages: the first stage is a coarse quantization stage where the vector is divided into $m$ subvectors, each of dimension $d/m$, and each subvector is assigned a quantized value (termed "reproduction value") that maps the original vectors to the centroid of the points in that subspace[^7].

The second stage is similar to k-means clustering, where a "codebook" of values is learned by minimizing the distance between the original vector and the quantized vector centroids. By mapping a large, high-dimensional vector into smaller, lower-dimensional subvectors, it is only the codebook of quantized values that is stored, making the memory footprint far smaller.

{{ figure(src="vector-db-pq.png" alt="Product quantization: Image inspired by [Pinecone blog](https://www.pinecone.io/learn/series/faiss/product-quantization/)") }}

IVF is then applied to the PQ vector space -- for each point in the space there is a corresponding region, called a [Voronoi cell](https://en.wikipedia.org/wiki/Voronoi_diagram), consisting of all points in the space closer to the source point (seed) than to any other. These seed points are used to create an inverted index that correlates each centroid with a list of vectors in the space.

{{ figure(src="vector-db-ivf-cells.png") }}

Depending on where the query vector lands, it may be close to the border of multiple Voronoi cells, making it ambiguous which cells to return nearest neighbours from, leading to the *edge problem*. As a result, IVF-PQ indexes involve setting an additional parameter, `n_probes`, that tells the search algorithm to expand outward to the number of cells specified by the `n_probes` parameter.

To effectively build an IVF-PQ index, it is necessary to make two choices: the number of subvectors for the PQ step, and the number of partitions for the IVF step. Larger the number of subvectors, the smaller each subspace is, reducing information loss due to compression. However, a larger number of subvectors also results in more I/O and computation within each PQ step, so this number must be minimized to keep computational costs low.

Similarly, the number of partitions in the IVF step must be chosen to balance the trade-off between recall and search speed. The limiting case of number of partitions being equal to the number of vectors in the dataset is *brute force search*, which is the most accurate (recall of 1), but it essentially makes it an IVF-Flat index.

Indeed, the LanceDB IVF-PQ index docs[^8] describe exactly these kinds of trade-offs when creating an index, so it's worth going deeper into these concepts to understand how to apply them in a real world scenario.

### HNSW

Hierarchical Navigable Small-World (HNSW) graphs is among the most popular algorithms for building vector indexes -- as of writing this post, nearly *every* database vendor out there uses it as the primary option. It's also among the most intuitive algorithms out there, and it's highly recommended that you give the [original paper](https://arxiv.org/abs/1603.09320) that introduced it, a read.

At a high level, HNSW builds on top of the [small world graph](https://en.wikipedia.org/wiki/Small-world_network) phenomenon, which states that even though most nodes in a graph are not neighbours of each other, the *neighbours* of any given nodes are likely to be neighbours of each other, regardless of the size of the graph. Essentially, any node in the graph can be reached from every other node by a relatively small number of steps. In the example below, the solid-filled blue and red nodes can be reached from each other in just 3 hops, even though they might be perceived as distant in the graph.

{{ figure(src="vector-db-small-world-graph.png") }}

The vector space in which your data lives can also be thought of as a *Navigable* Small World (NSW) graph, where the nodes represent the data points, and edges represent similarities, i.e., numbers that describe how close two nodes are to each other in vector space. NSW works by constructing an undirected graph that ensures global connectivity, i.e., any node can be reached in the graph given an arbitrary entry point. Long edges (connecting nodes that are far apart and require many traversals) are formed first, and short edges (which connect nearby nodes) are formed later on. The long edges improve search *efficiency* and the short edges improve search *accuracy*[^3]. The nearest nodes to a given query vector can be found by traversing this graph.

{{ figure(src="vector-db-nsw.png") }}

One of the problems with an NSW graph is that it is flat -- certain nodes can create dense "traffic hubs", reducing the efficiency of the traversal and causing the search complexity of the method to be poly-logarithmic[^9]. HNSW addresses this issue via a *hierarchical* graph structure and also fixes the upper bound of each node's number of neighbours, reducing the search complexity to logarithmic[^9]. The basic idea is to separate nearest neighbours into layers in the graph based on their distance scale. The long edges in the graph are kept in the top layers (which is the sparsest layer), with each layer below containing edges that are shorter-distance than the layers above it. The lowest layer forms the complete graph, and the search is performed from top to bottom. If all layers are "collapsed" into one another, the HNSW graph is essentially an NSW graph.

{{ figure(src="vector-db-hnsw.png") }}

The image above shows how, given an arbitrary entry point at the top layer, it's possible to rapidly traverse across the graph, dropping one layer at a time, until the nearest neighbour to the query vector is found.

The biggest strength of HNSW over IVF is that it is able to find approximate nearest neighbours in complex, high-dimensional vector space with a high degree of recall. In fact, at the time of its release ~2019, it produced state-of-the-art results on benchmark datasets specifically with regard to improving recall while also being fast, explaining its immense popularity. However, it is not as memory efficient, unless it is combined with methods like PQ to compress the vectors at search time. Databases like Qdrant and Weaviate, typically implement composite indexes that involve quantization, like HNSW-PQ[^10] for these reasons, while also offering tuning knobs to adjust the trade-off between recall and query latency.

### Vamana

Vamana is among the most recently developed graph-based indexing algorithms, first presented at NeurIPS 2019 by [Subramanya et al.](https://proceedings.neurips.cc/paper_files/paper/2019/file/09853c7fb1d3f8ee67a61b6bf4a7f8e6-Paper.pdf) in collaboration with Microsoft Research India.

The standout features of Vamana are:

* It was designed from the ground up to work both in-memory (which most indexes are designed to do) as well as *on-disk*, whose implementation as presented by Microsoft, is termed [DiskANN](https://github.com/microsoft/DiskANN).
  * On-disk indexes seem to be proving a huge implementation challenge for vector database vendors, so this is a key feature of Vamana that differentiates it from other algorithms
* It allows the indexing of datasets that are too large to fit in memory by constructing smaller indexes for overlapping partitions, that can be easily merged into one single index whose query performance is on par with single indexes constructed for the entire dataset
* It can also be combined with off-the-shelf vector compression methods like PQ, to build a Vamana-PQ index that powers a DiskANN system -- the graph index with the full-precision vectors of the dataset are stored on disk, whereas the compressed vectors are cached in memory, achieving the best of both worlds

Vamana iteratively constructs a directed graph, first starting off with a random graph, where each node represents a data point in vector space. At the beginning, the graph is well-connected, meaning nearly all nodes are connected to one another. The graph is then optimized using an objective function that aims to maximize the connectivity between nodes that are closest to one another. This is done by pruning most of the random short-range edges, while also adding certain long-range edges that connect nodes that are quite distant from one another (to speed up traversals in the graph).

{{ figure(src="vector-db-vamana-build.png") }}

During query time, the entry point is chosen to be the global centroid. The search rapidly progresses in the right direction via the long-range edges, which allow the algorithm to jump to the ends of the graph and narrow down on the nearest neighbour of the query vector relatively quickly. In the example below, it takes just three hops to traverse from the entry point, which is the global centroid, to the outer edge of the graph, and then to the nearest neighbour.

{{ figure(src="vector-db-vamana-query.png") }}

As you might have observed already, Vamana offers more of an "inside-out" approach to search, as opposed to the "outside-in" approach of HNSW, where the search starts from a random (potentially far out) node in the top layer and progresses inwards.

There are not many databases that currently (as of 2023) implement the Vamana index, presumably due to the technical challenges with on-disk implementations and their implications on latency and search speed. Milvus[^11], for now, is the only vendor that has a working, on-disk Vamana index, whereas Weaviate[^12] and LanceDB[^13] currently only have experimental implementations. However, this is a rapidly evolving space, so it's highly recommended that you follow up on the key vector DB vendors to stay up to date on the latest developments!

## Indexes in popular vector DBs

As shown in [part 1](../vector-db-1/) of this series, most databases implement the HNSW index as the default option.

{{ figure(src="../vector-db-1/vector-db-indexes.png") }}

Databases like Milvus, Weaviate, Qdrant and LanceDB offer simple tuning knobs to control the compression/quantization levels in their Product Quantization components. It's important to understand the fundamentals of how these indexes work, so that you can make the right choice of database and indexing parameters for your use case.

## Conclusions

The makers of purpose-built vector databases have spent thousands of man hours fine-tuning and optimizing their indexes and storage layers, so if you have large datasets and require <100 ms latency on vector search queries, resorting to full-fledged, scalable open-source databases like Weaviate, Qdrant and Milvus seems like a no-brainer, both from a developer's and a business's perspective.

* A `Flat` index is one that stores vectors in their unmodified form, and is used for exact kNN search. It is the most accurate, but also the slowest.
* `IVF-Flat` indexes use inverted file indexes to rapidly narrow down on the search space, which are much faster than brute force search, but they sacrifice some accuracy in the form of recall
* `IVF-PQ` uses IVF in combination with Product Quantization to compress the vectors, reducing the memory footprint and speeding up search, while being better in recall than a pure `PQ` index
* `HNSW` is by far the most popular index, and is often combined with Product Quantization, in the form of `HNSW-PQ`, to improve search speed and memory efficiency compared to `IVF-PQ`
* `Vamana` is a relatively new index, designed and optimized for on-disk performance -- it offers the promise of storing larger-than-memory vector data while performing as well, and as fast, as `HNSW`
  * However, it's still early days and not many databases have made the leap towards implementing it due to the challenges of on-disk performance

In my view, however, [LanceDB](https://lancedb.github.io/lancedb/) is among the most exciting databases to watch out for in the coming months. This is because it is the **only** vector database in the market where *all vector indexes are disk-based*. This is because they are innovating on multiple fronts all at once:

1. Building a new, efficient columnar data format, [Lance](https://github.com/lancedb/lance), that is aimed at becoming a modern successor to parquet, while also being optimized for vector search
   * It's because of this highly efficient storage layer that LanceDB is able to proceed with such confidence on the disk-based indexing front, unlike other vendors
2. Embedded (serverless), purpose-built architecture built from the ground up
3. Zero-copy data access, which is a huge performance boost for disk-based indexes
4. Automatic versioning of data without needing additional infrastructure
5. Direct integrations with cloud storage providers like AWS S3 and Azure Blob Storage, making it very easy to integrate with existing data pipelines

Regardless of which database you choose for your use case, we can all agree that this is a great time to be alive and experimenting with these tools. I know there was a lot of information in this article, but the references below really helped me understand the internals of vector databases and how they are built from the ground up. I hope you found this post interesting, and let's keep learning! 🚀

**Other posts in this series**

- [Vector databases (Part 1): What makes each one different?](../vector-db-1/)
- [Vector databases (Part 2): Understanding their internals](../vector-db-2/)
- [Vector databases (Part 4): Analyzing the trade-offs](../vector-db-4/)

---

[^1]: Not All Vector Databases Are Made Equal, [Dmitry Kan on Medium](https://towardsdatascience.com/milvus-pinecone-vespa-weaviate-vald-gsi-what-unites-these-buzz-words-and-what-makes-each-9c65a3bd0696)

[^2]: Trillion-scale similarity, [Milvus blog](https://milvus.io/blog/Milvus-Was-Built-for-Massive-Scale-Think-Trillion-Vector-Similarity-Search.md)

[^3]: A Comprehensive Survey and Experimental Comparison of Graph-Based Approximate Nearest Neighbor Search, [arxiv.org](https://arxiv.org/abs/2101.12631)

[^4]: Choosing the right vector index, [Frank Liu on Substack](https://thesequence.substack.com/p/guest-post-choosing-the-right-vector)

[^5]: Product quantization for nearest neighbor search, [J'egou, Douze & Schmid](https://lear.inrialpes.fr/pubs/2011/JDS11/jegou_searching_with_quantization.pdf)

[^6]: Scalar Quantization and Product Quantization, [Frank Liu on Zilliz blog](https://zilliz.com/blog/scalar-quantization-and-product-quantization)

[^7]: Product quantization: Compressing high-dimensional vectors by 97%, [Pinecone blog](https://www.pinecone.io/learn/series/faiss/product-quantization/)

[^8]: ANN indexes, [LanceDB docs](https://lancedb.github.io/lancedb/ann_indexes/))

[^9]: Hierarchical Navigable Small-World graphs paper, [Malkov & Yashunin](https://arxiv.org/abs/1603.09320)

[^10]: HNSW + PQ, [Weaviate blog](https://weaviate.io/blog/ann-algorithms-hnsw-pq)

[^11]: On-disk index, [Milvus docs](https://milvus.io/docs/disk_index.md#On-disk-Index)

[^12]: Vamana vs. HNSW, [Weaviate blog](https://weaviate.io/blog/ann-algorithms-vamana-vs-hnsw)

[^13]: Types of index, [LanceDB docs](https://lancedb.github.io/lancedb/ann_indexes/#types-of-index)