# Diagnosing a product that tells its own customers to break its supply-chain rule

**Status: DIAGNOSED** (root cause measured) · **FIXED — shipped and verified inside the published artefact**

## Summary

A runtime shipped with two correct halves of an upgrade path that had never been
introduced to each other.

The delivery side was real: a certified bundle, a signed manifest, an authenticated
release endpoint, and a purpose-built updater that updates the engine and the system
plugin payload together while preserving customer-owned identity, memory, skills and
credentials by construction.

None of it was reachable. The runtime's own update command detected the *kind* of
installation it was running inside, and that detection had no branch for a certified
bundle. It fell through every case to a default that assumed the runtime had been
installed from a package index — and then printed, and in one path **executed**,
`uv pip install --upgrade <product>`.

That command works. That is the problem. It resolves from a remote index, which is
exactly the class of operation the product's own dependency policy forbids after a
supply-chain incident in which an unpinned resolution pulled unrequested packages and
polluted `site-packages` with stray `.pth` hooks, breaking the customer install
pipeline.

So the product's advice to its own customer was to violate its own hard rule — and it
would have appeared to succeed.

## Why this case matters

The interesting failure here is not a crash. A crash is self-announcing.

This is a **correct-looking message**. Every version string in it was real, the command
was syntactically valid, and running it would have produced a plausible result. Nothing
in the output was wrong except the entire premise: that this installation was the kind
of installation that command was for.

That is the failure class that has to be caught by **measurement rather than by
reading**, because a code review of the message's formatting finds nothing, and a
successful run of the command *confirms* the bug instead of exposing it.

## What happened

### 1. Started from a claim, ended at a measurement

The work began as a documentation correction. A work ticket carried a completion claim
whose supporting evidence — plugin discovery counts and a `__pycache__` modification
time — was genuine but described a **different surface** from the one the ticket's
acceptance criteria governed.

The acceptance list demanded, at one item, *"a receipt/log showing old version → new
version and migrations applied."* So the question is narrow and checkable: does that
receipt exist?

### 2. Established the absent receipt by quantifying a thing that cannot be optional

The updater creates a timestamped backup directory on **every** run; the backup step is
not conditional in its flow. Therefore an installation with no backup directory has
never successfully invoked the updater.

Measured on a live customer installation:

| Measurement | Value |
| --- | --- |
| Installed engine (distribution metadata) | `1.0.6` |
| Version reported by the CLI | `1.0.0 (2026.09.12)` |
| Version the release site serves | `1.0.9` (re-cut same day) |
| Update backup directory | **absent** |

Three different version numbers live inside one installation, and no upgrade ever
reconciled them. **An upgrade that never ran produces no record of itself** — which is
why the required receipt could not exist, and why "absent" is the finding rather than
"not found yet."

### 3. Read the detection cascade instead of inferring it

Rather than theorising about why, the resolution order was read directly from source.
It tries, in sequence: an install-method stamp file → a managed-install marker → a
container marker → presence of a VCS directory → **a default**.

A certified bundle install has no stamp and no VCS directory. It therefore returns the
default, which is `"pip"`.

Downstream of that single wrong string:

- the recommendation helper maps `"pip"` to a remote-index upgrade command;
- **nine call sites** deliver that string to a human — several in the update command,
  one in the plugin command, one in the startup banner;
- one path does not merely print it: it builds and **executes** the argument vector.

### 4. Found the root cause by counting writers, not by guessing intent

The stamp file the first branch of the cascade looks for is written by **nothing**.

The identifier is present in the detection code and in the recommendation table. Across
all five shipped installers — three Unix variants, a Windows script, and the teardown
script — the string appears **zero times** as a write.

That is the whole bug: a detection branch reading a file no installer has ever
produced. Quantified absence, not an inference from silence.

### 5. Confirmed the fix is a connection, not a construction

The tempting conclusion — *"customers are stranded, build an update mechanism"* — is
wrong, and was disproven by measurement. The updater is a shipped, standalone script
that lives outside both the engine tree and the server tree. A search of those two
trees for update URLs found nothing, and absence of a URL in engine source is **not**
absence of a path.

Both ends already existed and were built correctly:

- **Client:** an authenticated updater script, installed into the user's home by all
  three Unix installers and symlinked onto the path. It authenticates with the
  installation's provisioning credential, updates the engine **and** the system plugin
  payload, reconciles the enabled-plugin allow-list, and refuses to proceed if a bundle
  ships a path that the installation's layer policy marks as user-protected.
- **Server:** a release endpoint that reads the certified manifest, **verifies the
  archive hash against it**, invalidates any prior unconsumed token, and mints a
  one-time download token.

### 6. Wrote the fix as a two-phase, fail-closed patch

The change adds a bundle method to the detection cascade, keys it on the *absence* of a
VCS directory **together with** the *presence* of the installed updater script, routes
the update command and the check command through it, and makes all four installers write
the stamp beside the identity file they already write.

It was authored as a transformer that first proves every anchor it intends to match
occurs **exactly once**, and only then writes. Any missing or ambiguous anchor aborts
the entire patch with nothing written. It was then run in check mode, which writes
nothing:

```
ANCHORS OK: 12 edits across 6 files, each found exactly once
(check mode -- nothing written)
```

One detail worth preserving, because it is the kind of thing that silently restores the
bug: the Windows installer is written to emit the stamp **without a byte-order mark**,
deliberately. The shell builtin that would have been idiomatic emits a BOM, and the
reader compares stripped text, so a BOM would have quietly put the installation straight
back on the index default.

## Verified outcome

- Root cause established by **reading the resolution cascade**, not by inference
- Root cause confirmed by **quantified absence**: 0 installer writers for the stamp the
  cascade depends on
- Blast radius enumerated: the wrong string reaches a human from 9 call sites, and is
  executed from 1
- The "customers are stranded" hypothesis **disproven** — both ends already exist
- Fix written as a fail-closed, anchor-verified transformer; **12/12 anchors, each
  matched exactly once, verified in a mode that writes nothing**
- Patch artefacts and the pre-patch tree state are recorded outside the repository

**Shipped, and verified in the shipped artefact — not in the branch.** The patch is applied, the
wheel is rebuilt, and the release is published. Verification was done by reading the *published*
bundle rather than the working tree: the wheel inside the bundle carries the change, and the
bundle’s own installer installs the updater **and then** writes the install-method stamp, in that
order — so detection finds both and the certified-bundle branch fires instead of the index default.

**Still not claimed:** that an upgrade has actually completed on a real installation from an old
version to a new one, with the receipt the acceptance criteria required. Exercising that path
consumes a one-time download token — **the first download is what creates a customer’s agent** —
so it is run by the owner on her own schedule. The end-to-end receipt remains outstanding.

**Also not established:** whether the check command should contact the release service
at all. There is **no read-only release-check endpoint**; the only installer route
consumes outstanding downloads and mints a fresh token, so using it to *check* would
mutate state. The shipped shape therefore makes no network call. That is a product
decision, not an engineering one.

## Directionality

This work produced **no inbound path from any code host to any production machine**. No
webhook, deploy key, CI target, or remote points at a runtime or deploy host, and none
was created. The only interaction with a code host in this work was reading it.

## Why it matters for agents

An agent that finds a bug by *reasoning about the message* finds nothing here — the
message is fine. An agent that finds it by *counting the writers of a file the code
depends on* finds it immediately.

That is the whole difference this case is evidence for: **the runtime is expected to
make an incorrect conclusion reachable by measurement.** Every step above is a query
that could have come out the other way. The escalation rule applied here is the same one
that resolves a single-customer symptom into a product claim — one installation missing
a capability is evidence about the *product*, and repairing only that installation hides
the defect and leaves every other installation broken.

---
[← All case studies](./README.md)
