# Team Adoption Playbook

Architecture fails more often for people reasons than technical ones. A team optimising only for this sprint's delivery will not adopt hexagonal architecture because someone told them to. This playbook (based on practitioner experience, including Florian's) favours teaching and evidence over directives.

## 1. Teach instead of directing
- Run a short hands-on session: take one real class from their codebase and split it into domain + port + adapter together.
- Explain at the principle level first: *separate technical code from business logic*. Patterns and folder names come after the principle lands.
- Appeal to craftsmanship: people like to be good at their job; show what "good" looks like.

## 2. Show benefits they personally feel
- **Testability without dependencies**: domain tests that run in milliseconds with no DB or container. Demo it live.
- **Tech changes, business rules don't**: point to a past migration or vendor swap that hurt, and show how it would be confined to one adapter.
- **Less fear of change**: smaller blast radius per change, easier code review.
- **Better AI assistance**: agents make safer changes when boundaries are explicit.

## 3. Start small and measure
- Pick one service. Capture a baseline (see `measuring-architecture.md`).
- Let an AI agent (Claude Code, Copilot) do the mechanical refactor of that one service; the team reviews ports and naming.
- Re-measure and share the before/after table, including costs. A clear signal, even a qualitative one, beats a debate.

## 4. Automate so it sticks
- Add dependency rules to CI (ArchUnit, import-linter, dependency-cruiser) and static analysis (SonarQube quality gates).
- Provide a template/scaffold for new services so the right structure is the easy path.
- Use dependency injection / a composition root consistently.

## 5. Get people excited, then let them own it
- Celebrate the first adapter swap that "just worked".
- Invite volunteers to refactor the next service; rotate ownership.
- Keep an architecture decision record (ADR) stating when the team uses the full pattern and when the lightweight option is enough.

## Handling common objections

| Objection | Response |
|---|---|
| "It slows us down." | True at first. Show the pilot numbers; offer the lightweight option for urgent work. |
| "Too many interfaces." | Only create ports where there is a real external dependency; not every class needs one. |
| "We'll never switch databases." | Maybe not, but testability alone pays for it, and vendors/APIs change more often than DBs. |
| "Our framework already does this." | Frameworks organise code around the framework; hexagonal organises it around the business. |
