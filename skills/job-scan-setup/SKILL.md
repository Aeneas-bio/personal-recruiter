---
name: job-scan-setup
description: "One-time onboarding that builds a personalized, autonomous recurring job-search scan for the current user — a persona/rubric file, a job tracker spreadsheet, and a scheduled task that runs the companion `job-scan` skill on a cadence. Use this whenever someone wants to automate their job search, set up job alerts smarter than a keyword filter, build a personal job-market scanning assistant, turn their job hunt into a recurring workflow, or asks something like \"can you scan job boards for me regularly,\" \"help me find roles that actually fit me,\" or \"set up something like a recruiter watching the market for me.\" Also trigger if the user mentions wanting to track job applications more systematically alongside searching for new ones. Run this only once per person (per job search) — if a persona file and tracker already exist for them, this skill's onboarding is likely already done and the `job-scan` skill should be used instead to actually run scans."
---

# Job Scan Setup

This skill turns one person's job search into a system: a written persona that captures who they are and what they're targeting, a spreadsheet that tracks every lead, and a scheduled task that keeps scanning on their behalf. It is deliberately an *interview*, not a form — the output is only as good as the specificity of what you draw out of the person. A generic persona produces generic, keyword-matched results; a sharp one produces a scan that reads job descriptions the way a good recruiter would.

Do this once per person. If you find an existing persona file and tracker already in their files, tell them so and point them to the `job-scan` skill instead — re-running full onboarding on top of an established system will overwrite calibration they've already earned through feedback.

## Before you start: set expectations

Tell the person up front, in plain terms, what this will produce: a persona file they should expect to review and correct, a tracker spreadsheet, and a recurring scheduled task. Let them know the first couple of scans will probably need tuning — this mirrors how a real search assistant would ramp up, not a one-shot config wizard. This sets the right expectation that Step 9 below (the calibration log) is not optional polish, it's how the system actually gets good.

## Step 1 — Confirm access to the tools this will actually use

The scan itself needs a browser automation connector (e.g., Claude in Chrome or equivalent) to read job postings and, on some boards, to browse while logged in. If the person doesn't have one connected yet, tell them plainly and point them to wherever their environment manages connectors — don't try to install anything yourself, and don't ask for or handle any of their credentials directly. Logging into a specific job board (if they want that for better results on gated postings) happens in their own browser session, never through you.

Also check (don't assume) whether the following are available and ask the person if they want to use them:
- A storage backend for the persona and job tracker — see `assets/storage_backends.md` for the options (Notion, Jira+Confluence, GitHub, local file, or Google Drive) and what each connector needs. This gets chosen properly in Step 6; here just confirm which connectors actually exist so Step 6 isn't guessing.
- A document-editing capability (needed if they want auto-tailored CV/cover-letter drafts).
- A chat tool connector (Slack, Teams, email) if they want scan summaries delivered somewhere instead of just saved to a file.
- A task-scheduling capability, to actually make the scan recurring.

## Step 2 — Build the persona (who this person is today)

Ask for, in this order:
1. **A folder or set of files of their CVs/resumes** — including old versions if they have them; a resume that changed over 3 jobs tells you more about trajectory than the latest one alone.
2. **A folder or set of cover letters**, if they have any saved. Cover letters are the best source for a person's actual voice and stated motivation — mine them for language, not just facts.
3. **Their LinkedIn profile URL, and recent posts if they write publicly.** If you have browser access, read the profile directly rather than asking them to describe it.
4. **Direct follow-up questions** to fill gaps the documents don't cover: certifications, side projects, specific tools/platforms they're deep in but might not have listed, what they'd say is their biggest unlisted strength, and — importantly — what they explicitly do NOT want to keep doing even if it's adjacent to their experience.

Synthesize all of this into a persona file using `assets/persona_template.md` as the structure (copy it, don't just describe it — the person should end up with a real file). Fill in Sections 1 and 2 now; Sections 3 and 4 need Step 3 and Step 4 below first.

Read the template's own inline guidance for what "good" looks like in each section — it's written to explain the reasoning, not just list fields.

## Step 3 — Build job history from past applications (the calibration seed)

Ask for URLs to jobs they've applied to before — explicitly say it's fine, even useful, to include ones they were rejected from or that never called back. Rejections are some of the highest-signal data available: they tell you what looked right on paper but wasn't. For each URL:
1. Open it (or search for the title + company if the original link is dead) and extract title, responsibilities, and stated requirements/prerequisites.
2. Ask the person: did you apply because you wanted it, or because it seemed like a reasonable stretch? What happened? In hindsight, was it actually a good fit?

Use these answers to draft the persona's Section 4 (Disqualifiers & Positive reinforcement) — this is where a generic rubric turns into a personal one. A pattern like "every clinical-operations-flavored role got rejected and in hindsight the person didn't want it anyway" becomes a standing disqualifier. A pattern like "the two roles this person got most excited about were both platform-building roles at early-stage companies" becomes a positive-reinforcement note.

Also use this step to backfill the tracker (Step 6) so the first live scan doesn't re-surface anything already in this list.

## Step 4 — Refine target levels and titles

Show the person the career-arc summary from Step 2 and ask directly: what seniority are you targeting (Staff? Principal? Director? Head of? VP?) — and which of those would you take a half-step down for if the role were exceptional? Then work with them to produce a concrete list of title variants worth searching (e.g., if "Director of X" is in scope, so is "Head of X," "Senior Director X," possibly "VP X" — don't assume these are equivalent to the person, ask). Write this into persona Section 2 (target role families) and Section 5 (search title seeds).

A common failure mode worth watching for explicitly: title-based search under-surfaces a level the person actually wants because the search seeds only used one phrasing. If Director-level (or whatever their target level is) is in scope, make sure the seed list includes that level's variants explicitly, not just the more senior titles that happen to come to mind first.

## Step 5 — Choose job boards

Propose the generic core list from `assets/default_job_boards.md`, then ask two direct questions: "any boards you already use or trust?" and "is there an industry-specific board for your field?" (there almost always is one, and it's often where the best senior postings with transparent salary bands live). Write the final list into persona Section 5.

## Step 6 — Choose storage and build the tracker

Read `assets/storage_backends.md`. Ask the person which backend they want for the persona and which for the job tracker (the same answer for both is the common case). Recommend Notion as the default suggestion if they have no preference and it's connected — full capability on both, one connector, native board view.

Before proceeding: confirm the chosen connector(s) actually exist (from Step 1's check). If a chosen backend has a ❌ in the capability table for something the setup will need — most importantly in-place updates — say so plainly and let the person decide whether to proceed anyway or pick something else. Don't silently fall back to Google Drive; it's the legacy default, not a safe fallback, since it can only create files, never update them.

Instantiate the job tracker using the chosen backend's create operation (see the tool mapping in `assets/storage_backends.md`), using `assets/tracker_template_spec.md` for the column/field structure. Backfill it with the job history gathered in Step 3 via the backend's `create()` operation, one job record at a time.

Write the `storage:` config block (persona backend + location, job backend + location) into persona Section 0, per the format in `assets/storage_backends.md`, so `job-scan` can self-configure on every run without re-asking.

## Step 7 — Configure optional features

Ask about each of these explicitly — don't silently default them on or off:
- **Stale-posting sweep** (retire dead postings automatically each run). Cheap, generally worth it; default to yes if the person has no preference.
- **Auto-tailored CV/cover-letter drafts** for high-fit roles. If yes, ask which base CV and cover letter files to draft from, and what fit-score threshold should trigger it (suggest 85+ as a starting point — high enough to only fire for genuinely strong matches).
- **Where scan summaries should go**: a chat channel (if a connector is available), an email draft, or just a saved report file the person reads when they check in. Get a specific channel ID / address if applicable — don't guess.
- **Fit-score threshold** for adding a lead to the tracker at all (suggest 65–70 as a starting point, explain it's easy to tune down later based on feedback, matching how this system's original instance evolved).
- **Salary floor rule**: a hard minimum, and whether to keep roles that don't post salary at all (usually yes, flagged rather than excluded).
- **Cadence**: how often to run (twice a week is a reasonable default suggestion, but ask — some people want daily, some want weekly).

Write all of this into the persona file (Sections 4, 7, 8, 9) as you go.

## Step 8 — Create the scheduled task

Use the scheduling capability to create a recurring task at the chosen cadence. The task's prompt should be short and should invoke the `job-scan` skill by name, passing it the specific paths and settings gathered above — persona file path, tracker file path, and a reminder of which optional features are enabled. Do not duplicate the full scan logic into the scheduled task's prompt; that logic lives in the `job-scan` skill so it can be improved once and apply to everyone using it.

Example shape for the scheduled task prompt (fill in the brackets, don't ship them literally):

> Run my recurring job scan using the `job-scan` skill. Persona file: `[path]`. Tracker file: `[path]`. Run autonomously; only pause if a strong match is genuinely ambiguous on fit.

## Step 9 — Wrap up

Summarize what was built (persona file, tracker, scheduled task and its cadence) and say plainly that the first couple of runs are a tuning period — encourage the person to correct anything the scan gets wrong (a missed role, an over-eager match, a disqualifier that should exist) and note that feedback belongs in the persona file's Calibration Log (Section 10) so it sticks. Offer to run the first scan immediately rather than waiting for the schedule, so they get a concrete result to react to right away.
