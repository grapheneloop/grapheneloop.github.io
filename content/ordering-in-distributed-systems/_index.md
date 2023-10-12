---
title: "Ordering"
date: 2023-10-08T15:26:15Z
lastmod: 2018-10-08T15:26:15Z
publishdate: 2018-10-08T15:26:15Z
draft: false
weight: 1
description: Ordering
TableOfContents: true
---

# Introduction
A function is an independent sequence of actions that can be reused and operates with a level of abstraction. It has the capability to accept input parameters and potentially provide an output. Each function consists of an execution process, a triggering mechanism, zero or more input parameters, and typically zero or one output result. When executed, it can either complete successfully or encounter errors and fail; with the option of including an exception handling mechanism. These attributes apply universally to any function or Function as a Service (FaaS) implementations.

# Some of the qualities of a function:
## Single Responsibility Principle
An effective function should perform a singular, well-defined task, which encourages the principle of modularity.
## Statelessness/No Side Effects
A well-constructed function should be devoid of state, meaning it does not alter global variables or induce unintended side effects. These functions encapsulate their behavior and maintain the integrity of the program's state, allowing for externalization of state, if needed.
## Testability
A function should be capable of being tested both in isolation and within the context of its specific sub-domain.
# Types of functions:
## Modularity and Reusability:
A function should be designed for reusability. However, when considering a Function as a Service (FaaS) in a Microservices ecosystem, each instance of the function should be associated with a specific Bounded Context within one of the subdomains. For instance, the same AWS Lambda function can serve multiple subdomains and may be triggered from various trigger points.
## Deterministic Functions
Deterministic function is one which returns same output for a given input. This makes it are more predictable and more easily testable. Deterministic functions however can cause side effects.
## Pure Functions
Similar to deterministic functions, but additionally **idempotent** - i.e.can be called any number of times without causing any side effects. e.g. of a side effect is Payment made on a purchase. 
## Stateful and Stateless Functions
It is possible to alter the internal state of a function, e.g. by means of changing a global variable from within the function itself. While this can be appropriate for some specific use cases, like tracking whether the function was invoked before or FaaS instance is a fresh instance from pool or cloned from another instance etc.., it can become problematic and difficult to debug in most usecases especially in multi-threaded runtime environments or in a Function as a Service (FaaS) context where state is expected to be shared across multiple instantiations.

# Problems/Solutions - Patterns/BestPractices:
Working with Functions as a Service (FaaS) offers various advantages, but it also introduces a range of challenges that are inherent to this approach. Here are some key considerations and strategies to effectively manage the issues associated with FaaS usage.
## Built to Scale
FaaS functions are almost always designed to scale by cloning [Y-Axis Scaling](https://akfpartners.com/growth-blog/scale-cube). So depending on the provider different strategies such as cloning of function instances may be adopted to quickly meet the scalability needs. Hence it becomes necessary to design function instances to not behave differently based on whether it's a newly created instance, an instance reused from a pool or even a cloned instance.
Depending on the environment this however may introduce new problems to solve e.g. order of processing, cold start.
### Problem of Ordering
#### Use of Event Broker
If ordering is important particularly with events one might need to resort to Channels such as an event broker (as against a Message Broker). Event Broker guarantees order at a shard or partition level; and since the number of consumers <= number of partitions(other words one consumer can read from multiple shards but the reverse isn't possible), this can give some level of guarantee. 
#### Use of FIFO Message Channel
FIFO Queues maintain order of event based on a message Group Id (similar to an Event Broker where a partition Id is used) and one when one message is acknowledged the next message in the same group is served. This is another strategy to guarantee ordering.

However the above can limit scalability particularly within the same Message Group or Partition. The solution is to use algorithms for [Event Ordering in Distributed Systems](/ordering-in-distributed-systems/) or park the events in a datastore and act upon an event when the dependant tasks are completed.
## Functions as Reusable and Modular set of operations
Functions are sometimes written highly with [Modularity and Reusability](#modularity-and-reusability) in mind. This would mean they will find a fitment across different subdomains and membership with multiple bounded contexts. This inturn means that the function would have to remain highly granular and sometimes when awareness of the usecases would have to be introduce into a function (:rofl:	- by means of an evil if conditions), things quickly get out of hand. 

If the function maintains the state externally (in an RDBMS for example), think in terms of datastore behind the function. The datastore must not be accessible outside any external components in any of the bounded context.
As much as possible keep a copy of the function code within the domain's own code repository. Another approach is to seperate out whats common and specific to use case this can be achieved with strategies such as - *wrapper function to encapsulate the common operations* or *make use of common libraries to share the capabilities*. 
## Pay Per Use:
The big advantage of using FaaS by popular cloud service providers is the Pay Per Use. We only pay for aspects such as number of invocations, Time and Resources such as reserved RAM and CPU allocated etc. This introduces a whole set of problems such as Cold start, abrupt termination due to function execution settings or Service provider's enforced time limits.
## Stateless v/s Stateful: 
Functions are designed to scale. As mentioned before providers may follow different strategies to achieve this. In FaaS environments, care should be taken when using mutable shared state to avoid concurrency issues, race conditions, and unintended side effects. While it's often clearer and safer to design functions to be stateless there are often need to maintain state like when state needs to shared across invocations. It's better to offload or manage state externally (such as a database or as provided by the Managed Service e.g. Azure Durable Functions).

A Function design may choose to externalize its state into an RDBMS, but now if the database operation has to make use of a connection pool it may need to use an maintained externally. Some Service Providers have solved this problem through inherent solutions e.g. RDS proxy in case of AWS Lambdas.
## Problem of Long running process:
FaaS Functions are generally 
## Cold Start v/s Warm Starts:
One of the problems with Pay-Per-Use is to an Function instance is made available (either by creating new or pulling one from pool of instances) only when there is an actual trigger. This is in contrary to a long running Service. E.g. a kubernetes pod may be running all the time actively waiting (or based on a scheduled task) for a request to come, it may designed to scale up based on thresholds set on CPU usage or RPS. The downside of the latter approach is it takes a minimum amount of compute (and thus cost) to keep the service running irrespective of if it is serving a request or not. By default it generally easier and quicker to scale (↑ & ↓) a FaaS function compared to the scaling a service based on thresholds.
This flexibility comes at the cost of Cold start as the default behaviour. Functions are generally slower at serving the first request or request after a period of inactivity. Function container must be started and code loaded, connection established, repartitioning due to consumer group etc. need to be done all may contribute to Cold Start.

Solution to this problem is Warm start. Warm function have everything ready to serve a request very much like a Service actively waiting to serve a request.

A hybrid approach is also possible with some FaaS frameworks where a function thats terminated is made available to serve subsequent request if it is within the expiry of the instance. This could however lead wastage of  limited resources (e.g. a DB Connection) are being reserved to these terminated instances even which they are not being actively used.
## Serverless:
Most FaaS providers provide Server Abstraction with their offering and like to identify it as the *Serverless*-ness managed service. While this takes away a lot of complexities it may impose restictions or leave us to choose from a set of defaults. Few examples of this:
* Choice of programming language. With some programming languages we may not be able to leverage all the resouces (e.g. multiple CPU cores).
* Memory/CPU selection - in a lot of cases this comes as a package. It's not hard to over allocate or encounter crash before learning that resources are under allocated.
* Function time out - FaaS functions generally have a timeout. The functions are killed beyond this timeout. It may require us to build control around any irrecoverable situations that may arise due to Atomicity or Consistency e.g. Corrupted file in an Elastic File System.
## Committing offsets in case of Event Brokers:
## Consumer group 
## Orchestration & Choreography
![AWS example](./images/FaaS%20Choreography%20&%20Orchestration.drawio.svg) 
### Consistency of rules
### Availablility
### Atomicity of operation