---
title: "Production security: attack surfaces, and the cost of being overlooked"
date: 2026-01-28
draft: false
---

Most production security problems do not start with elite hackers or clever zero‑day exploits. They start with boring stuff. A debug endpoint that never got turned off. A background job that trusts its inputs. A rate limit that was on the roadmap but never shipped.

Everything works fine in staging. Everything works fine with real users at normal scale. Then one day it does not.

A lot of real damage comes from **unintentional exploit angles** and **nuisance attacks**. These are not glamorous. They are low effort, often automated, and they use your system exactly as designed. But once real users, real money, and real attention are involved, the impact is anything but small.

This post looks at production attack surfaces in plain terms, how they quietly grow over time, and why even "annoying" abuse can seriously hurt your app and the people using it.

---

## What an attack surface actually is

An attack surface is not just your login page or your public API. It is every way the outside world can influence your system.

That includes things like:

* Public and private APIs, including the ones you forgot were public
* Auth and permission checks
* Webhooks and third‑party integrations
* Background jobs, queues, and schedulers
* Admin tools and internal dashboards
* Error messages, metadata, and configuration leaks
* Anything that costs money when it runs, like emails, SMS, file processing, or AI calls

In production, attack surfaces grow naturally. New features add new paths. Temporary shortcuts stick around. Internal tools slowly drift outward. None of this is malicious. It is just how software evolves.

The problem is not that the surface exists. The problem is that nobody is actively thinking about how it can be abused.

---

## Unintentional exploit angles

These are the most dangerous problems because nobody feels responsible for them. Nothing looks broken. Everything behaves exactly as expected, right up until someone uses it in a way you did not plan for.

Some common patterns show up again and again.

### Trusting the client too much

* Assuming requests only come from your UI
* Letting the frontend enforce limits
* Expecting parameters to stay within reasonable values

Scripts and bots do not care about your UI. If the backend accepts it, it will get used.

### Auth without real authorization

* Endpoints that check if you are logged in, but not what you own
* Role checks that exist in theory but not everywhere
* Admin functionality hidden by the UI instead of protected by the backend

These often look like edge cases until someone starts enumerating IDs or replaying requests.

### Features that become dangerous when combined

Individually, features may be fine. Together, they can create something you never modeled.

* Free trials mixed with referrals and automation
* Webhooks with retries and weak idempotency
* File uploads feeding async processing pipelines

Security issues often live in the space between features, not inside a single one.

### Cost as an attack vector

Modern apps expose something attackers love. Usage based billing.

If an action triggers emails, SMS, AI inference, file processing, or paid third‑party APIs, it can be abused without ever touching private data. Sometimes the goal is not access. It is to burn money.

---

## Nuisance attacks are not harmless

The phrase nuisance attack makes these sound trivial. They are not.

We have seen this clearly with consumer apps that suddenly go viral. Apps like Tea are a good example. After gaining attention, they saw waves of automated sign‑ups, scraping, harassment style behavior, and coordinated misuse. None of this required breaking into the system. It used normal product features at an abnormal scale.

Some common forms this takes:

* Automated account creation that floods databases and queues
* Scraping that degrades performance or violates user expectations
* Spam and harassment that overwhelms moderation tools
* Login attempts that do not succeed but scare users
* Request floods that stay under DDoS thresholds but still hurt
* Free tier abuse that quietly drives up infrastructure bills

These attacks are cheap to run and easy to repeat.

The consequences are very real:

* Higher infrastructure and vendor costs
* Slower experiences for real users
* Support teams drowning in noise
* Loss of user trust
* Engineers pulled away from product work to do cleanup

Nothing needs to be fully compromised for serious damage to happen.

---

## Why teams miss this stuff

Most teams are not careless. They are busy. A few structural issues make these problems easy to overlook.

### Security treated like a feature

When security is something you add later, it competes with roadmap work and usually loses. In reality, production security is not a feature. It is a property of the system.

### Happy path thinking

Developers optimize for correct usage. Attackers optimize for edge cases, scale, and repetition.

### Weak production feedback loops

Without visibility into abnormal usage or cost spikes, abuse stays invisible until users complain or invoices arrive.

### Belief in obscurity

Internal endpoints, undocumented APIs, and "nobody would find this" assumptions do not hold up. Bots do not need documentation.

---

## Practical ways to shrink the attack surface

You do not need perfect security. You need intentional friction.

Some practical principles help a lot.

### Assume automation

If something can be done once, it can be done a million times. Design limits with that in mind.

### Enforce rules on the backend

Anything important belongs server side.

* Rate limits
* Ownership checks
* Quotas
* Feature gating

### Make abuse visible

* Log abnormal patterns
* Track cost per user or per action
* Alert on spikes, not just outages

### Treat cost like data

Operations that burn money deserve the same protection as sensitive information.

### Review the system as a whole

Every so often, ask a simple question.

If I wanted to misuse this system without breaking in, where would I start

---

## The goal is boring security

The best production security is boring. Nothing dramatic happens. No late night incident calls. No heroic recoveries.

Just systems that fail safely, limit damage, and make abuse expensive.

Attackers do not need clever ideas. They just need one small thing you forgot to think about.

In production, small oversights scale fast. Good security is not about paranoia. It is about respecting how real systems behave once they meet the real world.
