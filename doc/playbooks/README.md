# Engineering Playbooks

Task guides for **humans and AI assistants**. Use with Plan mode and `AGENTS.md`.

**Strategy:** [ENGINEERING-AI-STRATEGY.md](../ENGINEERING-AI-STRATEGY.md)

## Index

| Situation | Playbook |
|-----------|----------|
| Redmine ticket → implementation | [starting-from-redmine-ticket.md](starting-from-redmine-ticket.md) |
| New service object | [create-application-service.md](create-application-service.md) |
| Plan, Agent, agentic review | [agentic-workflows.md](agentic-workflows.md) — `/grill-plan`, `/review-pr`, `/ultra-review` |
| React / API parsers | [react-and-api-contracts.md](react-and-api-contracts.md) |
| CMDB migrations | [cmdb-schema-changes.md](cmdb-schema-changes.md) |
| PR harvest (champions) | [.cursor/skills/harvest-pr-insights/SKILL.md](../../.cursor/skills/harvest-pr-insights/SKILL.md) |

Harvest triage: [pr-review-insights-playbook.md](../engineering-ai-metrics/pr-review-insights-playbook.md).

## Playbook format

1. When to use
2. Before you code (search, glossary, rules)
3. Steps
4. Checklist / PR gates
5. Golden references in repo

## vs other docs

| Doc | Role |
|-----|------|
| `ENGINEERING-AI-STRATEGY.md` | Why and phased outcomes |
| `AGENTS.md` / `.cursor/rules/` | Constraints |
| `doc/adr/` | Decisions |
| Playbooks | How-to for a task |

## Contributing

PR with champion or tech lead review; test via Plan mode on a real ticket.
