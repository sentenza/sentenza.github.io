---
layout: post
title: "How to achieve distributed S.O.L.I.D.ness without being capped"
description: "Mixing pricinples and best practices to get better code"
category: Software engineering
---
{% include JB/setup %}

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
Given that, we want to achieve three core features:

1. **C**onsistency
    * a reed sees all previous completed writes.
2. **A**vailability
    * reads and writes always succeed.
3. **P**artition tolerance
    * Guaranteed properties are maintained even when network failures prevent some machines from communicating with others

The **CAP theorem** states that a system can satisfy at most two of these three features above.
<!--more-->

For a distributed system to be continously **available**, every request received by a non-failing node in the system __must__ result in a response, so every algorithm used by the service have to eventually terminate. Hence this fact would be observed in terms of uptime percentage and liveness property of the algorithm.

Therefore, you have to apply the *art of choice* among two different paths:

* Scalability over Consistency
* Consistency over Scalability

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
