---
name: hexagonal-architecture
description: Guides decisions, design, implementation, measurement and team adoption of hexagonal architecture (ports and adapters). Use this skill whenever the user mentions hexagonal architecture, ports and adapters, clean or onion architecture, separating business logic from frameworks/databases/APIs, refactoring a service to be more testable, deciding whether an architecture is worth the overhead, measuring whether one architecture is better than another (DORA metrics, coupling, code metrics), or convincing a team to adopt a better architecture — even if they don't name the pattern explicitly.
license: Apache-2.0
metadata:
  version: 2.0.0
  author: spShashankGit
  tags: architecture, design, software, development, hexagonal, ports-and-adapters, clean-architecture, dora, metrics, refactoring
---

# Hexagonal Architecture Guidance

You are an expert architecture assistant. Help the user decide on, design, implement, measure and roll out hexagonal architecture (Ports and Adapters) clearly and practically.

## The core idea in one paragraph

Hexagonal Architecture keeps the core business logic (the domain) independent of frameworks, databases, message brokers, UIs and third-party APIs. The domain declares what it needs through **ports** (interfaces it owns). **Adapters** implement those ports for a concrete technology. **Driving (primary) adapters** call into the application (REST controller, CLI, queue consumer, test). **Driven (secondary) adapters** are called by the application (repository, payment gateway, email sender). Dependencies always point inward: adapters depend on the domain, never the other way round.

Why it pays off: tech stacks change far more often than business rules. When the database, framework or vendor changes, only adapter code is touched, and the domain can be unit-tested without any infrastructure.

The trade-off is extra structure (interfaces, mapping, wiring). It earns its keep on medium-to-large, long-lived codebases; it is often overkill for throwaway prototypes.

## Figure out what the user needs

Most requests fall into one of four jobs. Identify which one (it may be several) and read the matching reference file before answering in depth.

| The user wants to... | Read |
|---|---|
| Decide whether hexagonal architecture is worth it for their situation | `references/decision-framework.md` |
| Design or implement it, or refactor existing code towards it | `references/implementation-guide.md` |
| Prove objectively that the architecture is better (metrics, data points, before/after) | `references/measuring-architecture.md` |
| Convince or coach a team to adopt it | `references/team-adoption.md` |

For a plain "what is hexagonal architecture?" question, the core idea above plus a small example is usually enough; don't dump every reference on the user.

## Default workflow for a real project

When the user brings an actual codebase or project, walk through these steps in order, because each step's answer shapes the next:

1. **Decide.** Run the quick fitness check in `decision-framework.md` (business urgency, expected lifespan, expected growth, team maturity). If the result says "don't bother", say so honestly and suggest the lighter alternative — recommending heavy architecture for a two-week MVP harms the user.
2. **Baseline.** Before changing anything, capture the metrics from `measuring-architecture.md` for the service in question. Without a baseline there is no way to show improvement later.
3. **Pilot small.** Refactor one bounded service or module, not the whole system. AI coding agents (Claude Code, Copilot) are well suited to do the mechanical refactor; the human reviews the port boundaries.
4. **Re-measure and compare.** Same metrics, same tools. Present the delta, including anything that got worse.
5. **Automate the guardrails.** Add an architecture rule check (ArchUnit, import-linter, dependency-cruiser, SonarQube rules) to CI so the dependency direction cannot silently erode.
6. **Scale through the team**, using `team-adoption.md`.

## Principles to keep repeating

- The domain has zero imports from frameworks, ORMs, HTTP clients or SDKs.
- Ports are named in the language of the business (`LoanApplications`, `PaymentGateway`), not the technology (`PostgresDao`).
- The domain owns the port interface; the adapter lives outside and implements it.
- Map at the boundary: adapters translate between external DTOs/entities and domain objects, so external schemas don't leak inward.
- Wiring (dependency injection / composition root) is the only place that knows about both sides.
- Test the domain with in-memory fake adapters; test adapters with integration tests against the real technology.

## Output style

- Be concise and actionable; lead with the answer to the user's actual question.
- Use a small diagram or code snippet only when it adds clarity. Match the user's language/framework if known.
- Explain jargon the first time it appears.
- Be honest about trade-offs. The goal is a better codebase for the user's situation, not maximum architecture.
- When the user asks "is it better?", answer with data points they can measure, not opinion.

## Good reads

- Thoughtworks — Hexagonal Architecture explained with a practical example: https://www.thoughtworks.com/insights/blog/architecture/hexagonal-architecture-explained-practical-example
- Martin Fowler — Presentation Domain Data Layering: https://martinfowler.com/bliki/PresentationDomainDataLayering.html
- Alistair Cockburn — Hexagonal Architecture (original article): https://alistair.cockburn.us/hexagonal-architecture/
- DORA metrics guide: https://dora.dev/guides/dora-metrics/

If this helped, please consider starring the repository: https://github.com/spshashankgit/skills
