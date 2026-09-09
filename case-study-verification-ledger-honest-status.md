# Case study: verification ledger honest status

**Status: TESTED**

## Problem

A deterministic verification layer recorded one structured row for every shell and
code-execution tool call, then assigned each row a verdict. The ledger was meant to be
the runtime's durable memory of *what has actually been proven*. Within days it read as
broken: of 200 records, 182 carried the status `UNPARSEABLE`. Any operator or model
reading that ledger would reasonably conclude the verification system itself was 91%
non-functional.

## Symptom

- 200 records: **182 `UNPARSEABLE`**, 12 `ERROR`, 6 `PASS`, **0 `FAIL`**
- Exit-code distribution: the value `0` appeared exactly **182** times — identical to the
  `UNPARSEABLE` count
- A verification ledger that had never once recorded a failure, while nine tenths of it
  was unreadable

## Root cause

The classifier ran for *every* shell / code-execution call, but its status ladder only
had buckets for outcomes that look like tests:

- `FAIL` — only when a test framework's own summary line reported failures, or a nonzero
  exit accompanied a command that *looked like* a test run
- `PASS` — only when a test summary was parsed, or exit 0 accompanied a test-looking command
- `ERROR` — only when output was empty
- otherwise → `UNPARSEABLE`

A successful command that made no test claim — a directory listing, a `git status`, an
ad-hoc data probe — matched none of the positive buckets and fell through to
`UNPARSEABLE`. The records were perfectly well-formed. The schema simply had **no state
for "executed, no claim."** The exit-code histogram is the proof: `UNPARSEABLE` was
exactly equivalent to `exit_code == 0 && not a test run`, 182 records for 182 exit-zero
results.

## Engineering constraint

The classifier's conservatism was deliberate and correct: never claim `PASS` for a
command that merely exited 0. That rule must not be weakened. So the fix could not be
"call these PASS" — it had to add a *new, honest* state that neither claims verification
nor mislabels the record as corrupt.

## Solution

A fifth status, `EXECUTED`, was added for exactly the case the schema lacked: exit code 0,
output present, no test or verification claim. `UNPARSEABLE` is now reserved for what it
actually means — output that *should* carry a verdict but could not be read.

Downstream retirement logic already treats every non-`PASS` / non-`FAIL` status as
fail-open (raw evidence is preserved, nothing is compacted), so the new state is
behaviourally identical to before. Only the label changed — from "corrupt" to "truthful."

## Measurement

- Classifier unit cases: **6/6 pass** — `ls -la` → `EXECUTED`, `pytest` → `PASS`,
  `pytest` with failures → `FAIL`, exit 127 → `ERROR`, empty output → `ERROR`,
  ad-hoc code probe → `EXECUTED`
- Plugin's own test suite: **40 passed / 0 failed** (one pre-existing test asserted the
  old, incorrect behaviour and was corrected alongside the fix)
- Downstream behavioural delta: **none** — every consumption site was read before the
  change; non-`PASS` / non-`FAIL` records were already fail-open
- The "91% corrupt" reading was an artifact of labelling, not of data loss

## Why it matters for agents

A verification ledger is only useful if its *worst-looking* number is trustworthy. A
schema with no way to say "this ran, and I am making no claim about it" will eventually
say "unparseable" instead — and an agent that reads its own ledger will then believe it is
failing when it is not. Honest negative states are a correctness feature, not a cosmetic
one: they preserve the difference between *"I could not verify this"* and *"there was
nothing here to verify."*

---
[← All case studies](./README.md)
