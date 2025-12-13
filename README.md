# Awesome Architecture Stack [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A comprehensive guide to every architectural decision you need to make when building software systems.

Not just a list of technologies—a **decision framework** for understanding what problems each layer solves and why it matters.

## Two Ways to Navigate

### 1. By Horizontal Layers (Technology Stack)
Build from bottom up - foundation to presentation:
- **L1: Foundation** - Config, security, observability *(planned)*
- **L2: Platform** - Compute, storage, queues *(planned)*
- **L3: Data** - Caching, validation, schema *(planned)*
- **L4: Integration** - APIs, real-time, plugins *(planned)*
- **L5: Business** - Users, auth, billing, payments *(planned)*
- **L6: Application** - State, logic, AI/ML *(planned)*
- **L7: Presentation** - Frontend, design systems *(planned)*

### 2. By Vertical Stacks (Feature Domains)
Build complete features end-to-end:
- [Auth Stack](stacks/auth.md) - Identity, auth, sessions, permissions (L1→L7)
- [Data Stack](stacks/data.md) - Storage, caching, APIs, validation (L2→L7)
- [Payment Stack](stacks/payments.md) - Billing, metering, invoicing (L1→L7)
- [Communication Stack](stacks/communication.md) - Real-time, messaging, notifications (L2→L7)
- [Content Stack](stacks/content.md) - Storage, CDN, search, delivery (L2→L7)
- [Analytics Stack](stacks/analytics.md) - Events, metrics, dashboards (L1→L7)
- [AI Stack](stacks/ai.md) - Model hosting, inference, embeddings (L2→L7)
- [Integration Stack](stacks/integrations.md) - External APIs, webhooks, plugins (L4→L7)

### 3. By Cross-Cutting Concerns
Apply across all layers and stacks:
- **Testing Strategy** *(planned)*
- **Error Handling & Resilience** *(planned)*
- **Performance Optimization** *(planned)*
- **Security & Compliance** - See [SECURITY.md](../SECURITY.md)
- **Developer Experience** *(planned)*
- **Observability** *(planned)*

---

## Understanding the Architecture Matrix

Software architecture is a **matrix** of horizontal layers and vertical stacks:

```
                  AUTH     DATA    PAYMENTS  CONTENT   AI/ML
                  STACK    STACK   STACK     STACK     STACK
                  │        │       │         │         │
L7: Presentation  ├──UI────┼──UI───┼──UI─────┼──UI─────┼──UI──
L6: Application   ├──State─┼──Logic┼──Logic──┼──Logic──┼──Logic
L5: Business      ├──Users─┼───────┼──Billing┼─────────┼──────
L4: Integration   ├──API───┼──API──┼──API────┼──CDN────┼──API─
L3: Data         ├──Cache─┼──Valid┼──Meter──┼──Cache──┼──────
L2: Platform     ├───────┼──DB───┼─────────┼──Storage┼──GPU──
L1: Foundation   ├──Vault─┼──────┼──Secrets┼─────────┼──Logs─
```

**Choose your path:**
- **Horizontal (Layers)**: "I need to choose a database" → Data Layer *(see [Data Stack](stacks/data.md))*
- **Vertical (Stacks)**: "I'm building authentication" → [Auth Stack](stacks/auth.md)
- **Cross-Cutting**: "How do I test this?" → Testing *(planned)*

---

## How to Use This Guide

Each document follows this structure:

1. **What**: What problem does this solve?
2. **Why It Matters**: Business and technical impact
3. **Decision Dimensions**: Key factors to consider
4. **Common Choices**: Tools/patterns with trade-offs
5. **Trade-off Tables**: Quick comparison
6. **Decision Framework**: How to evaluate
7. **Real-World Examples**: Lessons from production

**This is NOT a "best practices" list.** It's a **decision framework** for choosing what's right for YOUR context.

---

## Quick Start Paths

### Path 1: Building a SaaS Product
1. Start with [Auth Stack](stacks/auth.md) - Get users in
2. Then [Data Stack](stacks/data.md) - Store their data
3. Then [Payment Stack](stacks/payments.md) - Monetize
4. Then [Analytics Stack](stacks/analytics.md) - Understand usage

### Path 2: Building an API Platform
1. Start with Integration Layer - API design *(planned)*
2. Then [Data Stack](stacks/data.md) - Data storage & access
3. Then [Auth Stack](stacks/auth.md) - API authentication
4. Then [Analytics Stack](stacks/analytics.md) - Track usage

### Path 3: Building an AI-Powered App
1. Start with [AI Stack](stacks/ai.md) - Model infrastructure
2. Then [Data Stack](stacks/data.md) - Training & inference data
3. Then [Auth Stack](stacks/auth.md) - User access control
4. Then [Payment Stack](stacks/payments.md) - Usage-based billing

### Path 4: Building a Content Platform
1. Start with [Content Stack](stacks/content.md) - Storage & CDN
2. Then [Data Stack](stacks/data.md) - Metadata & search
3. Then [Auth Stack](stacks/auth.md) - Access control
4. Then [Analytics Stack](stacks/analytics.md) - Content metrics

---

## Layer Reference (Horizontal)

### 1.1 Configuration Management

**What**: How application behavior is configured across environments.

**Why It Matters**:
- Different behavior in dev/staging/production
- Secrets management (API keys, credentials)
- Feature flags and rollout control
- Multi-tenant configuration

**Decision Dimensions**:
- **Storage**: Environment variables, config files, config service, vault
- **Loading**: Startup vs. runtime, hierarchical override
- **Validation**: Type-safe, schema-validated, or stringly-typed
- **Distribution**: Local files, remote config service, version control

**Common Choices**:

| Approach | Pros | Cons | Use When |
|----------|------|------|----------|
| **Env vars only** | Simple, 12-factor compliant | No structure, hard to validate | Small apps, simple config |
| **Config files (YAML/JSON)** | Structured, versioned | Secrets in repo (danger) | Need hierarchy, complex config |
| **Remote config service** | Dynamic updates, audited | Network dependency, complexity | Feature flags, A/B testing |
| **Secrets vault** | Secure, centralized | Ops burden, complexity | Storing credentials, keys |

**Tools**:
- **Environment Variables**: dotenv, dotenv-flow, direnv
- **Hierarchical Config**: Hydra (Python), config (Node.js), Viper (Go)
- **Remote Config**: AWS AppConfig, LaunchDarkly, ConfigCat
- **Secrets Management**: HashiCorp Vault, AWS Secrets Manager, Doppler

**Decision Framework**:
1. How many environments? (dev/staging/prod/...)
2. Do configs change at runtime? (feature flags)
3. How sensitive are secrets? (API keys, DB passwords)
4. Multi-tenant or single-tenant?

---

### 1.2 Environment Management

**What**: How different runtime environments are isolated and managed.

**Why It Matters**:
- Prevent production data corruption during testing
- Isolated development sandboxes
- Reproducible builds and deployments
- Dependency version locking

**Decision Dimensions**:
- **Runtimes**: Node version, Python version, system dependencies
- **Package Management**: Lock files, monorepo vs. polyrepo
- **Environment Isolation**: Docker, VMs, native
- **Reproducibility**: Nix, Docker, lock files

**Common Choices**:

| Approach | Pros | Cons | Use When |
|----------|------|------|----------|
| **Native + lock files** | Fast, simple | "Works on my machine" issues | Small teams, simple stack |
| **Docker containers** | Reproducible, isolated | Slow on macOS, complexity | Need exact environment parity |
| **Nix** | Perfectly reproducible | Steep learning curve | Need hermetic builds |
| **VMs** | Complete isolation | Heavy, slow | Legacy systems, strict isolation |

**Tools**:
- **Runtime Managers**: nvm (Node.js), pyenv (Python), rbenv (Ruby), asdf (multi)
- **Package Managers**: pnpm/npm/yarn (Node), poetry/pip (Python), cargo (Rust)
- **Containerization**: Docker, Podman
- **Hermetic Builds**: Nix, Bazel

---

### 1.3 Security & Identity

**What**: How users are identified, authenticated, and authorized.

**Why It Matters**:
- Prevent unauthorized access
- Audit trail (who did what, when)
- Compliance (HIPAA, SOC2, GDPR)
- Multi-tenancy and data isolation

**Decision Dimensions**:
- **Authentication**: Passwords, OAuth, SAML, SSO, MFA
- **Authorization**: RBAC, ABAC, policy-based (OPA)
- **Session Management**: JWT, session cookies, refresh tokens
- **Credential Storage**: Hashing (bcrypt, argon2), vault
- **API Security**: API keys, OAuth2, mutual TLS

**Common Choices**:

| Approach | Pros | Cons | Use When |
|----------|------|------|----------|
| **Auth provider (Auth0, Clerk)** | Fast setup, compliance-ready | Vendor lock-in, cost | Need quick launch, compliance |
| **Self-hosted (Keycloak, Ory)** | Full control, no vendor lock | Maintenance burden, security risk | Need customization, no vendor lock |
| **Framework built-in** | Integrated, flexible | Security responsibility on you | Simple auth, custom needs |
| **Enterprise SSO (Okta, Azure AD)** | Enterprise features, compliance | Expensive, complex setup | B2B SaaS, enterprise customers |

**Tools**:
- **Auth Providers**: Auth0, Clerk, Supabase Auth, Firebase Auth, WorkOS
- **Self-Hosted**: Keycloak, Ory, Authentik, FusionAuth
- **Framework**: NextAuth.js, Passport.js, Django AllAuth
- **Policy Engines**: Open Policy Agent (OPA), Casbin, Oso

**Key Questions**:
1. B2C or B2B? (Social login vs. SSO)
2. Compliance requirements? (HIPAA, SOC2)
3. Multi-tenant? (Org-level isolation)
4. Budget? (Self-hosted vs. managed)

---

### 1.4 Observability & Monitoring

**What**: How system behavior is measured, logged, and alerted.

**Why It Matters**:
- Detect production failures before users complain
- Debug issues in production (without local reproduction)
- Performance optimization (find bottlenecks)
- Compliance audit trails (who changed what)

**The Three Pillars**:
1. **Logs**: Discrete events (errors, warnings, info)
2. **Metrics**: Numerical measurements (CPU, latency, requests/sec)
3. **Traces**: Request flows across services (distributed tracing)

**Common Choices**:

| Approach | Pros | Cons | Use When |
|----------|------|------|----------|
| **OSS Stack (Prometheus + Grafana)** | Free, full control | Ops burden, setup complexity | Have DevOps team, cost-sensitive |
| **Cloud Native (CloudWatch, Azure)** | Integrated, managed | Vendor lock-in, cost scales | All-in on one cloud |
| **SaaS APM (Datadog, New Relic)** | Turnkey, powerful | Expensive at scale | Need fast setup, deep insights |
| **Error Tracking (Sentry, Rollbar)** | Error-focused, affordable | Limited to errors | Need error tracking first |

**Tools**:
- **Logging**: Winston, Bunyan, Pino (Node), loguru (Python), Loki
- **Metrics**: Prometheus, StatsD, InfluxDB
- **Tracing**: OpenTelemetry, Jaeger, Zipkin
- **APM**: Datadog, New Relic, Elastic APM, Honeycomb
- **Error Tracking**: Sentry, Rollbar, Bugsnag

**Decision Framework**:
1. Do you need real-time alerts? (metrics + alerting)
2. Microservices? (distributed tracing essential)
3. Compliance? (log retention, immutability)
4. Budget? (OSS vs. SaaS)

---

## L2: Platform Layer

### 2.1 Compute & Hosting

**What**: Where code runs (cloud, on-prem, edge).

**Why It Matters**:
- **Cost**: Pay per request vs. always-on servers
- **Scalability**: Handle traffic spikes (autoscaling)
- **Latency**: User proximity (edge, multi-region)
- **Compliance**: Data residency (EU-only, US-only)
- **Ops Burden**: Managed vs. self-managed

**Compute Models**:

| Model | How It Works | Best For | Avoid When |
|-------|--------------|----------|------------|
| **VMs (EC2, GCE)** | Always-on servers | Long-running processes, databases | Unpredictable traffic, tight budget |
| **Containers (K8s, ECS)** | Portable, scalable units | Microservices, polyglot apps | Small projects, simple apps |
| **Serverless (Lambda, Cloud Run)** | Pay per request, auto-scale | Event-driven, bursty traffic | Long-running tasks (15min limit) |
| **PaaS (Vercel, Heroku, Render)** | Deploy from git push | Fast prototyping, small teams | Need low-level control |
| **Edge (Cloudflare Workers, Vercel Edge)** | Run near users globally | Low latency, API gateways | Stateful apps, large bundles |

**Common Choices**:

| Provider | Strengths | Weaknesses | Use When |
|----------|-----------|------------|----------|
| **AWS** | Comprehensive, mature | Complex, steep learning curve | Need full cloud suite |
| **GCP** | Great for ML/data, BigQuery | Smaller ecosystem than AWS | Data-heavy, ML workloads |
| **Azure** | Enterprise, Microsoft integration | Less developer-friendly | Enterprise, .NET shops |
| **Vercel** | Best Next.js experience, fast deploys | Pricey at scale, vendor lock-in | Next.js apps, fast iteration |
| **Cloudflare Workers** | Global edge, fast, cheap | Limited runtime (no fs, 1MB bundle) | API gateways, edge functions |
| **Modal** | Serverless GPU, Python-first | Young, less mature | AI/ML workloads, GPU needs |

**Decision Framework**:
1. What's your traffic pattern? (steady, bursty, unpredictable)
2. Latency sensitivity? (need edge, multi-region)
3. Compliance? (data residency, HIPAA, SOC2)
4. Team skills? (K8s expertise, prefer managed)
5. Cost model? (pay-as-you-go vs. reserved)

---

### 2.2 Storage & Databases

**What**: How data is persisted long-term.

**Why It Matters**:
- **Durability**: Don't lose data (ACID guarantees)
- **Query Performance**: Fast reads/writes (indexes, caching)
- **Scalability**: Handle growth (vertical, horizontal, sharding)
- **Consistency**: Trade-offs (CAP theorem: Consistency, Availability, Partition tolerance)

**Database Types**:

| Type | Data Model | Use Cases | Examples |
|------|------------|-----------|----------|
| **Relational (SQL)** | Tables, rows, columns | Structured data, ACID transactions | PostgreSQL, MySQL, SQL Server |
| **Document (NoSQL)** | JSON-like documents | Semi-structured, flexible schema | MongoDB, DynamoDB, Firestore |
| **Key-Value** | Simple key → value | Caching, session storage | Redis, DynamoDB, etcd |
| **Graph** | Nodes & edges | Social networks, recommendations | Neo4j, DGraph, Neptune |
| **Time-Series** | Time-stamped data | Metrics, IoT, logs | TimescaleDB, InfluxDB |
| **Search** | Full-text search | Search engines, autocomplete | Elasticsearch, Algolia, Meilisearch |

**Common Choices**:

| Database | Strengths | Weaknesses | Use When |
|----------|-----------|------------|----------|
| **PostgreSQL** | ACID, mature, JSON support | Vertical scaling limits | Need ACID, relational data |
| **MongoDB** | Schema-less, horizontal scale | No ACID (older versions) | Flexible schema, rapid iteration |
| **DynamoDB** | Infinite scale, managed | Expensive, query limitations | AWS-native, need high scale |
| **Redis** | Blazing fast, pub/sub | In-memory (volatile) | Caching, real-time leaderboards |
| **Elasticsearch** | Full-text search, analytics | Complex, resource-heavy | Search, log analysis |

**Decision Framework**:
1. Data model? (relational, document, graph)
2. Query patterns? (key lookups, complex joins, full-text search)
3. Scale requirements? (GB, TB, PB)
4. Consistency needs? (ACID, eventual consistency OK)
5. Managed vs. self-hosted? (ops burden)

---

### 2.3 Message Queues & Event Streaming

**What**: How asynchronous work is distributed and events are published.

**Why It Matters**:
- **Decouple services**: Microservices don't call each other directly
- **Handle traffic spikes**: Queue work, process later
- **Guarantee delivery**: At-least-once, exactly-once semantics
- **Event-driven architecture**: React to events (pub/sub)

**Patterns**:

| Pattern | How It Works | Use Cases |
|---------|--------------|-----------|
| **Queue (Point-to-Point)** | One producer → One consumer | Background jobs, task processing |
| **Pub/Sub** | One producer → Many consumers | Notifications, webhooks, events |
| **Event Stream** | Append-only log, replay | Event sourcing, audit logs, analytics |

**Common Choices**:

| Tool | Type | Strengths | Weaknesses | Use When |
|------|------|-----------|------------|----------|
| **AWS SQS** | Queue | Managed, infinite scale | AWS lock-in, eventual consistency | Simple queues, AWS-native |
| **RabbitMQ** | Queue/Pub-Sub | Mature, flexible | Ops burden, scaling complexity | Need flexibility, on-prem |
| **Apache Kafka** | Event Stream | High throughput, replay | Ops burden, overkill for simple | Event sourcing, analytics |
| **Redis (BullMQ)** | Queue | Simple, Node-native | Single point of failure (Redis) | Node.js apps, simple queues |
| **Google Pub/Sub** | Pub/Sub | Managed, global | GCP lock-in | GCP-native, pub/sub pattern |

**Decision Framework**:
1. Delivery guarantees? (at-most-once, at-least-once, exactly-once)
2. Ordering requirements? (FIFO, unordered OK)
3. Message size? (<256KB, >1MB)
4. Replay needed? (event sourcing, debugging)
5. Cloud provider? (native integration)

---

## L3: Data Layer

### 3.1 Caching Strategy

**What**: How frequently accessed data is stored in-memory or near-compute.

**Why It Matters**:
- **Reduce latency**: Avoid DB round-trips (ms → µs)
- **Reduce cost**: Fewer API calls, DB queries
- **Improve UX**: Instant responses, offline-first
- **Handle rate limits**: Cache API responses

**Cache Levels**:

| Level | Location | Latency | Use Cases |
|-------|----------|---------|-----------|
| **Browser Cache** | User's browser | 0ms | Static assets (images, CSS, JS) |
| **CDN Cache** | Edge servers (global) | 10-50ms | Public content, APIs (GET only) |
| **Application Cache** | In-process memory | <1ms | Session data, computed results |
| **Distributed Cache** | Redis, Memcached | 1-5ms | Shared state across servers |
| **Database Cache** | Query cache, indexes | 5-20ms | Frequently queried data |

**Common Choices**:

| Approach | Pros | Cons | Use When |
|----------|------|------|----------|
| **In-process (lru-cache, Map)** | Fast, simple | Lost on restart, not shared | Single server, ephemeral data |
| **Redis** | Shared, persistent | Network hop, complexity | Multi-server, shared state |
| **CDN (Cloudflare, Fastly)** | Global, low latency | Stale data, invalidation hard | Public APIs, static content |
| **Client-side (React Query, SWR)** | Instant UX, offline | Browser storage limits | Frontend apps, improve UX |

**Eviction Policies**:
- **LRU (Least Recently Used)**: Remove oldest accessed item
- **LFU (Least Frequently Used)**: Remove least accessed item
- **TTL (Time To Live)**: Expire after N seconds
- **Write-through**: Update cache on every write
- **Write-behind**: Async write to DB, update cache immediately

**Decision Framework**:
1. What's the read/write ratio? (95% reads → cache aggressively)
2. How fresh must data be? (stale OK, real-time)
3. Cache hit rate target? (>80% = effective)
4. Multi-server? (need distributed cache)

---

### 3.2 Data Validation & Schema

**What**: How data integrity is enforced at boundaries (API, DB, forms).

**Why It Matters**:
- **Prevent corruption**: Invalid data breaks system
- **Type safety**: Catch errors at compile-time
- **API contracts**: Frontend ↔ backend agreement
- **Database migrations**: Schema evolution without downtime

**Validation Layers**:

| Layer | Tools | When |
|-------|-------|------|
| **Compile-Time** | TypeScript, Flow, PropTypes | Development (type checking) |
| **Runtime** | Zod, Yup, Joi, AJV (JSON Schema) | API boundaries, user input |
| **Database** | Prisma, TypeORM, Drizzle, SQL schema | Data persistence |
| **API Contract** | OpenAPI, tRPC, GraphQL | Frontend ↔ backend |

**Common Choices**:

| Approach | Pros | Cons | Use When |
|----------|------|------|----------|
| **Zod** | Runtime validation + inferred types | Runtime overhead | TypeScript apps, need both |
| **TypeScript only** | Compile-time safety, no runtime cost | No runtime validation | Internal code, trusted inputs |
| **Prisma** | Type-safe DB, migrations, codegen | Vendor lock-in, limited control | Relational DBs, need type safety |
| **tRPC** | End-to-end type safety | TypeScript only, tight coupling | Monorepo, shared types |
| **JSON Schema** | Language-agnostic, mature | Verbose, separate from code | API validation, polyglot |

**Decision Framework**:
1. Language? (TypeScript → Zod, Python → Pydantic)
2. Runtime validation needed? (user input, API)
3. Compile-time safety priority? (TypeScript)
4. API contract enforcement? (OpenAPI, tRPC)

---

## L4: Integration Layer

### 4.1 API Design & Routing

**What**: How clients communicate with backend services.

**Why It Matters**:
- **Evolvability**: Versioning, backward compatibility
- **Performance**: Payload size, round-trips (over/under-fetching)
- **Developer experience**: Discoverability, documentation
- **Security**: Authentication, rate limiting, CORS

**API Styles**:

| Style | Request Model | Best For | Avoid When |
|-------|---------------|----------|------------|
| **REST** | Resource-based (GET /users/:id) | CRUD operations, public APIs | Complex queries, real-time |
| **GraphQL** | Client specifies fields | Flexible querying, mobile apps | Simple CRUD, caching needs |
| **tRPC** | Type-safe RPC (TypeScript) | Monorepo, full-stack TypeScript | Polyglot, public API |
| **gRPC** | Binary, strongly typed | Microservices, low latency | Browser clients, public API |
| **WebSockets** | Bi-directional, real-time | Chat, live updates, gaming | Simple request/response |

**Common Choices**:

| Framework | Language | Strengths | Use When |
|-----------|----------|-----------|----------|
| **Express.js** | Node.js | Simple, flexible, mature | REST APIs, Node.js |
| **Fastify** | Node.js | Fast, schema-based validation | Need performance, Node.js |
| **Next.js API Routes** | Node.js | Co-located with frontend | Full-stack Next.js apps |
| **tRPC** | TypeScript | End-to-end type safety | Monorepo, TypeScript |
| **Apollo Server** | Node.js | GraphQL, ecosystem | Need GraphQL, flexible queries |
| **FastAPI** | Python | Fast, auto-docs, async | Python, API-first |

**Key Decisions**:
1. **Versioning**: URL path (`/v1/`), headers, content negotiation
2. **Rate Limiting**: Token bucket, leaky bucket, fixed window
3. **Documentation**: OpenAPI (Swagger), GraphQL introspection
4. **Error Handling**: RFC 7807 (Problem Details), custom

---

### 4.2 Real-Time Communication

**What**: How server pushes updates to clients (not just request/response).

**Why It Matters**:
- **Live updates**: Dashboards, chat, collaborative editing
- **Reduce polling**: Save bandwidth, improve UX
- **Event-driven UX**: Reactive interfaces

**Protocols**:

| Protocol | How It Works | Browser Support | Use Cases |
|----------|--------------|-----------------|-----------|
| **WebSockets** | Bi-directional, persistent connection | ✅ Excellent | Chat, gaming, collaborative editing |
| **Server-Sent Events (SSE)** | Server → client only, HTTP-based | ✅ Good (no IE) | Live dashboards, notifications |
| **Long Polling** | Client polls, server delays response | ✅ Universal | Fallback for old browsers |
| **WebRTC** | Peer-to-peer, low latency | ✅ Good | Video/audio calls, file sharing |

**Common Choices**:

| Tool | Type | Pros | Cons | Use When |
|------|------|------|------|----------|
| **Socket.IO** | WebSocket (+ fallbacks) | Auto-reconnect, room support | Large bundle | Node.js, need fallbacks |
| **Native WebSocket** | WebSocket | Lightweight, standard | Manual reconnect, no rooms | Simple use cases |
| **Pusher / Ably** | SaaS | Managed, scalable | Cost, vendor lock-in | Need managed, global scale |
| **SSE (EventSource)** | SSE | Simple, HTTP-based | Server → client only | One-way updates |

**Decision Framework**:
1. Bi-directional? (WebSockets vs. SSE)
2. Scaling needs? (Redis adapter, message broker)
3. Browser support? (IE11 → use Socket.IO)
4. Managed vs. self-hosted? (Pusher vs. Socket.IO)

---

### 4.3 External Integrations

**What**: How the system connects to third-party APIs and services.

**Why It Matters**:
- **Leverage platforms**: GitHub, Slack, Google, Stripe
- **Avoid reinventing**: Auth (OAuth), payments, analytics
- **Data synchronization**: Keep external systems in sync
- **Webhooks**: Receive events from external systems

**Integration Patterns**:

| Pattern | How It Works | Use Cases |
|---------|--------------|-----------|
| **REST API Calls** | HTTP requests to external APIs | Fetching data, triggering actions |
| **OAuth** | Delegated authorization | "Sign in with Google", access user data |
| **Webhooks** | External system calls your API | Real-time events (payment succeeded) |
| **Polling** | Periodically fetch changes | No webhooks available |
| **SDK/Library** | Vendor-provided client | Easier than raw HTTP |

**Common Integrations**:

| Category | Services | Integration Method |
|----------|----------|-------------------|
| **Auth** | Google, GitHub, Microsoft | OAuth 2.0 |
| **Payments** | Stripe, PayPal, Braintree | REST API + Webhooks |
| **Communication** | Twilio, SendGrid, Slack | REST API |
| **AI** | OpenAI, Anthropic, Google AI | REST API + Streaming |
| **Version Control** | GitHub, GitLab, Bitbucket | REST/GraphQL API + Webhooks |
| **Project Management** | Jira, Linear, Asana | REST API + Webhooks |

**Decision Framework**:
1. Official SDK? (use it, easier than raw HTTP)
2. Webhooks or polling? (webhooks = real-time, polling = fallback)
3. Rate limits? (implement backoff, caching)
4. Error handling? (retry logic, circuit breakers)
5. Security? (validate webhook signatures, rotate keys)

**Best Practices**:
- **Retry logic**: Exponential backoff (1s, 2s, 4s, ...)
- **Circuit breakers**: Stop calling failing services
- **Webhook validation**: Verify signatures (HMAC)
- **Idempotency**: Handle duplicate webhooks gracefully
- **Timeouts**: Don't wait forever (5-30s max)

---

### 4.4 Plugin & Extension Systems

**What**: How third-party developers extend your application.

**Why It Matters**:
- **Ecosystem growth**: Community-built features
- **Flexibility**: Customers customize without forking
- **Revenue**: Marketplace, paid plugins
- **Reduce core complexity**: Move optional features to plugins

**Plugin Architectures**:

| Architecture | How It Works | Pros | Cons |
|--------------|--------------|------|------|
| **Hooks/Events** | Plugins register callbacks for events | Simple, flexible | No isolation, security risk |
| **Sandboxed (iframe, WebWorkers)** | Plugins run in isolated context | Secure, crash-safe | Complex, performance overhead |
| **CLI Extensions** | Plugins add new commands | Easy to build | No runtime integration |
| **API-Based** | Plugins are separate services | Isolated, polyglot | Network latency, complexity |

**Common Choices**:

| System | Examples | Plugin Model |
|--------|----------|--------------|
| **WordPress** | Thousands of plugins | PHP hooks, full DB access (risky) |
| **VS Code** | Extensions | Node.js APIs, sandboxed processes |
| **Figma** | Plugins | Sandboxed JS, limited APIs |
| **GitHub Actions** | Actions | Docker containers, isolated |
| **Shopify** | Apps | External services, API-based |

**Decision Framework**:
1. Trust level? (first-party, vetted, untrusted)
2. Isolation needs? (security, crash isolation)
3. Plugin language? (same as core, polyglot)
4. Distribution? (bundled, marketplace, npm)

**Security Considerations**:
- **Sandboxing**: Prevent malicious plugins (WebWorkers, containers)
- **Permissions**: Explicit grants (read files, network access)
- **Code review**: Vet plugins before listing
- **Rate limiting**: Prevent plugin abuse

---

## L5: Business Layer

### 5.1 User Management & Multi-Tenancy

**What**: How users are organized into accounts, teams, and organizations.

**Why It Matters**:
- **B2B SaaS**: Multiple companies use same system
- **Data isolation**: Tenant A can't see tenant B's data
- **Billing**: Per-user, per-org pricing
- **Permissions**: Role-based access control (RBAC)

**Multi-Tenancy Models**:

| Model | Database | Pros | Cons | Use When |
|-------|----------|------|------|----------|
| **Shared DB, Shared Schema** | One DB, tenant_id column | Simple, cost-effective | Risk of data leaks | Small tenants, low data volume |
| **Shared DB, Separate Schemas** | One DB, schema per tenant | Better isolation | More complex queries | Medium tenants, PostgreSQL |
| **Separate Databases** | DB per tenant | Complete isolation, compliance | Ops complexity, cost | Large tenants, compliance needs |
| **Hybrid** | Small tenants shared, large tenants separate | Flexible, cost-optimized | Most complex | SaaS with varied customer sizes |

**Common Choices**:

| Approach | Example | Best For |
|----------|---------|----------|
| **Auth provider with orgs** | Clerk, Auth0 Organizations | Fast setup, managed |
| **Custom RBAC** | Casbin, OPA, custom code | Full control, complex rules |
| **Row-Level Security (RLS)** | PostgreSQL RLS, Prisma | DB-level isolation |

**Decision Framework**:
1. Number of tenants? (<100, <10k, >10k)
2. Tenant sizes? (uniform, varied)
3. Compliance needs? (HIPAA, GDPR)
4. Customization per tenant? (branding, features)

---

### 5.2 Authorization & Permissions

**What**: What actions users are allowed to perform.

**Why It Matters**:
- **Security**: Prevent unauthorized actions
- **Compliance**: Audit who can do what
- **Flexibility**: Complex permission rules
- **UX**: Hide/show UI based on permissions

**Permission Models**:

| Model | How It Works | Use Cases |
|-------|--------------|-----------|
| **Role-Based (RBAC)** | Users assigned roles (Admin, Editor, Viewer) | Simple permissions, fixed roles |
| **Attribute-Based (ABAC)** | Rules based on attributes (user.dept === resource.dept) | Complex rules, dynamic |
| **Relationship-Based (ReBAC)** | Permissions based on relationships (owner, collaborator) | Social apps, Google Docs-style |
| **Policy-Based** | Centralized policy engine (OPA, Cedar) | Microservices, audit needs |

**Common Choices**:

| Tool | Model | Pros | Cons | Use When |
|------|-------|------|------|----------|
| **Casbin** | RBAC/ABAC/ReBAC | Flexible, multi-language | Learning curve | Need flexibility |
| **Open Policy Agent (OPA)** | Policy-based | Declarative, auditable | Rego language | Microservices, compliance |
| **Auth0 Authorization** | RBAC | Managed, easy setup | Cost, vendor lock-in | Fast setup, low complexity |
| **Custom middleware** | Any | Full control | Build everything yourself | Simple needs, already built |

**Decision Framework**:
1. Permission complexity? (simple roles, complex rules)
2. Centralized or distributed? (policy engine, code)
3. Audit needs? (who accessed what, when)
4. Dynamic rules? (based on time, location, relationships)

---

### 5.3 Billing & Payments

**What**: How you collect money from customers.

**Why It Matters**:
- **Revenue**: The point of SaaS
- **Compliance**: PCI DSS (never store raw card numbers)
- **UX**: Frictionless checkout increases conversion
- **Flexibility**: Trials, coupons, refunds, proration

**Payment Flow**:

```
Customer → Your App → Payment Processor → Bank
                ↓
           Webhook (payment.succeeded)
                ↓
         Provision access
```

**Common Choices**:

| Provider | Strengths | Weaknesses | Use When |
|----------|-----------|------------|----------|
| **Stripe** | Developer-friendly, global, subscriptions | 2.9% + 30¢ per transaction | B2C/B2B SaaS, global |
| **PayPal** | High trust, consumer-friendly | Developer experience poor | B2C, high-trust needed |
| **Paddle** | Merchant of record (handles tax) | Higher fees, less control | Don't want tax burden |
| **Braintree** | PayPal-owned, Venmo support | Less popular than Stripe | Need PayPal + cards |
| **Lemon Squeezy** | Simple, merchant of record | Newer, less mature | Small SaaS, simple billing |

**Key Features**:

| Feature | What | Why |
|---------|------|-----|
| **Subscriptions** | Recurring billing | SaaS revenue model |
| **Trials** | Free X days, then charge | Increase conversion |
| **Proration** | Adjust charges mid-cycle (upgrades) | Fair billing |
| **Metered billing** | Pay per usage (API calls, seats) | Usage-based pricing |
| **Webhooks** | Payment events (succeeded, failed) | Provision/revoke access |
| **Invoices** | PDF receipts, email | Tax compliance, accounting |
| **Tax calculation** | Auto-calculate sales tax, VAT | Compliance (complex) |

**Decision Framework**:
1. B2C or B2B? (Stripe Billing vs. invoicing)
2. Global? (multi-currency, tax handling)
3. Pricing model? (subscriptions, usage-based, one-time)
4. Tax complexity? (Merchant of Record = easier)

**Best Practices**:
- **Never store raw card numbers** (PCI DSS violation)
- **Use webhooks, not polling** (real-time updates)
- **Handle failed payments gracefully** (retry, email)
- **Idempotency keys** (prevent duplicate charges)
- **Test mode** (test before production)

---

### 5.4 Usage Tracking & Metering

**What**: Measure how customers use your product (for billing, analytics, limits).

**Why It Matters**:
- **Usage-based pricing**: Charge per API call, seat, GB
- **Rate limiting**: Enforce plan limits (1000 requests/month)
- **Analytics**: Understand usage patterns, optimize
- **Fraud detection**: Detect abuse, block bad actors

**Tracking Dimensions**:

| Dimension | Examples | Use Cases |
|-----------|----------|-----------|
| **Requests** | API calls, page views | API billing, rate limits |
| **Storage** | GB stored, files uploaded | Storage billing |
| **Compute** | CPU hours, GPU minutes | Cloud billing (AWS Lambda) |
| **Seats** | Active users, team size | Per-seat pricing (Slack) |
| **Events** | Emails sent, SMS sent | Transactional pricing (Twilio) |

**Common Choices**:

| Approach | Pros | Cons | Use When |
|----------|------|------|----------|
| **In-app counters** | Simple, real-time | Not durable, hard to audit | Simple limits, ephemeral |
| **Database counters** | Durable, queryable | Write-heavy, scaling issues | Moderate traffic |
| **Redis counters** | Fast, atomic increments | Not durable (if not persisted) | High traffic, rate limiting |
| **Event streaming (Kafka)** | Scalable, auditable | Complex, ops burden | High volume, billing audit |
| **SaaS (Stripe Billing, Metronome)** | Managed, billing-integrated | Cost, vendor lock-in | Need turnkey solution |

**Rate Limiting Algorithms**:

| Algorithm | How It Works | Use Cases |
|-----------|--------------|-----------|
| **Fixed Window** | 100 requests per hour (resets at top of hour) | Simple, but burst at window edge |
| **Sliding Window** | 100 requests in last 60 minutes (rolling) | Smoother, more complex |
| **Token Bucket** | Tokens refill at rate, consume per request | Burst-friendly (bank tokens) |
| **Leaky Bucket** | Requests leak out at fixed rate | Smooth traffic, queue requests |

**Decision Framework**:
1. Usage-based pricing? (need accurate metering)
2. Traffic volume? (low → DB, high → Redis/Kafka)
3. Audit requirements? (immutable event log)
4. Real-time vs. batch? (rate limits = real-time, billing = batch OK)

---

### 5.5 Financial Operations

**What**: Accounting, invoicing, revenue recognition, tax compliance.

**Why It Matters**:
- **Legal compliance**: Tax laws (sales tax, VAT)
- **Audit trail**: Who paid what, when
- **Revenue recognition**: SaaS accounting (ASC 606)
- **Reconciliation**: Match payments to orders

**Key Financial Processes**:

| Process | What | Tools |
|---------|------|-------|
| **Invoicing** | Generate PDF invoices, email to customers | Stripe Invoicing, Xero, QuickBooks |
| **Revenue Recognition** | Recognize revenue over subscription period | Stripe Revenue Recognition, Zuora |
| **Tax Calculation** | Calculate sales tax, VAT, GST | Stripe Tax, Avalara, TaxJar |
| **Reconciliation** | Match payments to bank deposits | Stripe Reconciliation, accounting software |
| **Reporting** | P&L, balance sheet, cash flow | QuickBooks, Xero, NetSuite |

**Common Choices**:

| Tool | Category | Strengths | Use When |
|------|----------|-----------|----------|
| **Stripe Invoicing** | Invoicing | Integrated with payments | Use Stripe for payments |
| **QuickBooks Online** | Accounting | SMB standard, cheap | Small business (<$1M revenue) |
| **Xero** | Accounting | Modern, API-friendly | Tech startups, SaaS |
| **NetSuite** | ERP | Enterprise, comprehensive | Large companies (>$10M revenue) |
| **Avalara / TaxJar** | Tax compliance | Auto-calculate tax | Multi-state/country sales |
| **Chargebee / Zuora** | Subscription billing | Complex billing, usage-based | High billing complexity |

**Decision Framework**:
1. Revenue size? (<$1M, $1-10M, >$10M)
2. Tax complexity? (one state, multi-state, global)
3. Billing complexity? (simple subscriptions, usage-based, tiered)
4. Accounting needs? (DIY, CPA, CFO)

**SaaS-Specific Considerations**:
- **Deferred revenue**: Can't recognize full annual payment upfront
- **Churn tracking**: MRR, ARR, customer lifetime value (LTV)
- **Metrics**: CAC, LTV:CAC ratio, burn rate
- **Proration**: Handle upgrades/downgrades mid-cycle

---

## L6: Application Layer

### 6.1 State Management

**What**: How application state is coordinated (frontend, backend, distributed).

**Why It Matters**:
- **Consistency**: Avoid stale data, race conditions
- **Performance**: Minimize re-renders, DB queries
- **Developer experience**: Predictable state updates
- **Debugging**: Time-travel, state inspection

**Frontend State Management**:

| Tool | Philosophy | Pros | Cons | Use When |
|------|------------|------|------|----------|
| **Redux** | Single global store, reducers | Predictable, devtools, mature | Verbose, boilerplate | Large apps, complex state |
| **Zustand** | Minimal hooks-based store | Simple, fast, tiny | Less ecosystem | Simple state, React |
| **Jotai / Recoil** | Atomic state, fine-grained | Minimal re-renders | Learning curve | Performance-critical |
| **Context API** | Built-in React context | No dependencies, simple | Performance issues at scale | Simple state, small apps |
| **React Query / SWR** | Server state caching | Automatic caching, refetching | Not for UI state | Fetching from APIs |
| **XState** | State machines | Predictable, visual | Complex, steep curve | Complex workflows |

**Backend State Coordination**:

| Tool | Use Case | How It Works |
|------|----------|--------------|
| **Redis** | Distributed locks, sessions | In-memory key-value store |
| **etcd / Consul** | Service discovery, config | Distributed KV with strong consistency |
| **Kafka / EventStore** | Event sourcing | Append-only event log |
| **Database transactions** | ACID consistency | Pessimistic/optimistic locking |

---

### 6.2 Business Logic Architecture

**What**: How core application logic is structured.

**Why It Matters**:
- **Maintainability**: Avoid spaghetti code
- **Testability**: Unit test without dependencies
- **Reusability**: Share logic across contexts
- **Evolvability**: Change without breaking everything

**Architectural Patterns**:

| Pattern | Principles | Pros | Cons | Use When |
|---------|-----------|------|------|----------|
| **MVC (Model-View-Controller)** | Separate data, UI, logic | Simple, familiar | Tight coupling, fat controllers | Traditional web apps |
| **Hexagonal (Ports & Adapters)** | Business logic isolated from infra | Testable, flexible | Boilerplate, learning curve | Domain-heavy apps |
| **Clean Architecture** | Dependency inversion, layers | Highly decoupled | Over-engineering risk | Complex domains |
| **Domain-Driven Design** | Ubiquitous language, bounded contexts | Models complex domains | High complexity | Large, complex domains |
| **Microservices** | Separate services, API-based | Independent scaling, polyglot | Distributed complexity | Large teams, high scale |
| **Monolith** | Single deployable unit | Simple, fast | Scaling, deploy coupling | Small teams, MVP |

**Decision Framework**:
1. Team size? (1-5 = monolith, 10+ = microservices)
2. Domain complexity? (simple CRUD, complex logic)
3. Change frequency? (daily deploys, quarterly releases)
4. Scalability needs? (100 users, 1M users)

---

### 6.3 AI/ML Integration

**What**: How AI models are integrated into application logic.

**Why It Matters**:
- **Model lifecycle**: Versioning, retraining, A/B testing
- **Inference latency**: Real-time (<100ms), batch (minutes)
- **Cost**: Model hosting ($$$), API calls ($/request)
- **Monitoring**: Model drift, accuracy, hallucinations

**Integration Patterns**:

| Pattern | Latency | Cost | Use Cases |
|---------|---------|------|-----------|
| **Cloud API (OpenAI, Anthropic)** | 1-5s | High at scale | Chat, content generation |
| **Self-hosted inference** | <1s | Infra cost | Custom models, high volume |
| **Edge inference (ONNX, TensorFlow.js)** | <100ms | Minimal | Real-time, offline-first |
| **Batch processing** | Minutes | Optimized | Non-real-time, bulk |

**Common Choices**:

| Service | Model Type | Strengths | Use When |
|---------|------------|-----------|----------|
| **OpenAI API** | GPT-4, GPT-3.5 | Best quality, easy API | Text generation, chat |
| **Anthropic (Claude)** | Claude 3 | Long context, safety | Complex reasoning, analysis |
| **Hugging Face Inference** | Open models | Cost-effective, control | Budget-conscious, custom models |
| **Modal / Replicate** | Any model | Serverless GPU, pay-per-use | Custom models, bursty traffic |
| **AWS SageMaker / Vertex AI** | Any model | Fully managed, MLOps | Enterprise, complex pipelines |

**Decision Framework**:
1. Latency tolerance? (<100ms, <5s, batch)
2. Volume? (100 calls/day, 1M calls/day)
3. Model needs? (LLM, vision, custom)
4. Budget? (cost per 1k calls)

---

## L7: Presentation Layer

### 7.1 Frontend Framework

**What**: How UI is built and rendered.

**Why It Matters**:
- **Developer productivity**: Fast iteration, great DX
- **Performance**: Initial load, interactivity (Core Web Vitals)
- **SEO**: Server-side rendering (SSR) for Google
- **Ecosystem**: Libraries, tooling, hiring

**Rendering Strategies**:

| Strategy | How | Best For | Avoid When |
|----------|-----|----------|------------|
| **CSR (Client-Side Rendering)** | JavaScript renders everything | SPAs, dashboards (logged-in) | SEO-critical, slow devices |
| **SSR (Server-Side Rendering)** | Server renders HTML on each request | SEO-critical, dynamic content | Static content |
| **SSG (Static Site Generation)** | Build-time HTML generation | Blogs, docs, marketing | Frequently changing content |
| **ISR (Incremental Static Regen)** | Static + revalidate on interval | E-commerce, CMSs | Real-time updates |

**Common Choices**:

| Framework | Ecosystem | Strengths | Use When |
|-----------|-----------|-----------|----------|
| **React + Next.js** | Largest | SSR/SSG/ISR, huge ecosystem | Most projects, Vercel hosting |
| **Vue + Nuxt** | Growing | Simpler, progressive | Prefer Vue, smaller apps |
| **Svelte + SvelteKit** | Smaller | Smallest bundles, compile-time | Performance-critical |
| **Solid + Solid Start** | Emerging | Reactive, fast | Performance-critical, early adopter |
| **Remix** | React-based | Progressive enhancement, edge | Web standards, edge deployment |

**Decision Framework**:
1. Team skills? (React most common)
2. SEO needs? (SSR/SSG essential)
3. Performance priority? (Svelte, Solid)
4. Ecosystem needs? (React largest)

---

### 7.2 Design System & Component Library

**What**: How UI consistency is maintained.

**Why It Matters**:
- **Consistency**: Same look and feel everywhere
- **Velocity**: Don't rebuild buttons 100 times
- **Accessibility**: A11y baked in (WCAG compliance)
- **Branding**: Enforce design standards

**Common Choices**:

| Library | Philosophy | Pros | Cons | Use When |
|---------|------------|------|------|----------|
| **shadcn/ui** | Copy components, own them | Full control, Tailwind-first | More maintenance | Want control, Tailwind |
| **Material UI** | Opinionated, comprehensive | Mature, complete | Large bundle, opinionated | Need full system, React |
| **Chakra UI** | Accessible, themeable | Great DX, accessible | Less flexible styling | Accessibility priority |
| **Headless UI / Radix** | Unstyled primitives | Accessible, bring your own styles | Manual styling | Custom design, accessibility |
| **Ant Design** | Enterprise-focused | Comprehensive, professional | Opinionated, large | Enterprise, admin panels |

**Decision Framework**:
1. Design flexibility? (custom design = headless)
2. Accessibility priority? (Radix, Chakra)
3. Time to market? (Material UI, Ant = fast)
4. Control vs. speed? (shadcn = control, MUI = speed)

---

## Cross-Cutting Concerns

### CC.1 Testing Strategy

**What**: How correctness is verified at all layers.

**Test Pyramid**:

```
       /\
      /E2E\      ← Few, slow, expensive (cover critical paths)
     /------\
    /Integr.\   ← Some, medium speed (test multiple units)
   /----------\
  /   Unit     \ ← Many, fast, cheap (test single units)
 /--------------\
```

**Test Types**:

| Type | Scope | Speed | Tools | Use Cases |
|------|-------|-------|-------|-----------|
| **Unit** | Single function/class | Fast (<1s) | Jest, Vitest, pytest | Business logic, pure functions |
| **Integration** | Multiple modules, DB, API | Medium (1-10s) | Jest + DB, Supertest | API endpoints, DB queries |
| **E2E** | Full user flows | Slow (10s-1min) | Playwright, Cypress | Critical paths (signup, checkout) |
| **Contract** | API contracts | Fast | Pact, Postman | Microservices, API consumers |
| **Performance** | Load, stress testing | Slow | k6, Artillery, JMeter | Scalability, SLAs |
| **Visual Regression** | UI screenshots | Medium | Percy, Chromatic | Prevent UI breakage |

**Decision Framework**:
1. Test ratio? (70% unit, 20% integration, 10% E2E)
2. Coverage target? (80% = good, 100% = overkill)
3. CI/CD integration? (run on every PR)
4. TDD or test-after? (TDD = upfront, test-after = pragmatic)

---

### CC.2 Error Handling & Resilience

**What**: How failures are handled gracefully.

**Why It Matters**:
- **Prevent cascading failures**: One service failure doesn't kill everything
- **User experience**: Show helpful errors, not "500 Internal Server Error"
- **Debugging**: Error traces, context, reproducibility

**Resilience Patterns**:

| Pattern | What | Use Cases |
|---------|------|-----------|
| **Retry (exponential backoff)** | Retry failed requests with increasing delay | Transient failures (network blip) |
| **Circuit Breaker** | Stop calling failing service after N failures | Prevent cascading failures |
| **Fallback** | Return cached/default value on failure | Degrade gracefully (show stale data) |
| **Timeout** | Fail fast after N seconds | Don't wait forever (5-30s) |
| **Bulkhead** | Isolate resources (separate thread pools) | Prevent resource exhaustion |
| **Health Checks** | Liveness, readiness probes | K8s, load balancers |

**Error Handling Best Practices**:
- **Structured logging**: Include context (user ID, request ID)
- **Error codes**: Machine-readable codes (not just messages)
- **User-friendly messages**: "Payment failed" not "500 Internal Server Error"
- **Don't leak secrets**: Sanitize error messages (no SQL queries, API keys)
- **Alerting**: Page on-call for critical errors (not warnings)

---

### CC.3 Performance Optimization

**What**: How system speed is maximized.

**Why It Matters**:
- **UX**: Faster = better conversion (Amazon: 100ms = 1% revenue)
- **Cost**: Fewer servers, lower bills
- **SEO**: Google ranks fast sites higher (Core Web Vitals)

**Optimization Layers**:

| Layer | Techniques | Impact |
|-------|------------|--------|
| **Network** | CDN, compression (gzip/Brotli), HTTP/2 | High (reduce bytes, latency) |
| **Frontend** | Code splitting, lazy loading, tree shaking | High (reduce bundle size) |
| **Backend** | Caching, DB indexes, query optimization | High (reduce latency, DB load) |
| **Database** | Indexes, connection pooling, read replicas | High (query speed) |
| **Assets** | Image optimization (WebP), lazy loading | Medium (reduce page weight) |

**Quick Wins**:
1. **Add CDN**: Cloudflare, Fastly (global edge cache)
2. **Enable compression**: Gzip/Brotli (reduce transfer size 70%)
3. **Database indexes**: Index frequently queried columns
4. **Lazy load images**: Load off-screen images on-demand
5. **Code splitting**: Load only needed JavaScript

**Monitoring**:
- **Core Web Vitals**: LCP (<2.5s), FID (<100ms), CLS (<0.1)
- **TTFB (Time To First Byte)**: <200ms (server response)
- **Bundle size**: <200KB initial JavaScript

---

### CC.4 Developer Experience

**What**: How easy it is for developers to work on the system.

**Why It Matters**:
- **Velocity**: Fast feedback = ship faster
- **Quality**: Good DX = fewer bugs (type safety, linting)
- **Hiring**: Great DX attracts top talent
- **Onboarding**: New devs productive in days, not weeks

**DX Factors**:

| Factor | What | Tools |
|--------|------|-------|
| **Fast feedback** | Hot reload, fast tests, instant deploys | Vite, Next.js Fast Refresh, Vercel |
| **Type safety** | Catch errors at compile-time | TypeScript, Zod, Prisma |
| **Linting & formatting** | Consistent code style, auto-fix | ESLint, Prettier, Biome |
| **Documentation** | README, architecture docs, SOPs | Markdown, Docusaurus, mintlify |
| **Tooling** | CLI, scripts, code generation | Custom scripts, Nx, Turborepo |
| **Local dev** | One-command setup, fast start | Docker Compose, devcontainers |

**Best Practices**:
- **README-first**: Good README = 10x faster onboarding
- **One-command setup**: `npm install && npm run dev`
- **Fast CI**: CI runs in <5 minutes (parallel, caching)
- **Clear errors**: Helpful error messages (not "undefined is not a function")

---

## Decision Framework

### Making Architectural Decisions

When evaluating a new layer or technology, ask:

1. **What problem does it solve?** (Be specific, not "it's better")
2. **What are the alternatives?** (Never just one option)
3. **What are the trade-offs?** (Pros/cons, costs, complexity)
4. **Does it align with principles?** (Your team's values)
5. **What's the exit strategy?** (Can we migrate away if needed?)
6. **Document the decision**: ADR (Architecture Decision Record)

### Red Flags 🚩

Avoid these patterns:
- ❌ "Everyone uses it" (without understanding why)
- ❌ "It's the latest trend" (hype-driven development)
- ❌ "We can always refactor later" (tech debt trap)
- ❌ "It's just a quick hack" (there's nothing more permanent than a temporary solution)
- ❌ "We need to use X because I want to learn it" (not a business reason)

### Good Defaults

When in doubt, use boring technology:
- ✅ **PostgreSQL** (not MongoDB, unless document model essential)
- ✅ **React + Next.js** (largest ecosystem, easiest hiring)
- ✅ **TypeScript** (catch errors at compile-time)
- ✅ **Vercel/Netlify** (fast deploys, low ops)
- ✅ **Stripe** (payments just work)
- ✅ **Auth0/Clerk** (don't build auth yourself)

---

## Architecture Decision Records (ADRs)

Document important decisions:

**Template**:
```markdown
# ADR-001: Use PostgreSQL as primary database

## Status
Accepted

## Context
Need database for structured data (users, orders, inventory).
Alternatives: MongoDB, MySQL, DynamoDB.

## Decision
Use PostgreSQL (managed on AWS RDS).

## Consequences
**Positive:**
- ACID transactions
- JSON support (flexible schema)
- Mature, widely adopted
- Great tooling (Prisma, pgAdmin)

**Negative:**
- Vertical scaling limits (mitigate with read replicas)
- Ops burden (mitigate with managed RDS)

## Alternatives Considered
- MongoDB: No ACID, harder to query
- MySQL: Less feature-rich than PostgreSQL
- DynamoDB: Vendor lock-in, query limitations
```

Store ADRs in `docs/architecture/decisions/`.

---

## Contributing

This is a community resource! Contributions welcome:

- **Add tools**: Found a great tool? Add it to the relevant section.
- **Update trade-offs**: Technology evolves, update outdated info.
- **Share experiences**: Add "lessons learned" from real projects.
- **Fix errors**: Found a mistake? Submit a PR.

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## License

CC0 1.0 Universal (Public Domain)

You can copy, modify, distribute this guide without asking permission.

---

## References

- [Awesome Architecture](https://github.com/simskij/awesome-software-architecture)
- [C4 Model](https://c4model.com/) - Software architecture diagrams
- [Architecture Decision Records (ADRs)](https://adr.github.io/)
- [The Twelve-Factor App](https://12factor.net/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Google Cloud Architecture Center](https://cloud.google.com/architecture)
- [Martin Fowler's Architecture Guides](https://martinfowler.com/architecture/)
- [Thoughtworks Technology Radar](https://www.thoughtworks.com/radar)

---

**Maintained by the community**  
**Star ⭐ this repo if it helped you make better architecture decisions!**

