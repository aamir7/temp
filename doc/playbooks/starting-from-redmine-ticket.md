# Playbook: Starting from a Redmine Ticket

Use this when implementation work begins from a Redmine issue — before writing code.

## When to use

- New feature, enhancement, or bug fix tracked in Redmine
- Ticket description uses product language that may not match code symbols
- You will use Cursor Plan mode or another AI assistant

**Recommended:** Run `/grill-plan` when scope is ambiguous, crosses React+API+permissions, or involves multitenancy/product flags — before Plan mode or implementation.

## Before you code

- [ ] Read the full Redmine ticket (description, comments, acceptance criteria)
- [ ] Optional: `/grill-plan` for Socratic stress-test → approved plan block
- [ ] Open [domain glossary](../domain-glossary.md) — map product terms → code symbols
- [ ] Search the codebase for existing implementations (grep / Cursor search)
- [ ] Confirm product scope: which `IS_*` flag applies? (see `AGENTS.md`)
- [ ] Load relevant `.cursor/rules/` (multitenancy, product-specific)

## Steps

### 1. Capture ticket context

Copy into Plan mode (or assistant chat):

```markdown
## Redmine
- ID: #_______
- URL: [link]
- Summary: [one line]

## Acceptance criteria
- [paste from ticket]

## Domain terms (from glossary)
- "Work Order" → Task
- [add rows]

## Product / flag
- Deployment product: EZOffice | CMMS | AssetSonar | EZRentOut
- Relevant flags: IS_______
- UI visibility: which products see this screen/API? (required for React work)

## Search before create
- Similar code found: [paths or "none yet"]
- Reuse plan: extend X / create new because Y
```

### 2. Plan before implement

- [ ] Use **`/grill-plan`** when scope is ambiguous or high-risk (see [grill-plan/SKILL.md](../../.cursor/skills/grill-plan/SKILL.md))
- [ ] Use **Plan mode** for multi-file or non-trivial scope
- [ ] Plan must reference reuse search results
- [ ] Plan must note multitenancy (`company_id`) and authorization if touching models
- [ ] React UI: `/react-product-ui` before `/react-new-module` when pattern unclear

### 3. Implement

- Follow [create-application-service.md](create-application-service.md) if adding a service
- Follow [react-and-api-contracts.md](react-and-api-contracts.md) if touching `app/react_web/`
- Follow [cmdb-schema-changes.md](cmdb-schema-changes.md) if touching CMDB migrations
- Follow `.cursor/rules/` for the files you touch
- Prefer extending existing patterns over new abstractions

### 4. PR

- [ ] Redmine link in PR description
- [ ] AI assistance section in [.github/pull_request_template.md](../../.github/pull_request_template.md)
- [ ] Tests for behavior you changed or added

## Checklist (reviewer)

| Check | |
|-------|---|
| Ticket scope fully addressed | |
| Product flag guards correct | |
| Multitenancy respected | |
| No duplicate service/concern when reuse existed | |
| Glossary updated if new term mismatch found | |

## Golden references

- Multitenancy: `app/models/application_record.rb`, `.cursor/rules/multi-product-tenancy.mdc`
- Product flags: `config/initializers/app_constants.rb`
- Services: `app/services/application_service.rb`

## Automation (future)

- Redmine API / MCP fetch ticket into Plan context
- PR template auto-links ticket ID
