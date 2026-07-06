---
title: "Repository Complexity Baseline — Q2 2026"
purpose: "Week 0 snapshot for Engineering AI upskilling — project size, complexity, AI confusion points, PR review patterns"
captured_at: "2026-06-30"
repo: "7Vals/ezofficeinventory"
related:
  - doc/engineering-ai-metrics/README.md
  - doc/engineering-ai-metrics/baseline-template.csv
  - AGENTS.md
  - .cursor/rules/verify-before-inventing.mdc
---

# Repository Complexity Baseline — Q2 2026

Week 0 snapshot for the Engineering AI upskilling initiative. Use with [baseline-template.csv](baseline-template.csv) KPI tracking and team surveys (frictions, golden modules, utilization).

**Machine-readable PR metrics (git):** [pr-metrics-git.json](pr-metrics-git.json)

---

## Executive summary

| Dimension | Scale | Implication for AI upskilling |
|-----------|-------|------------------------------|
| Backend | ~1,200 models, ~500 controllers, ~1,280 services, ~525K Ruby LOC | Search-first mandatory; agents invent patterns without rules |
| Frontend | ~2,100 TSX files, ~624K LOC, ~285 Redux slices | Clearer module boundaries than Rails; still high coupling via shared converters |
| Tests | 383 spec files, ~11K examples, **0 feature specs**, ~26% of PRs touch specs | AI verification loops must assume **low automated coverage** |
| PR velocity | ~335 merges / 90 days (~3.7/day) | Review load is high; agentic review (`/review-pr`) has ROI |
| Hot churn | React, `company.rb`, workflows, CMDB, ITAM sync | Training and rules should cover these first |
| God objects | `basket.rb` (13K LOC), `company.rb` (11K LOC) | **Not** golden modules — advanced/caution only |

---

## 1. Rails backend

| Metric | Count | LOC |
|--------|------:|----:|
| Models (excl. concerns) | 1,210 | 187,344 |
| Model concerns | 386 | 35,147 |
| Controllers (`*_controller.rb`) | 505 | 115,818 |
| Controller concerns | 167 | 26,686 |
| Services | 1,284 | 157,879 |
| Jobs (ActiveJob, `app/jobs`) | 35 | 1,789 |
| delayed_job (`.delay(` calls) | 203 in ~102 files | — |
| Serializers (all) | 329 | — |
| API v2 serializers | 49 | — |
| API v2 controllers | 40 | — |
| Presenters | 140 | — |
| Helpers | 217 | — |
| Migrations | 6,114 | — |
| Rake tasks | 173 | — |
| Initializers | 107 | — |
| Lib (`lib/**/*.rb`) | 153 | 25,814 |

**Engine:** `workflow_automator` — 22 models, 116 services (+ 53 engine specs).

**Views:** 9,382 ERB (435K LOC) · 139 Slim (6K LOC)

**Total Ruby in `app/`:** ~525K LOC

---

## 2. React frontend (`app/react_web/src`)

| Metric | Count |
|--------|------:|
| Total TS/TSX files | 4,178 |
| TSX components | 2,104 |
| Total LOC | 623,577 |
| Screens | 166 |
| Containers | 1,095 |
| Design-system components | 240 |
| Hooks (in `hooks/`) | 178 |
| Custom hooks (`use*`) | 107 |
| Form-data hooks | 84 |
| Redux store files | 577 |
| Redux slices | 285 |
| Redux actions | 285 |

**Largest React modules:** items (384 files), workflow (158), tasks (112), softwareLicenses (108).

**Legacy JS:** 37 Webpacker packs.

---

## 3. Test landscape

| Metric | Count |
|--------|------:|
| RSpec files (spec + engines) | 383 |
| RSpec LOC | 184,298 |
| `it` examples (approx.) | 11,113 |
| Minitest (`test/`) | 48 |
| Feature specs | **0** |
| Request specs | 2 |
| Controller specs | 20 |
| Service specs | 34 |
| Pending/skip markers | ~25 |

**By product area:** EZRentOut 91 · Shared 85 · EZOffice 39 · AssetSonar 31 · engine 53

**Strongest spec clusters:** EZRentOut pricing/baskets (~63 files), CMDB, workflow_automator.

**Estimated file-level spec coverage:** ~32% of model files have a corresponding spec file pattern (heuristic).

---

## 4. Complexity hotspots

### Largest Ruby files (god objects)

| LOC | File |
|----:|------|
| 13,163 | `app/models/basket.rb` |
| 11,217 | `app/models/company.rb` |
| 8,891 | `app/controllers/companies_controller.rb` |
| 8,422 | `app/controllers/baskets_controller.rb` |
| 6,428 | `app/models/user.rb` |
| 5,846 | `app/models/fixed_asset.rb` |
| 5,574 | `app/models/software_license.rb` |
| 5,480 | `app/models/asset.rb` |

### Git churn (last 12 months)

| Area | File touches |
|------|-------------:|
| `app/react_web` | 19,079 |
| `app/models` | 6,554 |
| `app/views` | 5,289 |
| `app/services` | 4,884 |
| `app/controllers` | 4,273 |
| `engines/workflow_automator` | 2,587 |

**Most modified models:** `company`, `user`, `fixed_asset`, `basket`, `asset`, `ability`, `software_license`, `task`

**Most modified services:** `itam_sync_service`, `automation_params_converter`, `software_vulnerability_service`, `workflows/plan_executor`, ticketing/GLPI

---

## 5. AI confusion points

Patterns where coding assistants **reliably guess wrong** without repo context. Encoded in `AGENTS.md`, `.cursor/rules/verify-before-inventing.mdc`, and `doc/domain-glossary.md`.

### 5.1 Architecture & stack (hallucination table)

| Wrong assumption | Reality | Risk if missed |
|-----------------|---------|----------------|
| Sidekiq / `perform_async` for async | `delayed_job` via `.delay(queue: DELAYED_JOB_QUEUES[:...])` | Wrong job infrastructure, deploy failure |
| `Member` model for staff | `User` with `role_id` / `CustomRole` | Broken authorization, wrong queries |
| Single product deployment | Four products; `IS_RENTALS`, `IS_EZ_OFFICE`, `IS_CMMS`, `IS_ASSET_SONAR` | Wrong feature shipped to wrong product |
| Generic `ApplicationJob` for all async | ~35 ActiveJob classes; **most** async is delayed_job | Misplaced background work |
| Rails 7 patterns | Rails **6.1.7.3** | API incompatibility |
| One config file | `*.yml.{ezoffice,assetsonar,ezrentals,cmms}` variants | Product config drift |
| `type` column = STI always | Some models set `inheritance_column = nil` | Wrong model lookups |
| Paranoia everywhere | Selective; assets use `DeletedAsset` | Wrong delete behavior |

### 5.2 Domain terminology (Redmine ↔ code)

| Product / ticket language | Code symbol | Notes |
|---------------------------|-------------|-------|
| Member / Staff | `User` | Not `Member` |
| Customer (rentals) | `Customer < User` (STI) | Staff ≠ customer |
| Work Order (CMMS) | `Task` | Not `WorkOrder` |
| Basket line item | `BasketsAsset` | Not `BasketLineItem` |
| Basket checkout | `checkout_cart!` / `reserve_cart!` | Do not mutate `basket_state` directly |
| Software license | `SoftwareLicense` | AssetSonar; extends stock asset path |
| ITAM hardware | `ItamHardware` | `not_multitenant!`; tenant via `Asset.itam_hardware_id` |
| Tenant | `Company`, `company_id` | `Company.current_tenant` per request |
| Business logic | `ApplicationService` subclasses | Search `app/services/` before creating |

### 5.3 Structural confusion (size & coupling)

| Area | Why AI struggles | Mitigation |
|------|------------------|------------|
| `basket.rb`, `company.rb`, `companies_controller.rb` | 8K–13K LOC; implicit coupling | Bounded scope; denylist for Agent; human review required |
| `ability.rb` + 386 concerns | Authorization spread across concerns | Extend existing `*_ability` modules; never rewrite `Ability` wholesale |
| Asset STI (`FixedAsset`, `StockAsset`, `VolatileAsset`, …) | Type-specific behavior | Read parent + STI sibling before editing |
| Pricing engines (EZRentOut) | Multiple versions (v2/v3), heavy specs | Use existing spec files as ground truth |
| Workflow engine | Isolated but complex (`workflow_automator`) | Follow engine patterns; 53 specs available |
| React `modelData.converter.ts`, `strings.ts` | High churn shared files | Touch only required keys; avoid drive-by refactors |

### 5.4 Process confusion

| Mistake | Reality |
|---------|---------|
| "CI will catch it" | CI does not catch cross-module coupling, reuse violations, or wrong `IS_*` guards |
| "Tests will fail if wrong" | ~10% coverage; most domains have no safety net |
| "Create `UserService`" | Domain logic lives in models, concerns, and `ApplicationService` — grep first |
| Invent new directory layout | Follow `app/services/`, `app/models/concerns/`, `app/serializers/api/v2/serializers/` |

### 5.5 Flaky / untested areas (heuristic)

No `rspec-retry` or quarantine markers in repo. Likely high-risk zones:

- CMDB graph and relationship specs (actively changing)
- `workflow_automator` orchestrator / execution store
- ITAM sync integrations (`itam_sync_service`, MDM services)
- EZRentOut pricing engine specs (complex state)
- Bulk i18n PRs (hundreds of files — easy to miss functional regressions)

**Action:** Supplement with CI flake history and team survey.

---

## 6. Suggested golden modules (for AI training)

| Tier | Module | Rationale |
|------|--------|-----------|
| **Starter** | `spec/EZRentout/models/pricing_engine_v3/*` | Rich specs, bounded domain |
| **Starter** | `spec/EZRentout/models/basket_*` | State machine examples |
| **Starter** | React tasks/vendors modules | Moderate size, clear slice/hook pattern |
| **Starter** | `app/services/cmdb/*` + growing specs | Active, isolated domain |
| **Intermediate** | `engines/workflow_automator` | Engine boundary + 53 specs |
| **Caution** | `basket.rb`, `company.rb`, `companies_controller.rb` | God objects |
| **Caution** | `itam_sync_service`, external API services | Integration + retries |

Validate with team survey before formalizing.

---

## 7. GitHub PR review analysis (last 90 days)

**Source:** Git merge commits on `main` (local git analysis).  
**Period:** ~2026-04-01 → 2026-06-30 · **335 merged PRs**

> **GitHub API metrics (comments, review rounds, approval time):** Not captured in this run — `gh` CLI requires `gh auth login`. See [§7.5](#75-refresh-commands) to complete.

### 7.1 Files changed

| Metric | All PRs | Excluding bulk i18n PRs* |
|--------|--------:|-------------------------:|
| Median files | 40 | **11** |
| P75 | 274 | 35 |
| P90 | 555 | 73 |
| Max | 2,036 | 165 |
| PRs > 20 files | 197 (59%) | — |
| PRs > 50 files | 154 (46%) | — |

\*Bulk i18n = touches `localizedStrings.ts` / `strings.ts` with >100 files.

**Interpretation:** Raw file counts are skewed by i18n/string migration PRs. **Typical functional PR ≈ 11 files median.**

### 7.2 Cycle time (first branch commit → merge)

| Metric | Hours |
|--------|------:|
| Median | 24.5 |
| P75 | 161 |
| P90 | 383 |
| < 24h | 167 PRs (50%) |
| 1–3 days | 46 |
| 3–7 days | 47 |
| > 7 days | 75 |

### 7.3 By branch prefix

| Type | Count | Median files | Median cycle (h) |
|------|------:|-------------:|-----------------:|
| hotfix | 161 | 15 | 17 |
| feature | 159 | 84 | 137 |
| support | 3 | — | — |
| other | 12 | — | — |

**Observation:** Hotfixes are small and fast; features are **5× larger** and **8× longer** in cycle time.

### 7.4 What PRs touch (path frequency)

| % of PRs | Area |
|----------|------|
| 82% | `app/models` |
| 72% | `app/react_web` |
| 56% | `db/migrate` |
| 26% | `*_spec.rb` |

**Hot paths (aggregate file touches across all PR diffs):** `app/react_web` → `app/models` → `app/services` → `app/views` → `app/controllers` → `db/migrate` → `engines/workflow_automator`

### 7.5 Refresh commands

```bash
# Git-based metrics (no auth required)
rake engineering_ai:pr_metrics_from_git DAYS=90

# GitHub review metrics (comments, rounds, open→merge time)
gh auth login
rake engineering_ai:pr_review_metrics DAYS=90
# → writes doc/engineering-ai-metrics/pr-review-metrics.json

# Quick test (5 PRs only)
rake engineering_ai:pr_review_metrics DAYS=90 SAMPLE=5
```

**Note:** The task uses the **GitHub REST API** (`gh api`), not `gh pr list --json` (GraphQL). The GraphQL endpoint often returns `502 Bad Gateway` on large repos. PR numbers come from local git merge commits; each PR is fetched individually with retries on 502/503/429.

**Runtime:** ~335 PRs × 2 API calls ≈ 3–5 minutes. Use `SAMPLE=20` to validate first.

### 7.6 Review playbook & checklist implications

Mapped to [.github/pull_request_template.md](../../.github/pull_request_template.md) and [.cursor/skills/review-pr/checklist.md](../../.cursor/skills/review-pr/checklist.md).

| Observation | Playbook / checklist action |
|-------------|----------------------------|
| **56% of PRs include migrations** | Enforce `company_id` composite indexes; migration review gate in `/review-pr` |
| **Only 26% touch specs** | AI upskilling: "touched-file spec" norm for non-trivial changes |
| **Features median 84 files** | Plan mode required; scope limits in Agent prompts |
| **Hotfixes median 15 files, 17h** | Lighter checklist OK; still check `IS_*` and multitenancy |
| **React in 72% of PRs** | UI/UX section of PR template applies widely |
| **Bulk i18n PRs distort metrics** | Exclude from rework/lead-time KPIs or tag in GitHub labels |
| **No AI-use field in PR template yet** | Add optional "AI assistance" checkbox for CAR sampling (KPI-4) |
| **Recurring hot areas** (workflows, CMDB, ITAM, company settings) | Prioritize rules harvest from review comments in these paths |

### 7.7 GitHub API metrics

Run after `gh auth login`:

```bash
rake engineering_ai:pr_review_metrics DAYS=90
```

Captures: comments per PR, review submissions, `CHANGES_REQUESTED` rate, open→merge time, time to first review.

If you previously saw `gh pr list failed: HTTP 502` — update to latest `lib/tasks/engineering_ai_metrics.rake` (REST-based fetch).

---

## 8. Repo activity context

| Metric | Value |
|--------|------:|
| Total commits | ~200,250 |
| Commits (12 months) | ~19,557 |
| Contributors (12 months) | ~68 |

---

## 9. Regenerate this baseline

```bash
# Counts only
rake engineering_ai:repository_complexity

# PR git metrics
rake engineering_ai:pr_metrics_from_git DAYS=90 OUT=doc/engineering-ai-metrics/pr-metrics-git.json

# Full GitHub review analysis (after gh auth)
rake engineering_ai:pr_review_metrics DAYS=90
```

Rake tasks: `lib/tasks/engineering_ai_metrics.rake`

---

## 10. Next captures (survey + CI)

| Item | Source |
|------|--------|
| Flaky test list | CI/Jenkins build history |
| Golden modules (validated) | EM + champion survey |
| AI utilization examples | PRs tagged AI-assisted |
| Team frictions | Survey |
| Review comment themes | `pr_review_metrics` rake + manual harvest |
| Coverage % | SimpleCov / CI if available |

---

*Maintainer: Engineering AI champions. Update quarterly or when major architectural shifts occur.*
