---
layout: post
title: datomic "a better sql" high level summary
---

# {{ page.title }}

Notes from attending Stu Halloway's 3 hour training course on Datmoic.

Datomic is a functional database. Targetted at enterprise applications, meant to succeed SQL. a better relational database; or a re-implementation of a relational database taking modern hardware into account. (Oracle was relesed in 1978.) Datomic is designed for everyday CRUD application development.

Database is immutable (a persistent data structure). Clojure's philosophy (functional programming philosophy, really) applied to a database.

Datomic achieves full ACID (atomicity, consistency, isolation, durability) while also gaining horizontal read scalability (spread across many machines). SQL is also ACID but does not achieve horizontal read scalability, and also introduces the object/relational impedance mismatch.

## database is an immutable value

Datomic is literally implemented as a heavily lazy persistent hashmap (really a MultiMap as it can store sets). It is implemented in Clojure with a single atom (containing the map), and zero locks.

* a heavily lazy, persistent hashmap. lets you query the world at any point in time
* remove complexity of "the world has changed midway through my query". you want the query as of the current timestamp, your code doesn't need to account for change
* this implies writes are serialized (by the transactor)
* since data is immutable, writes are really fast. they are just inserts. CRUD becomes CR - you don't update or delete, you simply insert a new version or insert a retraction. the data existed in the past, it doesn't exist anymore. like git history. you can still run queries against a past database.
* this means audit trails are free. massive report jobs don't have to take the system offline because we can query against a constant value of the database without fear that the world will change half way through the report.

## transactor/storage/query separation

immutability means we don't have to have a central "place" where the most up to date data is where we have to do all the heavy lifting for queries. so we can seperate the "database system" into three systems which no longer need to be in the same machine. Thus Datomic is composed of three pieces: a transactor service, a storage service, and a query *library* ("peer library").

Consider a cluster of tomcat instances: each is a "peer". each peer gets a memcache of indexes, as well as a realtime stream of writes which are not-yet-indexed. This means *all the data you need to perform read queries is local in memcache*. This is very fast.

Since queries are performed in the same process as the application, against local data, we can run arbitrary code as part of our query. insert arbitrary logic using the same language as our application (Clojure, Java).

writes are serialized by the transactor service. Writes are sent out to each peer's realtime cache right away, and then added to the index every so often. This means Datomic is not suitable for massive write volumes (like tweets), use a custom NoSql solution with eventual consistency for that. Datomic is ACID compliant, which means all reads against any peer are always consistent. if the transactor service goes down it trivially fails over to a backup - and since it doesn't do reads, there is no warmup, its instantly available.

central point of failure is storage, but since it is decoupled from the implementation of the transactor and the query library, storage can be backed by anything. It is commonly backed by a dumb key/value store like amazon dynamo for high availability, or run off of ram/filesystem, or even use postgres as a key/value store.

## claims to solve object/relational impedance mismatch

* mechanical translation between nested maps (entities) and the storage format
* native cardinality/many allows storing sets without joining
* query results are sets of entities. An entity is more or less the same datastructure that our business logic uses - a hashmap - hence no ORM needed. the database internally uses those same interfaces, so you can provide your own in-memory "normal" hashmap that you could query against with datalog (like .NET's LINQ).

## relational query language

* trivial horizontal scaling means we can have more expensive query language.
* query language is datalog - lets you use logic programming for queries. think prolog. joins are not as complex.
* Entity results can be nested objects, which means you can navigate an entity graph instead of explicitly querying for sets of tuples. Just like an in-memory programming model.

### References

* http://www.datomic.com/rationale.html#DataModel
* http://docs.datomic.com/architecture.html
