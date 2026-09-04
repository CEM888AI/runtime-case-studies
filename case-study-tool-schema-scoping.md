# Case study: tool-schema scoping

**Status: TESTED**

## Problem

Every schema definition sent to a model provider on every call is pure overhead — tokens the model has to read before it does anything — and a static, session-wide tool surface pays that cost on every single turn regardless of what the turn actually needs.

## Symptom

A normal provider call carried the full registered tool surface: **84 tool schemas**, approximately **29.3K schema tokens** — even for turns as narrow as a greeting or a single read-only file lookup.

## Root cause

The tool surface was scoped to the session, not the turn: once a gateway session started, every provider call it made included the entire registered tool set. There was no mechanism deciding, per turn, which subset of tools that turn's task actually touched.

## Engineering constraint

Tools can't be removed by static category, because the runtime doesn't know in advance which tools a given turn needs — the scoping decision has to happen per turn, cheaply, without adding a second model call just to decide what the first call's tool surface should be.

## Solution

Added task-scoped tool-surface reduction: turns that are identifiably narrow (for example, a read-only local-code question) get a reduced tool schema — scoped down to the small set of tools actually relevant to that class of turn — instead of the full registered surface.

## Measurement

On a real workflow over a live messaging interface, this took the interaction from **4 model calls / 25.6 seconds** down to **1 model call / 15.7 seconds**, with the smaller payload producing the same result. Separately, the same class of fix collapsed a trivial conversational turn's provider payload from ~31.7K tokens (84 tools, full retrieval) to ~3.0K tokens (0 unnecessary tools, no retrieval) — one provider call either way, but roughly 40% faster and a fraction of the tokens.

## Why it matters for agents

Tool-schema tokens are a standing tax paid on every call, not a one-time cost — for a long-running or high-frequency agent, an unscoped tool surface compounds across every turn into real latency and real API spend. Scoping the tool surface to the task is one of the highest-leverage, least glamorous fixes available in an agent runtime.

---
[← All case studies](./README.md)
