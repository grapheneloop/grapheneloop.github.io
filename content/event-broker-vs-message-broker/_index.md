---
title: "Event Broker v/s Message Broker"
date: 2023-10-03T15:26:15Z
lastmod: 2018-12-08T15:26:15Z
publishdate: 2018-11-23T15:26:15Z
draft: true
weight: 1
description: event-broker-vs-message-broker
TableOfContents: true
---

# Introduction
## Streaming Aggregation
Broadly Messages can be one of 
* **Command**: Command produces a cause a State change
* **Event**: Event is a result of a state change
* **Query**: Request to obtain some information from a component, a safe operation that causes no side-effect
* **Reply** a response to a query.

## Gurantees of Event Broker
An Event has characteristics which are served by an event broker (not necessarily by a message broker - Event is a specialized form of Message) such as 
* Partitioning
* Strict Ordering
* Immutability
* Retention
* Replability
* Queryability

## Components/Roles in Event Driven Architecture
* Producer:
* Consumer:
* Intermediary:
* Mediator:

