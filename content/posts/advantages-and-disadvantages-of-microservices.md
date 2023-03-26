---
author: "Yves Adler"
title: "Advantages and Disadvantages of Microservices"
description: "Often enough teams or whole IT departments seem to stumble into a half-baked microservice architecture. I want to highlight my personal pros and cons of a distributed system."
date: "2022-04-03"
image: images/posts/microservices.jpg
tags:
  - architecture
  - decisions
  - monolith
  - microservices
---

# Introduction

Since 2013, I have repeatedly encountered ["microservice architectures"](https://www.martinfowler.com/articles/microservices.html) of varying granularity and scope. Therefore, I usually talk about "distributed systems" instead of microservices. In most cases, the services are not really "micro" anyway. 

I also want to emphasize that nowadays between monoliths and microservices there are many valid and reasonable architectural approaches: From [Self-Contained-Systems](https://scs-architecture.org/) to [Serverless Architectures](https://aws.amazon.com/lambda/serverless-architectures-learn-more/). All of these approaches distribute code across multiple deployments and thus the following advantages and disadvantages apply.

At the beginning of many microservice architectures there is often the statement that the maintenance of monoliths is hard and so on and so forth. Developers want to rewrite everything all the time anyway. They will argue with insights from blogs and books. Managers and shareholders want to scale the business - so we also need a scalable architecture.

I won't go into how to justify an architectural migration here, but I would like to write down my personal microservice pro/con list. In the end, one often says goodbye to the monolith too early out of fear of splitting it too late.
Even though I always advocate ["monolith first"](https://martinfowler.com/bliki/MonolithFirst.html), it has been shown that distributed systems are more effective under the right conditions. Definitely you need a certain organizational size before the advantages outweigh the disadvantages.

# Advantages


#### Ownership

One of the main arguments for microservices is certainly the organizational structure of the company. This means that the services can be distributed across different teams relatively easily. If necessary, they can also be quickly moved between the teams, for example, to mitigate some effects of [Conways's law](https://en.wikipedia.org/wiki/Conway%27s_law). 

A basic rule is certainly that a service belongs to exactly one team and thus the ownership of the service is clearly defined. Furthermore, the skills of the various developers can be better utilized, because, for example, problem-specific programming language, libraries and tools can be used in individual services.

#### Developer experience

Furthermore, microservices are easier to understand than larger and more complex constructs. They can improve the developer experience drastically because the constraints are hugely reduced by working in smaller chunks of code. In the worst case a microservice can be completely rewritten if its scope changed or architecture proved wrong. In fact that is one of the advantages which I've seen most in practice.

A big advantage is undoubtedly the possibility of small, frequent releases (of individual functions). Also, different developers or teams no longer have to go through the pain of a [release train](https://www.thoughtworks.com/en-de/radar/techniques/release-train) together.

Large APIs/interfaces are broken down into manageable sub-aspects. With a little discipline, these parts can be tested more easily and better than in a monolith. Thus, each service has a manageable API which can (and should) be extensively tested.

#### Scaling and performance

The possibility to scale individual services (with high load) is an argument that is often cited and absolutely justified. However, the problem is almost as often non-existent. Scaling horizontally has many other hurdles - service size is just one of them. Besides, most of us aren't building the new Netflix or Spotify. Do you run your software on dedicated hardware (not in a cloud)? Then autoscaling is nonsense - the available resources are constant (and limited) in this case, no matter how many levels of virtualization are running on the hardware.

Scheduling or background processes can be moved to dedicated services. This means that no service needs an internal scheduler, which at least improves process transparency and scalability. In a cloud environment these services could be deployed/started on demand to save costs.

# Disadvantages

#### You need better developers

You need experience, knowledge and mindfulness to build robust distributed systems. Unfortunately, there is no shortcut - without enough experience in the team, you will make too many mistakes and possibly fail. It is definitely easier to develop and maintain an existing monolith with junior developers. Have you debugged something across multiple backends all with their individual breakpoints? It's no fun.

If you went the polyglott way then you also have a higher burden if people want to switch teams. But this may even apply if different frameworks are used in these teams. Same problems apply if services are assigned to different teams.

#### Performance and error handling

While you can now scale services independently of each other you most likely introduced a significant amount of inter-service communication. Each communication attempt can fail, be slow or hang indefinitely. Each communication attempt can fail, be slow or hang indefinitely. Latency adds up quickly. One slow DNS server somewhere in k8s and your system performance will decrease quickly. 

You need to be ready to handle these from the start. Ready to discuss HTTP client and server settings in detail? Ready to trace DNS hiccups? Ready for request tracing across multiple protocols and services? Because you will have to do all of that sooner rather than later.

#### You are not Netflix

Not everyone has the problems of Netflix, Spotify or Amazon. Don't make them yours accidentally. Maybe you do not need 500+ services, if you do not need to scale a global streaming business. Please, make your own decisions.

#### Diversity vs Compatibility 

Releases are generally much less problematic and faster but there are some new challenges. You have to make sure that you do not break any internal APIs (which you now have to treat as public because it is exposed to your network). There can be instances where you have to orchestrate the release of multiple services in order to release one feature. Most of the time this will actually cause multiple release for most of the involved services due to downward compatibility. Be sure to read about *Open Host Service* and *Anti-Corruption Layer* patterns.

The dependencies between services are not always straight forward and can be transitive, especially when events / messaging systems  are used. It is a rather challenging task to track and document all inter-service dependencies. *Mono-repositories* are certainly an interesting concept to deal with compatibility of different services.

You will also have to find a solution to have a consistent public API which spans multiple services. This is a management/governance topic: (REST) API guidelines need to be established. Likely you want to have an API Gateway of some sort (HAProxy being one of the simpler solutions here). And you want to agree on technologies for synchronous and asynchronous communication, be it REST, gRPC, webservices, kafka or something else. Otherwise, will have to deal with a zoo of different technologies.

#### Security

With each service, the attack surface and the effort to keep all dependencies up-to-date increases. Fortunately, there are now good dependency scanning approaches. But almost always a developer has to step into action. If a central library (e.g. logging) has a security vulnerability, then all services must be updated and released.

You also have to think about authentication and authorization. Fortunately, in times of [keycloak](https://www.keycloak.org/) you don't have to write everything yourself. API Gateways can be a tool to solve many problems, such as auth, rate limiting and access management in general. Nonetheless, there are decisions to be made that affect the entire system. 

#### Infrastructure

It is very unlikely that running many services is cheaper than running a monolith! From an overall economic point of view, microservices might be better, but operations and deployment becomes more complicated and more expensive.

In addition, there are many more operations and delivery engineering topics to cover:
- Hosting: AWS, GCP, Azure or self-hosted kubernetes?
- Infrastructure as code? Terraform? Helm? Puppet? Ansible?
- Management of configuration and secrets
- Build processes and Deployment (to different environments)
- Development environments (how to run these services locally)
- Package and container registries
- Service discovery
- Observability - from [Logging]({{< relref "/posts/logging.md" >}}) to monitoring, tracing and alerting

Some of these issues might also be relevant for Monolith, but are not difficult to solve.

# Further reading

- [Microservices Guide by Martin Fowler](https://www.martinfowler.com/microservices/)
- [Self-Contained Systems - Assembling Software from Independent Systems](https://scs-architecture.org/)
- [AWS Serverless Architectures](https://aws.amazon.com/lambda/serverless-architectures-learn-more/)
- [Shopify - Microservices: Advantages and Disadvantages](https://www.shopify.com/enterprise/disadvantages-microservices)
- [Google Cloud - Best Practices for Microservice Performance](https://cloud.google.com/appengine/docs/legacy/standard/java/microservice-performance)
- [DZone - What Problems Do Microservices Solve?](https://dzone.com/articles/what-problems-do-microservices-solve)
- [Containerum - 10 Tech Challenges That Are Solved by Microservices](https://medium.com/containerum/10-tech-challenges-that-are-solved-by-microservices-d91adeecb2e7)
- [RedHat Developers - The disadvantages vs. benefits of microservices](https://developers.redhat.com/articles/2022/01/25/disadvantages-microservices)
- [Stackoverflow: Authentication: JWT usage vs session](https://stackoverflow.com/a/45214431)