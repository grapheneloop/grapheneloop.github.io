---
title: "Security <WIP>"
date: 2023-10-08T15:26:15Z
lastmod: 2018-10-08T15:26:15Z
publishdate: 2018-10-08T15:26:15Z
draft: false
weight: 1
description: Security
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

# Advantages of FaaS:
## Serverless:
Most FaaS are design with Server Abstraction in mind. Some providers even call it *Serverless*-ness.
## Design to Scale
FaaS functions are almost always designed to scale. So depending on the provider different strategies such as cloning of function instances may be adopted to quickly meet the scalability needs. Hence it becomes necessary to design function instances to not behave differently based on whether it's a newly created instance, an instance reused from a pool or even a cloned instance.

Depending on the environment this however may introduce new problems to solve e.g. order of processing, cold start.
# Problems/Solutions - Patterns/BestPractices:
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
## Committing offsets in case of Event Brokers:
## Consumer group 

