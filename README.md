## 📘 Senior Exercise: Real-Time Document Processing Pipeline

### Business Context
Users upload documents (JSON, CSV, images). The system must:
- Accept uploads via API
- Process them asynchronously (extract metadata, generate thumbnails for images, validate schemas for JSON/CSV)
- Notify connected clients in real-time about processing status
- Handle failures gracefully with retries
- Provide audit trail of all actions

---

## 🧠 Core Requirements (Senior-level complexity)

### Backend (.NET 8+)
- [ ] **Clean Architecture** – separate projects for Domain, Application, Infrastructure, API
- [ ] **MediatR** for command/query separation
- [ ] **Background processing** – not just `Task.Run`, but proper `IHostedService` or **Quartz.NET/Hangfire**
- [ ] **PostgreSQL** with Entity Framework Core (code-first migrations)
- [ ] **Redis** for:
  - Distributed locking (prevent duplicate processing)
  - Real-time status tracking (cache + pub/sub)
- [ ] **SignalR** hub for real-time client updates
- [ ] **Outbox pattern** (simplified) – store events in DB, background service publishes to Redis
- [ ] **Retry with exponential backoff** (failed processing retries up to 3 times)
- [ ] **Health checks** – liveness, readiness, database, Redis
- [ ] **Serilog** with structured logging + seq/console sinks
- [ ] **OpenTelemetry** traces (at least for HTTP requests and background jobs)

### API Design
- [ ] `POST /api/documents` – upload (multipart/form-data) → returns `202 Accepted` with `Location` header
- [ ] `GET /api/documents/{id}/status` – returns current status + progress
- [ ] `GET /api/documents/{id}/audit` – returns timeline of events (uploaded, processing started, retries, completed, failed)
- [ ] `GET /api/documents` – paginated list with filters (status, date range)
- [ ] Proper idempotency keys for uploads
- [ ] Rate limiting (e.g., 10 uploads per minute per IP)

### Frontend (Angular 17+)
- [ ] **Standalone components** (no NgModules except for root)
- [ ] **SignalR client** – real-time status updates
- [ ] **Drag-drop file upload zone** with progress indicator
- [ ] **Document list** with virtual scrolling (if many items)
- [ ] **Detail view** showing:
  - Full audit trail
  - Processing results (metadata, thumbnail preview)
  - Retry history
- [ ] **State management** with NgRx or Signals (not just simple services)
- [ ] **Error boundary pattern** – graceful fallback UI
- [ ] **Lazy loading** for detail route

### Docker & Orchestration
- [ ] Multi-stage Dockerfiles with **non-root user**
- [ ] `docker-compose.yml` with:
  - API
  - Angular (nginx)
  - PostgreSQL (with volume)
  - Redis
  - Seq (for logs)
  - Jaeger (for traces)
- [ ] **Healthcheck** commands on all containers
- [ ] **Profiles** for dev vs production-like (e.g., `--profile monitoring` spins up Seq+Jaeger)
- [ ] Environment-specific overrides (`docker-compose.override.yml`)
- [ ] **Init scripts** – ensure DB migrations run on startup (using `dotnet run --migrate` entrypoint)

### Quality & Observability
- [ ] Integration test covering: upload → processing → real-time notification
- [ ] Performance test document: **handle 100 concurrent uploads** without crashing (simulate with k6/Artillery)
- [ ] Graceful shutdown handling (SIGTERM → finish processing current jobs → 10s timeout)
- [ ] Structured logs include `CorrelationId` across HTTP + background jobs + SignalR
- [ ] Metrics endpoint (`/metrics`) exposing Prometheus-style: upload count, processing duration histogram, retry count

---

## 📅 1-Week Senior Delivery Schedule

| Day | Focus | Deliverable |
|-----|-------|-------------|
| **Day 1** | Architecture + Base Infrastructure | Solution structure, Docker Compose with DB/Redis, EF Core migrations, health checks |
| **Day 2** | Upload + Background Processing | MediatR, `IHostedService` processing, retry logic, outbox events |
| **Day 3** | SignalR + Angular Core | Real-time updates, drag-drop upload, basic document list |
| **Day 4** | Audit Trail + Observability | Logging, tracing, audit endpoints, Angular detail view with timeline |
| **Day 5** | Testing + Performance + Polish | Integration tests, k6 script, metrics, graceful shutdown, README with architecture diagram |

---

## 🎯 Senior-Level Differentiators

A junior/mid will struggle with:
- **Async processing decoupling** – not just "await Task.Run"
- **Distributed state** – Redis for locking + cache + pub/sub
- **Observability** – logs, metrics, traces working together
- **Resilience patterns** – retry, outbox, idempotency
- **Real-time architecture** – SignalR + background processing race conditions
- **Docker maturity** – non-root, healthchecks, profiles, init ordering

---

## 🔥 Stretch Goals (For exceptional candidates)

- [ ] **Circuit breaker** for downstream dependencies (simulated flaky processor)
- [ ] **Claim-check pattern** – store file in S3/MinIO, only metadata in API
- [ ] **Horizontal scaling** – multiple API instances with Redis backplane for SignalR
- [ ] **OpenAPI specification** (Swagger) with examples and descriptions
- [ ] **Docker Swarm or k8s** readiness probes YAML
- [ ] **GitHub Actions CI** – build, test, push to registry

---

## 📝 Deliverables for Review

- GitHub repo with **atomic commits** (no "fix lint" mess)
- **Architecture decision record (ADR)** explaining why Hangfire vs Quartz, why Redis pub/sub vs polling
- **README** with:
  - Architecture diagram (Mermaid or AsciiFlow)
  - How to run with `docker-compose up`
  - How to run integration tests
  - Performance test instructions
- **Postman/bruno collection** for API testing
- **Screencast** (3 min) showing:
  - Multiple concurrent uploads
  - Real-time status updates across browser tabs
  - Container failure recovery (kill Redis → auto-restart → processing resumes)

---

## ✅ Evaluation Criteria (Senior level)

| Area | Weight | Must demonstrate |
|------|--------|-------------------|
| Architecture | 25% | Clean separation, async patterns, no tight coupling |
| Resilience | 20% | Retries, idempotency, graceful shutdown |
| Observability | 15% | Can debug using logs+traces+metrics alone |
| Real-time | 15% | SignalR works under load, no missed updates |
| Docker maturity | 15% | Production-ready configs, not just "it runs" |
| Testing strategy | 10% | Integration test, not just unit tests |
