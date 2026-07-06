# Playbook: Agentic Workflows (Plan, Agent, Agentic Review)

When to use **Plan mode**, **Agent mode**, and agentic PR review skills.

**Phases:** See [ENGINEERING-AI-STRATEGY.md](../ENGINEERING-AI-STRATEGY.md) §4 — `/grill-plan` (Phase 1+), `/review-pr` (Phase 2+), `/ultra-review` (Phase 2+ high-risk), Agent (Phase 4+).

## Recommended flow

```
Redmine ticket → /grill-plan → approved plan → implement → /review-pr → human review
                                    ↓
              (high-risk PR) → /ultra-review → triage → /receiving-code-review → re-request review
```

## Spectrum (lowest autonomy that works)

| Mode | Use when | Human effort |
|------|----------|--------------|
| **Inline / Chat** | Quick questions, single edits | High |
| **Grill plan** (`/grill-plan`) | Ambiguous scope, architecture, cross-domain | Medium — answer questions |
| **Plan** | Multi-file, new features after grill (or skip grill if obvious) | Medium — approve plan first |
| **Agent** | Bounded tasks; allowlisted paths only (Phase 4+) | Lower — review every diff |
| **Agentic review** (`/review-pr`, `/review`) | Before human PR review (Phase 2+) | Low — triage findings |
| **Ultra-review** (`/ultra-review`) | High-risk PRs only | Low — triage merged findings |
| **Autonomous end-to-end** | **Not approved** | Human merges only |

**Default:** Plan mode or `/grill-plan` for non-trivial work.

---

## Grill plan (Phase 1+)

```
/grill-plan
```

Skill: [.cursor/skills/grill-plan/SKILL.md](../../.cursor/skills/grill-plan/SKILL.md)

Use when ticket scope is ambiguous, multitenancy/auth/products are involved, or you want Socratic pushback before coding. Outputs an approved plan block — does not implement.

Pair with [starting-from-redmine-ticket.md](starting-from-redmine-ticket.md).

---

## Plan mode

### When required

- More than one file changes
- New service, endpoint, migration, or concern
- Multitenancy, `IS_*` flags, or authorization
- [starting-from-redmine-ticket.md](starting-from-redmine-ticket.md) workflow

### Steps

1. Load ticket, glossary, relevant rules (or run `/grill-plan` first)
2. Plan includes **search before create**
3. Approve plan before execution
4. Run tests; complete PR template

---

## Agent mode (Phase 4+)

### Prerequisites

- [ ] Shared `.cursor/rules/` and `AGENTS.md` on `main`
- [ ] Phase 4 exit criteria from prior phases met
- [ ] Task on **allowlist** or architect approval

### Allowlist (publish and maintain with architects)

| Generally OK | Human-led / denylist |
|--------------|----------------------|
| `doc/`, comments, isolated specs | `db/migrate/` without review |
| Bounded module refactors | `Ability`, auth, payments |
| Test generation for touched files | Global initializers |
| Style cleanups | Cross-product flags without senior review |

### Checklist

- [ ] Scope in prompt (max files / directories)
- [ ] "Follow `.cursor/rules/` and `AGENTS.md`"
- [ ] Review every file change and command run
- [ ] Human PR review before merge

---

## Agentic review (Phase 2+)

```
/review-pr
```

Alias: `/review` (same skill).

Skill: [.cursor/skills/review-pr/SKILL.md](../../.cursor/skills/review-pr/SKILL.md)

After review: triage findings; recurring themes → champion proposes rule update.

### Ultra-review (high-risk PRs only)

```
/ultra-review
```

Skill: [.cursor/skills/ultra-review/SKILL.md](../../.cursor/skills/ultra-review/SKILL.md)

Run when PR touches Ability, migrations, CMDB, credentials, cross-product logic, raw SQL, or large React+API changes.

### Receiving human review

```
/receiving-code-review
```

Skill: [.cursor/skills/receiving-code-review/SKILL.md](../../.cursor/skills/receiving-code-review/SKILL.md)

After human reviewers comment — triage, fix, draft replies.

---

## React UI workflow

1. `/react-product-ui` — sibling pattern, product gating, tokens
2. `/react-new-module` — scaffold files if new module
3. `/review-pr` — before human review

---

## Not approved

| Technique | Status |
|-----------|--------|
| Autonomous PR merge by AI | No |
| Background agents on production paths | No |
| Agent without CI + human review | No |
| `/ultra-review` on every PR | No — high-risk only |

---

## Related

- [ENGINEERING-AI-STRATEGY.md](../ENGINEERING-AI-STRATEGY.md) §5b (guardrails vs evals)
- [create-application-service.md](create-application-service.md)
