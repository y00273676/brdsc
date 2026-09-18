# BRDSC

**BRDSC** is an engineering judgment framework for evaluating whether code and design form a coherent system, rather than merely solving the current problem.

BRDSC stands for:

* **B — Boundary**
* **R — Responsibility**
* **D — Dependency**
* **S — Single Authority**
* **C — Consistency**

It is designed to be used by both engineers and AI coding agents during design, implementation, refactoring, and code review.

---

## Why BRDSC?

A codebase can be correct, well-formatted, and fully tested, while still feeling fragmented.

The problem is often not syntax or code style.

The problem is that different parts of the system make different engineering decisions:

* similar logic lives in different places
* responsibilities move between layers
* dependencies grow in arbitrary directions
* the same business rule is implemented multiple times
* every new feature introduces a new architectural pattern

This is usually what people mean when they say:

> "The code works, but it doesn't feel systematic."

BRDSC provides a shared framework for making those engineering decisions.

It does not prescribe one architecture.

It helps answer:

> Given this codebase, what design keeps the system coherent?

---

# The Framework

## B — Boundary

Modules and business concepts should have clear boundaries.

Typical questions:

* What concept does this code belong to?
* Which module owns this behavior?
* Are independently changing concerns mixed together?
* Is one module leaking internal details into another?

A boundary should exist because there is a meaningful difference in responsibility, lifecycle, ownership, consistency, failure behavior, or rate of change.

---

## R — Responsibility

The same kind of responsibility should have a stable owner.

Typical questions:

* Who should own this logic?
* Is this orchestration, business logic, persistence, integration, or presentation?
* Where does similar logic already live?
* Would the next engineer know where to add similar behavior?

The goal is not "one function, one responsibility."

The goal is stable responsibility ownership.

---

## D — Dependency

Dependencies should have a clear and intentional direction.

Typical questions:

* Who depends on whom?
* Is this dependency expected by the architecture?
* Is business logic depending on technical implementation details?
* Is this change introducing a dependency cycle?

Dependency direction is not about maximizing abstraction.

Invert a dependency only when the boundary actually matters.

---

## S — Single Authority

Important business rules should have one authoritative expression.

For example:

> An order can only be paid while it is pending.

That rule should not be independently redefined in:

* HTTP handlers
* message consumers
* cron jobs
* application services
* SQL statements

Multiple layers may perform defensive checks, but the business meaning should have one authoritative source.

Single Authority means:

> one source of business truth, not necessarily one physical validation statement.

---

## C — Consistency

Similar problems should usually be solved in similar ways.

Before introducing a new design, inspect how the project already handles:

* repositories
* transactions
* errors
* events
* state transitions
* asynchronous workflows
* validation
* dependency injection

A theoretically cleaner local design may make the system worse if it introduces a second architectural model.

Always ask:

> Am I improving the existing system, or creating another way to solve the same problem?

---

# BRDSC Is Not an Architecture

BRDSC does not require:

* DDD
* Clean Architecture
* Hexagonal Architecture
* CQRS
* Event Sourcing
* Event-Driven Architecture
* Repository Pattern
* Layered Architecture

Those are implementation choices.

BRDSC sits one level above them.

It helps decide whether a pattern is appropriate for the current system.

For example, BRDSC does not say:

> Cross-module behavior must use events.

Instead it asks:

* Are the concerns independently changing?
* Do they have different failure semantics?
* Does the project already have an asynchronous mechanism?
* Would adding events strengthen or fragment the existing architecture?

---

# Example

Suppose a payment workflow looks like this:

```go
func PayOrder(ctx context.Context, orderID int64) error {
    if err := updateOrder(ctx, orderID); err != nil {
        return err
    }

    if err := addPoints(ctx, orderID); err != nil {
        return err
    }

    if err := sendCoupon(ctx, orderID); err != nil {
        return err
    }

    if err := sendNotification(ctx, orderID); err != nil {
        return err
    }

    return nil
}
```

BRDSC does not immediately say:

> Use domain events.

Instead evaluate it.

### Boundary

Payment, loyalty, coupon, and notification may belong to different business boundaries.

### Responsibility

The payment workflow is coordinating concerns with different responsibilities.

### Dependency

Payment is directly coupled to multiple downstream capabilities.

### Single Authority

The order state transition should still have one authoritative business rule.

### Consistency

Before introducing events, check how the project already handles cross-module side effects.

The recommendation may be events, an outbox, an existing job mechanism, or simply keeping the current code if the system is small enough.

BRDSC provides the reasoning process, not a predetermined answer.

---

# The Most Important Rule

Before introducing an abstraction, identify the design pressure that justifies it.

Useful design pressures include:

* protecting a business invariant
* isolating independently changing concerns
* separating failure or retry semantics
* creating a real dependency boundary
* removing meaningful conceptual duplication
* preserving transaction integrity
* following an established project convention

If there is no meaningful design pressure, prefer the simpler solution.

---

# BRDSC for AI Coding Agents

BRDSC can be used as a Skill for coding agents such as Codex, Claude Code, or other repository-aware agents.

The purpose is to prevent agents from making locally elegant but globally inconsistent changes.

An agent using BRDSC should:

1. understand the requested change
2. inspect similar code in the repository
3. infer the project's existing architectural conventions
4. evaluate Boundary, Responsibility, Dependency, Single Authority, and Consistency
5. identify the actual design pressure
6. recommend the smallest coherent change

The agent should not automatically introduce patterns such as:

* domain events
* repositories
* strategy patterns
* interfaces
* state machines
* additional layers

Architectural patterns should be introduced only when justified by the system.

---

# Code Review with BRDSC

BRDSC reviews should explain reasoning rather than produce vague architectural comments.

Avoid:

```text
This design is bad.

This violates DDD.

This should be decoupled.

Use events here.
```

Prefer:

```text
Observation:
PaymentService currently owns payment orchestration, loyalty updates,
coupon issuance, and notification delivery.

Principle:
Boundary / Responsibility

Evidence:
These behaviors have different failure, retry, and lifecycle semantics.

Risk:
Additional post-payment behaviors will continue expanding the payment
workflow and increase coupling.

Direction:
Keep the core payment transaction local and move independently retryable
side effects behind the project's existing asynchronous mechanism.
```

The goal is not merely to identify a problem.

The goal is to make the engineering judgment understandable and reusable.

---

# What BRDSC Tries to Prevent

BRDSC is especially useful against two opposite failure modes.

### No architecture

Every feature is implemented independently.

```text
Feature A → Pattern A
Feature B → Pattern B
Feature C → Pattern C
```

The code works, but the system gradually loses coherence.

### Architecture formalism

Every problem is forced into architectural patterns.

```text
if → Strategy Pattern

state → State Machine

database → Repository

cross-module call → Event

business object → Aggregate
```

The code becomes more complicated without gaining meaningful engineering value.

BRDSC aims for the middle:

> enough structure to preserve coherence, without architecture for architecture's sake.

---

# Core Question

The final BRDSC question is:

> Does this change make the system more coherent, or does it merely make this piece of code look cleaner?

The goal is **system coherence**, not architectural purity.

---

# Repository Structure

A typical BRDSC Skill repository can look like:

```text
brdsc/
├── README.md
├── SKILL.md
└── examples/
    ├── boundary.md
    ├── responsibility.md
    ├── dependency.md
    ├── single-authority.md
    └── consistency.md
```

`SKILL.md` contains the instructions used by AI agents.

`examples/` should contain real design and code review cases.

The examples are important because engineering judgment is learned better from concrete trade-offs than from abstract rules.

---

## License

Choose a license appropriate for how you want BRDSC to be reused and modified.

