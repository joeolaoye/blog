---
title: "From UUIDs to Snowflakes: how IDs grow up with your infrastructure"
date: 2026-04-19
draft: false
---

The first time you pick an ID strategy for a new service, it does not feel like a decision. You reach for whatever the framework gives you. An auto‑increment integer. A UUID v4. Maybe a `timestamp_uuid` composite because you saw it in a tutorial once.

It works. Everything works. You ship the feature, close the ticket, and move on.

Then the service grows up. A second server joins the cluster. A second region gets provisioned. Support starts asking why record `a47f2b9e‑...` came before record `b01c3d8a‑...` in the logs but after it in the database. And your ID strategy, which felt like a non‑decision a year ago, is suddenly on the whiteboard in a post‑incident review.

This is the path almost every system walks, whether you plan for it or not.

---

## Stage 1: one server, one database, one ID column

At the start, you do not need anything fancy.

If you are on Postgres or MySQL, the default is probably a `BIGSERIAL` or `AUTO_INCREMENT`. The database hands out sequential integers, they fit nicely in an index, and they sort themselves in insertion order. Easy.

If you wanted to avoid coupling your IDs to the database, you reached for UUID v4. Random 128 bits, generated anywhere, no coordination needed. Collisions are mathematically possible but practically impossible at small scale.

Some teams split the difference and do something like `{unix_ms}_{uuid}` or `{date}_{slug}`. Sortable‑ish, globally unique‑ish, human‑readable. Good enough.

All three work. On a single server, with a single writer, they all behave well. There is exactly one source of truth, and that source is either the database or a local clock nobody else is racing.

This stage feels fine because it *is* fine. The problems do not exist yet.

---

## Stage 2: the cracks

Add a second writer and the assumptions start to leak.

### Auto‑increment stops scaling

Auto‑increment IDs are cheap because the database is the coordinator. Once you have two databases, or a sharded write path, or you want to generate IDs before the row exists, the database stops being a convenient single point of truth and becomes a bottleneck.

People work around this with centralized sequence services, ID reservation batches, or dedicated "ticket server" tables. All of those turn ID generation into a network call. You are now paying a round trip per insert just to get a number.

### UUID v4 destroys your indexes

This one bites teams later than it should.

UUID v4 is random. When you use it as a primary key in a B‑tree index, every insert lands in a random place in the tree. Pages split, cache locality collapses, and write throughput falls off a cliff once the index no longer fits in memory.

You will see it first as degraded insert latency under load, then as mysterious IO spikes, then as a migration to "UUID v7" or a custom prefixed format written by someone who read a blog post.

### Time‑ordered IDs stop being time‑ordered

If you built a `{timestamp}_{uuid}` scheme, it was implicitly relying on one clock. Add a second server and you inherit clock skew. NTP keeps machines close, but "close" can still mean tens of milliseconds apart, and occasionally seconds after a bad sync.

Now your "sortable" IDs are only sortable *within a single host*. Across the fleet, order is approximate. In logs, in analytics, in debugging, that approximation leaks everywhere.

### Debuggability quietly gets worse

A bare UUID tells you nothing. Not when it was created. Not where. Not what shard. When a customer emails you with an ID, you have to look it up in three systems just to figure out which service owns it.

You can live with this for a long time. You should not want to.

### The coordination tax

At the root of all of this is one question: *who decides what the next ID is?*

On a single box, the answer is trivial. In a distributed system, every answer costs something. A central sequence service is a single point of failure. Reserving ID batches adds complexity and wastes IDs on restarts. Random UUIDs skip coordination but pay in index performance and lost order.

You want an ID scheme where every worker can generate IDs independently, the IDs still sort roughly by time, and collisions are structurally impossible. That is exactly the problem Twitter had in 2010.

---

## Stage 3: Snowflake IDs

Twitter's Snowflake was not a novel idea so much as a clean packaging of one. It fits a usable distributed ID into 64 bits, which matters because 64 bits is still a `BIGINT` and fits natively in every database, language, and index you already use.

The layout looks like this:

* **1 bit** — unused, reserved as a sign bit so the number stays positive
* **41 bits** — timestamp in milliseconds since a custom epoch
* **10 bits** — machine or worker ID
* **12 bits** — per‑millisecond sequence number

That is the whole design. From those four fields you get:

* **K‑sortability.** Because the timestamp sits in the high bits, IDs generated later sort after IDs generated earlier, roughly. Good enough for indexes, logs, and pagination.
* **No coordination per insert.** Each worker generates its own IDs from its local clock, its own worker ID, and a local counter. Nothing over the network.
* **4096 IDs per millisecond per worker.** That is 4 million per second per worker before you need to slow down or borrow from the next millisecond.
* **1024 workers.** Plenty of room for reasonable fleets, tight for very large ones.
* **~69 years of timestamp range** from your chosen epoch. Pick the epoch wisely.

The write pattern into a B‑tree is *almost sequential*, which is what you actually want. Hot pages stay hot, cold pages stay cold, and the index behaves.

There are variants worth knowing about, but they are adjacent, not replacements.

* **Sonyflake** shifts the bit allocation to trade per‑ms throughput for more workers and a longer lifetime.
* **Instagram's sharded IDs** bake the shard ID into the key so routing is free.
* **ULID** and **KSUID** drop the bit packing entirely and use a text‑friendly, lexically sortable format. Nicer for humans, larger on disk, not a drop‑in `BIGINT`.

Pick based on what matters. If you want sortable IDs that pass for opaque strings in a URL, ULID is lovely. If you want a 64‑bit integer that indexes well and generates fast, Snowflake is the default for a reason.

---

## Gotchas engineers still trip on

Snowflake does not remove problems. It moves them.

### Clocks can go backwards

NTP adjusts machine clocks. Most of the time it slews them smoothly. Sometimes it jumps them. Leap seconds, VM migrations, and bad hypervisors can all hand you a timestamp lower than the last one you issued.

The original Twitter implementation refuses to generate IDs until the clock catches up. That is the safe default. Silently letting the clock rewind breaks uniqueness. Some teams add a small buffer, pause for the skew to pass, and log loudly. Whatever you do, *do not ignore it*.

### Worker ID assignment is the real problem

Ten bits gives you 1024 workers. The hard part is making sure no two processes ever pick the same one.

Common approaches, roughly in order of operational pain:

* Hard‑coded in config. Fine for small fleets, fragile everywhere else.
* A Zookeeper or etcd ephemeral lock that hands out the next free ID.
* A database table with a row per worker, taken on startup.
* Kubernetes pod ordinals from a StatefulSet. Clean, but only works if your workload actually fits a StatefulSet.
* Hashing the hostname or IP into the worker ID space. Easy, but collisions are silent and terrible.

This is the part of Snowflake nobody writes tutorials about and everyone has to solve.

### 2080 is closer than it looks

41 bits of milliseconds from a 2010 epoch runs out somewhere around 2079. If your system is going to live that long, someone will be paged at 3am to migrate. Pick a sensible custom epoch now, document it, and move on.

### Public IDs leak information

A Snowflake ID in a URL tells the world your timestamp, your worker count, and your per‑millisecond throughput. For internal systems that is fine. For public endpoints, you are handing competitors a free dashboard. Either encode them, hash them, or use a separate public slug.

### Do you actually need this?

The honest answer is often no.

If you have one database, one region, and realistic growth, `BIGSERIAL` plus a slug column will carry you much further than Twitter scale content suggests. Reach for Snowflake when you can name the problem it solves for you — not because the architecture diagram looks nicer with it.

---

IDs are one of those parts of a system that feel like plumbing right up until they are the whole story. The right ID in the right place disappears. The wrong one shows up in every incident review for a year.

Pick the simplest scheme that survives your next order of magnitude. Then be ready to grow up when it does not.
