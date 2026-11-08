# Charles Adedotun

**Senior Systems, Kubernetes Platform & AI Runtime Infrastructure Engineer**  
Remote (US Central Overlap • UTC+1 / CT)  
📫 [charles.adedotun8@gmail.com](mailto:charles.adedotun8@gmail.com) • 🏢 [@NeuBlink](https://github.com/NeuBlink)

---

I design and operate low-latency systems software in **Safe Rust (`#![forbid(unsafe_code)]`)** and **Go**, alongside multi-tenant **Kubernetes (EKS/GKE)** platforms and **Model Context Protocol (MCP)** developer infrastructure. Across 4+ years of production platform engineering and 15,000+ lines of memory-safe Rust daemons, my work focuses on deterministic memory bounds, wire-protocol correctness, and fail-closed network security.

---

## Production Systems & Platform Portfolio

| Repository | Domain & Stack | Core Engineering Problem Solved | Key Measured / Verified Invariant |
| :--- | :--- | :--- | :--- |
| **[`airlock`](https://github.com/charles-adedotun/airlock)** | **Rust** (`#![forbid(unsafe_code)]`, Tokio, Axum, Nix) | High-throughput media streaming proxies thrash Linux page cache and expose internal networks to DNS-rebinding SSRF. | Evicts flushed segments via `posix_fadvise(..., POSIX_FADV_DONTNEED)` to hold RSS **< 32 MB** across 100+ GB streams; blocks RFC 1918/CGNAT/metadata IPs at socket dial time (`SafeDnsResolver`). |
| **[`sonus-rs`](https://github.com/charles-adedotun/sonus-rs)** | **Rust** (`#![forbid(unsafe_code)]`, PipeWire, ChaCha20-Poly1305, RTP/RTSP) | Linux-to-HomePod AirPlay bridges suffer 2,000ms buffering delays, 5-bit software volume dynamic range loss, and 55+ MB Python daemon bloat. | Clamps RAOP presentation latency to **250.0ms** (`11,025` frames) with PipeWire `ProcessLatency` A/V lip-sync injection, 2nd-order `SpaDLL` clock recovery, hardware S5 DSP volume control, and **< 8 MB RSS**. |
| **[`tail-gate`](https://github.com/charles-adedotun/tail-gate)** | **Rust** (`#![forbid(unsafe_code)]`, Hyper, Tokio UDS, Tailscale LocalAPI) | Reverse-proxying Tailscale traffic through Go/Caddy + Python TCP-to-Unix-socket relays consumed 50.9 MB RSS and traversed two extra loopback TCP stacks. | Connects directly to `tokio::net::UnixStream` sockets at **< 6 MB RSS** (88% reduction), enforces **RFC 9110 §15.5.22 HTTP 421** on unmapped hosts, scrubs 11 identity headers, and gates routes via `/localapi/v0/whois`. |
| **[`reference-platform`](https://github.com/charles-adedotun/reference-platform)** | **Terraform HCL, Go, Helm, ArgoCD, Kyverno, OTel** | Multi-cloud Kubernetes bootstrap often lacks deterministic ordering between security policies, CRDs, observability, and workloads. | 4.2k LOC AWS EKS + GCP GKE Terraform modules, 9-wave ArgoCD GitOps bootstrap (Kyverno Wave `-5` → OTel Wave `2` → Go `TaskFlow` service Wave `10`), and trace-correlated structured logging. |
| **[`stitch-rs`](https://github.com/charles-adedotun/stitch-rs)** | **Rust** (`#![forbid(unsafe_code)]`, JSON-RPC 2.0, MCP, CLI) | Upstream MCP tool schemas with nested `$defs`/`$ref` pointers fail strict LLM function-calling validators (e.g., Gemini `functionDeclarations`). | Recursive JSON Schema AST repair engine that injects missing `$defs`, flattens `$ref` trees for Gemini/OpenAI/Claude/MCP adapters, transforms FIFE image URLs, and redacts credentials in `Debug` impls. |
| **[`kubepulse`](https://github.com/charles-adedotun/kubepulse)** | **Go** (`k8s.io/client-go`, Gorilla WebSockets, React) | Alert fatigue during cluster incidents without unified SLO burn-rate tracking and circuit-broken triage. | Concurrent pod/node/service health engine (`pkg/health`), sliding-window SLO burn-rate tracker (`pkg/slo`), circuit-broken AI root-cause analyzer (`pkg/ai/circuit_breaker.go`), and live WebSocket topology feed. |

---

## Flagship Systems Deep-Dives

### 1. [`airlock`](https://github.com/charles-adedotun/airlock) — Safe Rust Streaming Proxy & Kernel Page-Cache Shield
*High-throughput read-ahead media streaming engine, rolling DVR ring buffer, and single-upstream arbiter (`5,500+ LOC`, `#![forbid(unsafe_code)]`).*

- **Linux Page-Cache Ejection (`posix_fadvise`)**: Sequential multi-gigabyte media writes rapidly fill the Linux page cache and trigger `kswapd` stalls. `airlock` issues `posix_fadvise(fd, 0, bytes_written, PosixFadviseAdvice::POSIX_FADV_DONTNEED)` after every segment flush (`src/storage/segment.rs`), keeping resident memory under **< 32 MB** across 100+ GB rolling windows.
- **Dial-Time SSRF Neutralization (`SafeDnsResolver`)**: Implements `reqwest::dns::Resolve` (`src/security/ssrf.rs`) to validate every DNS-resolved IP address prior to TCP handshake initiation—blocking loopback, RFC 1918 private subnets, cloud link-local metadata (`169.254.169.254`), Tailscale CGNAT (`100.64.0.0/10`), and IPv6-mapped IPv4 addresses.
- **Type-Level Credential Redaction (`RedactedUrl`)**: Wraps upstream URLs so userinfo, query tokens, and sensitive path segments are scrubbed at the type boundary across `Display`, `Debug`, and Prometheus/OpenTelemetry spans.

```bash
git clone https://github.com/charles-adedotun/airlock.git && cd airlock
cargo test --all-targets
cargo clippy --all-targets -- -D warnings
```

---

### 2. [`sonus-rs`](https://github.com/charles-adedotun/sonus-rs) — Sub-250ms PipeWire to AirPlay 2 / RAOP Audio & DSP Engine
*Real-time Linux audio engine and bidirectional DACP touch daemon written in Safe Rust (`6,800+ LOC`, `#![forbid(unsafe_code)]`).*

- **HAP Transient SRP-6a & ChaCha20-Poly1305 Framing**: Implements ephemeral Curve25519 ECDH + HKDF-SHA512 key derivation (`pair-verify`), 1,024-byte chunked `HapSession` RTSP control framing, and **1,444-byte MTU-safe RTP audio packets** (`12B` RTP header + `1,408B` encrypted 352-sample stereo PCM + `16B` Poly1305 tag + `8B` LE nonce) that never fragment over 1,500-byte Ethernet/Wi-Fi MTUs.
- **250ms Clamped Latency & PipeWire `ProcessLatency` Lip-Sync**: Overrides AirPlay's default 88,200-frame (2,000ms) buffer down to `latencyMin = latencyMax = 11025` (250.0ms) during RTSP `SETUP` and exports `ProcessLatency = 250,000,000 ns` on the PipeWire loopback node so Chromium, Firefox, and MPV automatically delay video rendering for frame-accurate lip-sync.
- **2nd-Order `SpaDLL` Clock Recovery & Hardware S5 Volume**: Compensates crystal oscillator drift between the Linux host and Apple HomePod via a proportional-integral loop filter ($K_p = 0.05, K_i = 0.001$) while streaming bit-perfect `0 dBFS` PCM and driving the HomePod's hardware S5 amplifier via RTSP `SET_PARAMETER volume: <dBFS>`.

```bash
git clone https://github.com/charles-adedotun/sonus-rs.git && cd sonus-rs
cargo test --all-targets
```

---

### 3. [`tail-gate`](https://github.com/charles-adedotun/tail-gate) — RFC 9110 HTTP 421 Fail-Closed UDS & Tailscale Gateway
*Zero-trust edge ingress proxy and native Unix Domain Socket router in Safe Rust (`1,000+ LOC`, `#![forbid(unsafe_code)]`).*

- **Direct `UnixStream` Routing (< 6 MB RSS)**: Replaces a 50.9 MB Caddy container + Python TCP relay stack with direct `tokio::net::UnixStream` upstream dispatch, cutting resident memory by **88%** and eliminating two loopback TCP hops per request.
- **RFC 9110 §15.5.22 HTTP 421 Misdirected Request Engine**: Rejects TLS connection-coalesced requests for unmapped `Host` authorities with `421 Misdirected Request` (`connection: close`) rather than falling back to a default vhost.
- **11-Header Identity Scrubber & Tailscale `/localapi/v0/whois` Guard**: Strips 11 client-spoofable identity headers (`Forwarded`, `X-Auth-Request-*`, `X-Forwarded-*`, `X-Platform-Auth`, `X-Internal-Auth-Token`) before route evaluation and verifies peer node identity/ACL tags against `/var/run/tailscale/tailscaled.sock`.

```bash
git clone https://github.com/charles-adedotun/tail-gate.git && cd tail-gate
cargo test --all-targets
```

---

### 4. [`reference-platform`](https://github.com/charles-adedotun/reference-platform) & [`kubepulse`](https://github.com/charles-adedotun/kubepulse) — Cloud-Native Kubernetes Platform & Observability
*End-to-end AWS EKS / GCP GKE Terraform modules, ArgoCD sync-wave GitOps, Kyverno policy enforcement, and Go cluster diagnostics.*

- **Multi-Cloud IaC & Sync-Wave GitOps (`reference-platform`)**: 4,200+ LOC of modular Terraform (VPC, EKS/GKE managed node groups, keyless GitHub OIDC IAM, S3/GCS remote state) paired with an ArgoCD app-of-apps bootstrap ordered across waves `-5` (Kyverno baseline policies) through `0..2` (`kube-prometheus-stack`, `loki-stack`, `opentelemetry-collector`) to `10` (`TaskFlow` Go REST/worker/cron microservice).
- **Concurrent K8s Health & SLO Engine (`kubepulse`)**: Built on `k8s.io/client-go` to execute concurrent pod, node, and service diagnostics, compute error-budget burn rates, and stream live cluster health topologies over Gorilla WebSockets.

```bash
git clone https://github.com/charles-adedotun/reference-platform.git && cd reference-platform
make test
```

---

### 5. [`stitch-rs`](https://github.com/charles-adedotun/stitch-rs) — Safe Rust MCP Client & Recursive Schema Compiler
*Rust SDK and CLI (`stitch-cli`) for autonomous UI generation over JSON-RPC 2.0 / MCP (`2,000+ LOC`, `#![forbid(unsafe_code)]`).*

- **Recursive JSON Schema Repair (`src/schema_repair.rs`)**: Automatically injects missing `$defs` (`ScreenInstance`, `SelectedScreenInstance`, `File`, `VariantOptions`) and recursively flattens `$ref` pointers so tool definitions pass strict Google Gemini `functionDeclarations`, OpenAI/Claude `tools`, and MCP `inputSchema` validators without runtime schema rejection.
- **Typed Error Taxonomy & Redacted Dual Auth (`src/auth.rs`, `src/errors.rs`)**: Supports `X-Goog-Api-Key` and OAuth Bearer + `X-Goog-User-Project` authentication with custom `Debug` implementations that redact secrets, plus typed retry classification (`RateLimited`, `AuthFailed`, `PermissionDenied`, `NotFound`).

```bash
git clone https://github.com/charles-adedotun/stitch-rs.git && cd stitch-rs
cargo test --all-targets
```

---

## Open-Source Developer Tooling & Community Contributions

| Project / Contribution | Adoption / Status | What It Does |
| :--- | :---: | :--- |
| **[`claude-code-sub-agents`](https://github.com/charles-adedotun/claude-code-sub-agents)** | **★ 33 Stars** | Lightweight sub-agent bootstrapper and modular `.claude/agents/` skill installer for Claude Code workspaces; preserves existing project `CLAUDE.md` context while registering specialized workflow agents. |
| **[`notifications-mcp-server`](https://github.com/charles-adedotun/notifications-mcp-server)** | **★ 16 Stars** | macOS Model Context Protocol (`FastMCP`) server delivering non-blocking acoustic (`afplay`) and visual (`terminal-notifier` / `osascript`) completion signals for long-running agent tasks (`uvx --from git+https://github.com/charles-adedotun/notifications-mcp-server claude-notifications`). |
| **[`modelcontextprotocol/python-sdk#2573`](https://github.com/modelcontextprotocol/python-sdk/pull/2573)** | **Upstream PR** | Regression-tested fix (`+59/-0`) preventing JSON-RPC envelope field collisions during MCP message forwarding. |
| **[`agent-sandbox-evals`](https://github.com/charles-adedotun/agent-sandbox-evals)** | **22-Vector Suite** | Adversarial containment benchmark evaluating SSRF, path traversal, privilege escalation, and resource exhaustion across raw host (`Layer 0`), pre-execution filter (`Layer 1`), and hardened `--cap-drop=ALL` `--read-only` OCI container (`Layer 2`) harnesses. |
| **[`auto-pr`](https://github.com/charles-adedotun/auto-pr)** | **Go CLI** | CLI tool that inspects local Git diffs, commit history, and repository label schemas to draft and publish structured pull requests and GitLab merge requests. |

---

## Production Reliability Track Record

- **Multi-Tenant Kubernetes Reliability (99.9% Uptime)**: Operated production AWS EKS and GCP GKE clusters across 4+ years, managing ingress gateways, network policies, Kyverno admission guardrails, and zero-downtime rolling deployments.
- **Cloud FinOps (-25% Spend)**: Reduced cloud compute spend by 25% through Horizontal Pod Autoscaler (HPA) tuning, workload request/limit right-sizing, and spot/preemptible node pool scheduling.
- **Observability & Incident Response (-60% MTTD / -45% MTTR)**: Architected unified Prometheus, Loki, Alertmanager, Grafana, and OpenTelemetry pipelines with trace-correlated structured logging and automated runbooks.
- **GitOps Delivery (-40% Lead Time)**: Built declarative ArgoCD and GitHub Actions release pipelines with automated smoke gates and fail-forward rollback.

---

## Technical Stack

| Domain | Technologies & Primitives |
| :--- | :--- |
| **Systems Programming** | Safe Rust (`#![forbid(unsafe_code)]`, Tokio, Axum, Hyper, Serde, Nix, Socket2), Go (`client-go`), Python, Bash |
| **Linux & Networking** | `posix_fadvise`, PipeWire SPA/Loopback, Unix Domain Sockets, RFC 9110 HTTP/2, RTP/RTSP, ChaCha20-Poly1305, SRP-6a, Tailscale LocalAPI |
| **Cloud & Orchestration** | Kubernetes (AWS EKS, GCP GKE, Kind, K3s), Docker / OCI Containers (`--cap-drop=ALL`, `--read-only`, cgroups v2), Envoy Gateway |
| **IaC & GitOps** | Terraform (HCL), Helm, ArgoCD (Sync Waves / App-of-Apps), Kyverno, GitHub Actions (Keyless OIDC), SOPS + `age` |
| **Observability & AI Protocols** | Prometheus, Loki, Grafana, Alertmanager, OpenTelemetry, Model Context Protocol (MCP / JSON-RPC 2.0) |
