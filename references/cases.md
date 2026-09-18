# BRDSC Decision Cases

These are synthetic examples with explicit requirements, not reports from a real repository. Use the reasoning relevant to the task; do not assume that a project's similarly named functions have these semantics.

## Workflow Coordination Can Remain Together

### Context

A requirement says that accepting an order reserves stock and records a coupon redemption atomically. Both stores use the same database transaction. Their operations enforce their own rules, and comparable application services coordinate capabilities in the same way.

```go
func (s *CheckoutService) Accept(ctx context.Context, cmd AcceptCommand) error {
    return s.tx.Within(ctx, func(tx Tx) error {
        if err := s.stock.Reserve(ctx, tx, cmd.Items); err != nil {
            return err
        }
        if err := s.coupons.Redeem(ctx, tx, cmd.CouponID); err != nil {
            return err
        }
        return s.orders.Accept(ctx, tx, cmd.OrderID)
    })
}
```

### Judgment

No boundary refactor is justified by the number of capabilities. The service owns workflow coordination and delegates stock and coupon semantics to their owners. Making these steps asynchronous would change the stated atomicity requirement.

Verify rollback and each operation's enforcement if those properties are in question. Recommend a change only when evidence shows a violated requirement, leaked responsibility, or another concrete pressure.

## A Repository Operation Can Own a Transition

### Context

The project exposes `MarkAsPaid` as its shared pending-to-paid capability. All payment entry points use it. The database evaluates the following predicate atomically, and the repository reports success only when exactly one row changes; it maps zero rows to the project's conflict result.

```sql
UPDATE orders
SET status = 'paid', paid_at = :paid_at, version = version + 1
WHERE id = :id AND status = 'pending' AND version = :version;
```

### Judgment

The capability has one semantic contract and uses SQL for atomic enforcement. Adding `Order.Pay()` or switching to `Save(order)` is not required to satisfy Single Authority. Two competing calls with the same version cannot both successfully transition the row under the stated database semantics.

Inspect callers' handling of conflicts and any alternate write paths. This update says nothing about whether a payment provider was charged twice before the write, or whether downstream work can safely retry. Investigate those only when relevant to the task.

## Correctness Can Require Breaking a Convention

### Context

The requirement forbids negative available inventory. Existing handlers read stock, check availability in memory, then decrement the database value without a predicate or transaction lock. Concurrent requests are supported. Each request may independently pass the check against the same available quantity.

```go
available, err := repo.Available(ctx, sku)
if err != nil {
    return err
}
if available < qty {
    return ErrInsufficientStock
}
return repo.Decrement(ctx, sku, qty)
```

`Decrement` executes `UPDATE stock SET available = available - :qty WHERE sku = :sku`. Assume requested quantities are validated as positive.

### Judgment

For available stock 1, two requests of quantity 1 can both pass the read check and then leave stock at -1. The concurrency trace establishes a correctness issue even if every current handler follows this convention.

Use the project's persistence mechanism to enforce the invariant atomically, for example a decrement conditioned on `available >= :qty` with checked affected-row count, or suitable locking. Prefer placing that enforcement in the shared decrement operation when its callers share this contract. Inspect and migrate bypassing writes within the required scope; identify any remaining paths explicitly rather than claiming the invariant is fully protected.

Verify competing decrements, insufficient stock, and unchanged valid behavior. A strategy hierarchy, event bus, or system-wide architecture rewrite does not address the demonstrated race.

## Side Effects Require Evidence About Failure Semantics

### Incomplete Context

```go
if err := s.orders.MarkAsPaid(ctx, id); err != nil {
    return err
}
return s.notifications.SendReceipt(ctx, id)
```

The snippet alone does not establish commit timing, receipt guarantees, retry behavior, or whether `SendReceipt` performs delivery or queues durable work.

### Judgment with Only the Snippet

State those limits. Inspect the implementations, caller, and requirements when available. Do not report different lifecycle or retry semantics as a fact or recommend events solely because two capabilities appear together.

### Judgment with Additional Evidence

Suppose investigation establishes that:

* `MarkAsPaid` commits before returning and rejects repeat transitions.
* `SendReceipt` directly calls a mail provider and can time out.
* Caller retries restart the whole workflow; no receipt retry or reconciliation exists.
* The requirement is eventual receipt delivery after a committed payment, independent of a provider outage.
* An existing durable job mechanism can record receipt work in the same database transaction as the transition; it supports retries and a stable deduplication key.

Now there is an actionable failure: a timeout after commit leaves a paid order without guaranteed receipt delivery, and retrying the workflow stops at the repeated transition. Recommend recording receipt work with the transition through the existing mechanism and retrying delivery independently.

Verify atomic recording, rollback, worker retries, and duplicate handling. State the delivery guarantee supported by the provider and job mechanism; recording one job alone does not guarantee exactly-once external delivery. This recommendation depends on the additional evidence, not on the function names.
