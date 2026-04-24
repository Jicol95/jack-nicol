+++
title = "Postgres Indexes You Should Know"
date = "2025-01-20"
description = "A practical guide to the Postgres index types that matter most for backend engineers."
tags = [
    "postgres",
    "databases",
    "backend",
]
+++

Most developers know about B-tree indexes but Postgres has several index types worth knowing. Here's a practical rundown.

## B-tree (the default)

Good for equality and range queries on orderable types. The right choice for most columns.

```sql
CREATE INDEX idx_users_email ON users (email);
```

Use a composite index when you frequently filter on multiple columns together — order matters. Put the most selective column first, and make sure your queries filter on a prefix of the index.

## Partial Indexes

Index only a subset of rows. Dramatically smaller and faster when you query a small subset of a large table.

```sql
CREATE INDEX idx_orders_pending ON orders (created_at)
WHERE status = 'pending';
```

If most of your queries are `WHERE status = 'pending'`, this is far more efficient than indexing the full table.

## GIN Indexes for Arrays and JSONB

B-tree can't efficiently search inside arrays or JSONB. GIN (Generalised Inverted Index) can.

```sql
CREATE INDEX idx_products_tags ON products USING GIN (tags);
```

Now `WHERE tags @> ARRAY['electronics']` will use the index.

For JSONB:

```sql
CREATE INDEX idx_events_payload ON events USING GIN (payload jsonb_path_ops);
```

## Index on Expressions

Index the result of an expression, not just a column. Useful when you frequently query on a transformed value.

```sql
CREATE INDEX idx_users_lower_email ON users (lower(email));
```

Now `WHERE lower(email) = 'jack@example.com'` uses the index.

## Covering Indexes

Add `INCLUDE` to store extra columns in the index leaf pages. The query can be satisfied entirely from the index without touching the heap.

```sql
CREATE INDEX idx_orders_user ON orders (user_id) INCLUDE (status, created_at);
```

Useful when you're selecting only a few columns alongside your filter.

## Practical Tips

- Use `EXPLAIN (ANALYZE, BUFFERS)` — not just `EXPLAIN` — to see whether an index is actually being used and how many buffer hits it causes
- `pg_stat_user_indexes` shows index usage statistics; drop indexes with zero scans
- Indexes cost write performance — don't index speculatively
- `CREATE INDEX CONCURRENTLY` to avoid locking in production
