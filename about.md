# About the engineer

Chandler Morone designed and built CEM888 from the ground up. Self-taught, no formal CS background — before software, she worked as a dressage trainer, a TIG welder/fabricator, and a self-taught builder of cars, motorcycles, and businesses. That background is where the engineering standard comes from: precise, unforgiving systems where "close enough" fails and you find out immediately when it does.

## How the system actually gets built

CEM888 is built using AI coding agents as engineering labor — they write code, run tests, execute migrations, and do a large share of the hands-on implementation work. That's not hidden or softened here, because it's part of the actual engineering story: **the system this organization documents is a system built for making AI agents reliable enough to trust with real engineering work, and it was built substantially by AI agents doing that work.** Using the product to build the product is the proof, not a shortcut around it.

What stays with Chandler, not delegated to the agents: system architecture and design decisions, debugging when something goes wrong, defining acceptance criteria for whether a change actually works, and every technical decision about what the runtime should and shouldn't do. The agents are workers; she's the engineer deciding what "correct" means and verifying that it's true — which is also why several of the [case studies](./README.md) in this repo are about catching the agents' own work being wrong (a telemetry bug, a stale context defect, a completion claim that didn't match reality) before it shipped.

## Where to look

- [Engineering case studies](./README.md) — real problems, root causes, fixes, measurements
- [Architecture](./architecture.md) — the conceptual system design
- [Benchmarks](https://github.com/CEM888AI/benchmarks) — reproducible memory-retrieval results
- [cem888.ai](https://cem888.ai) — the product
- creator@cem888.ai
