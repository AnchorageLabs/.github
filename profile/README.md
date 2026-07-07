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

<p>
  <img alt="Claude Code" src="https://img.shields.io/badge/Claude%20Code-0c0e13?style=for-the-badge&logo=claude&logoColor=D97757">
  <img alt="Cursor" src="https://img.shields.io/badge/Cursor-0c0e13?style=for-the-badge&logo=cursor&logoColor=white">
  <img alt="Codex" src="https://img.shields.io/badge/Codex-0c0e13?style=for-the-badge&logo=data%3Aimage%2Fsvg%2Bxml%3Bbase64%2CPHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0id2hpdGUiPjxwYXRoIGQ9Ik0yMi4yODE5IDkuODIxMWE1Ljk4NDcgNS45ODQ3IDAgMCAwLS41MTU3LTQuOTEwOCA2LjA0NjIgNi4wNDYyIDAgMCAwLTYuNTA5OC0yLjlBNi4wNjUxIDYuMDY1MSAwIDAgMCA0Ljk4MDcgNC4xODE4YTUuOTg0NyA1Ljk4NDcgMCAwIDAtMy45OTc3IDIuOSA2LjA0NjIgNi4wNDYyIDAgMCAwIC43NDI3IDcuMDk2NiA1Ljk4IDUuOTggMCAwIDAgLjUxMSA0LjkxMDcgNi4wNTEgNi4wNTEgMCAwIDAgNi41MTQ2IDIuOTAwMUE1Ljk4NDcgNS45ODQ3IDAgMCAwIDEzLjI1OTkgMjRhNi4wNTU3IDYuMDU1NyAwIDAgMCA1Ljc3MTgtNC4yMDU4IDUuOTg5NCA1Ljk4OTQgMCAwIDAgMy45OTc3LTIuOTAwMSA2LjA1NTcgNi4wNTU3IDAgMCAwLS43NDc1LTcuMDcyOXptLTkuMDIyIDEyLjYwODFhNC40NzU1IDQuNDc1NSAwIDAgMS0yLjg3NjQtMS4wNDA4bC4xNDE5LS4wODA0IDQuNzc4My0yLjc1ODJhLjc5NDguNzk0OCAwIDAgMCAuMzkyNy0uNjgxM3YtNi43MzY5bDIuMDIgMS4xNjg2YS4wNzEuMDcxIDAgMCAxIC4wMzguMDUydjUuNTgyNmE0LjUwNCA0LjUwNCAwIDAgMS00LjQ5NDUgNC40OTQ0em0tOS42NjA3LTQuMTI1NGE0LjQ3MDggNC40NzA4IDAgMCAxLS41MzQ2LTMuMDEzN2wuMTQyLjA4NTIgNC43ODMgMi43NTgyYS43NzEyLjc3MTIgMCAwIDAgLjc4MDYgMGw1Ljg0MjgtMy4zNjg1djIuMzMyNGEuMDgwNC4wODA0IDAgMCAxLS4wMzMyLjA2MTVMOS43NCAxOS45NTAyYTQuNDk5MiA0LjQ5OTIgMCAwIDEtNi4xNDA4LTEuNjQ2NHpNMi4zNDA4IDcuODk1NmE0LjQ4NSA0LjQ4NSAwIDAgMSAyLjM2NTUtMS45NzI4VjExLjZhLjc2NjQuNzY2NCAwIDAgMCAuMzg3OS42NzY1bDUuODE0NCAzLjM1NDMtMi4wMjAxIDEuMTY4NWEuMDc1Ny4wNzU3IDAgMCAxLS4wNzEgMGwtNC44MzAzLTIuNzg2NUE0LjUwNCA0LjUwNCAwIDAgMSAyLjM0MDggNy44NzJ6bTE2LjU5NjMgMy44NTU4TDEzLjEwMzggOC4zNjQgMTUuMTE5MiA3LjJhLjA3NTcuMDc1NyAwIDAgMSAuMDcxIDBsNC44MzAzIDIuNzkxM2E0LjQ5NDQgNC40OTQ0IDAgMCAxLS42NzY1IDguMTA0MnYtNS42NzcyYS43OS43OSAwIDAgMC0uNDA3LS42Njd6bTIuMDEwNy0zLjAyMzFsLS4xNDItLjA4NTItNC43NzM1LTIuNzgxOGEuNzc1OS43NzU5IDAgMCAwLS43ODU0IDBMOS40MDkgOS4yMjk3VjYuODk3NGEuMDY2Mi4wNjYyIDAgMCAxIC4wMjg0LS4wNjE1bDQuODMwMy0yLjc4NjZhNC40OTkyIDQuNDk5MiAwIDAgMSA2LjY4MDIgNC42NnpNOC4zMDY1IDEyLjg2M2wtMi4wMi0xLjE2MzhhLjA4MDQuMDgwNCAwIDAgMS0uMDM4LS4wNTY3VjYuMDc0MmE0LjQ5OTIgNC40OTkyIDAgMCAxIDcuMzc1Ny0zLjQ1MzdsLS4xNDIuMDgwNUw4LjcwNCA1LjQ1OWEuNzk0OC43OTQ4IDAgMCAwLS4zOTI3LjY4MTN6bTEuMDk3Ni0yLjM2NTRsMi42MDItMS40OTk4IDIuNjA2OSAxLjQ5OTh2Mi45OTk0bC0yLjU5NzQgMS40OTk3LTIuNjA2Ny0xLjQ5OTd6Ii8%2BPC9zdmc%2B">
  <img alt="Gemini CLI" src="https://img.shields.io/badge/Gemini%20CLI-0c0e13?style=for-the-badge&logo=googlegemini&logoColor=8E75B2">
  <img alt="GitHub Copilot" src="https://img.shields.io/badge/GitHub%20Copilot-0c0e13?style=for-the-badge&logo=githubcopilot&logoColor=white">
  <img alt="Windsurf" src="https://img.shields.io/badge/Windsurf-0c0e13?style=for-the-badge&logo=windsurf&logoColor=58C6A6">
  <img alt="Zed" src="https://img.shields.io/badge/Zed-0c0e13?style=for-the-badge&logo=zedindustries&logoColor=white">
</p>

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
