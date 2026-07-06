# EZOfficeInventory — Agent Steering Guide

This document orients AI coding assistants working in this repository.

**Before making changes:** read applicable `.cursor/rules/` (auto-loaded) and search the codebase — do not invent patterns, classes, or file paths.

| Need | Canonical source |
|------|------------------|
| Full rules index + precedence | [.cursor/README.md](.cursor/README.md) |
| Task workflows | [doc/playbooks/README.md](doc/playbooks/README.md) |
| Architecture decisions | [doc/adr/](doc/adr/) |
| Domain terms (Redmine → code) | [doc/domain-glossary.md](doc/domain-glossary.md) |
| Engineering AI hub | [doc/README-engineering-ai.md](doc/README-engineering-ai.md) |
| Strategy & phases | [doc/ENGINEERING-AI-STRATEGY.md](doc/ENGINEERING-AI-STRATEGY.md) |

## What This Codebase Is

A **Rails 6.1 monolith** (Ruby 3.1.4) powering four SaaS products from one repo:

| Product | `APPLICATION_NAME` | Key flag |
|---------|-------------------|----------|
| EZOffice Inventory | `"EZOffice"` | `IS_EZ_OFFICE_NOT_CMMS` |
| EZO CMMS | `"CMMS"` | `IS_CMMS` |
| AssetSonar | `"Asset Sonar"` | `IS_ASSET_SONAR` |
| EZRentOut | `"EZRentOut"` | `IS_RENTALS` |

Each **deployment** is built for one product. Product identity is set at compile/deploy time via shell scripts (`ezoffice.sh`, `assetsonar.sh`, `ezrentals.sh`, `cmms.sh`) that copy product-specific configs.

**Scale:** ~1,100 models, ~375 controllers, ~440 services, ~1,700 React/TS files.

---

## Tech Stack (Verified)

| Layer | Technology |
|-------|------------|
| Language | Ruby 3.1.4 |
| Framework | Rails 6.1.7.3 |
| Database | MySQL (`mysql2` gem) |
| ORM | ActiveRecord with custom multitenancy |
| Auth | Devise, devise-jwt, devise-two-factor, Doorkeeper + OIDC |
| Authorization | CanCanCan (`Ability` + `*_ability` concerns) |
| Background jobs | **delayed_job** (primary), ActiveJob (secondary, ~35 jobs) |
| Search | Elasticsearch (`elasticsearch-model`), SearchCop, Ransack |
| API serialization | active_model_serializers (JSON:API adapter) |
| Soft delete | Paranoia (`acts_as_paranoid`, selective) |
| Web UI | ERB (`app/views/zen/`), Slim (~137 files, mostly rentals) |
| Legacy JS | Webpacker 5 (`app/javascript/packs/`) |
| Modern UI | React + TypeScript (`app/react_web/`, esbuild) |
| CSS | Sass 3.4 (`public/zen/stylesheets/`) |
| Testing | RSpec (primary, `spec/`), Minitest (legacy, `test/`) |
| Factories | factory_girl_rails (`spec/factories/`) |
| Deployment | Rubber/Capistrano (`config/rubber/`) |
| Monitoring | Airbrake, Datadog (`ddtrace`), CodeMonitor plugin |
| Engines | `workflow_automator` (path gem in `engines/`) |
| Plugins | `code_monitor`, `ezo_task_migration` (dev only) |

**Not used:** Sidekiq, Redis queues, Hotwire/Turbo, Stimulus as primary patterns.

---

## Directory Map

```
app/
  models/           # ActiveRecord + concerns/ (~389 concern files)
  controllers/      # Web + api/v2/ REST API
  services/         # ApplicationService subclasses
  serializers/      # web/, mobile/, api/v2/serializers/
  jobs/             # ActiveJob (secondary to delayed_job)
  views/zen/        # Primary server-rendered UI (ERB)
  javascript/       # Webpacker packs
  react_web/        # React SPA (TypeScript, esbuild)
config/
  initializers/     # app_constants.rb, app_aname.rb, integrations
  routes/           # Split route files (ez_api_v2.rb, cmdb.rb, etc.)
  *.yml.{ezoffice,assetsonar,ezrentals,cmms}  # Product-specific configs
engines/workflow_automator/
plugins/code_monitor/
lib/tasks/          # ~172 rake tasks
spec/               # RSpec (primary tests)
```

---

## Critical Architecture Rules

### 1. Multi-Product Conditionals

Always check existing `IS_*` flags before adding product-specific code:

```ruby
# config/initializers/app_constants.rb (derived from APPLICATION_NAME)
IS_RENTALS, IS_EZ_OFFICE, IS_CMMS, IS_EZ_OFFICE_NOT_CMMS, IS_ASSET_SONAR
```

Grep sibling products for the pattern before inventing a new one.

### 2. Multi-Tenancy (Company Scoping)

- Tenant = `Company`, resolved per request via subdomain (`Company.current_tenant`)
- `ApplicationRecord` auto-adds `default_scope { where(company_id: Company.current_tenant_id) }`
- Models without `company_id` must call `not_multitenant!` (e.g. `Company` itself)
- Dev/test **raises `MultiTenancyError`** if a new model lacks `company_id`
- Delayed jobs stamp `company_id` via `config/initializers/delayed_job_tenant.rb`

### 3. Domain Model Hierarchy

**Assets (STI on `assets.type`):**
```
Asset → FixedAsset | VolatileAsset → StockAsset → SoftwareLicense (AssetSonar)
Asset → ServiceOffering (EZRentOut)
```

**Users:** `Customer < User` (STI). Staff are `User` with `role_id` / `CustomRole`.

**Rentals vs Inventory:**
- EZOffice/AssetSonar: checkout/checkin to **members** (`User`), reservations, audits
- EZRentOut: **Baskets** to **customers** — see [.cursor/rules/ezrentout-baskets.mdc](.cursor/rules/ezrentout-baskets.mdc)

**Product-specific depth (when editing those paths):**

| Domain | Rule / skill |
|--------|----------------|
| AssetSonar ITAM, MDM, CMDB, cloud inventory | [.cursor/rules/assetsonar-itam.mdc](.cursor/rules/assetsonar-itam.mdc) · skills: `mdm-integration`, `cmdb-add-ci`, `cloud-inventory-provider` |
| EZRentOut baskets, pricing, web store | [.cursor/rules/ezrentout-baskets.mdc](.cursor/rules/ezrentout-baskets.mdc) |
| Custom role permissions | [.cursor/rules/custom-role-permissions.mdc](.cursor/rules/custom-role-permissions.mdc) · [.cursor/skills/custom-role-permissions/SKILL.md](.cursor/skills/custom-role-permissions/SKILL.md) |

### 4. Patterns (see rules — do not duplicate here)

| Topic | Canonical source |
|-------|------------------|
| Services & async | [.cursor/rules/services-and-jobs.mdc](.cursor/rules/services-and-jobs.mdc) · [ADR-0003](doc/adr/0003-delayed-job-for-async-work.md) · [create-application-service.md](doc/playbooks/create-application-service.md) |
| Authorization | `app/models/ability.rb` + `*_ability` / `*_permissions` concerns · custom roles: [custom-role-permissions skill](.cursor/skills/custom-role-permissions/SKILL.md) |
| API v2 serializers | [.cursor/rules/api-serializers.mdc](.cursor/rules/api-serializers.mdc) |
| Search | `Indices::*Index` concerns, SearchCop, Ransack — grep siblings before adding |

---

## Frontend Stacks

| Stack | When to use | Rule / doc |
|-------|-------------|------------|
| React SPA | New interactive screens | `app/react_web/src/` · [.cursor/rules/react-frontend.mdc](.cursor/rules/react-frontend.mdc) |
| Zen ERB | Server-rendered pages | `app/views/zen/` |
| Webpacker | Legacy JS packs | `app/javascript/packs/` |
| Slim | Rentals web store, mailers | `app/views/web_store/`, `app/views/baskets/` |

React playbooks: [react-and-api-contracts.md](doc/playbooks/react-and-api-contracts.md) · skills: `react-product-ui`, `react-new-module`, `react-listing-filters`

---

## Testing Conventions

- **Write new tests in RSpec** (`spec/`) — [.cursor/rules/testing-rspec.mdc](.cursor/rules/testing-rspec.mdc)
- Product-specific specs: `spec/ez_office/`, `spec/rentals/`, `spec/asset_sonar/`
- Set tenant in tests: `Company.current_tenant = company`

---

## Product Config Changes

Update **all** product variants (`*.yml.{ezoffice,assetsonar,ezrentals,cmms}`); deploy scripts copy the active file. See [.cursor/rules/senior-engineering.mdc](.cursor/rules/senior-engineering.mdc).

---

## Anti-Hallucination Checklist

**Canonical table:** [.cursor/rules/verify-before-inventing.mdc](.cursor/rules/verify-before-inventing.mdc)

Before creating or referencing code:

1. **Class/method exists** — grep and read the real file
2. **Product flags** — check `IS_*` on similar code
3. **Tenant scoping** — `company_id` or `not_multitenant!`
4. **Async** — `delayed_job` + `DELAYED_JOB_QUEUES` (see ADR-0003)
5. **Reuse** — search `app/services/` and `app/models/concerns/` before creating new abstractions

## Engineering Principles

See [.cursor/rules/engineering-principles.mdc](.cursor/rules/engineering-principles.mdc) — do not duplicate here.

---

## Key Files to Read First

| Topic | File |
|-------|------|
| Product flags | `config/initializers/app_constants.rb` |
| Product name | `config/initializers/app_aname.rb` |
| Multitenancy | `app/models/application_record.rb` |
| Base controller | `app/controllers/application_controller.rb` |
| Authorization | `app/models/ability.rb` |
| Service base | `app/services/application_service.rb` |
| API v2 base | `app/controllers/api/v2/base_controller.rb` |
| Delayed job config | `config/initializers/delayed_job.rb` |
| Routes | `config/routes.rb`, `config/routes/ez_api_v2.rb` |
| Workflow engine | `engines/workflow_automator/` |
| ADRs (architecture decisions) | `doc/adr/` — credentials (0001), indexes (0002), delayed_job (0003) |
| Playbooks | `doc/playbooks/` — React/API, CMDB, Redmine, services |
| Domain glossary | `doc/domain-glossary.md` |
| PR harvest workflow | `.cursor/skills/harvest-pr-insights/SKILL.md` |
| Agentic workflows | `grill-plan`, `review-pr` (`/review`), `ultra-review`, `receiving-code-review` — [agentic-workflows.md](doc/playbooks/agentic-workflows.md) |

---

## Cursor Rules

Index and precedence: [.cursor/README.md](.cursor/README.md). Rules auto-apply from `.cursor/rules/*.mdc` — do not duplicate full rule text in this file.
