# How to Use This Book

Mobile system design operates at three zoom levels.

## Level 1: Product system

Start outside the code:

- Who uses the product?
- What are the critical journeys?
- What devices, networks, countries, and accessibility needs matter?
- Which failures are tolerable?
- Which data is sensitive?
- What does “fast” mean for this product?

Designing a banking app for recent flagship phones is different from designing
a messaging app expected to work on inexpensive phones, in tunnels, across
every language, and under aggressive background restrictions.

## Level 2: Application architecture

Decide how the client remains understandable as both code and organization
grow:

- state ownership;
- UI, domain, data, and platform boundaries;
- module boundaries;
- local source of truth;
- synchronization ownership;
- dependency direction;
- test seams;
- release and compatibility boundaries.

The architecture must scale for people as well as traffic. A codebase touched
by four engineers has different needs from one touched by hundreds.

## Level 3: Mechanism

Choose technologies and algorithms:

- REST, GraphQL, gRPC, WebSocket, SSE, or push;
- offset or cursor pagination;
- memory cache, disk cache, or local database;
- optimistic writes or server-confirmed writes;
- retry, backoff, deduplication, and idempotency;
- whole-file, streaming, multipart, or resumable transfers;
- last-write-wins, field merge, OT, or CRDT conflict resolution;
- Keychain/Keystore, transport encryption, or end-to-end encryption.

Do not jump to this level too early. “Use WebSockets” is not a design until the
delivery semantics, reconnection behaviour, background path, ordering,
authentication, and observability are explained.

## Two suggested reading paths

### Interview preparation

Read chapters 1, 2, 4, 5, 6, 11, then practise one worked design daily. For
each design, speak aloud and time-box:

- 5 minutes: clarify scope and priorities;
- 10 minutes: high-level client architecture;
- 20 minutes: two deep dives;
- 5 minutes: failure modes, trade-offs, and measurement.

### Production reference

Read sequentially. Each chapter moves from the product problem to solution
families, platform mechanisms, failure modes, and real-world evidence.

## The evidence discipline

Examples in this book intentionally avoid mythology. Engineering articles can
become outdated, and a company may use different architectures across apps or
platforms. Always preserve the date and scope of a source. A Dropbox desktop
sync-engine article can teach sync invariants, but it is not proof that the
Dropbox mobile app uses the identical engine.

