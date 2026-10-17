# CEM888 — Current Engineering Status

**Updated: 2026-10-17**

This page is the current technical status for founders, engineering partners, evaluators, and prospective design partners.

It exists for one reason: **do not make people infer current product readiness from an old benchmark, a source file, or a different artifact.**

CEM888 has three distinct proof surfaces:

1. **CEM engineering runtime / ancestor** — where product-relevant mechanisms are built, falsified, and hardened.
2. **Public repositories** — documentation, case studies, benchmark artifacts, current status, and public falsifiers. The runtime source itself is **private**; source access is granted to licensees and design partners rather than published.
3. **Customer product artifact** — the downloadable wheel/bundles that must be frozen and re-proven independently before a customer claim becomes certified.

A mechanism working on CEM is evidence. It is **not automatically proof that the current customer artifact carries the same behavior**.

---

## Executive technical status

The core state/control architecture is built and operating on the CEM engineering path.

Current release work is concentrated on the customer artifact: closing install-specific authority and concurrency defects, rebuilding the artifact chain, and then re-running conformance on the exact shipped build. Alongside that, the product has moved from a single download to **two lanes plus a desktop application** — see the product status below.

## Product status — 2026-10-17

CEM888 ships two lanes from one build pipeline: a **free agent download**, and a **paid lane** carrying the agent plus the **CEM888 Workspace** desktop app. Each lane has its own installer, manifest and artifact verification.

Status is per capability. **Shipped** means implemented and released in a product lane. **In progress** means open work with a known defect — it is not claimed complete. **Planned** means designed, not built.

| Capability | Status | Notes |
|---|---|---|
| Installer — free lane | **Shipped** | Account flow at cem888.ai; artifact verification is fail-closed, and a rebuild no longer serves a poisoned verification cache |
| Installer — paid lane | **Shipped** | Own installer and manifest; the Workspace is built into the paid bundle and launched on first install |
| CEM888 Workspace (desktop app) | **Shipped** | Tauri + React; Control Room, plus Rooms for State / Decisions / Writes / Receipts / Connections / Agents / Settings |
| Chat host selection | **Shipped** | Claude / Codex / DeepSeek on separate per-host threads; Codex drives a shared app-server daemon rather than cold-spawning one process per message |
| Streaming chat parity | **In progress** | Token streaming, activity line, steer, stop, pin |
| Receipts — hash | **Shipped** | Dedicated panel; fail-closed receipt binding |
| Receipts — tool | **Shipped** | Dedicated panel; tool receipts and memory disposition ship in the paid agent payload |
| Checkpoints | **Shipped** (API) · **In progress** (browser) | The customer API can open *and* close a checkpoint; the full checkpoints browser surface is still open |
| Connections / provider catalog | **Shipped** | Live probes per provider; GitHub is a first-class row with a real connectivity probe |
| Credential storage | **Shipped** | OS-native credential storage, never a plaintext `.env` |
| Provider connections — subscriptions | **Partially shipped** | Server half of the subscription OAuth device-code flow is implemented; provider parity for the Claude-only and Codex lanes inside the Workspace terminal is **in progress** |
| Persistent state | **Shipped** | SQLite as local durable truth; typed lifecycle and explicit supersession; typed active-work state driven by lifecycle events rather than prose. Multi-session write safety uses lock / compare-and-swap; contention behaviour is still under repair |
| Agent extensibility | **Shipped** | Plugin/skill packaging with an owner hard-NO gate; typed primitive contracts across Connections → Missions → Activity → Outputs → Evaluations |
| Update merging | **Shipped** (additive) · **Deferred** (three-way) | The update path is additive and retains local work; the agent lane receives engine updates through the paid carrier. Three-way merge and receive-only paid update controls remain open |
| Mobile live-remote workspace | **In progress** | Phone operating the live workspace over a private mesh; not shipped |
| Turn-truth contract | **In progress** | Every turn self-proving from the app through the runtime and back; not shipped |

Two things this table does **not** claim: that a mechanism proven on the engineering runtime is certified on the shipped customer artifact, and that any in-progress row is complete.

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


### Fresh-install customer evidence — 2026-10-01

A newly installed customer agent completed a first-boot acceptance run without manual repair during the test. The tested path demonstrated durable typed-state writes, automatic exhale, two genuine fresh-process restarts with continuity, explicit supersession, active BM25/vector/FTS/vault retrieval surfaces, an evidence-verified file write, duplicate/retry deduplication, and completed-work recovery after restart.

The run also found defects and therefore is **not** promoted as full customer certification: duplicate active current-decision state, a path-sensitive silent-zero hybrid retrieval route, shared-root residue, MCP stdio banner noise, a dead duplicate Chroma lane, memory/open-work classification hygiene issues, and a doctor false negative.

Public artifact record: fresh-install acceptance — 2026-10-01. The full record is held in the private runtime repository; the paragraph above is the public statement of it.

### Release lane advanced — the line is now 1.0.46

The customer artifact has moved well past the October 1 run. **1.0.41** was the frozen candidate that closed the locale-codec class; the current release line is **1.0.46** (2026-10-15), which republished the runtime source from the live customer runtime — 83 files advanced, none added or removed — and corrected a public conformance suite that had been asserting a contract this engine does not have.

| | |
|---|---|
| Current release line | **v1.0.46** (2026-10-15) |
| Earlier frozen candidate | `cem888_agent-1.0.41-py3-none-any.whl` — 3,963,252 bytes · SHA-256 `0d090610d74ebaa56edaf5ebb7a0efc568f26b859f2cd23e26b30f43350f9d5a` · release commit `5342bac3` |
| Current artifact digest | Recorded with the artifact in the private runtime repository. The digest is the release identity; a version label alone is not |
| Public conformance | **15 / 15 passing**, with two falsifier runs confirmed to turn the suite red |

The 1.0.41 digest is retained deliberately: it is the artifact named by hash, and it stays named.

**The class 1.0.41 closes.** On a process without UTF-8 mode, `subprocess(text=True)` and `open()` / `read_text()` / `write_text()` default to the **locale** codec — cp1252 on Windows — while this pipeline writes UTF-8. Reading UTF-8 as cp1252 either raises on one of the five undefined bytes or silently returns mojibake. Writes are worse: `open(path, "w")` truncates to zero bytes *before* writing, so a raise destroys the previous content too. Three of the four affected sites sit on the path that carries a hook's text into the model's context — which is exactly where the mojibake was first observed. **16 sites repaired across the installer plugin payload and three engine modules; 0 remain.**

**The durable part is the guard, not the fix.** The release gate **parses** each call rather than grepping lines, because the `encoding=` keyword is frequently on a *later* line of the same call — which is precisely how this class survived three separately-found defects. The detector carries a decoy self-check, so a detector that stops detecting reports RED rather than passing silently. *A gate that cannot fail proves nothing.*

The earlier release sequence has been executed through artifact rebuild and publish:

```text
promote customer-relevant changes together
  -> rebuild wheel                    [done — 1.0.40 -> 1.0.41]
  -> rebuild platform bundles         [done]
  -> freeze artifact + digest         [done — see the table above]
  -> publish + verify shipped bundle  [done — health-checked at 5342bac3]
  -> re-run install conformance       [IN PROGRESS — see below]
```

**Still open, and deliberately not claimed.** The two defects named in the previous status — model-originated memory writes acquiring owner authority, and fresh-store concurrent-write loss — remain open in the engineering record and are **not** represented as fixed on a shipped artifact.

The re-exercise also produced findings that are recorded with their measured mechanism but not yet repaired:

- finished current-state entries that do not decay, because the decay predicate is glyph-exact while the seeded example teaches no glyph;
- a seeded capacity figure in the installer that disagrees with the engine's own limit;
- a current-truth uniqueness gap: several decisions can be simultaneously current with no supersession inference, so superseded material can return as current.

Until the remaining items are fixed and the *same* acceptance test re-runs unchanged on a newly frozen artifact, the customer artifact is **not represented as fully conformant**.

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

### Release lane 1.0.38 -> 1.0.41

The 1.0.38 artifact was the one exercised in the October 1 acceptance run; it exposed the install-surface defects described above (memory-authority attribution, fresh-store concurrent-write loss).

The lane cut **1.0.41** — a locale-codec class repair plus a parser-based release guard — and then advanced to **1.0.46**, republished from the live customer runtime. A green source test is still not a shipped fix, which is why artifacts are named by digest and the remaining items are held open rather than declared closed.

---

## Public proof vs. internal proof vs. customer proof

CEM888 is deliberately separating these claims:

| Surface | What it can prove |
|---|---|
| **Live CEM engineering runtime** | Whether the mechanism works on the ancestor/runtime currently being hardened |
| **Published falsifiers + public evidence** | Whether a reviewer can inspect the published checks and their reported results |
| **Frozen customer artifact** | Whether the exact wheel/bundle a customer receives satisfies the release contract |

Runnable falsifiers for authority and context invariants are reported publicly with their results and method. They improve external reviewability, but they do not replace exact-artifact certification.

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
5. **CEM888 runtime source** — private. Source access for audit is granted to licensees and design partners; licensing terms are published in [CEM888AI/legal](https://github.com/CEM888AI/legal).

---

## Current partner policy

Technical discovery, architecture review, and targeted integration planning are welcome now.

The customer release claim remains gated on the rebuilt artifact passing its own conformance checks.

That is deliberate.

A partner should receive a build that can **prove what it does**, not one that merely appears operational.
