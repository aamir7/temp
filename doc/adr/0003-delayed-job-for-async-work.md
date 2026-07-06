---
adr: 0003
title: Use delayed_job for async work
status: accepted
date: 2026-07-01
tags: [async, jobs, architecture]
---

# ADR-0003: Use delayed_job for async work

- **Status:** Accepted
- **Date:** 2026-07-01
- **Deciders:** Engineering architects (platform baseline)
- **Consulted:** Integration leads, ITAM sync owners
- **Tags:** `[async]` `[jobs]` `[architecture]`

## Context

This Rails 6.1 monolith processes background work primarily through **delayed_job** with product-specific queues in `DELAYED_JOB_QUEUES` (`config/initializers/app_constants.rb`). Jobs are enqueued via `.delay(queue: ...)` on models and services.

AI assistants and new engineers frequently assume Sidekiq, Redis queues, or generic ActiveJob patterns from other codebases. That leads to invented `perform_async` calls, wrong queue names, and PR rework.

ActiveJob exists (~35 classes in `app/jobs/`) as a **secondary** pattern — not the default for new async orchestration.

## Decision

We will **use delayed_job with named `DELAYED_JOB_QUEUES` for new background work** unless extending an existing ActiveJob class in `app/jobs/`.

We will **not** introduce Sidekiq, Redis-backed job queues, or parallel async frameworks in this monolith.

## Options considered

| Option | Pros | Cons |
|--------|------|------|
| **A — delayed_job (chosen / status quo)** | Production-proven; tenant stamping; Rubber/Capistrano ops | Less familiar to engineers from Sidekiq shops |
| **B — Sidekiq + Redis** | Popular ecosystem | New infra; duplicate patterns; rejected at platform level |
| **C — ActiveJob for all new work** | Rails-standard API | Minority pattern here; most code uses `.delay` |

## Consequences

### Positive

- Single async mental model for humans and AI
- Queue names grep-able in `app_constants.rb`
- ITAM sync guards (`ItamSyncStatus`) align with delayed_job enqueue patterns

### Negative / trade-offs

- Engineers must learn `.delay` and queue constants
- ActiveJob remains for legacy/integration jobs — two patterns, but hierarchy is clear

### Neutral / follow-ups

- Reinforce in onboarding and `verify-before-inventing.mdc` anti-hallucination table
- `/review-pr` hard stop on Sidekiq references in new code

## Implementation notes

- Enqueue: `MyService.delay(queue: DELAYED_JOB_QUEUES[:default]).method_name(args)`
- Tenant: `config/initializers/delayed_job_tenant.rb`
- Rules: `.cursor/rules/services-and-jobs.mdc`, `verify-before-inventing.mdc`
- AssetSonar: use ITAM queues (`:itam_sync`, `:initial_itam_sync`) per `assetsonar-itam.mdc`

## Compliance & products

| Product flag | Affected? | Notes |
|--------------|-----------|-------|
| `IS_EZ_OFFICE_NOT_CMMS` | Yes | All products share delayed_job |
| `IS_CMMS` | Yes | |
| `IS_ASSET_SONAR` | Yes | Heavy ITAM queue usage |
| `IS_RENTALS` | Yes | Basket/payment async |

## References

- `AGENTS.md` — Background jobs section
- PR harvest anti-pattern: Sidekiq assumptions in AI-generated code

---

*Accepted — codifies long-standing platform decision.*
