---
title: "Designing an idempotent payment endpoint (the part most tutorials skip)"
date: 2026-06-01
draft: false
---

# Idempotency: The Payment Bug Almost Everyone Ships Once

If you build or work on payment systems, you've almost certainly seen a version of this.

An endpoint is clean. Tests are green. Everything looks good.

Then a customer's phone drops to one bar mid-checkout, the request times out, and they get charged twice.

The charge succeeded, but the response never made it back. The client retries, and the server happily executes the charge again because, as far as it knows, this is a brand-new request.

That's the bug almost everyone ships once.

## The Three Ways Duplicate Payments Happen

### 1. The Network Timeout

The charge succeeds, but the response never reaches the client.

The client assumes the operation failed and retries.

### 2. The Webhook Replay

Your payment provider doesn't receive a `2xx` response quickly enough and re-delivers the same event.

Your system processes the payment again.

### 3. The Double-Click

The user gets impatient.

The spinner feels slow.

They hit the Pay button twice.

Two requests arrive milliseconds apart.

---

The honest framing is this:

> A payment request is not "do this thing."
>
> A payment request is "make sure this thing has happened exactly once."

## The Trap: Check-Then-Act

Most engineers start with the same solution:

```text
Before charging:
1. Look up the order
2. If no charge exists, create one
```

It feels safe.

It isn't.

This is a classic **check-then-act race condition**.

Imagine two requests arriving almost simultaneously:

```text
Request A → checks → no charge found
Request B → checks → no charge found

Request A → creates charge
Request B → creates charge
```

Both reads happen before either write completes.

Both requests conclude they should proceed.

Both create charges.

Money leaks through the tiny gap between the read and the write.

You cannot fix a concurrency problem with logic that:

1. Reads
2. Decides
3. Writes

The read and write are not atomic.

That gap is where duplicate charges are born.

## The Real Fix: Idempotency Keys

The solution is an **idempotency key**.

A single token that identifies a specific operation.

If a request is retried, the system recognizes it as the same operation rather than a new one.

An idempotency implementation has two parts.

### Part 1: Request Fingerprint

Store:

* The idempotency key
* Enough of the request payload to identify what was intended

This catches misuse such as:

```text
Same key
Different amount
Different recipient
```

That should fail loudly.

### Part 2: Stored Response

When the operation completes:

* Persist the result against the key
* Return that stored result on future retries

This distinction matters.

Idempotency isn't merely:

> "Don't execute twice."

It's:

> "Return the same answer twice."

## Client-Supplied vs Server-Derived Keys

There are two common approaches.

### Client-Supplied Keys

The client generates a UUID and sends it via an `Idempotency-Key` header.

This is the approach used by Stripe.

**Advantages**

* Retries preserve the same key automatically
* Works even if the first request never reaches the server

**Trade-offs**

* You trust clients to generate keys correctly

### Server-Derived Keys

The server derives a key from values such as:

```text
Order ID + Amount + Time Window
```

**Advantages**

* Less dependence on client behavior

**Trade-offs**

* Legitimate payments can collide
* Harder to distinguish separate intended transactions

For public APIs, I generally prefer client-supplied keys combined with server-side fingerprint validation.

## The Most Important Rule: Let the Database Decide

Do not enforce uniqueness in application code.

Enforce it in the database.

Create a unique constraint:

```sql
UNIQUE(idempotency_key)
```

Then stop asking:

> "Does this already exist?"

Instead:

```text
Try to insert.
If it fails, the operation already exists.
```

The database becomes the source of truth.

Not your application.

Not timing.

Not luck.

The difference is subtle but critical.

### Check-Then-Act

```text
Read
Decide
Write
```

Contains a race window.

### Insert-and-Catch

```text
Insert
Catch unique violation
```

No race window.

The uniqueness decision happens atomically inside the database.

## The Edge Case Most Tutorials Skip

Consider two requests with the same key arriving simultaneously.

### Request A

Wins the insert.

### Request B

Hits the unique constraint.

Many tutorials stop here and say:

> "Return the stored response."

But there may not be a stored response yet.

Request A is still processing.

The row exists, but the operation is not finished.

You need three distinct states.

### In Progress

The operation exists but hasn't completed.

Options:

* Poll briefly
* Return `409 Conflict`
* Ask the client to retry

### Completed

The operation finished successfully.

Return the stored response verbatim.

### Fingerprint Mismatch

The same key is being reused for a different request.

Return `422 Unprocessable Entity`.

Anything else risks hiding a bug.

## Making It Truly Durable

The winning request must:

1. Execute the charge
2. Update the idempotency record
3. Commit both together

If the charge commits but the idempotency update fails, you're back where you started.

The dangerous gap is:

```text
Money moved
Record missing
```

That's how reconciliation nightmares begin.

## Lessons from Building Payment Systems

I'm not theorizing here.

I built the first APIs at Flutterwave and served as founding CTO at Korapay.

Idempotency and reconciliation aren't whiteboard topics for me. They're problems that have to work when real money is moving and mistakes mean:

* Customers charged twice
* Merchants losing trust
* Ledgers refusing to balance

I'm carrying the same discipline into ConchPay, the payment processor I'm currently designing.

The architecture decision I've settled on is straightforward:

> The idempotency record and the ledger entry live in the same transaction.

The unique-key write is not a protective layer sitting in front of the payment flow.

It is part of the payment flow.

The failure mode I trust least is:

```text
I charged the card.
But I failed to record that I charged the card.
```

The only reliable way to eliminate that gap is to make both operations part of the same commit.

Additionally:

* Keys are scoped per merchant
* Keys have explicit TTLs
* Expiration is intentional, not accidental

## Your Payment API Review Checklist

Before approving a payment endpoint, verify all seven:

* [ ] Unique database constraint on the idempotency key
* [ ] Insert-and-catch, not check-then-insert
* [ ] Charge and idempotency record committed in the same transaction
* [ ] Stored response returned on retries
* [ ] In-progress state handled separately from completed
* [ ] Fingerprint mismatches fail loudly with a 4xx response
* [ ] Keys are scoped per merchant and expire via TTL

If all seven are present, your endpoint is far more likely to survive:

* Network timeouts
* Webhook replays
* User double-clicks

## Final Thought: IDs Matter More Than They Look

IDs are one of those parts of a system that feel like plumbing right up until they become the entire story.

The right ID disappears into the background.

The wrong one shows up in every incident review for the next year.

Pick the simplest scheme that survives your next order of magnitude.

Then be ready to evolve it when it doesn't.
