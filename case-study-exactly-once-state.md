# Case study: exactly-once state

**Status: TESTED**

## Problem

Long-running agents crash, get retried, and get interrupted mid-turn. A memory or state write that isn't safe under those conditions produces duplicate records, lost records, or outright corruption — and an agent that runs for hours will eventually hit exactly that timing.

## Symptom

The runtime's turn lifecycle (start → work → checkpoint → complete) needed to survive a forced restart or crash at any point without double-writing memory or silently losing the record of what had already happened, across every memory sink the runtime writes to — not just one.

## Root cause

A naive "check if a record exists, then append" write pattern isn't safe under interrupted or retried execution: a crash landing between the check and the write either duplicates the entry (if retried) or loses it (if not).

## Engineering constraint

The fix has to work identically across every sink the runtime writes to, and it can't depend on wall-clock timestamps or an assumption that retries won't overlap — the identity of a given write has to be derivable and deterministic from the write itself, not from when it happened to run.

## Solution

Every exchange is assigned a deterministic identity, derived from the turn and session identifiers rather than wall-clock time. Each memory sink writes using that identity as an idempotency key, so the same logical write submitted twice produces the same stored result instead of two.

## Measurement

Verified with isolated crash/retry tests per sink: forced-restart-mid-write, duplicate-submission, and naive check-then-append patterns were all run against the deterministic-identity write path and passed without producing duplicate or lost records, using scratch stores isolated from production data so the tests couldn't contaminate real state.

## Why it matters for agents

An agent that runs for hours will crash, retry, or get killed mid-turn eventually. If that corrupts its own memory, every subsequent turn inherits a state the agent — and its owner — can no longer trust. Exactly-once semantics aren't a nice-to-have for a long-running agent; they're what makes "long-running" survivable at all.

---
[← All case studies](./README.md)
