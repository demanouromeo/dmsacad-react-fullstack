# CLAUDE.md

Guidance for Claude Code when working anywhere in this root folder. For the functional/architecture
overview, read [`README.md`](README.md) first — this file only covers things Claude specifically needs to
know that aren't documented there, or has gotten wrong before.

## Repo structure: three separate Git histories, not one

This root folder, `backend/`, and `frontend/` are three independent Git repositories (own remotes, own
history) nested under one `htdocs` folder, not a monorepo sharing one `.git`. Git commands must run from
inside the correct one (`cd backend` or `cd frontend`) or they silently operate on the wrong repo and
produce misleading (often empty) diffs/status — this already caused a false "an automatic process is
discarding my edits" scare in a past session, purely from a stray `cd ..` shifting the shell back to root.
Confirm `git remote -v`/`pwd` match the repo you mean to touch before trusting `git status`/`git diff`.

Remotes: root → `dmsacad-react-fullstack`, `backend/` → `dmsacad-backend`, `frontend/` → `dmsacad_frontend`.

`c:\xampp\htdocs\dmsacad_backend_dev` is a stale, empty leftover folder from an older deployment layout —
not this project's backend (both `frontend/CLAUDE.md` and `backend/CLAUDE.md` already call this out; if
you see it referenced anywhere, assume it's stale).

## Local environment gotcha: `.env` DB name vs. the actual local MySQL database

`backend/CLAUDE.md` documents the generic `mysql` connection's database as `sm_db2`, matching what
`backend/.env`'s `DB_DATABASE` used to say — but the actual local MySQL database the remote dump gets
imported into on this machine is named **`smdb_2`** (transposed characters). If `DB_DATABASE` ever drifts
back to `sm_db2`, every request against the local `mysql` connection fails with `SQLSTATE[HY000] [1049]
Unknown database 'sm_db2'`. Because several frontend fetch methods swallow errors and silently fall back to
`[]`/empty results (see below), this doesn't surface as an obvious error — it looked like flaky logins and
a report-card module whose computed averages changed on every reload, not a config typo. Run `SHOW
DATABASES` on the local MySQL instance before assuming `.env`'s `DB_DATABASE` is correct.

## Cross-cutting gotcha: unbounded parallel fetches vs. the remote host's MySQL connection limit

The remote backend (`https://dmsacad.com/dmsacad_backend_secured/`, the frontend's default `VITE_BASE_REMOTE_URL`)
runs on shared hosting with a MySQL account that has some connection-limit ceiling. A feature that fans out
many concurrent requests per page load (report cards fetch marks per subject × dbsequence/competence — up
to ~90 concurrent `fetch()` calls for one annual bulletin) can exceed that ceiling under load. When it
does, the backend does **not** return a clean error status — it returns **HTTP 200** with a raw PHP error
string as the body (e.g. `SQLSTATE[HY000] [2002] Operation not permitted`, from a failed
`MyHelper::getSchoolYearID()` connection attempt). Confirmed 2026-09-06 by replaying the exact request
burst directly against the live remote API with `curl` — intermittent and load-dependent, not a fixed hard
cap at any particular request count (a burst that fails once can succeed cleanly minutes later).

Every frontend `*Reader` class treats "response body isn't valid JSON" the same as "request failed" and
falls back to an empty result (`[]`) rather than surfacing an error — by convention, so one bad fetch
doesn't crash a whole screen. Combined with the above, a connection-limit hiccup under concurrent load
silently looks exactly like "this subject/sequence has no marks", not a fetch failure. This exact chain
caused the report-card module to show empty/wrong marks and a different computed average on every reload
on the remote deployment, even though the same bug never reproduced locally (a single local MySQL instance
never refuses this many connections from one client).

**Fix pattern established**: `frontend/src/utils/reportCard/loadAnnualReportCardData.ts` now flattens its
nested per-subject `Promise.all` fan-outs into flat lists of (subject, sequence/competence) pairs and runs
them through the app's existing bounded-concurrency helper — `frontend/src/utils/concurrency.ts`'s
`mapWithConcurrency`/`DEFAULT_REPORT_CONCURRENCY` (already the established convention for whole-section
per-classe loops in `EffectifsManager`'s siblings, `MarkEntryManager`, `ScholarshipManager`, `TimetableHub`,
etc. — reuse this one rather than writing a second concurrency-limiting utility, which is exactly the
mistake made and then corrected while fixing this bug) — instead of firing all of them at once, plus a
retry-with-backoff inside `MarkReader.fetchSeqMarks`/`fetchCompMarks` so a transient connection refusal
gets retried rather than immediately treated as "no marks". **Any future feature that fetches data per
(subject × classe) or otherwise fans out across many entities at once — SMS, livrets, promotions,
scholarships, and every other module still unbuilt per `frontend/CLAUDE.md`'s status table — should reuse
`mapWithConcurrency` from the start**, rather than a bare `Promise.all` over every item, to avoid
re-hitting this same remote connection ceiling.
