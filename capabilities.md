# Engineering capabilities

Technologies used in CEM888, connected to the systems they're actually used for — not a badge wall.

| Technology | Used for |
|---|---|
| Python | Primary runtime language — agent loop, context compiler, state/memory layer, tool governance |
| FastAPI | Internal service APIs between runtime components |
| SQLite | Local-first durable state and memory storage — the thing that makes "your machine, your data" true rather than a slogan |
| ChromaDB | Vector store for semantic memory retrieval |
| BM25 | Keyword retrieval, run alongside ChromaDB as a hybrid retrieval pair (semantic + lexical) so retrieval doesn't fail silently when a query is more keyword-shaped than semantic |
| MCP (Model Context Protocol) | Standardized tool/connector interface — how the runtime exposes governed tools to whichever model is driving |
| REST APIs | Provider calls (Claude, GPT, DeepSeek, Gemini) and internal service communication |
| Node.js | Integration/acceptance testing harnesses alongside the Python runtime |
| Playwright | Browser automation for agent-driven web tasks |
| AppleScript | Native macOS application control (part of desktop-level tool governance on Mac) |
| Matrix | Self-hosted chat protocol — one of the live agent interfaces, chosen over proprietary chat platforms for the same local-first reasoning as the rest of the stack |
| Telegram | Live agent interface / messaging surface |
| Slack | Live agent interface / messaging surface |
| DigitalOcean | Always-on host for the primary runtime instance |
| nginx | Reverse proxy / routing in front of runtime services |
| systemd | Service supervision for Linux-hosted runtime processes |
| launchd | Service supervision for the macOS-hosted runtime process |
| Tailscale | Private mesh network connecting the always-on host and satellite machines, so the runtime can operate across more than one physical machine without exposing services publicly |
| Git | Version control for the runtime codebase and the development workflow it runs inside |

Related: [architecture](./architecture.md) · [case studies](./README.md)
