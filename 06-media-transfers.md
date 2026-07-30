# Media and Large-File Transfers

Large media turns memory, networking, storage, background execution, and user
feedback into one system.

## Separate metadata from bytes

Create the logical media record before or alongside the byte transfer:

```text
MediaAsset(
  id,
  ownerId,
  localUri,
  remoteObjectKey?,
  mimeType,
  byteCount,
  checksum?,
  width?,
  height?,
  duration?,
  transferState
)
```

The feed or message references the asset ID. Upload completion can transition
the containing entity without forcing raw bytes through the main API service.

## Upload patterns

### Single request

Best for small bounded objects. Simple, but any interruption may restart the
whole transfer.

### Streaming request

Reads from file incrementally rather than loading all bytes into memory. It
controls memory but is not automatically resumable.

### Resumable upload

Typical flow:

```text
1. Create upload session and obtain session URL/ID.
2. Send a byte range or part.
3. Server acknowledges durable offset/part.
4. Persist acknowledged progress locally.
5. Resume from server-confirmed position after interruption.
6. Finalize and verify full object.
```

### Multipart parallel upload

Parts can transfer concurrently and be assembled by object storage.

Benefits:

- higher throughput on suitable networks;
- retry only failed parts;
- bounded memory if file-backed.

Costs:

- more radio/CPU use;
- server session and part cleanup;
- ordering/assembly metadata;
- too much parallelism competes with UI and other apps;
- small parts create request overhead.

Adaptive concurrency is better than a magic fixed number.

## What chunking does—and does not do

Chunking provides:

- resumability;
- bounded buffering;
- localized retry;
- progress checkpoints;
- optional parallelism;
- part-level integrity checks.

It does not inherently reduce the number of bytes. It can be faster through
parallelism or avoided retransmission, but it can also add requests and
bookkeeping.

## Integrity

Possible levels:

- transport integrity from TLS/TCP;
- per-part checksum;
- full-object checksum;
- content hash used as object identity;
- encrypted authenticated chunks.

Never treat an ETag as a universal MD5 checksum; semantics vary by storage
provider and multipart mode.

## Background transfers

**Documented — iOS:** Apple’s background `URLSession` allows HTTP(S) upload and
download work to be managed outside the application process. Apple also
documents resumable upload support. The app still needs durable mapping between
system task identifiers and product entities.

**Documented — Android:** WorkManager supports persistent constraint-aware work,
including long-running work where appropriate. For user-important active
transfers, Android foreground-service requirements and user-visible
notifications may apply depending on OS version and transfer model.

## Download design

Decide:

- stream for immediate consumption or save first;
- range requests for seeking/resume;
- temporary versus pinned storage;
- expected content length;
- checksum;
- encryption/decryption path;
- cache eviction;
- cellular and roaming policy;
- partial-file naming and cleanup.

Write partial content to a temporary path, then atomically publish it after
verification. A crash should not make a truncated file appear complete.

## Image delivery

Serve variants appropriate to the display:

- thumbnail for lists;
- screen-sized image for detail;
- original only for zoom/export;
- modern codecs where support and server pipeline justify them.

Client pipeline:

```text
memory cache
→ disk cache
→ network
→ decode near target size
→ render
```

Cache keys should incorporate transformations and content revision. Immutable
content-addressed URLs simplify invalidation.

## Video and audio

For playback, adaptive segmented streaming such as HLS or DASH lets the player
choose quality based on bandwidth and buffer. Progressive download may be
sufficient for small simple media.

Measure:

- time to first frame/audio;
- rebuffer ratio;
- selected bitrate;
- playback failure;
- bytes wasted after abandonment;
- offline-license/entitlement failures where relevant.

## Production examples

**Documented — Google Drive API:** Google documents resumable uploads intended
for large files and interruption-prone networks. A resumable session persists
progress so a client can continue rather than resend the whole file. This is
evidence for the protocol family; the exact consumer Drive app implementation
must not be invented.

**Documented — Apple:** Apple’s robust and resumable transfer material explains
pause/resume and background file-transfer mechanisms available to iOS apps.

**Illustrative — a Drive-like mobile app:** persist upload session ID, next
acknowledged offset, local file identity, account, total size, checksum,
attempts, and expiry. Before resuming, confirm the local asset still represents
the same bytes. Query server progress when the last acknowledgement is
uncertain.

**Illustrative — an Instagram-like feed:** request small preview variants,
prefetch the next likely assets under budget, cancel obsolete work during fast
scroll, bound decoded bitmap memory, and avoid downloading full-resolution
media until required.

## Failure checklist

- local file changed after upload began;
- server session expired;
- last part committed but acknowledgement was lost;
- process dies after OS transfer completes but before database update;
- user logs out while background transfer runs;
- checksum mismatch;
- storage fills during download;
- signed URL expires;
- duplicate finalization;
- CDN returns stale metadata;
- aggressive retries drain battery;
- cache eviction deletes an in-progress file.

