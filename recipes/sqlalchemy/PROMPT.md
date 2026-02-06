# SQLAlchemy Skill Update Instructions

When updating this skill, follow these guidelines:

## Priority: SQLAlchemy Core over ORM

This skill targets users who primarily use **SQLAlchemy Core** (SQL Expression Language) rather than the ORM. While the skill includes ORM documentation for completeness, prioritize:

1. **Core/SQL Expression Language patterns** - Direct SQL construction, `text()`, `select()`, `insert()`, `update()`, `delete()` without ORM models
2. **Engine and Connection management** - Connection pooling, raw SQL execution
3. **Query building** - Using Core constructs like `Table`, `Column`, `MetaData`

## Content Guidelines

- Keep ORM sections concise and reference-focused
- Expand Core sections with more practical examples
- When adding new content, prefer Core patterns over ORM patterns
- Include database reflection patterns (working with existing schemas)
- Focus on PostgreSQL-specific features when relevant (this is the most common database in the target environment)

## What NOT to Expand

- Do not add extensive ORM relationship patterns beyond what exists
- Do not add advanced ORM features like hybrid properties, custom collections, or polymorphism
- Do not add framework integrations (Flask-SQLAlchemy, FastAPI patterns, etc.)
