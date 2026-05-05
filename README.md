# Charles Adedotun

AI Platform / DevTools engineer building containerized agent systems, MCP servers, and Kubernetes observability tooling.

I work at the boundary between infrastructure and AI-assisted operations: isolated agent runtimes, developer workflows, cluster-aware SRE tools, and small systems that are easy to inspect and operate.

## Current Focus

- Containerized AI agents with explicit filesystem, process, and credential boundaries
- MCP servers that connect AI tools to operating-system and developer workflows
- Kubernetes observability tools for discovery, dashboards, alert tuning, and incident investigation
- CLI and automation tooling for Docker, homelab, and platform engineering workflows

## Selected Work

| Project | Area | Notes |
|---------|------|-------|
| [MetaClaw](https://github.com/charles-adedotun/metaclaw) | Containerized agents | Node.js orchestrator for Claude agents running in isolated containers, with messaging channels, SQLite state, IPC, scheduled tasks, and mount validation |
| [KubePulse](https://github.com/charles-adedotun/kubepulse) | Kubernetes monitoring | Go and React Kubernetes health monitor with cluster checks, alerting/SLO modules, AI diagnostics, Docker Compose, tests, and CI |
| [sre-plugin](https://github.com/charles-adedotun/sre-plugin) | Kubernetes observability | Claude Code SRE tooling for cluster discovery, service dashboards, Prometheus/Loki/Grafana context, and alert investigation |
| [notifications-mcp-server](https://github.com/charles-adedotun/notifications-mcp-server) | MCP | MCP server that exposes native macOS notifications to Claude Desktop and related agent workflows |
| [auto-pr](https://github.com/charles-adedotun/auto-pr) | DevTools CLI | Go CLI that uses Claude Code plus `gh`/`glab` to generate pull request and merge request text from local git context |
| [claude-code-sub-agents](https://github.com/charles-adedotun/claude-code-sub-agents) | Claude Code workflow | Lightweight installer for an `agent-architect` that creates project-specific Claude Code agents |

## Technical Areas

**AI platform and agent systems**

- Claude Code, Claude Agent SDK, Model Context Protocol
- Agent orchestration, per-session state, tool routing, IPC, and scheduled execution
- Container isolation for agent workloads using Docker and Apple Container

**Platform engineering**

- Kubernetes, Docker, Compose, Helm, Terraform
- CI/CD with GitHub Actions and related automation
- Linux/macOS service management, local infrastructure, and secure remote access

**Observability and reliability**

- Prometheus, Grafana, Loki, Alertmanager
- Cluster discovery, SLOs, dashboards, alert review, and incident workflows
- SQLite-backed state, recovery paths, retries, and operational debugging

## How I Build

- Keep the core system small enough to audit
- Prefer explicit trust boundaries over broad permissions
- Use automation to remove repetitive operational work
- Document architecture, security assumptions, and known limitations

## Contact

Email: charles.adedotun8@gmail.com
