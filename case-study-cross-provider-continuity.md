# Case study: cross-provider continuity

**Status: EXPERIMENTAL** — architecture built and exercised across providers and interfaces; not yet reduced to a single published benchmark number.

## Problem

Most agent frameworks treat each model conversation as its own isolated intelligence: swap providers, or even just start a fresh session, and the agent's working state resets. That makes model routing (cheap model for routine work, frontier model on escalation) expensive in a way that has nothing to do with API pricing — you lose continuity every time you switch.

## Symptom

An agent's persistent state — what it knows, what it's mid-task on, what happened last session — needed to survive not just a session boundary, but a change of the underlying model provider driving it.

## Root cause

In the designs this runtime was built to replace, state, identity, and memory live inside the model's own context. That makes them only as durable as that one conversation, with that one provider — there's no representation of "what's true" that exists independent of the model currently attached to it.

## Engineering constraint

The state layer has to be provider-neutral by construction: it can't assume anything about a specific model's context format, tool-calling convention, or session semantics, because the same state has to be usable behind Claude, GPT, DeepSeek, Gemini, or a fully local model interchangeably — including models that weren't available when the state was originally written.

## Solution

State, memory, and identity live in the runtime, external to any model's context. A per-turn context-compilation step assembles what the currently-driving model needs to know for that turn from that external state — the state doesn't move when the model does; only the compiled packet does.

## Measurement

Continuity has been exercised across multiple model providers and multiple live interfaces (chat clients, messaging platforms) with the same underlying agent identity and memory intact across the switch. This is documented as working, not yet reduced to a single reproducible before/after number the way the other case studies are — that benchmark is in progress.

## Why it matters for agents

Provider lock-in isn't only a pricing problem. If an agent's memory lives inside one provider's context, you can't route cheap tasks to a cheap model and hard tasks to a frontier model without losing continuity every time you do it. Provider-neutral state is what makes model routing — and model competition — possible at the agent level instead of the API level.

---
[← All case studies](./README.md)
