# CEM888 — Current Engineering Status

**Updated: 2026-09-23**

This page is the current technical status for founders, engineering partners, evaluators, and prospective design partners.

It exists for one reason: **do not make people infer current product readiness from old benchmark numbers, source files, or architecture diagrams.**

CEM888 is being hardened toward one frozen customer/partner artifact that can prove its own claims.

---

## Executive technical status

The core architecture is implemented far enough that the remaining work is primarily **hardening, promotion into the customer artifact, conformance testing, and current benchmarking** — not inventing the product from scratch.

Several important capabilities have been completed and exercised on CEM's own live runtime, including:

- authority-aware current state and decision supersession;
- bounded replace-not-append working state;
- typed active-work lifecycle;
- multi-session scratchpad write safety / CAS;
- native continuity behavior used by CEM's own runtime.

Several other controls are still being hardened before they are promoted and called customer-certified:

- deterministic action authority;
- verification evidence integrity;
- complete verification coverage;
- liveness-with-effect testing for gates;
- mutation-history / provenance reconstruction;
- authority-aware provider-neutral memory search.

The customer artifact is intentionally being held back from partner handoff until these surfaces are promoted and proven on the installed artifact.

---

## The architecture being proven

```text
TURN START
  -> resolve principal / agent / task
  -> load current authoritative state
  -> retrieve relevant candidates
  -> reconcile authority + supersession
  -> compile bounded current context
  -> model reasons / proposes action
  -> pre-action authority check
  -> allowed action executes
  -> collect observable evidence
  -> verify claimed outcome
  -> commit state transition exactly once
  -> emit receipt
  -> next turn reconstructs from durable state
```

The core invariant is:

> **STATE decides what is true. MODELS decide what to do about it.**

The model is replaceable intelligence. It is not the source of truth, permission, or proof.

---

## Proven on CEM's current engineering path

### Authoritative current state + supersession

The runtime can represent newer authoritative decisions as current while retaining older decisions as history.

The key falsifier is deliberately adversarial:

1. create Decision A;
2. later create Decision B that supersedes A;
3. make A more semantically/lexically similar to a future query;
4. current-state retrieval must still return B as current;
5. A may appear only as superseded/history.

**Status:** implemented on the CEM engineering path. Customer-artifact parity still has to be re-proven.

### Bounded working state

The active working surface is treated as a bounded replacement snapshot rather than an append-only transcript.

Current working context is intended to carry only what the next turn still needs: objective, active constraints, current decisions, blockers, pending verification, verified progress, next action, and compact evidence references.

**Status:** implemented on the CEM engineering path. Customer-artifact promotion and long-horizon install proof still required.

### Typed active-work lifecycle

Active work is driven by explicit lifecycle state rather than phrase matching over prose.

Current states such as ACTIVE / UNRESOLVED / PENDING_VERIFICATION / BLOCKED_ON_USER are distinguished from COMPLETED / RESOLVED / VERIFIED / SUPERSEDED / ARCHIVED / HISTORICAL.

**Status:** implemented on the CEM engineering path. Customer-artifact parity still required.

### Multi-session write safety

Concurrent sessions use compare-and-swap / serialization protections rather than blind overwrite of shared working state.

**Status:** implemented on the CEM engineering path. Customer-install concurrency proof still required.

---

## Still being hardened on CEM before customer promotion

### Deterministic action authority

The target design is:

> **MODELS PROPOSE. STATE RESOLVES. RUNTIME AUTHORIZES. TOOLS EXECUTE. VERIFIER PROVES.**

The remaining work is to finish the bypass audit and prove that consequential actions cannot silently widen their own target, authority, or scope.

**Status:** in progress.

### Verification evidence integrity

A real adversarial test exposed a correctness hole where caller-supplied command output could be treated as evidence strongly enough to produce a verified pass.

The release-blocking fix requires evidence-backed raw bytes / persisted evidence identity rather than trusting model- or caller-supplied stdout.

**Status:** in progress. CEM888 should not claim universal verification coverage until this is closed.

### Verification coverage

Some action classes are covered, some partial, and some intentionally still open. The public verification-coverage matrix is being completed so evaluators can see the exact boundary.

**Status:** in progress.

### Gate liveness

CEM888 has previously found mechanisms that were installed or registered but had no real effect.

The permanent test pattern is therefore: a gate must demonstrate a real deny, withhold, or other measurable effect — not merely report that it loaded.

**Status:** not complete yet.

### Mutation provenance

The runtime is being hardened so it can mechanically answer not only *what is current*, but *why it changed*, who/what changed it, what it superseded, and which evidence caused the transition.

**Status:** not complete yet.

### Authority-aware deep memory search

The provider-neutral search contract is being completed so semantic/keyword/timeline retrieval cannot elevate stale information into current authority.

**Status:** in progress.

---

## Current customer-artifact baseline

The latest measured customer-install baseline was run on an installed customer profile using engine **1.0.9**.

That artifact was intentionally treated as a test subject rather than assumed correct.

**Result: 2 PASS / 9 FAIL — NON-CONFORMANT.**

The measured failures included:

- no proven durable exactly-once/idempotency key;
- no cheap turn-start inhale gate;
- incomplete authority metadata;
- schema-fragile memory writer;
- Chroma/BM25 present but not wired into the automatic inhale compiler;
- no timeline/FTS retrieval leg;
- no automatic typed-state exhale writer;
- provider-neutral operations absent from the installed artifact;
- broken local MCP serve import.

That artifact is **not** the partner build and is not being represented as certified.

The important result of this test was not the failure count. It was that the install now has a deterministic conformance harness capable of falsifying runtime claims on the artifact itself.

---

## Release sequence for the partner/customer build

CEM888 is deliberately avoiding repeated wheel churn.

The current sequence is:

```text
finish CEM engineering
  -> prove/falsify each product-relevant capability on CEM
  -> freeze the CEM capability set
  -> promote all customer-relevant fixes in one batch
  -> build one candidate customer artifact
  -> freeze artifact + digests
  -> run install conformance
  -> run clean install / upgrade tests
  -> fix only certification defects
  -> rerun current benchmark suite
  -> partner handoff
```

A capability is not considered customer-proven because:

- it exists in source;
- it works on CEM;
- a plugin directory exists;
- the installer says it is enabled.

It is customer-proven only when the **same falsifier passes on the exact installed artifact** and the result names the build/digests being tested.

---

## Benchmark status

Earlier benchmark and case-study results remain useful historical engineering evidence, but the runtime has changed substantially since those runs.

A current re-baseline is planned against the frozen partner/customer artifact.

The new benchmark campaign will compare, on the same workload:

- verified task success;
- false-completion rate;
- repeated tool/action work;
- model/generative call count;
- input/output tokens where available;
- actual inference spend;
- retries;
- human intervention/review;
- latency;
- restart/fresh-session continuity;
- cost per verified successful task.

Where context/state improvements are being tested, a **static-context control arm** will be included so CEM888 does not take credit for improvements caused merely by giving the model a better prompt.

No new cost-reduction percentage will be promoted until it is measured on the frozen current artifact.

---

## What a technical partner should read next

1. [Partner Technical Brief](./partner-technical-brief.md) — integration boundaries without implementation secrets.
2. [Architecture](./architecture.md) — conceptual runtime/turn flow and what is deliberately withheld.
3. [Engineering Case Studies](./README.md) — measured failures, fixes, and historical evidence.
4. [Benchmarks](https://github.com/CEM888AI/benchmarks) — raw benchmark repository; current re-baseline pending.
5. [CEM888 source repository](https://github.com/CEM888AI/cem888) — public source and license.

---

## Current partner policy

Technical discovery, architecture review, and targeted integration planning are welcome now.

The customer installer is being held until the frozen candidate build passes the current conformance and benchmark gates.

That is deliberate.

A partner should receive the build that can **prove what it does**, not one that merely appears operational.
