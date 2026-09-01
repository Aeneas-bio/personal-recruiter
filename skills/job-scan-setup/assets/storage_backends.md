# Storage backends reference

This system stores two kinds of information, and they don't have to live in the same place:

- **Persona** — free text, sectioned, updated occasionally in whole-section rewrites (plus incremental additions to the Calibration Log).
- **Job pipeline** — structured records with a `stage`, updated frequently, one field at a time (mostly stage moves) — true for every backend below except Google Drive and OneDrive, where the rotation scheme further down batches all of a run's changes into one new full-snapshot file instead of touching individual fields.

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

`JobRecord` fields (same regardless of backend): `title, company, url, source_board, date_found, stage (sourced|applied|interviewing|offer|rejected|withdrawn|closed|stale), fit_score, salary_min, salary_max, salary_disclosed, notes, tailored_docs_url`

`closed` and `stale` are easy to conflate but mean different things: `closed` is a verdict about the posting itself (confirmed filled or taken down, set by the closed-posting sweep) — `stale` is a time-based flag on a record that hasn't changed in a configured window (default suggestion: 30 days), independent of whether the posting is still live. A record can be `stale` without being `closed`, or go straight to `closed` well before it would otherwise have gone stale.

## Capability flags — check before assuming a backend can do something

| Backend | Persona write (in-place) | Job update (in-place) | Native board view | Works from a headless/cloud scheduled task |
|---|---|---|---|---|
| Google Drive (xlsx or ODS, file-rotation scheme) | ❌ create-only (persona still can't rotate — see note below) | ⚠️ not true in-place, but the rotation scheme below makes "append" safe and non-destructive despite that | ❌ | ❌ (needs a local Drive-sync workaround) |
| Microsoft OneDrive (Excel, same rotation scheme) | ❌ create-only (same caveat as Drive) | ⚠️ same as Drive — rotation scheme substitutes for real in-place update | ❌ | ❌ (needs a local OneDrive-sync workaround) |
| Notion | ✅ | ✅ | ✅ | ✅ |
| Confluence (persona) + Jira (jobs) | ✅ | ✅ | ✅ | ✅ |
| GitHub (repo file + Issues) | ✅ | ✅ (via labels) | ⚠️ labels only, no board API | ✅ |
| Local file (markdown + JSON/CSV) | ✅ | ✅ | ❌ | only if the run environment has local file access |

If a chosen backend has ❌ on something the run needs (e.g. in-place update) **and no rotation scheme is defined for it**, tell the person plainly during setup rather than silently working around it — the original, pre-rotation Drive setup was exactly this problem. Drive and OneDrive are no longer flat ❌ for job updates: the rotation scheme in the next section makes repeated "creates" behave like safe appends from the person's point of view, at the cost of managing multiple files instead of one. Persona storage on Drive/OneDrive is **not** covered by the rotation scheme below — it's document-shaped, not row-shaped, so a persona still needs a backend from the ✅ rows above (Notion, Confluence, GitHub, or Local file) even if the person's job tracker lives on Drive or OneDrive. Persona and job backends not matching is normal and expected (see "Where the choice gets recorded" below).

## Google Drive / OneDrive job-tracker rotation scheme

This section exists because Drive and OneDrive (via the tools available in this environment) can create files but cannot edit a file's *content* in place — only metadata like title or parent folder. Rather than pretend this limitation away or silently fall back to something less safe, the tracker is kept as a **sequence of full-snapshot files**, each one a complete copy of everything before it plus this run's changes, with old snapshots pruned once enough newer ones exist. This is deliberate designed-around-the-constraint behavior, not a workaround to hide from the person — mention it during onboarding (Step 6) so they understand why they'll see multiple `JobTracker*` files in their Drive/OneDrive folder rather than one.

**Format choice**: ask the person, during onboarding, whether they want the tracker as `.xlsx` or `.ods` (OpenOffice format) — some people don't have Microsoft Office and prefer ODS for cost or tooling reasons. Whichever is chosen applies to the original file and every rotated file consistently; don't mix formats within one person's rotation sequence. **On OneDrive specifically, skip this question — always use `.xlsx`** (Excel is native there and there's no reason to offer ODS on a Microsoft-native backend).

**Naming and sequence**:
- The very first file, created at onboarding, is **`JobTracker.xlsx`** (or `.ods`) — no date in the name. This file is **never deleted, under any circumstance** — treat it as a permanent point-in-time anchor from before any rotation began.
- Every subsequent `job-scan` run — including a run that finds nothing new to append — creates a new file named **`JobTracker_{{DATE}}.xlsx`** (`{{DATE}}` = that run's calendar date, e.g. `2026-09-05`).
- If a second (or third, etc.) run happens on the same calendar date, append a zero-padded counter: the first run of a day produces `JobTracker_2026-09-05.xlsx` with no suffix; a second run that same day produces `JobTracker_2026-09-05_01.xlsx`; a third produces `_02`; and so on.

**Content — copy-forward, not a diff**: resolve the current file (see "Resolving the current file" below), read its **complete** contents in full, then write a **new** file containing everything from the current file plus this run's new/updated records. Every dated file is a full, standalone, human-openable snapshot of the entire job history to date — never a partial diff that only makes sense alongside other files.

**Resolving the current file — by filename, never by modified-time metadata**: file-system or Drive/OneDrive metadata (e.g. "last modified") is not trustworthy here — something else could touch a file's metadata without it being the actual latest snapshot in the sequence. Instead:
1. List every file matching `JobTracker*.xlsx` (or `.ods`) in the tracker's folder.
2. Parse each name: the bare `JobTracker` file sorts earliest; `JobTracker_{{DATE}}` sorts by that date with implicit suffix 0; `JobTracker_{{DATE}}_{{NN}}` sorts by that date then by `NN`.
3. Sort by (date descending, then suffix descending). The top result is the current file — read from it, and copy-forward from it when writing this run's new file.
4. If only the bare `JobTracker` file exists (i.e., this is the first `job-scan` run after onboarding), that's the current file.

**Retention — keep the original plus the 2 most recent dated files**: after writing a new dated file, count how many dated files (anything matching `JobTracker_{{DATE}}...`, excluding the bare original) now exist. If a **3rd** dated file now exists, delete the **oldest** dated file by the same (date, suffix) ordering used to resolve the current file — never delete the bare undated original, ever, regardless of how many dated files exist. Same-day files count individually toward this limit; a day with several reruns burns through the retention window faster than a day with one run, and that's expected behavior, not a bug.

**Same-day duplicate cleanup is advisory only, never automatic**: if this run detects that one or more same-day sibling files already exist from an earlier run today (before this run's own new file is written), mention it plainly in the run's summary — e.g. "This is your 2nd tracker run today; `JobTracker_2026-09-05.xlsx` is now superseded by `_01` and safe to delete if you want to tidy up." Never delete a same-day sibling automatically; this is a suggestion for the person to act on if they choose, separate from and in addition to the automatic 2-dated-file retention rule above.

**What this means for `JobStore` operations against this backend**: `list()`, `find_by_url()`, and `get()` all read from the resolved current file only — never from older dated files, which exist purely as recent-history snapshots, not as additional data to merge. `create()` and `update()` don't modify the current file in place; they're both satisfied by the same copy-forward-then-write-new-file operation described above, run once per `job-scan` pass (not once per individual record — batch all of a run's creates/updates into the single new dated file that run produces).

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

**Google Drive (job tracker only — persona needs a different backend, see above)**
- Jobs → the rotation scheme described in "Google Drive / OneDrive job-tracker rotation scheme" above. Resolve the current file via `search_files` (query on title pattern `JobTracker`, filtered to the tracker's parent folder), download its contents via `download_file_content`, then write the new dated file via `create_file` with the combined contents. Prune the oldest dated file via whatever delete/trash operation the Drive connector exposes, once the 3rd dated file exists. Do not attempt to use `update_file` for content changes — its parameters only cover title/parent, never file content, in the tools available in this environment.

**Microsoft OneDrive (job tracker only — persona needs a different backend, see above)**
- Jobs → identical rotation scheme to Google Drive above, using whichever OneDrive/Excel-capable tool is connected for search/download/create/delete-equivalent operations. Always `.xlsx` here (see format-choice note above). Capability details for a specific OneDrive connector should be verified against that connector's actual tool description before relying on them, the same way Drive's `update_file` limitation was confirmed directly from its tool schema rather than assumed.

**Google Drive persona storage (avoid — no rotation equivalent exists for documents)**
- Persona → doc: `create_file` only, every "write" is actually a new file, and unlike the job tracker above, there's no defined rotation/retention scheme for persona documents. Steer persona to a ✅ backend instead (Notion, Confluence, GitHub, or Local file) rather than using Drive for persona at all, even if Drive or OneDrive is the chosen job backend.

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

**For Google Drive or OneDrive specifically**, `job_location` must be a **folder** (ID or path), not a fixed filename — the actual current filename changes every run under the rotation scheme above, so `job-scan` needs to resolve it fresh each time via the filename-parsing rule, not read a filename that was fixed once at onboarding and never revisited. Also record the chosen file format (`xlsx` or `ods`) alongside `job_location` so every rotation writes the same format consistently:

```yaml
storage:
  persona_backend: notion
  persona_location: <page id>
  job_backend: google_drive
  job_location: <folder id>
  job_file_format: xlsx
```

## Choosing a backend during onboarding

Ask the person which backend they want for persona, and which for jobs (same answer is fine and is the common case). Before proceeding:
1. Check the corresponding connector is actually available — don't assume from this file alone.
2. If not connected, tell the person and offer Local file as a no-connector fallback rather than silently defaulting to Drive.
3. Recommend Notion as the default suggestion if the person has no preference and it's connected: single connector, full capability on both interfaces, includes a board view natively.
4. **If the job backend is Google Drive**, also ask which file format they want — `.xlsx` or `.ods` — and record it as `job_file_format` per the config block above. Explain briefly why they'll see multiple `JobTracker*` files over time (the rotation scheme above), so it isn't a surprise later. **If the job backend is OneDrive, skip the format question** — it's always `.xlsx` there.
