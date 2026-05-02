# Project Design Document
## StreamForge — AI-Native Video Streaming Platform

**Version:** 1.0
**Last Updated:** April 2026
**Status:** In Design
**Author:** [Your Name]

---

> **Document Purpose**
>
> This is the single source of truth for the design, architecture, and implementation plan of StreamForge. It should be read before writing any code, updated as decisions evolve, and referenced throughout every phase of development.
>
> This document does not answer every question upfront. It captures what is known, what has been decided, and — more importantly — *why* each decision was made. The reasoning behind a decision is more valuable than the decision itself, especially in interviews and code reviews.

---

## Table of Contents

1. Problem Statement
2. Goal & Vision
3. Users & Use Cases
4. Feature Breakdown — All Phases
5. Tech Stack & Decisions
6. High Level Design (HLD)
7. Low Level Design (LLD) — Phase 1
8. Database Design
9. API Design
10. Component & Module Breakdown
11. Engineering Considerations — Edge Cases & Tradeoffs
12. Non-Functional Requirements
13. Project Timeline
14. Future Scope & Learning Notes

---
---

## 1. Problem Statement

**General form:** [WHO] faces difficulty in [WHAT PROBLEM], because [WHY it is inefficient], resulting in [IMPACT].

**This project:**
Content creators and platform operators face difficulty in building a video streaming platform that handles the full lifecycle of video — from upload through processing, delivery, discovery, and personalisation — because building each piece in isolation is well-documented, but integrating them into a cohesive, scalable system requires deep knowledge of distributed systems, video protocols, AI tooling, and infrastructure that is rarely taught together.

**The secondary problem this project solves — for you as the builder:**
Existing portfolio projects for backend engineers demonstrate individual skills in isolation. There is no single project that credibly demonstrates system design depth, event-driven architecture, video processing pipelines, AI integration, observability, and production deployment simultaneously. StreamForge is that project.

**What makes this non-trivial:**
The hard part is not building a video player. The hard part is designing a system where a 500MB video upload triggers an async pipeline that transcodes into multiple quality levels, enriches the video with AI-generated subtitles and tags, serves segments via a CDN with adaptive bitrate, and recommends the video to the right users — all without any single component blocking another, and all recoverable if any component fails.

---
---

## 2. Goal & Vision

**General form:** To enable [WHO] to [DO WHAT], using [HOW], so that [OUTCOME].

**This project:**
To build a production-grade, AI-native video streaming platform that enables content creators to upload and publish videos, and viewers to discover and stream personalised content — using an event-driven microservices architecture, HLS adaptive bitrate streaming, and an AI enrichment pipeline — so that the platform demonstrates real-world engineering depth across backend systems, distributed architecture, AI integration, and cloud infrastructure.

**What success looks like at completion:**
- A user uploads a video → it is automatically transcoded into 360p, 720p, and 1080p HLS streams
- AI generates subtitles, tags, and a smart thumbnail without any manual input
- A viewer streams the video with adaptive bitrate — quality adjusts to their bandwidth automatically
- Search returns semantically relevant results, not just title matches
- The platform handles 1000 concurrent users without degradation, demonstrated by a load test
- Every service is deployed on Kubernetes, monitored in Grafana, and recoverable from failure

---
---

## 3. Users & Use Cases

### User Roles

| Role | Who they are | Primary goal |
|---|---|---|
| **Content Creator** | Anyone who uploads videos to the platform | Upload, publish, and track performance of videos |
| **Viewer** | Anyone who watches content | Discover, stream, and interact with videos |
| **Admin** | Platform operator | Moderate content, manage users, view platform analytics |

---

### Use Cases by Role

**Content Creator**
- Register and create an account
- Upload a video file (up to 2GB)
- Add title, description, and tags manually (optional — AI fills gaps)
- View upload and processing status in real time
- See AI-generated subtitles, thumbnail, and tags after processing
- Edit or override AI-generated metadata
- View analytics — watch time, view count, drop-off curve
- Delete or unpublish a video

**Viewer**
- Browse a personalised video feed on the homepage
- Search for videos using natural language queries
- Stream a video with adaptive quality based on connection speed
- Toggle subtitles on/off (auto-generated, multi-language)
- Like, save, and share videos
- View recommended videos based on watch history
- Continue watching from where they left off

**Admin**
- View all uploaded content
- Remove or flag content that failed moderation
- View platform-wide analytics (uploads, streams, active users)
- Manage user accounts

---
---

## 4. Feature Breakdown — All Phases

### How to read this section
Features are grouped by phase. Each phase builds on the previous one. The question for every feature is:
- *If I remove this, does the core flow break?* → Core (MVP)
- *Does this make the product significantly better?* → Phase 2
- *Is this about scale, observability, or polish?* → Phase 3+

---

### Phase 1 — MVP (Core Pipeline)
**Goal: Upload a video, process it, stream it. Everything else is secondary.**

| # | Feature | Description |
|---|---|---|
| 1 | **User Auth** | Register, login, JWT access + refresh tokens. No OAuth yet. |
| 2 | **Video Upload** | Upload video file to S3/MinIO. Chunked upload for large files. |
| 3 | **Upload Progress** | Real-time upload progress via WebSocket or SSE. |
| 4 | **Video Transcoding** | FFmpeg transcodes uploaded video into 360p, 720p, 1080p HLS. |
| 5 | **HLS Delivery** | Master `.m3u8` playlist + quality playlists + `.ts` segments served via CDN. |
| 6 | **Adaptive Bitrate Player** | `hls.js`-powered player that switches quality based on bandwidth. |
| 7 | **Video Metadata** | Title, description, tags, duration, resolution stored in PostgreSQL. |
| 8 | **Processing Status** | Kafka event pipeline updates video status (uploading → processing → ready → failed). |
| 9 | **Basic Video Feed** | Homepage grid of published videos, sorted by upload date. |
| 10 | **Basic Search** | Keyword search on title and description via Meilisearch. |
| 11 | **Video Detail Page** | Player + metadata + uploader info. |

---

### Phase 2 — AI Enrichment Layer
**Goal: Every video is enriched automatically the moment processing completes.**

| # | Feature | Description |
|---|---|---|
| 12 | **Auto Subtitles** | OpenAI Whisper transcribes audio → generates `.vtt` subtitle file. |
| 13 | **Multi-language Translation** | Subtitles auto-translated to 5 languages (Hindi, Spanish, French, German, Japanese). |
| 14 | **Smart Thumbnails** | CLIP model scores every 2-second keyframe → top 3 candidates presented to creator. |
| 15 | **Scene & Object Tagging** | YOLOv8 detects objects in frames → auto-generates searchable tags. |
| 16 | **Content Moderation** | NSFW and violence classifier runs on frames before video is made public. |
| 17 | **Semantic Search** | Videos embedded via sentence-transformer → stored in pgvector → natural language queries work. |
| 18 | **Recommendation Engine** | User vectors built from watch history → cosine similarity against video vectors → personalised feed. |
| 19 | **Watch History** | Track per-user watch progress, resume position, and completed videos. |
| 20 | **Like & Save** | Users can like and save videos to a personal library. |

---

### Phase 3 — Scale, Observability & Infrastructure
**Goal: Make it production-grade. Prove it scales. Monitor everything.**

| # | Feature | Description |
|---|---|---|
| 21 | **Kubernetes Deployment** | All services deployed as K8s pods with Helm charts. |
| 22 | **Horizontal Pod Autoscaling** | Transcoding worker pods scale based on Kafka queue depth. |
| 23 | **Prometheus + Grafana** | Metrics for every service — request rate, error rate, queue depth, transcoding time. |
| 24 | **ClickHouse Analytics** | View events streamed via Kafka → stored in ClickHouse → analytics dashboard (watch time, drop-off). |
| 25 | **CDN Integration** | CloudFront (prod) or MinIO (dev) for HLS segment caching at edge. |
| 26 | **Load Testing** | k6 scripts simulating 100 → 1000 concurrent streamers. Bottlenecks identified and fixed. |
| 27 | **CI/CD Pipeline** | GitHub Actions → Docker build → push to ECR → deploy to EKS. |
| 28 | **Rate Limiting** | Per-user and per-IP rate limits on all public APIs via Redis. |
| 29 | **Structured Logging** | Pino logger across all Node.js services → log aggregation via Loki. |
| 30 | **Distributed Tracing** | OpenTelemetry spans across services → trace a single video upload end-to-end. |

---

### Phase 4 — Advanced Features & Polish
**Goal: Feature completeness. Real product feel.**

| # | Feature | Description |
|---|---|---|
| 31 | **OAuth Login** | Google and GitHub social login. |
| 32 | **Notifications** | In-app + email notifications (new upload from followed creator, processing complete). |
| 33 | **Creator Analytics Dashboard** | Per-video: views, watch time, audience drop-off curve, device breakdown. |
| 34 | **Multiple Thumbnail Templates** | Creators choose from 3 AI-generated thumbnail candidates or upload their own. |
| 35 | **Video Chapters** | Creators add chapter markers → shown in player timeline. |
| 36 | **Shorts / Clips** | Clip any 60-second segment of a video → shareable as a short. |
| 37 | **Comments & Replies** | Threaded comments on videos. |
| 38 | **Subscriptions** | Users follow creators → personalised feed prioritises followed creators. |
| 39 | **Mobile Responsive UI** | Full mobile optimisation across all pages. |
| 40 | **Demo Video + Architecture Docs** | 15-min recorded walkthrough of the platform under load. Public architecture doc. |

---
---

## 5. Tech Stack & Decisions

Every decision here has a reason. The reason is more important than the decision.

### Frontend

| Choice | Alternatives Considered | Why this |
|---|---|---|
| **React + Vite** | Next.js, Vanilla JS | React for component reusability and ecosystem. Vite over CRA — faster dev server, better DX. Next.js skipped because SSR adds complexity without meaningful SEO benefit for a streaming platform (content is behind auth anyway). |
| **Tailwind CSS** | Bootstrap, plain CSS | Utility-first means no context-switching between files. Faster to iterate on UI without a design system. |
| **hls.js** | Video.js, Shaka Player | Lightweight, focused solely on HLS playback. Video.js is heavier with plugins. hls.js gives direct control over quality switching logic. |
| **TanStack Query** | SWR, raw useEffect | Handles caching, background refetch, loading and error states cleanly. Eliminates the useEffect-for-data-fetching anti-pattern. |
| **Zustand** | Redux, Context API | Lightweight global state for auth and player state. Redux is overkill here. Context causes unnecessary re-renders at scale. |

---

### Backend

| Choice | Alternatives Considered | Why this |
|---|---|---|
| **Node.js + Fastify** | Express, Spring Boot, Django | Node's event-driven I/O handles thousands of concurrent HTTP range requests (HLS segment serving) efficiently. Fastify over Express — schema-based validation, better TypeScript support, measurably faster. Spring Boot is a stronger choice for a Java team but adds JVM overhead and slower startup time that hurts Kubernetes pod scaling. |
| **Python (FastAPI) — AI workers only** | Node.js for all | ML ecosystem lives in Python. Whisper, CLIP, YOLOv8 — all Python-native. Calling these from Node.js via FFI or child_process is fragile. Clean separation: Node.js orchestrates, Python infers. Kafka as the bridge. |
| **TypeScript throughout** | Plain JavaScript | Type safety catches interface mismatches between services early. In a multi-service system, the cost of a wrong payload shape is high. TS pays for itself within the first week. |

---

### Message Queue

| Choice | Alternatives Considered | Why this |
|---|---|---|
| **Apache Kafka** | RabbitMQ, BullMQ, AWS SQS | Kafka is a distributed log, not just a queue. Events are retained and replayable — if a transcoding worker crashes, the event is not lost, the consumer group resumes from last committed offset. RabbitMQ is a better choice for simple job queues but doesn't give you event replay or the consumer group model. BullMQ (Redis-backed) is used for simpler background jobs within a service. |

---

### Databases

| Database | Used for | Why this |
|---|---|---|
| **PostgreSQL** | Users, videos, metadata, subtitles, watch history, likes | Relational integrity matters here — a video belongs to a user, a subtitle belongs to a video. ACID transactions for writes that must not partially succeed. pgvector extension adds vector similarity search without a separate service. |
| **Redis** | Sessions, JWT refresh tokens, API rate limiting, hot video cache, pub/sub for notifications | In-memory speed for things that need sub-millisecond access. Refresh token rotation, session state, and rate limit counters all fit this profile. |
| **MongoDB** | — | Not used. PostgreSQL with JSONB handles any semi-structured data needs. Adding MongoDB would be a third database with no clear advantage over what PostgreSQL already provides. |
| **ClickHouse** | View events, watch time analytics, drop-off curves | Columnar storage optimised for analytical queries over billions of rows. PostgreSQL cannot efficiently compute "average watch time grouped by video and day" over 100M rows. ClickHouse can. This is what YouTube-scale analytics actually uses. |
| **pgvector (PostgreSQL extension)** | Video embeddings, user embeddings for semantic search and recommendations | Avoids running a separate vector database (Pinecone, Weaviate) for MVP. pgvector on PostgreSQL handles millions of vectors well. Can migrate to a dedicated vector DB if scale demands it. |
| **Meilisearch (Phase 1) → Elasticsearch (Phase 3)** | Full-text keyword search | Meilisearch is lightweight (~100MB RAM) and trivial to set up — perfect for development and early production. Elasticsearch is more powerful (custom analysers, complex queries, horizontal scaling) but costs 1–1.5GB RAM minimum. Swap at Phase 3 when load testing justifies it. |

---

### Storage & CDN

| Choice | Why |
|---|---|
| **MinIO (local dev)** | S3-compatible object storage that runs locally in Docker. Identical API to AWS S3, so switching to real S3 in production requires zero code changes. |
| **AWS S3 (production)** | Industry standard for object storage. Durability, availability, and native CloudFront integration. |
| **AWS CloudFront (production)** | CDN for HLS segment delivery. Caches `.ts` segments at edge PoPs globally. A viewer in Mumbai doesn't fetch segments from a US server. This is critical for streaming latency. |

---

### Infrastructure

| Choice | Why |
|---|---|
| **Docker + Docker Compose** | Every service containerised. Local dev runs the full stack with one command. |
| **Kubernetes (k3s locally, EKS production)** | Container orchestration for rolling deployments, HPA on transcoding workers, health checks, and self-healing. k3s uses 512MB vs minikube's 2GB — critical on an 8GB laptop. |
| **Helm** | Templated Kubernetes manifests. Manage environment-specific config (dev vs prod) without duplicating YAML. |
| **GitHub Actions** | CI/CD — test on push, build Docker image on merge, push to ECR, deploy to EKS. |
| **Prometheus + Grafana** | Metrics collection and visualisation. Every service exposes a `/metrics` endpoint. |

---
---

## 6. High Level Design (HLD)

### Architecture Style
**Event-driven microservices.** Services communicate primarily via Kafka events, not direct HTTP calls. This means:
- Services are independently deployable
- A slow transcoding worker does not block the upload API
- Failed jobs are retried automatically via Kafka consumer group semantics
- The system is auditable — every state change is a Kafka event with a timestamp

HTTP is used for: client-to-API calls, AI worker results returned to the orchestrator, inter-service queries where a synchronous response is needed immediately.

---

### System Components

```
┌─────────────────────────────────────────────────────────┐
│                        CLIENTS                          │
│          Web (React)  ·  Mobile (future)                │
└───────────────────────────┬─────────────────────────────┘
                            │ HTTPS
┌───────────────────────────▼─────────────────────────────┐
│                     EDGE LAYER                          │
│     CloudFront CDN (HLS segments)                       │
│     WebSocket Server (upload progress, notifications)   │
└──────────────┬──────────────────────────┬───────────────┘
               │                          │
┌──────────────▼──────────────────────────▼───────────────┐
│                    API GATEWAY                          │
│   JWT Auth · Rate Limiting · Request Routing            │
│   (Fastify — single entry point for all client calls)   │
└──┬─────────┬──────────┬─────────┬────────┬─────────────┘
   │         │          │         │        │
┌──▼──┐  ┌───▼──┐  ┌────▼──┐ ┌───▼──┐ ┌───▼────────┐
│Auth │  │Video │  │Search │ │User  │ │Analytics   │
│Svc  │  │Svc   │  │Svc    │ │Svc   │ │Svc         │
└──┬──┘  └───┬──┘  └────┬──┘ └───┬──┘ └───┬────────┘
   │         │           │        │         │
   │    ┌────▼────────────────────▼─────────▼────────┐
   │    │              KAFKA EVENT BUS                │
   │    │  Topics: video.uploaded · video.transcoded  │
   │    │          video.ai-enriched · view.event     │
   │    │          notification.send                  │
   │    └────┬──────────────┬────────────────────────┘
   │         │              │
   │    ┌────▼─────┐  ┌─────▼──────────────────────┐
   │    │Transcode │  │  AI Worker Service (Python) │
   │    │Worker    │  │  Whisper · CLIP · YOLOv8    │
   │    │(Node+    │  │  Content Moderation         │
   │    │ FFmpeg)  │  └─────────────────────────────┘
   │    └──────────┘
   │
┌──▼──────────────────────────────────────────────────────┐
│                     DATA LAYER                          │
│  PostgreSQL · Redis · ClickHouse · MinIO/S3             │
└─────────────────────────────────────────────────────────┘
```

---

### Request Flow — Video Upload

```
1. Creator selects file in browser
2. Client requests a pre-signed S3 upload URL from Video Service
3. Client uploads file directly to S3 (bypasses our servers — no bandwidth bottleneck)
4. S3 triggers a webhook → Video Service creates a DB record (status: UPLOADED)
5. Video Service publishes `video.uploaded` event to Kafka
6. Transcoding Worker consumes event → pulls file from S3 → runs FFmpeg
7. FFmpeg produces 360p/720p/1080p HLS segments + master playlist
8. Worker uploads segments to S3 → updates DB (status: TRANSCODED)
9. Worker publishes `video.transcoded` event to Kafka
10. AI Worker consumes event → runs Whisper, CLIP, YOLOv8 in sequence
11. AI Worker stores results (subtitles, tags, thumbnail scores) in PostgreSQL
12. AI Worker publishes `video.ai-enriched` event
13. Video Service consumes event → sets status: PUBLISHED → video appears in feed
14. WebSocket server notifies creator: "Your video is live"
```

---

### Request Flow — Video Streaming

```
1. Viewer opens video detail page
2. Client fetches video metadata from Video Service (title, description, subtitle URL)
3. Client requests master.m3u8 URL → Video Service returns pre-signed CloudFront URL
4. hls.js fetches master.m3u8 → reads available quality levels
5. hls.js selects quality based on current bandwidth estimate
6. hls.js fetches 6-second .ts segments from CloudFront edge (cached)
7. Player buffers 2–3 segments ahead → smooth playback
8. If bandwidth drops → hls.js switches to lower quality playlist automatically
9. Every 30 seconds, client sends a view heartbeat to Analytics Service
10. Analytics Service publishes `view.event` to Kafka → consumed by ClickHouse writer
```

---

### Data Flow Diagram

```
Upload:    Creator → S3 (direct) → Kafka → Transcoder → S3 (HLS) → AI Worker → PostgreSQL
Stream:    Viewer → CloudFront → .ts segments (cached at edge)
Search:    Viewer query → Search Service → Meilisearch (keyword) + pgvector (semantic) → ranked results
Recs:      User history → embedding update (nightly batch) → pgvector similarity → feed API
Analytics: View heartbeat → Kafka → ClickHouse → dashboard queries
```

---
---

## 7. Low Level Design (LLD) — Phase 1

> LLD answers: how does each feature actually work, step by step, inside the code?
> Design LLD for what you are building now, not for the entire system.
> Feature → Steps → Functions → Data structures

---

### Feature 1 — Video Upload

**Steps:**
1. Client calls `POST /api/v1/videos/upload-url` with `{ filename, contentType, fileSize }`
2. Video Service validates: file must be video/* MIME type, max 2GB
3. Video Service generates a pre-signed S3 PUT URL (15-minute expiry) via AWS SDK
4. Video Service creates a DB record: `{ id, userId, status: 'AWAITING_UPLOAD', s3Key, createdAt }`
5. Returns `{ uploadUrl, videoId }` to client
6. Client uploads file directly to S3 using the pre-signed URL (multipart for files > 100MB)
7. Client calls `POST /api/v1/videos/:id/confirm` when upload completes
8. Video Service updates status to `UPLOADED`, publishes `video.uploaded` Kafka event
9. WebSocket sends progress updates to creator during client-side upload via `onprogress`

**Key functions:**
```
generatePresignedUploadUrl(s3Key, contentType, expirySeconds) → string
createVideoRecord(userId, s3Key, metadata) → Video
confirmUpload(videoId, userId) → void
publishVideoUploaded(videoId, s3Key) → void
```

**Why pre-signed URL instead of proxying through our server:**
If every upload went through our Node.js server, a 1GB file upload would hold one connection open for minutes. With pre-signed URLs, S3 handles the bytes and our server just issues a token. This is how YouTube and S3-based platforms actually work.

---

### Feature 2 — Video Transcoding Worker

**Steps:**
1. Kafka consumer receives `video.uploaded` event `{ videoId, s3Key }`
2. Worker checks if this videoId has already been processed (idempotency check via DB)
3. Worker downloads the raw video from S3 to a temp directory
4. FFmpeg command runs — produces three HLS renditions:
   - 360p: 640×360, 800kbps video, 96kbps audio
   - 720p: 1280×720, 2500kbps video, 128kbps audio
   - 1080p: 1920×1080, 5000kbps video, 192kbps audio
5. FFmpeg generates per-quality `.m3u8` playlist and 6-second `.ts` segments
6. Worker generates master `playlist.m3u8` referencing all three quality playlists
7. Worker uploads all files to S3 under `hls/{videoId}/`
8. Worker cleans up temp files
9. Worker updates DB: `status: TRANSCODED`, stores HLS paths
10. Worker commits Kafka offset only after successful DB update (at-least-once delivery)
11. Worker publishes `video.transcoded` event

**FFmpeg command structure:**
```bash
ffmpeg -i input.mp4 \
  -filter_complex "[v]split=3[v1][v2][v3]" \
  -map "[v1]" -map 0:a -c:v libx264 -b:v 800k -s 640x360 \
    -hls_time 6 -hls_playlist_type vod -hls_segment_filename "360p_%03d.ts" 360p.m3u8 \
  -map "[v2]" -map 0:a -c:v libx264 -b:v 2500k -s 1280x720 \
    -hls_time 6 -hls_playlist_type vod -hls_segment_filename "720p_%03d.ts" 720p.m3u8 \
  -map "[v3]" -map 0:a -c:v libx264 -b:v 5000k -s 1920x1080 \
    -hls_time 6 -hls_playlist_type vod -hls_segment_filename "1080p_%03d.ts" 1080p.m3u8
```

**Key functions:**
```
consumeVideoUploadedEvent(event) → void
checkIdempotency(videoId) → boolean
downloadFromS3(s3Key, localPath) → void
runFFmpegTranscode(inputPath, outputDir, renditions) → HLSOutput
uploadHLSToS3(outputDir, videoId) → string[]
generateMasterPlaylist(renditions) → string
updateVideoStatus(videoId, status, hlsPaths) → void
publishVideoTranscoded(videoId) → void
cleanupTempFiles(dir) → void
```

**Idempotency — why it matters:**
Kafka guarantees at-least-once delivery. If the worker crashes after uploading to S3 but before committing the Kafka offset, it will receive the same event again on restart. Without an idempotency check, the video would be transcoded twice, wasting compute and producing duplicate S3 files. The DB status check prevents this.

---

### Feature 3 — HLS Delivery

**Steps:**
1. Client calls `GET /api/v1/videos/:id/stream`
2. Video Service fetches video record from DB, verifies status is `PUBLISHED`
3. Video Service generates a pre-signed CloudFront URL for `hls/{videoId}/master.m3u8` (1-hour expiry)
4. Client receives the URL and passes it to `hls.js`
5. `hls.js` fetches `master.m3u8` — parses available quality levels
6. `hls.js` begins fetching `.ts` segments from CloudFront (cached at edge after first request)
7. Player auto-switches quality based on bandwidth (ABR algorithm built into hls.js)

**Master playlist format:**
```
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-STREAM-INF:BANDWIDTH=896000,RESOLUTION=640x360
360p.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=2628000,RESOLUTION=1280x720
720p.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=5192000,RESOLUTION=1920x1080
1080p.m3u8
```

---

### Feature 4 — Auth Service

**Steps — Registration:**
1. `POST /api/v1/auth/register` receives `{ email, password, username }`
2. Validate: email format, password min 8 chars, username unique check
3. Hash password with bcrypt (cost factor 12)
4. Insert user record in PostgreSQL
5. Send email verification link (BullMQ job → Nodemailer)
6. Return `{ message: "Verify your email" }` — no token yet

**Steps — Login:**
1. `POST /api/v1/auth/login` receives `{ email, password }`
2. Fetch user by email — if not found, return generic error (do not reveal whether email exists)
3. Compare password with bcrypt hash
4. Generate access token (JWT, 15-minute expiry, signed with RS256)
5. Generate refresh token (opaque random string, 30-day expiry)
6. Store refresh token in Redis: `refresh:{token}` → `userId` with 30-day TTL
7. Return access token in response body, refresh token in HttpOnly cookie

**Steps — Token Refresh:**
1. `POST /api/v1/auth/refresh` reads HttpOnly cookie
2. Validate refresh token exists in Redis
3. Delete old refresh token from Redis (rotation — each refresh token is single-use)
4. Generate new access token and new refresh token
5. Store new refresh token in Redis
6. Return new access token

**Why RS256 over HS256:**
RS256 uses a private/public key pair. The auth service signs with the private key. All other services verify with the public key. No service other than auth ever sees the private key. With HS256, every service that needs to verify tokens must share the secret — a security risk in a multi-service architecture.

**Why refresh token in HttpOnly cookie, not localStorage:**
JavaScript cannot read HttpOnly cookies. This means XSS attacks cannot steal the refresh token. Access tokens in memory (not localStorage) are also not persisted across tabs — a deliberate tradeoff for security.

---

### Feature 5 — Basic Search

**Steps:**
1. On video `PUBLISHED` event, Search Service indexes the video in Meilisearch
2. Index document: `{ id, title, description, tags, uploaderName, publishedAt }`
3. Client calls `GET /api/v1/search?q=cooking tutorials&page=1&limit=20`
4. Search Service queries Meilisearch with the query string
5. Meilisearch returns ranked results with highlighted matches
6. Search Service fetches thumbnail URLs for result video IDs from PostgreSQL
7. Returns paginated results to client

**Phase 2 addition — semantic search:**
When query arrives, embed it with the same sentence-transformer model used to embed videos. Query pgvector for the 50 nearest video vectors. Merge with Meilisearch keyword results using a reciprocal rank fusion score. Return the combined, re-ranked list.

---
---

## 8. Database Design

### PostgreSQL Schema

```sql
-- Users
CREATE TABLE users (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email         TEXT UNIQUE NOT NULL,
  username      TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  avatar_url    TEXT,
  role          TEXT NOT NULL DEFAULT 'viewer', -- viewer | creator | admin
  email_verified BOOLEAN DEFAULT FALSE,
  created_at    TIMESTAMPTZ DEFAULT NOW(),
  updated_at    TIMESTAMPTZ DEFAULT NOW()
);

-- Videos
CREATE TABLE videos (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  title           TEXT NOT NULL,
  description     TEXT,
  status          TEXT NOT NULL DEFAULT 'AWAITING_UPLOAD',
  -- Status flow: AWAITING_UPLOAD → UPLOADED → TRANSCODING → TRANSCODED → AI_PROCESSING → PUBLISHED | FAILED
  s3_raw_key      TEXT,                  -- original uploaded file
  hls_master_key  TEXT,                  -- path to master.m3u8 in S3
  duration_secs   INTEGER,
  thumbnail_url   TEXT,
  view_count      BIGINT DEFAULT 0,
  like_count      BIGINT DEFAULT 0,
  is_public       BOOLEAN DEFAULT TRUE,
  published_at    TIMESTAMPTZ,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_videos_user_id ON videos(user_id);
CREATE INDEX idx_videos_status ON videos(status);
CREATE INDEX idx_videos_published_at ON videos(published_at DESC);

-- Video Tags
CREATE TABLE video_tags (
  video_id  UUID REFERENCES videos(id) ON DELETE CASCADE,
  tag       TEXT NOT NULL,
  source    TEXT DEFAULT 'manual', -- manual | ai
  PRIMARY KEY (video_id, tag)
);

-- Subtitles
CREATE TABLE subtitles (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  video_id   UUID NOT NULL REFERENCES videos(id) ON DELETE CASCADE,
  language   TEXT NOT NULL,        -- en, hi, es, fr, de, ja
  vtt_s3_key TEXT NOT NULL,        -- path to .vtt file in S3
  is_auto    BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Watch History (resume position + view tracking)
CREATE TABLE watch_history (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  video_id        UUID NOT NULL REFERENCES videos(id) ON DELETE CASCADE,
  progress_secs   INTEGER DEFAULT 0,
  completed       BOOLEAN DEFAULT FALSE,
  last_watched_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, video_id)
);
CREATE INDEX idx_watch_history_user ON watch_history(user_id, last_watched_at DESC);

-- Likes
CREATE TABLE likes (
  user_id    UUID REFERENCES users(id) ON DELETE CASCADE,
  video_id   UUID REFERENCES videos(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (user_id, video_id)
);

-- Saved Videos
CREATE TABLE saved_videos (
  user_id    UUID REFERENCES users(id) ON DELETE CASCADE,
  video_id   UUID REFERENCES videos(id) ON DELETE CASCADE,
  saved_at   TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (user_id, video_id)
);

-- Video Embeddings (pgvector — Phase 2)
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE video_embeddings (
  video_id  UUID PRIMARY KEY REFERENCES videos(id) ON DELETE CASCADE,
  embedding vector(384),   -- sentence-transformer output dimension
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_video_embeddings_ivfflat
  ON video_embeddings USING ivfflat (embedding vector_cosine_ops)
  WITH (lists = 100);

-- User Embeddings (Phase 2)
CREATE TABLE user_embeddings (
  user_id    UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  embedding  vector(384),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

---

### Redis Key Patterns

| Key pattern | Value | TTL | Purpose |
|---|---|---|---|
| `refresh:{token}` | `userId` | 30 days | Refresh token store |
| `session:{userId}` | session JSON | 24 hours | Active session data |
| `rate:{ip}:{endpoint}` | request count | 1 minute | Rate limiting |
| `video:hot:{videoId}` | serialised video JSON | 5 minutes | Hot video cache |
| `feed:{userId}` | JSON array of videoIds | 10 minutes | Cached personalised feed |
| `presence:{userId}` | `1` | 5 minutes | Online presence (refreshed on activity) |

---

### Kafka Topics

| Topic | Producer | Consumer(s) | Payload |
|---|---|---|---|
| `video.uploaded` | Video Service | Transcoding Worker | `{ videoId, s3Key, userId }` |
| `video.transcoded` | Transcoding Worker | AI Worker, Video Service | `{ videoId, hlsMasterKey, duration }` |
| `video.ai-enriched` | AI Worker | Video Service | `{ videoId, subtitleKeys, tags, thumbnailUrl }` |
| `video.failed` | Transcoding Worker / AI Worker | Video Service, Notification | `{ videoId, stage, error }` |
| `view.event` | Analytics Service | ClickHouse Writer | `{ videoId, userId, progressSecs, sessionId, ts }` |
| `notification.send` | Any service | Notification Worker | `{ userId, type, payload }` |

---
---

## 9. API Design

### Conventions
- Base path: `/api/v1`
- Auth: Bearer token in `Authorization` header for all protected routes
- Errors: `{ error: { code: string, message: string, details?: any } }`
- Pagination: cursor-based — `{ data: [], nextCursor: string | null, hasMore: boolean }`
- HTTP methods used correctly — GET reads, POST creates, PATCH updates partially, DELETE removes

---

### Auth Endpoints

| Method | Path | Auth | Description |
|---|---|---|---|
| POST | `/auth/register` | None | Register new user |
| POST | `/auth/login` | None | Login, receive tokens |
| POST | `/auth/refresh` | Cookie | Rotate refresh token |
| POST | `/auth/logout` | Bearer | Invalidate refresh token |
| POST | `/auth/verify-email` | None | Verify email via token |
| POST | `/auth/forgot-password` | None | Send reset email |
| POST | `/auth/reset-password` | None | Reset with token |

---

### Video Endpoints

| Method | Path | Auth | Description |
|---|---|---|---|
| POST | `/videos/upload-url` | Bearer | Get pre-signed S3 upload URL |
| POST | `/videos/:id/confirm` | Bearer | Confirm upload complete |
| GET | `/videos/:id` | Optional | Get video metadata |
| GET | `/videos/:id/stream` | Optional | Get pre-signed HLS master URL |
| PATCH | `/videos/:id` | Bearer | Update title, description, tags |
| DELETE | `/videos/:id` | Bearer | Delete video and all assets |
| GET | `/videos/:id/subtitles` | Optional | List available subtitle tracks |
| POST | `/videos/:id/like` | Bearer | Like a video |
| DELETE | `/videos/:id/like` | Bearer | Unlike a video |
| POST | `/videos/:id/save` | Bearer | Save to library |
| POST | `/videos/:id/progress` | Bearer | Update watch progress |

---

### Feed & Discovery Endpoints

| Method | Path | Auth | Description |
|---|---|---|---|
| GET | `/feed` | Optional | Homepage feed (personalised if auth) |
| GET | `/feed/trending` | None | Trending videos (last 24 hours) |
| GET | `/search?q=&page=&limit=` | Optional | Search videos |
| GET | `/videos/:id/recommendations` | Optional | Related videos |

---

### User Endpoints

| Method | Path | Auth | Description |
|---|---|---|---|
| GET | `/users/:id` | None | Public profile |
| GET | `/users/me` | Bearer | Own profile + private data |
| PATCH | `/users/me` | Bearer | Update profile |
| GET | `/users/me/videos` | Bearer | Own uploaded videos |
| GET | `/users/me/library` | Bearer | Saved videos |
| GET | `/users/me/history` | Bearer | Watch history |

---

### Analytics Endpoints (Phase 3)

| Method | Path | Auth | Description |
|---|---|---|---|
| GET | `/analytics/videos/:id` | Bearer (owner) | Per-video stats |
| GET | `/analytics/overview` | Bearer (creator) | Creator dashboard summary |

---
---

## 10. Component & Module Breakdown

### Repository Structure

```
streamforge/
│
├── apps/
│   ├── api-gateway/          # Fastify — routing, auth middleware, rate limiting
│   ├── video-service/        # Video CRUD, upload URL generation, status updates
│   ├── auth-service/         # Registration, login, token management
│   ├── search-service/       # Meilisearch + pgvector query handling
│   ├── analytics-service/    # View event ingestion, Kafka → ClickHouse
│   ├── notification-service/ # Email + in-app notification dispatch
│   ├── transcoding-worker/   # Kafka consumer + FFmpeg orchestration
│   ├── ai-worker/            # Python FastAPI — Whisper, CLIP, YOLOv8
│   └── websocket-server/     # Upload progress, real-time notifications
│
├── packages/
│   ├── shared-types/         # TypeScript interfaces shared across services
│   ├── kafka-client/         # Shared Kafka producer/consumer setup
│   ├── db-client/            # Shared PostgreSQL connection pool
│   ├── redis-client/         # Shared Redis client
│   └── logger/               # Shared Pino logger config
│
├── frontend/
│   ├── src/
│   │   ├── components/       # Reusable UI components
│   │   ├── pages/            # Route-level page components
│   │   ├── hooks/            # Custom React hooks (usePlayer, useUpload, useAuth)
│   │   ├── store/            # Zustand stores
│   │   ├── api/              # TanStack Query hooks + Axios config
│   │   └── utils/            # Helpers, formatters
│   └── public/
│
├── infrastructure/
│   ├── docker-compose.yml    # Full local dev stack
│   ├── helm/                 # Helm charts per service
│   └── terraform/            # AWS infrastructure as code (Phase 3)
│
├── scripts/
│   ├── seed.ts               # DB seed data for local dev
│   └── load-test/            # k6 scripts
│
└── .github/
    └── workflows/            # CI/CD pipeline definitions
```

---

### Key Module Responsibilities

**api-gateway**
- Single entry point for all client requests
- JWT validation middleware (verifies RS256 signature with public key)
- Rate limiting middleware (Redis-backed, per IP and per user)
- Request routing to downstream services via HTTP
- No business logic — pure routing and auth

**video-service**
- Pre-signed URL generation for uploads
- Video CRUD operations
- Status state machine management
- Kafka event publication for upload events
- HLS URL generation for streaming

**transcoding-worker**
- Kafka consumer for `video.uploaded` events
- FFmpeg orchestration (spawned as child process)
- S3 download and upload management
- Idempotency checking via DB
- Error handling and `video.failed` event publication

**ai-worker (Python FastAPI)**
- Receives Kafka events via a Python Kafka consumer (confluent-kafka)
- Runs Whisper for transcription
- Runs CLIP for thumbnail scoring
- Runs YOLOv8 for object detection
- Calls translation API for multi-language subtitles
- Stores results in PostgreSQL via async SQLAlchemy
- Publishes `video.ai-enriched` event

**websocket-server**
- Maintains per-user WebSocket connections
- Subscribes to Redis pub/sub channels for user-specific events
- Pushes upload progress, processing status, and notification events to connected clients

---
---

## 11. Engineering Considerations

### Edge Cases

| Scenario | How it is handled |
|---|---|
| User uploads a corrupt or non-video file | MIME type validated on upload-url request. FFmpeg probe step validates file before transcoding begins. Status set to FAILED with error message. |
| Transcoding worker crashes mid-job | Kafka offset not committed until DB is updated. On restart, consumer receives the same event. Idempotency check detects the videoId is already being processed and skips or resumes. |
| Two workers consume the same Kafka event | Consumer group semantics — Kafka delivers each event to exactly one consumer in a group. Not a risk unless partitions are misconfigured. |
| AI worker times out on a 2-hour video | Whisper runs in chunks. Worker implements a heartbeat to Kafka to prevent session timeout. Dead-letter topic for events that fail after 3 retries. |
| User deletes video while transcoding is in progress | Video Service marks record as DELETED. Transcoding worker checks status before uploading results — if DELETED, it cleans up S3 files and stops. |
| Duplicate upload confirmation request | `confirm` endpoint is idempotent — if status is already past UPLOADED, it returns 200 without re-publishing the Kafka event. |
| CDN cache stale after video deletion | CloudFront cache invalidation issued for `hls/{videoId}/*` on video deletion. Pre-signed URL expiry (1 hour) limits window of access anyway. |
| 8GB laptop runs out of RAM in local dev | Meilisearch instead of Elasticsearch (saves 1GB). AI worker started on-demand, not always running. Kafka in KRaft mode (no ZooKeeper). Docker memory limits set per container. |

---

### Tradeoffs & Decisions

| Decision | What was given up | Why it is the right call |
|---|---|---|
| Pre-signed S3 upload instead of server proxy | Server doesn't see raw bytes, harder to validate file content mid-upload | Avoids upload bottleneck entirely. Validation before and after (MIME check + FFmpeg probe) is sufficient. |
| Kafka over direct HTTP between services | Added operational complexity, Kafka cluster to manage | Event replay, at-least-once delivery, consumer group scaling, and decoupling are worth it for this use case. |
| Node.js + Python split (not Node.js for everything) | Two runtimes to manage | ML libraries are Python-native. The alternative (TensorFlow.js) is less mature and not what production AI systems use. |
| Meilisearch first, Elasticsearch later | Less powerful search in Phase 1 | Meilisearch is operational in 10 minutes. Elasticsearch takes 2 hours to configure correctly. Swap when load testing reveals the need. |
| No auth in transcoding worker | Worker trusts Kafka events — no verification of caller identity | Kafka is internal infrastructure, not exposed to the internet. mTLS between services handles this in Phase 3. |
| pgvector over Pinecone for embeddings | Less specialised, lower query performance at extreme scale | Avoids a paid external service dependency. pgvector handles millions of vectors comfortably — more than enough for this project's scale. |
| Cursor pagination over offset pagination | Slightly more complex implementation | Offset pagination breaks when new items are inserted while paginating. Cursor pagination is stable and performant at scale. |

---
---

## 12. Non-Functional Requirements

| Category | Requirement | How it is met |
|---|---|---|
| **Performance** | Video playback starts within 3 seconds | CDN edge caching of first HLS segment. hls.js preloads 2 segments. |
| **Performance** | API p95 response time < 200ms | Fastify (fastest Node.js framework), Redis caching for hot data, DB indexes on all query paths. |
| **Scalability** | Support 1000 concurrent streamers | HLS segments served by CDN — zero load on origin servers per stream. API Gateway and services scale horizontally via K8s HPA. |
| **Scalability** | Transcoding queue handles spikes | HPA scales transcoding worker pods based on Kafka consumer group lag metric. |
| **Reliability** | No video upload is silently lost | Kafka event retention (7 days). At-least-once delivery. Dead-letter topic after 3 retries. Alerting on dead-letter topic messages. |
| **Reliability** | System recovers from single service failure | K8s restarts failed pods automatically. Kafka retains events for consumers to catch up. Redis and PostgreSQL have health probes. |
| **Security** | Auth tokens cannot be stolen via XSS | Refresh token in HttpOnly cookie. Access token in memory only. |
| **Security** | Rate limiting on all public endpoints | Redis-backed rate limiter in API Gateway — 100 req/min per IP, 1000 req/min per authenticated user. |
| **Security** | Content is not publicly accessible without auth | S3 bucket is private. All access via pre-signed URLs with short expiry. CloudFront uses signed URLs. |
| **Observability** | Every failure is detectable | Prometheus alert on: Kafka consumer lag > 1000, error rate > 1%, p95 latency > 500ms, dead-letter topic non-empty. |
| **Maintainability** | Code is understandable by a new developer | Monorepo with shared packages. Consistent error handling pattern across services. Every Kafka event and API endpoint documented here. |

---
---

## 13. Project Timeline

**Working hours:** ~2.5–3 hours/day on weekdays, ~3 hours/day on weekends (~18–20 hours/week)

| Phase | Focus | Duration | Milestone |
|---|---|---|---|
| **Phase 1** | Core pipeline — upload, transcode, stream | Weeks 1–8 | Video uploaded → HLS playing in browser |
| **Phase 2** | AI enrichment layer | Weeks 9–16 | Whisper subtitles, CLIP thumbnails, semantic search working |
| **Phase 3** | Kubernetes, observability, load testing | Weeks 17–22 | Deployed on K8s, Grafana dashboard live, k6 load test passed |
| **Phase 4** | Advanced features, polish, demo | Weeks 23–26 | Public demo, architecture doc, recorded walkthrough |

### Phase 1 Weekly Breakdown

| Week | Tasks |
|---|---|
| 1–2 | Monorepo setup · Docker Compose · PostgreSQL + Redis · Auth service (register, login, JWT) |
| 3–4 | Video service · S3/MinIO integration · Pre-signed upload URL · FFmpeg single-quality HLS ✦ |
| 5–6 | Kafka setup · Upload → transcode pipeline · Adaptive bitrate (3 qualities) · hls.js player ✦ |
| 7–8 | Meilisearch setup · Basic search · Video feed · Frontend UI · Buffer + debug time |

✦ = hardest weeks — budget extra debugging time here

### Phase 2 Weekly Breakdown

| Week | Tasks |
|---|---|
| 9–11 | Python AI worker setup · Whisper transcription · Subtitle storage + VTT serving · CLIP thumbnail scoring |
| 12–14 | pgvector setup · Video embeddings · Semantic search · User embeddings · Recommendation feed |
| 15–16 | Content moderation · Watch history + resume · Like + save · Buffer + end-to-end polish |

---
---

## 14. Future Scope & Learning Notes

### What this project teaches — by phase

**Phase 1 teaches:**
Event-driven architecture, Kafka producer/consumer patterns, FFmpeg and video processing, HLS protocol and adaptive bitrate streaming, pre-signed URL patterns, S3 object storage, at-least-once delivery and idempotency, JWT auth with refresh token rotation

**Phase 2 teaches:**
LLM/ML model integration (Whisper, CLIP, YOLOv8), multi-service orchestration via Kafka, vector embeddings and similarity search, recommendation system fundamentals, content moderation pipeline design

**Phase 3 teaches:**
Kubernetes deployment and Helm charts, horizontal pod autoscaling, Prometheus metrics and Grafana dashboards, ClickHouse for analytical queries, k6 load testing and bottleneck identification, CloudFront CDN configuration, CI/CD with GitHub Actions

---

### Interview talking points this project enables

- *"Walk me through what happens when a user uploads a video"* — full async pipeline, 9 steps, Kafka events, FFmpeg, HLS, AI enrichment
- *"How would you scale this to 10 million users?"* — CDN for segment delivery, Kafka consumer group scaling, read replicas, Redis caching strategy, ClickHouse for analytics
- *"What happens if the transcoding service crashes?"* — Kafka offset uncommitted, consumer group resumes, idempotency prevents duplicate work
- *"Why did you choose Kafka over a simple job queue?"* — event replay, consumer group model, decoupling, the specific scenario where RabbitMQ would have lost the event
- *"How does your recommendation system work?"* — user embedding built from watch history, cosine similarity against video vectors, nightly refresh batch, cold start problem handling
- *"What was the hardest problem you faced?"* — answer from real debugging experience during Phases 1–2

---

### Things to research as you build (not before)

- HLS spec (RFC 8216) — read it when you're implementing the master playlist
- Kafka consumer group rebalancing — read when you add a second transcoding worker
- pgvector IVFFlat vs HNSW index — read when semantic search feels slow
- CloudFront signed URL vs signed cookies — read when implementing CDN
- FFmpeg libx264 encoding parameters — read when optimising transcode quality/size ratio
- ClickHouse MergeTree engine — read when setting up the analytics schema

---

### Future features beyond Phase 4

- Live streaming via RTMP ingest + HLS output (completely separate pipeline)
- Shorts / 60-second clip creation from any video segment
- Creator monetisation (ad insertion into HLS stream via SSAI)
- Multi-CDN failover (CloudFront primary, Fastly fallback)
- ML-based content moderation (replace rule-based classifier)
- Mobile app (React Native + Expo)
- Chapter detection (auto-detect scene changes → suggest chapter markers)
- Multi-language audio tracks (dub entire video, not just subtitles)

---

### Key concepts to understand before each phase

**Before Phase 1:** HTTP range requests (how video players fetch partial files), HLS specification basics, S3 presigned URL security model, Kafka fundamentals (topics, partitions, consumer groups, offsets)

**Before Phase 2:** How Whisper works (encoder-decoder transformer, audio chunking), what an embedding is and why cosine similarity works for recommendation, how YOLOv8 differs from classification models

**Before Phase 3:** Kubernetes resource model (requests vs limits), Prometheus data model (counters, gauges, histograms), what a p95 latency means and why it matters more than average, ClickHouse columnar storage model

---

*Document status: Living document. Update when decisions change. Never let the code diverge silently from this document.*
*Next update trigger: When Phase 1 Week 1 begins — confirm tech stack choices are final.*