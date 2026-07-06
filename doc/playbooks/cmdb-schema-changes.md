# Playbook: CMDB Schema Changes

Use when PRs touch CMDB relationship tables, migrations under `cmdb_*`, or graph/relationship serializers.

**Source:** PR harvest 2026-07-01 — recurring deferred schema debates on relationship indexes and column redundancy.

## When to use

- New/changed tables: `cmdb_configuration_item*`, relationship types, relationship edges
- Migrations adding indexes on CMDB join tables
- Removing `source_type` / `target_type` or similar STI columns

## Before you merge

- [ ] **Architect review required** for relationship table shape and composite indexes
- [ ] Apply label `Ready for Architect Review` on PR when schema is non-trivial
- [ ] Composite indexes **lead with `company_id`** — `EXPLAIN` on representative queries
- [ ] Discuss redundant columns vs `cmdb_configuration_item_relationship_type_id` with architect — don't assume in PR thread alone

## Migration checklist

| Check | |
|-------|---|
| `company_id` on tenant-scoped tables | |
| Composite index: `(company_id, …)` matches query patterns | |
| No cross-tenant FK leaks | |
| Product flags if CMDB feature is product-specific | |
| Rollback / data migration plan for existing rows | |

## Reviewer prompts

Use structured comments ([reviewer-comment-template.md](../../.cursor/skills/review-pr/reviewer-comment-template.md)):

```markdown
🟠 **SQL**: Add composite index on (company_id, cmdb_configuration_item_relationship_type_id, source_id) — discuss table design with architect before merge.
```

## Escalation

If author and reviewer disagree on schema shape:

1. Do **not** merge on thread consensus alone for new relationship tables
2. Document decision in `doc/adr/` if pattern is reusable
3. Link Redmine / follow-up ticket if deferred

## Golden references

- Rules: `.cursor/rules/database-migrations.mdc`
- Services: `app/services/cmdb/`
- Specs: `spec/services/cmdb/`, `spec/controllers/cmdb/`

---

*Maintainer: architects (CMDB domain).*
