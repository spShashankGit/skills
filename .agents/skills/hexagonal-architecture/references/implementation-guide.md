# Implementation Guide

## Contents
1. Folder structure
2. Worked example (TypeScript; same shape in any language)
3. Refactoring an existing service step by step
4. Testing strategy
5. Common mistakes

---

## 1. Folder structure

```
src/
├── domain/            # Entities, value objects, domain services. No framework imports.
├── application/       # Use cases; defines ports
│   ├── ports/in/      # Driving ports (what the app offers): e.g. ApplyForLoan
│   └── ports/out/     # Driven ports (what the app needs): e.g. LoanRepository, CreditScoring
├── adapters/
│   ├── in/            # REST controllers, queue consumers, CLI → call driving ports
│   └── out/           # DB repositories, HTTP clients, email → implement driven ports
└── config/            # Composition root: wires adapters to ports (DI)
```

Smaller projects can merge `domain` and `application` into one `core` package. What matters is the dependency direction, not the folder names.

## 2. Worked example

```ts
// domain/Loan.ts — pure business rules
export class Loan {
  constructor(readonly id: string, readonly amount: number, private status: 'PENDING'|'APPROVED'|'REJECTED' = 'PENDING') {}
  decide(score: number) {
    if (this.amount <= 0) throw new Error('Amount must be positive');
    this.status = score >= 650 ? 'APPROVED' : 'REJECTED';
  }
  get currentStatus() { return this.status; }
}

// application/ports/out
export interface LoanRepository { save(loan: Loan): Promise<void>; }
export interface CreditScoring  { scoreFor(customerId: string): Promise<number>; }

// application/ApplyForLoan.ts — use case (driving port implementation)
export class ApplyForLoan {
  constructor(private loans: LoanRepository, private scoring: CreditScoring) {}
  async execute(customerId: string, amount: number) {
    const loan = new Loan(crypto.randomUUID(), amount);
    loan.decide(await this.scoring.scoreFor(customerId));
    await this.loans.save(loan);
    return loan.currentStatus;
  }
}

// adapters/out/PostgresLoanRepository.ts — implements the port
export class PostgresLoanRepository implements LoanRepository {
  constructor(private db: Pool) {}
  async save(loan: Loan) {
    await this.db.query('INSERT INTO loans(id, amount, status) VALUES ($1,$2,$3)',
      [loan.id, loan.amount, loan.currentStatus]);
  }
}

// adapters/in/LoanController.ts — driving adapter
app.post('/loans', async (req, res) => {
  res.json({ status: await applyForLoan.execute(req.body.customerId, req.body.amount) });
});

// config/wiring.ts — composition root
const applyForLoan = new ApplyForLoan(new PostgresLoanRepository(pool), new BureauCreditScoring(http));
```

## 3. Refactoring an existing service step by step

1. **Find the business rules** hiding in controllers, services and SQL. List them.
2. **Extract domain objects** holding those rules, with no framework imports.
3. **Define driven ports** for each external thing the rules need (DB, vendors, clock, IDs).
4. **Move existing infrastructure code into adapters** implementing those ports; add mapping between ORM/DTO and domain objects.
5. **Create use-case classes** (driving ports) and make controllers/consumers thin callers.
6. **Wire everything** in one composition root.
7. **Add the CI dependency rule** (see `measuring-architecture.md`).

Do it incrementally behind existing tests; each step should leave the service deployable. An AI coding agent is well suited to steps 2, 4 and 6 when given the port definitions.

## 4. Testing strategy

- **Domain + use cases**: fast unit tests with in-memory fakes (`InMemoryLoanRepository`, `FixedCreditScoring`). No mocks framework needed.
- **Adapters**: integration tests against the real technology (Testcontainers, sandbox APIs).
- **A few end-to-end tests** through a driving adapter for confidence in wiring.
- Optional: **contract tests** run against both the fake and the real adapter, proving they behave alike.

## 5. Common mistakes

- ORM entities used as domain objects (annotations leak the DB into the domain).
- Ports named after technology (`MongoPort`) instead of business capability.
- Creating a port for every class, including pure in-process helpers. Ports are for boundaries.
- Business logic drifting into adapters ("just one if-statement in the controller").
- Anemic domain: all logic in use-case classes, domain objects are just data bags.
- No automated rule check, so the structure erodes within months.
