# Microservices Notes

This [Martin Fowler post](https://martinfowler.com/articles/microservices.html) is a good summary of some key themes on the topic of microservices however I'll jot down a few thoughts on it.

>In short, the microservice architectural style 1 is an approach to developing a single application as a suite of small services, each running in its own process and communicating with lightweight mechanisms, often an HTTP resource API. These services are built around business capabilities and independently deployable by fully automated deployment machinery. There is a bare minimum of centralized management of these services, which may be written in different programming languages and use different data storage technologies. - Martin Fowler

## Modular design

Traditionally monolith applications all fit inside a single/multiple processes and all the internal services scale with each other. They cannot be done independently like microservices - a similar analogy would be normal programs and the containerization of programs.

Microservices usually communicate over a web protocol such as HTTP (e.g. RabitMQ), using tools that are simple and ubiquitous. This has a performance cost over inter-process communication and the communication can become noisy from a monitoring perspective.

This allows CI/CD pipelines to be independent from each other and can allow faster deployments for fixes/features. It also means there is no single-point of failure - it introduces complexities when diagnosing problems too.

## Domain focused

Traditionally, organisations have structure teams according to the technology layer e.g. UI, DBAs, which then flows into the monolith structure.

This promotes silos and prevents the fostering of cross-functional teams. Microservices operate with often full-stack teams - the cost is up-skilling individuals but does risk silos.

Eah microservice can be "product" focused instead of project-based, encouraging teams to own the service/product for its lifetime.

The modularity enables services/products to be created and likewise, destroyed. This is particularly useful for companies that have more transient products/experiments e.g. seasonality.

## Distributed data

Monolith applications which require strong consistency usually rely on a single database and transactions. This is a simple and resilient approach. With microservices, wrapping downstream calls to various services around a transaction can incur significant performance cost - this is usually a design smell.

Often, teams need to relax the consistency guarantees, however, this may not be acceptable in certain situations.

## Serverless

Serverless is largely offloading the infrastructure scaling and maintenance to a cloud service provider e.g. AWS. It was initially focused on being "cloud functions" - simple, small functions that run based on triggers/events. It has potential to shift the microservices discussion as we find ways to simplify the DevOps load on developers.

**Pros**:

* Very little infra and has auto scaling
* Devs only have to care about the code
* Well-connected with other AWS services

**Cons**:

* Little customisation, can't have "container" access or run multiple containers along side the service e.g. ECS you can for logging/kafka
* Limited scaling options, batching is one way of managing costs
* Suited for "simple" CRUD APIs or simple event handlers
  * In complex services and in-house/self-hosted services it can get more difficult to interface with
* 15 min execution limit
* Transient storage
