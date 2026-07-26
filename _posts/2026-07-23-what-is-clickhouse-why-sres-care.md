---
title: What Is ClickHouse and Why Should SREs Care?
date: 2026-07-23 18:00:00 +1000
categories: [Infrastructure, Databases]
tags: [clickhouse, sre, devops, databases, olap, observability]
description: "A practical introduction to ClickHouse from an SRE and DevOps perspective: what it is, where it fits, and what operators should watch."
image: /assets/img/posts/what-is-clickhouse-why-sres-care.svg
---

## Introduction

Most engineers first hear about ClickHouse as a fast analytics database.

That description is correct, but it is not enough for SREs and DevOps engineers. From an operations perspective, ClickHouse is not just "another database." It is a system that often sits behind dashboards, observability platforms, event analytics, fraud detection, product metrics, security logs, trading analytics, and customer-facing reporting.

In other words, when ClickHouse becomes slow or unavailable, people do not just lose a database. They lose visibility.

That is why SREs should care.

This post explains what ClickHouse is, where it fits, why it is different from traditional row-based databases, and what operational questions engineers should ask before running it in production.

---

## What Is ClickHouse?

**ClickHouse is an open-source, column-oriented SQL database designed for fast analytical queries at scale.**

It is built for OLAP workloads: Online Analytical Processing. These are workloads where users ask questions across large volumes of data:

- How many requests failed in the last hour?
- What is p95 latency by service and region?
- Which customers generated the most events today?
- How many trades matched this strategy condition?
- Which log patterns increased after the latest deployment?

These queries often scan millions or billions of rows, but only need a small subset of columns. That is exactly where a column-oriented database is strong.

A traditional row-oriented database stores all fields for a row together. That is excellent when the system needs to read or update one full record, such as a user profile or an order. ClickHouse stores data by column, which means an analytical query can read only the columns it needs.

For example, this query:

```sql
SELECT
  service,
  quantile(0.95)(duration_ms) AS p95_latency
FROM request_events
WHERE event_time >= now() - INTERVAL 1 HOUR
GROUP BY service
ORDER BY p95_latency DESC;
```

does not need every column in `request_events`. It mainly needs `service`, `duration_ms`, and `event_time`. A columnar engine can avoid reading unrelated fields such as request body, user agent, trace attributes, or deployment metadata.

That physical storage model is one reason ClickHouse can be extremely fast for analytical workloads.

---

## OLTP vs OLAP: The Operational Difference

SREs should avoid treating every database as if it has the same failure model.

An OLTP database, such as PostgreSQL or MySQL, is usually optimized for:

- Point lookups
- Short transactions
- Frequent row-level updates
- Strong transactional consistency
- Application state

An OLAP database like ClickHouse is usually optimized for:

- Large scans
- Aggregations
- Time-series and event data
- Append-heavy ingestion
- Analytical queries over wide tables

That distinction matters because operational pressure appears in different places.

In an OLTP system, SREs often worry about lock contention, transaction latency, connection pools, replication lag, and hot rows.

In ClickHouse, SREs worry about different signals:

- Query memory usage
- Query concurrency
- Disk usage and compression ratio
- Insert volume and batch size
- Number of active data parts
- Background merges
- Replication health
- Slow distributed queries
- Retention and partition design

The database is still "just SQL" from the user's point of view, but the operational model is different.

---

## Why ClickHouse Is Interesting for SREs

ClickHouse matters to SREs because it is often used in systems where speed, cost, and visibility are tightly connected.

### 1. It Can Power Operational Visibility

Logs, metrics, traces, events, and audit records are all analytical data. They are usually append-heavy, time-based, and queried by aggregation.

That makes ClickHouse a natural fit for observability-style workloads:

- Search logs by service, level, and time range
- Aggregate request latency by endpoint
- Count error patterns after a deployment
- Slice infrastructure events by host, region, or Kubernetes namespace
- Explore raw telemetry without pre-aggregating every possible dashboard

For SREs, this is important because debugging production systems depends on asking new questions quickly. If every new question requires a new precomputed metric, visibility becomes slow and brittle.

ClickHouse gives teams a way to keep high-cardinality, granular data queryable.

### 2. It Changes the Cost Model

Observability and analytics costs grow quickly. Logs and events are noisy, and the easy answer is often "store less data." But storing less data can make incidents harder to debug.

ClickHouse helps because columnar storage usually compresses analytical data well, especially when similar values sit together. Its query engine can also avoid reading unused columns.

The result is not magic. You still need good retention policies, table design, and capacity planning. But ClickHouse can make it realistic to keep large volumes of operational data online for longer than a traditional row store would allow.

### 3. It Supports Real-Time Analytics

SRE dashboards are not useful if they are always behind reality.

ClickHouse is designed for high-throughput inserts and fast analytical reads. That makes it useful for near real-time dashboards and alert investigation where engineers need to ask, "What is happening now?"

Examples:

- Deployment health dashboards
- API latency breakdowns
- Security event exploration
- Trading system event analysis
- Customer-facing usage analytics

For DevOps teams, this can reduce the gap between deployment, signal collection, and diagnosis.

### 4. It Is Deeply Inspectable with SQL

One of ClickHouse's strongest operational features is that it exposes a lot of internal state through `system` tables.

Instead of relying only on external dashboards, operators can query ClickHouse about ClickHouse.

For example, slow queries can be investigated through `system.query_log`:

```sql
SELECT
  event_time,
  query_duration_ms,
  read_rows,
  read_bytes,
  memory_usage,
  query
FROM system.query_log
WHERE type = 'QueryFinish'
  AND event_time >= now() - INTERVAL 1 HOUR
ORDER BY query_duration_ms DESC
LIMIT 10;
```

Table storage and part counts can be investigated through `system.parts`:

```sql
SELECT
  database,
  table,
  count() AS active_parts,
  sum(rows) AS rows,
  formatReadableSize(sum(bytes_on_disk)) AS size_on_disk
FROM system.parts
WHERE active
GROUP BY database, table
ORDER BY active_parts DESC;
```

That matters during incidents. When a ClickHouse cluster is unhealthy, the fastest path to understanding the problem is often a direct SQL query against the system tables.

---

## The Mental Model: Data Parts and Background Merges

ClickHouse performance is closely tied to how data is stored.

In the common `MergeTree` family of table engines, inserted data is written as immutable data parts. Over time, ClickHouse merges smaller parts into larger parts in the background. This keeps storage efficient and helps query performance.

For SREs, this creates a useful mental model:

```text
Inserts create parts.
Too many small parts create pressure.
Background merges reduce part count.
Slow or stuck merges become an operational signal.
```

This is one reason insert strategy matters. Sending tiny inserts too frequently can create many small parts. Many small parts can increase filesystem overhead, slow queries, and put pressure on background merges.

So a ClickHouse incident may not look like a typical database incident. The application might be "writing successfully," but the cluster may be quietly accumulating too many parts until query performance degrades.

This is exactly the kind of system behavior SREs need to understand before production traffic arrives.

---

## Where ClickHouse Fits Well

ClickHouse is a strong fit when the workload has these properties:

- Data is mostly append-only
- Queries scan many rows
- Queries read only some columns
- Aggregations are common
- Data is time-based or event-based
- Users need fast dashboards or ad-hoc analysis
- Storage cost matters

Good use cases include:

| Use Case | Why ClickHouse Fits |
| --- | --- |
| Observability data | Logs, traces, and events are high-volume and query-heavy |
| Product analytics | Users need fast slicing by time, customer, feature, and segment |
| Security analytics | Event data needs fast filtering and aggregation |
| Trading analytics | Large event streams need low-latency exploration |
| Billing and usage analytics | Append-heavy usage records are naturally analytical |
| Internal BI dashboards | Aggregations over large datasets are common |

ClickHouse is not automatically the right answer for every database problem. It is usually not the first choice for transactional application state, high-frequency row updates, or workloads dominated by single-record lookups.

The simple rule:

> Use ClickHouse when the question is analytical. Use an OLTP database when the question is transactional.

---

## What SREs Should Watch First

If you are new to operating ClickHouse, start with a short list of signals. Do not try to monitor everything on day one.

### Query Health

Watch:

- Query latency
- Read rows and read bytes
- Memory usage
- Query error rate
- Expensive repeated query patterns
- Query concurrency

Useful places to look:

- `system.query_log`
- `system.processes`
- `system.errors`

### Storage Health

Watch:

- Disk usage
- Compression ratio
- Active part count
- Parts per partition
- Largest tables
- Retention behavior

Useful places to look:

- `system.parts`
- `system.disks`
- `system.tables`

### Ingestion Health

Watch:

- Insert rate
- Insert latency
- Insert batch size
- Failed inserts
- Async insert failures if async inserts are used
- Part creation rate

Bad ingestion patterns are one of the easiest ways to create long-term operational pain.

### Background Work

Watch:

- Merge backlog
- Mutation progress
- Replication queues
- CPU and I/O pressure from background tasks

ClickHouse does a lot of important work outside the foreground query path. If background work falls behind, the system may continue serving traffic while accumulating risk.

---

## The SRE Questions Before Production

Before running ClickHouse seriously, I would ask these questions:

1. What is the main workload: observability, analytics, BI, customer-facing dashboards, or something else?
2. What is the expected daily ingest volume?
3. What are the main query patterns?
4. Which columns are used for filtering and grouping?
5. What retention period is required?
6. What is the acceptable query latency?
7. What happens when ClickHouse is unavailable?
8. Is the system single-node, replicated, or sharded?
9. What is the backup and restore strategy?
10. How will we detect too many parts, slow merges, disk pressure, and memory-heavy queries?

These questions are more useful than asking, "Can ClickHouse handle big data?"

ClickHouse can handle very large datasets, but only if the workload, schema, ingestion, and operations model are designed with the engine in mind.

---

## Common Mistakes

### Mistake 1: Treating ClickHouse Like PostgreSQL

ClickHouse speaks SQL, but it is not a drop-in replacement for a transactional database.

If you model tables as if every query is a point lookup and every update should be row-by-row, you will fight the engine.

### Mistake 2: Ignoring Table Design

Partition keys and `ORDER BY` choices affect query performance, storage layout, and operational behavior.

This is not just a data modeling concern. It is an SRE concern because bad table design becomes slow queries, high memory usage, excessive storage, and difficult incidents.

### Mistake 3: Sending Too Many Tiny Inserts

ClickHouse likes efficient ingest patterns. Tiny inserts can create too many small parts, which increases merge pressure.

For production systems, ingestion design should be discussed early: batching, buffering, retries, backpressure, and deduplication all matter.

### Mistake 4: Monitoring Only Host Metrics

CPU, memory, and disk are necessary, but not enough.

A ClickHouse dashboard should also include database-native signals from system tables: query latency, read volume, part counts, merge activity, replication queues, and query errors.

### Mistake 5: Skipping Restore Testing

Backups are not a strategy unless restores are tested.

This matters even more for analytical systems because the data volume can be large, the restore time can be surprising, and some teams only discover the real recovery time during an incident.

---

## Why This Series Starts Here

ClickHouse is exciting because it makes large-scale analytics feel simple. You can load data, run SQL, and get fast answers.

But production operations are where the details matter:

- Why is this query reading so many rows?
- Why did memory usage spike?
- Why are there thousands of parts?
- Why are merges falling behind?
- Why is replication delayed?
- Why did dashboard latency jump after a schema change?

Those are SRE questions.

This series will approach ClickHouse from that angle: not just how to query it, but how to understand, operate, monitor, scale, and recover it.

---

## Key Takeaways

- ClickHouse is a column-oriented OLAP database designed for fast analytical queries.
- It is well suited to append-heavy, event-based, time-series, and observability-style workloads.
- SREs should care because ClickHouse often powers critical visibility and operational analytics.
- Operating ClickHouse requires a different mental model from operating OLTP databases.
- The first things to learn are columnar storage, `MergeTree`, parts, merges, query logs, system tables, and capacity signals.
- Fast queries are not an accident. They come from good schema design, good ingestion patterns, and active operational visibility.

---

## Further Reading

- [ClickHouse: Real-Time Data Analytics Platform](https://clickhouse.com/clickhouse)
- [ClickHouse Docs](https://clickhouse.com/docs)
- [When should you use a columnar database?](https://clickhouse.com/resources/engineering/when-to-use-columnar-database)
- [The definitive guide to ClickHouse query optimization](https://clickhouse.com/resources/engineering/clickhouse-query-optimisation-definitive-guide)
- [Top 10 best practices tips for ClickHouse](https://clickhouse.com/blog/10-best-practice-tips)

