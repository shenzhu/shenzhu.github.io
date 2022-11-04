---
title: "The Contract-Powered Data Platform"
tags:
  - 数据系统
  - 业界实践
  - Event System
---

这篇笔记来自于[Buz Blog](https://buz.dev/blog/the-contract-powered-data-platform)

# 1. Components of a Good Data Platform

## 1.1. Instrumentation and Integration

这条比较不言自明，首先必须有data，之后才能讨论它的quality

## 1.2. The pipeline

Pipelines通常是`batch`或者`streaming`，虽然这两种religion经常有争论，但一些相似的concept是共通的. Pipelines collect dat aand put it somewhere

Best pipelines通常有以下特点:
- Move data reliably
- Annotate payloads with metadata such as provenance, `collected_at` timestamps, fingerprints, etc.
- Generate stats to provde the operator with feedback
- Validate and bifurcate payloads, if you're lucky
- Know about and act on payload sensitivities - obfuscate, hash, tokenize, redact, redirect, etc
- Minimize moving pieces
- Don't spend all the CEO's 💸💸💸 so they can afford that house in the Bahamas

## 1.3. Storage and access

Storage/access systems range from a wee little Postgres database, to Snowflake, to a data lake filled with Parquet fish and the Loch Ness Trino monster

## 1.4. Date Discovery

As things scale, pipelines/databases/data models变得越来越繁杂，这时候discovery就非常重要

## 1.5. Observability, Monitoring, and Alerting

# 2. Design Goals of a Good Data Platform

## 2.1. Comply with rules

比如CPRA

## 2.2. Minimize bad data

有bad data本身已经很坏了，更糟的是不知道bad data存在

## 2.3. Maximize knowledge of what the system is doing

要有各种各样的monitoring/logs来了解系统的现状

## 2.4. Minimize friction for all parties involved

参与开发/使用/维护data platform的engineer可能来自各个方面，甚至是不太懂技术的data engineer，要尽量让这些人都有good exprience

Want to get buy-in? Minimize friction. Want to increase adoption? Automate others' toil. Want sustainable systems? Reduce cognitive load

# 3. The Contract-Powered Platform

Schemas are the nucleus of sustainable data platforms

在一开始就enfornce schema的选项通常并不被采纳并且被认为是unnecessary overhead，但它是一项重要的长期投资，而且之后再enfornce schema会比较麻烦

- Schemas empower the "producer" <-> "consumer" relationship
- Schemas are data discovery: Schematizing data upfront means data discovery and documentation writes itself
- Schemas power data validation in transit
- Schemas help stop bad instrumentation from being implemented in the first place
- Schemas improve code quality
- Schemas power automation: Destination tables can be automatically created and migrated as schemas evolve
- Schemas as observability: Calculating namespace-level statistics and splicing them into observability tools is the natural next step, 当每个payload都有对应的namespace/schema并且与相关的Datadog等metrics系统整合起来的时候，相关的metrics/table都可以变得self-serve
- Schemas power compliance-oriented requirements
- Schemas are the foundation of higher-order data models: It is pretty easy to turn a schema into a [dbt source](https://docs.getdbt.com/docs/build/sources) so analytics engineers can reliably build upon a well-defined, trustworthy foundation
- Schemas are the foundation of data products

# 4. The Contract-Powered Workflow

## 4.1. Draft, iterate on, and deploy a schema

Non-engineer stakeholders可以自己定义最初版本的schema，而不需要来自data platform team的人参与

## 4.2. Bring tracking libraries and systems up to parity

每当新的schema加入或者version更新的时候，automation kicks in and (at minimum):
- Builds and deploys new tracking SDK's for engineering teams
- Pushes schema metadata ∆ to data discovery tools
- Ensures infrastructure dependencies (Kafka topics, database tables, etc)
- Pushes the schema to the appropriate place for pipeline-level validation
- Creates dbt sources for the analytics engineers

## 4.3. Implement tracking

不论是frontend, backend还是infrastructure tool，都可以发送tracking data到系统里

## 4.4. Deploy

With contract-powered workflows the following prereqs are taken care of before instrumentation rolls out, not after:
- Implementers and stakeholders talk to each other using shared verbiage.
- Versioned, language-specific data structures are generated like all other code dependencies.
- Metadata is pushed to discovery tools.
- The pipeline is primed to accept incoming payloads and mark them as "good" or "bad".
- Observability tools are ready to go for instantaneous feedback in development and production.
- Downstream analytics/modeling entrypoints (like dbt sources) are in place and can be immediately used.

# 5. The Schema-Powered Future

Some other reading if you want to dive in:

- [Data Wrangling at Slack](https://slack.engineering/data-wrangling-at-slack/)
- [Jitney at AirBnb](https://www.slideshare.net/alexismidon/jitney-kafka-at-airbnb)
- [Pegasus at LinkedIn](https://engineering.linkedin.com/blog/2020/pegasus-data-language)
- [Dragon at Uber](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2020/03/Schema-Integration-at-Uber-Scale-US2TS-2020-1.pdf)
- [Building Scalable Real Time Event Processing with Kafka and Flink at Doordash](https://doordash.engineering/2022/08/02/building-scalable-real-time-event-processing-with-kafka-and-flink/)
- [Data Mesh at Netflix](https://netflixtechblog.com/data-mesh-a-data-movement-and-processing-platform-netflix-1288bcab2873)
- [Building a Real-time Buyer Signal Data Pipeline for Shopify Inbox](https://shopify.engineering/real-time-buyer-signal-data-pipeline-shopify-inbox)