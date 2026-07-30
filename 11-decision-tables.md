# Decision Tables

## Communication

| Need | Start with | Add when required |
|---|---|---|
| ordinary resource reads/writes | REST/HTTP | GraphQL/BFF for complex composition |
| foreground bidirectional events | WebSocket | cursor-based resume and HTTP recovery |
| foreground server-only events | SSE | HTTP commands in opposite direction |
| process may be suspended/killed | APNs/FCM hint | reconcile via HTTP |
| low-frequency approximate freshness | polling/refresh-on-open | push when cost justifies |

## Local data

| Need | Solution |
|---|---|
| repeated within-session read | memory cache |
| reconstructible assets across sessions | bounded disk cache |
| queryable durable entities | local database |
| sensitive credential | Keychain/Keystore |
| durable outgoing intent | transactional outbox |
| user-pinned large content | managed file storage plus database metadata |

## Conflict resolution

| Data | Likely starting strategy |
|---|---|
| social like/read marker | idempotent operation or set semantics |
| profile field | revision check plus field/last-write policy |
| chat message creation | append-only client ID |
| payment | server-authoritative state machine and idempotency |
| note edited on two devices | revision conflict, merge or user choice |
| collaborative rich text | OT or CRDT after proven requirement |

## Large transfers

| Situation | Solution |
|---|---|
| small bounded attachment | single streaming request |
| large, unreliable network | resumable upload |
| very large and bandwidth-rich | multipart with bounded parallelism |
| must continue while app suspended | OS-managed background transfer |
| seekable media playback | range/segmented streaming |

