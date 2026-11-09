# Charles Adedotun

Infrastructure & systems engineer building **AI-agent-powered workflows**, developer tooling, and self-hosted platforms in **Rust**, **Go**, and **Python**.

📫 [charles.adedotun8@gmail.com](mailto:charles.adedotun8@gmail.com)

---

### AI Agent Workflows & Automation

Most of my day-to-day work lives in a private, self-hosted operations platform (4,900+ contributions over the past year) that runs autonomous agent workflows end-to-end:

- **Scheduled & Event-Driven Agent Pipelines**: Orchestrating multi-step workflows (Windmill, Postgres, Redis) where LLM agents gather data, run quantitative or codebase analysis, and prepare structured proposals on a schedule.
- **Human-in-the-Loop Execution**: Routing agent proposals through Telegram and web approval gates so high-impact actions (deployments, external API writes, browser automation) require one-tap operator sign-off before running.
- **Model Context Protocol (MCP) & Multi-Model Tooling**: Building custom MCP servers, schema adapters, and sub-agent harnesses that connect Claude and Gemini to Git, Kubernetes, browser sessions, and internal services.

### Open-Source Projects (Extracted from the Platform)

**AI Agent Tooling & Sandboxing**
- **[claude-code-sub-agents](https://github.com/charles-adedotun/claude-code-sub-agents)** *(★ 33)* — Bootstrapper that analyzes a repository and generates project-specific sub-agent workflows inside `.claude/agents/`.
- **[stitch-rs](https://github.com/charles-adedotun/stitch-rs)** — Rust SDK, CLI, and MCP client that lets agents generate and iterate on UI screens from text prompts, with automatic JSON Schema cleanup for Gemini, OpenAI, and Claude tool calling.
- **[notifications-mcp-server](https://github.com/charles-adedotun/notifications-mcp-server)** *(★ 16)* — macOS MCP server that triggers sound and banner alerts when long-running background agent tasks finish.
- **[agent-sandbox-evals](https://github.com/charles-adedotun/agent-sandbox-evals)** — Test suite measuring what untrusted agent code can access across raw hosts, command filters, and locked-down containers.
- **[auto-pr](https://github.com/charles-adedotun/auto-pr)** & **[sre-plugin](https://github.com/charles-adedotun/sre-plugin)** — Agent-assisted CLI for drafting pull requests from local Git context, plus Claude Code SRE skills for Kubernetes, Prometheus, Loki, and Grafana incident triage.

**Systems, Networking & Kubernetes Infrastructure**
- **[airlock](https://github.com/charles-adedotun/airlock)** — Live stream proxy and rolling DVR buffer for media servers (Plex, Jellyfin) that shares a single upstream connection across viewers while keeping RAM usage flat (`< 32 MB`).
- **[sonus-rs](https://github.com/charles-adedotun/sonus-rs)** — Streams Linux desktop audio (PipeWire) to Apple HomePods over AirPlay 2 with `250ms` latency and automatic video lip-sync.
- **[tail-gate](https://github.com/charles-adedotun/tail-gate)** — Lightweight reverse proxy (`< 6 MB` RAM) that routes Tailscale traffic straight to local Unix sockets for internal platform services.
- **[reference-platform](https://github.com/charles-adedotun/reference-platform)** & **[kubepulse](https://github.com/charles-adedotun/kubepulse)** — Multi-cloud Kubernetes setup (AWS EKS & GCP GKE with Terraform, ArgoCD, Kyverno, and OpenTelemetry) paired with a Go CLI and live dashboard for cluster health checks and SLO tracking.

### Background

4+ years running production Kubernetes platforms (AWS EKS, GCP GKE), Terraform infrastructure, observability stacks (Prometheus, Loki, Grafana, OpenTelemetry), and CI/CD pipelines.
