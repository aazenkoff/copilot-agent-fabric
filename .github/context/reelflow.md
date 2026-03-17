# ReelFlow — Project Context

> **Audience:** AI agents. Be precise. No fluff.
> **Last updated:** 2025-03

---

## Identity

| Field | Value |
|---|---|
| **Project** | ReelFlow |
| **Repo** | https://github.com/AZenkIO/reelflow |
| **Local path** | `/Users/azenkov/Develop/projects/reelflow` |
| **Purpose** | Video editing SaaS — upload, caption, burn subtitles, remix, publish |

---

## Architecture (monorepo)

| App / Package | Stack | Notes |
|---|---|---|
| `apps/api/` | Express.js 5 + Prisma 7 + PostgreSQL 16 | TypeScript, CommonJS |
| `apps/ai-service/` | FastAPI + ffmpeg + Whisper + GPT-4o-mini | Python |
| `apps/web/` | Next.js 16 + React 19 + Tailwind CSS | TypeScript |
| `packages/shared/` | Shared TypeScript types | |

---

## Key Conventions

### API Error Handling
- Use `HttpError(status, message)` from `src/lib/errors.ts`.
- All routes must call `next(err)` — never throw or respond directly in catch blocks.

### Ownership Checks
- Always verify the chain: `video → project → userId`.
- Never trust `videoId` alone; confirm the video belongs to the authenticated user's project.

### Serialization
- `serializeVideo()` in `src/lib/serializers.ts` converts `BigInt fileSize` → `Number` for safe JSON serialization.
- Always use this serializer before sending any video object in a response.

### Video Types
- `'source'` — original uploaded video.
- `'processed'` — output of a burn/merge operation; has `sourceVideoId` pointing back to the source.

### API Response Shape
- Always wrap responses: `{ video }` or `{ videos }`.
- Never return bare arrays.

### Async Jobs
- Long-running operations (e.g., subtitle burn) return `202` with `{ jobId }`.
- Client polls `GET /api/jobs/:jobId` until `status === 'completed'` or `'failed'`.

### Migrations
- Located in `apps/api/prisma/migrations/`.
- File naming: `YYYYMMDDHHMMSS_<descriptive_name>/migration.sql`.

### Git Conventions
- **Commit style:** Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`, etc.).
- **Co-author trailer** on every commit:
  ```
  Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>
  ```
- **Branch naming:** `copilot/<type>/<slug>` — e.g., `copilot/feat/my-feature`, `copilot/fix/bug-name`.
- **PRs:** Always open for manual review. Agents **never** merge PRs.

---

## Database Schema

### `users`
| Column | Type |
|---|---|
| id | UUID PK |
| email | String (unique) |
| passwordHash | String |
| name | String |

### `projects`
| Column | Type |
|---|---|
| id | UUID PK |
| userId | FK → users |
| title | String |
| description | String? |
| status | String |

### `videos`
| Column | Type | Notes |
|---|---|---|
| id | UUID PK | |
| projectId | FK → projects | |
| type | String | `'source'` or `'processed'` |
| filePath | String | |
| originalName | String | |
| fileSize | BigInt | Serialize with `serializeVideo()` |
| durationSeconds | Float? | |
| width | Int? | |
| height | Int? | |
| createdAt | DateTime | |
| sourceVideoId | FK → videos (self) | Added in migration `20250317100000` |

### `captions`
| Column | Type | Notes |
|---|---|---|
| id | UUID PK | |
| videoId | FK → videos | |
| content | JSONB | Array of caption segments |
| language | String | |

### `jobs`
| Column | Type | Notes |
|---|---|---|
| id | UUID PK | |
| type | String | e.g., `'subtitle_burn'` |
| status | String | `pending` / `processing` / `completed` / `failed` |
| videoId | FK → videos | |
| userId | FK → users | |
| payload | JSONB | Input params |
| result | JSONB? | Output data |
| progress | Int | 0–100 |
| attempts | Int | |
| lockedBy | String? | Worker instance ID |

### `remixes`
| Column | Type |
|---|---|
| id | UUID PK |
| projectId | FK → projects |
| title | String |
| status | String |
| outputVideoId | FK → videos? |

### `remix_clips`
| Column | Type |
|---|---|
| id | UUID PK |
| remixId | FK → remixes |
| videoId | FK → videos |
| orderIndex | Int |
| startTime | Float |
| endTime | Float |
| transitionType | String? |

### `subtitle_presets`
| Column | Type | Notes |
|---|---|---|
| id | UUID PK | |
| userId | FK → users | |
| name | String | |
| style | JSONB | Style configuration object |
| isBuiltIn | Boolean | |

### `publications`
| Column | Type |
|---|---|
| id | UUID PK |
| projectId | FK → projects |
| videoId | FK → videos |
| platform | String |
| status | String |

---

## Video Burn Flow

```
POST /api/videos/:videoId/subtitles/burn
  → creates Job (type='subtitle_burn', status='pending')
  → 202 { jobId }

AI worker (apps/ai-service/app/worker.py)
  → polls jobs table for pending jobs
  → runs ffmpeg with caption overlay
  → inserts new videos row (type='processed', sourceVideoId=<source>)
  → updates job status='completed', result={ videoId }

Client
  → polls GET /api/jobs/:jobId
  → on completed: re-fetches source video
  → source video response now includes burnedVideos[]
```

---

## Active Feature: Burned Video Linking + Video Rename

**Branch:** `copilot/feat/burned-video-linking`

### What's being built
1. **`source_video_id` FK on `videos`** — self-relation `BurnedVideos` (one source → many processed).
2. **`PATCH /api/videos/:id`** — rename endpoint; updates `originalName`.
3. **Source video API response** — includes `burnedVideos[]` array of processed video objects.
4. **Frontend** — "Burned Versions" list on the source video panel.
5. **Inline rename UI** — for all videos (source and processed).

### Migration added
`apps/api/prisma/migrations/20250317100000_add_source_video_id/migration.sql`

---

## Key Files Reference

| File | Purpose |
|---|---|
| `apps/api/prisma/schema.prisma` | Prisma schema — source of truth for DB model |
| `apps/api/src/lib/errors.ts` | `HttpError` class |
| `apps/api/src/lib/serializers.ts` | `serializeVideo()` — BigInt-safe JSON serialization |
| `apps/api/src/services/video.service.ts` | Video CRUD + ownership checks |
| `apps/api/src/routes/videos.routes.ts` | Video REST routes |
| `apps/api/src/routes/ai.routes.ts` | Caption + burn endpoints |
| `apps/ai-service/app/worker.py` | Async job worker (ffmpeg, Whisper) |
| `apps/web/src/types/index.ts` | Frontend TypeScript types |
| `apps/web/src/app/dashboard/projects/[id]/page.tsx` | Main project page (videos, burn, captions) |
| `apps/web/src/lib/api.ts` | `apiFetch` helper |

---

## Quick Reference: Agent Checklist

Before writing any API route or service method:

- [ ] Is the `HttpError` pattern used for all errors?
- [ ] Is ownership verified (`video → project → userId`)?
- [ ] Is `serializeVideo()` called before sending the response?
- [ ] Does the response use `{ video }` or `{ videos }` wrapper?
- [ ] If async, does it return `202 { jobId }` and use the jobs table?
- [ ] Does any new migration follow `YYYYMMDDHHMMSS_<name>` naming?
- [ ] Is the git branch named `copilot/<type>/<slug>`?
- [ ] Is the commit a conventional commit with the `Co-authored-by` trailer?
- [ ] Is the PR left open for manual review (not merged)?
