# Users, Requirements, and Constraints

Mobile architecture begins with operating conditions, not frameworks.

## Characterize the users

Translate each user characteristic into a possible engineering consequence.

| User or context | Possible consequence |
|---|---|
| Low-end devices | smaller memory budget, fewer concurrent decodes, reduced animation |
| Older OS versions | compatibility layer, larger test matrix, fewer new platform APIs |
| Intermittent connectivity | durable local state, resumable operations, explicit staleness |
| Metered data | thumbnails, adaptive quality, Wi-Fi-only transfer option |
| Global audience | localization, RTL, font coverage, regional latency |
| Accessibility needs | semantic UI, dynamic type, screen-reader flows, larger targets |
| Regulated work | stronger authentication, audit trails, data retention controls |
| Shared devices | account isolation, fast logout, encrypted local data |

Avoid stereotypes. Age does not imply technical inability. Use evidence about
device distribution, accessibility, and support demand.

## Functional requirements: choose journeys, not feature nouns

“Profile,” “feed,” and “upload” are too shallow. Describe end-to-end behaviour:

```text
The user selects a 2 GiB video, leaves the app, loses connectivity, reopens it
tomorrow, and expects the upload to continue without starting from zero.
```

That journey implies:

- durable transfer metadata;
- background execution integration;
- resumable server protocol;
- network and battery constraints;
- integrity checks;
- visible progress;
- cleanup of abandoned sessions.

## Quality attributes

### Reliability

Specify the unit:

- message eventually sent;
- file bytes durably uploaded;
- payment submitted at most once;
- local edit never silently lost;
- screen can render cached data during an outage.

“99.9% reliable” is meaningless until the event and measurement window are
defined.

### Latency

Split perceived latency:

```text
tap
→ local feedback
→ cached content
→ fresh network content
→ background completeness
```

A UI can respond in 50 ms even when server confirmation takes two seconds.
Optimistic state, skeletons, cached content, and prefetching optimize different
parts of this timeline.

### Freshness

Define a staleness budget:

- live driver position: seconds;
- chat presence: seconds to minutes;
- social feed: minutes may be acceptable;
- account balance after transfer: correctness over cached appearance;
- reference catalogue: hours or days.

Freshness decides whether to use push, sockets, polling, refresh-on-open, TTL,
or manual refresh.

### Battery

Battery cost comes from more than CPU:

- radio wake-ups;
- high-accuracy location;
- long-lived connections;
- frequent timers;
- repeated retries;
- background execution;
- media encoding and decoding;
- excessive rendering.

Batching ten network operations is often cheaper than waking the radio ten
times. Push notification services are generally more battery-efficient than
each app continuously polling.

### Memory

Design for peak memory, not just stored file size. A compressed 12 MP JPEG may
occupy tens of megabytes when decoded into a bitmap. Lists should load windows,
not complete histories. Streams and file-backed uploads avoid reading an entire
asset into memory.

### Application size

Size affects installation conversion, updates, and storage pressure. Possible
solutions include:

- architecture-specific binaries and app bundles;
- resource shrinking;
- removing unused assets and transitive dependencies;
- server-delivered content;
- dynamic feature delivery where platform and UX justify it;
- measuring the size contribution of each module.

### Scale

Separate four scales:

1. backend population and request volume;
2. one account’s data size;
3. one device’s local dataset;
4. engineering organization and codebase size.

A business chat app may have moderate global traffic but one enterprise account
with enormous history. A consumer app can have billions of accounts while one
device holds only a bounded cache.

## Turn requirements into budgets

An interview-grade design should state provisional budgets:

| Signal | Example starting budget |
|---|---:|
| Cold start to usable content | p95 under 2 s on supported mid-tier devices |
| Cached screen render | under 200 ms |
| Local interaction feedback | under 100 ms |
| Pending mutation age online | p95 under 10 s |
| Crash-free sessions | above 99.9% |
| Attachment upload recovery | no restart after acknowledged chunks |
| Disk cache | bounded and evictable |

These are examples, not universal targets. The important move is to make the
quality attribute measurable.

## Real-world evidence

**Documented — Instagram:** Meta has described Instagram background data
prefetching as a way to reduce dependence on network availability while
controlling cellular usage and improving perceived speed. This illustrates a
trade-off, not a command to prefetch everything: the app predicts likely
future demand and spends data, battery, and cache space to lower interaction
latency.

**Documented — Facebook iOS:** Meta’s account of Facebook’s iOS architecture
describes how feature growth allowed many products to add work to startup,
creating a “tragedy of the commons.” The lesson is organizational: startup is a
shared, budgeted resource, and module boundaries need governance.

**Documented — Dropbox Carousel:** Dropbox described designing a mobile photo
experience for collections exceeding 100,000 items and working around Android
garbage-collection pressure. The lesson is to design lists and media pipelines
for worst credible collections rather than demo-sized datasets.

## Interview checkpoint

Before proposing architecture, be able to say:

> The top priorities are X, Y, and Z. I am deliberately accepting A to improve
> B. I will validate that choice with metrics C and D.

