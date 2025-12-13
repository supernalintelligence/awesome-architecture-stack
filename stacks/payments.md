# Payment Stack: Billing, Metering & Financial Operations

**End-to-End Feature Stack**: Collect money, track usage, handle subscriptions, generate invoices

---

## What Problems Does the Payment Stack Solve?

1. **Collect Payments**: Credit cards, bank transfers, invoices
2. **Recurring Billing**: Subscriptions, auto-renew, trials
3. **Usage Metering**: Track API calls, storage, compute for billing
4. **Invoicing**: Generate PDFs, email receipts, accounting
5. **Tax Compliance**: Calculate sales tax, VAT, GST across jurisdictions
6. **Revenue Operations**: Revenue recognition, refunds, disputes

---

## The Complete Payment Stack

```
┌─────────────────────────────────────────────────┐
│ L7: Presentation                                 │
│ • Checkout forms, pricing pages                 │
│ • Subscription management UI (upgrade/cancel)   │
│ • Invoice viewer, receipt download              │
│ • Usage dashboards (show consumption)           │
├─────────────────────────────────────────────────┤
│ L6: Application                                  │
│ • Payment state management (cart, checkout)    │
│ • Subscription status (active, past_due, etc)  │
│ • Usage calculations (meter → price)           │
├─────────────────────────────────────────────────┤
│ L5: Business                                     │
│ • Billing logic (subscriptions, one-time)      │
│ • Usage-based pricing calculations             │
│ • Proration (upgrades/downgrades mid-cycle)    │
│ • Refund/dispute handling                      │
├─────────────────────────────────────────────────┤
│ L4: Integration                                  │
│ • Payment processor API (Stripe, PayPal)       │
│ • Webhook handling (payment.succeeded)         │
│ • Tax calculation API (Avalara, TaxJar)        │
│ • Accounting system sync (QuickBooks, Xero)    │
├─────────────────────────────────────────────────┤
│ L3: Data                                         │
│ • Usage event validation (idempotency)         │
│ • Subscription cache (reduce API calls)        │
│ • Payment retry logic (failed payments)        │
├─────────────────────────────────────────────────┤
│ L2: Platform                                     │
│ • Usage event storage (Kafka, time-series DB)  │
│ • Subscription database (PostgreSQL)           │
│ • Invoice PDF storage (S3, CDN)                │
├─────────────────────────────────────────────────┤
│ L1: Foundation                                   │
│ • PCI DSS compliance (never store raw cards)   │
│ • Audit logging (who charged what, when)       │
│ • Secrets (API keys for payment processor)     │
└─────────────────────────────────────────────────┘
```

---

## Decision Framework: Payment Processor

### Stripe (Winner for Most Use Cases)

**When:**
- B2C or B2B SaaS
- Global customers (140+ countries)
- Need subscriptions, usage-based billing, invoicing
- Want developer-friendly API

**Pricing**:
- 2.9% + 30¢ per transaction (US)
- No monthly fee (pay as you go)

**Pros**:
- Best API, docs, developer experience
- Subscription management built-in
- Automatic tax calculation (Stripe Tax)
- Webhooks for every event
- Test mode (sandbox)

**Cons**:
- 2.9% adds up at scale (negotiate at $1M+/mo)
- Some countries not supported
- Payout delays (2-7 days)

**Best for**: 80% of startups, SaaS, B2C/B2B

---

### PayPal/Braintree

**When:**
- High consumer trust needed (PayPal brand)
- Need Venmo support (US consumer market)
- Already have PayPal business

**Pricing**:
- 2.9% + 30¢ per transaction
- Owned by PayPal (Braintree = dev-friendly PayPal)

**Pros**:
- High consumer trust
- Venmo support
- Global reach

**Cons**:
- Developer experience worse than Stripe
- Less modern API
- Account holds (risk management aggressive)

**Best for**: Consumer marketplaces, e-commerce

---

### Paddle (Merchant of Record)

**When:**
- Don't want to handle tax compliance
- Selling digital goods globally
- Small team, want turnkey solution

**Pricing**:
- 5% + 50¢ per transaction (higher than Stripe)
- Paddle handles ALL tax compliance

**Pros**:
- Paddle is merchant of record (you don't handle tax)
- Auto-calculate and remit sales tax, VAT globally
- Simpler compliance (no tax forms)

**Cons**:
- Higher fees (5% vs. 2.9%)
- Less control (Paddle owns customer relationship)
- Fewer features than Stripe

**Best for**: Digital products, small teams, global sales

---

### Lemon Squeezy (Simple SaaS Billing)

**When:**
- Simple SaaS, info products
- Solo founder, small team
- Want merchant of record (like Paddle)

**Pricing**:
- 5% + 50¢ per transaction
- Merchant of record

**Pros**:
- Super simple setup
- Merchant of record (no tax burden)
- Developer-friendly

**Cons**:
- Newer (less mature than Stripe/Paddle)
- Fewer features
- Higher fees

**Best for**: Solo founders, info products, simple SaaS

---

## Billing Models

### 1. Subscription Billing (SaaS Standard)

**How it works**: Charge fixed amount monthly/annually.

```
Starter: $10/mo (up to 1000 API calls)
Pro: $50/mo (up to 10k API calls)
Enterprise: $500/mo (unlimited)
```

**Pros**: Predictable revenue (MRR), easy to forecast  
**Cons**: Inflexible, doesn't scale with usage

**Implementation**:
- Stripe Subscriptions
- Chargebee (complex billing)
- Paddle

**Best for**: Most SaaS, B2B software

---

### 2. Usage-Based Billing

**How it works**: Charge per unit consumed (API calls, GB stored, compute hours).

```
$0.01 per API call
$0.10 per GB storage/month
$0.50 per GPU hour
```

**Pros**: Fair (pay for what you use), revenue scales with usage  
**Cons**: Unpredictable revenue, billing complexity

**Implementation**:
- Stripe Metered Billing
- Metronome (dedicated usage billing)
- AWS Cost Explorer (AWS services)

**Best for**: API platforms, cloud services, AI/ML apps

---

### 3. Hybrid (Seats + Usage)

**How it works**: Base price (per seat) + overage charges.

```
$10/user/month + $0.01 per API call over 1000
```

**Pros**: Predictable base + fair usage charges  
**Cons**: Most complex to implement

**Implementation**:
- Stripe (subscriptions + metered billing)
- Chargebee (complex billing engine)
- Zuora (enterprise billing)

**Best for**: B2B SaaS with variable usage

---

## Usage Metering Architecture

### Pattern 1: Real-Time Metering (Simple)

**How it works**: Increment counter on every API call, batch report to Stripe.

```javascript
// On API call
await redis.incr(`usage:${customerId}:${month}`);

// Batch report to Stripe (every hour)
const usage = await redis.get(`usage:${customerId}:${month}`);
await stripe.subscriptionItems.createUsageRecord(subscriptionItemId, {
  quantity: usage,
  action: 'set'
});
```

**Pros**: Simple, real-time  
**Cons**: Redis single point of failure, hard to audit

**Best for**: Low volume (<10k events/day), simple apps

---

### Pattern 2: Event Streaming (Scalable)

**How it works**: Emit events to Kafka, aggregate in batch, report to Stripe.

```
API Gateway → Kafka → Aggregator → Stripe
                  ↓
              Time-Series DB (audit)
```

**Pros**: Scalable, auditable, reliable  
**Cons**: Complex, ops burden

**Best for**: High volume (>1M events/day), billing audit requirements

---

### Pattern 3: SaaS Metering (Turnkey)

**How it works**: Use dedicated metering service (Metronome, Amberflo).

**Pros**: Turnkey, handles aggregation, reporting, de-duplication  
**Cons**: Another vendor, cost

**Best for**: Complex usage billing, don't want to build metering

---

## Handling Failed Payments

### Retry Strategy

**Stripe default**: 4 automatic retries over 3 weeks.

```
Day 0: Payment fails
Day 3: Retry 1
Day 5: Retry 2
Day 7: Retry 3
Day 9: Retry 4 (final)
Day 10: Mark subscription canceled
```

**Best practices**:
- Email user on failure (update payment method)
- Dunning emails (remind before final retry)
- Graceful degradation (read-only mode, not hard delete)

---

### Payment Recovery Flow

1. **Payment fails** → Email user immediately
2. **Day 3**: Reminder email + in-app banner
3. **Day 7**: Final warning (account will be canceled)
4. **Day 10**: Subscription canceled → read-only mode
5. **Day 30**: Data deletion warning
6. **Day 60**: Delete data (GDPR compliance)

**Implementation**: Stripe webhooks + email automation (SendGrid, Postmark)

---

## Invoice Generation

### When to Generate Invoices

- **B2C**: Rarely (send receipts via email)
- **B2B**: Always (accounting requires PDF invoices)
- **Enterprise**: Custom invoices (net 30, PO numbers)

### Tools

| Tool | Use Case | Cost |
|------|----------|------|
| **Stripe Invoicing** | Automated, email to customer | Free (with Stripe) |
| **QuickBooks Online** | Small business accounting | $30-200/mo |
| **Xero** | Modern SaaS accounting | $13-70/mo |
| **Custom (PDFKit, Puppeteer)** | Full control, branding | Dev time |

**Best practice**: Start with Stripe Invoicing, upgrade to QuickBooks/Xero when revenue >$100k/mo.

---

## Tax Compliance

### Sales Tax (US)

**Problem**: 50 states, each with different rules. Charge sales tax if you have "nexus" (physical presence, or >$100k revenue in state).

**Solutions**:
- **Stripe Tax**: Auto-calculate sales tax ($0.50 per transaction)
- **Avalara**: Enterprise tax compliance ($500+/mo)
- **TaxJar**: Mid-market ($19-99/mo)

**When to care**: Revenue >$100k in any state (nexus threshold).

---

### VAT (Europe)

**Problem**: 27 EU countries, charge VAT based on customer location (not your location).

**Solutions**:
- **Stripe Tax**: Auto-calculate VAT
- **Paddle / Lemon Squeezy**: Merchant of record (they handle VAT)

**When to care**: Selling to EU customers (>€10k/year).

---

### GST (Global)

**Problem**: Many countries have GST (Australia, India, Canada).

**Solutions**:
- **Stripe Tax**: Global coverage
- **Merchant of Record**: Paddle, Lemon Squeezy

---

## Revenue Recognition (SaaS Accounting)

### The Problem

**Wrong**: Recognize full annual subscription revenue upfront.

```
Customer pays $1200/year → Record $1200 revenue today ❌
```

**Right**: Recognize revenue monthly over 12 months.

```
Customer pays $1200/year → Record $100/mo for 12 months ✅
```

**Why**: ASC 606 (accounting standard) - recognize revenue as service is delivered.

### Solutions

| Tool | Use Case | Cost |
|------|----------|------|
| **Stripe Revenue Recognition** | Automated, built-in | Free (with Stripe Billing) |
| **Chargebee** | Complex billing | $249+/mo |
| **Zuora** | Enterprise | $2000+/mo |
| **Manual (spreadsheet)** | <$1M ARR | Free (time-consuming) |

**When to care**: Revenue >$500k/year, or raising VC (investors want clean books).

---

## Common Pitfalls

### 🚩 Storing Credit Card Numbers
**Problem**: PCI DSS violation (huge fines, security risk)  
**Solution**: NEVER store raw card numbers. Use Stripe tokens.

### 🚩 Not Validating Webhooks
**Problem**: Fake webhook calls (bypass payment)  
**Solution**: Verify webhook signature (Stripe provides secret).

### 🚩 Forgetting Idempotency
**Problem**: Duplicate charges (network retry)  
**Solution**: Use idempotency keys (Stripe built-in).

### 🚩 Not Handling Prorations
**Problem**: User upgrades mid-cycle, charged incorrectly  
**Solution**: Use Stripe proration (automatic).

### 🚩 Hard-Deleting on Failed Payment
**Problem**: User loses data, can't resubscribe  
**Solution**: Graceful degradation (read-only mode, 60-day grace).

---

## Example Stacks

### Stack 1: Simple SaaS Subscriptions
- **Payment Processor**: Stripe Billing
- **Plans**: Starter ($10/mo), Pro ($50/mo), Enterprise (custom)
- **Invoicing**: Stripe automatic invoices
- **Tax**: Stripe Tax
- **Cost**: 2.9% + 30¢ per transaction

**Perfect for**: B2C SaaS, simple pricing

---

### Stack 2: Usage-Based API Platform
- **Payment Processor**: Stripe Metered Billing
- **Metering**: Redis counters → hourly batch to Stripe
- **Plans**: $0.01 per API call (pay-as-you-go)
- **Invoicing**: Stripe monthly invoices
- **Tax**: Stripe Tax
- **Cost**: 2.9% + 30¢ per transaction

**Perfect for**: API platforms, cloud services

---

### Stack 3: B2B SaaS (Hybrid Billing)
- **Payment Processor**: Stripe + Chargebee
- **Plans**: $10/seat/month + $0.01 per API call over 1000
- **Metering**: Kafka → Aggregator → Stripe
- **Invoicing**: Stripe Invoicing
- **Tax**: Avalara (enterprise)
- **Accounting**: Xero
- **Cost**: 2.9% + Chargebee ($249/mo) + Avalara ($500/mo)

**Perfect for**: B2B SaaS, complex billing

---

### Stack 4: Global Digital Products
- **Payment Processor**: Paddle (merchant of record)
- **Plans**: One-time purchases ($49, $99, $299)
- **Tax**: Paddle handles globally
- **Invoicing**: Paddle automatic
- **Cost**: 5% + 50¢ per transaction

**Perfect for**: Info products, courses, ebooks, solo founders

---

## Decision Checklist

- [ ] B2C or B2B? (subscriptions vs. invoices)
- [ ] Pricing model? (subscription, usage-based, hybrid)
- [ ] Global sales? (VAT, sales tax complexity)
- [ ] Revenue size? (<$1M = simple, >$1M = rev rec)
- [ ] Tax burden? (merchant of record = easier)
- [ ] Team size? (small = Paddle, large = Stripe)
- [ ] Custom billing? (Chargebee, Zuora)

---

**Next Steps**:
- Review [Business Layer - Billing](../layers/05-business.md#billing-payments)
- Review [Integration Layer - Payment APIs](../layers/04-integration.md#external-integrations)
- Review [Analytics Stack](analytics.md) - Track revenue metrics

**Last Updated**: 2025-12-12  
**Contribute**: Found a better billing pattern? [Open a PR](../CONTRIBUTING.md)


