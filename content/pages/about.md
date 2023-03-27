---
title: "About"
url: "about"
image: images/about-header.jpg
menu:
  main:
    name: "About"
    weight: 4
---

# Hi, I'm Yves

I currently work as a team lead and full-stack developer for logistics solutions at [Pamyra](https://www.pamyra.de).

# Tech stack

## Greenfield, utopian product

What technologies would I use if I were to start a greenfield product today?

### Backend

- [Monolith first](https://martinfowler.com/bliki/MonolithFirst.html), then [Self-Contained Systems](https://en.wikipedia.org/wiki/Self-contained_system_(software)) as baseline architecture
- [Kotlin](https://kotlinlang.org/), my favorite JVM language
- [Dropwizard](https://www.dropwizard.io/en/latest/), [Spring Boot](https://spring.io/projects/spring-boot) or [Micronaut](https://micronaut.io/) as service / application framework
- [Jdbi](https://jdbi.org/) or [jOOQ](https://www.jooq.org/) as data access layer on top of JDBC
- [Liquibase](https://www.liquibase.org/) for database schema management
- [PostgreSQL](https://www.postgresql.org/) as database
- [Elasticsearch](https://www.elastic.co/elasticsearch/), if (fulltext) search is actually required
- [kafka](https://kafka.apache.org/) or [RabbitMQ](https://www.rabbitmq.com/) as message broker
- [Keycloak](https://www.keycloak.org/) for identity and access management
- [Testcontainers](https://www.testcontainers.org/) as essential
- [Ruby](https://www.ruby-lang.org/en/) for (one-time) scripts and tooling

### Frontend

- SSR as default, SPA if required
- [HUGO](https://gohugo.io/) or [Jekyll](https://jekyllrb.com/) for static pages ([Jamstack](https://jamstack.org/) approach looks promising)
- [TypeScript](https://www.typescriptlang.org/) plus [React](https://react.dev/) or [Svelte](https://svelte.dev) for (dynamic) applications
- SCSS, styled components as default
- [strapi](https://strapi.io/) as (headless) CMS
- [cypress](https://www.cypress.io/) for integration/system tests

### Tooling

- [Ubuntu](https://ubuntu.com/) or any other Linux which I installed myself on the laptop I've selected myself
- [IntelliJ IDEA](https://www.jetbrains.com/idea/business/)
- [gitlab](https://about.gitlab.com/) as code hosting and CI platform
- [Grafana](https://grafana.com/) and [Prometheus](https://prometheus.io/) for metrics and graphs
- [Airbyte](https://airbyte.com), [dbt](https://getdbt.com/) and [Metabase](https://www.metabase.com/) as a baseline business intelligence/reporting stack
- [OpenSearch](https://opensearch.org/) for logs
- [HAProxy](https://www.haproxy.org/) for load balancing and routing
- [Fastly](https://www.fastly.com/) or [Cloudflare](https://www.cloudflare.com/) as CDNs
- [git](https://git-scm.com/) as distributed version control with a [git-flow](https://nvie.com/posts/a-successful-git-branching-model/) or [GitHub Flow](https://githubflow.github.io/) approach


## Things I am productive with

We are not living in a perfect world. I would be absolutely fine if the stack includes one or more of the following technologies or frameworks. Depending on the problem, some of these might also be my preferred solution.

- [Java](https://openjdk.org/) (>= 17)
- [Spring JDBC (Template)](https://www.baeldung.com/spring-jdbc-jdbctemplate)
- [kubernetes](https://kubernetes.io/)
- [MariaDB](https://mariadb.org/) or [MySQL](https://www.mysql.com/)
- [MongoDB](https://www.mongodb.com/)


## Stuff, I'd like to avoid

Learned my lessons, every one of these technologies/frameworks actually deserve an article on why I would like to avoid it. 

- [Project Lombok](https://projectlombok.org/)
- [Hibernate ORM](https://hibernate.org/orm/documentation/6.1/), probably most of [JPA](https://www.oracle.com/technical-resources/articles/java/jpa.html)
- [Angular](https://angular.io/) 
- [Vaadin](https://vaadin.com/) or [GWT](https://www.gwtproject.org/)
- [Java](https://openjdk.org/) below version 17
- [Eclipse](https://www.eclipse.org/)
- [Microsoft Windows](https://www.microsoft.com/de-de/windows)
