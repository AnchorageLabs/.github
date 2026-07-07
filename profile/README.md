<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="anchoragedark.svg">
  <source media="(prefers-color-scheme: light)" srcset="anchoragelight.svg">
  <img alt="Anchorage Labs" src="anchoragelight.svg" width="440">
</picture>

### The state layer of AI software engineering

A deterministic map of every repository, and a runtime that takes an issue to a merged, deployed change.

<p>
  <img alt="Protocol: Apache-2.0" src="https://img.shields.io/badge/protocol-Apache--2.0-2ee9ff?style=for-the-badge&labelColor=0c0e13">
  <img alt="Cartographer: private beta" src="https://img.shields.io/badge/Cartographer-private%20beta-4cc9ff?style=for-the-badge&labelColor=0c0e13">
  <img alt="Agents: MCP + A2A" src="https://img.shields.io/badge/agents-MCP%20%C2%B7%20A2A-a78bff?style=for-the-badge&labelColor=0c0e13">
  <img alt="Built on AWS" src="https://img.shields.io/badge/built%20on-AWS-ff9900?style=for-the-badge&labelColor=0c0e13">
</p>

</div>

Anchorage Labs builds infrastructure for software automation: coding agents that
plan, write, review, and ship code, and the structural context those agents
need to do it without re-discovering a repository from scratch every time.

## What we build

Two products, designed to be used together but useful independently.

### Anchorage — protocol, agents, and orchestration

An open-core stack for end-to-end software automation: a wire protocol for
task envelopes, a TypeScript SDK, a reference CLI runner, and reference
agents (issue reading, planning, coding, review, merge, deploy) — all
Apache-2.0. The proprietary orchestrator ("the mainframe") sits above that
protocol, sequencing agents into durable workflows: issue → code,
issue → merge, issue → deploy, and equivalents starting from a Notion task.

```mermaid
flowchart LR
    subgraph pub["anchorage (public, Apache-2.0)"]
        direction TB
        Protocol["Protocol spec"]
        SDK["TypeScript SDK"]
        Runner["CLI runner"]
        Agents["Reference agents\nplan · code · review · merge · deploy"]
    end

    subgraph priv["anchorage-orchestrator (private)"]
        direction TB
        Server["HTTP API"]
        Worker["Worker"]
    end

    Trigger["GitHub issue / User Instruction"] --> Server
    Server --> Worker
    Worker -- "invokes via protocol" --> Runner
    Runner --> Agents
    Agents --> Outcome["PR opened → CI watched →\nreviewed → merged → deployed"]
```

Anyone can build an agent against the public protocol; the orchestrator is
what runs it durably in production.

### Cartographer — symbolic repo context for agents

Cartographer scans a repository and persists a machine-queryable context
artifact — stack, commands, entry points, architecture, env vars — plus a
tree-sitter-backed symbol index (definitions, references, imports, routes,
tests) in a local SQLite cache. Agents query `cmd:test` or run
`cartographer impact <symbol>` instead of `grep`/`cat package.json`, with
zero LLM tokens and no network calls. The index can be pushed to a hosted
org graph so every agent working across a company's repos — Claude Code,
Codex, Cursor, or the Anchorage runtime itself — queries the same fresh
structure.

```mermaid
flowchart LR
    Repo[("Git repository")] -- "scan" --> Engine["cartographer engine"]
    Engine --> Context["repo-context.json\nsymbolic facts"]
    Engine --> Index[("SQLite symbol index\ntree-sitter, ~25 languages")]

    Context --> Coding["Coding agents\nClaude Code · Codex · Cursor · ..."]
    Index --> Coding

    Index -- "push" --> Graph[("Hosted org graph\nversioned, CAS-protected")]
    Graph -- "pull" --> Other["Every other repo / agent in the org"]
```

Cartographer is currently in **private beta — invite-only**. Invited members get a
one-command CLI install and a full developer setup guide:

```bash
# macOS · Linux · Git Bash
curl -fsSL https://api.anchoragelabs.dev/cli/install.sh | sh

# Windows PowerShell
irm https://api.anchoragelabs.dev/cli/install.ps1 | iex
```

**Works with your agents.** One token wires Cartographer's graph into the tools
your team already uses — over MCP, with a stdio bridge for anything else:

| Claude Code | Cursor | Codex | Gemini CLI | GitHub Copilot | Windsurf | Zed |
| :---------: | :----: | :---: | :--------: | :------------: | :------: | :-: |

### How they fit together

Cartographer's compact context digest is injected directly into Anchorage
task envelopes, so an orchestrated agent starts a task already knowing the
repo's shape instead of spending its first turns discovering it.

```mermaid
flowchart LR
    Graph[("Cartographer\nhosted org graph")] -- "context digest" --> Envelope["Anchorage task envelope"]
    Envelope --> Runner["anchorage-runner"]
    Runner --> Agent["Reference agent"]
```

### Under the hood

TypeScript · Node 22 · tree-sitter · SQLite · PostgreSQL · MCP / A2A ·
AWS (CDK · Bedrock · ECS Fargate · RDS · S3) — grounded in facts, provisioned as
code, and inspectable end to end.

## Principles

- Build useful systems, not demos.
- Prefer clear protocols and explicit contracts.
- Never invent facts: explicit declarations beat inference beat heuristics;
  undetectable means "unknown," not a guess.
- Keep architecture understandable and automation auditable.
- Design for reliability and determinism from the beginning.
- Use simple tools until complexity is justified.

## How we work

- GitHub-centered development, with agent-driven contribution docs
  (`AGENTS.md`) in every repo.
- Structured planning before implementation; every substantive PR cites the
  issue or ADR that motivates it.
- Clear boundary between open-core (protocol, SDK, reference agents,
  Cartographer engine) and proprietary infrastructure (the orchestrator,
  the hosted graph).
- Infrastructure as code where infrastructure exists.

## Founders

AnchorageLabs is a personal venture by Valentin Torassa and Sol Soletti.

## Contact

For now, AnchorageLabs is founder-led and privately operated.
