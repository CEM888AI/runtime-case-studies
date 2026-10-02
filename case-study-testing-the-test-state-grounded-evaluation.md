# Case study: when the test was wrong, the agent tested the test

**Status: TESTED INCIDENT — one fresh-install/release-validation episode; not a cross-product benchmark**

## Summary

During Windows fresh-install validation of CEM888 **v1.0.41**, an external AI evaluator
(ChatGPT) produced a follow-up acceptance test containing a mixture of valid checks and
incorrect assumptions about the product.

A freshly installed CEM888 agent attempted the test literally and produced a broad
`FIX VERIFICATION FAIL`. A long-lived CEM888 engineering agent then reviewed the raw
evidence against the actual release contract and did something materially different from
blind rubric compliance:

- rejected a reported failure that was actually intended product behavior;
- separated pre-existing machine state from files created by the installer;
- preserved failures that were real instead of dismissing the whole test;
- found two additional defects by reading the shipped bytes/source rather than trusting
  the report;
- refused to push fixes while the frozen **1.0.41** artifact was still under test, because
  doing so would change the served revision and invalidate the measurement.

The useful result was not "the agent disagreed with the evaluator." It was that it
**reconciled the evaluator's instructions against authoritative product state, release
rules, code behavior, and the mission of the test.**

## Context

The run was a real customer-style Windows install. The tested agent reported:

- runtime: **CEM888 Agent v1.0.41 (2026.09.12)**
- Python: **3.14.7**
- running/default model: **deepseek-v4-flash**
- customer profile: `testing10`
- profile root: `C:\Users\azureuser\.cem888\profiles\testing10`
- customer vault: `C:\Users\azureuser\testing10-Vault`

Before the follow-up test, the release owner had verified the wheel *inside the actually
served Windows ZIP*:

- served carrier: `cem888-windows.zip`
- release revision: `5342bac3`
- wheel version: **1.0.41**
- wheel hash prefix: `0d090610…`
- one wheel served; no stale alternate wheel observed

That mattered because any failure in the run could then be treated as a property of the
served artifact rather than hand-waved as "maybe the customer downloaded stale bytes."

## What the external evaluator got wrong

### 1. A blank day-one objective was marked as a failure

The generated test asked the fresh agent to read its world state's
`PRIMARY_OBJECTIVE`, then treated a blank value as a failure.

The installed bytes reported:

```text
PROJECT: testing10
PRIMARY_OBJECTIVE:
RECENT_DECISIONS: - use postgresql for the test system
NEXT_ACTION:
```

The engineering agent rejected that failure.

Why: the installer had intentionally been changed so it **does not seed vendor roadmap
fiction into a customer's world state**. An older observer path had inserted CEM888's own
roadmap into customer installs, causing customer agents to repeat goals such as serving
"millions of users" as if those were the customer's goals.

For a brand-new customer with no real project objective yet, **blank is the correct
state**.

Changing the product to make this test green would have reintroduced the contamination
bug the release had just removed.

### 2. Pre-existing VM state was at risk of being misread as installer contamination

The follow-up test also treated the whole shared `~/.cem888` root as though every file
under it must have been created by this one install.

The actual filesystem diff separated the two:

**Created by the installer:**

```text
~/.cem888/.install_method
~/.cem888/active_profile
~/.cem888/bin/cem888.cmd
~/.cem888/connector/agents.json
~/.cem888/profiles/testing10/
C:\Users\azureuser\testing10-Vault\
```

**Already present before the install:**

```text
mcp-host/
scripts/
kanban.db
profiles/{exec,testing2..testing9}
their corresponding vaults
```

The engineering question was therefore not "is this reused VM globally empty?" It was
"what did this installer create, and was that output customer-scoped?"

That distinction prevented a reused test machine from being mistaken for a contaminated
installer payload.

### 3. An unexecuted restart test was not promoted into evidence

The 12-test report explicitly said its final restart check had **not** been performed as a
live process restart; it had only performed a disk/INHALE-equivalent inspection.

The engineering agent preserved that boundary rather than treating the numbered checklist
as proof merely because a line item existed.

## What the evaluator got right

The engineering agent did **not** use the bad questions as a reason to dismiss the whole
test.

It retained the substantive current-truth findings.

### Current truth still has a real unresolved case

The public typed-memory path supports explicit supersession. When a caller supplies
`supersedes=<old_id>`, the old row is closed and becomes historical.

But two differently worded statements about the same logical fact produce different
content hashes. Without a layer that recognizes that they refer to the same state key,
both can remain active.

The useful distinction is:

- **explicit supersession works;**
- **semantic/logical current-truth resolution for differently worded statements remains a
  real engineering problem.**

The engineering agent kept that issue open instead of "fixing the test" by declaring the
whole result invalid.

## What the CEM888 agent found that the evaluator did not

The engineering agent read the shipped behavior/source and identified two additional
defects.

### 1. Scratchpad decay and the fresh-install convention disagreed

The decay rule recognized finished markers `✅` / `❌`.

The fresh-install agent naturally wrote completed entries using `[✓]`.

The seed taught the scratchpad section names but did not teach the completion-marker
convention. As a result, completed work could remain under `## CURRENT` indefinitely.

Observed fresh-install state:

```text
## CURRENT
- [✓] Suite COMPLETE...
- [✓] Report delivered...
```

The correct fix was narrow:

1. recognize the completion glyphs actually produced by fresh customer data; and
2. teach the convention in the seed.

This was not evidence that the context architecture needed a new cap or a redesign.

### 2. Seeded doctrine still described an obsolete AGENTS.md limit

All four installers still told the agent:

```text
Cap 20,000 characters.
```

The current engine limit was **32,000**.

An earlier false "Budget 8,000" doctrine statement had already been removed, but this
sibling stale value remained.

Again, the fix was a documentation/seed correction, not an architecture change.

## What the shipped Windows artifact did prove in the same run

The raw answers independently confirmed several recent Windows fixes were present in the
served **1.0.41** bytes:

| Check | Observed result |
| --- | --- |
| Runtime version | `CEM888 Agent v1.0.41` |
| UTF-8 Windows write/read | em-dash, arrow, curly quotes, and checkmark round-tripped byte-identically |
| Identity payload | middle of `SOUL.md` present; no identity-truncation doctrine found |
| USER scaffold | 1,091 characters; `How They Want You To Work` present |
| Model selection | running model and DeepSeek default both `deepseek-v4-flash` |
| Doctor venv check | `✓ Virtual environment active` |
| dotenv MCP quieting | `DOTENV_CONFIG_QUIET=true` present |
| last-session context | `state/last_session_context.md` present and 4,711 bytes |

These are scoped observations from this artifact. They do not certify unrelated runtime
capabilities.

## Release discipline: why the agent refused to push

The engineering agent had candidate fixes ready, but deliberately did **not** push them.

Reason: the customer was still testing the pinned **1.0.41** carrier. The release process
re-cuts customer carriers on push. Landing a fix during the run would therefore change the
artifact under test and destroy the evidentiary value of the result.

The agent explicitly kept the fixes staged until the current run was complete.

That behavior is important because "fix the bug immediately" and "preserve the validity
of the measurement" conflict in this situation.

The mission was not:

> make every test line green as quickly as possible.

The mission was:

> determine what is true about the exact customer artifact currently being tested.

The agent chose the latter without being reminded.

## Why this matters

Long-lived engineering agents receive bad instructions sometimes.

The bad instruction may come from:

- a human;
- an AI evaluator;
- an outdated test;
- stale documentation;
- a copied checklist;
- a plausible but incorrect assumption about the architecture.

A model that treats the newest instruction as the whole source of truth can optimize for
the rubric and damage the system it was supposed to evaluate.

In this incident, the useful behavior was:

```text
NEW TEST / INSTRUCTION
        |
        v
COMPARE AGAINST ACTUAL ARTIFACT + PRODUCT CONTRACT
        |
        +--> incorrect assumption -> reject / reclassify
        |
        +--> genuine defect -> preserve / investigate
        |
        +--> missing evidence -> do not claim
        |
        v
PROTECT THE RELEASE + CONTINUE THE REAL MISSION
```

That is the architectural thesis in operational form:

> **STATE decides what is true. MODELS decide what to do about it.**

The evaluator supplied reasoning and a proposed rubric. The runtime-backed engineering
agent retained enough authoritative product/release context to evaluate the rubric rather
than merely obey it.

## Evidence boundary

This is a **tested incident**, not a general benchmark proving that CEM888 always detects
bad instructions or that another coding agent never can.

What this incident establishes is narrower:

- an external AI evaluator supplied materially flawed acceptance criteria;
- a fresh CEM888 install exposed raw results from those criteria;
- the long-lived CEM888 engineering agent independently distinguished at least one false
  failure from real defects;
- it preserved unsupported evidence boundaries;
- it found additional defects outside the evaluator's diagnosis; and
- it protected the pinned artifact from a mid-test release mutation.

A controlled comparative study against bare Codex, Claude Code, or other harnesses would
be required before making a quantitative cross-product claim about this behavior.

## Engineering lesson

A reliable agent should not merely remember instructions.

It needs enough durable project state, authority, evidence, and release context to answer
a harder question:

> **Should this instruction be trusted as a description of reality?**

In this run, the answer was sometimes no — and the agent stayed on the actual job.

---

[← All case studies](./README.md)
