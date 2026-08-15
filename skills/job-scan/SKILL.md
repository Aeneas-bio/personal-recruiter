---
name: job-scan
description: "Runs one pass of a person's recurring, personalized job-search scan: reads their persona/rubric file, sweeps their tracker for dead postings, scans their chosen job boards for roles matching their target titles, scores fit against their personal rubric, dedupes against existing tracker rows, appends qualifying new leads, optionally auto-drafts tailored CV/cover-letter documents for standout matches, and delivers a summary. Trigger this when a scheduled job-scan task fires, or when the user directly asks to run their job scan now, check for new job leads, refresh their job search, or see what's new on the market for them. Requires a persona file and tracker spreadsheet already built by the `job-scan-setup` skill — if those don't exist yet for this person, run `job-scan-setup` first instead of guessing at their preferences."
---

# Job Scan

This is the recurring engine behind a personalized job search. It only works well if it's actually reading a real persona file — not applying a generic template — so the first thing to do on every run is load that file and treat it as the rubric of record, even where it might seem to disagree with generic best practice. The person who built that file (or corrected it after a previous run) knows their own search better than any default would.

## Inputs

You need, at minimum:
- **A persona file** (built by `job-scan-setup`, following `job-scan-setup/assets/persona_template.md`'s structure) — read the ENTIRE file every run, not a cached summary. People edit these between runs specifically to fix mistakes; skipping the fresh read defeats the purpose.
- **A tracker spreadsheet path** — the running record of every lead, applied or not.

If either is missing or clearly stale/placeholder (template text never filled in), stop and say so rather than guessing — this almost always means onboarding wasn't finished, and a scan run on an empty rubric produces noise, not leads.

## Step 1 — Load the persona and treat it as the rubric of record

Read the full persona file. Pay particular attention to:
- Section 4's Disqualifiers and Positive reinforcement — these exist because a generic scorer got something wrong in the past. They override generic judgment.
- Section 10's Calibration Log — the most recent entries are the freshest signal about what to change. If a log entry contradicts something earlier in the document, the log wins (it's the correction).

## Step 2 — Stale-posting sweep (if enabled in the persona file's Section 9)

For every tracker row with a real URL in Job Link AND an empty/unclear Decision Date: follow the link. If dead/expired/no-longer-accepting, set Decision Date = today, Outcome = "Cold Posting," and append a short note (preserve any prior status text rather than replacing it). If live, capture the Posted Date if shown. Don't overwrite rows that already carry a richer, current status (e.g., "Application Submitted," "Reviewed - not a fit") — for those, only note liveness if it adds information.

Never attempt to use or ask for account credentials to get past a login wall. If a link is genuinely gated with no existing session, leave it and report it under "needs sign-in" in the final summary.

## Step 3 — Scan the configured boards

Use the board list and search-title seeds from the persona's Section 5. Prefer recent postings (roughly the last week, or since the last run, whichever is longer) but don't hard-exclude a strong older posting that's still live — note its age in the summary so the person can judge urgency themselves.

For each promising result, get the canonical URL and enough of the actual responsibilities section to score it properly — a title and a two-line teaser is not enough to apply Section 4's rubric. Open the posting, or use a targeted search for the exact title + company if the board's own listing doesn't expose the full text.

## Step 4 — Score every candidate against Section 4 of the persona

This is the step most likely to go wrong via shortcut, so slow down here specifically:
- Score based on how the required skills are actually applied in the responsibilities, not on how many keywords from Section 1 appear. A role can look like a bullseye on title and buzzwords and still be the wrong day-to-day work — that's exactly what Section 4's disqualifiers exist to catch. Read them like a checklist before finalizing any score.
- Apply the salary rule and location scoring exactly as the persona defines them.
- Where a role sits right at the edge of a disqualifier category (e.g., senior enough to be in scope but at a company type the person has historically not converted with), still surface it — flag the caveat in Notes rather than silently dropping it. The person deciding not to pursue something is a better outcome than never seeing it.

## Step 5 — Dedupe

Never add a row that's already in the tracker. Match on Job Link first; if that's ambiguous (some boards recycle or repost with new IDs), match on Company + closely-matching Title. If a role is a near-duplicate of an existing row (reposted, slightly retitled), note that rather than adding a new row.

## Step 6 — Append qualifying rows

Follow the tracker's column structure from persona Section 6 exactly — the columns and their order matter because this file gets read programmatically on every run, including by future runs of this same skill. Before writing, make a timestamped backup of the tracker file. After writing, verify the file still opens correctly and no sheets were lost — spreadsheet tools can silently corrupt files on save if interrupted mid-write; if a save produces a file that won't reopen cleanly, don't overwrite the last known-good version, and tell the person what happened.

## Step 7 — Auto-tailor documents, if enabled (persona Section 8)

For any new row at or above the configured fit threshold: draft a tailored CV and cover letter from the person's real base documents, using the document-editing capability available in this environment. Keep every claim truthful to the persona file — never invent experience, credentials, or accomplishments the person doesn't actually have, even if it would make the fit look stronger. Save drafts to a per-job folder with `_draft` in every filename, and link them in the tracker's CV Link / Cover Letter Link columns. Never point those columns at the person's pre-existing real documents.

## Step 8 — Deliver the summary

Send it wherever persona Section 7 configured (chat channel, email draft, or a saved report) — not somewhere else, even if another channel seems more convenient in the moment. Include:
- Counts: postings scanned, new leads added, cold postings retired.
- The new-leads list, each with a link, company, location (with score), salary, fit score, and any caveats.
- A "seen but didn't make the cut" section for notable near-misses — this is often as useful to the person as the hits, especially early on while they're still tuning the rubric.
- Anything needing sign-in, and any auto-drafted documents generated this run.

If nothing new qualified, say so plainly rather than padding the summary — a quiet week in the market is real information too.

## A note on getting better over time

If something about this run felt like it required judgment the persona file didn't clearly cover, that's worth surfacing to the person rather than silently resolving it one way. Their answer is calibration data — encourage them to add it to the persona file's Section 10 (or add it yourself if they say so directly) so the next run doesn't have to re-derive the same judgment call from scratch.
