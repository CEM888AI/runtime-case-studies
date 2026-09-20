# "Installed and enabled" is not "loaded"

**Status: TESTED**

## Summary

A runtime plugin was present on disk, correctly versioned, byte-identical to its source
of truth, and switched on in configuration. It was doing nothing.

The configuration file said `enabled: true`. The plugin directory existed with the right
contents. A file-level audit of the entire installation came back clean. And the running
gateway had started **before** that configuration landed — so its plugin list had been
read, evaluated, and frozen at process start, and the newly enabled plugin was not in it.

Nothing anywhere reported a problem. There was no error, no warning, and no degraded
state. The plugin was simply absent from the running process while being continuously
present in every artifact that could be inspected.

The distinguishing evidence is one comparison: **process start time against
configuration modification time.** Everything file-level looks correct in the broken
state, which is precisely why "installed and enabled" is not evidence that anything is
running.

## Why this case matters

Configuration that is read once at startup is a **snapshot**, and a snapshot that is
never re-validated will always disagree with the thing it snapshotted.

This produces a specific and dangerous verification failure: the natural way to check
whether a feature is active is to inspect the feature's *definition* — does the file
exist, is it the right version, is the switch on. In this failure mode, **all three
answers are yes while the feature is inert.** A verifier who trusts that evidence
reports success on a system where the capability is entirely absent, and the report is
wrong in the direction that matters, because it is wrong in the direction of
overclaiming.

## What happened

### 1. Two stacked causes, not one

The investigation had been framed as a single bug: the capability was missing. It was
actually two independent defects that produced the same symptom, and fixing either one
alone would have left the symptom in place.

**Cause one — a stale copy in a config repository.** The installer-owned plugin payload
was stale in a configuration repository for **13 of 15** carried plugins. The canonical
installer carried the current payload; the copy that actually deploys into the live
profile did not. So a deploy that was *correct at every step* still handed the runtime old
code. This is the substantive one, and it is the whole bug at the payload layer.

**Cause two — a configuration snapshot that predated the config.** The plugin was
switched on in configuration, but the running process had been started before that config
landed. The file was on disk, switched on, and read by nobody.

### 2. Corrected the stale payload without deleting anything

The 16 installer-owned plugins were brought byte-current against the canonical installer
payload, one plugin that had gone missing was restored, and the count of deletions was
**zero**. A structural validator confirmed the result.

A missing plugin restored rather than reinstalled matters: restore preserves any
customer-local state, while reinstall is a write that has to be justified.

### 3. Proved the load by the only evidence that can prove it

The count of discovered-and-enabled plugins moved from **23/23 to 24/24** — sixteen agent
plugins plus eight providers. That is a count, and a count is a claim about discovery.

The independent evidence was a **`__pycache__` timestamp landing in the restart minute**:
Python does not write a bytecode cache for a module it has not imported. That is the
process itself saying it loaded the code, and it is not obtainable by editing files.

### 4. Ran the same audit on a second installation and found the same shape

Cross-checking a second, independently installed runtime showed the same configuration
saying the same thing — **`enabled`, and therefore assumed active** — while the deployed
copy diverged from the canonical payload. The failure was not one machine's mistake; it
was the shape of the deploy surface.

The correct scope conclusion: a single installation missing a capability is **evidence
about the product**, not a broken installation. Repairing the one box would have hidden
the defect, made the box a contaminated measurement surface, and left every other
installation broken. So the surfaces that determine what **every** installation receives
were measured — the release manifest allow-list, the payload directory, the installer
enable lists, and one stock installation's own configuration as arbiter — and were made
to agree.

## Verified outcome

| Check | Method | Result |
| --- | --- | --- |
| Plugin payload currency | Byte-compare against canonical installer payload, 16 plugins | current, **0 deletions** |
| Structural validity | Validator (`valid: True`) | pass |
| Native load | `__pycache__` timestamp vs restart minute | bytecode written **in the restart minute** |
| Runtime discovery | Plugin registry count | **23/23 → 24/24** |
| Structural honesty | Manifest declares what the code registers | hooks declared **⊇** hooks registered |

The last row, added during this work, closes a silent-failure mode: **a manifest that
does not declare a hook the code registers produces a dead hook with no error at all.**
Nothing raises. The hook simply never runs. So the check is not "does the code register
it" but "does the declaration contain it" — and it is asserted, not assumed.

**Not available — and therefore not reported:** the exact operational cost of the
restart, and any measure of how long the capability had been inert. The runtime did not
expose either, and neither is estimated here.

## Directionality

The plugins in question execute entirely locally. No part of this work introduced a
component that contacts a remote service, and the payload audit ran read-only against
local copies. The capability is inspectable and verifiable without leaving the machine.

## Why it matters for agents

The failure here is not that something broke. It is that **every cheap check said the
system was healthy while the capability was absent** — and the expensive check that said
otherwise was a timestamp on a bytecode cache.

An agent doing verification by inspection will confidently report success on this system.
An agent that has internalised *"a configuration snapshot is not a running process"* will
reach for the one comparison that settles it. The general form, worth more than the
specific bug:

> When a capability is controlled by configuration, **the config file is a claim about
> intent, not evidence about behaviour.** Evidence about behaviour has to come from the
> process.

That is the same discipline as refusing to read a shutdown code as a pass, refusing to
count a cancelled run as green, and refusing to accept "installed" as a synonym for
"running" — the standing rule that **a verification document which fills a gap with a
plausible answer is worse than one that names the gap.**

---
[← All case studies](./README.md)
