# Networking and API Design

Protocol selection follows interaction shape.

## Communication models

| Model | Best fit | Main costs |
|---|---|---|
| REST/HTTP | resource operations, cacheable reads, broad tooling | over/under-fetching, endpoint proliferation |
| GraphQL | client-shaped composite screens, evolving field needs | cache normalization, query cost, schema governance |
| gRPC | typed internal contracts, streaming, compact binary payloads | browser constraints, debugging and gateway complexity |
| WebSocket | foreground bidirectional low-latency events | connection lifecycle, ordering, reconnect storms |
| SSE | server-to-client foreground event stream | one direction, mobile background limits |
| Push | wake-up or user notification when app is not active | best-effort, platform mediation, payload limits |
| Polling | simple low-frequency freshness | latency, data and battery cost |

Many production apps combine them. A chat app may send foreground messages over
a WebSocket, fetch history through HTTP, upload attachments to object storage,
and rely on APNs/FCM when suspended or killed.

## REST

REST is a good default when resources and operations are stable.

```http
GET /v2/conversations/abc/messages?before=cursor&limit=50
POST /v2/conversations/abc/messages
Idempotency-Key: 018f...
```

Design considerations:

- stable identifiers;
- explicit pagination;
- conditional requests with ETag/If-None-Match;
- compression;
- field and endpoint deprecation policy;
- structured error codes;
- client operation IDs;
- server timestamps and revisions;
- request deadlines.

## GraphQL

GraphQL helps when one screen needs a changing composition of data:

```graphql
query Home($after: Cursor) {
  viewer {
    profile { displayName avatar(size: SMALL) }
    feed(after: $after, first: 20) {
      edges { cursor node { id revision summary } }
    }
  }
}
```

It does not automatically make mobile faster. A good design still needs:

- persisted queries or query allowlists;
- response and normalized-cache policy;
- pagination conventions;
- query depth/cost limits;
- error semantics for partial data;
- schema compatibility;
- observability by operation name;
- protection against one screen creating an expensive backend fan-out.

## WebSockets

A socket is a transport, not a messaging guarantee.

Define:

- connection authentication and token refresh;
- heartbeat strategy;
- reconnect with exponential backoff and jitter;
- network-change handling;
- session resume cursor;
- per-stream ordering;
- acknowledgement semantics;
- duplicate detection;
- maximum buffered data;
- foreground/background transition;
- fallback when the process is dead.

Example event envelope:

```json
{
  "eventId": "evt_984",
  "stream": "conversation_123",
  "sequence": 4831,
  "type": "message.created",
  "payload": {"messageId": "m_91"},
  "serverTime": "2026-07-30T09:00:00Z"
}
```

On reconnect, the client sends its last applied cursor. The server returns
missed events or tells the client to perform a full resync if history has
expired.

## Push notification is a hint, not a database

APNs and FCM are essential when the app cannot keep its own process and socket
alive. Treat push as:

- a user-visible notification; or
- a hint that fresh data may exist.

Do not assume exactly-once delivery or global ordering. When opened or awakened,
the app reconciles against authoritative server state. Sensitive payloads
should be minimized; notification previews are a product/privacy decision.

## Pagination

### Offset pagination

```http
GET /items?offset=100&limit=20
```

Simple and supports jumping to a page. Concurrent insertions/deletions can
cause duplicates or gaps, and large offsets may be expensive.

### Cursor pagination

```http
GET /items?after=opaque_cursor&limit=20
```

Better for changing feeds and histories. The cursor should be opaque and tied
to a stable sort order.

### Keyset pagination

Conceptually:

```sql
WHERE (created_at, id) < (:created_at, :id)
ORDER BY created_at DESC, id DESC
LIMIT 20
```

The unique tie-breaker prevents records sharing a timestamp from being skipped.

On the client, preserve page boundaries only if useful. Often it is simpler to
merge records by stable ID into a database and query a local window.

## Retry correctly

Classify failures:

| Failure | Typical response |
|---|---|
| No connectivity | wait for network constraint |
| Timeout before response | retry only if safe/idempotent |
| HTTP 429 | respect Retry-After, back off |
| HTTP 5xx | bounded exponential retry with jitter |
| HTTP 401 expired token | single coordinated refresh, then retry |
| HTTP 403 | do not retry blindly |
| Validation error | show actionable permanent failure |
| Conflict | fetch revision and resolve policy |

Use exponential backoff:

```text
delay = min(cap, base × 2^attempt) + random_jitter
```

Jitter prevents thousands of devices reconnecting simultaneously after an
outage.

## Idempotency and uncertain outcomes

Suppose a payment request times out. The server may have committed it even
though the response never reached the phone. Retrying with a new identifier can
charge twice.

The client creates a stable operation ID:

```http
POST /payments
Idempotency-Key: op_123
```

The server stores the result for that key. Repeating the request returns the
same logical outcome.

For local mutations, persist the operation and entity change atomically where
possible:

```text
database transaction:
  update visible entity as pending
  insert outbox operation with clientOperationId
```

## Request efficiency

Concrete options:

- batch small independent writes;
- coalesce simultaneous identical reads;
- cancel obsolete search requests;
- debounce user input where semantics permit;
- request thumbnails rather than originals;
- compress suitable payloads;
- cache immutable assets by content hash;
- preconnect or prefetch only when prediction value exceeds cost;
- use a mobile BFF when a critical screen otherwise causes many serial calls.

## Backward compatibility

Mobile binaries remain installed for months or years. The backend must:

- add fields compatibly;
- avoid changing existing field meaning;
- accept older enum behaviour;
- version destructive changes;
- maintain a minimum-supported-version policy;
- allow emergency kill switches without making startup depend entirely on
  remote configuration.

Client parsers should tolerate unknown fields and, where possible, unknown enum
values.

## Interview answer

> I’ll use HTTP for history and commands, object storage for media, and a
> WebSocket for foreground events. Push is the background wake-up path. Events
> carry IDs and per-stream sequence numbers; reconnect resumes from a cursor.
> Mutating requests use stable client operation IDs so timeout retries are
> idempotent. Lists use cursor pagination and merge into local storage by ID.

