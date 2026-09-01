---
name: job-scan
description: "Runs one pass of a person's recurring, personalized job-search scan: reads their persona/rubric file, sweeps their tracker for dead postings, scans their chosen job boards for roles matching their target titles, scores fit against their personal rubric, dedupes against existing tracker rows, appends qualifying new leads, optionally auto-drafts tailored CV/cover-letter documents for standout matches, and delivers a summary. Trigger this when a scheduled job-scan task fires, or when the user directly asks to run their job scan now, check for new job leads, refresh their job search, or see what's new on the market for them. Requires a persona file and tracker spreadsheet already built by the `job-scan-setup` skill — if those don't exist yet for this person, run `job-scan-setup` first instead of guessing at their preferences."
---

# Job Scan

This is the recurring engine behind a personalized job search. It only works well if it's actually reading a real persona file — not applying a generic template — so the first thing to do on every run is load that file and treat it as the rubric of record, even where it might seem to disagree with generic best practice. The person who built that file (or corrected it after a previous run) knows their own search better than any default would.

## Inputs

You need, at minimum:
- **A persona location** (built by `job-scan-setup`, following `job-scan-setup/assets/persona_template.md`'s structure) — read via `PersonaStore.read()`, in full, every run, not a cached summary. People edit these between runs specifically to fix mistakes; skipping the fresh read defeats the purpose.
- **A job tracker location** — the running record of every lead, applied or not, accessed via `JobStore`.

Both the persona and job backends, and their locations, come from the `storage:` block in persona Section 0 (see `job-scan-setup/assets/storage_backends.md`). Read that block first — before anything else — to know which `PersonaStore`/`JobStore` implementation to use for the rest of this run. Persona and job backends can differ (e.g. persona in Notion, jobs in Jira); never assume they match. All storage operations below go through the `PersonaStore`/`JobStore` interface, never backend-specific mechanics directly — if a step here describes something a chosen backend can't do in-place (see the capability table in `storage_backends.md`), say so rather than working around it silently.

**If `job_backend` is `google_drive` or `onedrive`**, `job_location` is a folder, not a fixed filename — before any `JobStore` operation this run, resolve the actual current file by listing everything matching `JobTracker*` in that folder and applying the filename-based ordering rule in `storage_backends.md`'s rotation scheme (closest date, then highest numeric suffix — never trust modified-time metadata for this). Every `JobStore` read in this run targets that resolved file; every `JobStore` write this run produces happens at Step 6, as one new rotated file, not as in-place edits to the resolved file.

If the persona location, job tracker location, or the `storage:` block itself is missing or clearly stale/placeholder (template text never filled in), stop and say so rather than guessing — this almost always means onboarding wasn't finished, and a scan run on an empty rubric produces noise, not leads.

## Step 1 — Load the persona and treat it as the rubric of record

Read the full persona via `PersonaStore.read()`. Pay particular attention to:
- Section 4's Disqualifiers and Positive reinforcement — these exist because a generic scorer got something wrong in the past. They override generic judgment.
- Section 10's Calibration Log — the most recent entries are the freshest signal about what to change. If a log entry contradicts something earlier in the document, the log wins (it's the correction).

## Step 2 — Closed-posting sweep and staleness flag (if enabled in the persona file's Section 9)

These are two separate checks on the same set of records — don't conflate them. `closed` is a verdict about the posting (it's gone); `stale` is a nudge that a record hasn't been touched in a while, regardless of whether the posting is still live.

Use `JobStore.list()` to get every record still open — stage `sourced` or `applied` — and follow each one's `url`. If dead/expired/no-longer-accepting, call `JobStore.retire(job_id, reason)`, which sets stage `closed` and preserves any existing notes rather than replacing them. If live, note the posted date if shown, via `JobStore.update(job_id, {...})`, without touching stage. Don't overwrite records that already carry a richer, current stage (e.g., `interviewing`, `rejected`) — for those, only note liveness if it adds information.

Separately, for every open record whose notes/stage haven't changed since the staleness window configured in the persona's Section 9 (default suggestion: 30 days), set stage `stale` via `JobStore.update()`. This runs independent of the closed-posting check above — a record can be flagged `stale` on one run and turn out to still be live, or go straight to `closed` before it ever goes stale.

**On Google Drive or OneDrive**, every `retire()`/`update()` call in this step is staged in memory against the resolved current file's contents, not written anywhere yet — the actual write happens once, at Step 6, as this run's single new rotated file containing all of this step's changes plus anything Step 6 itself adds. Don't write a file after this step and another after Step 6; batch everything into the one dated file this run produces.

Never attempt to use or ask for account credentials to get past a login wall. If a link is genuinely gated with no existing session, leave it and report it under "needs sign-in" in the final summary.

## Step 3 — Scan the configured boards

Use the board list and search-title seeds from the persona's Section 5. Prefer recent postings (roughly the last week, or since the last run, whichever is longer) but don't hard-exclude a strong older posting that's still live — note its age in the summary so the person can judge urgency themselves.

Title-seed search alone under-surfaces roles that fit Section 2's Problems of interest but use a title that isn't in the seed list (a smaller company solving the same problem under a differently-titled role, for instance). Where a board supports it, also run a description/full-text search using language pulled from the Problems of interest themselves, not just the title seeds — treat this as a supplement to the title search, not a replacement.

For each promising result, get the canonical URL and enough of the actual responsibilities section to score it properly — a title and a two-line teaser is not enough to apply Section 4's rubric. Open the posting, or use a targeted search for the exact title + company if the board's own listing doesn't expose the full text.

## Step 4 — Score every candidate against Section 4 of the persona

This is the step most likely to go wrong via shortcut, so slow down here specifically:
- Score based on how the required skills are actually applied in the responsibilities, not on how many keywords from Section 1 appear. A role can look like a bullseye on title and buzzwords and still be the wrong day-to-day work — that's exactly what Section 4's disqualifiers exist to catch. Read them like a checklist before finalizing any score.
- Score Problem fit deliberately, not as an afterthought folded into domain — read the responsibilities and ask which of Section 2's Problems of interest (if any) this role would actually have the person spending their time on. A role that names the right domain but whose day-to-day is a different problem than any on the list should score low here even if everything else looks strong.
- Apply the salary rule and location scoring exactly as the persona defines them.
- Where a role sits right at the edge of a disqualifier category (e.g., senior enough to be in scope but at a company type the person has historically not converted with), still surface it — flag the caveat in Notes rather than silently dropping it. The person deciding not to pursue something is a better outcome than never seeing it.

## Step 5 — Dedupe

Never add a record that's already in the tracker. Check via `JobStore.find_by_url(url)` first; if that's ambiguous (some boards recycle or repost with new IDs), fall back to `JobStore.list()` and match on Company + closely-matching Title. If a role is a near-duplicate of an existing record (reposted, slightly retitled), note that on the existing record rather than creating a new one.

## Step 6 — Append qualifying records

Create one record per qualifying role via `JobStore.create(record)`, populated with the standard `JobRecord` fields (`title, company, url, source_board, date_found, stage, fit_score, salary_min, salary_max, salary_disclosed, notes, tailored_docs_url`) as defined in `storage_backends.md` — the field set is the same regardless of backend, so no backend-specific column mapping is needed here. New records start at stage `sourced`. If the backend's create operation fails or returns something unexpected, don't retry blindly — surface it in the summary rather than silently dropping the lead.

**On Google Drive or OneDrive, this step is where every staged change actually gets written**, as a single new rotated file, following `storage_backends.md`'s rotation scheme exactly:
1. Take the resolved current file's full contents (already read at the start of this run) plus every staged change from Step 2 (retires, staleness flags, liveness notes) plus this step's new records.
2. Determine today's filename: `JobTracker_{{DATE}}` where `{{DATE}}` is this run's date; if one or more files already exist for today (check the folder listing again — don't assume from the start-of-run resolution, since this run itself is what's adding today's entry), append the next zero-padded counter (`_01`, `_02`, ...).
3. Write the combined contents as that new file, in the format recorded in `job_file_format` (`xlsx` or `ods`; always `xlsx` for OneDrive), with the same formatting as every prior file in the sequence (bold header, frozen row 1 — see `tracker_template_spec.md`).
4. Count dated files now present (excluding the bare undated original). If a 3rd now exists, delete the oldest by the same date/suffix ordering — never the undated original.
5. If today's new file has one or more same-day siblings from an earlier run today, note that plainly in Step 8's summary as an advisory (not automatic) cleanup suggestion — see `storage_backends.md` for exact phrasing guidance.

This means a Drive/OneDrive run produces exactly one file-write operation for the whole run, not one per record and not one per step — batch everything into it.

## Step 7 — Auto-tailor documents, if enabled (persona Section 8)

For any new record at or above the configured fit threshold: draft a tailored CV and cover letter from the person's real base documents, using the document-editing capability available in this environment. Keep every claim truthful to the persona file — never invent experience, credentials, or accomplishments the person doesn't actually have, even if it would make the fit look stronger. Save drafts to a per-job folder with `_draft` in every filename, and record the link via `JobStore.update(job_id, {tailored_docs_url: ...})`. Never point that field at the person's pre-existing real documents. **On Google Drive or OneDrive**, this update is staged the same way Step 2's are — it lands in Step 6's single batched rotation write, not a separate file write here.

## Step 8 — Deliver the summary

Send it wherever persona Section 7 configured (chat channel, email draft, or a saved report) — not somewhere else, even if another channel seems more convenient in the moment. Include:
- Counts: postings scanned, new leads added, postings closed, records newly flagged stale.
- The new-leads list, each with a link, company, location (with score), salary, fit score, and any caveats.
- A "seen but didn't make the cut" section for notable near-misses — this is often as useful to the person as the hits, especially early on while they're still tuning the rubric.
- Anything needing sign-in, and any auto-drafted documents generated this run.
- **On Google Drive or OneDrive**, if this run's new file has a same-day sibling from an earlier run today, a plain-language note that the earlier file is now superseded and safe to delete if the person wants to tidy up (never delete it automatically — see `storage_backends.md`).

If nothing new qualified, say so plainly rather than padding the summary — a quiet week in the market is real information too.

## A note on getting better over time

If something about this run felt like it required judgment the persona file didn't clearly cover, that's worth surfacing to the person rather than silently resolving it one way. Their answer is calibration data. For a quick, single correction offered right here in the summary (Step 8), it's fine to log a dated entry to Section 10 yourself via `PersonaStore.append_section()` if they confirm the change on the spot. For anything bigger — several corrections at once, a shift in target seniority or Problems of interest, or the person wanting to sit down and go through recent results systematically — point them to the `persona-review` skill instead of trying to conduct that fuller conversation inline here; it's built for exactly that and keeps this skill focused on running scans, not re-deriving the rubric mid-run.
