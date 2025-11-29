---
title: 'Neo4j Super Node Performance Issues'
date: 2024-02-26
permalink: /posts/neo4j-super-node-performance-issues
tags:
  - data science
  - statistics
---
We recently developed a multi-billion relationship scale knowledge graph, representing the wider academic landscape. How we developed the data pipeline to build this graph in Neo4j is a story in itself, one I may write up here or on the [Wellcome Data blog](https://medium.com/wellcome-data). This post however focuses on a specific database modelling issue that has caused us severe query performance issues, the issue of low cardinality "super nodes".


