# A green test suite over dead code: a source edit is not a deploy

**Status: TESTED (source-level).** The live post-reload confirmation is described in section 5; its results are not included here.

## Summary

Three defects in the runtime's verification and memory-maintenance plugins were repaired in source on one day, and none of the repairs were in effect. The long-lived gateway process had started about 32 minutes before the edits landed. Python imports a plugin once at boot and holds the module in memory for the life of the process. There is no file watch and no hot reload, so the process kept serving the old code while the source on disk was correct.

A source edit is not a deploy.

## 1. The headline defect: a passing suite over code that is not running

| Fact | Value |
| -- | -- |
| Gateway process start | before the repair |
| Plugin source edits written | 32 minutes after process start |
| Result | the process predates the fix |

The decisive evidence was not the timestamp comparison. It was the bug's own fingerprint appearing after the edit. A scheduled maintenance pass ran after the repair and appended a new stale marker while the older one survived. That is exactly the accumulation the repair exists to prevent, and it cannot happen if the fix is running.

Bytecode cache files are not proof either. Python recompiles a changed source file the moment anything imports it, including a test run or a script, while the long-lived process keeps serving the module already in memory. A cache file newer than its source but older than the process is the signature of this exact trap.

Consequence for reporting, now a standing rule: **"in source" is not "in effect", and neither is "in the release the customer downloads."** Those are three claims and each needs its own measurement.

## 2. Three defects that were all inert

### 2a. The verification plugin was recording nothing

A rewrite of the attestation path referenced a variable it never assigned. The result-recording call raised an error on every invocation, and the calling hook swallowed the error at debug log level. From the moment that edit landed the plugin recorded zero receipts, and nothing anywhere reported a problem. The same faulty merge had also dropped a double-fire guard, so where the engine dispatches the same post-tool event from two independent places for one tool call, every result would have been written twice, with the first superseded for no reason.

Test suite for the module: 20 failing before, 42 passing and 0 failing after.

The lesson is about swallowed errors. A hook that catches everything and logs at debug converts a total failure into silence. The failure was visible only because the suite was run, and only useful once the running process was also reloaded.

### 2b. Root cause of a repeating "inconclusive" loop

The result classifier refused to record a pass for any command whose text did not look like a test invocation. Ordinary commands that exited 0 were filed as unparseable, which fails open, which triggers no consumer action, which invites a re-run. That is the origin of the measured finding that 156 of 171 terminal receipts were exit code 0 with an unparseable classification, alongside 103 receipts stuck active.

A dated policy reversal was present in a test but had never been implemented in the code. The test documented behaviour the code did not have. It was restored as one new branch with every other branch untouched, and the anti-overclaim guarantee was preserved: pass and fail counts stay empty for such a receipt, so an exit-0 pass can never be read as test coverage.

### 2c. A stale-marker pointer that accumulated forever

When a section of the agent's scratchpad was retired, a pointer to the retirement was appended into the section body. The next maintenance pass re-parsed that pointer as an ordinary item, carried it forward, and appended a second. One more per pass, and nothing ever removed them.

The pointer is now a status line: it is recognised and dropped at parse time, and its figure is carried forward so exactly one is re-emitted with the same number instead of being lost. Retirement is never silent. Verified against the real scratchpad rather than a synthetic fixture: two stale markers collapsed to exactly one, and the result was a fixed point from the second pass through the tenth.

## 3. A fourth repair: the install path could reach a public package index

An optional-dependency installer ran a fallback ladder of installation methods with no index restriction, so merely enumerating the available tools could resolve a missing optional backend from a live package index at runtime. The ladder was deleted, not gated behind a setting, because a default-on flag is one edit away from being wrong again. The error messages that used to advise the user to install from the public index were themselves teaching the violation, and now give the offline form only. The 8 tests each assert that no child process is spawned. See also: [the update-path case study](case-study-update-path-forbidden-command.md).

## 4. Per-turn cost

Two instruction files that ship with every turn were each just under 100 KB. They were split, never truncated, to about 20 KB each. Losslessness was checked twice by independent methods (per section and by a global line-multiset comparison) with zero lines lost, and no existing reference file shrank.

On the runtime side, an old skill-view result is replaced with a one-line pointer in the copy sent to the model only. Results are identified by their tool-call identity, never by sniffing content, so a terminal result that merely quotes a skill is never deleted. The current turn is untouched, the stored conversation is never mutated, and the operation is idempotent. 8 tests.

The mechanism is tested. The token saving on a live run is not yet measured, and no number is claimed.

## 5. The reload, and how it is verified

The restart is scheduled as a one-shot delayed job so that it fires after the reply ships; restarting mid-turn would kill the in-flight turn. The trigger is deliberately one-shot only. A restart helper that can re-fire is a kill loop by construction, and a previous instance of that mistake produced roughly 35 consecutive failed boots over more than an hour.

Verification is defined to run after the reload, and each check is one that can only pass if the new module is loaded:

1. The process has a new identity, reports a running state with no restart pending, and platforms reconnect.
2. The stale markers in the scratchpad collapse to exactly one on the next maintenance pass. This is a process-level check, not a file-level check.
3. An exit-0 non-test command now records as a pass instead of unparseable.
4. The shutdown line reports whether any turn was in flight, so it is known that no message was eaten.

## 6. What is not claimed

- These repairs were made in working copies and are not pushed to the public release.
- Whether the vendored dependency bundle is complete is not verified. The install-path change means a missing optional backend now reports as unavailable instead of self-healing from a package index, and that may surface on a fresh machine until the wheels are vendored.
- No metered before and after token number for the instruction-file work.
- One named fix in another profile's tree was not touched, because writing across profiles was out of scope without explicit direction.

## 7. Propagation is measured, not hand-copied

An instance defect is treated as a product-scope claim until measured. So propagation starts with a census: every copy of each repaired file across profiles and the served installer is located and hashed. The surfaces that decide what every install receives are measured (release allow-list, payload directory, installer enable lists, and one stock install as arbiter). Each finding is classified as a regression or as never built for that profile, because the classification changes the fix from restore to build. And a fix for new installs is kept separate from a fix for existing installs: the update path has to deliver new payload, not just a new version number.

## Why it matters for agents

An agent that reports success by reading source, tests, or configuration will report success on a system where the running process has not changed. The general form: source, tests and config are claims about intent. Evidence about behaviour has to come from the running process, and from a fingerprint that only the fixed code can produce.

[← All case studies](README.md)
