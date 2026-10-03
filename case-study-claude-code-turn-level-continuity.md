# Case study: Claude Code with CEM888 turn-level continuity

**Status: TESTED HOST OBSERVATION (2026-10-03)** — live Claude Code session using the CEM888 continuity hook. This is a host-behavior observation, not a controlled benchmark and not a universal claim about competing products.

## What was being tested

The question was not whether an external host could *store* memory. The question was whether Claude Code, when attached to CEM888, received enough current project state and authority information automatically to behave like an already-briefed project collaborator rather than a fresh model session.

The CEM888 packet presented to Claude included current project state, prior rulings, open work, repo boundaries, deployment constraints, install policy and owner-controlled prohibitions. Claude reported that the host boundary delivered only about half of the declared packet in this run (roughly 12 KB of a declared 24 KB), so the observation below is conservative with respect to the full packet.

## Observed host behavior

After inspecting the injected packet, Claude Code identified several pieces of context it said materially changed how it would work:

- **Repository boundaries.** It knew which repositories were in scope and which were explicitly out of bounds, instead of discovering repositories by sweeping the filesystem.
- **Deployment policy.** It knew the project used the approved deployment path and verification step and that direct shortcuts such as ad-hoc remote edits were prohibited.
- **Install policy.** It knew the exact-pinned, vendored install constraints and said it would otherwise have been likely to suggest a normal package install.
- **Current work and prior decisions.** It received accumulated rulings and open work instead of reconstructing them from a new conversation.
- **Owner authority.** Claude reported that the CEM888 prohibition gate had already blocked an attempted edit to `settings.json` before tool execution.

Claude summarized the difference this way:

> "Without it, I'm a smart stranger every session. With it, I'm a smart colleague who was briefed before walking in."

It separately described the effect as not increasing the model's raw intelligence, but reducing project drift by pulling each turn back toward current truth, current work and explicit boundaries.

## Concrete engineering consequence

When asked to investigate the Codex continuity connector, Claude did not begin by asking for the project architecture. It explored the installed connector and reported a concrete profile-guard contradiction:

- `cem_hook.py` expected `onze`
- `hook_runner.py` expected `nove`
- `lifecycle.py` expected `onze`
- the active connector profile was `nove`

Claude concluded that the contradictory guards caused `begin_turn` to fail before a Codex turn could complete.

The important observation is not that Claude found one bug. It is that the host began the investigation with project-specific operating constraints, state and scope already available, instead of requiring the owner to rebuild that context manually.

## What CEM888 is doing here

CEM888's continuity path is broader than a memory lookup:

```text
TURN START
  -> resolve agent / owner / task
  -> load authoritative current state
  -> exclude superseded state
  -> rank + compile the bounded working packet
  -> inject current decisions, open work, scope and constraints

BEFORE PROTECTED ACTION
  -> resolve target + scope
  -> apply owner authority
  -> block when the host exposes an enforceable tool boundary

TURN FINISH
  -> commit resulting state
  -> preserve continuity for the next session / host
  -> emit receipts / verification evidence where applicable
```

The model can change. The host can change. The state authority does not move into the model.

## Competitive check performed after the observation

Because Claude characterized this behavior as unusual, we checked current public documentation for adjacent systems rather than publishing a blanket "nobody else does this" claim.

### What already exists elsewhere

- **Letta** provides stateful agents whose memory and identity can persist across sessions, computers and interfaces.
- **Mnemoverse** provides a shared memory layer across Claude Code, Cursor, VS Code and ChatGPT.
- **Supermemory** provides cross-session and cross-tool memory through MCP.
- **Mem0** provides persistent memory usable from Claude and other MCP-compatible clients.
- **Zep** can automatically inject retrieved context before turns in supported agent integrations.
- **LangGraph** provides thread checkpoints and long-term stores for application-defined agent state.

### The important boundary

Public documentation for Mnemoverse explicitly says that merely connecting the MCP server **does not make the assistant use the memory automatically**; the user is instructed to add standing memory-use instructions.

Zep documents automatic pre-turn context injection in supported integrations, but its own security guidance states that memory context **does not guarantee model behavior or authorize external actions** and that the application must separately control permissions and action authorization.

OpenAI's published Codex guidance uses `AGENTS.md`, persisted thread state and repository knowledge to provide durable context, but those mechanisms are host/repository scoped rather than an external provider-neutral runtime that owns current project state and action authority across multiple model hosts.

## Defensible differentiation

The result of this review is **not** "CEM888 is the only persistent-memory system."

A narrower claim is supported:

> **In the systems reviewed, we found products that provide cross-tool memory, and products that provide automatic context injection, but we did not find a documented system that combines CEM888's provider-neutral cross-host continuity with per-turn compilation of authoritative project state, explicit lifecycle state, and owner-enforced pre-tool action prohibitions as one runtime.**

That claim should continue to be treated as a dated competitive observation, not a permanent exclusivity claim.

## Why this matters

Memory answers: **"What might be useful to remember?"**

CEM888 is designed to answer a stricter set of operational questions on every turn:

1. **What is true now?**
2. **What work is actually in flight?**
3. **What has been superseded?**
4. **What is this host/model allowed to do?**
5. **What context is relevant enough to inject now?**
6. **What evidence proves the action happened?**

That is why the Claude Code observation matters: the value showed up as changed engineering behavior, not merely successful recall.

## Sources checked

- Letta stateful agents: https://docs.letta.com/concepts/stateful-agents
- Mnemoverse Claude Code / cross-tool memory: https://mnemoverse.com/docs/api/claude
- Mnemoverse MCP behavior: https://mnemoverse.com/docs/api/mcp-server
- Supermemory MCP: https://supermemory.ai/mcp/
- Mem0 Claude persistent memory: https://mem0.ai/library/llms-and-models/adding-persistent-memory-to-claude-with-mem0
- Zep AutoGen memory / automatic context injection: https://help.getzep.com/autogen-memory
- Zep agent-memory boundary: https://help.getzep.com/v3/agent-memory-solution
- LangGraph persistence: https://langchain-ai.github.io/langgraphjs/how-tos/persistence-postgres/
- OpenAI Codex persistent context: https://openai.com/business/guides-and-resources/how-openai-uses-codex/

---

[← All case studies](./README.md)
