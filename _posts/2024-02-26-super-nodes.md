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

## Improving Perfomance
### Query Refactoring
The simplest way we can improve performance is to refactor our queries to make them more efficient. One of the simplest ways to do this is to ensure both the relationship type and directionality are defined in the Cypher query.

```cypher
//Non-directional and undefined
MATCH(p:Publication)-[]-(p:FieldOfResearch)

//Directional and defined
MATCH(p:Publication)-[l:LINKED]->(p:FieldOfResearch)
```

Unfortunately this does not help in our use case as all of the super nodes in our model have mono-directional, single type relationships with their related nodes. 

### Indexing Relationships
Neo4j does index relationships by default using a token lookup index that only contains type information for each index rather than information on which nodes they connect. However since version 4.3, Neo4j also supports indexing on relationship properties, this can be used to reduce the number of relationships that have to be checked during query execution.

For example the relationship between a Field of Research and a Publication node might have a publication_year property,  which could be used as a filter to reduce the number of relationships required in a query by filtering to a specific year:

```cypher
MATCH(p:Publication)-[l:LINKED {year:2020}]->(p:FieldOfResearch)
```

This moderately improves performance of some queries where the use of this property is an appropriate filter. However in our data model there are no properties that could reasonably be used as filters, and these relationships are often queried in bulk reducing any performance advantage. 


