# Charles Adedotun

Systems & infrastructure engineer building networking daemons, Linux media/audio tools, Kubernetes platforms, and developer CLI tooling in **Rust**, **Go**, and **Python**.

📫 [charles.adedotun8@gmail.com](mailto:charles.adedotun8@gmail.com) · 🏢 [@NeuBlink](https://github.com/NeuBlink)

---

### What I've Been Building

- **[airlock](https://github.com/charles-adedotun/airlock)** — Live stream proxy and rolling DVR buffer for media servers (Plex, Jellyfin). Shares a single upstream connection across multiple viewers for instant channel switching while keeping memory usage flat (`< 32 MB` RSS) on multi-gigabyte streams.
- **[sonus-rs](https://github.com/charles-adedotun/sonus-rs)** — Streams Linux desktop audio (PipeWire) directly to Apple HomePods over AirPlay 2. Cuts AirPlay's default 2-second delay down to `250ms` and syncs video playback in browsers and media players so audio doesn't lag behind video.
- **[tail-gate](https://github.com/charles-adedotun/tail-gate)** — Lightweight reverse proxy that routes Tailscale traffic straight to local Unix sockets, replacing a heavier Caddy + TCP relay setup at `< 6 MB` RAM.
- **[reference-platform](https://github.com/charles-adedotun/reference-platform)** & **[kubepulse](https://github.com/charles-adedotun/kubepulse)** — Multi-cloud Kubernetes setup (AWS EKS & GCP GKE with Terraform, ArgoCD, Kyverno, and OpenTelemetry) paired with a Go CLI and live dashboard for cluster health checks and SLO tracking.
- **[stitch-rs](https://github.com/charles-adedotun/stitch-rs)** — Rust SDK, CLI, and MCP client for generating UI screens from text prompts, with automatic JSON Schema cleanup for LLM tool-calling APIs.

### Developer Tools & Open Source

- **[claude-code-sub-agents](https://github.com/charles-adedotun/claude-code-sub-agents)** *(★ 33)* — CLI installer for modular sub-agent workflows in Claude Code workspaces.
- **[notifications-mcp-server](https://github.com/charles-adedotun/notifications-mcp-server)** *(★ 16)* — macOS notification server for Model Context Protocol (MCP) clients that plays a sound or shows a banner when long-running tasks finish.
- **[agent-sandbox-evals](https://github.com/charles-adedotun/agent-sandbox-evals)** — Test suite measuring what untrusted AI agent code can access across raw hosts, pre-execution filters, and locked-down containers.
- **[auto-pr](https://github.com/charles-adedotun/auto-pr)** — Go CLI that inspects local Git diffs and commit history to draft pull requests and GitLab merge requests.
- **[modelcontextprotocol/python-sdk#2573](https://github.com/modelcontextprotocol/python-sdk/pull/2573)** — Upstream fix for JSON-RPC field collisions during MCP message forwarding.

### Background

4+ years running production Kubernetes platforms (AWS EKS, GCP GKE), Terraform infrastructure, observability stacks (Prometheus, Loki, Grafana, OpenTelemetry), and CI/CD pipelines.
