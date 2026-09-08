# Legacy IBM i: autonomous debugging to a verified live sign-on

**Status: TESTED**

## Summary

CEM was given a real operational objective against a live IBM i 7.5 environment: establish a working 5250 session, complete the account sign-on flow, and continue into a real system task.

The existing client path was not usable on the modern host environment. Rather than stopping at the dependency failure, CEM recovered the prior session state, continued the task, debugged a legacy Python 5250 implementation at the protocol level, corrected the failing field-address behavior, and verified the repair against the live IBM i system.

The result was a successful authenticated sign-on, completion of the required password-change flow, and arrival at the live IBM i main menu.

## Why this case matters

This was not a canned demo and not a synthetic benchmark. The agent had to work through an unfamiliar legacy system where the available tooling itself was broken.

The useful behavior was the full sequence:

**objective -> tool failure -> investigation -> root-cause isolation -> tooling repair -> live verification -> continuation of the original task**

That is the behavior CEM888 is designed to support: the model is not expected to already know every environment or have a perfect connector available. The runtime preserves the work state while the agent investigates, adapts its tooling, verifies the result, and keeps moving toward the original objective.

## What happened

### 1. Resumed an interrupted engineering task

A previous session had ended at the tool-iteration limit while the 5250 client was still being repaired. On the next session, CEM recovered the exact work state and continued from the unverified driver fix instead of restarting the investigation from scratch.

### 2. Replaced a dead client path with a workable one

The existing native 5250 client was unstable in the current environment. CEM moved to a legacy Python implementation, ported the required path to modern Python, and built a minimal driver around it to establish a real session with the IBM i host.

### 3. Diagnosed a real host rejection

The live host accepted the connection but rejected sign-on input. CEM traced the failure through the client library rather than treating the server message as the root cause.

It checked modified-field state, transmit construction, field metadata, wire payloads, and the legacy implementation's response framing.

### 4. Cross-checked against an independent implementation

When the first hypothesis was insufficient, CEM compared the legacy Python client's behavior against the known 5250 implementation in the native client source.

That comparison exposed the actual defect: a coordinate-base mismatch introduced during the Python 3 migration caused submitted input fields to be addressed one row and one column away from the fields displayed by the host.

### 5. Repaired and verified against the live system

CEM corrected the field-address behavior and reran the real sign-on flow.

The IBM i host then accepted the credentials, presented the expected forced-password-change step, and allowed the session through to the main menu.

## Verified outcome

- Live 5250 connection established to an IBM i 7.5 system
- Input-field transmission defect isolated at the protocol/addressing layer
- Legacy client repaired sufficiently to complete the real sign-on flow
- Credentials accepted by the host
- Required password-change flow completed
- Authenticated session reached the live IBM i main menu
- Agent continued toward the original system task after the repair

## What the operator had to do

The operator authorized CEM to begin the proof run and asked to be notified when it was finished.

The debugging sequence itself — inspecting the library, forming and rejecting hypotheses, comparing the wire behavior against a reference implementation, locating the coordinate-base mismatch, applying the repair, and verifying it against the live system — was carried out by the agent during the task.

## What this demonstrates

This case is evidence for several practical runtime behaviors:

- **Persistent task continuity:** the agent resumed interrupted technical work from the prior state.
- **Autonomous troubleshooting:** it did not require step-by-step operator instructions to localize the defect.
- **Tool adaptation:** when the existing tool path failed, the agent repaired the tooling needed to continue.
- **Cross-source technical reasoning:** it compared two independent implementations to isolate a protocol-level mismatch.
- **Real-world verification:** success was determined by the external IBM i system accepting the session, not by the model claiming that its code looked correct.
- **Progressive execution:** fixing the tool was treated as part of the larger objective, not as the endpoint of the task.

## What is intentionally not published

This repository documents observable behavior and engineering outcomes without publishing CEM888's proprietary runtime implementation.

Not included here:

- CEM888 source code
- agent memory/state internals
- private prompts or identity files
- credentials, tokens, host secrets, or account identifiers
- private filesystem paths
- internal tool-routing and orchestration implementation
- proprietary persistence, verification, or provider-routing code

The purpose of the case study is to show what the runtime enabled the agent to accomplish, not provide a recipe for reproducing the CEM888 implementation.
