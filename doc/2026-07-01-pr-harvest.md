# PR Harvest Report — 2026-07-01

> **Period:** last 90 days · **PRs analyzed:** 271 · **Analyst:** Cursor `/harvest-pr-insights`  
> **Data:** `doc/engineering-ai-metrics/pr-comments-export.json` (captured 2026-07-01)  
> **Status:** DRAFT — requires champion approval before any rule changes

---

## Executive summary

1. **Reviewer-structured tags (`🔴 **Architecture**`) are high-signal** — 19 architecture + 3 security explicit tags; mostly from one senior reviewer pattern worth standardizing team-wide.
2. **React/API contract drift is the top technical theme** — `Member = any`, snake_case vs camelCase parsers, missing serializer fields, async navigation before thunk completion (PR #37307 cluster).
3. **Security: credentials in source** appeared in multiple PRs — flagged 🔴 but merge status unclear; **CAR audit recommended** on #37556, #37684, #37685.
4. **CMDB & cost-center features drove highest comment volume** — schema/index debates often end as `deferred_followup` ("must discuss with me") rather than rule updates.
5. **Keyword harvest baseline was misleading** — prior `pr-review-insights.json` (5-PR sample) classified Brakeman *pass* messages as "security"; LLM harvest excludes that noise.

---

## Methodology

| Step | Detail |
|------|--------|
| Export | `pr-comments-export.json` — 271 PRs, 1,141 threads, 1,950 messages |
| PRs processed | All PRs with comments (90-day window, no SAMPLE) |
| Batches | Full export analyzed; top 50 PRs by message count weighted in synthesis |
| Focus | All categories |
| Keyword baseline | Compared to `pr-review-insights.json` (5-PR sample — not representative) |
| Context | `AGENTS.md`, `verify-before-inventing.mdc`, `review-pr/checklist.md` |

**Population context (from `pr-review-metrics.json`):** 335 merges / 90d · 99 PRs with `CHANGES_REQUESTED` (30%) · median 1 comment/PR (long tail: few PRs dominate discussion).

---

## Theme catalog

| # | Theme | Category | Confidence | Disposition mix | PRs | Rule exists? |
|---|-------|----------|------------|-----------------|-----|--------------|
| 1 | React TS safety & API shape | architecture | **HIGH** | missed + addressed | 8+ | Partial — `react-frontend.mdc` |
| 2 | Credentials / secrets in repo | security | **HIGH** | missed / unclear | 3 | Partial — Brakeman only |
| 3 | `company_id` composite indexes | performance_sql / multitenancy | **HIGH** | addressed + deferred | 6+ | Yes — `database-migrations.mdc` |
| 4 | Frontend–backend rule duplication | architecture | **MEDIUM** | missed | 2 | No dedicated rule |
| 5 | `IS_*` / product exposure | product_flags | **MEDIUM** | discussed | 4+ | Yes — `00-project-context.mdc` |
| 6 | i18n hardcoding (React + mailers) | i18n | **MEDIUM** | missed | 5+ | PR template only |
| 7 | Rake vs model callback duplication | architecture | **MEDIUM** | discussed | 3 | No |
| 8 | CMDB schema / index design debates | architecture | **MEDIUM** | deferred_followup | 4+ | No ADR |
| 9 | Cost center domain completeness | specification_scope | **MEDIUM** | deferred + PM TODOs | 10+ | Glossary gap |
| 10 | Brakeman bot "Good stuff" comments | process_noise | **HIGH** | noise | 50+ | N/A — exclude from harvest |

---

### Theme 1: React TypeScript safety & API contract

**Summary:** Reviewers repeatedly catch `type X = any`, parser fallbacks that pass raw snake_case API objects to UI, wrong i18n keys in dialogs, and navigation before async thunks complete. This is the densest architecture cluster in the period.

**Why it matters:** AI-assisted React work amplifies type erasure and API-shape bugs; UI looks correct in dev but breaks on edge paths (parser throw, wrong label, race on delete).

**Evidence:**

| PR | Path / thread | Disposition | Quote (≤120 chars) | Reviewer |
|----|---------------|-------------|---------------------|----------|
| #37307 | `member.data.ts` · 1 msg | actionable_missed | "`export type Member = any` completely defeats TypeScript safety" | @aliadnanaslam7 |
| #37307 | `memberListingScreen.action.ts` · 1 msg | actionable_missed | "raw API response objects are cast as `Member[]`… snake_case keys" | @aliadnanaslam7 |
| #37307 | `memberActivate.dialog.tsx` · 1 msg | actionable_missed | "activate dialog uses deactivate_confirmation_label" | @aliadnanaslam7 |
| #37307 | vendor delete · 1 msg | actionable_missed | "`handleVendorDeleteSuccess()` called right after dispatching async thunk" | @aliadnanaslam7 |
| #37810 | `glpi_ticket_serializer.rb` · 1 msg | actionable_missed | "status chip depends on `status_color` from API but serializer does not…" | @aliadnanaslam7 |
| #37826 | workflow converter · 1 msg | actionable_addressed | "relevant attrs not added in workFlowAutomation.ts converter file" | @Zaid-7vals |

**Recommended practice:**

- **Follow:** Use existing typed models under `core/models/data/`; never widen to `any` for expedience.
- **Avoid:** Parser catch blocks that dispatch untyped API payloads; copy-paste wrong i18n keys; navigate before thunk settles.

**Proposal (draft — not merged):**

- [x] Update `react-frontend.mdc` — add bullets: no `type X = any` when interface exists; parser failures must not dispatch raw API objects; verify i18n key matches action (activate vs deactivate).
- [x] Update `review-pr/checklist.md` — React row: "API shape / converter / serializer parity checked"
- [x] Add glossary term — "Member" in React vs `User` in Rails (see domain-glossary)
- [x] Playbook — `doc/playbooks/react-and-api-contracts.md`

---

### Theme 2: Credentials and secrets in source

**Summary:** Multiple PRs introduced `access_token` / `itam_access_token` in `company.rb` or predictable `secure_code` derivation in rake tasks. Reviewer flagged 🔴 **Security**; disposition unclear without post-merge audit.

**Why it matters:** Committed credentials are irreversible exposure; AI may copy "add token to model" from training data.

**Evidence:**

| PR | Path / thread | Disposition | Quote (≤120 chars) | Reviewer |
|----|---------------|-------------|---------------------|----------|
| #37556 | `company.rb` · 1 msg | actionable_missed | "adds access_token / itam_access_token directly into source" | @aliadnanaslam7 |
| #37684 | `company.rb` · 1 msg | actionable_missed | "new credentials directly in source code" | @aliadnanaslam7 |
| #37685 | `one_time_2026.rake` · 1 msg | actionable_missed | "`secure_code` derived deterministically… truncated to 4 chars" | @aliadnanaslam7 |
| #37677 | AWS integration · 1 msg | actionable_addressed | "encrypt credentials" | @bk-az |

**Recommended practice:**

- **Follow:** Encrypt at rest; use existing credential vault / env patterns; request `encrypt credentials` in security-sensitive integrations.
- **Avoid:** Tokens in model constants, mappings in Ruby source, predictable codes from subdomain.

**Proposal (draft — not merged):**

- [x] Update `verify-before-inventing.mdc` — add row: credentials never in `company.rb` / rake / mapping constants
- [x] Update `review-pr/checklist.md` — Critical: no secrets in diff
- [x] Add ADR — ADR-0001 integration credentials
- [ ] Schedule CAR audit on PRs #37556, #37684, #37685

---

### Theme 3: `company_id` composite indexes on migrations

**Summary:** CMDB and relationship PRs include explicit reviewer requests for composite indexes leading with `company_id`. Some threads defer full schema discussion ("must discuss this table with me in detail").

**Why it matters:** Multitenancy performance at scale; already in rules but **migration PRs still miss indexes**.

**Evidence:**

| PR | Path / thread | Disposition | Quote (≤120 chars) | Reviewer |
|----|---------------|-------------|---------------------|----------|
| #37518 | migration thread · 3 msgs | deferred_followup | "add index on company_id + cmdb_configuration_item_relationship_type_id + source_id" | @bk-az |
| #37518 | migration · 1 msg | deferred_followup | "index on company_id + … **must discuss this table with me in detail**" | @bk-az |
| #37518 | migration · 1 msg | discussed_not_actioned | "remove source_type and target_type… relationship_type_id will serve the purpose" | @bk-az |

**Recommended practice:**

- **Follow:** `EXPLAIN` + composite index with `company_id` first (PR template SQL section).
- **Avoid:** Merging CMDB schema without architect sign-off when reviewer explicitly requests discussion.

**Proposal (draft — not merged):**

- [x] No new rule — enforce existing `database-migrations.mdc` via `/review-pr` on `db/migrate/**`
- [x] Add ADR + playbook — ADR-0002, `doc/playbooks/cmdb-schema-changes.md`
- [x] CMDB schema review gate — label `Ready for Architect Review` in playbook

---

### Theme 4: Frontend–backend business rule duplication

**Summary:** Work order qualifying item types duplicated in React submit button and `Task` model; reviewer warns of drift risk.

**Evidence:**

| PR | Path | Disposition | Quote | Reviewer |
|----|------|-------------|-------|----------|
| #37595 | `submitButton.tsx` | actionable_missed | "rules duplicated in frontend and backend… creates drift risk" | @aliadnanaslam7 |
| #37269 | workflow slices | positive_pattern | "copy-pasted identically across 5 form slices… consider shared matcher" | @aliadnanaslam7 |

**Confidence:** MEDIUM (2 PRs, clear reasoning)

**Proposal:**

- [x] Add playbook bullet: business rules live in backend — `doc/playbooks/react-and-api-contracts.md` §3

---

### Theme 5: Product flags (`IS_*`) and rentals exposure

**Summary:** Features merged without explicit guards for which product sees UI (cost centers, baskets, workflows).

**Evidence:**

| PR | Disposition | Quote | Reviewer |
|----|-------------|-------|----------|
| #37826 | discussed_not_actioned | "rentals check? or are we exposing this in ezr ?" | @Zeeshan-Siddhu |
| #37749 | actionable_addressed | "wrap inside check of IS_EZ_OFFICE_NOT_CMMS and enable_cost_centers?" | @Zeeshan-Siddhu |

**Proposal:**

- [x] Reinforce in `starting-from-redmine-ticket.md` Plan prompt: list target `IS_*` products before UI work.
- [x] Already in `00-project-context.mdc` — enforcement via review, not new rule.

---

## Practices to follow

| Pattern | Evidence PRs | Guidance |
|---------|--------------|----------|
| Structured reviewer tags `🔴 **Category**:` | #37307, #37685, #37810 | Standardize in reviewer guide — improves harvest + author triage |
| DRY callout across Redux slices | #37269 | Extract shared matchers when 3+ slices copy-paste |
| Security fix + ask for regression test | #37450 | "add regression test covering encoded payloads" |
| PM/TODO gate before merge | #37381 | Unresolved "Correct Data from PM" flagged 🟢 but blocks silent debt |
| Encrypt credentials prompt | #37677 | Integration PRs: default to encrypted storage |

---

## Practices to avoid

| Anti-pattern | Confidence | Evidence PRs | Encode in |
|--------------|------------|--------------|-----------|
| `type Foo = any` when types exist | HIGH | #37307, #37360 | `react-frontend.mdc` |
| Secrets / tokens in Ruby source | HIGH | #37556, #37684 | `verify-before-inventing.mdc` + security review |
| Migrations without `company_id` index plan | HIGH | #37518 | `/review-pr` + PR template SQL |
| Duplicate FE/BE business rules | MEDIUM | #37595 | Playbook |
| Hardcoded English in user-facing React | MEDIUM | #37179, #37307 | PR template i18n + `react-frontend.mdc` |
| Relying on Brakeman bot pass as "security review" | HIGH | many | Exclude bots from harvest; human security on creds |

---

## Overlooked themes

Themes keyword analysis or casual reading would miss:

| Theme | Why overlooked | Evidence |
|-------|----------------|----------|
| CMDB schema deferred to architect 1:1 | Thread ends in "discuss with me" — no rule created | PR #37518 @bk-az |
| Brakeman bot false positive in keyword task | Automated "Good stuff 💯" classified as security theme | Compare keyword JSON vs this report |
| Workflow converter attr parity | Buried in cost-center PR thread | PR #37826 @Zaid-7vals |
| Async race on delete success handler | Logic bug not architecture keyword | PR #37307 |
| `AI Comments Pending` label as quality signal | 10+ PRs carry label — process metric not in template | #37677, #37505, etc. |
| Wrong dialog i18n key (activate/deactivate) | Specification/UI bug | PR #37307 |

---

## Discussed but not actioned (intentional)

| PR | Topic | Decision (from thread) | Follow-up ticket? |
|----|-------|------------------------|-------------------|
| #37518 | CMDB column removal vs relationship_type_id | Reviewer: remove redundant type columns; author must discuss table in detail | Unknown — architect gate |
| #37826 | Cost center UI on EZR | "rentals check?" — exposure question raised | Unknown |
| #37839 | Skybitz auth_credentials blank check | Integration-specific exception to blank check | Document in integration playbook |
| #37381 | `#cost_center_select` for future phase | 🟢: OK if future phase — comment or remove | PM phase 2 |

---

## CAR audit candidates

PRs with `actionable_missed` on HIGH themes — sample at +6 weeks:

| PR | Theme | Merged | Audit question |
|----|-------|--------|----------------|
| #37556 | Credentials in source | 2026-Q2 | Were tokens rotated / removed from git history? |
| #37684 | Credentials in source | 2026-Q2 | Same |
| #37685 | Predictable secure_code | 2026-Q2 | Was algorithm changed before prod rake run? |
| #37307 | React logic/type bugs | 2026-Q2 | Any hotfix on members detail screen post-merge? |
| #37595 | FE/BE duplication | 2026-Q2 | Did drift cause ticket within 6 weeks? |

---

## Comparison to keyword harvest

| Theme | Keyword task count | LLM count | Notes |
|-------|-------------------|-----------|-------|
| security | 5 (100% of 5-PR sample) | 3 explicit 🔴 + 8 credential-adjacent | Keyword sample was Brakeman bot noise |
| architecture | 0 | 19 tagged + ~30 untagged | Keyword missed structured tags |
| multitenancy/index | 0 | 27+ mentions | Keyword patterns too narrow |
| i18n | 0 | 25+ mentions | |
| cost centers (domain) | 0 | 48 mentions | Domain cluster invisible to generic keywords |

---

## Champion decisions required

- [x] **Approve** react-frontend.mdc bullets (Theme 1) — merged 2026-07-01
- [x] **Approve** verify-before-inventing credentials row (Theme 2) — merged 2026-07-01
- [x] **ADR-0001** integration credentials — merged 2026-07-01
- [x] **ADR-0002** multitenancy composite indexes — merged 2026-07-01
- [x] **ADR-0003** delayed_job for async — merged 2026-07-01
- [x] **Reviewer tag template** — `.cursor/skills/review-pr/reviewer-comment-template.md`
- [x] **CMDB playbook** — `doc/playbooks/cmdb-schema-changes.md`
- [x] **Keyword bot filter** — `lib/engineering_ai/pr_comment_analyzer.rb`
- [x] **PR template** — AI assistance checkbox + touched-file tests
- [ ] **Schedule CAR audit** on 5 PRs (#37556, #37684, #37685, #37307, #37595)
- [ ] **Next harvest date:** 2026-10-01

---

*Champion sign-off: policy artifacts merged 2026-07-01. CAR audits and Q3 harvest remain open.*

---

## Appendix A — Top 20 PRs by discussion volume

| PR | Title (truncated) | Messages | CHANGES_REQUESTED | Labels |
|----|-------------------|----------|-------------------|--------|
| #37381 | Cost Centers Foundation | 34 | Yes | — |
| #37826 | Cost center april changes | 33 | Yes | — |
| #37307 | Members detail screen | 32 | Yes | — |
| #37518 | CMDB revamp phase 2 | 32 | Yes | — |
| #37520 | ITSM modules in reporting | 32 | Yes | — |
| #37749 | Cost centers depreciation | 31 | Yes | — |
| #37677 | AWS integration p1 | 31 | Yes | AI Comments Pending |
| #37656 | Bulk import warranty dates | 32 | Yes | AI Comments Pending |
| #37505 | Member forms | 29 | Yes | AI Comments Pending |
| #37615 | Active contract fields | 29 | Yes | — |
| #34814 | Vendors Module React | 65 | No | — |
| #37670 | Form identification ticket | 56 | Yes | — |
| #37441 | Custom Fields for Ticket reporting | 56 | Yes | — |
| #36400 | Domains listing react | 53 | No | — |
| #37498 | Software vulnerability matching | 50 | Yes | — |
| #37856 | Webhook JSON schema validation | 44 | Yes | — |
| #37124 | Domains detail react | 39 | No | — |
| #37839 | GPS Skybitz integration | 36 | No | — |
| #37672 | Slack webhook handler | — | Yes | — |
| #37595 | Work order submit rules | — | Yes | — |
