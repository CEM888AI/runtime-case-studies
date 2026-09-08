# Legacy IBM i: autonomous debugging and live system operation

**Status: TESTED**

## Summary

CEM was given a real operational objective against a live IBM i 7.5 environment: establish a working 5250 session, complete the account sign-on flow, operate the live system, verify changing workload data, inspect system messages, and exit cleanly.

The existing client path was not usable on the modern host environment. Rather than stopping at the dependency failure, CEM recovered prior task state, continued the work, debugged a legacy Python 5250 implementation at the protocol level, repaired the failing client behavior, and verified the repair against the live IBM i system.

After repairing the tooling, CEM authenticated successfully, completed the required password-change flow, reached the IBM i main menu, ran WRKACTJOB against the live system, refreshed the workload view and observed changing telemetry, inspected QSYSOPR messages, and signed off cleanly.

CEM then preserved the working driver as reusable legacy-software-operation tooling so the solved capability can be used again rather than rediscovered from scratch.

## Why this case matters

This was not a canned demo or synthetic benchmark. The agent had to complete a real task in an unfamiliar legacy environment where the available tooling itself was broken.

The observed sequence was:

**objective -> interrupted work recovered -> tool failure -> investigation -> root-cause isolation -> tooling repair -> live authentication -> real system operation -> external verification -> reusable capability**

That is the behavior CEM888 is designed to support: the model is not expected to already know every environment or have a perfect connector available. The runtime preserves enough task state for the agent to investigate, adapt its tooling, verify the result against the external system, continue toward the original objective, and retain useful capability afterward.

## What happened

### 1. Resumed an interrupted engineering task

A previous session ended while the 5250 client was still being repaired. On the next session, CEM recovered the relevant work state and continued from the unfinished driver verification instead of restarting the investigation.

### 2. Replaced a dead client path with a workable one

The existing native 5250 client was unstable in the current environment. CEM moved to a legacy Python implementation, ported the required path to modern Python, and built a driver around it to establish a real session with the IBM i host.

### 3. Diagnosed a real host rejection

The live host accepted the connection but rejected sign-on input. CEM traced the failure through the client library rather than treating the server response as the root cause.

The investigation covered modified-field state, transmit construction, field metadata, wire payloads, response framing, and field addressing.

### 4. Cross-checked against an independent implementation

When the initial transmit-state hypothesis did not fully resolve the problem, CEM compared the legacy Python client's wire behavior against a separate native 5250 implementation.

The comparison exposed an addressing mismatch introduced during the Python 3 migration: submitted input fields were being addressed one row and one column away from the fields displayed by the host.

### 5. Repaired and authenticated against the live system

CEM corrected the field-address behavior and reran the real sign-on flow.

The IBM i host accepted the credentials, presented the expected forced-password-change step, and allowed the authenticated session through to the main menu.

### 6. Operated the live IBM i workload

Authentication was not treated as the finish line. CEM continued the original proof task and drove WRKACTJOB on the live IBM i system.

The workload view showed 1,256 active jobs. A subsequent refresh showed the live system changing — including job-count movement from 1,256 to 1,254 and CPU telemetry changing from 0.0% to 8.5% — providing external evidence that the agent was operating against a real changing workload rather than a static fixture.

CEM also inspected DSPMSG QSYSOPR and completed a clean SIGNOFF.

### 7. Preserved the solved capability

After the proof succeeded, CEM saved the working sign-on/password-change and operational driver paths as reusable legacy-software-operation tooling.

The important outcome is not simply that the agent solved the IBM i task once. The successful path became reusable capability for future work.

## Verified outcome

- Live 5250 connection established to an IBM i 7.5 system
- Input transmission defects isolated in the legacy client path
- Field-addressing defect diagnosed through wire-level comparison
- Legacy client repaired sufficiently for real operation
- Credentials accepted by the live host
- Required password-change flow completed
- Authenticated session reached the IBM i main menu
- WRKACTJOB executed against the live system
- 1,256 active jobs observed during the proof run
- Refresh produced changing live workload/CPU telemetry
- QSYSOPR messages inspected
- Clean SIGNOFF completed
- Working operational path preserved as reusable agent tooling

## What the operator had to do

The operator authorized CEM to begin the proof run and asked to be notified when the job was finished.

The technical sequence — recovering the prior work state, inspecting the library, forming and rejecting hypotheses, comparing wire behavior against a reference implementation, locating the addressing mismatch, applying the repair, authenticating, navigating the IBM i environment, verifying live workload behavior, and preserving the resulting tooling — was carried forward by the agent during the task.

## What this demonstrates

- **Persistent task continuity:** interrupted technical work resumed from relevant prior state.
- **Autonomous troubleshooting:** the defect was localized without step-by-step operator debugging instructions.
- **Tool adaptation:** the agent repaired the tooling required to accomplish the larger objective.
- **Cross-source technical reasoning:** independent implementations were compared to isolate a protocol-level mismatch.
- **Real-world verification:** success was determined by behavior of the external IBM i system, not by a model-generated claim of completion.
- **Legacy-system operation:** after fixing its access path, the agent performed real operational commands against the live environment.
- **Capability accumulation:** the solved access/operation path was retained as reusable tooling rather than discarded after the run.
- **Progressive execution:** debugging, authentication, operation, verification, and capability capture remained parts of one continuing objective.

## What is intentionally not published

This repository documents observable behavior and engineering outcomes without publishing CEM888's proprietary runtime implementation.

Not included here:

- CEM888 source code
- reusable driver source
- agent memory/state internals
- private prompts or identity files
- credentials, tokens, host secrets, or account identifiers
- private filesystem paths
- internal tool-routing and orchestration implementation
- proprietary persistence, verification, or provider-routing code

The purpose of this case study is to show what the runtime enabled the agent to accomplish, not provide a recipe for reproducing the CEM888 implementation.
