---
title: "Designing an idempotent payment endpoint (the part most tutorials skip)"
date: 2026-06-01
draft: false
---

## Idempotency in Real Systems

If you build or work on payment systems, you've almost certainly seen a version of this.

An endpoint is clean. Tests are green. Then a customer's phone drops to one bar mid-checkout and they get charged twice.

The request timed out on their end, the client retried, and the server happily ran the whole charge again because, as far as it knew, this was a brand-new request.

That's the bug almost everyone ships once.

The network timeout: the charge succeeded, but the response never made it back, so the client retries.

The webhook replay: your provider re-delivers an event because it didn't get a 2xx fast enough, so you process the same payment twice.

And the impatient double-click: a user mashes the pay button because the spinner is slow, and two requests land milliseconds apart.

The honest framing is that a payment request is not "do this thing."

It's:

> Make sure this thing has happened exactly once.

## The Trap: Check Then Act

The first instinct is always the same:

> Before charging, look it up. If no charge exists for this order, create one.

This feels safe.

It is not.

It's a classic check-then-act race condition.

Two requests arrive at nearly the same time. Both look up the charge. Both see nothing because neither has written yet. Both proceed to create the charge.

You can't fix a concurrency bug with logic that reads, then decides, then writes. The read and the write aren't atomic, and the gap between them is where the money leaks.

## Idempotency Is More Than "Don't Run It Twice"

The fix is an idempotency key: a single token that identifies a specific operation so that a retry is recognised as the same operation.

The contract has two halves, and skipping the second is the most common mistake I see.

### 1. A Request Fingerprint

Store the key together with enough of the request payload to detect misuse.

If someone reuses:

* The same key
* A different amount
* A different recipient

The request should fail loudly.

### 2. A Stored Response

When the operation completes, persist the result against that key.

When a retry arrives, return the stored result instead of executing the operation again.

This distinction matters.

Idempotency isn't just:

> Don't run it twice.

It's:

> Give the same answer twice.

## Client-Supplied vs Server-Derived Keys

There are two common approaches.

### Client-Supplied Keys

The client generates a UUID and sends it in an `Idempotency-Key` header.

This is the model Stripe uses.

The advantage is that retries from the same logical attempt carry the same key, even when failures occur before the server can respond.

The trade-off is that you trust clients to generate keys correctly.

### Server-Derived Keys

The server derives a key from fields such as:

* Order ID
* Amount
* Time window

This removes dependence on the client but introduces the possibility of collisions between legitimate payments.

For public APIs, I generally prefer client-supplied keys with server-side fingerprint validation as the backstop.

## Let the Database Be the Source of Truth

The part that makes idempotency durable is surprisingly simple:

Don't enforce uniqueness in your application.

Enforce it in the database.

Create a unique constraint on the idempotency key.

The database—not your code and not your luck with timing—guarantees that two rows with the same key cannot exist.

Instead of asking:

> Does this operation already exist?

Attempt the insert and let the constraint answer the question.

If the insert raises a unique-violation, the operation already exists.

Check-then-act has a race window.

Insert-and-catch does not.

The uniqueness decision happens atomically inside the database under its own locks.

## The Edge Case Most Tutorials Skip

Consider two concurrent requests with the same idempotency key.

One wins the insert.

The other receives a unique-violation.

Many tutorials say:

> Return the stored response.

But what if the winner hasn't finished yet?

The row exists, but the operation is still in progress.

The losing request can't return a response that doesn't exist.

You need three distinct states:

### In Progress

The operation exists but hasn't been completed.

Options include:

* Polling briefly
* Returning HTTP 409
* Asking the client to retry

### Completed

Return the stored response verbatim.

### Fingerprint Mismatch

The caller reused a key with a different payload.

Return HTTP 422.

Anything else risks hiding a bug.

The winning request performs its work and updates the idempotency record to completed in the same transaction that performs the charge.

If the charge commits but the idempotency update doesn't, you're back where you started.

## Closing the Most Dangerous Gap

I'm not theorising here.

I built the first APIs at Flutterwave and was the founding CTO at Korapay. Idempotency and reconciliation aren't whiteboard topics for me. These are problems I've had to solve while real money was moving, and mistakes meant customers being charged twice or ledgers refusing to balance.

I'm carrying the same discipline into ConchPay, the payment processor I'm currently designing.

The architectural decision I've landed on is simple:

> The idempotency record and the ledger entry live in the same transaction.

The unique-key write isn't a layer bolted in front of the payment flow.

It is part of the payment flow.

The most dangerous failure mode in payments is the gap between:

> "I charged the card."

and

> "I recorded that I charged the card."

The only way I trust to eliminate that gap is to make them the same commit.

Keys carry a TTL and are scoped per merchant.

## The PR Review Checklist

Before approving a payment endpoint, verify all seven:

* A unique constraint in the database on the idempotency key
* Insert-and-catch, not check-then-insert
* Charge and idempotency record committed in the same transaction
* Stored responses returned on retries
* In-progress requests handled separately from completed requests
* Fingerprint mismatches failing loudly with a 4xx response
* Keys scoped per merchant and expired intentionally via TTL

Tick all seven, and your endpoint survives the timeout, the replay, and the double-click.

## About Me

I built Flutterwave's first APIs and was Korapay's founding CTO. I've migrated core-banking systems and shipped cross-border banking applications.

Today, I build payment systems for fintech founders.

If your company depends on the layer where money moves, that's the layer I work on.
