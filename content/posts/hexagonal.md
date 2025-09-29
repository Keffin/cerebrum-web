---
title: "Hexagonal Architecture"
date: "2024-05-22"
summary: "My take on hexagonal architecture."
description: "A general intro to hexagonal."
toc: false
autonumber: false
math: true
tags: ["architecture", "java", "hexagonal"]
showTags: true
hideBackToTop: true
hidePagination: true
---

Hexagonal architecture is a pattern that aims to keep applications as loosely coupled as possible.
While some might argue that it can be overkill for smaller projects, I’ve personally had good experiences when building larger projects with it in OOP languages.

My interpretation may differ slightly from other resources, but it generally follows the basic principles of hexagonal architecture.

Below is a folder structure diagram that illustrates how I typically organize applications with this design in mind:

```sh
.
├── adapter/
│   ├── mq/
│   │   ├── producers
│   │   └── consumers
│   ├── db/
│   ├── http/
│   ├── grpc/
│   └── api/
│       ├── controllers/
│       └── mappers/
├── domain/
│   ├── models/
│   ├── ports/
│   │   ├── dbPort
│   │   ├── httpPort
│   │   ├── grpcPort
│   │   └── producerPort
│   └── usecases/
└── infrastructure/
    ├── metrics/
    ├── cron/
    └── feature_flags/
```

### Adapter layer

The adapter layer consists of both incoming and outgoing logic:
* Incoming adapters include API controllers, MQ consumers, etc
* Outgoing adapters include database integrations, MQ producers, or HTTP clients for external APIs

The adapter layer typically implements ports defined in the domain layer. These ports are then used in the use cases.
Adapter modules also include any necessary configuration for initializing their dependencies, e.g, the database adapter includes connection settings such as database name, user, and password.

### Domain layer

Domain layer consists of 3 three main parts:
* Models - usually simple POJOs (or equivalent) that represent domain objects
* Ports - interfaces that are implemented by the adapter layer
* Use cases - the application's core business logic

Domain layer should always aim to be independent of external dependencies. Tools like [ArchUnit](https://www.archunit.org/) can help enforce this in Java projects, so you can setup tests to validate your Java applications architecture.

Use cases access external systems (e.g., databases, APIs, or message queues) exclusively through the ports defined in the domain layer.

### Infrastructure layer

The infrastructure layer is where I place cross-cutting or operational concerns that don’t directly belong to the business logic. Examples include cron jobs, metrics, or feature flags. These are essential for running the application but are not part of the core domain.
