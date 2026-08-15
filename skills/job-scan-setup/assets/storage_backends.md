# Storage backends reference

This system stores two kinds of information, and they don't have to live in the same place:

- **Persona** — free text, sectioned, updated occasionally in whole-section rewrites (plus incremental additions to the Calibration Log).
- **Job pipeline** — structured records with a `stage`, updated frequently, one field at a time (mostly stage moves).

Use this file during `job-scan-setup` Step 6 to pick backends, and read it again in `job-scan` any time storage operations are needed — the run logic should call these operations, never backend-specific mechanics directly.

## The two interfaces

**PersonaStore**
- `read()` → text
- `write(text)` → full replace
- `append_section(section_name, text)` → add without resending the whole document (use for Calibration Log entries)

**JobStore**
- `list(filter?)` → records
- `get(job_id)` → record
- `find_by_url(url)` → record or none (dedup check before inserting)
- `create(record)` → job_id
- `update(job_id, partial)` → record (stage moves, notes, etc.)
- `retire(job_id, reason)` → for the stale-posting sweep

`JobRecord` fields (same regardless of backend): `title, company, url, source_board, date_found, stage (sourced|applied|interviewing|offer|rejected|withdrawn|stale), fit_score, salary_min, salary_max, salary_disclosed, notes, tailored_docs_url`

## Capability flags — check before assuming a backend can do something

| Backend | Persona write (in-place) | Job update (in-place) | Native board view | Works from a headless/cloud scheduled task |
|---|---|---|---|---|
| Google Drive (xlsx) | ❌ create-only | ❌ create-only | ❌ | ❌ (needs a local Drive-sync workaround) |
| Notion | ✅ | ✅ | ✅ | ✅ |
| Confluence (persona) + Jira (jobs) | ✅ | ✅ | ✅ | ✅ |
| GitHub (repo file + Issues) | ✅ | ✅ (via labels) | ⚠️ labels only, no board API | ✅ |
| Local file (markdown + JSON/CSV) | ✅ | ✅ | ❌ | only if the run environment has local file access |

If a chosen backend has ❌ on something the run needs (e.g. in-place update), tell the person plainly during setup rather than silently working around it — the Drive setup was the original version of this problem.

## Backend → tool mapping

**Notion**
- Persona → a page: read via `notion-fetch`; write/append via `notion-update-page`.
- Jobs → a database: create via `notion-create-pages`; update (incl. stage) via `notion-update-page`; list/find_by_url/filter via `notion-query-data-sources`. Board view is just how the database renders — no extra setup.

**Confluence + Jira**
- Persona → a Confluence page: read via `getConfluencePage`; write via `updateConfluencePage` (append by including prior content plus the new section).
- Jobs → a Jira board: create via `createJiraIssue`; update/stage-move via `editJiraIssue` / `transitionJiraIssue`; list/find_by_url/filter via `searchJiraIssuesUsingJql`.

**GitHub**
- Persona → a markdown file in a private repo: read/write via the repo file content API (bonus: free version history for the Calibration Log specifically).
- Jobs → Issues + `stage:*` labels: create an issue per job, swap labels for updates, search/list via issue search. No native board — if the person wants visual pipeline, they'd need to add a Projects v2 board manually and Claude can only manage it if a matching MCP tool becomes available.

**Google Drive (legacy default — avoid for new setups)**
- Persona → doc: `create_file` only, every "write" is actually a new file.
- Jobs → xlsx via the `xlsx` skill: same limitation — every update means regenerating and re-uploading the sheet.

**Local file**
- Persona → a markdown file on disk, plain read/write.
- Jobs → a JSON or CSV file, read-modify-write. No board, but works anywhere with file access and needs no connector — reasonable fallback if nothing else is connected.

## Where the choice gets recorded

Once chosen, write the backend selection into the persona file itself (see Section 0 of `persona_template.md`) so `job-scan` can self-configure on every run without re-asking:

```yaml
storage:
  persona_backend: notion
  persona_location: <page id, file path, or equivalent>
  job_backend: notion
  job_location: <database id, board key, file path, or equivalent>
```

Persona and job backends don't have to match — e.g. persona in Notion, jobs in Jira, if that's what the person prefers.

## Choosing a backend during onboarding

Ask the person which backend they want for persona, and which for jobs (same answer is fine and is the common case). Before proceeding:
1. Check the corresponding connector is actually available — don't assume from this file alone.
2. If not connected, tell the person and offer Local file as a no-connector fallback rather than silently defaulting to Drive.
3. Recommend Notion as the default suggestion if the person has no preference and it's connected: single connector, full capability on both interfaces, includes a board view natively.
