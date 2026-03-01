---
name: supabase-postgres-best-practices
description: Postgres performance optimization and best practices from Supabase. Use this skill when writing, reviewing, or optimizing Postgres queries, schema designs, or database configurations.
license: MIT
metadata:
  author: supabase
  version: "1.1.0"
---

# Supabase Postgres Best Practices

Comprehensive performance optimization guide for Postgres, maintained by Supabase. Contains rules across 8 categories, prioritized by impact.

## When to Apply

Reference these guidelines when:
- Writing SQL queries or designing schemas
- Implementing indexes or query optimization
- Reviewing database performance issues
- Configuring connection pooling or scaling
- Optimizing for Postgres-specific features
- Working with Row-Level Security (RLS)

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Query Performance | CRITICAL | `query-` |
| 2 | Connection Management | CRITICAL | `conn-` |
| 3 | Security & RLS | CRITICAL | `security-` |
| 4 | Schema Design | HIGH | `schema-` |
| 5 | Concurrency & Locking | MEDIUM-HIGH | `lock-` |
| 6 | Data Access Patterns | MEDIUM | `data-` |
| 7 | Monitoring & Diagnostics | LOW-MEDIUM | `monitor-` |
| 8 | Advanced Features | LOW | `advanced-` |

## Quick Reference

### 1. Query Performance (CRITICAL)

- **Use indexes on foreign keys**: Unindexed FKs cause sequential scans on JOINs
- **Avoid `SELECT *`**: Fetch only needed columns to reduce I/O
- **Use `EXPLAIN ANALYZE`**: Always check query plans before optimizing
- **Partial indexes**: Index only the rows you query (`WHERE deleted_at IS NULL`)
- **Avoid functions on indexed columns**: `WHERE lower(email) = ...` skips index; use expression indexes

```sql
-- Bad: function prevents index use
SELECT * FROM users WHERE lower(email) = 'user@example.com';

-- Good: expression index
CREATE INDEX idx_users_email_lower ON users (lower(email));
SELECT * FROM users WHERE lower(email) = 'user@example.com';
```

### 2. Connection Management (CRITICAL)

- **Use connection pooling**: Use PgBouncer (Supabase Transaction mode) for serverless
- **Avoid long-held connections**: Release connections promptly in serverless
- **Set statement timeout**: Prevent runaway queries

```sql
-- Set per-session timeout
SET statement_timeout = '30s';
```

### 3. Security & RLS (CRITICAL)

- **Enable RLS on all user-facing tables**
- **Use `auth.uid()`** for row-level ownership checks
- **Don't rely on client-side filtering** for security

```sql
-- Enable RLS
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

-- Policy: users can only see their own posts
CREATE POLICY "Users see own posts"
  ON posts FOR SELECT
  USING (user_id = auth.uid());
```

- **RLS performance**: Add an index on the column used in RLS policies

```sql
CREATE INDEX idx_posts_user_id ON posts (user_id);
```

### 4. Schema Design (HIGH)

- **Use appropriate types**: `uuid` for IDs, `timestamptz` (not `timestamp`) for times
- **Always use `timestamptz`**: Stores in UTC, displays in any timezone
- **Normalize judiciously**: Don't over-normalize; use JSONB for flexible fields

```sql
-- Good: timestamptz
CREATE TABLE events (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  created_at timestamptz NOT NULL DEFAULT now()
);
```

### 5. Concurrency & Locking (MEDIUM-HIGH)

- **Use `SELECT ... FOR UPDATE SKIP LOCKED`** for job queues
- **Avoid long transactions**: They hold locks and bloat WAL
- **Use `ON CONFLICT DO NOTHING/UPDATE`** instead of SELECT then INSERT

```sql
-- Upsert pattern
INSERT INTO user_settings (user_id, key, value)
VALUES ($1, $2, $3)
ON CONFLICT (user_id, key) DO UPDATE SET value = EXCLUDED.value;
```

### 6. Data Access Patterns (MEDIUM)

- **Batch inserts**: Use `INSERT INTO ... VALUES (...), (...)` over individual inserts
- **Use CTEs for complex queries**: Improves readability, Postgres may inline them
- **Paginate with keyset pagination** (not OFFSET for large datasets)

```sql
-- Keyset pagination (fast)
SELECT * FROM posts
WHERE created_at < $last_seen_created_at
ORDER BY created_at DESC
LIMIT 20;

-- OFFSET pagination (slow at large pages)
SELECT * FROM posts ORDER BY created_at DESC LIMIT 20 OFFSET 10000;
```

### 7. Monitoring & Diagnostics (LOW-MEDIUM)

```sql
-- Slow queries (requires pg_stat_statements)
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Missing indexes
SELECT schemaname, tablename, attname, n_distinct, correlation
FROM pg_stats
WHERE tablename = 'your_table';

-- Table sizes
SELECT relname, pg_size_pretty(pg_total_relation_size(oid))
FROM pg_class
WHERE relkind = 'r'
ORDER BY pg_total_relation_size(oid) DESC;
```

### 8. Advanced Features (LOW)

- **JSONB for flexible data**: Use `jsonb` (not `json`), index with GIN
- **Full-text search**: Use `tsvector`/`tsquery` with GIN index
- **Generated columns**: Pre-compute derived values

```sql
-- JSONB with GIN index
CREATE INDEX idx_metadata ON documents USING GIN (metadata);
SELECT * FROM documents WHERE metadata @> '{"type": "invoice"}';

-- Full-text search
CREATE INDEX idx_fts ON articles USING GIN (to_tsvector('english', content));
SELECT * FROM articles WHERE to_tsvector('english', content) @@ plainto_tsquery('search term');
```

## Supabase-Specific Notes

- **Use `supabase_url` + anon key** for client; never expose service role key client-side
- **Realtime**: Enable per-table in Supabase dashboard; adds overhead
- **Storage**: Use Supabase Storage for files, not BYTEA columns
- **Edge Functions**: Use Postgres functions for DB-heavy logic to avoid round trips

## References

- https://www.postgresql.org/docs/current/
- https://supabase.com/docs
- https://supabase.com/docs/guides/database/overview
- https://supabase.com/docs/guides/auth/row-level-security
