---
author: "Yves Adler"
title: "Microservices Prerequisites"
description: "My personal microservice-readiness-checklist: Topics that have proven to be essential in the development and operation of distributed systems."
date: "2022-05-10"
image: images/posts/prerequisites.jpg
tags:
  - architecture
  - decisions
  - microservices
  - checklist
---

# Introduction

After I've briefly discussed the [advantages and disadvantages of Microservices]({{< relref "/posts/advantages-and-disadvantages-of-microservices.md" >}}) I want to discuss a couple of prerequisites that you should implement before you move to a distributed system.

This article will not discuss any of the following topics in detail. Below I have listed points that have proven to be essential in the development and operation of distributed systems in recent years.

Many points may be self-evident for software developers by now. Nevertheless, in conversations and discussions I repeatedly reach the point where I would like to use such a list as a foundation for further discussions.

Please consider the following points as food for thought.

# Farewell to the monolith

Teams need to change their mindset and approach to problems as you move from one service to many. You can't just use all your old concepts and patterns and deploy them seamlessly in a distributed system. Say goodbye to easy transaction handling and a single breakpoint to debug a problem. Embrace the possibility to try new technologies in a small and well-defined context.

#### Reflect your mindset, rules and guidelines

Not every rule you've learned, read about, or heard of should be applied to microservices in the same way you might have applied it to your monolith. In my experience, there are often discussions about how to deal with the [DRY principle](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) and shared libraries.

Don't be tempted to share business logic between multiple services. This is a sign that you have a problem with the bounded contexts of your services. Generally you will want to have a dedicated service instead of a shared library.
Personally, I'm not a fan of putting DTOs in libraries either. Often consumers of APIs don't need all fields and can save a lot of RAM (and CPU time) by using only the fields they really need.

The [Microservice chassis](https://microservices.io/patterns/microservice-chassis.html) is an exception to the shared library rule. All basic infrastructure topics should be handled within this chassis. These include: Security, Logging, Metrics, Tracing and Health Checks.

#### Invest in architectural knowledge

This paragraph could fill an entire article. Therefore, very briefly: Educate yourself on architecture topics. Much of the common literature for developers focuses primarily on the design of a single application. Now, application modules must operate as individual services. This changes the knowledge requirements of each individual developer.

Personally, I have experienced an immense increase in knowledge through lectures, articles, and books on distributed systems / microservices, functional programming, immutability, event sourcing, DDD, and messaging brokers. 

# New challenges

## Development

#### Workflows

Gone are the days when it was enough to start a database and the backend, maybe a separate frontend, to develop the next feature. Now you have at least one docker-compose file with some services. Can your laptop handle a complete system? Do you want to be able to develop offline? You really have to think about the development process to maintain high productivity.

Decide if you want to use the [monorepo](https://en.wikipedia.org/wiki/Monorepo) approach - it could save you a lot of trouble in terms of version and compatibility management of your services. Speaking of compatibility, consider using [semver](https://semver.org/) for everything.

You may also want to review your [git](https://git-scm.com/) [workflows](https://www.atlassian.com/git/tutorials/comparing-workflows). Any problem you have can be multiplied if you have separate Git repositories for your services.

Think about documentation from the beginning. You need to aggregate the [OpenAPI](https://www.openapis.org/) specifications provided by each service and make them available from a central point that ideally also includes written documentation, guidelines, and examples of how to use your API. I think even if you don't plan to develop a publicly available API, you should always keep the standards high enough to eventually open it up for external use.

#### Testing

When it comes to unit/integration testing, microservices are fun. The public interface is small(er) and can be tested better accordingly. Often, the entire application stack is leaner and integration tests run correspondingly faster.

Personally, I think for such small services it is more worthwhile to emphasize integration tests more than unit tests. Couple this with test containers (instead of in-memory databases) and you can test pretty quickly, pretty effectively. Of course, it is important that all outgoing calls are served by mock servers so that you have a controlled test environment.

As a rule of thumb: Make sure that call to the API of the service has at least a happy path integration test. This way you can quickly add regression tests should a bug appear.

System tests on the other hand become way more complicated: You have to be very cautious that the system is stable and functional before you run any kind of system tests (see [Environments](#environments)). And they are really expensive to develop and maintain. Currently, my preferred tool for most system testing is [cypress](https://www.cypress.io/) as a UI testing tool. If you want to do API testing there are lots of good options as well.

## Distributed systems - distributed failures

- Microservices are a bit like software libraries: you always have to think about the users. Changes to their public interfaces must only rarely happen and should be controlled (and versioned).

- You will have to think a lot about possible communication errors. Whether synchronous or asynchronous, it will happen, and more often than you'd like.

- Every service should be polite to its (upstream) dependencies. One should at least know about the [Circuit Breaker](https://learn.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker) pattern. I personally consider a reasonably configured (or implemented) [Exponential backoff](https://en.wikipedia.org/wiki/Exponential_backoff) in clients as a basic precaution against cascading failures.

- Repeat failed operations only if they are actually repeatable. I've seen many cases where retries corrupted data or sent a lot of emails. This has to be evaluated for each and every operation.

- Be prepared that systems will not be available and learn about [graceful degradation](https://developer.mozilla.org/en-US/docs/Glossary/Graceful_degradation).

- Upstream services may return unexpected, erroneous or inconsistent data in unexpected data formats (e.g. JSON API with HTML error pages). Learn about the [anti-corruption layer](https://learn.microsoft.com/en-us/azure/architecture/patterns/anti-corruption-layer) pattern and consider which error classes must lead to cascading errors (in your service and its clients) and which can be handled.

- You should know enough about [two-phase commits](https://en.wikipedia.org/wiki/Two-phase_commit_protocol), [distributed transactions](https://en.wikipedia.org/wiki/Distributed_transaction), and [eventual consistency](https://en.wikipedia.org/wiki/Eventual_consistency) to make an informed decision about when and how to deploy these solutions. You also need to know about the advantages and disadvantages of the different methods. Unfortunately, I have seen more than once how developers have blamed message brokers for supposedly corrupt or missing data, but it was in fact a matter of simple implementation mistakes or the lack of knowledge regarding transactional behavior.

## Debugging and monitoring

- Please, get your logs in order.  You don't want to spread the mess you've made in your monolith. I have already written a detailed article about [logging]({{< relref "/posts/logging.md" >}}).

- Use a distinct user-agent for each service and add a similar header (or equivalent identifier) to your message for asynchronous communication.

- Access logs are useful on every layer. Period. If you ever have to debug complex routing issues you will be glad if you have them on CDN, proxy (most likely multiple) and application level.

- In extreme scenarios you might find yourself with 4 breakpoints in 3 different services to debug one issue. Make sure your (staging/test) system allows this.

- Same applies to service, database and infrastructure metrics. Each (public) API call should be measured, at least. Nowadays, the tools (usually [Grafana](https://grafana.com/) on top of [Prometheus](https://prometheus.io/)) are excellent, so this should be a piece of cake. Start with the critical paths and add new metrics as needed. For example, if performance issues went undetected or a metric is suitable for alerting. 

- An externally hosted uptime monitoring solution is also a good practice.

- Sooner rather than later you want to introduce [distributed tracing](https://microservices.io/patterns/observability/distributed-tracing.html). Try to establish it beginning at the CDN edge node down to the (asynchronous)  messages you sent internally. Later you can also add application performance management (APM) tools to the mix.

## Operations 

### Environments

You should be able to easily reproduce errors in your development environment. Additionally, you should have at least one staging environment that replicates the production environment as close as possible. These environments should be reverted at any time after you have manipulated their state during testing or debugging.

[Ephemeral Environments](https://ephemeralenvironments.io/) seems to be the gold standard these days. I think we'll be talking a lot more about Environment-as-a-Service solutions in the future. As a baseline, you should be able to create an environment for each development team. Preferably for each feature. Also, you need to consider whether you want to be able to develop offline or if you want to use the same mechanisms for the development environment.

### Servers, Pods and Containers

To run your services, you want [cattle, not pets](https://devops.stackexchange.com/questions/653/what-is-the-definition-of-cattle-not-pets). You absolutely must avoid [snowflake servers](https://martinfowler.com/bliki/SnowflakeServer.html) and aim for a automatic and reproducible creation/setup of hardware and virtual machines. [Infrastructure as code](https://en.wikipedia.org/wiki/Infrastructure_as_code) and clearly documented or automatic processes help enormously to achieve and maintain this state.

[Configuration management](https://www.atlassian.com/microservices/microservices-architecture/configuration-management) is no longer limited to providing a different version of a configuration file for each environment. You need to establish templating for configuration files, think about managing secrets, and make all these solutions as developer-friendly and easy to understand as possible. There [are](https://etcd.io/) [many](https://www.hashicorp.com/products/vault) [tools](https://www.puppet.com/) in this area, but be aware that each tool increases the indirection your configuration value must take to get to the destination. 

It can be difficult to keep track of services, their versions, deployments and changes. Tools like [backstage](https://backstage.io/) or [consul](https://www.consul.io/) can help where the UI of your container leaves information gaps.

Oh, the much emphasized *autoscaling* only makes sense if you don't operate on your own hardware - otherwise you have the usual problem of resource planning based on the peak load. Consider this a cost-saving feature for running in public clouds.

### CI/CD Tooling

Surely you already have fully automated build pipelines (CI/CD). These have to stay manageable for a tenfold increase in services and deployments. In case you went with a monorepo, can they should handle creating multiple artifacts from a single pipeline. 

Make sure you enough space available for your maven, npm or docker repositories. And do not forget to set up a security scanner for your dependencies and containers. 

Be it a [Gitlab](https://docs.gitlab.com/ee/ci/) pipeline, a [Rundeck](https://www.rundeck.com/) job or a command in your [ChatOps](https://www.pagerduty.com/blog/what-is-chatops/) channel - you need to decide on how you want to control your deployments and rollbacks (including your database). It's critical that you give teams the power and flexibility to actually live [DevSecOps](https://www.redhat.com/en/topics/devops/what-is-devsecops). The fact that you're likely to get an audit trail is a nice extra. 

While deployments should generally become easier and more frequent, new challenges will arise. You definitely want to learn how to do [zero downtime deployments](https://www.ioconnectservices.com/insight/4-ways-to-get-zero-downtime-deployment-with-devops) because downtimes tend to cascade through a system. It is a good idea to think about orchestrated releases and different approaches to maintain backward compatibility.

### Databases and BI/Reporting

It must also be possible to set up new databases quickly and easily, because services should use a separate database if they need one. This also includes their backups, developer dumps and (user) account management. You will need tooling for audited data injection into production systems as well as handling for *'Right to erasure'* requests

Another challenge is certainly the management of the inevitable variety of versions of the database systems. Encourage the developers to always work on the version preferred by the company (e.g. the latest LTS).

Unfortunately, managing (test) data is getting more complicated: You need to create multiple automated and anonymized dumps of each production service. Depending on the amount and complexity of the data, you could also generate test data or use static seeds.

The same goes for any kind of reporting or data warehousing. You will have to deal with many different data sources, most likely using different technologies. One solution could be to define a specific *reporting API* which every service has to implement. This way, direct database connections can be avoided and the business intelligence team can quickly add new (internal) services as data sources once they are implemented. 


# Further reading

- [Randy Shoup - Monoliths, Migrations, and Microservices](https://www.youtube.com/watch?v=gOZFmFNl1uk)
- [Martin Fowler - MicroservicePrerequisites](https://www.martinfowler.com/bliki/MicroservicePrerequisites.html)
- [Martin Fowler - Microservices Guide](https://www.martinfowler.com/microservices/)
- [microservices.io - various patterns and architectural styles](https://microservices.io/index.html)
- [Vivek Juneja - Microservices: Four Essential Checklists when Getting Started ](https://thenewstack.io/microservices-four-essential-checklists-getting-started/)
- [Pattern: Microservice chassis](https://microservices.io/patterns/microservice-chassis.html)
- [Manuel Kruisz - Microservice Bad Smells and Anti Patterns](https://www.manuelkruisz.com/blog/posts/microservice-bad-smells)
- [Matt Rickard - Reflections on 10,000 Hours of DevOps](https://matt-rickard.com/reflections-on-10-000-hours-of-devops)
- [Jason Barry - The benefits of on-demand staging environments](https://dev.to/featurepeek/the-benefits-of-on-demand-staging-environments-1fjj)
- [Arun Surendran - What are On-Demand Feature Environments?](https://medium.com/techbeatly/what-are-on-demand-feature-environments-982eaa96fdd5)
- [Philipp Hauer - Don't Share Libraries among Microservices](https://phauer.com/2016/dont-share-libraries-among-microservices/)
- [Yves Adler - Advantages and Disadvantages of Microservices]({{< relref "/posts/advantages-and-disadvantages-of-microservices.md" >}})
