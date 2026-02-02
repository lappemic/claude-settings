# Claude Code Config

Personal Claude Code configuration, portable across machines.

## Structure

```
CLAUDE.md        # Global instructions and coding conventions
MEMORY.md        # Persistent memory across sessions
settings.json    # Tool permissions, plugins, status line config
```

### `/commands` — Slash Commands

Reusable workflows invoked via `/command-name`:

| Command | Purpose |
|---------|---------|
| `/debug` | Debug helper |
| `/migrate` | Database migration helper |
| `/new-ai-route` | Scaffold AI API route |
| `/new-component` | Scaffold new component |
| `/perf` | Performance analysis |
| `/pr` | Create pull request |
| `/refactor` | Refactor assistant |
| `/review` | Code review |
| `/scaffold-test` | Scaffold test files |
| `/sec` | Security audit |
| `/smart-commit` | Semantic commit grouping |

### `/agents` — Specialized Subagents

Long-running agents for complex domain tasks:

| Agent | Purpose |
|-------|---------|
| ai-feature-engineer | AI feature implementation and eval |
| analytics-growth | Analytics, SEO, and growth optimization |
| data-ingestion-builder | Scrapers and data pipelines |
| performance-cost-optimizer | Cost and latency optimization |
| postgres-schema-architect | Schema design, RLS, migrations |
| stripe-integration-manager | Stripe payment integrations |
| test-architect | Test infrastructure and generation |

### `/skills` — Portable Skills

Skills that extend Claude's capabilities:

| Skill | Purpose |
|-------|---------|
| remotion-best-practices | Remotion video creation in React |
| seo-audit | Technical SEO auditing |
| ship-to-prod | Promote staging to production via PR |
| stage-and-ship | Full git workflow automation |
| umami-cloud | Umami Cloud analytics configuration |

## Setup

Clone into `~/.claude/`:

```sh
git clone <repo-url> ~/.claude
```
