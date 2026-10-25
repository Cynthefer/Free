# WebServer Engine in C++ — Development Roadmap

> **Status Legend:** `[ ]` Not Started · `[~]` In Progress · `[x]` Complete
> **Target:** Production-grade, modern C++ (C++20/23), passing modern standards (CERT, MISRA where applicable, sanitizers clean, high test coverage).

---

## 🧱 Phase 1 — Basic Features (Foundation)

### Core Networking
- [ ] TCP socket abstraction (POSIX + Windows/Winsock portability layer)
- [ ] IPv4 & IPv6 support (`AF_INET` / `AF_INET6`, dual-stack)
- [ ] Listening socket with `SO_REUSEADDR` / `SO_REUSEPORT`
- [ ] Accept loop with graceful shutdown
- [ ] Non-blocking sockets + I/O multiplexing
  - [ ] `epoll` (Linux)
  - [ ] `kqueue` (BSD/macOS)
  - [ ] `IOCP` / `select` fallback (Windows)
- [ ] Connection lifecycle management (accept → read → write → close)
- [ ] Configurable backlog, timeouts, and `TCP_NODELAY`

### HTTP/1.1 Protocol
- [ ] Request line parsing (method, URI, version)
- [ ] Header parsing (case-insensitive, multi-value, `obs-fold` rejection)
- [ ] Content-Length & chunked transfer decoding
- [ ] Response generation (status line, headers, body)
- [ ] Keep-alive / persistent connections
- [ ] Correct status codes (1xx–5xx) and reason phrases
- [ ] URI decoding + path normalization (reject `..` traversal)
- [ ] Query string parsing
- [ ] MIME type resolution
- [ ] `HEAD`, `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS` handlers

### Static File Serving
- [ ] Root directory configuration
- [ ] `sendfile` / `splice` zero-copy on Linux
- [ ] `Range` request support (206 Partial Content)
- [ ] `ETag` + `Last-Modified` + `If-None-Match` / `If-Modified-Since`
- [ ] Directory index (`index.html`) + directory listing (opt-in)
- [ ] Compression negotiation (`gzip`, `br` if feasible)
- [ ] Path sandboxing / chroot-style containment

### Build & Tooling
- [ ] CMake ≥ 3.20 build system
- [ ] Package managers: `vcpkg` / `Conan` manifest
- [ ] Compiler matrix: GCC, Clang, MSVC (C++20)
- [ ] `-Wall -Wextra -Wpedantic -Werror` clean build
- [ ] Basic unit test harness (Catch2 / GoogleTest / doctest)

---

## ⚙️ Phase 2 — Intermediate Features

### Event Loop & Concurrency
- [ ] Event-loop abstraction (reactor pattern) decoupled from backend
- [ ] Thread pool for blocking work (file I/O, handlers)
- [ ] Timer wheel / `timerfd` for connection & keep-alive timeouts
- [ ] Signal handling (`SIGINT`/`SIGTERM` graceful drain, `SIGHUP` reload)
- [ ] Backpressure handling (write buffer high-water marks)
- [ ] Slowloris protection (header/body read deadlines)

### HTTP/1.1 Hardening
- [ ] Strict header size / count limits
- [ ] Request smuggling defenses (CL/TE conflict rejection)
- [ ] `Host` header validation
- [ ] `Expect: 100-continue` handling
- [ ] Multipart/form-data parser (streaming)
- [ ] `multipart/byteranges` response support
- [ ] Cookie parsing/serialization
- [ ] `Connection`, `Upgrade`, `Keep-Alive` semantics

### TLS
- [ ] OpenSSL 3.x / BoringSSL integration
- [ ] TLS 1.2 + 1.3 only (no SSLv3/TLS 1.0/1.1)
- [ ] SNI support + certificate selection
- [ ] ALPN negotiation (`h2`, `http/1.1`)
- [ ] OCSP stapling (optional)
- [ ] Session resumption (tickets + IDs)
- [ ] Hot certificate reload

### Configuration & Operations
- [ ] Config file format (TOML / YAML / JSON) with schema validation
- [ ] CLI flags (`--config`, `--port`, `--root`, `--workers`)
- [ ] Structured logging (JSON) with levels + rotation
- [ ] Access logs (Common/Combined Log Format)
- [ ] Runtime stats endpoint (`/metrics` internal)
- [ ] Graceful restart / zero-downtime reload
- [ ] PID file + daemonization (POSIX)

### Testing
- [ ] Unit tests for parsers, URI handling, config
- [ ] Integration tests with real sockets
- [ ] HTTP conformance suite (h2spec for HTTP/2, WPT subset for HTTP/1)
- [ ] Fuzzing harnesses (libFuzzer: request parser, header parser, multipart)
- [ ] Benchmark harness (`wrk` / `h2load` CI baselines)

---

## 🚀 Phase 3 — High-Level Features

### HTTP/2
- [ ] HPACK compression (encoder/decoder with dynamic table)
- [ ] Binary framing layer (DATA, HEADERS, SETTINGS, WINDOW_UPDATE, RST_STREAM, GOAWAY, PING)
- [ ] Stream multiplexing + flow control (connection + stream windows)
- [ ] Server push (optional, deprecated in browsers — behind flag)
- [ ] `h2c` (cleartext upgrade / prior knowledge) for internal use
- [ ] Priority handling (RFC 9218 extensible priorities)

### HTTP/3 + QUIC
- [ ] QUIC transport (integrate `quiche`, `msquic`, or `ngtcp2` + `nghttp3`)
- [ ] QPACK header compression
- [ ] 0-RTT resumption
- [ ] Connection migration
- [ ] Alt-Svc advertisement from HTTP/1.1 / HTTP/2

### Advanced Delivery
- [ ] WebSocket (RFC 6455): handshake, framing, ping/pong, close codes
- [ ] Server-Sent Events (SSE)
- [ ] Reverse proxy mode (upstream pools, health checks, retries)
- [ ] Load balancing strategies (round-robin, least-conn, consistent hash)
- [ ] HTTP caching layer (RFC 9111): `Cache-Control`, `Vary`, `stale-while-revalidate`
- [ ] Rate limiting (token bucket / leaky bucket per-IP, per-route)
- [ ] Virtual hosts (SNI + `Host`-based routing)
- [ ] URL rewriting & redirect rules

### Security
- [ ] HSTS, CSP, `X-Content-Type-Options`, `Referrer-Policy` presets
- [ ] CORS engine (preflight, credentials, wildcard rules)
- [ ] Basic + Digest auth, optional JWT verification hook
- [ ] mTLS client certificate verification
- [ ] Request size / rate quotas
- [ ] WAF-style rule hooks (regex deny-lists, IP allow/deny)
- [ ] Secrets management (env / file / vault integration hook)

### Observability
- [ ] Prometheus metrics exporter (`/metrics`)
- [ ] OpenTelemetry tracing (W3C `traceparent` propagation)
- [ ] Health / readiness / liveness endpoints
- [ ] Admin API (stats, config dump, connection list)
- [ ] Profiling hooks (`perf`, `tracy`, `gperftools` integration)

---

## 💎 Phase 4 — Exclusive / Differentiating Features

- [ ] **Zero-copy pipeline**: `io_uring` backend (Linux 5.19+) alongside epoll
- [ ] **Coroutine-native handlers**: C++20 coroutines as the primary async model
- [ ] **Hot module reloading** for handler plugins (`.so`/`.dll` with ABI versioning)
- [ ] **Scriptable routes**: embedded Lua / QuickJS for request hooks
- [ ] **Built-in ACME (Let's Encrypt) client** with auto-renewal
- [ ] **Adaptive concurrency**: auto-tune worker count / buffer sizes from live load
- [ ] **Fair-share scheduler**: per-tenant / per-IP bandwidth & request quotas
- [ ] **Deterministic replay mode**: record & replay request traces for debugging
- [ ] **In-process cache with stampede protection** (singleflight per key)
- [ ] **Edge features**: Brotli dynamic compression with shared dictionaries (RFC 8478 / RFC 9842)
- [ ] **QUIC datagram extension** support (WebTransport readiness)
- [ ] **eBPF observability** integration (Linux): per-connection latency histograms
- [ ] **WASM handler sandbox** (wasmtime) for user-supplied request filters
- [ ] **Chaos-testing mode**: inject latency, drops, malformed frames behind a flag
- [ ] **Self-hosted dashboard** (static SPA served by the engine itself)

---

## 🏭 Production Readiness Checklist

### Correctness & Standards
- [ ] RFC 9110/9111/9112 (HTTP semantics, caching, HTTP/1.1) compliance
- [ ] RFC 9113 (HTTP/2) + RFC 9114 (HTTP/3) compliance
- [ ] RFC 8446 (TLS 1.3) + RFC 5280 (X.509 path validation)
- [ ] RFC 6455 (WebSocket) compliance
- [ ] HTTP conformance suites pass in CI (`h2spec`, `h3spec`, WPT subset)

### Code Quality
- [ ] C++20/23 throughout; no raw `new`/`delete`; RAII everywhere
- [ ] `std::span`, `std::string_view`, `std::expected`/`tl::expected` for errors
- [ ] No exceptions across ABI boundaries; `noexcept` correctness audited
- [ ] Thread sanitizer (TSan), Address sanitizer (ASan), UBSan clean in CI
- [ ] Valgrind / `-fsanitize=memory` clean
- [ ] `clang-tidy` with `modernize-*`, `bugprone-*`, `performance-*`, `cert-*`
- [ ] `cppcheck` + `include-what-you-use` clean
- [ ] Static analysis: Coverity / SonarQube / CodeQL clean
- [ ] API/ABI stability policy documented (semver + `extern "C"` plugin ABI)

### Reliability
- [ ] Zero-downtime upgrades (SO_REUSEPORT handoff + graceful drain)
- [ ] Crash-safe config parsing with rollback
- [ ] Watchdog + supervisor-friendly exit codes (systemd `sd_notify`)
- [ ] Memory limits + OOM behavior documented
- [ ] Circuit breakers on upstream proxy failures
- [ ] Soak tests: 24h+ under load, RSS stable, FD leaks = 0

### Security
- [ ] Threat model documented (STRIDE)
- [ ] SLSA Level 3 provenance + signed releases (Sigstore/cosign)
- [ ] SBOM generated (CycloneDX / SPDX) per release
- [ ] Dependency scanning (Dependabot / Renovate) + CVE SLAs
- [ ] Fuzz targets run in OSS-Fuzz or self-hosted 24/7
- [ ] Responsible disclosure policy (`SECURITY.md`)
- [ ] Secrets never logged; PII scrubbing in logs

### Performance
- [ ] Benchmark suite with regression gates in CI (p99 latency, RPS, RSS)
- [ ] `perf` flamegraphs for hot paths; no obvious syscall storms
- [ ] Target: >100k req/s static on 8-core, p99 < 1 ms LAN
- [ ] Memory per idle connection < 8 KB
- [ ] TLS handshake latency benchmarked and tracked

### Documentation & DX
- [ ] README with 5-minute quickstart
- [ ] Full config reference + recipes
- [ ] API docs (Doxygen/Sphinx) for embeddable use
- [ ] Architecture doc + diagrams (event loop, threading model)
- [ ] `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `CHANGELOG.md`
- [ ] Example deployments: Docker, Kubernetes Helm chart, systemd unit
- [ ] Reproducible builds (hermetic toolchain, pinned deps)

### Release Engineering
- [ ] CI matrix: Linux (glibc + musl), macOS, Windows
- [ ] Static + shared library builds
- [ ] Docker images (distroless, multi-arch)
- [ ] Semantic versioning + LTS branch policy
- [ ] Release checklist (tag → build → sign → SBOM → publish)

---

## 📊 Milestones

| Milestone | Scope | Definition of Done |
|-----------|-------|--------------------|
| **M1 — MVP** | Phase 1 | Serves static files over HTTP/1.1, passes basic tests |
| **M2 — Stable** | Phase 2 | TLS, config, logging, graceful reload, sanitizer clean |
| **M3 — Modern** | Phase 3 | HTTP/2 + WebSocket + proxy, h2spec green |
| **M4 — 1.0** | Phase 4 + Prod checklist | Signed release, SBOM, OSS-Fuzz, benchmarks published |

---

*Last updated: 2026-10-25*
