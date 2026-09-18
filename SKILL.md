# BRDSC

BRDSC is an engineering judgment framework for evaluating whether code and design form a coherent system, rather than merely solving the current problem.

BRDSC stands for:

* **B — Boundary**
* **R — Responsibility**
* **D — Dependency**
* **S — Single Authority**
* **C — Consistency**

The purpose of BRDSC is not to enforce a specific architecture such as DDD, Clean Architecture, Hexagonal Architecture, CQRS, or Event-Driven Architecture.

Its purpose is to help make consistent engineering judgments when there is no single obviously correct implementation.

---

## When to Use

Use BRDSC when:

* designing a new module or feature
* reviewing code or pull requests
* refactoring existing code
* deciding where logic should live
* evaluating abstractions
* changing module dependencies
* introducing new architectural patterns
* deciding whether existing project conventions should be preserved
* identifying why code feels fragmented or "not systematic"

Do not use BRDSC as a mechanical lint checklist.

BRDSC is primarily a reasoning framework.

---

# Core Principles

## B — Boundary

Modules and business concepts should have clear boundaries.

Ask:

* What concept does this code belong to?
* Which module owns this behavior?
* Is this code crossing a module boundary?
* Does one module know too much about another module's internals?
* Are two independently changing concerns being mixed together?
* Could this change cause unrelated modules to change together?

A good boundary separates concepts that have different responsibilities, change drivers, ownership, or lifecycle.

Do not create boundaries merely to make the architecture appear cleaner.

A boundary should have an engineering reason.

Typical signals of weak boundaries:

* one service coordinates many unrelated domains
* modules directly modify each other's internal state
* internal database structures leak across modules
* unrelated features repeatedly change the same package
* a module exposes implementation details instead of domain capabilities

---

## R — Responsibility

The same kind of responsibility should have a stable owner.

Ask:

* Who should be responsible for this behavior?
* Is this orchestration, business logic, persistence, integration, validation, or presentation?
* Where does similar logic currently live?
* Would a future engineer know where to add the next piece of similar logic?
* Is one component accumulating unrelated responsibilities?

The goal is not to make every class or function do exactly one tiny thing.

The goal is stable responsibility ownership.

For example:

* transport layers handle protocol concerns
* application layers coordinate workflows
* domain logic protects business rules
* repositories handle persistence semantics
* infrastructure handles external systems

These are examples, not mandatory layers.

Judge responsibility relative to the architecture already established by the project.

---

## D — Dependency

Dependencies should have a clear and intentional direction.

Ask:

* Who depends on whom?
* Is this dependency expected by the architecture?
* Is a lower-level business concept depending on a higher-level technical detail?
* Is a module reaching across layers because doing so is convenient?
* Would replacing an infrastructure implementation force business logic to change?
* Is this dependency creating a cycle?

Prefer dependency relationships that preserve conceptual stability.

Stable business concepts should generally not depend on volatile technical details unless the project deliberately follows a different architecture.

Avoid introducing abstractions solely for dependency inversion.

An abstraction is useful only when the dependency boundary matters.

---

## S — Single Authority

Each important business rule should have one authoritative expression.

This does not mean a rule can appear only once in the entire system.

Defensive validation, database constraints, API validation, and integrity checks may legitimately duplicate parts of a rule.

The key question is:

> Where is the authoritative business meaning defined?

Ask:

* What is the business invariant?
* Where is it enforced?
* Can another code path bypass this rule?
* Are multiple modules independently defining the same rule?
* If the rule changes, how many places must change?
* Could two implementations gradually develop different semantics?

Example:

If an order can only be paid while it is pending, there should be one authoritative business rule representing that transition.

Prefer:

```go
func (o *Order) Pay(paidAt time.Time) error {
    if o.Status != StatusPending {
        return ErrInvalidOrderStatus
    }

    o.Status = StatusPaid
    o.PaidAt = &paidAt
    return nil
}
```

over independently implementing the same transition semantics in handlers, consumers, services, and jobs.

Single Authority means:

**one source of business truth, not necessarily one physical validation statement.**

---

## C — Consistency

Similar problems should usually be solved in similar ways.

Before introducing a design, inspect how the project already solves the same category of problem.

Ask:

* How does this repository normally represent persistence?
* How are transactions handled elsewhere?
* How are business errors represented?
* How are cross-module side effects handled?
* How are asynchronous workflows implemented?
* How are state transitions represented?
* How are similar modules structured?
* Am I strengthening the existing architecture or introducing a second architecture?

Do not automatically replace an existing convention because another pattern appears theoretically cleaner.

For example, if the project consistently uses:

```go
repo.MarkAsPaid(ctx, id, version, paidAt)
```

do not automatically replace one repository with:

```go
repo.Save(ctx, order)
```

simply because aggregate persistence looks more aligned with DDD.

First determine what repository means in this codebase.

Consistency is not blind conformity.

Break an existing convention when there is a clear architectural reason, and make the change explicit.

---

# BRDSC Reasoning Process

When reviewing or designing code, do not immediately recommend a pattern.

Follow this sequence.

## Step 1 — Understand the Change

Identify:

* what behavior is being added or changed
* what business concept is involved
* what data is affected
* what side effects occur
* what failure modes exist
* what is expected to change independently in the future

Do not reason from isolated functions if broader project context is available.

---

## Step 2 — Inspect Existing Project Conventions

Before recommending a new abstraction, search the project for similar cases.

Determine:

* existing module boundaries
* responsibility placement
* dependency patterns
* transaction patterns
* error handling
* persistence conventions
* event conventions
* naming conventions
* similar business rules

Prefer learning the architecture before judging it.

---

## Step 3 — Evaluate BRDSC

### Boundary

Determine whether responsibilities and concepts remain properly separated.

### Responsibility

Determine whether the behavior has an appropriate and stable owner.

### Dependency

Determine whether dependencies remain intentional and directional.

### Single Authority

Determine whether business invariants retain one authoritative definition.

### Consistency

Determine whether the solution fits the project's established engineering model.

---

## Step 4 — Identify the Real Design Pressure

Do not introduce abstraction without identifying the pressure that justifies it.

Useful design pressures include:

* protecting a business invariant
* isolating independently changing concerns
* preventing architectural dependency violations
* supporting different failure or retry semantics
* removing meaningful duplication
* establishing an explicit transaction boundary
* handling scalability differences
* separating lifecycle ownership
* following an established project convention

If none of these pressures exist, prefer the simpler design.

---

## Step 5 — Recommend the Smallest Coherent Change

Prefer the smallest change that improves system coherence.

Avoid rewriting architecture merely because a different design is theoretically cleaner.

Recommendations should preserve local consistency unless there is a strong reason to change the architecture itself.

---

# Avoid Architectural Formalism

BRDSC must not become architecture ceremony.

Do not automatically conclude:

* every business object needs an aggregate
* every cross-module call requires an event
* every conditional needs a strategy pattern
* every database access needs another abstraction
* every service must be split
* every state change requires a state machine
* every duplicate line requires abstraction
* every module must follow DDD

Architecture exists to manage change and complexity.

If the problem is simple, the solution should remain simple.

---

# Abstraction Rule

Introduce an abstraction only when it does at least one meaningful thing:

* protects a business invariant
* isolates an independently changing concern
* creates a necessary dependency boundary
* removes meaningful conceptual duplication
* provides independent failure or retry behavior
* represents an important domain concept
* follows an already established project convention

Do not abstract merely to make code appear architecturally sophisticated.

---

# Consistency vs Improvement

Existing consistency is valuable, but consistency does not justify preserving a clearly harmful architecture forever.

When an existing pattern should change:

1. identify the concrete problem
2. explain why the existing pattern causes it
3. define the new principle
4. decide whether the migration is local or system-wide
5. avoid leaving two competing architectural models without a migration strategy

A local improvement that creates a second architecture may make the system worse overall.

Always ask:

> Is this change improving the system, or merely creating another way to solve the same problem?

---

# Review Style

Do not produce vague architectural feedback.

Avoid comments such as:

* "This design is bad."
* "This is not DDD."
* "This should be decoupled."
* "Responsibilities are unclear."
* "This violates clean architecture."
* "Use events here."

Instead explain the reasoning.

Use this format when useful:

## Observation

Describe what the code currently does.

## Principle

Identify the relevant BRDSC dimension.

## Evidence

Point to concrete code structure or behavior.

## Risk

Explain what type of future problem this design creates.

## Direction

Describe the architectural direction of improvement.

## Scope

State whether the issue requires:

* no change
* local refactoring
* module-level change
* architecture-level change

Example:

### Observation

`PaymentService` currently performs:

* payment orchestration
* order state transition
* point allocation
* coupon issuance
* user notification

### Principle

Boundary / Responsibility

### Evidence

These behaviors have different lifecycle, retry, and failure semantics.

### Risk

Future additions such as loyalty levels, marketing notifications, and warehouse synchronization will continue expanding the payment workflow and increase coupling.

### Direction

Keep the core payment transaction in the payment workflow and move independently retryable side effects behind the project's existing asynchronous mechanism.

Do not introduce a new event architecture if the project already has another established mechanism that solves the same problem.

---

# Questions to Ask Before Making a Design Decision

Use these questions as a reasoning aid.

### Concept

* What business or technical concept is this code representing?
* Is that concept explicit in the current design?

### Ownership

* Who should own this behavior?
* Does similar behavior already have an owner?

### Change

* Why is this code likely to change?
* What should be able to change independently?

### Invariant

* What must always remain true?
* Where is that rule authoritatively enforced?
* Can another path bypass it?

### Dependency

* What does this component depend on?
* Is that dependency conceptually correct?
* Could this dependency introduce cycles or inappropriate coupling?

### Consistency

* How does this project solve similar problems?
* Am I following an existing pattern?
* If I am breaking it, why?

### Complexity

* Is the proposed abstraction solving a real problem?
* Would a simpler implementation remain understandable and safe?

### Evolution

* If three similar requirements are added later, what will this design become?
* Does the design make future changes more local or more distributed?

---

# Business Rule Detection

Treat a rule as a likely business invariant when it expresses something that must remain true regardless of entry point.

Examples:

* an order cannot be paid twice
* a refund cannot exceed the paid amount
* inventory cannot become negative
* an employee allocation cannot exceed the allowed ratio
* an account cannot transition directly from closed to active
* a coupon cannot be used after expiration

Such rules should not live only in:

* HTTP validation
* UI validation
* individual consumers
* scheduled jobs
* database update statements

Those may enforce the rule defensively, but they should not become independent definitions of business meaning.

---

# Entry Points Are Not Business Authorities

A system may have many entry points:

* HTTP
* RPC
* message consumer
* cron job
* admin tool
* migration script

Do not allow each entry point to independently implement core business semantics.

Prefer:

```text
HTTP ───────┐
RPC ────────┤
Consumer ───┼──> Business Capability
Cron ───────┤
Admin ──────┘
```

over:

```text
HTTP      -> business rule A
RPC       -> business rule A'
Consumer  -> business rule A''
Cron      -> business rule A'''
```

---

# Boundary Detection

Possible indicators that two concerns deserve separate boundaries:

* different owners
* different lifecycle
* different scaling requirements
* different consistency requirements
* different failure semantics
* different security boundaries
* different business language
* different rates of change

Do not split modules based only on file size or line count.

---

# Responsibility Detection

A responsibility is not simply "a block of code that can be extracted."

A responsibility should describe why the code exists.

Weak responsibility:

> processOrderData

Stronger responsibilities:

* calculate payable amount
* enforce refund eligibility
* persist order aggregate
* publish order state changes
* translate HTTP input
* coordinate payment workflow

Prefer responsibility boundaries based on purpose rather than implementation mechanics.

---

# Dependency Detection

Watch for:

* domain packages importing database clients
* application code depending directly on transport DTOs
* repositories calling HTTP handlers
* infrastructure packages defining business semantics
* circular module references
* utility packages becoming global dependency hubs
* business logic importing vendor SDKs unnecessarily

Not every dependency requires inversion.

Invert a dependency when the boundary matters.

---

# Consistency Detection

Search for existing answers before inventing a new one.

Useful searches include:

* similar repository methods
* similar state transitions
* existing events
* transaction wrappers
* error types
* retry mechanisms
* dependency injection patterns
* package layout
* interface placement
* validation conventions

Consistency applies to decisions, not superficial syntax.

Two implementations can look different while still following the same architectural principle.

---

# BRDSC Is Not a Score

Do not assign numeric scores to BRDSC dimensions.

Do not require all five dimensions to be maximized simultaneously.

Engineering involves trade-offs.

Examples:

* improving Boundary may temporarily reduce Consistency during a migration
* preserving Consistency may be preferable to introducing a theoretically cleaner Dependency model
* reducing Responsibility overlap may introduce unnecessary abstraction in a small module

Explain trade-offs instead of scoring them.

---

# Priority Order

When principles conflict, reason using this approximate priority:

1. correctness and business invariants
2. safety and data integrity
3. clear ownership and boundaries
4. dependency integrity
5. consistency with existing architecture
6. abstraction elegance

This is guidance, not an absolute rule.

---

# Default Bias

When evidence is insufficient:

* prefer the existing project convention
* prefer fewer abstractions
* prefer explicit code
* prefer local changes
* avoid introducing a second architectural model

Do not redesign the system based on isolated code.

---

# Final Review Question

Before recommending a change, ask:

> Does this change make the system more coherent, or does it merely make this piece of code look cleaner?

The goal of BRDSC is system coherence.

Not architectural purity.

