# Playbook: Create an ApplicationService

Use when adding new business logic that spans models, external APIs, or orchestration — **after** confirming no existing service fits.

## When to use

- New cross-model workflow, integration, or command object
- Logic does not belong in a controller or fat model callback
- Ticket or Plan mode suggests "create a service"

## When NOT to use

- Simple CRUD already handled by model + controller
- An existing service can be extended (preferred)
- One-off rake task (use `lib/tasks/` with justification)

## Before you code

- [ ] **Search first:** `rg "SimilarName" app/services/` and grep for related domain words
- [ ] Check golden examples below
- [ ] Read `.cursor/rules/services-and-jobs.mdc` and `verify-before-inventing.mdc`
- [ ] Confirm async needs: `delayed_job` via `.delay(queue: DELAYED_JOB_QUEUES[:...])` — not Sidekiq

## Steps

### 1. Reuse decision

Document in Plan or PR:

| Question | Answer |
|----------|--------|
| Existing service found? | Yes: `_______` / No |
| Why extend vs new? | |
| Rule of Three: is this the 3rd similar case? | |

### 2. Scaffold

```ruby
# app/services/my_domain/my_action_service.rb
class MyDomain::MyActionService < ApplicationService
  def initialize(company:, user:, params: {})
    @company = company
    @user = user
    @params = params
  end

  def call
    # orchestration here
  end
end

# Entry point:
# MyDomain::MyActionService.call(company: company, user: user, params: params)
```

Adjust namespace to match existing `app/services/` patterns — **grep siblings first**.

### 3. Multitenancy & products

- [ ] All queries scoped to `company` / `company_id`
- [ ] `IS_RENTALS`, `IS_ASSET_SONAR`, etc. guarded explicitly — no assumed single product
- [ ] Authorization checked (`Ability` / permissions concerns) before mutations

### 4. Errors & monitoring

- [ ] Use `handle_exceptions` if sibling services do (read neighbors)
- [ ] `CodeMonitor::Trackable` if pattern exists in similar services
- [ ] Fail loudly in dev; graceful user-facing errors in prod

### 5. Tests

- [ ] RSpec in `spec/services/` (preferred for new tests)
- [ ] Set `Company.current_tenant = company` in specs

### 6. Async (if needed)

```ruby
MyDomain::MyActionService.delay(queue: DELAYED_JOB_QUEUES[:appropriate_queue]).call(...)
```

Grep `DELAYED_JOB_QUEUES` in `config/initializers/app_constants.rb` for valid queue names.

## PR checklist

| Check | |
|-------|---|
| No duplicate of existing service | |
| `ApplicationService` subclass | |
| `.call` entry point | |
| Multitenancy | |
| Product flags | |
| Specs added | |

## Golden references

Search and read these before inventing patterns:

```text
app/services/application_service.rb
app/services/          # grep for domain-related siblings
spec/services/
```

Examples to locate via search (names may vary):

- Export/report services
- Sync services under product-specific namespaces (`Ticketing::`, `AssetSonar/`, etc.)

## Related

- [starting-from-redmine-ticket.md](starting-from-redmine-ticket.md)
- `AGENTS.md` — Service Objects section
- ADR (when exists): async job choice — `doc/adr/`

---

*Add real service paths from your domain after PR harvest identifies patterns.*
