# Case Study: Cache-Stable Context for Long-Running AI Agents

**Status: EXPERIMENTAL — controlled optimization in progress**  
**System under test:** CEM888 live agent runtime using the DeepSeek API

## The engineering problem

A common optimization instinct in LLM systems is to make every request as small as possible. In a long-running tool-using agent, that can be the wrong objective.

The agent under test was already capable of completing a real authenticated browser workflow, but its prompt composition changed enough between model calls that only about half of its input was being served from provider cache. At the same time, aggressive context reduction risks removing information a long autonomous run still needs.

The engineering question is therefore not simply:

> How few tokens can the agent send?

It is:

> How much of the context can remain stable and reusable while the agent still preserves the state required to finish a long task correctly?

## Baseline

One representative live workflow produced:

| Metric | Baseline |
|---|---:|
| Verified outcome | Pass with warnings |
| Wall time | 107.9 s |
| Model calls | 16 |
| Tool calls | 13 |
| Cache-read input | 107,264 tokens |
| Cache-miss input | 90,732 tokens |
| Total input | 197,996 tokens |
| Cache-hit ratio | 54.2% |
| Output | 8,168 tokens |

The token figures are runtime telemetry, not a provider invoice. Actual cost is reported only when billing evidence is available.

## Controlled change: Round 1

Only one behavior is being changed for the first experiment:

**Create the task-entry context once, keep it stable for that task, and append new execution events rather than repeatedly rebuilding earlier context.**

Everything else is held constant for the round: model/provider, reasoning mode, available capabilities, context ceiling, retrieval budget, termination policy, and benchmark task.

This is deliberately a one-variable experiment. A separate fresh-chat transcript-recovery mechanism is being evaluated independently and is not being mixed into this round.

## Hypothesis

If the working context remains stable during the active task, two things should improve together:

1. **Provider cache reuse:** later model calls should reuse a substantially larger fraction of previous input.
2. **Execution continuity:** the task objective and relevant working state should remain consistent across a long tool sequence instead of being repeatedly reconstructed.

The target is approximately **97–98% steady-state cache reuse after warm-up**, but cache percentage alone cannot pass the experiment.

## Release gate

The optimization is kept only if all of the following remain true:

- the agent still completes the same verified workflow;
- a longer multi-tool task completes without losing its objective or progress;
- user steering and current state remain authoritative;
- no new duplicate-call or runaway-loop behavior appears;
- cache reuse improves materially;
- actual cost per successful task improves when provider billing data is available.

A lower bill with worse task completion is a regression, not an optimization.

## Why this experiment matters

This is a practical example of provider-aware agent engineering: **token count is not the same thing as cost, and smaller context is not automatically better context.** Optimization has to be evaluated at the level of successful tasks, not isolated request size.

It also demonstrates the tradeoff that appears in real autonomous systems: an agent needs enough stable working state to finish multi-step work, while the runtime needs to control latency, cache behavior, and spend.

## What is intentionally withheld

This public case study does **not** expose CEM888's proprietary prompt contents, state schemas, retrieval/scoring logic, internal paths, tool manifests, credentials, endpoints, customer data, or implementation code.

The public artifact contains the engineering question, experimental method, measurements, and result. The implementation remains private.

## Next update

This page will be updated with the Round 1 after-metrics and a clear **KEEP / REJECT** verdict after the identical workflow, a long multi-tool task, and a fresh-session continuity probe have been rerun.

---

Built and tested by **Chandler Morone** — CEM888.AI
