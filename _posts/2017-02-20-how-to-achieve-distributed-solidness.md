---
layout: post
title: "How to achieve distributed S.O.L.I.D.ness without being capped"
description: "Mixing pricinples and best practices to get better code"
category: Software engineering
tags: "Design Patterns"
---

### S.O.L.I.D. principles

| SOLID Initial |  Stands for  |          Concept         |
| :-----: |:-------------:| :---------- | :----------------------- |
|    S 	  | **S**ingle responsibility principle | 	A class should have only a single responsibility (i.e. only one potential change in the software's specification should be able to affect the specification of the class) |
|    O 	  |	**O**pen/closed principle | 	“software entities … should be open for extension, but closed for modification.” |
| L |  	**L**iskov substitution principle | 	“objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program.” |
| I | 	**I**nterface segregation principle | 	“many client-specific interfaces are better than one general-purpose interface.” |
| D |	**D**ependency inversion principle | 	one should “Depend upon Abstractions. Do not depend upon concretions.” |

### Distributed systems withstand CAP theorem

A **distributed system** essentially is a software system in which components located on interconnected systems communicate and coordinate their actions by passing messages.

We generally make the following two assumptions: we have **a system that stores data** within an **asynchronous network**.

<!--more-->

### CAP theorem

<p align="center">
  <img src="https://user-images.githubusercontent.com/2479647/135652337-f87fde57-1d42-4241-8783-9d66d5ef9c97.png">
  <br/>
  <i>[Source: CAP theorem revisited](http://robertgreiner.com/2014/08/cap-theorem-revisited)</i>
</p>

The **CAP theorem** states that a system can satisfy at most two of these three features above.

In a distributed computer system, you can only support two of the following guarantees:

* **Consistency** - Every read receives the most recent write or an error (it sees all the previous writes)
* **Availability** - Every request receives a response, without guarantee that it contains the most recent version of the information (reads and writes always succeed)
* **Partition Tolerance** - The system continues to operate despite arbitrary partitioning due to network failures and then guaranteed properties are maintained even when network failures prevent some machines from communicating with others

*Networks aren't reliable, so you'll need to support partition tolerance.  You'll need to make a software tradeoff between consistency and availability.*

For a distributed system to be continously **available**, every request received by a non-failing node in the system __must__ result in a response, so every algorithm used by the service have to eventually terminate. Hence this fact would be observed in terms of uptime percentage and liveness property of the algorithm.

Therefore, you have to apply the *art of choice* among two different paths:

* **Scalability over Consistency**
* **Consistency over Scalability**

#### CP - consistency and partition tolerance

Waiting for a response from the partitioned node might result in a timeout error.  CP is a good choice if your business needs require atomic reads and writes.

#### AP - availability and partition tolerance

Responses return the most readily available version of the data available on any node, which might not be the latest.  Writes might take some time to propagate when the partition is resolved.

AP is a good choice if the business needs allow for [eventual consistency](#eventual-consistency) or when the system needs to continue working despite external errors.

### Source(s) and further reading

* [CAP theorem revisited](http://robertgreiner.com/2014/08/cap-theorem-revisited/)
* [A plain english introduction to CAP theorem](http://ksat.me/a-plain-english-introduction-to-cap-theorem)
* [CAP FAQ](https://github.com/henryr/cap-faq)
* [The CAP theorem](https://www.youtube.com/watch?v=k-Yaq8AHlFA)

## Consistency patterns

With multiple copies of the same data, we are faced with options on how to synchronize them so clients have a consistent view of the data.  Recall the definition of consistency from the [CAP theorem](#cap-theorem) - Every read receives the most recent write or an error.

### Weak consistency

After a write, reads may or may not see it.  A best effort approach is taken.

This approach is seen in systems such as memcached.  Weak consistency works well in real time use cases such as VoIP, video chat, and realtime multiplayer games.  For example, if you are on a phone call and lose reception for a few seconds, when you regain connection you do not hear what was spoken during connection loss.

### Eventual consistency

After a write, reads will eventually see it (typically within milliseconds).  Data is replicated asynchronously.

This approach is seen in systems such as DNS and email.  Eventual consistency works well in highly available systems.

### Strong consistency

After a write, reads will see it.  Data is replicated synchronously.

This approach is seen in file systems and RDBMSes.  Strong consistency works well in systems that need transactions.

## Transactional systems

One could also need to have a transactional system based on a distributed one, and here it comes **ACID**ity:

* **A**tomicity
* **C**onsistency
* **I**solation
* **D**urability

Keep always in mind that NoSQL is never ACID-compliant; it lacks of atomic operations across the documents and collections. On the contrary, SQL is normally ACID when using the right engine and a good relational scheme made of restrains and keys.

### Other really worth to know patterns

**State machine**:  a set of defined states, where transition between states is initiated by triggering an action.

**Workflow**: a sequence of activities, where transition between them occurs when the previous activity is completed. Workflows can include branching to other activities. A workflow can be built on top of a state machine. A process manager is a workflow pattern.

**Saga**: multiple workflows, each providing compensating actions for every step of the workflow where it can fail. Sagas are not necessarily implemented using workflows.


## Resources

* [Designing something SOLID](https://www.novoda.com/blog/designing-something-solid/)
