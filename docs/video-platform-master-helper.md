# Video Platform — Master Helper

A living guide for learning Django, FastAPI, Vite + React, and real-world engineering by building a small YouTube-like platform. Keep this file in the repo root and update it as you go.

---

## 0. How to use this file

- Work on **one milestone at a time**. Ignore everything below your current milestone.
- Each milestone has a **Definition of Done**. If it isn't met, you're not done.
- Tools are added only when the **trigger** in section 4 happens. Not before.
- Update the progress log (section 11) at the end of every session.

---

## 1. The rules

1. **Write first, ask AI second.** Try for 20-30 minutes. Then use AI to review, explain an error, or compare approaches. Don't let it write whole features.
2. **End every session with something that runs.** Even if it's tiny.
3. **Small and polished beats big and half-done.** Good, not perfect.
4. **Fixed daily slot.** 1 hour daily beats 6 hours once a week.
5. **Read the official docs first.** Videos and blogs only to fill gaps.
6. **Commit often, in small commits, with clear messages.** Use branches and PRs even when working alone.
7. **No new tool, framework, or rewrite mid-milestone.**

---

## 2. Project vision

**Goal:** a small video sharing platform where users register, upload videos, and watch them in a browser (later: as a PWA and a mobile app).

**Core features (v1):** auth, upload, video list, video page with player, processing status.

**Not in v1:** comments, likes, recommendations, live streaming, monetization, subscriptions. Add them later, one at a time, or never.

**Honest framing:** this is a learning and portfolio project, not a YouTube competitor. Video hosting is expensive and the market is taken. The value is the skills and the polished, deployed result.

---

## 3. Roadmap

### M0 — Project setup (2-3 days)
- [ ] Git repo, `.gitignore`, README skeleton
- [ ] Folder layout: `backend/`, `frontend/`, `docker-compose.yml`
- [ ] PostgreSQL running via Docker Compose
- [ ] `.env` for config (never committed), `.env.example` committed
- [ ] `ruff` + `pytest` installed
- [ ] GitHub Actions: lint + tests on every push

**Done when:** `docker compose up` starts the database and CI is green on an empty test.

### M1 — Django API (1-2 weeks)
- [ ] Django project, custom `User` model from day one
- [ ] `Video` model: title, description, file, owner, status, created_at
- [ ] Django REST Framework: list, detail, create, delete
- [ ] Auth: register, login (JWT via `djangorestframework-simplejwt`)
- [ ] Permissions: only the owner can edit or delete
- [ ] File upload with validation (type, size)
- [ ] Pagination
- [ ] Tests for auth, permissions, upload

**Done when:** you can register, log in, upload a video, and list it using only an API client (Bruno/Postman), and tests pass.

### M2 — Vite + React frontend (1-2 weeks)
- [ ] Vite + React (TypeScript recommended)
- [ ] Routing (React Router)
- [ ] Pages: video list, video page with `<video>` player, upload form, login/register
- [ ] API client layer (one place that talks to the backend)
- [ ] Auth flow: token handling, protected routes, logout
- [ ] Loading and error states
- [ ] Responsive layout (works on a phone-sized screen)

**Done when:** the full flow works in the browser: register, log in, upload, watch.

### M3 — Background processing (2-3 weeks)
- [ ] Redis in Docker Compose
- [ ] Celery worker in Docker Compose
- [ ] After upload: task generates a thumbnail and transcodes with **ffmpeg** to **HLS**
- [ ] Video `status`: uploaded → processing → ready / failed
- [ ] Frontend shows status and plays HLS (e.g. `hls.js`)
- [ ] Files stored in **MinIO** (S3-compatible) instead of the app disk
- [ ] Nginx serving video segments

**Done when:** uploading a video produces a streamable HLS version and the UI updates when it's ready.

### M4 — FastAPI service + shared auth (2-3 weeks)
- [ ] Move one real responsibility into a FastAPI service (e.g. processing jobs API or a stats/search service)
- [ ] Service verifies the same JWTs issued by Django (RS256 with public key)
- [ ] Both services in Docker Compose
- [ ] Consider RabbitMQ only if you need retries, guaranteed delivery, or routing
- [ ] Tests for the new service

**Done when:** a request with a Django-issued token is accepted by FastAPI, and an expired or tampered token is rejected.

### M5 — Deploy (1-2 weeks)
- [ ] Linux VPS, Docker Compose in production mode
- [ ] Nginx reverse proxy, HTTPS via Let's Encrypt
- [ ] Gunicorn (Django), Uvicorn (FastAPI)
- [ ] `DEBUG=False`, secrets from environment, allowed hosts set
- [ ] Error tracking (Sentry) and basic logs
- [ ] Database backups
- [ ] README with setup instructions, architecture diagram, short demo video

**Done when:** a stranger can open your URL, register, upload, and watch.

### M6 — PWA (3-5 days)
See section 7.

### M7 — Mobile app (later, 3-6 weeks)
See section 8.

### M8 — ML features (later)
- [ ] Auto-transcription (Whisper) as a background job
- [ ] Auto-tags or search over transcripts
- [ ] Simple recommendations from watch history

---

## 4. Tool triggers: add it when this happens

| Tool | Add it when... |
|---|---|
| Docker / Compose | From day one (consistent environments) |
| PostgreSQL | From day one |
| GitHub Actions | M0, once you have a single test |
| Redis | You need background jobs or caching (M3) |
| Celery | A task takes more than a second or two (video processing) |
| MinIO / S3 storage | Files shouldn't live on the app server (M3) |
| Nginx | You serve static/video files or deploy (M3/M5) |
| FastAPI service | You have a separate responsibility worth isolating (M4) |
| RabbitMQ | You need retries, guaranteed delivery, or routing between services |
| Prometheus + Grafana | You're live and want metrics (optional) |
| Kubernetes | You have real scaling needs. Not for this project, only read about it |
| Kafka | Not for this project |

---

## 5. Security checklist

### JWT essentials
- [ ] Structure: header.payload.signature. The payload is **encoded, not encrypted**. No secrets in it.
- [ ] Signing: HS256 (shared secret) vs RS256 (private key signs, public key verifies). Use RS256 for multiple services.
- [ ] Always validate `exp`; pin the allowed algorithm; never accept `alg: none`.
- [ ] Short-lived access token (about 5-15 min) + refresh token with rotation.
- [ ] Web storage: refresh token in an `httpOnly`, `Secure`, `SameSite` cookie. Avoid `localStorage` for tokens.
- [ ] Mobile storage: Keychain / Keystore (via secure-store libraries), never plain storage.
- [ ] Revocation: understand rotation and blocklists (JWTs can't be cancelled by default).
- [ ] Libraries: `djangorestframework-simplejwt` (Django), `PyJWT` (FastAPI).

### General
- [ ] Password hashing (Django default is fine; never roll your own)
- [ ] Upload validation: check real content type and size; never trust the file extension; don't serve uploads from a path that can execute code
- [ ] CORS configured to your actual frontend origins only
- [ ] CSRF understood (matters when using cookies)
- [ ] Rate limiting on login and upload endpoints
- [ ] Secrets only in environment variables or a secrets manager
- [ ] Walk through the **OWASP Top 10** once as a review checklist
- [ ] Dependencies updated; `pip-audit` / `npm audit` occasionally

---

## 6. API decisions to make now (so web, PWA, and mobile all work later)

- Version the API from the start: `/api/v1/...`
- Auth must work for **non-browser clients**: support bearer tokens, not only cookies.
- Use **cursor pagination** for video lists (stable and fast).
- Return **HLS** manifests for playback, since both browsers and mobile players handle HLS well.
- Consider **resumable uploads** (tus protocol) later; mobile networks drop often.
- Generate an **OpenAPI schema** (`drf-spectacular` for Django; FastAPI does it automatically) and generate a TypeScript client from it. The web app and the mobile app can then share the same typed client.
- Keep business logic in the backend, not in the frontend, so every client behaves the same.

---

## 7. PWA notes

**What it is:** your website plus a manifest and a service worker, so it can be installed and work partly offline.

**How:** Vite + `vite-plugin-pwa` (it uses Workbox). You need HTTPS, a web app manifest (name, icons, theme color), and a service worker strategy.

**What to cache:** the app shell (HTML/JS/CSS), icons, and maybe API responses for the video list. **Don't cache large video files** in the service worker.

**Useful web APIs for a video app:** Media Session API (lock-screen controls), Picture-in-Picture, Web Share.

**Limits to know:**
- iOS support is more limited: install is manual (Add to Home Screen), push notifications work only for installed PWAs on recent iOS versions, background sync is not available, storage can be cleared.
- Offline video downloads are limited compared to a native app.

**Checklist:**
- [ ] Manifest + icons (including maskable)
- [ ] Service worker with app shell caching
- [ ] Offline fallback page
- [ ] Installable on Android Chrome
- [ ] Lighthouse PWA/performance audit reviewed

---

## 8. Mobile app options

| Option | Language | Reuse from web | Control | Notes |
|---|---|---|---|---|
| **React Native** | TypeScript | High (React knowledge, API client, types) | Good, real native views | Best fit for this project. Android + iOS from one codebase |
| **Capacitor** | TypeScript | Very high (wraps your Vite web app) | Limited to web UI + plugins | Fastest route to store apps; good first step |
| **Kotlin + Jetpack Compose** | Kotlin | Low | Maximum on Android | Android only. Use Media3/ExoPlayer for video |
| **Kotlin Multiplatform / Compose Multiplatform** | Kotlin | Low | High | Shares code between Android and iOS; ecosystem is younger |
| **Flutter** | Dart | None | High | Solid framework, but you'd leave the React/TypeScript ecosystem |
| **JavaFX** | Java | None | Desktop only | A desktop UI toolkit, not a practical mobile choice |

**Clarification:** React Native is **not** a Kotlin interpreter. You write JavaScript/TypeScript, which runs on the Hermes engine and talks to native Android/iOS UI components. You don't need to know Kotlin to use it, though you may touch native code for rare native modules.

**Recommended ladder:**
1. API (M1-M4)
2. Responsive web (M2)
3. PWA (M6)
4. React Native app (M7), reusing the typed API client
5. Optional: Capacitor first, if you want store presence quickly

**React Native notes:**
- Start with Expo if you want speed; use a development build / prebuild when you need native modules. The bare CLI is fine too, you just manage more setup yourself.
- Video: `expo-video` or `react-native-video` (both play HLS).
- Tokens: store in Keychain/Keystore.

**Checklist:**
- [ ] App shell with navigation
- [ ] Login with token stored securely
- [ ] Video list and HLS playback
- [ ] Upload from the camera roll
- [ ] Release build on a real Android device

---

## 9. Learning resources (official docs first)

- Django: docs.djangoproject.com (start with the tutorial)
- Django REST Framework: django-rest-framework.org
- FastAPI: fastapi.tiangolo.com (the tutorial section)
- React: react.dev
- Vite: vite.dev
- Docker: docs.docker.com (Get Started, Compose)
- PostgreSQL: postgresql.org/docs
- MDN Web Docs: developer.mozilla.org (HTTP, service workers, media)
- OWASP: owasp.org (Top 10, Cheat Sheet Series)
- jwt.io for inspecting tokens while learning

---

## 10. Weekly rhythm and session template

**Weekly:**
- 5-6 sessions of about 1-2 hours
- 1 day off
- End of week: 15-minute review (what shipped, what blocked, what's next week's single goal)

**Session template:**
1. Read the milestone checklist, pick **one** item (2 min)
2. Write down what you expect the code to do (2 min)
3. Build it yourself (45-90 min)
4. If stuck over 30 min: ask AI to explain or review, not to write
5. Run it, commit it, log it (5 min)

---

## 11. Logs

### Progress log
```
YYYY-MM-DD | Milestone | What I built | What blocked me | Next
```

### Decision log
```
YYYY-MM-DD | Decision | Why | Alternatives rejected
```

---

## 12. What "job-ready" means for this project

- [ ] Deployed at a real URL with HTTPS
- [ ] Clear README: what it is, architecture diagram, how to run it locally, tech choices and why
- [ ] Tests that run in CI
- [ ] Clean Git history with meaningful commits
- [ ] A 2-3 minute demo video
- [ ] A short write-up of one hard problem you solved (e.g. video processing pipeline, shared JWT auth between services)
- [ ] You can explain every part of it without AI's help
