---
title: "Building Borderless: an OS for money movement"
date: 2026-04-21
draft: false
---

A few years ago I sat in an office in Lagos, watching a support queue fill up with the same message in different words. *Did it go through?* *It says sent but nothing has arrived.* *Please help, the rent is due tomorrow.*

The payment had, technically, gone through. An operator somewhere had accepted the debit. A stablecoin had moved from one wallet to another in under a minute. A correspondent bank in a third country had a ledger entry that matched. But the person who pressed the button in London could not see any of that, and the person expecting the money in Lagos had not received it yet, and between those two facts a family was quietly breaking.

I have been in and around payments infrastructure for most of my career — early at Flutterwave, building the first API; later building FX processing at Canary Point; running the B2C remittance product at Korapay; launching digital banks at mKobo and Casha; leading engineering at Rova. In each of those roles, the same pattern repeated. The **rails existed**. The technology to move money cheaply, quickly, compliantly across borders was sitting there, in plain sight. What did not exist was a sensible way for a product team to *plug in* to those rails without rebuilding the world.

That is the gap I am now trying to close with [Borderless](https://borderless.tjoc.dev).

---

## The rails are fine. The access is broken.

This is the part that took me years to admit out loud. The actual physical (and increasingly digital) infrastructure for moving money across borders is, in 2026, genuinely good. Stablecoin settlement on a modern chain takes seconds and costs cents. Tier 1 KYC vendors can verify an identity in under three minutes. Licensed liquidity providers exist in almost every corridor that matters. Compliance frameworks are published. The Wolfsberg questionnaires have templates.

What is *not* fine is what sits between all of those pieces and a product team that wants to use them.

If you are a neobank in Manila, or a payroll platform in Nairobi, or a marketplace in São Paulo, and you decide today that you want to offer cross-border payouts, here is what actually happens:

- You talk to three infrastructure providers. Two of them require $1M annual minimum commitments before they will even give you sandbox access.
- The one that will let you test makes you sign an NDA to read the API docs. The docs are out of date.
- You spend four months negotiating the master services agreement. Then another eight weeks integrating.
- Your compliance officer asks a reasonable question — *who is the responsible person for SAR filings in this corridor?* — and nobody can answer it without a three-week legal review.
- You go live. A payment fails. The status code says `REJECTED`. There is no reason, no retry path, no metadata. You open a support ticket.

Every company I have worked at has been, at some point, in exactly this loop. Everyone rebuilds the same plumbing. Everyone writes their own reconciliation layer because the settlement file shows up without structured metadata. Everyone hires a compliance consultant to answer the same questions that have been answered a hundred times before in the same jurisdiction. And the person waiting for rent money does not care about any of this — they just see *sent* and no arrival.

---

## What I mean by "the OS for money movement"

An operating system does not make your computer faster. What it does is hide the messy, incompatible, stateful complexity of the hardware underneath a clean interface, so that the person writing an application can assume `write_to_disk()` works and spend their time on the thing that actually differentiates their product.

Borderless is that, for money movement.

If you are building a product that needs to move money across a border — whether that is a remittance app, a payroll tool, a marketplace disbursement, a treasury function, a tokenised savings product — you should not have to:

- Negotiate with a liquidity provider in every destination country.
- Build your own KYC flow from scratch for every jurisdiction.
- Invent a reconciliation format because the rail provider gave you a CSV.
- Explain to your compliance officer which obligations sit with you versus your vendor, per corridor, at 11pm on a Sunday.
- Rebuild webhook signing, idempotency, audit trails, or sanctions screening because the provider you chose does not offer them.

You should be able to integrate a single API, get compliant access to stablecoin-native rails with fiat on- and off-ramps in the corridors that matter, receive structured settlement metadata that reconciles cleanly against your ledger, and launch in six to ten weeks — not six to ten months.

Underneath that interface, Borderless handles the actual mess: custodian agreements, liquidity pools, KYC/AML partner orchestration, chain selection, corridor routing, treasury operations, regulatory posture per market. The operator above the line gets to focus on the product their users see. The sender and the receiver get a payment that arrives in under a minute with the fee they were quoted.

---

## Why this, why now

Two things have changed recently that make this version of the problem actually solvable.

First, **stablecoin settlement is boring now**. Five years ago, suggesting that a regulated financial product should use a stablecoin for corridor settlement was a controversial architectural position. Today, the largest remittance corridors in the world are already doing it quietly, behind the scenes, and the regulators are publishing frameworks for it rather than fighting it. The ideological argument is over. What is left is an engineering and compliance integration problem.

Second, **the compliance stack has productised**. Identity verification, transaction monitoring, sanctions screening, travel-rule messaging — these are now SaaS products with APIs and SLAs. What was a twelve-month, seven-figure build in 2020 is a three-vendor integration in 2026. An infrastructure layer like Borderless can sit on top of them and present a single, coherent compliance surface to the operator, rather than asking each operator to re-assemble the puzzle.

The ingredients are on the counter. Nobody has bothered to cook the meal in a way that a normal product team can actually eat.

---

## The part that is personal

I grew up adjacent to every one of these problems. I have family who send and receive money across borders. I have sat in rooms where the phrase *"the rail is down"* meant a real person was not going to make rent. I have watched brilliant product teams burn quarters rebuilding payment plumbing because there was no other option. I have been the engineer on both sides of that failure — the one building the rail, and the one trying to wire it into an app that works.

The thing that keeps pulling me back is how *unnecessary* most of the pain is. The technology exists. The rails work. The standards are published. What is missing is the layer that makes it boringly easy to use them — an operating system, in the older sense of the word. Something you stop noticing, because it quietly does the right thing every time.

So that is what I am building. Not another neobank. Not another remittance app. The layer underneath those things, so the next founder who wants to launch one can stop negotiating with liquidity providers at 11pm and spend their time on what their users actually care about: *did it go through, and will it arrive?*

If you are building something in this space — as an operator, a regulator, a partner, an investor, or just someone who has lived the other side of a failed cross-border payment — I would love to talk. [hello@tjoc.dev](mailto:hello@tjoc.dev).

The rent should just arrive.
