+++
title = "AI amplifies your habits — the good ones and the bad ones"
date = 2026-09-08
draft = false
description = "AI does not give a team or developer better habits. It makes their existing habits faster, cheaper, and more visible."
author = "Joseph Olaoye"
tags = ["AI", "Engineering Leadership", "Teams", "Developer Productivity"]
categories = ["AI", "Software Engineering"]
+++

# AI amplifies your habits — the good ones and the bad ones

The most useful way I have found to think about AI at work is not as automation, replacement, or even intelligence.

It is an amplifier.

An amplifier does not decide whether the signal is good. It makes the signal louder.

If a developer has a habit of writing down the problem, checking assumptions, and testing the result, AI lets them run that loop more often. If they have a habit of accepting the first plausible answer and postponing the difficult part, AI lets them produce more polished-looking versions of the same avoidance.

The same is true for a team.

AI can turn a thoughtful engineering organisation into one that learns at an unusual speed. It can also turn a confused organisation into one that creates tickets, documents, code, dashboards, and messages faster than anyone can make sense of them.

The difference is rarely the model.

It is the habit being amplified.

## The good habits AI compounds in a team

### 1. Clear problem framing

Teams that start with a concrete problem get a disproportionate benefit from AI.

Not a vague request like “improve onboarding.” A useful frame has a user, an observed failure, a constraint, and a measurable change:

> New users abandon the setup flow after connecting their data source. Reduce that drop-off without making the setup longer.

That kind of framing gives an AI workflow something real to work with. It can inspect the relevant code paths, suggest instrumentation, generate test cases, find edge cases, and propose a sequence of experiments.

Without it, the workflow produces a larger cloud of possibilities. The team still has to decide what the problem was.

### 2. Small, reviewable changes

AI is exceptionally good at accelerating the mechanical parts of a well-scoped change: locating call sites, drafting migrations, updating tests, documenting a new interface, or making a repetitive refactor.

This works best when a team already prefers small pull requests with an explicit contract.

For example:

- A developer defines the new idempotency rule for a payment endpoint.
- An agent traces every write path and drafts the schema migration and tests.
- A reviewer checks the business invariant: one logical payment must produce one charge and one stored response.

The AI makes the implementation loop faster. The human-owned invariant keeps it safe.

### 3. Written decisions and durable context

Teams that record why they chose something get more value from AI than teams that only preserve the final code.

An architecture decision record, a concise incident review, or a good `AGENTS.md` gives a workflow constraints it can actually honour. It reduces the chance that an agent optimises a local task while undoing a decision made elsewhere.

This is one reason documentation becomes more valuable in an AI-native organisation, not less. It is executable context for people and tools.

### 4. Measurement before opinion

Healthy teams already ask, “What happened?” before asking, “Who has the best explanation?”

AI can help pull together logs, traces, support conversations, conversion data, and cost data. It can summarise patterns that would take a person hours to assemble.

But the habit that matters is still measurement.

The useful loop is:

```text
Observe → form a hypothesis → change one thing → measure → learn
```

AI can shorten every step. It cannot rescue a team that treats a fluent summary as evidence.

### 5. Generous knowledge sharing

When people leave useful traces of their work—good commit messages, decisions, runbooks, examples, and postmortems—AI can make that knowledge easier to retrieve and apply.

The best outcome is not that one senior engineer becomes ten times faster. It is that a new engineer can understand the system earlier, ask sharper questions, and make a safe contribution sooner.

That is compounding organisational learning.

## The bad habits AI compounds in a team

### 1. Ambiguity disguised as speed

An unclear organisation used to be slowed down by the cost of writing things. Now it can generate a detailed plan, thirty tickets, a design document, and an implementation draft before anyone has agreed on the customer problem.

The output looks like progress because it is abundant.

It is often just ambiguity with headings.

A useful question for any AI-generated plan is simple:

> What decision does this let us make that we could not make before?

If the answer is unclear, more output is not the answer.

### 2. Skipping verification

AI makes it easier to produce code that looks right. That is not the same thing as code that survives retries, real permissions, production data, partial failures, or a customer doing something unexpected.

The dangerous pattern is familiar:

1. Ask for a solution.
2. Get a plausible implementation.
3. See that it compiles.
4. Move on.

At human speed, this creates bugs. At AI speed, it creates a backlog of bugs with excellent formatting.

Tests, staging checks, production observability, and review are not friction to remove. They are the feedback system that makes acceleration useful.

### 3. Tool sprawl

If a team responds to every problem by adding another agent, another integration, or another dashboard, AI will make the sprawl easier to build and harder to reason about.

An agent that can send messages, mutate CRM records, open tickets, and trigger deployments needs clear boundaries: what it may do, when it must stop, what budget it has, and where its actions are recorded.

The mature pattern is not “give the agent access.”

It is:

```text
Give it a narrow capability, an approval boundary, an audit trail, and a way to stop.
```

### 4. Avoiding the hard conversation

Some work is difficult because it needs judgement, not because it needs words.

An AI can draft a performance note, a customer response, a strategy memo, or a project update. It cannot take ownership of the relationship consequences. Used badly, the draft becomes a way to avoid saying clearly what a colleague, customer, or team needs to hear.

Use AI to prepare for the conversation. Do not use it to disappear from it.

### 5. Optimising activity instead of outcomes

More campaigns sent, more code merged, more research completed, more tickets closed: all of these can be useful signals. None is the outcome on its own.

AI workflows make activity very cheap. That makes it more important to measure the thing that matters:

- Revenue, not messages sent.
- Resolved customer problems, not support replies drafted.
- Reliable systems, not lines of code generated.
- Learning, not documents produced.

## The individual developer version

The same pattern shows up in personal habits.

### Good habits to amplify

AI can make a disciplined developer unusually effective when they already:

- Write a short plan before editing a large system.
- Ask for counterexamples and failure modes, not only solutions.
- Keep changes small enough to understand and review.
- Run the tests, read the diff, and inspect the rendered or deployed result.
- Use tools to explain unfamiliar code, then verify the explanation against the source.
- Preserve decisions in code comments, issues, or ADRs when the reasoning will matter later.

For example, instead of asking an agent to “fix payments,” a developer can ask it to map the retry path, identify every external side effect, propose idempotency keys, and list tests for concurrent requests. The developer still owns the invariant. The agent helps expose the surface area.

That is leverage.

### Bad habits to watch for

AI also makes certain individual habits more expensive:

- **Prompt thrashing:** repeatedly asking for a better answer instead of narrowing the question or inspecting the system.
- **Borrowed understanding:** merging code you cannot explain because the generated explanation sounds convincing.
- **Premature abstraction:** creating a framework because an agent can generate one before a real repeated need exists.
- **Research as procrastination:** asking for endless comparisons when the next useful action is a small experiment.
- **Context avoidance:** letting a tool guess at requirements you could clarify in one conversation.
- **Infinite polishing:** using AI to improve wording, code style, or architecture after the value of the change has already peaked.

The warning sign is not that you used AI. It is that you can no longer point to the decision you made, the evidence behind it, or the check that would prove you wrong.

## Design workflows that amplify the right thing

The answer is not to use less AI. It is to build better loops around it.

For a team, that usually means:

- Define the owner, objective, constraint, and success measure before delegating work.
- Keep agents behind scoped tools rather than giving them unrestricted credentials.
- Record inputs, actions, outputs, costs, and outcomes.
- Make external side effects idempotent and reviewable.
- Give agents authority for routine work and escalation paths for exceptions.
- Review outcome quality periodically, not only throughput.

For an individual developer, it means treating an AI response as a strong first draft, not a completed thought.

Ask it to challenge your plan. Ask it what breaks under retries, load, cancellation, partial failure, or a malicious input. Ask it to produce a test plan. Then run the test plan.

The point is not to make the human a slower approval layer after the clever work is done.

The point is to put human judgement where it has the most leverage: framing the problem, setting the constraints, deciding what matters, and learning from the result.

## The habit worth building

AI will keep getting faster, cheaper, and more capable. That makes habits more important, not less.

If you have a good loop, AI gives you more attempts at it.

If you have a bad loop, AI gives you more confidence, more output, and less time to notice the mistake.

So the practical question is not:

> How can we use AI to move faster?

It is:

> What are we already doing repeatedly, and is that the behaviour we want to compound?

That is the real AI strategy for a team—and for a developer.
