# Case study: worker spin without progress — the containment gap

**Status: TESTED**

## Problem

A hardened agent gateway stopped responding to every platform for 40 hours while consuming a full CPU core. Every deterministic loop protection — failure counters, duplicate-call guards, iteration budgets, an idle watchdog, and cooperative cancellation — was enabled and configured correctly. **None of them fired.**

## Symptom

- Gateway process pinned at 99–100% CPU continuously for 40 hours.
- Zero agent activity logged for the entire window: no turn started, no tool completed, no response delivered. Only a periodic memory-monitor heartbeat and platform reconnect noise.
- The in-flight session froze mid-turn, and no new message could be processed.
- Preceding event: the same gateway had already died once via its own whole-process containment exit, minutes earlier, after a turn worker *ignored* an inactivity interrupt while a tool call hung for 30 minutes.

## Root cause

A turn-worker thread was stuck **inside a single `json.dumps()` call in the C encoder**. Process-level sampling showed one thread at 100% CPU whose stack was, top to bottom:

```
json encoder (listencode_obj ↔ encode_key_value recursion, allocating)
  → _PyEval_EvalFrameDefault          (python frame: the dumps call site)
  → context_run                        (asyncio context inside the worker)
  → thread_run                         (raw worker thread)
```

The trigger: a per-inference telemetry block serialized the **entire outgoing request payload twice on every API call** — the full message list plus the full system-message list, with `sort_keys=True, default=str` — solely to compute sha256 prefixes for prompt-cache observability. The payload had grown pathological during a long diagnostic turn, and one of those serializations became effectively non-terminating.

Why the deterministic layer could not see it — three distinct gaps:

1. **Failure counters can't count a hang.** Every loop guard counted *failures* and *duplicates* (same-tool failure, exact failure, idempotent no-progress). A single call that never returns produces zero failures and zero duplicates. All counters stayed at zero.
2. **Cancellation was cooperative only.** Task cancellation and interrupt events are observed when the worker returns to Python. A worker blocked inside a C extension cannot observe anything until the C call returns — and it never did.
3. **The idle watchdog fired, but its enforcement was the same cooperative interrupt.** The watchdog correctly detected 30 minutes of no activity; the interrupt was ignored by the C-blocked worker. The only remaining containment was whole-process exit — which the previous process incarnation had to take, and which left the recovered session to replay the same failure.

## Engineering constraint

Deterministic agent control (budgets, guardrails, containment) is only as strong as its ability to observe *progress*. Anything that can block a worker inside a C extension — unbounded serialization, hashing of a growing payload, native I/O — is invisible to every cooperative, failure-counting control. Telemetry added to make the runtime more observable introduced a new unbounded serialization site, which became the hang.

## Solution

1. **Bounded the telemetry serialization.** Instead of dumping the full payload to hash its prefix, serialize incrementally per message, cap each chunk, and stop once past the deepest hashed prefix depth. Worst-case cost per inference is now bounded (~64 KB + one chunk) instead of O(full payload), and a pathological message can no longer spin the encoder.
2. **Restarted the gateway under its supervisor** so the accumulated fixes that had landed *after* the process booted — and were therefore never loaded — came in together with the patch.
3. **Verified the loaded code via the runtime's own boot-time artifact manifest.** The gateway logs content hashes (sha12) of the live loop modules at startup; the new boot banner showed the patched hash, proving the fix — not a stale copy — was the code running.

## Measurement

- **Before:** 99.4% CPU sustained for 40 hours; 2,368 CPU-minutes burned on a single thread; zero completed turns; unresponsive on every platform.
- **After (restart + patch):** CPU 0.0% at 51 s uptime; all platforms connected; process sampling showed a flat thread distribution (57 threads, no concentration); boot manifest displayed the patched module hash.
- **Telemetry semantics preserved:** the prefix-stability hashes still report the same signals; only the cost model changed (bounded instead of unbounded).

## Why it matters for agents

The failure mode is generic: an agent runtime can hang anywhere a synchronous call doesn't return, and every layer of deterministic scaffolding added on top (telemetry, hashing, verification, persistence) introduces new serialization sites. Three rules fall out:

- **Every per-turn and per-inference serialization must be size-bounded.** Full-payload dumps for observability are a hang vector dressed as a feature.
- **Loop protection needs a wall-clock no-progress signal with non-cooperative containment** (process-level restart), not just cooperative cancellation — a worker in C code cannot be asked politely to stop.
- **Boot-time artifact manifests pay for themselves.** Content hashes of the live modules turned "which code is actually running?" from an hour of archaeology into a one-line answer.

---

[← All case studies](./README.md)
