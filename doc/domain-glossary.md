---
title: "Domain Glossary"
purpose: "Map product / Redmine language → code symbols for engineers and AI assistants"
status: "Living document — grow via PR when terms confuse reviews or tickets"
related: "doc/playbooks/starting-from-redmine-ticket.md"
---

# Domain Glossary

Redmine tickets, customer docs, and sales language often differ from code identifiers. Before implementing from a ticket, **look up terms here** and load the code symbols into your Plan mode / assistant context.

## How to use

1. Scan the Redmine ticket for domain terms (nouns, feature names)
2. Find each term in the tables below
3. In Cursor Plan mode, include: *"Use code symbol X, not Y; see doc/domain-glossary.md"*
4. When you discover a new mismatch, add a row via PR (champion or tech lead review)

## How to maintain

| Trigger | Action |
|---------|--------|
| Same term confuses two PRs | Add or clarify row |
| New product-facing feature named in Redmine | Add before merge |
| Code rename | Update symbol column; note old name in Notes |

**Growth target:** Expand when harvest or reviews surface ticket ↔ code mismatches (see [ENGINEERING-AI-STRATEGY.md](ENGINEERING-AI-STRATEGY.md) Phase 3).

---

## Core terms (seed — expand via PR)

| Product language (Redmine / docs) | Code symbol(s) | Type | Products | Notes |
|-----------------------------------|----------------|------|----------|-------|
| Work Order | `Task` | Model (CMMS) | CMMS, EZOffice | Not `WorkOrder` |
| Custom Field | `cust_attr`, custom attributes | Pattern | All | See custom attribute models/concerns |
| Basket / Order | `Basket`, `BasketsAsset` | Models | EZRentOut (`IS_RENTALS`) | Rentals checkout flow |
| Customer | `Customer` (STI `< User`) | Model | EZRentOut | Staff are `User`, not `Member` |
| Member / Staff | `User` | Model | EZOffice, AssetSonar, CMMS | No `Member` model for staff |
| Member (React UI) | `Member` interface in `core/models/data/member/` | TypeScript | EZOffice, AssetSonar, CMMS | UI label "member" ≠ Rails model; **never** `type Member = any` |
| Cost Center | `CostCenter`, cost center concerns | Model/Feature | EZOffice (`enable_cost_centers`) | Guard with `IS_EZ_OFFICE_NOT_CMMS`; not all products |
| Asset | `Asset`, `FixedAsset`, `StockAsset`, … | STI | All | Check `assets.type` |
| Software License | `SoftwareLicense` | Model | AssetSonar | Extends stock asset path |
| Company / Tenant | `Company`, `company_id` | Multitenancy | All | `Company.current_tenant` per request |
| Service (business logic) | `ApplicationService` subclasses in `app/services/` | Pattern | All | Search before creating new service |
| Background job | `delayed_job` (`.delay`) | Pattern | All | Not Sidekiq; see `DELAYED_JOB_QUEUES` |

---

## Product-specific terms

### EZOffice / CMMS

| Product language | Code symbol(s) | Notes |
|------------------|----------------|-------|
| `[TBD]` | | |

### AssetSonar

| Product language | Code symbol(s) | Notes |
|------------------|----------------|-------|
| ITAM / Hardware sync | `ItamHardware`, `ItamSyncService`, … | `ItamHardware` is `not_multitenant!` |

### CMDB (AssetSonar / ITSM)

| Product language | Code symbol(s) | Notes |
|------------------|----------------|-------|
| Configuration Item (CI) | CMDB models under `cmdb_*`, graph tab React | Schema changes need architect review — `doc/playbooks/cmdb-schema-changes.md` |
| Relationship type | `CmdbConfigurationItemRelationshipType` | Index with `company_id` first |

### EZRentOut

| Product language | Code symbol(s) | Notes |
|------------------|----------------|-------|
| Web Store | `web_store` routes, controllers | |
| `[TBD]` | | |

---

## Ambiguous terms (disambiguation)

| Term | Meaning A | Meaning B | How to decide |
|------|-----------|-----------|---------------|
| License | Software license (AssetSonar) | Rental license / agreement | Product flag + ticket product |
| `[TBD]` | | | |

---

## Changelog

| Date | Author | Change |
|------|--------|--------|
| 2026-07-01 | PR harvest | Member (React), Cost Center, CMDB rows |

---

*Part of Engineering AI Phase 3 — [ENGINEERING-AI-STRATEGY.md](ENGINEERING-AI-STRATEGY.md).*
