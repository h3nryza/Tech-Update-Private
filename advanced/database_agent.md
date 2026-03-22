# Database Management Agent

> AI agent for schema design, migration generation, query optimization, and database operations.

---

## Table of Contents

- [Overview](#overview)
- [Agent Architecture](#agent-architecture)
- [Schema Design Skills](#schema-design-skills)
- [Migration Generation](#migration-generation)
- [Query Optimization](#query-optimization)
- [Database-Specific Patterns](#database-specific-patterns)
- [MCP Server for Database Operations](#mcp-server-for-database-operations)
- [CI/CD Integration](#cicd-integration)

---

## Overview

The Database Agent handles the full lifecycle of database management: designing schemas from requirements, generating migrations, optimizing queries, and providing ongoing operational guidance.

```mermaid
graph TB
    subgraph "Input"
        Req[Requirements / ERD]
        Code[Application Code]
        Logs[Query Logs]
        Schema[Existing Schema]
    end

    subgraph "Database Agent"
        SD[Schema Designer]
        MG[Migration Generator]
        QO[Query Optimizer]
        HM[Health Monitor]
    end

    subgraph "Output"
        DDL[DDL Statements]
        Mig[Migration Files]
        Idx[Index Suggestions]
        Report[Health Report]
    end

    Req --> SD --> DDL
    Schema --> MG --> Mig
    Code --> QO --> Idx
    Logs --> QO
    Schema --> HM --> Report
```

---

## Agent Architecture

| Component | Responsibility |
|-----------|----------------|
| **Schema Designer** | Translate requirements into normalized, indexed schemas |
| **Migration Generator** | Generate safe, reversible migration files from schema diffs |
| **Query Optimizer** | Analyze queries, suggest indexes, rewrite inefficient SQL |
| **Health Monitor** | Check for schema anti-patterns, unused indexes, bloat |

---

## Schema Design Skills

### Master Schema Skill

Create `.claude/skills/db-schema.md`:

```markdown
---
name: db-schema
description: Design database schemas from requirements, following normalization rules and best practices
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
---

# Database Schema Designer

## Process

### 1. Requirements Analysis

Read the requirements and extract:
- **Entities**: Nouns that need persistence (User, Order, Product)
- **Attributes**: Properties of each entity
- **Relationships**: How entities connect (1:1, 1:N, M:N)
- **Constraints**: Uniqueness, nullability, value ranges
- **Access patterns**: How data will be queried (critical for index design)

### 2. Schema Design Principles

- **Normalize to 3NF** by default, then selectively denormalize for performance
- **Use appropriate types**: Don't store UUIDs as VARCHAR(36) -- use UUID type
- **Always include**:
  - `id` primary key (UUID or BIGSERIAL depending on project conventions)
  - `created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()`
  - `updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()`
- **Soft deletes**: Add `deleted_at TIMESTAMPTZ` if the project uses soft deletes
- **Naming conventions**: snake_case for columns, plural for tables, `_id` suffix for FKs
- **Indexes**: Create indexes for all foreign keys and frequently queried columns

### 3. Output Format

Generate:
1. Entity Relationship Diagram (Mermaid)
2. SQL DDL with comments
3. Migration file(s) for the project's migration tool
4. Seed data script (if applicable)

### 4. ER Diagram Template

Use this Mermaid format:

```
erDiagram
    USERS ||--o{ ORDERS : "places"
    USERS {
        uuid id PK
        varchar email UK
        varchar name
        timestamptz created_at
    }
    ORDERS ||--|{ ORDER_ITEMS : "contains"
    ORDERS {
        uuid id PK
        uuid user_id FK
        varchar status
        decimal total
        timestamptz created_at
    }
```

### 5. Anti-Patterns to Avoid

- **Entity-Attribute-Value (EAV)**: Use JSONB column instead
- **Polymorphic associations**: Use separate tables or a discriminator column with proper FKs
- **Missing foreign keys**: Always create FK constraints (don't rely on application logic)
- **Overusing JSONB**: Structured, queryable data belongs in columns
- **VARCHAR without limit**: Always specify a reasonable max length
```

### Schema Review Skill

Create `.claude/skills/db-review.md`:

```markdown
---
name: db-review
description: Review existing database schema for anti-patterns, missing indexes, and optimization opportunities
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
---

# Database Schema Review

## Checklist

### Structure
- [ ] All tables have a primary key
- [ ] All foreign keys have corresponding indexes
- [ ] No column stores multiple values (1NF violation)
- [ ] No transitive dependencies (3NF violation)
- [ ] Appropriate column types (not oversized VARCHAR for everything)
- [ ] NOT NULL on columns that should never be null
- [ ] DEFAULT values where appropriate

### Naming
- [ ] Consistent naming convention (snake_case)
- [ ] Table names are plural
- [ ] Foreign key columns end with `_id`
- [ ] Index names follow pattern: `idx_{table}_{columns}`
- [ ] No reserved words used as column names

### Indexes
- [ ] All FK columns are indexed
- [ ] Composite indexes match query patterns (leftmost prefix rule)
- [ ] No duplicate indexes (index on (a) when index on (a, b) exists)
- [ ] Unique constraints where business rules require uniqueness
- [ ] Partial indexes for common filtered queries

### Performance
- [ ] No unbounded text columns that should be separate tables
- [ ] JSONB used only for truly unstructured data
- [ ] Appropriate use of enum types vs lookup tables
- [ ] Timestamps use TIMESTAMPTZ (not TIMESTAMP)
- [ ] Large tables have partitioning strategy documented

## Output

For each finding, provide:
- Location (table.column or migration file)
- Severity (Critical / High / Medium / Low)
- Issue description
- Suggested fix with SQL
```

---

## Migration Generation

### Migration Skill

Create `.claude/skills/db-migrate.md`:

```markdown
---
name: db-migrate
description: Generate safe, reversible database migration files
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
---

# Database Migration Generator

## Principles

1. **Every migration must be reversible** (include both up and down)
2. **Never drop data without explicit user confirmation**
3. **Avoid long-running locks**: Use `ALTER TABLE ... ADD COLUMN` (no default) + backfill
4. **One logical change per migration**: Don't combine unrelated schema changes
5. **Test both directions**: up, then down, then up again

## Safe Migration Patterns

### Adding a column

```sql
-- Up
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- Down
ALTER TABLE users DROP COLUMN phone;
```

### Adding a NOT NULL column (safe)

```sql
-- Up: Add nullable first, backfill, then add constraint
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
UPDATE users SET phone = '' WHERE phone IS NULL;
ALTER TABLE users ALTER COLUMN phone SET NOT NULL;
ALTER TABLE users ALTER COLUMN phone SET DEFAULT '';

-- Down
ALTER TABLE users ALTER COLUMN phone DROP NOT NULL;
ALTER TABLE users ALTER COLUMN phone DROP DEFAULT;
ALTER TABLE users DROP COLUMN phone;
```

### Renaming a column (zero-downtime)

```sql
-- Migration 1: Add new column
ALTER TABLE users ADD COLUMN full_name VARCHAR(255);
UPDATE users SET full_name = name;

-- Application code: Write to both columns, read from new
-- (deploy and verify)

-- Migration 2: Drop old column (after application is fully migrated)
ALTER TABLE users DROP COLUMN name;
```

### Adding an index concurrently

```sql
-- Up (PostgreSQL)
CREATE INDEX CONCURRENTLY idx_orders_user_id ON orders (user_id);

-- Down
DROP INDEX CONCURRENTLY idx_orders_user_id;
```

### Creating an enum type

```sql
-- Up
CREATE TYPE order_status AS ENUM ('pending', 'confirmed', 'shipped', 'delivered', 'cancelled');
ALTER TABLE orders ADD COLUMN status order_status NOT NULL DEFAULT 'pending';

-- Down
ALTER TABLE orders DROP COLUMN status;
DROP TYPE order_status;
```

## ORM-Specific Templates

### Prisma

```prisma
// schema.prisma changes
model Order {
  id        String      @id @default(uuid())
  userId    String      @map("user_id")
  status    OrderStatus @default(PENDING)
  total     Decimal     @db.Decimal(10, 2)
  createdAt DateTime    @default(now()) @map("created_at")
  updatedAt DateTime    @updatedAt @map("updated_at")
  user      User        @relation(fields: [userId], references: [id])
  items     OrderItem[]

  @@index([userId])
  @@index([status, createdAt])
  @@map("orders")
}
```

Generate migration:
```bash
npx prisma migrate dev --name add_order_status
```

### Knex.js

```javascript
exports.up = function(knex) {
  return knex.schema.alterTable('orders', (table) => {
    table.enu('status', ['pending', 'confirmed', 'shipped', 'delivered', 'cancelled'])
      .notNullable()
      .defaultTo('pending');
    table.index(['user_id', 'status']);
  });
};

exports.down = function(knex) {
  return knex.schema.alterTable('orders', (table) => {
    table.dropIndex(['user_id', 'status']);
    table.dropColumn('status');
  });
};
```

### Django

```python
from django.db import migrations, models

class Migration(migrations.Migration):
    dependencies = [
        ('orders', '0003_auto_20260322'),
    ]

    operations = [
        migrations.AddField(
            model_name='order',
            name='status',
            field=models.CharField(
                max_length=20,
                choices=[
                    ('pending', 'Pending'),
                    ('confirmed', 'Confirmed'),
                    ('shipped', 'Shipped'),
                    ('delivered', 'Delivered'),
                    ('cancelled', 'Cancelled'),
                ],
                default='pending',
            ),
        ),
        migrations.AddIndex(
            model_name='order',
            index=models.Index(fields=['user_id', 'status'], name='idx_order_user_status'),
        ),
    ]
```

### SQLAlchemy / Alembic

```python
"""Add order status column

Revision ID: abc123
"""
from alembic import op
import sqlalchemy as sa

def upgrade():
    op.add_column('orders', sa.Column(
        'status',
        sa.Enum('pending', 'confirmed', 'shipped', 'delivered', 'cancelled',
                name='order_status'),
        nullable=False,
        server_default='pending',
    ))
    op.create_index('idx_orders_user_status', 'orders', ['user_id', 'status'])

def downgrade():
    op.drop_index('idx_orders_user_status', 'orders')
    op.drop_column('orders', 'status')
    op.execute("DROP TYPE order_status")
```

## Dangerous Operation Warnings

The agent must ALWAYS flag these:
- `DROP TABLE` / `DROP COLUMN` -- data loss risk
- `ALTER COLUMN ... TYPE` -- may fail on existing data
- `TRUNCATE` -- irreversible data loss
- Removing `NOT NULL` -- may allow invalid data
- Changing enum values -- may break existing rows
```

---

## Query Optimization

### Query Optimization Skill

Create `.claude/skills/db-optimize.md`:

```markdown
---
name: db-optimize
description: Analyze and optimize database queries for performance
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
---

# Query Optimizer

## Analysis Process

### 1. Collect Queries

Find all database queries in the codebase:
```bash
# For raw SQL
rg -n "SELECT|INSERT|UPDATE|DELETE" --type sql
rg -n "\.query\(|\.execute\(|\.raw\(" src/

# For ORMs
rg -n "\.findMany|\.findAll|\.findOne|\.where\(|\.select\(" src/
rg -n "objects\.filter|objects\.get|objects\.all" src/  # Django
```

### 2. For Each Query, Check

**Index usage:**
```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) <query>;
```

Look for:
- `Seq Scan` on large tables (needs index)
- `Nested Loop` with large outer table (consider Hash Join)
- `Sort` without index (add index with matching order)
- High `rows removed by filter` (index not selective enough)

**Common rewrites:**

| Pattern | Issue | Fix |
|---------|-------|-----|
| `SELECT * FROM t WHERE col LIKE '%text%'` | Can't use B-tree index | Full-text search or trigram index |
| `SELECT * FROM t WHERE col IN (SELECT ...)` | Subquery may execute per row | Rewrite as JOIN |
| `SELECT DISTINCT a, b FROM t` | May sort entire table | Add unique index or use GROUP BY |
| `SELECT * FROM t ORDER BY col LIMIT 10` | Sorts all rows, takes 10 | Index on (col) |
| `SELECT COUNT(*) FROM large_table` | Full table scan | Approximate with `pg_stat_user_tables` |

### 3. Index Recommendations

For each suggested index:
- Expected improvement (from EXPLAIN comparison)
- Write overhead (indexes slow down INSERTs)
- Storage cost estimate
- Whether it's a covering index

### 4. Output

| Query Location | Current Plan | Issue | Suggested Fix | Expected Improvement |
|---------------|-------------|-------|---------------|---------------------|
```

---

## Database-Specific Patterns

### PostgreSQL

```markdown
## PostgreSQL Optimization Toolkit

### Useful queries for analysis

-- Table sizes
SELECT schemaname, tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as total_size,
  pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) as table_size,
  pg_size_pretty(pg_indexes_size(schemaname||'.'||tablename::regclass)) as index_size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Unused indexes
SELECT indexrelname, idx_scan, pg_size_pretty(pg_relation_size(indexrelid))
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY pg_relation_size(indexrelid) DESC;

-- Most expensive queries (requires pg_stat_statements)
SELECT query, calls, mean_exec_time, total_exec_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 20;

-- Lock contention
SELECT blocked.pid AS blocked_pid,
  blocked_activity.usename AS blocked_user,
  blocking.pid AS blocking_pid,
  blocking_activity.usename AS blocking_user,
  blocked_activity.query AS blocked_statement
FROM pg_catalog.pg_locks blocked
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked.pid = blocked_activity.pid
JOIN pg_catalog.pg_locks blocking ON blocking.locktype = blocked.locktype
  AND blocking.relation = blocked.relation
  AND blocking.pid != blocked.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking.pid = blocking_activity.pid
WHERE NOT blocked.granted;
```

### MySQL

```markdown
## MySQL Optimization Toolkit

-- Table sizes
SELECT table_name,
  ROUND(data_length/1024/1024, 2) AS data_mb,
  ROUND(index_length/1024/1024, 2) AS index_mb,
  table_rows
FROM information_schema.tables
WHERE table_schema = DATABASE()
ORDER BY data_length + index_length DESC;

-- Slow query analysis (requires slow_query_log)
-- SET GLOBAL slow_query_log = 'ON';
-- SET GLOBAL long_query_time = 1;

-- Index cardinality check
SELECT TABLE_NAME, INDEX_NAME, COLUMN_NAME, CARDINALITY
FROM INFORMATION_SCHEMA.STATISTICS
WHERE TABLE_SCHEMA = DATABASE()
ORDER BY TABLE_NAME, INDEX_NAME, SEQ_IN_INDEX;
```

---

## MCP Server for Database Operations

A specialized MCP server that gives Claude Code direct database access:

```python
#!/usr/bin/env python3
"""Database operations MCP server."""

from mcp.server.fastmcp import FastMCP
import os

mcp = FastMCP("db-agent")

def get_connection():
    import psycopg2
    return psycopg2.connect(os.environ["DATABASE_URL"])

@mcp.tool()
def explain_query(sql: str) -> str:
    """Run EXPLAIN ANALYZE on a query and return the execution plan.

    Args:
        sql: The SQL query to analyze (SELECT only)
    """
    if not sql.strip().upper().startswith("SELECT"):
        return "Error: Only SELECT queries can be explained."

    conn = get_connection()
    try:
        cur = conn.cursor()
        cur.execute(f"EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) {sql}")
        plan = "\n".join(row[0] for row in cur.fetchall())
        return plan
    finally:
        conn.close()

@mcp.tool()
def get_table_schema(table_name: str) -> str:
    """Get the full schema for a table including columns, types, indexes, and constraints.

    Args:
        table_name: Name of the table
    """
    conn = get_connection()
    try:
        cur = conn.cursor()

        # Columns
        cur.execute("""
            SELECT column_name, data_type, is_nullable, column_default
            FROM information_schema.columns
            WHERE table_name = %s AND table_schema = 'public'
            ORDER BY ordinal_position
        """, (table_name,))
        columns = cur.fetchall()

        # Indexes
        cur.execute("""
            SELECT indexname, indexdef
            FROM pg_indexes
            WHERE tablename = %s AND schemaname = 'public'
        """, (table_name,))
        indexes = cur.fetchall()

        # Foreign keys
        cur.execute("""
            SELECT tc.constraint_name, kcu.column_name,
                   ccu.table_name AS foreign_table, ccu.column_name AS foreign_column
            FROM information_schema.table_constraints tc
            JOIN information_schema.key_column_usage kcu ON tc.constraint_name = kcu.constraint_name
            JOIN information_schema.constraint_column_usage ccu ON ccu.constraint_name = tc.constraint_name
            WHERE tc.constraint_type = 'FOREIGN KEY' AND tc.table_name = %s
        """, (table_name,))
        fks = cur.fetchall()

        output = f"## Table: {table_name}\n\n"
        output += "### Columns\n| Column | Type | Nullable | Default |\n|--------|------|----------|--------|\n"
        for col in columns:
            output += f"| {col[0]} | {col[1]} | {col[2]} | {col[3] or ''} |\n"

        output += "\n### Indexes\n"
        for idx in indexes:
            output += f"- `{idx[0]}`: {idx[1]}\n"

        output += "\n### Foreign Keys\n"
        for fk in fks:
            output += f"- `{fk[1]}` -> `{fk[2]}.{fk[3]}` ({fk[0]})\n"

        return output
    finally:
        conn.close()

@mcp.tool()
def get_table_stats(table_name: str) -> str:
    """Get statistics for a table: row count, size, index usage.

    Args:
        table_name: Name of the table
    """
    conn = get_connection()
    try:
        cur = conn.cursor()
        cur.execute("""
            SELECT
                pg_size_pretty(pg_total_relation_size(%s::regclass)) as total_size,
                pg_size_pretty(pg_relation_size(%s::regclass)) as table_size,
                pg_size_pretty(pg_indexes_size(%s::regclass)) as index_size,
                (SELECT n_live_tup FROM pg_stat_user_tables WHERE relname = %s) as row_estimate,
                (SELECT last_vacuum FROM pg_stat_user_tables WHERE relname = %s) as last_vacuum,
                (SELECT last_analyze FROM pg_stat_user_tables WHERE relname = %s) as last_analyze
        """, (table_name,) * 6)
        row = cur.fetchone()
        return (
            f"## Stats for {table_name}\n\n"
            f"- Total size: {row[0]}\n"
            f"- Table size: {row[1]}\n"
            f"- Index size: {row[2]}\n"
            f"- Estimated rows: {row[3]}\n"
            f"- Last vacuum: {row[4]}\n"
            f"- Last analyze: {row[5]}\n"
        )
    finally:
        conn.close()

@mcp.tool()
def suggest_indexes(table_name: str) -> str:
    """Analyze a table and suggest missing or useful indexes based on query patterns.

    Args:
        table_name: Name of the table to analyze
    """
    conn = get_connection()
    try:
        cur = conn.cursor()

        # Get columns used in foreign keys that might lack indexes
        cur.execute("""
            SELECT kcu.column_name
            FROM information_schema.table_constraints tc
            JOIN information_schema.key_column_usage kcu
              ON tc.constraint_name = kcu.constraint_name
            WHERE tc.constraint_type = 'FOREIGN KEY'
              AND tc.table_name = %s
        """, (table_name,))
        fk_columns = [row[0] for row in cur.fetchall()]

        # Get existing indexed columns
        cur.execute("""
            SELECT a.attname
            FROM pg_index i
            JOIN pg_attribute a ON a.attrelid = i.indrelid AND a.attnum = ANY(i.indkey)
            WHERE i.indrelid = %s::regclass
        """, (table_name,))
        indexed_columns = {row[0] for row in cur.fetchall()}

        suggestions = []
        for col in fk_columns:
            if col not in indexed_columns:
                suggestions.append(
                    f"- `CREATE INDEX idx_{table_name}_{col} ON {table_name} ({col});`"
                    f" -- FK column missing index"
                )

        # Check for sequential scans from pg_stat_user_tables
        cur.execute("""
            SELECT seq_scan, idx_scan
            FROM pg_stat_user_tables
            WHERE relname = %s
        """, (table_name,))
        row = cur.fetchone()
        if row and row[0] > row[1] * 10:
            suggestions.append(
                f"- High sequential scan ratio ({row[0]} seq vs {row[1]} idx)"
                f" -- consider adding indexes for common query patterns"
            )

        return "\n".join(suggestions) if suggestions else "No obvious index improvements found."
    finally:
        conn.close()

if __name__ == "__main__":
    mcp.run()
```

Register in `.claude/settings.json`:

```json
{
  "mcpServers": {
    "db-agent": {
      "command": "python3",
      "args": ["./tools/db_mcp_server.py"],
      "env": {
        "DATABASE_URL": "postgresql://user:pass@localhost:5432/mydb"
      }
    }
  }
}
```

---

## CI/CD Integration

### Migration Safety Check

```yaml
name: Migration Safety Check

on:
  pull_request:
    paths: ['migrations/**', 'prisma/migrations/**']

jobs:
  check-migration:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Analyze migration safety
        run: |
          # Find new migration files
          MIGRATIONS=$(git diff --name-only origin/main...HEAD | grep -E 'migrations/')

          claude --print "
            Review these database migration files for safety:
            $MIGRATIONS

            Check for:
            1. Missing DOWN migration (irreversible changes)
            2. Dangerous operations (DROP TABLE, DROP COLUMN without backup)
            3. Long-running locks (ALTER TABLE on large tables)
            4. Data loss risk
            5. Missing indexes on new foreign keys
            6. Concurrent index creation (should use CONCURRENTLY)

            Rate each migration: SAFE / CAUTION / DANGEROUS
          " > migration-review.md
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

---

## Sources

- [AI Database Schema Designer & Migration Planner](https://www.mejba.me/agent-skills-marketplace/ai-database-schema-designer-migration-planner)
- [Database Schema Designer Skill](https://agent-skills.md/skills/OneWave-AI/claude-skills/database-schema-designer)
- [AI for SQL Performance 2026](https://www.syncfusion.com/blogs/post/ai-sql-query-optimization-2026)
- [Top Database Schema Migration Tools 2026](https://www.bytebase.com/blog/top-database-schema-change-tool-evolution/)
- [Modern Data Migration Framework for AI](https://www.datafold.com/blog/modern-data-migration-framework/)
