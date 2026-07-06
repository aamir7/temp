# Playbook: React & API Contracts

Use when adding or changing React UI that consumes Rails APIs, serializers, or workflow converters.

**Source:** PR harvest 2026-07-01 (Theme: React TS safety & API contract drift).

## When to use

- New React screen, dialog, or tab in `app/react_web/`
- Changes to parsers, converters, Redux actions that store API data
- Serializer changes that feed React UI
- ITSM embed / external base URL link behavior

## Before you code

- [ ] Read `.cursor/rules/react-frontend.mdc`
- [ ] Find sibling screen — copy parser/converter/hook pattern, not logic
- [ ] Confirm `IS_*` products for this UI ([starting-from-redmine-ticket.md](starting-from-redmine-ticket.md))
- [ ] Glossary: `Member` in React = staff user UI model, not Rails `Member` model ([domain-glossary.md](../domain-glossary.md))

## Steps

### 1. Types first

- Locate interface in `core/models/data/<module>/`
- **Never** add `export type Foo = any` when interface exists
- Extend interface if API adds fields — don't bypass with `any`

### 2. Parser / converter path

```
API response (snake_case)
    → parser in core/repository/parser/ OR converter in core/utils/converter/
    → typed domain object (camelCase)
    → Redux slice / component props
```

- On parser failure: surface error state — **do not** dispatch raw API JSON as typed array
- When adding API fields: update serializer **and** converter/workflow TS files in same PR

### 3. Business rules stay on server

| Wrong | Right |
|-------|-------|
| Duplicate `Task` qualifying-type logic in `submitButton.tsx` | API returns eligibility or disabled state |
| Client-side-only validation for permissions | CanCan / API 403 + UI disabled state |

If you need a new rule in UI, add it to model/service first, then consume in React.

### 4. i18n & copy

- User-visible strings → locales / `localizedStrings` / `strings.ts` per existing screen pattern
- Verify string key matches action (activate dialog ≠ deactivate label)

### 5. Async actions

```typescript
// ❌ navigate immediately after dispatch
dispatch(deleteVendor(id));
handleDeleteSuccess();

// ✅ navigate in fulfilled handler or after await unwrap
await dispatch(deleteVendor(id)).unwrap();
handleDeleteSuccess();
```

### 6. PR self-check

- [ ] `/review-pr` or manual pass on checklist React section
- [ ] Demo on correct product deployment if `IS_*` gated

## Checklist (reviewer)

| Check | |
|-------|---|
| No `any` type aliases where interface exists | |
| Parser/converter covers new API fields | |
| Serializer exposes fields UI reads | |
| No FE/BE duplicate business rules | |
| i18n keys match actions | |
| Async success handlers after completion | |
| Product flags correct | |

## Golden references

- Parsers: `app/react_web/src/core/repository/parser/`
- Converters: `app/react_web/src/core/utils/converter/`
- Member types: `app/react_web/src/core/models/data/member/`
- Rules: `.cursor/rules/react-frontend.mdc`

---

*Maintainer: architects + frontend champions.*
