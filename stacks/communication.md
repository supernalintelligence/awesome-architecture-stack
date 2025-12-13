# Communication Stack: Real-Time, Messaging & Notifications

**End-to-End Feature Stack**: User-to-user messaging, notifications, real-time updates

---

## System Features vs. User Features

### System/Infrastructure Features (What Developers Build)
- WebSocket server infrastructure
- Message queue (Redis, Kafka)
- Push notification service (FCM, APNS)
- Presence system (who's online)
- Message persistence & sync
- Rate limiting, spam detection

### User-Facing Features (What Users Experience)
- Chat interfaces (1-on-1, group, channels)
- Notifications (in-app, push, email, SMS)
- Typing indicators, read receipts
- File sharing in chat
- Emoji reactions, threads
- Notification preferences

---

## The Complete Communication Stack

```
┌─────────────────────────────────────────────────┐
│ L7: Presentation (User-Facing)                   │
│ • Chat UI (messages, input, threads)            │
│ • Notification center (unread badge, dropdown)  │
│ • Typing indicators, online status              │
│ • Emoji picker, reactions, file upload          │
├─────────────────────────────────────────────────┤
│ L6: Application (Business Logic)                │
│ • Message state management (Redux, Zustand)    │
│ • Optimistic updates (instant UI, sync later)  │
│ • Notification queue (dedupe, prioritize)      │
│ • Presence tracking (online/offline/away)      │
├─────────────────────────────────────────────────┤
│ L5: Business (Domain Logic)                     │
│ • Channel permissions (who can post)           │
│ • Notification rules (mute, priority)          │
│ • Moderation (spam filter, content policy)     │
│ • Retention policy (delete old messages)       │
├─────────────────────────────────────────────────┤
│ L4: Integration (APIs & Protocols)              │
│ • WebSocket API (bi-directional messages)      │
│ • REST API (fetch history, settings)           │
│ • Push notification API (FCM, APNS)            │
│ • Email gateway (SendGrid, Postmark)           │
│ • SMS gateway (Twilio)                         │
├─────────────────────────────────────────────────┤
│ L3: Data (Caching & Validation)                 │
│ • Message cache (Redis - recent messages)      │
│ • Presence cache (Redis - who's online)        │
│ • Read receipt tracking (cache before persist) │
│ • Message deduplication (idempotency)          │
├─────────────────────────────────────────────────┤
│ L2: Platform (Infrastructure)                   │
│ • Message database (PostgreSQL, MongoDB)       │
│ • Message queue (Kafka, RabbitMQ)             │
│ • File storage (S3 for attachments)           │
│ • Push notification service (FCM, APNS)        │
├─────────────────────────────────────────────────┤
│ L1: Foundation (Core Infrastructure)            │
│ • WebSocket load balancing (sticky sessions)  │
│ • Audit logging (who said what, when)         │
│ • Rate limiting (prevent spam, DoS)           │
│ • Encryption (TLS, E2E for sensitive apps)    │
└─────────────────────────────────────────────────┘
```

---

## Decision Framework: Chat/Messaging

### SaaS Solutions (Don't Build Chat Yourself)

| Provider | Best For | Pricing | Pros | Cons |
|----------|----------|---------|------|------|
| **Stream** | Full-featured chat, social apps | $0.50/MAU | Feature-rich, SDKs for all platforms | Expensive at scale |
| **PubNub** | Real-time messaging, IoT | $49-499/mo | Global edge, low latency | Complex pricing |
| **Ably** | Real-time infrastructure | $0/mo (free tier) → $$ | Reliable, good DX | Chat not primary focus |
| **Sendbird** | In-app messaging, support chat | $499+/mo | Enterprise-grade | Expensive |
| **Firebase (Firestore)** | Simple chat, mobile apps | Pay-as-you-go | Easy setup, mobile SDKs | Not built for chat (need schema design) |

**Recommendation**: Use Stream or Sendbird for 95% of apps. Only build custom if chat IS your product.

---

### Build Custom Chat (When You Must)

**When to build:**
- Chat is your core product (Slack competitor)
- Extreme customization needs (proprietary features)
- Cost at massive scale (1M+ DAU, cheaper to self-host)
- Security requirements (government, healthcare, no SaaS)

**Stack for custom chat:**
```
Frontend: React + Socket.IO client
Backend: Node.js + Socket.IO server
Database: PostgreSQL (messages) + Redis (presence, cache)
Queue: Redis Pub/Sub (room broadcasts)
Storage: S3 (file uploads)
```

**Estimated build time**: 3-6 months (for basic Slack clone)

---

## Real-Time Protocols

### WebSockets (Most Common)

**How it works**: Persistent bi-directional connection between client and server.

**Use cases**:
- Chat (messages flow both ways)
- Collaborative editing (Google Docs-style)
- Gaming (real-time multiplayer)
- Live dashboards (metrics update live)

**Tools**:
- **Socket.IO** (Node.js) - Auto-reconnect, room support, fallbacks
- **Native WebSocket** (standard) - Lightweight, manual reconnect
- **WS** (Node.js library) - Low-level WebSocket server

**Scaling challenges**:
- Sticky sessions (user must hit same server)
- Redis adapter (broadcast across servers)
- Load balancing (WebSocket-aware LB needed)

**Example (Socket.IO)**:
```javascript
// Server
io.on('connection', (socket) => {
  socket.join(`user:${userId}`); // Join room
  socket.on('message', (msg) => {
    io.to(`channel:${channelId}`).emit('message', msg); // Broadcast
  });
});

// Client
socket.emit('message', { text: 'Hello', channelId: '123' });
socket.on('message', (msg) => console.log(msg));
```

---

### Server-Sent Events (SSE)

**How it works**: Server pushes updates to client over HTTP (one-way only).

**Use cases**:
- Live notifications (new message, mention)
- Live dashboards (server → client updates)
- Stock tickers, sports scores

**Pros**:
- Simple (just HTTP, no special protocol)
- Auto-reconnect built-in
- Easy to implement

**Cons**:
- Server → client only (no client → server messages)
- Browser limit (6 connections per domain)

**Example**:
```javascript
// Server (Node.js)
res.writeHead(200, {
  'Content-Type': 'text/event-stream',
  'Cache-Control': 'no-cache'
});
setInterval(() => {
  res.write(`data: ${JSON.stringify({ count: 42 })}\n\n`);
}, 1000);

// Client
const evtSource = new EventSource('/notifications');
evtSource.onmessage = (e) => console.log(JSON.parse(e.data));
```

**Use when**: Notifications, live updates (no need for bi-directional)

---

### Long Polling (Fallback)

**How it works**: Client sends request, server holds it open until data available, then responds. Client immediately sends new request.

**Pros**: Works on any HTTP connection  
**Cons**: Inefficient (constant requests), high latency

**Use when**: Fallback for ancient browsers (IE8), or WebSocket blocked by firewall

---

## Push Notifications

### Mobile Push (iOS & Android)

**How it works**: Server sends notification to FCM (Android) or APNS (iOS), which delivers to device.

**Services**:
- **Firebase Cloud Messaging (FCM)**: Android + iOS (Google's service)
- **Apple Push Notification Service (APNS)**: iOS only
- **OneSignal**: Unified push (FCM + APNS), free tier
- **Pusher Beams**: Push notifications (paid)

**Implementation**:
```javascript
// Server (send notification via FCM)
await admin.messaging().send({
  token: deviceToken,
  notification: {
    title: 'New message from Alice',
    body: 'Hey, are you free tomorrow?'
  },
  data: {
    channelId: '123',
    messageId: '456'
  }
});

// Client (React Native)
messaging().onMessage(async remoteMessage => {
  Alert.alert('New message', remoteMessage.notification.body);
});
```

**Key concepts**:
- **Device token**: Unique ID for each device
- **Topics**: Send to groups (e.g., "all_users", "channel_123")
- **Data payload**: Custom data (open specific screen)
- **Silent notifications**: Background updates (no user alert)

---

### Web Push (Browser)

**How it works**: Browser receives push even when tab closed (Service Worker).

**Browser support**: Chrome, Firefox, Edge (not Safari on iOS)

**Implementation**:
```javascript
// Client (register for push)
const registration = await navigator.serviceWorker.register('/sw.js');
const subscription = await registration.pushManager.subscribe({
  userVisibleOnly: true,
  applicationServerKey: vapidPublicKey
});

// Send subscription to server
await fetch('/api/subscribe', {
  method: 'POST',
  body: JSON.stringify(subscription)
});

// Server (send push)
await webpush.sendNotification(subscription, JSON.stringify({
  title: 'New message',
  body: 'Alice sent you a message'
}));
```

**Use cases**: Web app notifications (Gmail, Slack web)

---

### Email Notifications

**When to use**: Non-urgent updates, digests, user preference.

**Services**:
- **SendGrid**: Transactional email ($0.0001 per email)
- **Postmark**: Transactional email (better deliverability)
- **AWS SES**: Cheapest ($0.10 per 1000 emails)
- **Mailgun**: Transactional + marketing

**Best practices**:
- Let users control email frequency (instant, daily digest, off)
- Unsubscribe link (CAN-SPAM compliance)
- Plain text + HTML versions
- Track opens/clicks (optional)

---

### SMS Notifications

**When to use**: Urgent alerts, 2FA codes, delivery updates.

**Services**:
- **Twilio**: $0.0075 per SMS (US)
- **AWS SNS**: $0.00645 per SMS (US)
- **Vonage (Nexmo)**: $0.0052 per SMS (US)

**Cost warning**: SMS is expensive at scale. Use sparingly.

---

## Notification Preferences

### User Control

Users should control:
- **Channels**: In-app, push, email, SMS
- **Frequency**: Instant, hourly digest, daily digest, off
- **Types**: Mentions, messages, system updates

**Example preferences**:
```javascript
notificationSettings: {
  mentions: { inApp: true, push: true, email: true, sms: false },
  directMessages: { inApp: true, push: true, email: false, sms: false },
  channelMessages: { inApp: true, push: false, email: false, sms: false },
  systemUpdates: { inApp: true, push: false, email: true, sms: false }
}
```

### Smart Defaults

**Best practices**:
- Default to in-app only (least invasive)
- Ask permission for push notifications (don't auto-prompt)
- Auto-disable email if user opens in-app immediately
- Mute notifications when user is active in app

---

## Presence System (Who's Online)

### Implementation Pattern

**Challenge**: Track millions of users efficiently.

**Solution**: Redis + heartbeat pattern.

```javascript
// Client sends heartbeat every 30s
setInterval(async () => {
  await fetch('/api/presence', { method: 'POST' });
}, 30000);

// Server updates Redis with TTL
await redis.setex(`presence:${userId}`, 60, 'online');

// Check if user online
const isOnline = await redis.exists(`presence:${userId}`);
```

**Status levels**:
- **Online**: Active in last 60s
- **Away**: Active in last 10 min, but >60s ago
- **Offline**: >10 min since last activity

**Scaling**: Use Redis cluster for millions of users.

---

## Message Persistence

### Database Schema (PostgreSQL)

```sql
-- Messages table
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  channel_id UUID NOT NULL,
  user_id UUID NOT NULL,
  content TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  edited_at TIMESTAMP,
  deleted_at TIMESTAMP,
  parent_id UUID REFERENCES messages(id), -- For threads
  INDEX(channel_id, created_at DESC)
);

-- Reactions table
CREATE TABLE reactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  message_id UUID REFERENCES messages(id),
  user_id UUID NOT NULL,
  emoji TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(message_id, user_id, emoji)
);

-- Read receipts
CREATE TABLE read_receipts (
  channel_id UUID NOT NULL,
  user_id UUID NOT NULL,
  last_read_at TIMESTAMP NOT NULL,
  PRIMARY KEY(channel_id, user_id)
);
```

### Message Retention

**Options**:
1. **Keep forever**: Slack free tier (10k messages)
2. **Time-based**: Delete after 90 days
3. **Storage-based**: Delete oldest when >1TB
4. **Export + delete**: Archive to S3, delete from DB

---

## System Features: Architecture Patterns

### Pattern 1: Simple Chat (Small Scale)

**Stack**:
```
Frontend: React + Socket.IO
Backend: Node.js + Socket.IO
Database: PostgreSQL (messages)
Cache: Redis (presence, recent messages)
```

**Scaling limit**: ~10k concurrent users  
**Good for**: Startups, MVPs

---

### Pattern 2: Scalable Chat (Medium Scale)

**Stack**:
```
Frontend: React + Socket.IO
WebSocket Servers: Node.js + Socket.IO + Redis adapter
API Servers: Node.js REST API (horizontal scaling)
Database: PostgreSQL (messages) + read replicas
Cache: Redis cluster (presence, cache)
Queue: Kafka (message persistence, analytics)
```

**Scaling limit**: ~100k concurrent users  
**Good for**: Growing SaaS, B2B products

---

### Pattern 3: Massive Scale Chat

**Stack**:
```
Frontend: React + custom WebSocket client
Edge: Cloudflare Workers (WebSocket termination)
Backend: Go/Rust WebSocket servers + NATS (message bus)
Database: Cassandra (messages), Redis cluster (cache)
Queue: Kafka (persistence, analytics, webhooks)
CDN: Cloudflare (static assets, media)
```

**Scaling limit**: Millions of concurrent users  
**Good for**: Slack, Discord, WhatsApp competitors

---

## User Features: UI Patterns

### Chat UI Components

**Essential**:
- Message list (virtualized for performance)
- Input box (auto-resize, file upload, emoji)
- Typing indicators ("Alice is typing...")
- Online status (green dot)
- Unread badge (red bubble with count)

**Nice-to-have**:
- Threads (reply to specific message)
- Reactions (👍, ❤️, 😂)
- File preview (images, PDFs)
- Link previews (unfurl URLs)
- Search (messages, users, channels)
- Pinned messages
- Giphy integration
- Code syntax highlighting

---

### Notification UI

**In-App**:
```
┌─────────────────────────────────┐
│ 🔔 Notifications          (3)  │
├─────────────────────────────────┤
│ 💬 Alice mentioned you          │
│    "Can you review this?"       │
│    2 min ago                    │
├─────────────────────────────────┤
│ ⭐ Bob reacted to your message  │
│    👍                           │
│    5 min ago                    │
├─────────────────────────────────┤
│ 🎉 New feature released!        │
│    Check out dark mode          │
│    1 hour ago                   │
└─────────────────────────────────┘
```

**Best practices**:
- Show most recent first
- Mark as read on click
- Clear all button
- Deep link to source (open message in chat)
- Group similar notifications

---

## Common Pitfalls

### 🚩 Not Handling Reconnects
**Problem**: User loses connection, misses messages  
**Solution**: Socket.IO auto-reconnect + fetch missed messages on reconnect

### 🚩 No Message Deduplication
**Problem**: Network retry sends duplicate messages  
**Solution**: Client-generated message ID (UUID), server dedupes

### 🚩 Sending Push Notifications While User Active in App
**Problem**: Annoying (user sees notification AND in-app message)  
**Solution**: Check presence before sending push

### 🚩 No Rate Limiting
**Problem**: Spam, DoS attacks  
**Solution**: Rate limit messages (10/min per user), IP-based limits

### 🚩 Storing Large Files in Database
**Problem**: DB bloat, slow queries  
**Solution**: Store files in S3, save URL in DB

---

## Example Stacks

### Stack 1: Simple Team Chat (MVP)
- **Chat**: Socket.IO (Node.js)
- **Database**: PostgreSQL (messages, users)
- **Cache**: Redis (presence, recent messages)
- **Notifications**: Email only (SendGrid)
- **Storage**: Local disk (small files)
- **Cost**: $50-200/mo

**Perfect for**: Internal team chat, MVP

---

### Stack 2: Customer Support Chat
- **Chat**: Stream Chat SDK
- **Notifications**: In-app + email (SendGrid)
- **Storage**: S3 (file uploads)
- **Cost**: $500-2000/mo (based on MAU)

**Perfect for**: SaaS support chat, low volume

---

### Stack 3: Social App with Chat
- **Chat**: Sendbird SDK
- **Push**: FCM (mobile), OneSignal (web)
- **Storage**: Cloudflare R2 (media)
- **CDN**: Cloudflare (images, videos)
- **Cost**: $1000-5000/mo

**Perfect for**: Social apps, community platforms

---

### Stack 4: Custom High-Scale Chat
- **Chat**: Custom (Go + NATS)
- **Database**: Cassandra (messages), PostgreSQL (users)
- **Cache**: Redis cluster
- **Queue**: Kafka (analytics, webhooks)
- **Push**: FCM + APNS (direct integration)
- **Storage**: S3 (files), CloudFront (CDN)
- **Cost**: $5000-20k/mo (infrastructure)

**Perfect for**: Slack/Discord competitor, >100k DAU

---

## Decision Checklist

- [ ] Use case? (team chat, support, social)
- [ ] Scale? (<1k users = simple, >10k = scalable)
- [ ] Budget? (SaaS = fast, custom = cheaper at scale)
- [ ] Features needed? (threads, reactions, video calls)
- [ ] Platform? (web, mobile, both)
- [ ] Real-time critical? (yes = WebSocket, no = polling)
- [ ] Push notifications? (mobile = FCM/APNS, web = Web Push)

---

**Next Steps**:
- Review [Integration Layer - Real-Time](../layers/04-integration.md#real-time-communication)
- Review [AI Stack](ai.md) - For AI-powered chat (ChatGPT-style)
- Review [Analytics Stack](analytics.md) - Track message metrics

**Last Updated**: 2025-12-12  
**Contribute**: Found a better messaging pattern? [Open a PR](../CONTRIBUTING.md)


