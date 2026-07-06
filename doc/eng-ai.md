# Engineering AI — Engineer Hub

**Strategy and phases:** [ENGINEERING-AI-STRATEGY.md](ENGINEERING-AI-STRATEGY.md) (leadership + incremental plan).

This page is the **day-to-day index** for engineers, champions, and AI assistants.

| I want to… | Go to |
|------------|-------|
| Understand the repo | [AGENTS.md](../AGENTS.md) → [.cursor/rules/](../.cursor/rules/) |
| Stress-test a plan before coding | `/grill-plan` → [grill-plan/SKILL.md](../.cursor/skills/grill-plan/SKILL.md) |
| Follow a task workflow | [playbooks/README.md](playbooks/README.md) |
| Agentic PR review | `/review-pr` (alias `/review`) → [review-pr/SKILL.md](../.cursor/skills/review-pr/SKILL.md) |
| High-risk multi-angle PR review | `/ultra-review` → [ultra-review/SKILL.md](../.cursor/skills/ultra-review/SKILL.md) |
| Respond to human review comments | `/receiving-code-review` → [receiving-code-review/SKILL.md](../.cursor/skills/receiving-code-review/SKILL.md) |
| PR comment harvest (champions) | `/harvest-pr-insights` → [harvest-pr-insights/SKILL.md](../.cursor/skills/harvest-pr-insights/SKILL.md) |
| React UI consistency | `/react-product-ui` → [react-product-ui/SKILL.md](../.cursor/skills/react-product-ui/SKILL.md) |
| Domain workflows (MDM, React, queues, custom roles, …) | [.cursor/skills/README.md](../.cursor/skills/README.md) |
| Architecture decision | [adr/README.md](adr/README.md) |
| Redmine term → code symbol | [domain-glossary.md](domain-glossary.md) |
| Metrics & KPIs | [engineering-ai-metrics/README.md](engineering-ai-metrics/README.md) |

---

## Setup

1. Pull latest `main`
2. Read [AGENTS.md](../AGENTS.md) once
3. Rules load from `.cursor/rules/` — index: [.cursor/README.md](../.cursor/README.md)
4. **Plan mode** or **`/grill-plan`** for multi-file, multitenancy, new services, ambiguous tickets
5. Optional: `/review-pr` before requesting human reviewers; `/ultra-review` for high-risk PRs

---

## Champions

```bash
rake engineering_ai:pr_review_metrics DAYS=90
rake engineering_ai:pr_comments_export DAYS=90
# Cursor: /harvest-pr-insights DAYS=90
```

Triage workflow: [engineering-ai-metrics/pr-review-insights-playbook.md](engineering-ai-metrics/pr-review-insights-playbook.md).

---

## Document map

| Layer | Location |
|-------|----------|
| Strategy | [ENGINEERING-AI-STRATEGY.md](ENGINEERING-AI-STRATEGY.md) |
| Steering | `AGENTS.md`, `.cursor/` |
| Decisions | `doc/adr/` |
| Procedures | `doc/playbooks/`, `.cursor/skills/` |
| Measurement | `doc/engineering-ai-metrics/` |
