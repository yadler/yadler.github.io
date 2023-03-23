---
author: "Yves Adler"
title: "Logging"
description: "Logging is a key ingredient to observability. Think about your logs as a way for your software to tell you a story. In this post, I explain my personal guidelines and practices related to application/service logs."
date: "2022-01-15"
image: images/posts/logging.jpg
tags:
  - logs
  - observability
---

# Introduction

Logging is a key ingredient to [observability](https://en.wikipedia.org/wiki/Observability_(software)). 
Think about your logs as a way for your software to tell you a story. You might only read this story in times of trouble, 
therefore it has to be short, easy to understand and free of ambiguities. Unfortunately, 
many times we are let down by software logs (especially in the Java world).

Logs are used for retrospective analysis. Usually, you dive into your logs when you encounter a problem. 
They can also be used for monitoring, alerting and certain analytics. Access logs can, for example, 
be parsed to retrieve certain information about your visitors which can be used for server-side analytics.

# Good, clean logs

The quality of the logs is critical for a good [developer experience](https://en.wikipedia.org/wiki/User_experience#Developer_experience). I try to stick to a couple of simple rules when it comes to logging:

- Logs must be understandable!
- Only log what is important. Most companies drown in logs. Some of the larger Elasticsearch indexes that I have seen contained nothing but application logs
- Keep your logs in English (or your language of choice) do not localize your log messages
- The number of different log formats/patterns should be kept to an absolute minimum. For example, Java services should all use the same format for their application logs and (another) for access logs
- Access logs are useful on every layer. Period. If you ever have to debug complex routing issues you will be glad if you have them on CDN, proxy (most likely multiple) and application level

## Security

- Never log any secrets! Never! Not even in the case where you think you absolutely need to! 
- Be sure that your awesome framework does not log them via a convenient display of all environment variables during the startup of your software
- Do not log HTTP headers if you use them for session/security-related features
- Logs must not contain any personal/sensitive information. If this is necessary you should treat the logs accordingly (access rights, data privacy, ...)

## Logging levels

It is important that your team, and you develop a common understanding of log levels. i use log levels roughly as follows. The environment defines where a log level is enabled by default. When troubleshooting all levels can be enabled (piece by piece) in the production system as well.

| Level     | Description                                                                                                                                                                                                               | Environment         |
|-----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------|
| **ERROR** | Should be critical and indicate that someone has to look into the problem, your users will experience this as a problem in some way                                                                                       | *all*               |
| **WARN**  | Indicates corrupt data or problematic state. This problem could be handled by the software (anti-corruption layer). Sources of the problem are likely outside of the pure code path (bad input, database inconsistencies) | *all*               |
| **INFO**  | Events which might give following log lines more context (user registered, order created). They should not mirror access logs!                                                                                            | *all*               |
| **DEBUG** | Should be used sparsely. Good examples would be SSL certificate issues, connection problem details which are likely leading to a WARN or ERROR                                                                            | *dev*, *qa*         |
| **TRACE** | Personally, I never use this level. Heavy usage usually indicates bad developer tooling. One might argue to use this for performance measurements                                                                        | *dev (when needed)* |

I've had good experience with some sort of marker for new/unhandled exceptions/errors that were not handled properly. Most of the time these should be the only ones logged with a stack trace (you could use the presence of a stack trace as a marker)

# Processes and Tooling

- Logs should be automatically collected, parsed and enriched (server, environment, version). This is usually done with tools like [fluentd](https://www.fluentd.org/), [filebeat](https://www.elastic.co/de/beats/filebeat) or [logstash](https://www.elastic.co/de/logstash/)
- Logs should be accessible via a dedicated frontend (kibana or similar) for every developer. The default seems to be [kibana](https://www.elastic.co/de/kibana/) or its fork [OpenSearch](https://opensearch.org/)
- Dedicated log/error analysis tools like [graylog](https://www.graylog.org/), [Sentry](https://sentry.io/) or [Datadog](https://www.datadoghq.com/) might be considered - especially for their abilities to cluster errors
- If you are using distributed tracing you should always include the trace, parent and span id in each log line. While you can use kibana for tracing filters a dedicated frontend like [Jaeger](https://www.jaegertracing.io/) for your tracing data is a good idea

# Maintenance

Typically, services and their logs (in production) are in an improvable state. A good approach is to review the logs regularly. I tend to go through the logs every few days while I have my first coffee, others may do this weekly or monthly. Whatever your schedule, you need to set aside some time to get the logs into a consistent and useful state.

## Workflow 

### Prioritize

I would suggest reviewing the log lines roughly in this order on a regular schedule:

- Top 2-3 sources/emitters of log lines. Just count them, no matter which level they are
- All log lines with stack traces. The goal would be to reduce the amount of stack traces in your logs to a absolute minimum. If you handled the problem correctly you won't need the trace anymore!
- The check for (undetected) operational problems, review all **ERROR**s
- To check the consistency and general state of your software, reviw all **WARN**ings

This may be very exhausting at first. After a while, however, the quality of the logs increases rapidly and the scope of this task becomes very manageable.

### Improve
For each log line ask the following questions:

- Is the log line required?
- Is the logging level correct?
- Improve the log message if you answer any of the following questions with no
    - Is the message helpful?
    - Is the message self-explanatory?
    - Do you have all information you need (no need to open your IDE or query a database)?
    - Will others understand the problem just by reading the log line?
- Does the message contain any personal/sensitive information?
- Do you need a stack trace?
    - The default should be 'no' for known and handled exceptions/errors
    - Do you have to log the reason if you throw an exception? The default should be to log on catch, otherwise, you will most likely log the information twice (or more)

## Example

An example is worth a thousand words. The following log is logged as error with a stack trace. Also, the actual message is extremely generic.

```
[ERROR] Couldn't find token. Linked entity will be removed.

com.example.service.exceptions.NotFoundException: Token with refId '1234' and type 'LINKED_ENTITY' not found.
    at com.example.token.service.TokenService.lambda$getTokenByRefId$0(TokenService.java:123) ~[example.jar:1.0.0]
    at java.util.Optional.orElseThrow(Optional.java:290) ~[na:1.8.0_212]
    at com.example.token.service.TokenService.getTokenByRefId(TokenService.java:123) ~[example.jar:1.0.0]
    [...]
```

Looking at the code, it quickly becomes apparent that this is part of a cleanup procedure and the case is actually expected. Accordingly, one could formulate this log line as follows (without the stack trace):

```
[INFO] Removed product '1234' during scheduled task 'remove-orphaned-products' because it is not linked to any shop
```

The new log line reflects the fact that it is not a problem to remove a product if it has not been assigned to a store for a long time. In addition, all the information that helps to understand the event is displayed directly in the log.


# Further reading

- [Gunnar Morling: What's in a Good Error Message?](https://www.morling.dev/blog/whats-in-a-good-error-message/)
- [OWASP: Improper Error Handling](https://owasp.org/www-community/Improper_Error_Handling)
- [Martin Fowler: Domain-Oriented Observability](https://martinfowler.com/articles/domain-oriented-observability.html)
- [Wix UX - When life gives you lemons, write better error messages](https://wix-ux.com/when-life-gives-you-lemons-write-better-error-messages-46c5223e1a2f)
- [Filebeat vs Logstash - The Evolution of a Log Shipper](https://logz.io/blog/filebeat-vs-logstash/)