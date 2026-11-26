---
name: gate-data-integrity
description: "data integrity gate", a review of race conditions, atomicity or persistence on commits. The data integrity checks of a stage review, used by the gate-data-integrity subagent.
---

# Gate: data integrity

This gate checks what happens to the data when two requests arrive at the same time, when a step fails halfway, and when the process restarts. It checks race conditions, atomicity and persistence. It does not check the design, the security or the API contract: other gates do.

## Running this gate

- You are the `gate-data-integrity` subagent: go to the next section. Your prompt says how you work and how you report.
- You are in the main session, and the user asked for this gate: do not review the commits yourself. The agent that wrote the work does not review it. Send the `gate-data-integrity` subagent with `task`. Give it the north star (the stage's, or the need in one line), the base commit (the stage's, in the notebook; without one, `git merge-base HEAD <default branch>`), and the commits since. Give the user your synthesis of its report, not the raw report. An issue the user wants fixed is a new commit, in the normal cycle; never amend.

## Doctrine

- Every write to shared state is read as if two requests run it at the same time, and as if the process stops between any two lines.
- What concurrent requests must produce is a domain decision, and it belongs to the user. "Two refunds of the same order at the same time: one succeeds, the other gets a conflict" is a rule the user decides. When the stage did not decide it, the issue is a question, not a fix.
- A concurrency claim is proven only by a test that overlaps two requests through the real application boundary, on the same database engine as production. A unit test with an in-memory adapter does not prove it. When this proof is missing, say so, and name the test the user should have.
- The database guarantees what Python code cannot: a unique constraint, a transaction, a row lock, an atomic `UPDATE`. A check done in Python before a write is not a guarantee.

## What you check

On each commit, look at every line that writes: an ORM `save()`, `add()`, `commit()`, `flush()`, an `INSERT`, `UPDATE`, `DELETE`, a file write, a cache write, a message published.

1. **Lost update.** A value is read, changed in Python, then written back: `order.balance -= amount; session.commit()`. Two requests read the same value; the second write erases the first. Look for an atomic update (`UPDATE ... SET balance = balance - :amount WHERE ...`), a row lock (`SELECT ... FOR UPDATE`, `with_for_update()`, `select_for_update()`), or a version column checked at write time.
2. **Check, then act.** The code checks a condition, then acts on it: `if not refund_exists(order_id): create_refund(...)`. Two requests both pass the check. Look for a unique constraint in the database, and for the code that handles its violation (`IntegrityError`) as a domain outcome, not as a crash.
3. **Atomicity.** Writes that must succeed or fail together are in one transaction. One use case is one transaction. Look for a `commit()` in the middle of a use case, a repository that commits on its own, or two sessions used in one use case. If the process stops between two writes, the data must stay valid.
4. **Side effects inside a transaction.** An HTTP call, an email, a payment, a message published while the transaction is open. If the transaction rolls back, the effect has already happened. If the effect runs after the commit and fails, the database says done and the world does not. Name the pattern that fits, as a proposal: the effect after the commit with a retry, or a transactional outbox.
5. **Idempotency.** A request that can arrive twice: a webhook, a message consumer, a retried `POST`, a job run again after a crash. Processing it twice must not refund twice. Look for an idempotency key stored with a unique constraint, or a state check done inside the same lock or transaction as the write.
6. **Process memory used as storage.** A module-level variable, a class attribute, a dictionary used as cache, `functools.lru_cache` on data that changes. It is lost at restart and not shared between workers or between instances. With `async`, shared mutable state changes between two `await`.
7. **Persistence.** What is written is really persisted: the session is committed, the file is closed, the write is not lost in an exception path. What is read back has the same shape and type as what was written: money as `Decimal` or as integer cents, never `float`; dates with their timezone.
8. **Schema changes.** A schema change comes with its migration, in the same commit. The migration works on a table that already has rows: a new `NOT NULL` column needs a default or a backfill. A destructive change (drop a column, rename it, change its type) breaks the code of the commit before it: look for expand, then contract, across commits. A migration on a large table can lock it: say so when you see it.
9. **Consistency boundary.** One transaction changes one aggregate (the cluster of data that must stay consistent together, named with the user in the plan). A change that writes two aggregates in one transaction, or one aggregate across two transactions, is reported with the invariant it puts at risk.

## Examples of issues

```
a1b2c3d feat(refunds): refund an order
  issues:  ⚠ refunds/service.py:42 reads `order.refunded`, then sets it and commits. Two refund
           requests at the same time both read False: the customer is refunded twice.
           A unique constraint on refunds.order_id, or a row lock, would stop it.
           Question for the user: the second request gets 409 Conflict, or the first refund again?
  limits:  no test overlaps two requests on PostgreSQL; the unit test uses the in-memory repository.

e4f5a6b feat(refunds): notify the customer
  issues:  refunds/service.py:58 sends the email before `session.commit()`. If the commit fails,
           the customer is told about a refund that does not exist.
```

## Not in this gate

- Who may call the code, and the input it trusts: `gate-security`.
- Where the transaction code sits in the layers: `gate-design`.
- The status code of a conflict: `gate-api`.
