+++
title = "Getting Started with Kafka: A Backend Engineer's Perspective"
date = "2024-06-01"
description = "A practical introduction to Apache Kafka from the perspective of a backend engineer working with distributed systems."
tags = [
    "kafka",
    "distributed-systems",
    "backend",
]
+++

Apache Kafka is one of those tools that sounds intimidating at first but quickly becomes indispensable once you understand its core model. Here's how I think about it after working with it in production.

## The Core Model

Kafka is a distributed log. Producers write messages to topics, consumers read from them. The key insight is that messages aren't deleted when consumed — they persist for a configurable retention period. This means multiple consumers can independently read the same data, and you can replay events from any point in time.

## Topics and Partitions

A topic is split into partitions, which is how Kafka achieves parallelism. Each partition is an ordered, immutable sequence of records. Consumers in the same consumer group each own a subset of partitions — so more partitions means more parallelism, up to the number of consumers in the group.

```
Topic: payments
├── Partition 0: [msg1, msg4, msg7]
├── Partition 1: [msg2, msg5, msg8]
└── Partition 2: [msg3, msg6, msg9]
```

## Ordering Guarantees

Kafka guarantees ordering within a partition, not across partitions. If you need all events for a given entity (say, a user) to be processed in order, use the entity ID as the message key — Kafka will route all messages with the same key to the same partition.

## Consumer Groups

Consumer groups are how you scale consumption. Each group maintains its own offset per partition, so you can have multiple independent services consuming the same topic without interference.

## When to Use Kafka

Kafka shines when you need:
- **Event sourcing** — store a durable log of what happened
- **Decoupling services** — producers don't need to know who's consuming
- **Fan-out** — multiple consumers processing the same events independently
- **Replay** — reprocess historical events after deploying a bug fix

It's overkill for simple request/response patterns. A regular HTTP call or a database queue is often the right tool for those.

## Practical Tips

- Start with more partitions than you think you need — you can't reduce them later
- Monitor consumer lag; it's your primary signal that something is wrong
- Idempotent consumers are a must — Kafka delivers at least once, not exactly once by default
- Use `kafka-consumer-groups.sh --describe` liberally during debugging
