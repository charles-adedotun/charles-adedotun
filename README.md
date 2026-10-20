# Charles Adedotun

**Safe Rust Systems & AI Execution Runtime Engineer**  
Lagos, Nigeria (UTC+1 • Full US-East & EU synchronous overlap)  
`charles.adedotun8@gmail.com` • `charles@krysp.org` • [GitHub (@charles-adedotun)](https://github.com/charles-adedotun)

---

I build low-latency systems infrastructure and secure execution runtimes for autonomous AI agents. My work combines kernel-level Linux resource control, memory-safe systems programming in Safe Rust (`#![forbid(unsafe_code)]`), and multi-tenant Kubernetes platform reliability.

### Core Systems Engineering Principles
- **Kernel-Enforced Boundaries Over Userland Filters**: Language-level string filtering cannot substitute for OS primitives; sandboxes must enforce cgroups v2, network namespaces, read-only rootfilesystems, and dropped capabilities (`--cap-drop=ALL`).
- **Deterministic Memory Footprints**: Zero-copy ring buffers and explicit page-cache management (`posix_fadvise`) prevent host memory thrashing and OS buffer bloat under sustained multi-gigabyte throughput.
- **Fail-Closed Security by Construction**: Network boundaries intercept SSRF pivots at socket dial time; secrets architectures encrypt at rest via SOPS + age with strictly in-memory runtime injection.

---

## Systems Architecture & Production Runtimes

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│ PRODUCTION SYSTEMS CAPABILITY MATRIX                                                   │
├──────────────────────────┬─────────────────────────────────────────────────────────────┤
│ Systems Language         │ Safe Rust (#![forbid(unsafe_code)], zero unsafe blocks)      │
│ Linux Kernel Primitives  │ posix_fadvise (page-cache drops), cgroups v2, namespaces   │
│ Agent Sandbox Security   │ OCI isolation, read-only rootfs, non-root UID 10001, no-new-priv │
│ Network Boundary Defenses│ SafeDnsResolver (socket dial-time SSRF & DNS-rebinding guard)│
│ Secrets Architecture     │ GitOps SOPS + age, zero-daemon in-memory injection           │
│ Multi-Tenant Kubernetes  │ 99.9% uptime, 25% cost cut (HPA), 60% MTTD cut (Prometheus/Loki)│
└──────────────────────────┴─────────────────────────────────────────────────────────────┘
```

---

## Flagship Systems & Tools

### 1. [`airlock`](https://github.com/charles-adedotun/airlock) — Safe Rust Streaming Proxy & Kernel Page-Cache Shield
*High-throughput read-ahead media streaming engine, rolling DVR jitter shield, and single-upstream arbiter.*

- **Zero-Unsafe Rust**: Built under `#![forbid(unsafe_code)]` using Tokio, Axum, and zero-copy byte buffers.
- **Linux Page-Cache Ejection (`posix_fadvise`)**: Under high-throughput streaming writes, sequential segments rapidly pollute the host OS page cache, causing memory thrashing and I/O stalls. `airlock` executes `posix_fadvise(fd, 0, bytes_written, PosixFadviseAdvice::POSIX_FADV_DONTNEED)` on every segment flush, maintaining process RSS strictly under **< 32 MB** across 100+ GB rolling data windows.
- **Dial-Time SSRF Neutralization (`SafeDnsResolver`)**: Implements `reqwest::dns::Resolve` to inspect resolved IP addresses directly at socket dial time—blocking RFC 1918 subnets, cloud metadata (`169.254.169.254`), Tailscale CGNAT (`100.64.0.0/10`), and IPv6-mapped IPv4 addresses before TCP handshake initiation.
- **Type-Safe Credential Scrubber (`RedactedUrl`)**: Formally scrubs HTTP credentials and path tokens at the type system level across `Display`, `Debug`, and OpenTelemetry/Prometheus tracing pipelines.

```bash
# Verify airlock source safety and compile
cargo check --manifest-path airlock/Cargo.toml
cargo test --manifest-path airlock/Cargo.toml
```

---

### 2. [`agent-sandbox-evals`](https://github.com/charles-adedotun/agent-sandbox-evals) — Empirical Agent Containment Benchmark
*Adversarial evaluation suite testing 22 attack vectors across agent execution tiers.*

Evaluates the containment boundaries of LLM tool execution across three architectural paradigms:
1. **Layer 0 (Raw Host)**: Unrestricted subprocess execution (typical agent defaults).
2. **Layer 1 (Sanitized MCP Harness)**: Pre-execution AST verification, regex filtering, and path traversal guards.
3. **Layer 2 (Hardened OCI Container)**: Non-root execution (UID 10001), `--read-only` rootfs, `--cap-drop=ALL`, `--security-opt=no-new-privileges`, and cgroups v2 resource caps.

```
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│ ADVERSARIAL CONTAINMENT BENCHMARK LEDGER                                                         │
├──────────────────────────┬──────────────┬───────────────┬──────────────────┬─────────────────────┤
│ Isolation Tier           │ Vectors Tested│ Intercept (ms)│ Containment Rate │ Failure Modes       │
├──────────────────────────┼──────────────┼───────────────┼──────────────────┼─────────────────────┤
│ Layer 0: Raw Host        │ 22           │ N/A           │ 22.7%            │ SSRF, Traversal, OOM│
│ Layer 1: Sanitized MCP   │ 22           │ 0.10 ms       │ 90.9%            │ RAM Bomb / Disk Full│
│ Layer 2: Hardened OCI    │ 22           │ N/A           │ 100.0%           │ Fully Contained     │
└──────────────────────────┴──────────────┴───────────────┴──────────────────┴─────────────────────┘
```
*Empirical finding: Userland filtering blocks 100% of network pivot and path traversal vectors with sub-millisecond overhead (0.10 ms), but kernel-enforced cgroups are strictly mandatory to prevent host resource starvation (RAM/disk bombs).*

```bash
# Run the 22-vector automated security evaluation matrix
pytest agent-sandbox-evals/tests/ -v
```

---

### 3. [`metaclaw`](https://github.com/charles-adedotun/metaclaw) — Containerized Agent Sandbox Architecture Spike
*Systems architecture prototype evaluating isolated agent runtime execution and zero-network IPC.*

- **Container Isolation Boundary**: Routes messaging-channel agent sessions into dedicated per-group OCI containers with non-root UID 10001, immutable read-only code mounts, and strict volume allowlists (`mount-allowlist.json`).
- **Filesystem-Based IPC**: Eliminates internal network listeners by establishing an asynchronous JSON message-passing protocol over mounted directories (`/workspace/ipc/messages/`), ensuring agent subprocesses have zero lateral network access to the host orchestrator.
- **Resource Boundary Defense**: Enforces CPU and memory limits per agent container, protecting multi-tenant hosts from runaway execution.

```bash
# Verify container isolation boundary
docker run --rm --entrypoint sh metaclaw-agent:latest -lc 'id -un && pwd'
docker run --rm --entrypoint sh -v "$PWD:/workspace/project:ro" metaclaw-agent:latest \
  -lc 'touch /workspace/project/.metaclaw-write-test 2>/dev/null || echo project-read-only'
```

---

### 4. [`sops-gate`](https://github.com/charles-adedotun/sops-gate) — Safe Rust GitOps Secrets Pre-Commit Audit Gate
*Parallelized, sub-second pre-commit audit gate and cryptographic recipient validator for SOPS+age GitOps repositories.*

- **Zero-Daemon Secrets Model**: Replaces heavyweight secret manager daemons with GitOps-native `SOPS` + `age` encryption at rest (`secrets.enc.json`) and in-memory injection (`sops exec-env`).
- **High-Throughput AST & Regex Traversal**: Scans multi-repository estates in < 150 ms using `rayon` thread pools, verifying that all secrets match target age recipient public keys before staging.
- **Fail-Closed Pre-Commit Engine**: Intercepts unencrypted plaintext `.env` files, rogue AWS/GCP access keys, and malformed SOPS metadata before commits touch the Git DAG.

```bash
# Verify sops-gate recipient auditing
cargo run --manifest-path sops-gate/Cargo.toml -- --check-recipients --strict
```

---

### 5. [`notifications-mcp-server`](https://github.com/charles-adedotun/notifications-mcp-server) — Model Context Protocol Utility
*Native notification MCP server for background agent execution runtimes (16 GitHub stars).*

- Exposes OS-level async notifications and acoustic alerts to AI developer environments (Claude Desktop, Claude Code).
- Provides deterministic completion signals for long-running compile, test, and infrastructure automation tasks without polling.

---

### 6. [`claude-code-sub-agents`](https://github.com/charles-adedotun/claude-code-sub-agents) — Modular Sub-Agent Bootstrapper
*Deterministic configuration framework for project-specific sub-agents (33 GitHub stars).*

- Installs lightweight agent-architect primitives into developer workspaces to orchestrate specialized testing, auditing, and platform workflows.
- Eliminates configuration bloat by maintaining isolated, auditable markdown skill boundaries.

---

## Multi-Tenant Cloud Platform & Reliability

Beyond systems code, I operate 4+ years of production multi-tenant Kubernetes and cloud infrastructure:

- **99.9% Service Reliability**: Managed production Kubernetes clusters across AWS (EKS) and GCP (GKE), configuring ingress, service meshes, network security policies, and zero-downtime rolling updates.
- **Cloud Financial Engineering**: Reduced cloud infrastructure spend by **25%** through Horizontal Pod Autoscaler (HPA) tuning, pod resource requests/limits right-sizing, and automated spot/preemptible node instance scheduling.
- **Telemetry & MTTR Optimization**: Built unified observability pipelines (Prometheus, Loki, Alertmanager, Grafana), reducing Mean Time to Detection (MTTD) by **60%** and Mean Time to Remediation (MTTR) by **45%**.
- **Automated GitOps Pipelines**: Engineered automated release pipelines via ArgoCD and GitHub Actions, reducing deployment lead time by **40%** with automated health checks and fail-forward remediation.

---

## Technical Stack & Systems Primitives

```
Systems Programming    : Safe Rust (#![forbid(unsafe_code)], Tokio, Axum, Rayon, Nix)
Linux Systems & Kernel : posix_fadvise, cgroups v2, namespaces, UID/GID mapping, IPC
Container Security     : Docker, OCI Hardening, Cap-Drop, Read-Only Rootfs, Volume Allowlists
Agent Runtimes         : Model Context Protocol (MCP), Claude Code, Filesystem IPC
Cloud & Orchestration  : Kubernetes (EKS, GKE, KinD), Docker Compose, Caddy
Infrastructure as Code : Terraform, Helm, ArgoCD, GitHub Actions
Observability & Ops    : Prometheus, Loki, Grafana, Alertmanager, OpenTelemetry
Secrets Management     : SOPS, age, fail-closed in-memory injection
```

---

## Contact & Working Engagements

- **Email**: `charles.adedotun8@gmail.com` | `charles@krysp.org`
- **Location**: Lagos, Nigeria (UTC+1 • Synchronous overlap with US-East and European working hours)
- **Contract Engagement**: Independent B2B Contractor / Employer of Record (via Deel or Remote.com)
