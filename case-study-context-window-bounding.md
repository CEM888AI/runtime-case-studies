# Case study: context window bounding

**Status: TESTED**

## Problem

A context window that's supposed to stay bounded — "the last N messages" — needs to actually stay bounded regardless of what any individual turn contains, or every downstream cost, latency, and reliability assumption built on top of that bound is false.

## Symptom

One user request produced a carrier context of **207 messages** instead of the intended small window, because that single request owned a large assistant/tool-call exchange.

## Root cause

The context compiler anchored the window by searching backward for the nearest user message and building the window from there. When that nearest user message was the one that had opened a long tool-use exchange, the backward search walked all the way back through the entire exchange — silently re-expanding a window that was supposed to be bounded to match the size of whatever the largest single exchange happened to be.

## Engineering constraint

The fix couldn't simply truncate the window. Tool-call and tool-result messages have to stay paired — most model providers reject a tool result that's missing its originating call, or vice versa — so any bounding logic has to respect protocol integrity while still enforcing a real size limit.

## Solution

Replaced the backward-search anchor with deterministic bounded selection: the window is now built forward from a fixed selection budget, tool/assistant message pairs are kept atomic (never split across the boundary), and the size limit is enforced independent of how large any single turn's exchange grew.

## Measurement

The pathological case went from a **~65,559-token** conversation carrier to **~3,357 tokens**, while the runtime's separately compiled Minimal Sufficient Context packet — the part actually carrying task-relevant state — held steady at **~1,010 tokens** throughout, unaffected by the carrier bug.

## Why it matters for agents

A context window that "usually" stays bounded isn't bounded. A single pathological turn silently reintroduces the cost, latency, and failure modes of unbounded context — and an agent running for hours or days across many sessions will eventually produce that turn. Bounding has to be a structural guarantee, not an emergent property of normal conversation shape.

---
[← All case studies](./README.md)
