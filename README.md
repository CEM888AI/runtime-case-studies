# CEM888 Runtime — Engineering Case Studies

> ### ⬅️ This is supporting evidence, not the project.
> **CEM888** is a local-first, model-agnostic agent runtime — state, identity, authority, and verification that live on your machine and persist across Claude, GPT, DeepSeek, and local models.
>
> **→ The project: [CEM888AI/cem888](https://github.com/CEM888AI/cem888)** — ⭐ star it · [cem888.ai](https://cem888.ai) · [💗 Sponsor](https://ko-fi.com/cem888ai)

Sanitized write-ups of real engineering problems found and fixed in CEM888's runtime, with real before/after measurements. This is evidence that the architecture works, not a description of how to reproduce it. See [What's shown vs. withheld](./architecture.md#whats-shown-vs-whats-withheld).

Every claim below is labeled:

- **TESTED** — run and measured, with the measurement shown
- **EXPERIMENTAL** — implemented and exercised, not yet reproduced/stable enough to call settled
- **PLANNED** — designed, not yet built

## Start here

| | |
|---|---|
| [Architecture](./architecture.md) | Conceptual request/turn flow — what's shown, what's withheld |
| [Current engineering status](./current-engineering-status.md) | What is proven on CEM, what is still open, the measured customer-artifact gaps, and the exact partner-release sequence |
| [Partner technical brief](./partner-technical-brief.md) | Technical partner view: integration boundaries, lifecycle, authority, verification, continuity, and evaluation path |
| [Engineering capabilities](./capabilities.md) | Technologies, connected to the systems they're used in |
| [About the engineer](./about.md) | Background, how the system gets built |
| [Framework-to-runtime architecture evolution](./case-study-framework-to-runtime-architecture-evolution.md) | How the system evolved from LangGraph-based agents to a direct Python/HTTP agent loop and then to the current provider-neutral runtime/control layer |

## Case studies

| Case study | What it's about | Status |
|---|---|---|
| [Framework-to-runtime architecture evolution](./case-study-framework-to-runtime-architecture-evolution.md) | Historical progression from framework-based LangGraph agents → custom direct agent loop → provider-neutral deterministic runtime | TESTED historical implementation evidence |
| [Provider-neutral AI continuity through MCP](./case-study-mcp-continuity.md) | External hosts retrieving current work state and persisting explicit updates; tested operations, payload findings, and the remaining host lifecycle boundary | TESTED for scoped operations; automatic host-wide continuity EXPLORATORY |
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
| [Worker spin containment gap](./case-study-worker-spin-containment-gap.md) | Why failure-counting loop guards can't see a worker stuck in a C extension; unbounded per-inference serialization as the trigger; bounded-serialization fix | TESTED |
| [Verification ledger honest status](./case-study-verification-ledger-honest-status.md) | A ledger that read as 91% "unparseable" had no state for "executed, no claim" — honest negative states as a correctness feature | TESTED |

| [Why a one-line change to a shipped runtime stalls a release](./case-study-atomic-train-shipped-runtime-changes.md) | A shipped runtime is a sealed artifact declared in several independent places; a reconciling gate between branch and release, and why the fix train has to be atomic | VERIFIED PROCESS |
| ["Installed and enabled" is not "loaded"](./case-study-installed-and-enabled-is-not-loaded.md) | A plugin present on disk, correctly versioned, byte-identical and switched on — and doing nothing. Declaration is not registration | TESTED |
| [A product telling its own customers to break its supply-chain rule](./case-study-update-path-forbidden-command.md) | Two correct halves of an upgrade path never introduced to each other; install-method detection falling through to the remote-index default | FIXED — shipped and verified in the published artefact |
| [A green test suite over dead code: a source edit is not a deploy](./case-study-source-edit-is-not-a-deploy.md) | Three verification and maintenance defects repaired in source while the running process kept serving the old code; the bug's own fingerprint, dated after the edit, was the proof | TESTED (source-level) |

## Benchmarks

Public benchmark artifacts, raw result files, and scorecard verification: [CEM888AI/benchmarks](https://github.com/CEM888AI/benchmarks)

---

**CEM888** — local-first agent runtime. **[⭐ Star the repo](https://github.com/CEM888AI/cem888)** · [cem888.ai](https://cem888.ai) · [💗 Sponsor](https://ko-fi.com/cem888ai)

Chandler Morone — creator@cem888.ai
