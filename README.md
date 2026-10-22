# Bugtrail

A small but real issue tracker: a REST API plus a minimal web UI. It is
deliberately not a toy — it has existing features, tests, and conventions, so
working on it feels like landing in an unfamiliar codebase at your job.

You will use this repository for the **Claude Code for Developers capstone**.
Pick **one** feature from the roadmap below and build it end to end using the
ten-step workflow from the lesson.

---

## Setup

Requires **Node 18.17+**.

```bash
npm install
npm start          # serves the UI + API at http://localhost:3000
npm test           # runs the end-to-end test suite
npm run lint       # eslint
```

Once dependencies install, make a clean baseline commit so you have something to
diff against during the capstone:

```bash
git init
git add -A
git commit -m "chore: import Bugtrail starter"
```

Open [http://localhost:3000](http://localhost:3000) to see six seeded issues.

## Architecture

- `src/app.js` — Express app factory (`createApp`): JSON parsing, request
  logging, routes, and the 404 / error handlers.
- `src/server.js` — entry point; reads `PORT`, starts the server.
- `src/routes/issues.js` — the issue endpoints.
- `src/db.js` — a file-backed datastore (the only module that touches disk).
- `src/lib/validate.js` — validation and the allowed `priority` / `status` values.
- `src/lib/logger.js` — the only place `console.*` is allowed.
- `public/` — the web UI the API serves (`index.html`, `ui.js`, `styles.css`).

See [CONTRIBUTING.md](./CONTRIBUTING.md) for the conventions this codebase
follows. Read it before you write code — the grader rewards a CLAUDE.md that
captures conventions you actually noticed here.

## Existing API

| Method | Path                     | Notes                                             |
| ------ | ------------------------ | ------------------------------------------------- |
| GET    | `/api/health`            | `{ ok: true }`                                    |
| GET    | `/api/issues`            | supports `?status=` and `?priority=` filters      |
| GET    | `/api/issues/:id`        | `404` if missing                                  |
| POST   | `/api/issues`            | creates an issue; validates title/priority        |

An issue looks like:

```json
{
  "id": 1,
  "title": "Login button unresponsive on Safari",
  "description": "...",
  "priority": "high",
  "status": "open",
  "createdAt": "2026-03-01T09:12:00.000Z"
}
```

`priority` is one of `low | medium | high`. `status` is one of
`open | in_progress | resolved | closed`.

---

## Roadmap — pick ONE for your capstone

Each of these is a realistic next feature. Each has edge cases worth surfacing in
**plan mode** before you write code, and each is small enough to finish, test,
and verify in one session. Do **not** build more than one — depth beats breadth.

### 1. Change an issue's status
`PATCH /api/issues/:id/status` moving an issue between statuses.
- Allowed transitions only: `open → in_progress → resolved → closed`, and
  `resolved`/`closed` may reopen to `open`. Reject anything else.
- Edge cases: unknown status, illegal transition, missing issue, no-op (same status).
- Verify in the UI or with a browser-automation MCP that the badge updates.

### 2. Comments on an issue
`GET`/`POST /api/issues/:id/comments`.
- A comment has an author, a non-empty body, and a timestamp.
- Edge cases: commenting on a missing issue, empty body, comment ordering
  (newest vs oldest), commenting on a `closed` issue (allow or block? decide and
  document it).

### 3. Search and filter
Extend `GET /api/issues` with `?q=` (matches title/description) plus pagination
(`?page=`, `?pageSize=`).
- Edge cases: empty query, case-insensitivity, page past the end, combining `q`
  with the existing `status`/`priority` filters, an upper bound on `pageSize`.

### 4. Assignees
Add an optional `assignee` to issues, an endpoint to assign/unassign, and an
`?assignee=` filter.
- Edge cases: assigning on a missing issue, unassigning when already unassigned,
  filtering for "unassigned", validating the assignee format.

### 5. Soft delete and restore
`DELETE /api/issues/:id` marks an issue deleted; `POST /api/issues/:id/restore`
brings it back. Deleted issues are excluded from the list unless
`?includeDeleted=true`.
- Edge cases: deleting an already-deleted issue, restoring one that was never
  deleted, making sure `GET /api/issues/:id` on a deleted issue behaves consistently.

Whatever you pick: follow the conventions in CONTRIBUTING.md, add tests, and keep
the error shape consistent.
