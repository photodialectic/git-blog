# MySQL

Database management in a containerized homestack requires careful consideration of resource usage, schema evolution, and operational simplicity. Rather than using a managed MySQL service, I run MySQL as another container with a focus on minimal memory usage and schema-as-code using Skeema.

## Why Self-Hosted MySQL?

For a HomeStack, self-hosted MySQL offers several advantages over managed services:

- **Cost control**: No monthly fees or per-query charges
- **Resource efficiency**: Tuned for low-memory environments
- **Full control**: Custom configurations and direct access
- **Data locality**: No external dependencies or network latency
- **Learning opportunity**: Better understanding of MySQL internals

The trade-off is operational responsibility, but for a home lab this becomes a feature rather than a bug.

## Memory-Optimized Configuration

My HomeStack runs on modest hardware, so MySQL is configured for minimal memory usage:

```ini
# low-memory-my.cnf
[mysqld]
# Core memory optimizations
innodb_buffer_pool_size=5M
innodb_log_buffer_size=256K
max_connections=10
key_buffer_size=8
thread_cache_size=0
host_cache_size=0

# Per-thread settings
thread_stack=256K
sort_buffer_size=32K
read_buffer_size=8200
read_rnd_buffer_size=8200
max_heap_table_size=16K
tmp_table_size=1K

# Disable performance schema
performance_schema = off
```

This configuration reduces MySQL's memory footprint from ~400MB to under 50MB while maintaining functionality for moderate workloads. The key optimizations:

- **Tiny buffer pool**: 5MB instead of the default 128MB
- **Limited connections**: 10 concurrent connections vs. default 151
- **Minimal caches**: Disabled or drastically reduced cache sizes
- **No performance schema**: Saves significant memory overhead

## Container Setup

MySQL runs as a standard Docker container with persistent storage:

```yaml
mysqldb:
  container_name: mysqldb
  environment:
    - MYSQL_ROOT_PASSWORD=${MYSQL_PWD}
  image: mysql:8.4
  networks:
    - mysqldb
  volumes:
    - ./mysqldb/low-memory-my.cnf:/etc/mysql/conf.d/low.cnf
    - do-vol-mysqldb:/var/lib/mysql
  restart: unless-stopped
```

The setup includes:

- **Custom configuration**: Memory-optimized settings override defaults
- **Persistent volumes**: Data survives container recreation
- **Isolated network**: Separate network for database access
- **Environment-based secrets**: Root password from encrypted environment

## Schema Management with Skeema

The game-changer for my database workflow is [Skeema](https://www.skeema.io/), which treats database schema as declarative code rather than imperative migrations.

### Schema-as-Code Structure

Each service's schema is defined as SQL CREATE statements:

```bash
migration-cli/
  databases/
    chores/
      user.sql
      chore_submission.sql
      goal.sql
    chatgpt/
      chat.sql
      message.sql
    tools/
      pipeline.sql
```

Here's an example table definition:

```sql
-- chores/user.sql
CREATE TABLE `user` (
  `id` binary(16) NOT NULL,
  `email_hash` binary(32) NOT NULL,
  `auth0_id` varchar(255) DEFAULT NULL,
  `role` enum('parent','child') NOT NULL,
  `display_name` varchar(100) NOT NULL,
  `profile_image_key` varchar(255) NULL,
  `profile_image_crop_data` json NULL,
  `family_id` binary(16) NOT NULL,
  `data` json NOT NULL,
  `created_at` datetime(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  `updated_at` datetime(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
  PRIMARY KEY (`id`),
  UNIQUE KEY `email_hash` (`email_hash`),
  KEY `idx_family_role` (`family_id`,`role`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### Migration Workflow

Schema changes follow a simple declarative workflow:

```bash
# 1. Edit the .sql files to match desired state
vim databases/chores/user.sql

# 2. Preview changes
nhdc db migrate # this loads a container with bash to run skeema commands

# 3. Apply changes
skeema diff
skeema push
```

This approach has several advantages:

- **No migration files**: Just modify the target schema
- **Automatic diffing**: Skeema calculates required changes
- **Safe operations**: Preview exactly what will change
- **Version controlled**: Schema definitions live in git

### Migration CLI Container

Schema management runs in a dedicated container:

```dockerfile
FROM golang:latest as builder
RUN go install github.com/skeema/skeema@v1.12.0
COPY . /app
WORKDIR /app/databases
ENTRYPOINT ["/bin/bash"]
```

This provides a consistent environment with Skeema pre-installed, accessed via:

```bash
nhdc db migrate
# Equivalent to:
docker run -it -v `pwd`/migration-cli:/app --rm --env-file .env --network=mono_mysqldb migration-cli
```

## Database Design Patterns

### UUID Primary Keys

I use binary(16) UUIDs for primary keys across all tables:

```sql
`id` binary(16) NOT NULL,
-- Selected as: bin_to_uuid(id) AS id
```

Benefits:

- **Distributed-friendly**: No coordination required for ID generation
- **Non-sequential**: Harder to enumerate records
- **Portable**: IDs work across database instances

### JSON Columns for Flexibility

MySQL's JSON column type handles semi-structured data:

```sql
`data` json NOT NULL,
`profile_image_crop_data` json NULL
```

This provides schema flexibility without sacrificing query performance through generated columns and JSON path indexing when needed.

### Soft Deletes

Most tables include soft delete support:

```sql
`deleted_at` datetime(6) DEFAULT NULL,
```

This preserves referential integrity and enables data recovery without complex backup restoration.
