# Case study: identity & ownership isolation

**Status: TESTED**

## Problem

A runtime taking real actions on someone's behalf needs to know, with certainty, whose authority it's currently acting under. Without an explicit rule, a stale correction, an old backup, or state belonging to a different session can quietly leak into the wrong execution context.

## Symptom

During a live internal audit of the runtime's own behavior, a specific case surfaced where the developer's own identity and working state could bleed into what a clean customer-facing instance should see — a state-scoping gap between "this is Chandler's dev/test context" and "this is a customer's isolated instance."

## Root cause

Without an explicit, ordered authority hierarchy, competing sources of truth — a live correction from the current owner, long-term memory, an old configuration file, the model's own inference about what's probably true — resolve based on whichever source the model happens to weight most heavily on a given turn. That's not deterministic, and it isn't safe once real actions are involved.

## Engineering constraint

The fix has to be enforced in the runtime's execution path, not left as an instruction inside a prompt — a rule that lives only in the prompt is a rule a model can miss under load, exactly the failure mode being defended against.

## Solution

An explicit ownership/authority gate, enforced as a check in the execution path rather than a prompt convention: a live correction from the current owner outranks historical memory, which outranks backups, which outranks a stale registry entry, which outranks the model's own inference. Every correction the gate applies is recorded as a receipt.

## Measurement

Implemented and exercised against a real trigger case (a component the runtime needed to definitively stop treating as active/live): the correction was applied deterministically and logged as a receipted authority decision, rather than depending on the model inferring it correctly turn after turn.

## Why it matters for agents

The moment an agent does real work across sessions, environments, or customers, "who does this state belong to, and who's allowed to change it" stops being a design nicety and becomes a correctness and security boundary. Getting it wrong is exactly how one context's state leaks into another's, or a stale fact silently overrides a live correction nobody asked to happen.

---
[← All case studies](./README.md)
