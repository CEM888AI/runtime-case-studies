# Moving Agent Reliability Into the Runtime

**Date:** September 9, 2026  
**Status:** Component-tested findings and observational runtime evidence. Targeted corrections recommended; no production change or matched performance improvement claimed.

## The question

Can deterministic runtime engineering improve the reliability of inexpensive agent models without turning the system into a brittle rule engine?

We use a stronger agent system as an observable behavioral reference: investigate when evidence is missing, recognize when supplied information is enough, verify consequential actions, recover from failures, preserve constraints and stop when finished. We then ask which parts can be supported by ordinary software around a less expensive model.

We are not distilling Astra's model, cloning proprietary model reasoning, or extracting private chain-of-thought. We are studying observable agent-system behavior and independently engineering CEM888 runtime controls. “Astra” names the reference agent used in this investigation, not a model whose internals we can inspect. This work implies no provider partnership or endorsement.

## Why we are testing this

Model capability is one part of an agent system. A capable model can still receive stale evidence, repeat a failed operation, lose an important constraint or accept a misleading success signal. A smaller model benefits when the runtime reliably supplies state and enforces well-defined execution contracts.

That does not make every deterministic intervention beneficial. A rule that saves one unnecessary read can also prevent a necessary retry. A fast path can reduce overhead while removing the only route to fresh evidence. A verifier can be deterministic and consistently wrong.

Our objective is verified useful work per unit of inference and cost. Neither fewer tokens nor more runtime controls establishes progress toward that objective by itself.

## The experiment

One runtime serves as a canary for incremental changes. We inspected its current implementation, configuration and deployment provenance, compared it with two deployed peer runtimes, ran focused existing tests, and added small diagnostic probes around ambiguous boundaries.

The component probes used isolated fixtures and current source. They made **zero model requests and zero live runtime tool calls**. They test control behavior, not the model's ability to solve the task. We also inspected retained live observations without coaching or changing the running agents.

The peer observations involved different tasks and histories. They are useful for locating differences and measurement problems, but do not constitute a controlled performance comparison. There was no matched Astra-versus-canary run and no before/after runtime intervention in this audit.

## What belongs in the runtime

Good candidates have explicit, checkable contracts:

- Preserve an authorized objective, constraints and evidence references across work.
- Distinguish an attempted operation from a successful result.
- Suppress repeated access to evidence only when that evidence remains usable and current.
- Retain a route to external verification when supplied context is insufficient.
- Separate tool execution status, verification results and actual task completion.
- Honor interruption at dispatch boundaries and report the disposition of in-flight work.
- Record usage and outcomes with enough provenance to audit them.

The model still decides how evidence bears on an ambiguous question and which action will resolve it. The runtime should make that reasoning better informed and less wasteful.

## What should remain probabilistic

Understanding arbitrary intent, challenging a false premise, selecting a useful investigation and synthesizing conflicting evidence require semantic judgment. Short wording does not establish that a question is simple or that its answer is temporally stable.

Our probes also exposed the risk of mechanically rewriting an answer to enforce a constraint: a correct negative statement lost essential meaning. Enforcing an action boundary and editing natural-language meaning are different responsibilities.

We do not propose an exhaustive classifier for all questions or a mandatory reasoning pipeline for every turn. A direct answer should remain possible when the information is already sufficient.

## The canary

The canary includes controls for context selection, bounded execution, evidence handling, interruption, continuity and tool exposure. This article reports their observable purposes and tested outcomes. It does not disclose their proprietary implementation, prompts, configuration or state representation.

We preserved the working baseline. The active agents were not restarted, reconfigured or moved to another model. Recommendations remain subject to one-change-at-a-time acceptance.

## What we measured

| Evidence set | Result | What it establishes |
|---|---|---|
| Existing focused regression tests | 44 tests and 20 subtests passed | Covered context and tool-surface contracts still work in isolation |
| Added diagnostic checks | 25 checks: 17 passed, 8 failed | Boundary cases expose specific control defects; this is not an agent success rate |
| Diagnostic provider activity | 0 model requests; 0 live runtime tool calls | Failures can be reproduced without increasing model capability or spend |
| Retained live greeting | 1 model call; 0 tool events; 7.905 seconds runtime duration | Direct response behavior occurred in one observed turn; transport latency was not separately measured |
| Retained live investigation | 6 model calls; 6 tool events; 52.081 seconds runtime duration | Investigation and duplicate suppression occurred; the final explanation included an unsupported inference |

The six tool events include a capability-opening operation and a suppressed duplicate. They are not six successful external operations. Runtime duration is not the same as receive-to-first-visible-response latency.

The diagnostic set intentionally combines adversarial boundaries with ordinary positive and negative controls. It is too small and selectively constructed to estimate real-world failure prevalence. It does show that passing the existing tests was insufficient to establish the intended behavior.

### Verification

Two independent paths accepted insufficient or contradictory execution evidence as success. Ordinary success and failure controls behaved as expected, but the boundary cases did not. That supports a narrow correction to verification authority before adding further execution machinery.

These were component failures. We did not infer that every live task receives a false completion result, nor did we claim a measured production incident rate.

### Recovery and evidence access

An unchanged successful read was correctly suppressed. A necessary recovery read was also suppressed in a diagnostic case. Changed evidence was handled correctly by one component but blocked by an earlier control.

The engineering lesson is that controls must be tested together. A correct component underneath an incompatible earlier gate does not produce a correct system.

A separate pair of near-equivalent factual questions showed that small wording changes could decide whether an evidence path remained available. That is a routing limitation, not proof that the model would necessarily answer incorrectly.

### Evidence calibration

The observed live investigation gathered useful information and still drew a conclusion that the available evidence did not establish. More evidence gathering does not automatically produce sound inference. We would not fix that by adding a special-case rule for that particular question.

### Usage and caching

We found a reporting ambiguity: some completion records combined invocation-level counts and elapsed time with accumulated session usage. Treating every field as a per-turn measurement would overcount. We reconstructed the canary observations from individual inference receipts rather than summing those summaries.

Cache reuse was observable, but it was not a controlled cost experiment. DeepSeek documents prefix-based caching and exposes cache-hit information, so reusable prefixes matter alongside prompt volume. [DeepSeek context caching documentation](https://api-docs.deepseek.com/guides/kv_cache/).

Runtime-normalized token counters were available; independently captured raw provider receipts and billed task costs were not. No dollar saving, causal cache improvement or cost-per-verified-task advantage is claimed.

## What we learned

Deterministic runtime structure can enforce useful local contracts. It can also convert a mistaken assumption into a hard obstacle to reasoning. The most consequential boundary in this audit was the distinction between an observation and a verified result.

The evidence supports correcting narrow defects. It does **not** establish that the canary's accumulated controls improve overall quality per dollar relative to its previous baseline, the peer runtimes or the stronger reference agent.

This distinction matters: finding a defect is useful engineering evidence even when a broad performance claim remains unproven.

## What we changed

No production runtime, configuration or model changes were made. We created isolated diagnostics, corrected the audit's interpretation of existing usage records and documented a prioritized correction plan.

The first recommended change is to make explicit failed execution evidence outrank incidental success text. A separate follow-up should ensure that only trustworthy verification results become verified working facts. Each change needs its own acceptance evidence.

## What we deliberately did not change

We did not add a larger classifier, a new mandatory planner, extra model critics, broader tool permissions or a more expensive model. We did not promote canary code to the peer runtimes or tune context size solely to reduce token counts.

Fresh-session continuity, live interruption timing, externally verified multi-step completion and matched before/after cost remain acceptance work. Existing tests do not substitute for those measurements.

## Next question

Can one minimal correction to verification authority eliminate the demonstrated false positives while preserving legitimate success, useful diagnostics, latency and cache behavior?

The next experiment should compare the same small tasks before and after that one change, then test differently worded cases and a real artifact-producing task. Keep the change only if verified outcomes improve without an unacceptable regression.
