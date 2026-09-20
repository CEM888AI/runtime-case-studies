# Why a one-line change to a shipped runtime stalls a release

**Status: VERIFIED PROCESS** (adopted after two recorded failures)

## Summary

A shipped runtime is delivered as more than source. It is delivered as a sealed artifact;
that artifact is **declared** in several independent places, each of which can disagree
with it; and a reconciling gate sits between the branch and the release.

The practical consequence is counter-intuitive in exactly the wrong direction: **the
smaller the change, the easier it is to stall the release.** A one-line edit to shipped
code invalidates the sealed artifact without touching any of the declarations that
describe it. The gate then correctly refuses to publish — an artifact whose declared
identity no longer matches its bytes — and the release stops until someone rebuilds and
re-declares.

The rule that came out of this is: **a change to shipped runtime code is atomic, or it is
a stalled release.** Edit → rebuild whatever the edit invalidated → re-declare every site
that names the artifact → verify → one commit → one release.

## Why this case matters

This is a process case study rather than a debugging one, and it is included because the
failures were **procedural, not technical**. Both were made by a competent agent doing
something locally reasonable, and neither was caught by any test.

The second one is the instructive one, and it was a *publishing-integrity* failure: a
sweep-style staging command carried an unrelated generated file into a commit, and that
file **advertised three plugins the release did not contain.** The commit was clean, the
push succeeded, and the release shipped a customer-facing document that overstated what
the customer was receiving.

Nothing failed. The product lied slightly, in a file no test reads.

## What happened

### 1. Failure one — the change that could not ship

A source-only edit was made to shipped runtime code. It sat unshippable for **days**,
because the release gate compares the sealed artifact against the source that is supposed
to have produced it, and a source edit invalidates that comparison without any part of
the toolchain being wrong.

The fix is not to weaken the gate — the gate is doing its job. The fix is to stop
treating "edit the source" as a complete unit of work. The unit of work is the **train**.

### 2. Failure two — the sweep that shipped an overstatement

A sweep-style staging command was used in a working tree that contained unrelated
generated output. The generated file was stage-adjacent, and it went into the commit.

That file declared **three plugins the release payload did not contain.** So the release
advertised a capability it did not ship. Every gate that existed passed: the artifact was
internally consistent, the tests were green, the push succeeded.

The lesson was recorded as a rule: **stage by explicit path, always.** A sweep in a tree
containing in-progress work makes a commit unreviewable, and an unreviewable commit that
touches a customer-facing declaration is a publishing-integrity failure even when the
push itself succeeds.

### 3. The declarations, enumerated

The sealed artifact's identity is asserted in **five** places, and the version itself is
declared in a sixth that the build asserts against. Any change to the artifact's bytes
must re-declare the **digest** in each site that carries it.

Two more invariants sit beside the digest set, and they are the ones a rebuild silently
breaks:

- a manifest-on-disk invariant (what the release *declares* must match what the payload
  *contains*), and
- a **payload completeness** invariant: the vendor payload is hash-pinned, and an
  incomplete payload is **refused by design** rather than partially installed.

That second one is a feature, not a bug. A dependency missing from the offline payload
**must** fail the build; the failure is the mechanism that keeps customers off remote
indexes.

### 4. Re-cut in place versus bump — and the pin-string consequence

Two changes exist and they are not the same operation:

- A **contract** change alters what the artifact promises. It takes a version bump.
- A **fix** alters only the implementation. It is **re-cut in place at the same version.**

Re-cutting in place has a concrete operational advantage beyond tidiness: because the
artifact's *name* does not change, there is no entry to add to the removal list and no
pin string to rewrite across the pipeline. Only the bytes and the digests that describe
them move. Fewer edits in the critical path means fewer places to be wrong.

There is a working precedent for this: a geometry fix to a desktop-control backend was
re-cut at the same version, changing the wheel digest while leaving the filename
identical.

### 5. The train, as adopted

The ordered discipline that came out of both failures:

1. Ground the change in the current tree state — is it clean, is it aligned with the
   release branch. **No edit begins from an assumed state.**
2. Make the edit.
3. **Rebuild everything the edit invalidated.** A source change invalidates the sealed
   artifact; nothing downstream is trustworthy until it is rebuilt.
4. **Re-declare every site that names the artifact**, plus the build-time assertion
   between the source version and the release.
5. **Stage by explicit path.** Never a sweep. Confirm the staged set is exactly the
   intended set, then confirm the unrelated in-progress work is *still* unrelated and
   still uncommitted.
6. Run the test suite.
7. Verify the declarations against the artifact — not against the build log.
8. **One commit.**
9. **One release action.**
10. **Poll the destination** to confirm the release actually landed. The release action
    returns before its effect is visible, so a report written from its return value is
    unverified.

## Verified outcome

- Both failure modes recorded with their cause, not just their symptom
- The declaration set enumerated — **5 digest sites + 1 version source**, all of which a
  rebuild must move together
- The two rebuild-breaks-something invariants named: manifest-vs-payload, and
  payload-completeness-fails-closed
- The re-cut-in-place precedent recorded, with its pin-string consequence
- The train adopted as a 10-step ordered procedure with a fail-closed staging rule

**Not claimed:** that this procedure prevents all release defects. It prevents the two
that were recorded. It is a description of a pipeline that **fails closed** — which is the
property actually being relied on, and which is only as good as the next unforeseen way
to make the declarations and the artifact disagree.

## Directionality

This procedure governs an internal release pipeline. It introduces **no inbound path from
any code host or external service to a production machine**, and no such path is required
by any step in it.

## Why it matters for agents

Two properties make this case worth recording.

**First, the failures were invisible to testing.** Both commits were clean, tests were
green, and the pushes succeeded. One of them shipped a customer-facing document that
overstated the release. The only thing that catches that class is a **staging discipline
that refuses to sweep** — and a rule about staging is a rule an agent will violate by
default, because sweeping is the convenient action and the failure is silent.

**Second, the gate that stalled the first failure was right to stall it.** It would have
been easy to read the stall as obstruction and to weaken the comparison that caused it.
The correct read is that the pipeline had a working safety property and the *process* was
wrong about the unit of work. An agent that treats a refusing gate as an obstacle to be
cleared has inverted the safety model; the gate is the only part of the system that cannot
be talked out of a claim.

---
[← All case studies](./README.md)
