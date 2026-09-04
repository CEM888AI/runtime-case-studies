# Case study: telemetry self-verification

**Status: TESTED**

## Problem

An agent runtime's own telemetry runs on the same code it's measuring. If the instrumentation has a bug, it doesn't produce no numbers — it produces confident, wrong ones, and every decision downstream of "what does the data say the agent is doing" inherits the error.

## Symptom

While investigating tool-call volume, per-turn counts looked uniformly high across a live session — enough to suggest a runaway tool-calling problem worth engineering against.

## Root cause

The field used to group telemetry events by turn was defaulting to the session identifier rather than the true per-turn identifier in the large majority of recorded events. Grouping "by turn" was therefore frequently grouping by session instead — aggregating an entire session's tool calls into what looked like single-turn counts.

## Engineering constraint

The fix needed to be verifiable independent of the buggy field itself — re-deriving segmentation from a different, trustworthy signal rather than patching the mislabeled field and hoping the patch was complete.

## Solution

Re-derived true per-turn boundaries from explicit turn-start/turn-end events instead of the mislabeled identifier field, and used the corrected segmentation to re-baseline what normal tool-call volume per turn actually looks like.

## Measurement

Correcting the segmentation changed the picture materially: what looked like uniformly heavy tool use resolved into a much lower typical count once measured correctly — a median of 4 tool calls per turn, a mean of 8.2, with a small number of genuine outlier turns (a handful over 20 calls, rare above 40) rather than a system that looked heavy everywhere.

## Why it matters for agents

If you're using your own telemetry to judge whether an agent is behaving well, a measurement bug doesn't just hide real problems — it can manufacture fake ones, sending engineering time after a "runaway tool-calling" issue that's actually a labeling bug in the dashboard. Evaluation infrastructure needs the same verification discipline as the system it's evaluating, not an exemption from it.

---
[← All case studies](./README.md)
