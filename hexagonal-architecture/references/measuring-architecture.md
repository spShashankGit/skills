# Measuring Architecture Objectively

"Is this architecture better?" should be answered with data, measured before and after a change on the same service with the same tools. This file lists data points, which of them hexagonal architecture should move, and how to collect them.

## Contents
1. Delivery outcomes: DORA metrics
2. Code-structure metrics
3. Change-locality metrics (from git history)
4. Testability metrics
5. AI-assistance metrics
6. Which metrics matter most for hexagonal architecture
7. Running a before/after experiment

---

## 1. Delivery outcomes: DORA metrics

Source: https://dora.dev/guides/dora-metrics/

| Metric | Definition | Expected effect of hexagonal |
|---|---|---|
| Deployment Frequency (Bereitstellungshäufigkeit) | How often a team successfully deploys to production | Indirect ↑, via faster reliable tests |
| Lead Time for Changes (Vorlaufzeit für Änderungen) | Time from commit to running in production | ↓, if domain tests are fast and changes stay local |
| Time to Restore Service / MTTR (Wiederherstellungszeit) | Time to recover after an incident | Indirect ↓, faults easier to isolate to an adapter |
| Change Failure Rate (Änderungsfehlerrate) | % of deployments causing failures in production | ↓, via better-tested domain logic |

Caveat to tell the user: DORA metrics measure the whole delivery system (pipeline, process, people), so architecture is only one influence. They are lagging indicators and noisy on a single service over a short period. Use them to confirm the trend over months, and use the code metrics below as leading indicators.

## 2. Code-structure metrics

- **Dependency direction violations**: count of imports from the domain package into framework/infrastructure packages. Target: 0. This is the single most direct measure of "is this actually hexagonal?"
- **Afferent/efferent coupling (Ca/Ce) and Instability (I = Ce / (Ca + Ce))**, per package (Robert C. Martin). Domain should be stable (low I); adapters unstable (high I).
- **Cyclic dependencies** between packages. Target: 0.
- **Cyclomatic / cognitive complexity** per method in the domain (SonarQube reports cognitive complexity).
- **Framework annotations in domain** (`@Entity`, `@Autowired`, `@Column`, decorators): count; target 0.
- **Duplication** (SonarQube): business rules copied across controllers should drop.

Tools: SonarQube/SonarCloud, ArchUnit (Java/Kotlin), jQAssistant, NDepend (.NET), import-linter or pydeps (Python), dependency-cruiser or madge (TypeScript/JavaScript), Structure101.

## 3. Change-locality metrics (from git history)

These test the claim "tech changes a lot more than business logic; adapter code gets touched more."

- **Files touched per change**: median files per commit/PR for typical changes. Should drop for tech changes (they stay inside one adapter).
- **Churn by layer**: lines changed in domain vs. adapters over time. Healthy hexagonal code shows most churn in adapters, and domain churn tied to actual business-rule changes.
- **Change coupling / co-change**: files that always change together across layer boundaries indicate a leaky port. Tools: CodeScene, code-maat, or `git log --name-only` scripting.
- **Blast radius of a tech swap**: in a pilot, swap one adapter (e.g. in-memory repo → Postgres, or vendor A → vendor B) and count files changed outside the adapter. Target: only wiring.

## 4. Testability metrics

- **Domain unit tests without infrastructure**: % of domain tests that run with no DB, network or container. Target: ~100%.
- **Test suite runtime** for domain tests (seconds, not minutes).
- **Coverage of domain package** (line/branch), and ideally **mutation score** (PIT, Stryker, mutmut), which says whether tests actually check behaviour.
- **Flaky test rate**.
- **Mocks per test**: dropping, because in-memory fake adapters replace mocking frameworks.

## 5. AI-assistance metrics

Relevant if the team uses coding agents (Claude Code, Copilot, Kiro):
- Agent task success rate on a fixed set of change requests (first-attempt pass of tests)
- Number of files/context the agent needed to read for a change
- Review rework: human edits needed after agent output

Clear ports give agents a small, well-defined surface to change, so these should improve. Treat them as qualitative-to-semi-quantitative signals.

## 6. Which metrics matter most for hexagonal architecture

Prioritise in this order for a pilot:
1. Dependency direction violations (proves the structure exists)
2. % domain tests runnable without infrastructure, and domain test runtime
3. Blast radius of a tech swap / files touched per change
4. Churn by layer and change coupling across layers
5. Complexity and duplication in the domain
6. DORA metrics over the following quarters (confirms business value)

## 7. Running a before/after experiment

1. Pick one service with real business rules and at least one volatile integration.
2. Record the baseline: run SonarQube + dependency tool + test timings; extract git metrics for the last 3–6 months.
3. Refactor (an AI agent can do the mechanical parts; humans own the port design).
4. Re-run the same tools. Do the tech-swap exercise.
5. Report a small table: metric, before, after, delta, and note anything that got worse (e.g. more files, more interfaces). Honesty here builds trust with sceptical teams.
6. Add the dependency rule check to CI so the result doesn't erode.

### Example CI guardrails

ArchUnit (Java):
```java
@ArchTest
static final ArchRule domainIsIndependent = noClasses()
    .that().resideInAPackage("..domain..")
    .should().dependOnClassesThat()
    .resideInAnyPackage("..adapter..", "org.springframework..", "jakarta.persistence..");
```

import-linter (Python, `.importlinter`):
```ini
[importlinter]
root_package = app

[importlinter:contract:domain-independent]
name = Domain must not import adapters or frameworks
type = forbidden
source_modules = app.domain
forbidden_modules = app.adapters, sqlalchemy, fastapi, requests
```

dependency-cruiser (TypeScript, `.dependency-cruiser.js` excerpt):
```js
forbidden: [{
  name: 'domain-independent',
  severity: 'error',
  from: { path: '^src/domain' },
  to:   { path: '^src/(adapters|infrastructure)|node_modules/(express|typeorm|axios)' }
}]
```
