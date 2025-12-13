# Auth Stack: Identity, Authentication & Authorization

**End-to-End Feature Stack**: User identity, login, permissions, sessions

---

## What Problems Does Auth Solve?

1. **Identity**: Who is this user?
2. **Authentication**: Prove you are who you say you are
3. **Authorization**: What are you allowed to do?
4. **Session Management**: Stay logged in
5. **Multi-tenancy**: Isolate organizations/teams

---

## The Complete Auth Stack

```
┌─────────────────────────────────────────────────┐
│ L7: Presentation                                 │
│ • Login forms, signup flows                     │
│ • MFA prompts, password reset                   │
│ • Permission-based UI (show/hide features)      │
├─────────────────────────────────────────────────┤
│ L6: Application                                  │
│ • Session state management (Redux, Zustand)    │
│ • Auth context providers (React Context)        │
│ • Client-side route protection                  │
├─────────────────────────────────────────────────┤
│ L5: Business                                     │
│ • User management (CRUD users, orgs, teams)    │
│ • Authorization logic (RBAC, ABAC, ReBAC)      │
│ • Multi-tenancy models (shared/separate DBs)   │
├─────────────────────────────────────────────────┤
│ L4: Integration                                  │
│ • Auth APIs (login, logout, refresh tokens)    │
│ • OAuth flows (Google, GitHub, Microsoft)      │
│ • API authentication (JWT, API keys)           │
│ • SSO integration (SAML, OIDC)                 │
├─────────────────────────────────────────────────┤
│ L3: Data                                         │
│ • Session validation & caching (Redis)         │
│ • Permission checks (cached, policy engine)    │
│ • Token verification (JWT validation)          │
├─────────────────────────────────────────────────┤
│ L2: Platform                                     │
│ • User database (PostgreSQL, MongoDB)          │
│ • Session store (Redis, database)              │
├─────────────────────────────────────────────────┤
│ L1: Foundation                                   │
│ • Secrets management (API keys, OAuth secrets) │
│ • Audit logging (who accessed what, when)      │
│ • Credential hashing (bcrypt, argon2)          │
└─────────────────────────────────────────────────┘
```

---

## Decision Framework: Build vs. Buy

### ✅ Use Auth Provider (Auth0, Clerk, Supabase)

**When:**
- Need compliance (HIPAA, SOC2) fast
- B2B SaaS (need SSO, SAML)
- Want MFA, magic links, social login out of box
- Small team, fast launch priority

**Trade-offs:**
- **Pro**: Fast setup, compliance-ready, maintained
- **Con**: Vendor lock-in, monthly cost scales with users
- **Cost**: $0-$25/mo (free tier) → $1000s/mo at scale

**Best for**: 90% of startups, B2B SaaS, compliance-heavy

---

### ⚙️ Self-Host (Keycloak, Ory, Authentik)

**When:**
- Need full control (custom flows, white-label)
- High user volume (cost-sensitive)
- On-prem requirements (air-gapped, data residency)

**Trade-offs:**
- **Pro**: Full control, no vendor lock-in, predictable cost
- **Con**: Ops burden, security responsibility, slow setup
- **Cost**: Infrastructure only (~$100-500/mo) but high eng time

**Best for**: Large orgs, security-first, high user volume

---

### 🔨 Build Custom

**When:**
- Extremely custom requirements (can't compromise)
- Have security expertise in-house
- Auth IS your product (identity platform)

**Trade-offs:**
- **Pro**: Total flexibility, no vendor dependency
- **Con**: High eng cost, security risk if done wrong, maintenance burden
- **Cost**: 6-12 months eng time, ongoing maintenance

**Best for**: <5% of projects (you probably don't need this)

---

## Common Choices by Use Case

### B2C Consumer App

| Need | Solution | Why |
|------|----------|-----|
| Social login | Clerk, Supabase Auth, Firebase Auth | Easy Google/Apple/Facebook login |
| Magic links | Clerk, Supabase | Passwordless, better UX |
| MFA | Auth0, Clerk | Built-in, compliance-ready |
| Anonymous users | Firebase Auth | Guest users, later upgrade |

**Typical Stack**: Clerk (auth) + PostgreSQL (user data) + Redis (sessions)

---

### B2B SaaS

| Need | Solution | Why |
|------|----------|-----|
| SSO (SAML, OIDC) | Auth0, WorkOS, Keycloak | Enterprise customers demand it |
| Org/team management | Clerk Organizations, Auth0 Orgs | Multi-tenant out of box |
| RBAC | Casbin, OPA, Clerk Roles | Role-based permissions |
| Audit logs | Custom + Logs service | Compliance requirement |

**Typical Stack**: Auth0 (auth + SSO) + Casbin (RBAC) + Elasticsearch (audit logs)

---

### API Platform

| Need | Solution | Why |
|------|----------|-----|
| API keys | Custom + database | Simple, standard |
| OAuth2 | Auth0 M2M, Keycloak | Machine-to-machine |
| Rate limiting | Redis + middleware | Prevent abuse |
| Webhook auth | HMAC signatures | Verify webhook source |

**Typical Stack**: Custom API keys + Auth0 M2M (OAuth2) + Redis (rate limits)

---

### Enterprise On-Prem

| Need | Solution | Why |
|------|----------|-----|
| LDAP/AD integration | Keycloak, Authentik | Existing user directory |
| Air-gapped deployment | Keycloak (self-hosted) | No cloud dependencies |
| FIPS compliance | Keycloak + FIPS modules | Government contracts |
| Custom branding | Keycloak themes | White-label |

**Typical Stack**: Keycloak + PostgreSQL + custom themes

---

## Authentication Methods

### 1. Password-Based

**How it works**: User enters email + password, server verifies hash.

| Approach | Pros | Cons | Use When |
|----------|------|------|----------|
| **Bcrypt** | Industry standard, slow (good) | CPU-intensive | Most apps |
| **Argon2** | Newer, more secure | Less tooling | High-security apps |
| **PBKDF2** | FIPS-compliant | Older, less secure | Government/compliance |

**Best practices**:
- Never store plaintext passwords
- Use password strength requirements (zxcvbn)
- Implement rate limiting (prevent brute force)
- Hash + salt (bcrypt does this automatically)

---

### 2. OAuth / Social Login

**How it works**: Delegate to Google/GitHub/etc., receive user token.

| Provider | Use Cases | Trade-offs |
|----------|-----------|------------|
| **Google** | Consumer apps, workspace apps | Ubiquitous, trusted |
| **GitHub** | Developer tools, technical products | Developer audience |
| **Microsoft** | Enterprise, Office integration | B2B, enterprise SSO |
| **Apple** | iOS apps (required for "Sign in with Apple") | Privacy-focused users |

**Implementation**: Use Auth0, Clerk, or NextAuth.js (don't build OAuth flows yourself).

---

### 3. Magic Links (Passwordless)

**How it works**: Send email with one-time link, user clicks, auto-login.

**Pros**:
- No passwords to forget
- Better security (no password reuse)
- Faster signup flow

**Cons**:
- Email deliverability issues
- Slower than password (wait for email)
- Requires functional email

**Use when**: Consumer apps, B2C, quick signup priority

---

### 4. Multi-Factor Authentication (MFA)

**How it works**: Something you know (password) + something you have (phone, device).

| Method | Security | UX | Use Cases |
|--------|----------|-----|-----------|
| **SMS codes** | Low (SIM swap attacks) | Easy | Consumer apps (better than nothing) |
| **TOTP (Google Authenticator)** | High | Moderate | B2B, security-conscious users |
| **WebAuthn (passkeys)** | Highest | Best (biometric) | Modern apps, iOS/Android support |
| **Push notifications** | High | Easy | Mobile apps |

**Best practice**: Offer multiple MFA methods, make it optional for B2C, required for B2B admins.

---

## Session Management

### JWT (JSON Web Tokens)

**How it works**: Server signs token, client stores it, server verifies signature.

**Pros**:
- Stateless (no session store needed)
- Easy to scale (no shared state)
- Good for microservices

**Cons**:
- Can't revoke (until expiry)
- Token theft (XSS risk if stored in localStorage)
- Larger payload (vs. session ID)

**Best practices**:
- Short expiry (15 min access token)
- Refresh tokens (7 days, stored in httpOnly cookie)
- Store in httpOnly, secure, sameSite cookies (NOT localStorage)

---

### Session Cookies (Server-Side)

**How it works**: Server stores session in database/Redis, client gets session ID cookie.

**Pros**:
- Can revoke instantly (delete from store)
- Smaller cookies (just session ID)
- More control (logout all devices)

**Cons**:
- Requires session store (Redis, DB)
- Harder to scale (shared state)
- Database load (lookup on every request)

**Best practices**:
- Use Redis for session store (fast lookups)
- Set reasonable TTL (7-30 days)
- Implement "remember me" (longer TTL)

---

## Authorization Models

### Role-Based Access Control (RBAC)

**How it works**: Users have roles (Admin, Editor, Viewer), roles have permissions.

**Example**:
```
Admin: can(create, update, delete, read)
Editor: can(update, read)
Viewer: can(read)
```

**Pros**: Simple, easy to understand, works for most apps  
**Cons**: Inflexible (what if Editor can only edit THEIR posts?)

**Use when**: <10 roles, simple permissions, B2B SaaS

---

### Attribute-Based Access Control (ABAC)

**How it works**: Rules based on attributes (user.dept === post.dept).

**Example**:
```javascript
allow if (
  user.department === resource.department &&
  resource.status === "draft"
)
```

**Pros**: Flexible, dynamic rules, context-aware  
**Cons**: Complex to implement, harder to debug

**Use when**: Complex permissions, context-dependent access

---

### Relationship-Based (ReBAC)

**How it works**: Permissions based on relationships (owner, collaborator, viewer).

**Example**: Google Docs - owner can share, editors can edit, viewers can read.

**Pros**: Natural for social/collaborative apps  
**Cons**: Requires graph queries, complex

**Use when**: Collaborative apps, social networks, Google Docs-style sharing

**Tools**: Ory Permissions, Zanzibar (Google's model), Authzed

---

## Multi-Tenancy Models

### Model 1: Shared DB, Shared Schema (tenant_id column)

```sql
users:
  id, email, name, tenant_id

posts:
  id, title, content, tenant_id
```

**Pros**: Simple, cost-effective, easy to scale  
**Cons**: Risk of data leaks (query without tenant_id), noisy neighbors

**Use when**: Small tenants (<1000 users/tenant), SaaS MVP

---

### Model 2: Shared DB, Separate Schemas

```sql
tenant_1:
  users, posts, orders

tenant_2:
  users, posts, orders
```

**Pros**: Better isolation, still cost-effective  
**Cons**: Schema migrations complex, query complexity

**Use when**: Medium tenants, PostgreSQL (schemas built-in)

---

### Model 3: Separate Databases

```
tenant_1_db → users, posts, orders
tenant_2_db → users, posts, orders
```

**Pros**: Complete isolation, compliance-friendly, per-tenant backups  
**Cons**: Ops complexity, cost (one DB per tenant), scaling hard

**Use when**: Large tenants (>10k users), compliance (HIPAA), enterprise

---

## Common Pitfalls

### 🚩 Storing Tokens in localStorage
**Problem**: XSS attacks can steal tokens  
**Solution**: Use httpOnly, secure, sameSite cookies

### 🚩 Long-Lived Access Tokens
**Problem**: Can't revoke, security risk  
**Solution**: Short access tokens (15 min) + refresh tokens

### 🚩 Forgetting tenant_id in Queries
**Problem**: Data leaks between tenants  
**Solution**: Use Row-Level Security (PostgreSQL RLS) or ORM middleware

### 🚩 Not Rate Limiting Login
**Problem**: Brute force attacks  
**Solution**: Rate limit by IP + email (Redis)

### 🚩 Exposing User Enumeration
**Problem**: "Email not found" vs. "Wrong password"  
**Solution**: Generic error "Invalid credentials"

---

## Example Stacks

### Stack 1: B2C SaaS (Simple)
- **Auth**: Clerk (social login, MFA, email/password)
- **Database**: PostgreSQL (user profiles, app data)
- **Sessions**: Clerk (built-in session management)
- **Cost**: $25/mo (up to 10k users)

### Stack 2: B2B SaaS (Enterprise)
- **Auth**: Auth0 (SSO, SAML, OIDC)
- **Authorization**: Casbin (RBAC)
- **Database**: PostgreSQL (shared DB, tenant_id)
- **Sessions**: Redis (server-side sessions)
- **Audit**: Elasticsearch (compliance logs)
- **Cost**: $500-2000/mo

### Stack 3: API Platform
- **Auth**: Custom API keys + Auth0 M2M (OAuth2)
- **Rate Limiting**: Redis + middleware
- **Database**: PostgreSQL (API keys, usage)
- **Cost**: $100-500/mo

### Stack 4: Self-Hosted (Enterprise)
- **Auth**: Keycloak (LDAP, SAML, custom themes)
- **Database**: PostgreSQL
- **Sessions**: Redis
- **Cost**: $200/mo (infra) + eng time

---

## Decision Checklist

- [ ] B2C or B2B? (social login vs. SSO)
- [ ] Compliance needs? (HIPAA, SOC2, GDPR)
- [ ] Multi-tenant? (shared vs. separate DBs)
- [ ] MFA required? (enterprise = yes)
- [ ] Budget? (Auth0 = $$$, self-host = eng time)
- [ ] Team size? (small = buy, large = build)
- [ ] Custom requirements? (most can use Clerk/Auth0)

---

**Next Steps**:
- Review [Business Layer - User Management](../layers/05-business.md#user-management)
- Review [Integration Layer - API Auth](../layers/04-integration.md#api-security)
- Review [Security Cross-Cutting Concern](../concerns/security.md)

**Last Updated**: 2025-12-12  
**Contribute**: Found a better auth pattern? [Open a PR](../CONTRIBUTING.md)

