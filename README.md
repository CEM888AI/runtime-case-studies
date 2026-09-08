# CEM888 Runtime — Engineering Case Studies

Sanitized write-ups of real engineering problems found and fixed in CEM888's runtime, with real before/after measurements. This is evidence that the architecture works, not a description of how to reproduce it — CEM888's runtime, memory indexing, tool-governance, and provider-routing implementations stay private. See [What's shown vs. withheld](./architecture.md#whats-shown-vs-whats-withheld).

Every claim below is labeled:

- **TESTED** — run and measured, with the measurement shown
- **EXPERIMENTAL** — implemented and exercised, not yet reproduced/stable enough to call settled
- **PLANNED** — designed, not yet built

## Start here

| | |
|---|---|
| [Architecture](./architecture.md) | Conceptual request/turn flow — what's shown, what's withheld |
| [Engineering capabilities](./capabilities.md) | Technologies, connected to the systems they're used in |
| [About the engineer](./about.md) | Background, how the system gets built |

## Case studies

| Case study | What it's about | Status |
|---|---|---|
| [Legacy IBM i autonomous debugging](./case-study-legacy-ibmi-autonomous-debugging.md) | Resuming an interrupted task, repairing a broken legacy 5250 client at the protocol layer, and verifying the fix against a live IBM i 7.5 system | TESTED |
| [Cache-stable agent context](./case-study-cache-stable-agent-context.md) | Testing whether a stable task context can improve DeepSeek cache reuse without sacrificing long-run task continuity | EXPERIMENTAL |
| [Context window bounding](./case-study-context-window-bounding.md) | A "bounded" context window silently expanded to 207 messages; fixed with deterministic selection | TESTED |
| [Tool-schema scoping](./case-study-tool-schema-scoping.md) | 84 tool schemas (~29.3K tokens) on every call, cut to a task-scoped surface | TESTED |
| [Completion verification](./case-study-completion-verification.md) | Distinguishing a model's claim of "done" from verified evidence that it happened | TESTED |
| [Exactly-once state](./case-study-exactly-once-state.md) | Making memory writes survive crash/retry without duplication or loss | TESTED |
| [Cross-provider continuity](./case-study-cross-provider-continuity.md) | Keeping agent state and identity intact across a change of model provider | EXPERIMENTAL |
| [Identity & ownership isolation](./case-study-identity-ownership-isolation.md) | An explicit authority order for whose state and correction wins | TESTED |
| [Customer tenancy isolation](./case-study-customer-tenancy-isolation.md) | Live QA as a fresh tenant found owner-era pages rendering the founder's account to customers; fixed with session-verdict guards and re-verified in the customer's browser | TESTED |
| [Telemetry self-verification](./case-study-telemetry-self-verification.md) | Finding and fixing a bug in the runtime's own measurement of itself | TESTED |

## Benchmarks

Reproducible memory-retrieval results with raw data: [CEM888AI/benchmarks](https://github.com/CEM888AI/benchmarks)

## Contact

Chandler Morone — creator@cem888.ai · [cem888.ai](https://cem888.ai)
