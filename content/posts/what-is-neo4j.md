---
title: "An introduction to Neo4j"
date: 2021-11-10T18:38:21+01:00
draft: false
math: true
tags: ["Neo4J", "Database", "Graph", "Graph-database"]
---

![](/neo4j.png)

## What is Neo4j

Neo4j is a graph database.
Now what is a graph database you ask? Instead of storing data in tables like in traditional relational databases like MySQL, graph databases store data as nodes in a graph. Each datapoint is a node, and relations between nodes are edges in the graph. In a relational database this would be accomplished by having references to other rows, so why should we bother having the data in a graph?

## Motivation

Even though it is simple to model relations in traditional databases more complex networks is a bit harder. Say you want to model a social graph and want to find your friends' friends. In a table you would first have to get all your friends, and then join that result with all their friends. In a graph database like Neo4j however, this can be done much easier since querying transitive relations is quit natural in query languages like cypher (which Neo4j uses).

```cypher
MATCH (:Person)-[:Friend*..2]->(p:Person)
RETURN p
```
