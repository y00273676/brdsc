# BRDSC

**BRDSC** is an engineering judgment framework for keeping code and design coherent as a system.

* **B — Boundary**
* **R — Responsibility**
* **D — Dependency**
* **S — Single Authority**
* **C — Consistency**

It is intended for engineers and repository-aware coding agents making design, review, and refactoring decisions. It does not prescribe DDD, Clean Architecture, events, repositories, or another architectural style.

## Why BRDSC?

A codebase can work correctly while becoming harder to change: similar rules live in different places, ownership shifts between layers, and each feature introduces another way of solving the same problem.

BRDSC asks what structure serves the current system and its requirements. It also checks whether a proposed abstraction adds enough value to justify its complexity.

The central question is:

> Does this change improve system coherence, and what evidence supports that conclusion?

## The Five Dimensions

| Dimension | What to establish |
| --- | --- |
| Boundary | Which concepts need separate ownership, lifecycle, or consistency boundaries? |
| Responsibility | Who owns each behavior, and where would the next similar change belong? |
| Dependency | Which relationships are intentional, and which expose internals or force unrelated changes? |
| Single Authority | Who defines each business rule, and how is it enforced across entry points and concurrent execution? |
| Consistency | How does comparable code solve this problem, and is there a concrete reason to differ? |

These are reasoning aids. Do not score the dimensions or require every review to discuss all five.

Two distinctions matter in practice:

* Coordinating several capabilities can be a valid application-service responsibility. Different business nouns or multiple calls do not alone justify splitting a workflow.
* A rule's semantic owner and its atomic enforcement can span a business capability and a database operation. Conditional updates and constraints can be essential enforcement. Single Authority does not require moving SQL checks into entity methods or removing defensive validation.

## Using the Skill

[SKILL.md](SKILL.md) is the agent entrypoint. Its metadata describes when BRDSC applies; its body contains the principles, investigation process, and output expectations.

Use it when a task involves module boundaries, responsibility placement, dependencies, shared business rules, or a meaningful abstraction trade-off. Formatting and mechanical changes do not need a BRDSC review.

Example requests after installing the skill in your agent's supported skill location:

```text
Use $brdsc to review this PR's transaction boundary and business-rule ownership.

Use $brdsc to decide where refund eligibility should live, using the existing
refund and cancellation flows as context.

Implement this change using $brdsc to preserve the project's existing
dependency and persistence conventions.
```

A review request produces findings and recommendations. An implementation request applies the framework within the requested change. Neither implies a repository-wide architecture overhaul.

## How a Decision Is Made

1. Establish the behavior, invariant, side effects, and relevant requirements.
2. Inspect the changed capability, its callers and callees, and comparable implementations.
3. Separate observed facts, inferred consequences, and material unknowns.
4. Identify the actual design pressure and choose the smallest coherent change.
5. Verify the relevant consequences and report what was actually checked.

Recommendations should point to concrete code or supplied requirements and explain the trigger, consequence, and scope of a problem. Unsupported claims about lifecycle, retry behavior, or future requirements are not evidence.

“No change” is a valid result. When information is incomplete, state the limitation and make advice conditional. Preserving the current structure does not establish that its behavior is correct.

Existing conventions are valuable, but a demonstrated correctness or integrity problem can justify breaking them. Explain the replacement principle and affected paths; where migration is necessary, make temporary coexistence explicit.

## Worked Cases

The [decision cases](references/cases.md) are synthetic examples with stated requirements:

* [Workflow coordination](references/cases.md#workflow-coordination-can-remain-together): retaining one coordinator for capabilities that must commit together.
* [Database enforcement](references/cases.md#a-repository-operation-can-own-a-transition): keeping a conditional state transition in a shared repository operation.
* [Changing a harmful convention](references/cases.md#correctness-can-require-breaking-a-convention): correcting a race even though nearby code uses the same unsafe pattern.
* [Incomplete evidence](references/cases.md#side-effects-require-evidence-about-failure-semantics): distinguishing an unexplained sequence of calls from a demonstrated delivery and retry problem.

They illustrate how different evidence leads to different recommendations. They are not assumptions about the code under review.

## Repository Structure

```text
brdsc/
├── README.md
├── SKILL.md
└── references/
    └── cases.md
```

Keep essential instructions in `SKILL.md`. Read `references/cases.md` only when its decision cases are relevant.

## Validating Changes to the Skill

Check metadata and reference links after editing. For changes to the reasoning process, also try realistic requests with enough raw code and requirements to judge the response.

Useful evaluation cases include a sound design that needs no refactoring, a harmful established convention, a database-enforced invariant, and a snippet with insufficient context. Evaluate whether recommendations follow the supplied evidence, preserve task scope, and distinguish proposed checks from completed tests.

Format validation alone does not establish that the skill makes good engineering judgments.
