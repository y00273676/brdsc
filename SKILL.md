---
name: brdsc
description: Evaluate module boundaries, responsibility ownership, dependency direction, business-rule authority, and architectural consistency during design, code review, or refactoring. Use when these decisions or an abstraction trade-off matter to the task. Do not activate for formatting, mechanical edits, or routine fixes that need no architectural judgment.
---

# BRDSC

Use BRDSC to judge whether a change keeps the system coherent. It does not prescribe DDD, Clean Architecture, events, repositories, or any other architecture.

## Scope the Work

Follow the user's requested mode: review, design, or implementation. A review produces findings; it does not authorize refactoring. During implementation, use this framework to guide the requested change without expanding it into an architecture overhaul.

Scale the investigation to the decision. For a small change, inspect the relevant dimensions and nearby context. Do not require a five-dimension report or assign numeric BRDSC scores.

## Core Principles

### B — Boundary

Separate concerns when differences in business meaning, ownership, lifecycle, security, consistency requirements, or rate of change justify a boundary.

Look for internal state or storage details leaking across modules, and unrelated changes repeatedly forcing modules to change together. File size, the number of calls, or different business nouns alone do not establish a boundary problem.

### R — Responsibility

Give each kind of behavior a stable, discoverable owner. Distinguish protocol handling, workflow coordination, business decisions, persistence, and external integration according to the project's architecture; these are categories to investigate, not mandatory layers.

Coordinating several capabilities can be a valid application-service responsibility. Investigate whether the coordinator delegates to their owners or redefines their internals. Name responsibilities by purpose, such as enforcing refund eligibility or coordinating payment, rather than by extraction opportunities.

### D — Dependency

Keep dependencies intentional and consistent with the project's boundaries. Inspect cycles, callers depending on implementation details, transport types crossing inappropriate boundaries, and technical dependencies forcing unrelated business changes.

Prefer stable concepts depending on stable contracts when that separation matters. Introduce an interface or invert a dependency only when there is a concrete boundary to protect; a database import or vendor SDK is a signal to investigate, not sufficient evidence of a defect.

### S — Single Authority

Give each important business rule an identifiable authoritative meaning and owner. Trace whether HTTP, RPC, consumers, jobs, and administrative paths share that meaning or independently redefine it. Defensive validation at multiple layers can be legitimate.

Distinguish **semantic ownership** from **atomic enforcement**. A business capability may define its contract through an entity method, application service, repository operation, or database constraint, according to the project. Do not require an entity method or duplicate a rule in application code merely because its enforcement lives in SQL.

For example, `MarkAsPaid` can own the pending-to-paid transition through a conditional update and consistent handling of the affected-row count. Database constraints, locking, conditional updates, and transactions may be essential to correctness. An in-memory `Order.Pay()` check alone does not prevent concurrent requests from both acting on stale state.

Check bypass paths, concurrent execution, retries, and related side effects where relevant. An atomic status transition does not by itself make an external charge or notification idempotent. One source of business truth does not imply one physical validation statement.

### C — Consistency

Inspect how comparable code handles transactions, persistence, errors, state transitions, retries, and module dependencies before introducing a different approach.

For example, preserve a deliberate `repo.MarkAsPaid(ctx, id, version, paidAt)` convention unless there is a concrete reason to replace it with `repo.Save(ctx, order)`. Consistency concerns the underlying decision, not superficial syntax. When conventions conflict, identify which examples are relevant and why; do not infer a repository-wide standard from one file.

## Investigation and Decision Process

### 1. Establish the Behavior and Constraints

Identify the requested behavior, business invariant, affected data, side effects, and any stated failure or consistency requirements. For a review, distinguish changes introduced by the diff from pre-existing issues. For design work without code, use the supplied requirements and mark unverified assumptions.

Do not invent future requirements to justify abstraction. Distinguish a demonstrated source of change from a hypothetical extension.

### 2. Gather Evidence Around the Decision

Start with the changed or proposed capability. Follow relevant callers and callees far enough to establish ownership and observable behavior. When correctness depends on persistence or retries, inspect the transaction boundary, write conditions, error handling, and side-effect ordering.

Search for comparable implementations, tests, and architecture guidance. Read enough context to understand whether a pattern is deliberate, legacy, or transitional. Expand the search when evidence conflicts or a bypass path remains unresolved; stop when the decision is supported rather than auditing the entire repository by default.

Keep these distinctions explicit in the reasoning:

* **Observed:** code, tests, documentation, or supplied requirements that establish a fact.
* **Inferred:** a consequence derived from those facts, with the trigger and reasoning explained.
* **Unknown:** an assumption whose truth would change the recommendation.

Do not infer transaction, retry, or lifecycle semantics from names alone. When a material requirement is unavailable, state the limitation and make the recommendation conditional or ask a focused question. Evidence insufficient to justify redesign is a reason to preserve the current structure, not proof that its behavior is correct.

### 3. Identify the Design Pressure

Use the relevant BRDSC dimensions to explain a concrete pressure: protecting an invariant, isolating independently changing concerns, preserving a dependency boundary, removing duplicated business meaning, or separating confirmed failure and retry requirements.

Introduce an abstraction only when its benefit addresses that pressure and justifies its added indirection and maintenance cost. An established abstraction can be reused without inventing a new one. Do not automatically map a conditional to a strategy, a state change to a state machine, or a cross-module call to an event.

### 4. Choose the Smallest Coherent Change

Prefer explicit code, existing capabilities, and local changes when they satisfy the requirements. **No change** is a valid conclusion. A coordinator owning a workflow or defensive checks repeating part of a rule do not by themselves require refactoring.

When principles conflict, prioritize correctness and data integrity, then ownership and necessary dependency boundaries, then existing conventions and abstraction elegance. Explain the trade-off; do not treat the order as a scoring formula.

Break a convention when concrete evidence shows it causes harm. Identify the replacement principle, the affected paths, and whether the change is local or needs migration. Temporary coexistence can be appropriate when the boundary and migration path are explicit; do not turn a local fix into a system-wide rewrite.

### 5. Verify the Relevant Consequences

For implementation, preserve the requested behavior and verify the properties the design relies on. Choose checks according to the risk: boundary or dependency checks for structural changes; competing writes for concurrency; repeated delivery for idempotency; rollback or failure paths for transactions and side effects. Use existing project checks where appropriate; do not add tests that merely mirror code structure.

For review or design, identify the verification needed to resolve uncertainty. Report what was actually inspected or tested, and distinguish proposed checks from completed ones.

## Output

Lead with the conclusion and adapt the detail to the user's task. Use these elements when useful, without imposing fixed headings on every response:

* **Review:** report actionable findings with a concrete file/symbol location, observed behavior, triggering condition, consequence, relevant principle, and smallest improvement. Distinguish defects from optional design suggestions and pre-existing issues. Rank findings by impact; do not manufacture one for every dimension. If no supported issue remains, say so and state any material coverage limit.
* **Design:** recommend an owner, boundary, and dependency direction; explain the relevant trade-off, assumptions, and why the added complexity is justified. Compare alternatives only when they change the decision.
* **Implementation:** explain the resulting behavior, the scope of the change, and validation performed. Mention unresolved risks that affect the result.

Avoid unsupported labels such as “unclear responsibilities,” “not DDD,” or “use events.” Ground advice in observable consequences. Do not present assumed business requirements as evidence.

## Examples on Demand

Read [references/cases.md](references/cases.md) when judging workflow orchestration, database-enforced rules, a harmful existing convention, or insufficient evidence about asynchronous side effects. The cases are illustrative; their requirements do not establish facts about the user's project.

Before recommending a change, ask: does this improve system coherence, and what evidence supports that conclusion?
