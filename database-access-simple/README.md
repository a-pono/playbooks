# Database Access for AI Agents - The Simple Way

Stop building MCP servers for database access. A single markdown file + CLI tools is all you need.

## How It Works

1. You write one file (`DATABASES.md`) describing your databases - connection info, tables, relationships, query patterns
2. Your AI agent reads it
3. The agent picks the right CLI tool (`psql`, `clickhouse-client`, etc.) and runs queries directly

No servers. No protocol overhead. No dependencies beyond the CLI tools you already have installed.

## What's Inside

- `DATABASES.md` - the actual file your AI agent reads. Drop it into your project and customize it.

## Example Setup

This example covers a realistic scenario:

- **PostgreSQL** - transactional data, product catalog, user accounts
- **ClickHouse** - analytical warehouse, billions of events
- **Cross-source joins** - ClickHouse querying Postgres dictionaries via `postgresql()` table function
- **Local file queries** - ClickHouse querying CSV/Parquet files with `file()` or `clickhouse-local`

## Setup

1. Copy `DATABASES.md` into your project root
2. Edit connection details, table names, and query patterns to match your setup
3. Done - most AI coding agents (Claude Code, Cursor, Copilot) will pick it up automatically

If your agent doesn't discover the file on its own, add this line to your `CLAUDE.md`, `AGENTS.md`, or equivalent instructions file:

```
Before running any database query, read DATABASES.md for connection details, available tables, and query patterns.
```

That's it. No install, no config, no build step.

## Why Not MCP?

MCP is great for some use cases - tool orchestration, multi-step workflows, external API access. But for databases, you already have battle-tested CLI tools that do the job. Adding a protocol layer between your agent and `psql` is complexity with no upside.
