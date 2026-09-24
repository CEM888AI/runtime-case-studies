# CEM888 — Current Engineering Status

**Updated: 2026-09-24**

This page is the current technical status for founders, engineering partners, evaluators, and prospective design partners.

It exists for one reason: **do not make people infer current product readiness from an old benchmark, a source file, or a different artifact.**

CEM888 has three distinct proof surfaces:

1. **CEM engineering runtime / ancestor** — where product-relevant mechanisms are built, falsified, and hardened.
2. **Public repositories** — documentation, source/evidence, case studies, benchmark artifacts, and public falsifiers.
3. **Customer product artifact** — the downloadable wheel/bundles that must be frozen and re-proven independently before a customer claim becomes certified.

A mechanism working on CEM is evidence. It is **not automatically proof that the current customer artifact carries the same behavior**.

---

## Executive technical status

The core state/control architecture is built and operating on the CEM engineering path.

Current release work is concentrated on the customer artifact: closing install-specific authority and concurrency defects, rebuilding the artifact chain, and then re-running conformance on the exact shipped build.

### Proven on the current CEM engineering path

The 2026-09-24 CEM ledger records the following live acceptance results after deploy:

- **owner-prohibition / authority path:** 8 passed, 0 failed;
- **structured verification:** 53 passed, 0 failed;
- **hook/source synchronization falsifiers:** 17 passed, 0 failed;
- fabricated-evidence receipt falsifier: 4 passed;
- consequential-action / `execute_code` bypass matrix: 4 passed plus ordered allow/block controls;
- cross-process store locking: 4 passed;
- scratchpad reconcile behavior corrected so the newest current write survives later reconciliation.

These measurements are **CEM engineering-runtime proof**, not customer-install certification.

### Still not certified for the customer artifact

The current customer release lane is not finished.

Two confirmed defects were found directly against the shipped **1.0.10 wheel**:

1. the durable memory writer allowed model-originated writes to acquire user authority through its default/callable authority surface;
2. fresh-store concurrent writes could fail with `database is locked`, causing a memory write to be lost rather than queued/retried.

Repairs have been exercised in the customer-runtime worktree, but the release chain is not complete yet. The remaining sequence is:

```text
wire owner-attestation producer
  -> commit customer-runtime source
  -> rebuild wheel
  -> rebuild platform bundles
  -> re-certify exact artifacts
  -> publish
  -> verify shipped bundles
```

Until that chain passes, the customer artifact is **not represented as fully conformant**.

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
  -> commit resulting state
  -> emit receipt / provenance
  -> next turn reconstructs from durable state
```

The core invariant is:

> **STATE decides what is true. MODELS decide what to do about it.**

The model is replaceable intelligence. It is not the source of truth, permission, or proof.

A stronger action-control formulation used in the engineering architecture is:

> **MODELS PROPOSE. STATE RESOLVES. RUNTIME AUTHORIZES. TOOLS EXECUTE. VERIFIER PROVES.**

---

## Current state / memory behavior

CEM888 does not treat conversation history as authoritative working memory.

The runtime separates:

- current owner authority;
- durable identity / invariants;
- current authoritative state;
- unfinished active work;
- bounded compiled working context;
- durable memory;
- reusable skills/pathways;
- historical evidence / receipts.

The critical rule is:

> **Relevance may retrieve a candidate. Relevance does not increase its authority.**

Superseded material can remain available as history without silently becoming current truth again because it happens to be a strong lexical or semantic match.

---

## Context compilation

The active reasoning packet is intended to be a bounded working surface rather than an append-only transcript.

The compiler keeps the minimum current material needed for the next reasoning event: objective, active constraints, current decisions, blockers, pending verification, verified progress, next action, and compact evidence references.

Completed, resolved, superseded, archived, or historical material is removed from ordinary active carry while remaining recoverable from durable storage.

This is the mechanism behind the product requirement that a user should not have to repeatedly rebuild project state every time the model, host, or session changes.

---

## Action authority and owner prohibitions

Consequential actions are not supposed to inherit authority from model confidence or natural-language plausibility.

An explicit authenticated owner prohibition is intended to govern both sides of the lifecycle:

- **retrieval/current-state side** — prohibited or superseded material must not be surfaced as current authoritative truth;
- **execution side** — a matching protected action must be denied at a CEM-controlled boundary.

The host boundary matters. CEM888 can only claim hard enforcement where the host/action actually crosses an enforceable CEM boundary. A host capability outside that boundary must be described as partial / host-restricted rather than universally controlled.

---

## Verification and receipts

CEM888 does not treat a model saying "done" as proof.

The current CEM verification path has been hardened against a real fabricated-evidence failure class: caller/model-supplied output cannot simply promote itself into verified truth. The current CEM structured-verification suite is passing on the engineering runtime, but verification coverage is still action-class-specific and must be re-proven on the customer artifact.

A receipt is provenance/evidence. It is not independent certification.

---

## Customer-artifact history

### Earlier 1.0.9 baseline

An earlier installed customer artifact was intentionally tested rather than assumed correct.

**Published result: 2 PASS / 9 FAIL — NON-CONFORMANT.**

That baseline was useful because it proved the release process could falsify its own runtime claims. It should not be read as the status of the current engineering runtime.

### Current 1.0.10 release lane

The shipped 1.0.10 wheel has since exposed the two install-surface defects described above: memory-authority attribution and fresh-store concurrent-write loss.

The fixes must be promoted into a rebuilt exact artifact and then re-tested. A green source test is not a shipped fix.

---

## Public proof vs. internal proof vs. customer proof

CEM888 is deliberately separating these claims:

| Surface | What it can prove |
|---|---|
| **Live CEM engineering runtime** | Whether the mechanism works on the ancestor/runtime currently being hardened |
| **Public source + public falsifiers** | Whether a reviewer can inspect and run scoped checks against the public tree |
| **Frozen customer artifact** | Whether the exact wheel/bundle a customer receives satisfies the release contract |

The public `cem888` repository is being updated with runnable source-level falsifiers for authority and context invariants. Those tests improve external reviewability, but they do not replace exact-artifact certification.

---

## Benchmark status

Historical benchmark runs remain engineering evidence, but benchmark proof is being tightened so the repository distinguishes:

- scorecard arithmetic that can be independently recomputed;
- raw-answer scoring utilities;
- the original live-agent generation environment, which a scorecard checker alone does not recreate.

For BEAM, the public Vetta 77.2% scorecard is the independently checkable result currently shipped. The CEM 78.2% run remains labeled experimental until equivalent per-question public evidence is present.

A new runtime benchmark campaign should be run again against the frozen customer candidate after the release artifact is rebuilt.

---

## What a technical partner should read next

1. [Partner Technical Brief](./partner-technical-brief.md) — integration boundaries without implementation secrets.
2. [Architecture](./architecture.md) — conceptual runtime/turn flow and what is deliberately withheld.
3. [Engineering Case Studies](./README.md) — measured failures, fixes, and historical evidence.
4. [Benchmarks](https://github.com/CEM888AI/benchmarks) — public benchmark artifacts and scorecards.
5. [CEM888 source repository](https://github.com/CEM888AI/cem888) — public source, status, and license.

---

## Current partner policy

Technical discovery, architecture review, and targeted integration planning are welcome now.

The customer release claim remains gated on the rebuilt artifact passing its own conformance checks.

That is deliberate.

A partner should receive a build that can **prove what it does**, not one that merely appears operational.
