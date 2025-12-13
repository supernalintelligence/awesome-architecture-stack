# Data Stack: Storage, Caching & Data Management

**End-to-End Feature Stack**: Databases, caching, validation, queries, data pipelines

---

## System Features vs. User Features

### System/Infrastructure Features
- Database choice & schema design
- Query optimization & indexing
- Caching layers (Redis, CDN)
- Data validation & sanitization
- Backup & disaster recovery
- Data migrations & versioning

### User-Facing Features
- CRUD operations (create, read, update, delete)
- Search & filtering
- Data export (CSV, JSON, PDF)
- Bulk operations (import, delete)
- Data privacy controls (GDPR, delete my data)
- Audit history (who changed what, when)

---

## The Complete Data Stack

```
┌─────────────────────────────────────────────────┐
│ L7: Presentation                                 │
│ • Forms (create, edit)                          │
│ • Tables (list, sort, filter, paginate)        │
│ • Search UI (autocomplete, filters)            │
│ • Export buttons (download CSV, PDF)           │
├─────────────────────────────────────────────────┤
│ L6: Application                                  │
│ • Form state management                         │
│ • Optimistic updates (instant UI)              │
│ • Client-side validation                        │
│ • Pagination & infinite scroll                  │
├─────────────────────────────────────────────────┤
│ L5: Business                                     │
│ • Business rules (constraints, calculations)   │
│ • Data access control (who can see what)       │
│ • Audit logging (change history)               │
│ • Data retention policies                       │
├─────────────────────────────────────────────────┤
│ L4: Integration                                  │
│ • REST/GraphQL API (CRUD endpoints)            │
│ • Bulk import API (CSV upload)                 │
│ • Export API (generate files)                  │
│ • Webhooks (notify on changes)                 │
├─────────────────────────────────────────────────┤
│ L3: Data                                         │
│ • Query optimization (indexes, explain)        │
│ • Caching (Redis, in-memory)                   │
│ • Validation (Zod, JSON Schema)                │
│ • Pagination (cursor, offset)                   │
├─────────────────────────────────────────────────┤
│ L2: Platform                                     │
│ • Primary database (PostgreSQL, MongoDB)       │
│ • Cache layer (Redis, Memcached)              │
│ • Search engine (Elasticsearch, Algolia)       │
│ • Backup storage (S3, snapshots)               │
├─────────────────────────────────────────────────┤
│ L1: Foundation                                   │
│ • Connection pooling                            │
│ • Monitoring (slow queries, errors)            │
│ • Encryption at rest (database encryption)     │
│ • Disaster recovery (backups, replication)     │
└─────────────────────────────────────────────────┘
```

---

## Decision Framework: Database Choice

### SQL vs. NoSQL Decision Tree

```
START
  ↓
Do you need complex joins? (orders + users + products)
  YES → SQL (PostgreSQL, MySQL)
  NO → Continue
    ↓
  Is your schema fixed and well-defined?
    YES → SQL (easier to enforce constraints)
    NO → Continue
      ↓
    Need horizontal scaling (billions of records)?
      YES → NoSQL (Cassandra, DynamoDB)
      NO → SQL (PostgreSQL with read replicas)
```

---

### Relational Databases (SQL)

| Database | Best For | Pros | Cons | Cost |
|----------|----------|------|------|------|
| **PostgreSQL** | Most use cases | ACID, JSON support, mature | Vertical scaling limits | Free (self-hosted), $25-500/mo (managed) |
| **MySQL** | WordPress, legacy apps | Popular, simple | Less features than Postgres | Free (self-hosted), $15-400/mo (managed) |
| **SQLite** | Small apps, embedded, mobile | Simple, file-based, no server | Not for multi-user | Free |
| **CockroachDB** | Global scale, multi-region | Distributed SQL, ACID | Complex, expensive | $295+/mo |

**Use when**: 
- Need ACID transactions (banking, e-commerce)
- Complex joins (orders + users + products)
- Well-defined schema
- <10M records

---

### NoSQL Databases

| Database | Type | Best For | Pros | Cons |
|----------|------|----------|------|------|
| **MongoDB** | Document | Flexible schema, rapid iteration | Easy to start, horizontal scale | No ACID (older versions), expensive |
| **DynamoDB** | Key-Value | AWS-native, infinite scale | Managed, fast | Expensive, query limitations, vendor lock |
| **Cassandra** | Wide-Column | Time-series, write-heavy | Horizontal scale, HA | Complex, eventual consistency |
| **Redis** | Key-Value | Caching, sessions, queues | Blazing fast | In-memory (volatile), not for primary DB |

**Use when**:
- Flexible schema (rapidly changing requirements)
- Horizontal scaling needed (>10M records)
- Simple queries (key lookups, no joins)
- High write throughput

---

## Database Design Patterns

### Pattern 1: Normalized (SQL Standard)

```sql
-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Orders table (references users)
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  total DECIMAL(10,2) NOT NULL,
  status TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Order items table (references orders + products)
CREATE TABLE order_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID NOT NULL REFERENCES orders(id),
  product_id UUID NOT NULL REFERENCES products(id),
  quantity INT NOT NULL,
  price DECIMAL(10,2) NOT NULL
);
```

**Pros**: No data duplication, easy to update  
**Cons**: Requires joins (slower for reads)

---

### Pattern 2: Denormalized (NoSQL Style)

```javascript
// MongoDB: Embed order items in order document
{
  _id: ObjectId("..."),
  user: {
    id: "user-123",
    email: "alice@example.com",
    name: "Alice"
  },
  items: [
    { productId: "prod-1", name: "Laptop", quantity: 1, price: 1200 },
    { productId: "prod-2", name: "Mouse", quantity: 2, price: 25 }
  ],
  total: 1250,
  status: "shipped",
  createdAt: ISODate("2025-12-12")
}
```

**Pros**: Fast reads (no joins), single query  
**Cons**: Data duplication, harder to update (if user changes name, update everywhere)

**When to denormalize**: Read-heavy, data rarely changes (e.g., order history)

---

## Caching Strategy

### Cache Patterns

#### 1. Cache-Aside (Lazy Loading)

```javascript
async function getUser(userId) {
  // 1. Check cache first
  const cached = await redis.get(`user:${userId}`);
  if (cached) return JSON.parse(cached);
  
  // 2. Cache miss → fetch from DB
  const user = await db.query('SELECT * FROM users WHERE id = $1', [userId]);
  
  // 3. Store in cache (TTL: 1 hour)
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user));
  
  return user;
}
```

**Pros**: Only cache what's accessed  
**Cons**: Cache misses are slow (DB query)

---

#### 2. Write-Through Cache

```javascript
async function updateUser(userId, updates) {
  // 1. Update database
  const user = await db.query(
    'UPDATE users SET name = $1 WHERE id = $2 RETURNING *',
    [updates.name, userId]
  );
  
  // 2. Update cache immediately
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user));
  
  return user;
}
```

**Pros**: Cache always fresh  
**Cons**: Slower writes (two operations)

---

#### 3. Write-Behind Cache (Async)

```javascript
async function updateUser(userId, updates) {
  // 1. Update cache immediately
  const user = { id: userId, ...updates };
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user));
  
  // 2. Queue DB write (async)
  await queue.publish('user_updates', { userId, updates });
  
  return user;
}
```

**Pros**: Fast writes (async DB update)  
**Cons**: Risk of data loss (if cache dies before DB write)

---

### Cache Invalidation

**Problem**: How to keep cache fresh when data changes?

**Solutions**:

1. **TTL (Time To Live)**: Auto-expire after N seconds
   ```javascript
   await redis.setex('user:123', 3600, data); // Expire in 1 hour
   ```

2. **Manual Invalidation**: Delete cache on update
   ```javascript
   await redis.del('user:123'); // Force re-fetch from DB
   ```

3. **Pub/Sub Invalidation**: Notify all servers
   ```javascript
   await redis.publish('cache_invalidate', 'user:123');
   ```

---

## Query Optimization

### Indexes (Essential)

```sql
-- Without index: Full table scan (slow for millions of rows)
SELECT * FROM users WHERE email = 'alice@example.com';

-- With index: Fast lookup (microseconds)
CREATE INDEX idx_users_email ON users(email);

-- Composite index (multi-column queries)
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
```

**When to index**:
- Primary keys (automatic)
- Foreign keys (references)
- Frequently queried columns (email, username)
- Sort/filter columns (created_at, price)

**When NOT to index**:
- Rarely queried columns
- High-write tables (indexes slow down writes)

---

### Pagination Patterns

#### Offset Pagination (Simple)

```sql
-- Page 1 (first 20)
SELECT * FROM posts ORDER BY created_at DESC LIMIT 20 OFFSET 0;

-- Page 2 (next 20)
SELECT * FROM posts ORDER BY created_at DESC LIMIT 20 OFFSET 20;
```

**Pros**: Easy to implement, jump to any page  
**Cons**: Slow for large offsets (OFFSET 10000 = scan 10k rows)

---

#### Cursor Pagination (Fast)

```sql
-- First page
SELECT * FROM posts ORDER BY created_at DESC LIMIT 20;

-- Next page (cursor = last post's created_at)
SELECT * FROM posts 
WHERE created_at < '2025-12-12T10:00:00Z'
ORDER BY created_at DESC 
LIMIT 20;
```

**Pros**: Fast (uses index), consistent for real-time data  
**Cons**: Can't jump to arbitrary page

---

## Data Validation

### Schema Validation (Zod Example)

```typescript
import { z } from 'zod';

// Define schema
const UserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2).max(100),
  age: z.number().int().min(18).max(120),
  role: z.enum(['user', 'admin', 'moderator'])
});

// Validate input
try {
  const user = UserSchema.parse(req.body);
  // ✅ Valid → save to DB
  await db.insert('users', user);
} catch (error) {
  // ❌ Invalid → return errors
  res.status(400).json({ errors: error.errors });
}
```

---

### Database Constraints

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,                    -- Must be unique
  age INT CHECK (age >= 18 AND age <= 120),      -- Age range
  role TEXT CHECK (role IN ('user', 'admin')),   -- Enum
  created_at TIMESTAMP DEFAULT NOW() NOT NULL
);
```

**Layers of validation**:
1. **Client-side** (instant feedback, can be bypassed)
2. **API layer** (Zod, Joi - catch bad requests)
3. **Database** (constraints - last line of defense)

---

## Backup & Disaster Recovery

### Backup Strategies

| Strategy | RPO (Data Loss) | RTO (Downtime) | Cost | Use When |
|----------|-----------------|----------------|------|----------|
| **Daily snapshots** | 24 hours | Hours | Low | Non-critical data |
| **Hourly snapshots** | 1 hour | 30 min | Medium | Important data |
| **Continuous backup (WAL)** | Seconds | Minutes | High | Critical (banking, healthcare) |
| **Read replicas** | Real-time | Instant | High | Mission-critical |

---

### PostgreSQL Backup Example

```bash
# Full database dump (manual)
pg_dump -U postgres myapp > backup.sql

# Restore
psql -U postgres myapp < backup.sql

# Automated daily backups (cron)
0 2 * * * pg_dump -U postgres myapp | gzip > /backups/myapp-$(date +\%Y\%m\%d).sql.gz

# Point-in-time recovery (WAL archiving)
# Enable in postgresql.conf:
# wal_level = replica
# archive_mode = on
# archive_command = 'cp %p /archive/%f'
```

---

## Data Migrations

### Schema Migration Pattern

```javascript
// Migration: 001_create_users.js
export async function up(db) {
  await db.query(`
    CREATE TABLE users (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      email TEXT UNIQUE NOT NULL,
      name TEXT NOT NULL,
      created_at TIMESTAMP DEFAULT NOW()
    );
  `);
}

export async function down(db) {
  await db.query('DROP TABLE users;');
}
```

**Tools**:
- **Node.js**: Knex.js, Sequelize, Prisma Migrate
- **Python**: Alembic, Django migrations
- **Go**: golang-migrate
- **Ruby**: Rails migrations

---

### Zero-Downtime Migration Pattern

**Problem**: Adding a column requires table lock (downtime).

**Solution**: Multi-step migration.

```sql
-- Step 1: Add column (nullable, no default)
ALTER TABLE users ADD COLUMN phone TEXT;

-- Step 2: Backfill data (batched, no lock)
UPDATE users SET phone = '+1234567890' WHERE phone IS NULL LIMIT 1000;
-- Repeat until all rows updated

-- Step 3: Add constraint (after all data backfilled)
ALTER TABLE users ALTER COLUMN phone SET NOT NULL;
```

---

## Common Pitfalls

### 🚩 No Indexes on Foreign Keys
**Problem**: Slow joins  
**Solution**: Index all foreign keys

### 🚩 SELECT * (Select All Columns)
**Problem**: Fetch unnecessary data (slow, large payloads)  
**Solution**: `SELECT id, name, email` (only needed columns)

### 🚩 N+1 Query Problem
**Problem**: Fetch users, then fetch each user's orders (1 + N queries)

```javascript
// ❌ Bad: N+1 queries
const users = await db.query('SELECT * FROM users');
for (const user of users) {
  user.orders = await db.query('SELECT * FROM orders WHERE user_id = $1', [user.id]);
}

// ✅ Good: 2 queries
const users = await db.query('SELECT * FROM users');
const orders = await db.query('SELECT * FROM orders WHERE user_id = ANY($1)', [users.map(u => u.id)]);
```

### 🚩 No Connection Pooling
**Problem**: Opening new DB connection per request (slow)  
**Solution**: Connection pool (reuse connections)

```javascript
const { Pool } = require('pg');
const pool = new Pool({ max: 20 }); // 20 connections
```

---

## Example Stacks

### Stack 1: Simple CRUD App
- **Database**: PostgreSQL (Heroku Postgres, $9-50/mo)
- **Cache**: Redis (Heroku Redis, $15/mo)
- **API**: Node.js + Express
- **ORM**: Prisma
- **Validation**: Zod

**Perfect for**: SaaS MVP, <100k users

---

### Stack 2: High-Traffic App
- **Database**: PostgreSQL (AWS RDS Multi-AZ, $100-500/mo)
- **Read Replicas**: 2x read replicas (scale reads)
- **Cache**: Redis Cluster (AWS ElastiCache, $50-200/mo)
- **Search**: Elasticsearch (AWS OpenSearch, $100/mo)
- **Backup**: Daily snapshots + WAL archiving

**Perfect for**: E-commerce, >1M users

---

### Stack 3: Global Scale
- **Database**: CockroachDB (multi-region, $500+/mo)
- **Cache**: Redis cluster per region
- **CDN**: Cloudflare (cache static data at edge)
- **Search**: Algolia (managed search)

**Perfect for**: Global SaaS, >10M users

---

### Stack 4: Serverless
- **Database**: PlanetScale (serverless MySQL, $29-399/mo)
- **Cache**: Upstash Redis (serverless Redis, $0-10/mo)
- **API**: Vercel Functions (serverless Node.js)
- **ORM**: Prisma

**Perfect for**: Bursty traffic, pay-per-use

---

## Decision Checklist

- [ ] SQL or NoSQL? (joins = SQL, flexible schema = NoSQL)
- [ ] Data volume? (<10M = single DB, >10M = read replicas)
- [ ] Query patterns? (complex joins = SQL, key lookups = NoSQL)
- [ ] Budget? (managed = easy, self-hosted = cheaper)
- [ ] Backup needs? (daily, hourly, continuous)
- [ ] Global? (multi-region replication)
- [ ] Compliance? (GDPR, HIPAA)

---

**Next Steps**:
- Review [Integration Layer - APIs](../layers/04-integration.md)
- Review [Content Stack](content.md) - For file storage
- Review [Analytics Stack](analytics.md) - For data warehousing

**Last Updated**: 2025-12-12  
**Contribute**: Found a better data pattern? [Open a PR](../CONTRIBUTING.md)

