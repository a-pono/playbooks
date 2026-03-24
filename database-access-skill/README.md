# Database Access Skill for AI Agents

A structured approach to giving AI coding agents safe, accurate access to your databases.

Instead of building MCP servers or custom tooling, this skill uses a folder of markdown files + the CLI tools you already have installed.

## How It Works

1. **SKILL.md** - the entry point. Security rules, available databases, query rules, workflow.
2. **Database reference files** - one per database. Connection info, table registry with descriptions and row counts. The agent reads this to figure out where the data lives.
3. **DDL files** - one per table/view. Schema, relationships, and statistics (row counts, null rates, distinct values, value ranges). The agent reads only the tables it needs - not everything.
4. **doc-rules.md** - documentation standards. The agent can create new DDL files following the same format. The skill grows itself.

## What's Inside

This example covers a realistic e-commerce scenario:

- **PostgreSQL** - transactional data (products, users, orders)
- **ClickHouse** - analytical warehouse (5B+ events, 12B+ page views)

## Setup

1. Copy this folder into your project
2. Copy `assets/.env.example` to `.env` in the project root and fill in your credentials
3. Edit the database reference files and DDL files to match your actual databases
4. Your AI agent reads `SKILL.md` and takes it from there

## Why Not MCP?

[See the companion post](https://www.linkedin.com/feed/update/urn:li:activity:7439634762421567490/)
