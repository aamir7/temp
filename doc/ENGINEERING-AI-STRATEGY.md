---
title: "Engineering AI Strategy"
audience: "Leadership, EMs, architects, champions, senior engineers"
status: "Active"
owner: "Engineering leadership (CTO + EMs)"
version: "1.1"
last_reviewed: "2026-07-01"
review_cadence: "Quarterly, or after a harvest cycle encodes major rule changes"
---

# Engineering AI Strategy

## Executive summary (leadership)

We upskill engineering **incrementally** by putting standards where AI and humans already work: **this repository**. No separate platform is required to start.

| | |
|--|--|
| **Problem** | Assistants are available but uneven; AI speeds coding while **verification** (multitenancy, product flags, reuse) remains the constraint |
| **Approach** | Repo-first guardrails: `AGENTS.md`, `.cursor/rules/`, skills, ADRs, playbooks — grown from PR harvest |
| **Success** | Six outcomes (O-1–O-6): standard practice, shared standards, context in tools, less rework, output holds up (CAR), organizational memory |
| **Execution** | Five **sequential phases** with exit criteria — schedule in Redmine, not in git |
| **Not in scope** | Customer product AI; autonomous merge; fixed calendar deadlines in docs |

**Start here for depth:** §3 Outcomes → §4 Phases → §6 Expectations.

---

**Goal:** Incrementally upskill engineering so AI coding assistants **amplify** our standards — not bypass them.

**Scope:** How we build and maintain the monolith (EZOffice, CMMS, AssetSonar, EZRentOut). Customer-facing product AI is a separate product track.

**Scheduling:** Phases below are **sequential capabilities**, not calendar commitments. Prioritization and delivery are tracked in Redmine (or equivalent PM tooling) — not in this document.

---

## 1. Why this matters

| Reality | Implication |
|---------|-------------|
| Assistants are provisioned; usage is uneven | Entitlement ≠ capability — we standardize practice in the repo |
| AI writes code faster | **Verification** is the bottleneck (tests, review, multitenancy, product flags) |
| Knowledge lives in PRs and Redmine | It must flow back into rules, ADRs, and playbooks assistants load |
| 15-year monolith, four products, tribal knowledge | High-context, high-guardrail environment — not greenfield improvisation |

Industry pattern ([DORA 2025](https://dora.dev/dora-report-2025/)): AI is an **amplifier**. Where standards and measurement exist, quality improves; where they do not, rework increases.

**We do not need a separate engineering intelligence platform to start.** Version-controlled rules, skills, ADRs, and playbooks in this repository are the implementation vehicle.

---

## 2. Principles

1. **Repo-first** — Model-agnostic knowledge lives in git (`AGENTS.md`, `.cursor/`, `doc/`), not personal prompts.
2. **Verify, don't trust** — Every AI-assisted change passes the same loops as human work: tests, CI, review.
3. **Search before create** — Extend existing services, concerns, and patterns; avoid duplicate abstractions.
4. **Capture learning** — Recurring PR friction becomes rules, ADRs, or playbooks via harvest.
5. **Human ownership** — Architecture, security, and merge approval stay with engineers.
6. **Incremental depth** — Each phase adds capability only when the previous phase's outcomes are stable.

---

## 3. Outcomes (what “good” looks like)

Leadership and EMs judge progress against **outcomes**, not tool licenses or prompt volume.

| # | Outcome | Signal |
|---|---------|--------|
| **O-1** | Assistants are **standard practice** | Engineers with access use them for implementation; PR template reflects honest use |
| **O-2** | **Shared standards** on `main` | `.cursor/rules/`, skills, `AGENTS.md` — team pulls same baseline |
| **O-3** | **Context reaches the assistant** | Ticket + PR patterns loadable without Slack archaeology |
| **O-4** | **Less rework** | Review rounds and repeated comment themes trend down vs baseline |
| **O-5** | **AI output holds up** | Correct Acceptance Rate (CAR): sampled merges still correct weeks later |
| **O-6** | **Organizational memory grows** | ADRs, glossary, harvest-driven rule updates merge regularly |

Detailed KPI definitions: [engineering-ai-metrics/README.md](engineering-ai-metrics/README.md).

**CAR (KPI-4):** Begin sampling in Phase 2 so Phase 4 “stable CAR” is evidence-based (+6 weeks post-merge per sample).

---

## 4. Phased implementation

Advance when **exit criteria** are met (EM + architect/champion). Track work in Redmine — not in this file.

### Phase 1 — Foundation (shared context)

**Purpose:** Every engineer and assistant starts from the same facts about the monolith.

| Aspect | Detail |
|--------|--------|
| **Work** | Onboard to `AGENTS.md` + `.cursor/rules/`; Plan mode and `/grill-plan` for non-trivial work; PR template; capture metrics baseline |
| **Artifacts** | `AGENTS.md`, `.cursor/rules/`, `.cursor/skills/grill-plan/`, [starting-from-redmine-ticket.md](playbooks/starting-from-redmine-ticket.md), `baseline-YYYY-QN.csv` from [baseline-template.csv](engineering-ai-metrics/baseline-template.csv) |
| **Exit criteria** | Rules on `main`; ≥80% of engineers with access pulled latest `.cursor/`; Plan mode or `/grill-plan` norm for multi-file / multitenancy / new service work; baseline file started |

### Phase 2 — Verification (quality loops)

**Purpose:** AI output is checked before human reviewers spend time.

| Aspect | Detail |
|--------|--------|
| **Work** | `/review-pr` on non-trivial diffs; `/ultra-review` on high-risk PRs (see skill); playbooks for common tasks; ADRs for debated decisions; begin CAR sampling (≥5 PRs/period); optional PR **reuse note** (`extended: ServiceName` or `new: justified in comment`) for KPI-5 |
| **Artifacts** | `.cursor/skills/review-pr/`, `.cursor/skills/ultra-review/`, `.cursor/skills/receiving-code-review/`, playbooks, `doc/adr/` |
| **Exit criteria** | ≥50% of AI-assisted PRs (self-reported) run agentic review before human review; ADR process used for security/schema/async; CAR samples recorded |

### Phase 3 — Organizational memory (learning loop)

**Purpose:** Review comments improve the system for the next engineer.

| Aspect | Detail |
|--------|--------|
| **Work** | `/harvest-pr-insights`; champion triage → PR for rules/ADRs/playbooks; grow [domain-glossary.md](domain-glossary.md) as product-term → code bridge; link **golden modules** in playbooks (existing files, not rewrites) |
| **Artifacts** | `harvest/*.md`, [pr-review-insights-playbook.md](engineering-ai-metrics/pr-review-insights-playbook.md) |
| **Exit criteria** | ≥1 harvest cycle completed; top P1 themes encoded or explicitly deferred with ticket; ≥10 glossary terms or all recurring harvest terms covered |

### Phase 4 — Supervised agentic work

**Purpose:** Higher autonomy only where guardrails and verification are proven.

| Aspect | Detail |
|--------|--------|
| **Work** | Supervised Agent on published allowlist; agentic review habit; golden examples in playbooks; Boy Scout rule on touched legacy (local improvement only, no drive-by rewrites) |
| **Artifacts** | [agentic-workflows.md](playbooks/agentic-workflows.md), allowlist/denylist |
| **Exit criteria** | CAR acceptable on Agent-assisted samples; no increase in multitenancy/security incidents; EM sign-off to broaden allowlist |

### Phase 5 — Selective automation (optional)

**Purpose:** Machine-check only what harvest proves stable.

| Aspect | Detail |
|--------|--------|
| **Work** | CI/Danger rules mapped to Accepted ADRs after two stable harvest cycles |
| **Artifacts** | CI checks linked from ADR Consequences |
| **Exit criteria** | Targeted checks reduce repeat review themes without high false-positive rate |

```
Phase 1 ──► Phase 2 ──► Phase 3 ──► Phase 4 ──► Phase 5
 context     verify      learn       agent        CI
```

---

## 5. Guardrail layers

| Layer | Mechanism | Examples |
|-------|-----------|----------|
| **1 — Expectations** | This strategy + team norms | Plan mode, coaching |
| **2 — Rules** | `.cursor/rules/*.mdc` | multitenancy, `verify-before-inventing` |
| **3 — Workflow** | Playbooks, skills, PR template | `/grill-plan`, `/review-pr`, `/ultra-review`, Redmine playbook |
| **4 — CI** | Rubocop, Brakeman, Bullet | Unchanged; extend only in Phase 5 |
| **5 — Human** | Architect review, CAR sampling | CMDB migrations, credentials |

Precedence when documents conflict: **Accepted ADR** → scoped rule → `AGENTS.md` → playbook/skill.

---

## 5b. Evals and the feedback loop

**Guardrails** constrain or guide work *before and during* implementation. **Evals** measure whether guardrails worked *after* the fact. **Harvest** closes the loop: eval signals → champion triage → PR updating rules, skills, ADRs, or playbooks.

| Term | Meaning |
|------|---------|
| **Guardrail** | Rules, skills, playbooks, ADRs, PR template, CI checks (Phase 5) |
| **Eval** | CAR, harvest metrics, review rework, checklist compliance, skill adoption |
| **Harvest** | Structured learning from merged PR comments → encoded guardrails |

A guardrail without an eval is unmeasured. An eval without a harvest path is noise.

### Guardrail → eval mapping

| Guardrail layer | What it prevents | Eval signal | Owner |
|-----------------|------------------|-------------|-------|
| Rules (`.mdc`) | Hallucinated patterns, tenancy bugs | Harvest theme frequency; CAR defects in category | Champion |
| `/grill-plan` | Wrong architecture before code | Plan rework; mid-PR scope changes | Engineer |
| `/review-pr` | Review-round rework | % AI-assisted PRs with agentic review; fix rate on findings | EM |
| `/ultra-review` | Missed cross-cutting bugs on risky PRs | Ultra-review findings vs human reviewer overlap | Champion |
| ADRs | Repeated debates | Same harvest theme after ADR accepted | Architect |
| CAR (KPI-4) | “Looked fine at merge” failures | % acceptable at +6 weeks post-merge | Champion |
| Harvest (KPI-8) | Stagnant standards | Rules/skills merged per quarter | Leadership |

### Eval cadence

| Eval | When | Example pass signal |
|------|------|---------------------|
| PR harvest | Quarterly | Top P1 themes encoded or ticketed |
| CAR sample | Ongoing (Phase 2+) | ≥80% acceptable at +6 weeks |
| Review rework (KPI-7) | Quarterly | Trend down vs baseline |
| Agentic review adoption | Monthly | ≥50% AI-assisted PRs run `/review-pr` (Phase 2 exit) |
| Guardrail drift audit | Quarterly | No broken skill links; rules match codebase paths |

Detailed KPI definitions: [engineering-ai-metrics/README.md](engineering-ai-metrics/README.md).

**Evals are not:** prompt volume, license count, raw suggestion acceptance %, or individual engineer rankings.

---

## 6. Expectations (all engineers)

### Tools

| Tool | Status |
|------|--------|
| **Cursor** | Primary approved assistant |
| **Claude** (desktop/API) | Approved for analysis; same data rules as Cursor |
| Other tools with company source | CTO exception required |

Use **approved tools only** for repo code, Redmine content, and internal docs.

### Data handling (required)

| Allowed | Requires caution | Never paste |
|---------|------------------|-------------|
| Source from this repo | Redmine ticket text (project scope) | Production credentials, API keys, `.env` |
| Internal `doc/` artifacts | Customer PII from tickets | Full production DB dumps |
| Local test fixtures | | Unapproved third-party AI tools |

Align with SOC2 / ISO27001 practices. Credentials: [ADR-0001](adr/0001-integration-credentials-storage.md). When unsure, ask security lead before pasting.

### Practice by experience

| Role | Expectation |
|------|-------------|
| **All engineers** | Use an approved assistant for implementation, refactoring, and discovery when access is granted |
| **Junior engineers** | Plan mode for non-trivial work; follow [playbooks/](playbooks/) |
| **Senior engineers** | Model effective use; contribute rules, playbooks, ADRs |
| **Champions** | Coach, run harvest triage, surface gaps to architects |

### Plan mode — use when any apply

- More than one file changes
- New service, API endpoint, migration, or concern
- Multitenancy, `IS_*` flags, or authorization involved
- Redmine scope is ambiguous

### Agentic modes

| Mode | When |
|------|------|
| **Plan** | Default for non-trivial work (Phase 1+); use `/grill-plan` when scope is ambiguous or high-risk |
| **Agentic review** (`/review-pr`, alias `/review`) | Encouraged Phase 2+ before human review |
| **Ultra-review** (`/ultra-review`) | Phase 2+ for high-risk PRs only (auth, migrations, CMDB, cross-product, raw SQL) |
| **Receiving review** (`/receiving-code-review`) | After human review comments — triage and respond |
| **Agent** | Phase 4+ only; allowlisted paths; human reviews every diff |
| **Autonomous merge** | **Not approved** |

### Human-only

Architecture decisions (document in ADR when significant), security-sensitive changes, production operations, **final PR merge**.

### Pull requests

Use [.github/pull_request_template.md](../.github/pull_request_template.md) — Redmine link, AI assistance section, standard checklist.

---

## 7. Roles

| Role | Responsibility |
|------|----------------|
| **Engineer** | Load repo standards; Plan/review loops; honest PR disclosure |
| **Reviewer** | [reviewer-comment-template.md](../.cursor/skills/review-pr/reviewer-comment-template.md) tags; enforce ADRs |
| **Champion** | Harvest triage; coaching; metrics sampling |
| **Architect** | ADRs; CMDB/schema gates; allowlist for Agent |
| **EM** | Phase progression; team outcomes; escalation to leadership |

---

## 8. Repository artifacts

Full index: [README-engineering-ai.md](README-engineering-ai.md) (engineer hub). Leadership reads this strategy; engineers start at the hub.

---

## 9. What we are not doing

- Building a separate engineering AI platform before repo guardrails work
- Mandating Agent or autonomous merge org-wide
- Replacing human architecture or security review
- Customer-facing AI (product team scope)
- Fixed calendar deadlines in git docs — use Redmine for scheduling

---

## 10. Industry alignment (summary)

- **Verification-first SDLC** — test, CI, and review loops around generation ([DORA](https://dora.dev/dora-report-2025/))
- **ADRs** — durable decisions with context and consequences ([Google Cloud ADR guide](https://cloud.google.com/architecture/architecture-decision-records))
- **Repo-native context** — same pattern as Shopify/GitHub engineering playbooks: standards in version control assistants load

---

*Single source of truth for Engineering AI direction and phased upskilling.*
