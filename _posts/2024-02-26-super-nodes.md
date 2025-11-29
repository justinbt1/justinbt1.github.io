---
title: 'Neo4j Super Node Performance Issues'
date: 2024-02-26
permalink: /posts/neo4j-super-node-performance-issues
tags:
  - data science
  - statistics
---
We recently developed a multi-billion relationship scale knowledge graph, representing the wider academic landscape. How we developed the data pipeline to build this graph in Neo4j is a story in itself, one I may write up here or on the [Wellcome Data blog](https://medium.com/wellcome-data). This post however focuses on a specific database modelling issue that has caused us severe query performance issues, the issue of low cardinality "super nodes".

## What are Super Nodes?
Nodes with a large number of relationships (there is no formal number, but roughly 100k or more relationships) are known as "super nodes". These high degree nodes may occur either due to the underlying structure of the real world network the graph represents, or the presence of low cardinality node types (nodes with a small number of possible categorical values) in the data model. The possible underlying causes of super nodes are well explained by David Allen in his blog post [here](https://medium.com/neo4j/graph-modeling-all-about-super-nodes-d6ad7e11015b).

For us it was a case of our data outgrowing our original data model, with performance issues surfacing only once we had fully populated our knowledge graph. These issues presented as very slow queries when traversing or filtering relationships on certain node types, sometimes taking several days for relatively simple queries to run. On investigation I noticed these node types contained many nodes that were acting as super nodes in our graph.

Super nodes can impact performance in a number of ways:

- The default token lookup index Neo4j creates for each relationship only contains relationship type information. This means Neo4j has to assess all connections from a node it is traversing in a query, so it can identify the next node in the traversal.

- Dense nodes can also increase the number of relationships within a given relationship type, which may make queries slower.

- When creating a new relationship between nodes in the database, it is common practices to use MERGE to check such a relationship does not already exist. Neo4j will check if the relationship yet exists, which will be slower where there are a large number of relationships.

- Super nodes can also lead to issues with scalability, as everything in the network is close to everything else, making the data hard to break into partitions.

## Field of Research Example
As an example our data model included Field of Research (FoR) nodes, which each represent a broad category of scientific research as defined by ANZSRC. There are 100 million Publication nodes in our database, each linked to one or more of the 176 Field of Research nodes.

In a best case scenario, where the number of relationships is evenly distributed, each field of research would be linked to approximately 568,000 publications. However in reality publications from a small number of fields of research are over represented in our database, meaning millions of publications may have a relationship with a single field of research.

For example the below Cypher query returns all researchers who have published clinical science papers in 2020:

```cypher
MATCH
(r:Researcher)-[a:AUTHORED]->(p:Publication {year:2020})
-[]-(f:FieldOfResearch {name:"Clinical Sciences"})
RETURN r
```

This query would take an extremely long time to execute as several million unindexed relationships between publications and the clinical science FoR node need to be checked during query execution.

