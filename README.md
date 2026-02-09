# skill-forge

[![conda-forge](https://img.shields.io/badge/prefix.dev%2Fskill--forge-F7CC49?style=flat-square)](https://prefix.dev/channels/skill-forge)

A collection of agent skills packaged as conda packages and published to the [skill-forge](https://prefix.dev/channels/skill-forge) channel on prefix.dev.

Agent skills are markdown files that give AI coding agents specialized knowledge about libraries, tools, and domains.
They are managed by [pixi-skills](https://github.com/pavelzw/pixi-skills) and can be installed into any pixi project.

## Available skills

| Skill | Package | Description |
|-------|---------|-------------|
| [conda-forge](https://conda-forge.org) | `agent-skill-conda-forge` | conda-forge packaging operations |
| [Polars](https://pola.rs) | `agent-skill-polars` | DataFrame library for fast data manipulation |
| [SQLAlchemy](https://www.sqlalchemy.org) | `agent-skill-sqlalchemy` | Python SQL toolkit and ORM |
| [Typst](https://typst.app) | `agent-skill-typst` | Modern markup-based typesetting system |

## Usage

### Managing skills with pixi-skills

The recommended way to use agent skills is through [pixi-skills](https://github.com/pavelzw/pixi-skills).
Install it with:

```bash
pixi exec pixi-skills manage
```

This will interactively guide you through adding skills to your project.

### Manual setup

Add the `skill-forge` channel and the desired skill packages to your `pixi.toml`:

```toml
[workspace]
channels = ["conda-forge", "https://prefix.dev/skill-forge"]
platforms = ["linux-64", "osx-arm64", "win-64"]

[dependencies]
polars = ">=1,<2"

[feature.dev.dependencies]
agent-skill-polars = "*"
```
