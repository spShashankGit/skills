# Decision Framework: Is Hexagonal Architecture Worth It Here?

Hexagonal architecture is an investment: you pay up front (interfaces, mapping, wiring, learning time) and collect returns later (cheaper change, easier testing, swappable tech). Whether that investment pays off depends on the situation, so ask before prescribing.

## The four fitness questions

Ask the user (or infer from context) and rate each one.

### 1. Business urgency — how dependent is the company's survival on shipping this feature now?
Example: a start-up that must demo a feature to close its next funding round.
- **Existential / very high** → speed wins. Ship, but keep a thin seam (see "Lightweight option" below).
- **Normal** → no penalty for doing it properly.

### 2. Lifespan — how certain am I that this code will still exist and be maintained three years from now?
- **Low** (prototype, experiment, campaign microsite, likely to be rewritten)
- **Medium**
- **High** (core product, regulated domain, system of record)

### 3. Growth — how certain am I that this code will grow in features, integrations or team size?
- **Low** (stable CRUD, one integration, done-and-dusted)
- **Medium**
- **High** (new channels, more vendors, more teams touching it)

### 4. Team maturity — is the team optimising only for delivery-in-the-moment, or also for the long term?
- A team under pure delivery pressure will resist; you'll need the adoption playbook (`team-adoption.md`) and a small, measurable pilot rather than a mandate.
- A team that already values craftsmanship will usually adopt quickly once shown the benefits.

## Reading the result

| Lifespan | Growth | Recommendation |
|---|---|---|
| Low | Low | Skip hexagonal. Simple layered or even single-module code is fine. |
| Low | Medium/High | Lightweight option: isolate the one or two volatile integrations behind ports. |
| Medium | any | Apply to the core domain; keep peripheral modules simple. |
| High | Medium/High | Full hexagonal for the core domain; strong fit. |
| High | Low | Apply where tech is likely to change (DB, vendors); keep ceremony minimal. |

A **low** rating on lifespan *and* growth means you can reasonably ignore hexagonal architecture. Overlay urgency: if survival depends on shipping this month, defer the full refactor but still use the lightweight option, and put a date on revisiting it.

## Lightweight option (for urgent or uncertain cases)

Even when full hexagonal isn't justified, keep business rules out of controllers and ORM entities, and put an interface in front of any third-party vendor. This costs very little and keeps the door open for a later refactor.

## Additional signals that tilt towards hexagonal
- Multiple entry points to the same logic (REST + queue + batch job + CLI)
- A vendor or database migration is planned or likely
- Tests are slow or flaky because they need real infrastructure
- Business rules are duplicated across controllers or scattered in SQL
- AI agents will do a lot of the coding: clear boundaries give agents smaller, well-defined contexts to change safely

## Signals that tilt away
- Pure data pipeline / thin CRUD with almost no business rules
- Very small team with a short, fixed project horizon
- Framework-heavy domain where the framework *is* the product (e.g. a CMS theme)
