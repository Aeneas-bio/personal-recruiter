---
name: job-scan-setup
description: "One-time onboarding that builds a personalized, autonomous recurring job-search scan for the current user — a persona/rubric file, a job tracker spreadsheet, and a scheduled task that runs the companion `job-scan` skill on a cadence. Use this whenever someone wants to automate their job search, set up job alerts smarter than a keyword filter, build a personal job-market scanning assistant, turn their job hunt into a recurring workflow, or asks something like \"can you scan job boards for me regularly,\" \"help me find roles that actually fit me,\" or \"set up something like a recruiter watching the market for me.\" Also trigger if the user mentions wanting to track job applications more systematically alongside searching for new ones. Run this only once per person (per job search) — if a persona file and tracker already exist for them, this skill's onboarding is likely already done and the `job-scan` skill should be used instead to actually run scans."
---

# Job Scan Setup

This skill turns one person's job search into a system: a written persona that captures who they are and what they're targeting, a spreadsheet that tracks every lead, and a scheduled task that keeps scanning on their behalf. It is deliberately an *interview*, not a form — the output is only as good as the specificity of what you draw out of the person. A generic persona produces generic, keyword-matched results; a sharp one produces a scan that reads job descriptions the way a good recruiter would.

Do this once per person. If you find an existing persona file and tracker already in their files, tell them so and point them to the `job-scan` skill instead — re-running full onboarding on top of an established system will overwrite calibration they've already earned through feedback.

## Before you start: set expectations

Tell the person up front, in plain terms, what this will produce: a persona file they should expect to review and correct, a tracker spreadsheet, and a recurring scheduled task. Let them know two things: (1) the first couple of scans will probably need tuning — this mirrors how a real search assistant would ramp up, not a one-shot config wizard; and (2) there is a short **pre-flight** (Step 1) that gets their browser and accounts into the right state before any scanning — doing this first prevents the most common onboarding problems (wrong account read, credentials confusion, gated postings invisible).

## Step 1 — Pre-flight: browser, authentication & account hygiene (do this FIRST, and gate on it)

This system reads job boards through the person's **own logged-in browser**, never through credentials you handle. Getting the browser and account state right before anything else is what separates a clean onboarding from a confusing one. Treat the first three checks as **hard gates** — do not move on to building the persona, choosing boards, or scanning until they pass.

1. **Claude browser extension installed — REQUIRED (hard gate).** Confirm the person has the Claude browser extension installed and enabled. This is what handles authentication to job boards and keeps their **passwords and tokens out of Claude entirely** — Claude never sees, types, or stores credentials; the person stays logged in in their own browser and Claude reads through that live session. If the extension isn't installed, **stop here** and point them to install it before continuing. Never work around a missing extension by asking for a password, token, or API key.

2. **Confirm the exact email to use — REQUIRED (hard gate).** Ask which email account this job search should run under, and read it back to confirm. Everything keys off this one account: board logins, profiles, saved searches, and any delivered summaries. Note it down to persist later (persona Section 0).

3. **One logged-in user, and it's that account — REQUIRED (confirm-gate).** Ask the person to ensure that **only** the target account is logged in in the browser Claude will drive, and that other browsers or browser profiles signed into different accounts are **closed**. A second identity logged in elsewhere (a personal Google/LinkedIn alongside the work one) is a common cause of the scan reading the wrong profile or the wrong recommendations. Get an explicit "yes — only that account is logged in" before proceeding; don't just remind and move on. If you have browser tools, you may optionally spot-check the active session's account, but the person's explicit confirmation is the gate.

Then confirm the **supporting capabilities** (flag anything missing, but these are not hard gates):
- Spreadsheet editing (for the tracker) — should exist in the base environment.
- Document editing (only if they want auto-tailored CV/cover-letter drafts).
- A chat/email connector (Slack, Teams, email) if they want summaries delivered somewhere instead of a saved file.
- Task scheduling, to make the scan recurring.

**Standing authentication rule (applies here and everywhere in this system):** never collect, store, type, or ask for the person's passwords, tokens, or API keys. Authenticated access always happens in their own browser via the extension. If a board can't be read because no session is active, the scan reports "needs sign-in" rather than touching credentials.

## Step 2 — Build the persona (who this person is today)

Create the persona file now from `assets/persona_template.md` (copy it to a real file in wherever they keep career files — the person should end up with an actual file, not a description). As you create it, **record the pre-flight results from Step 1 into the persona's "Section 0 — Environment & Access"**: the confirmed job-search email, that the browser + Claude extension are installed/enabled, the date they confirmed only that account is logged in, and the authentication rule. (If your copy of the template doesn't already contain a Section 0, add it — this info must live in the persona so scans and future sessions have it durably.)

Then build Sections 1 and 2 by asking for, in this order:
1. **A folder or set of files of their CVs/resumes** — including old versions if they have them; a resume that changed over 3 jobs tells you more about trajectory than the latest one alone.
2. **A folder or set of cover letters**, if they have any saved. Cover letters are the best source for a person's actual voice and stated motivation — mine them for language, not just facts.
3. **Their LinkedIn profile URL, and recent posts if they write publicly.** If you have browser access, read the profile directly rather than asking them to describe it.
4. **Direct follow-up questions** to fill gaps the documents don't cover: certifications, side projects, specific tools/platforms they're deep in but might not have listed, their biggest unlisted strength, and — importantly — what they explicitly do NOT want to keep doing even if it's adjacent to their experience.

Fill Sections 1 and 2 now; Sections 3 and 4 need Steps 3 and 4 below first. Read the template's own inline guidance for what "good" looks like in each section.

## Step 3 — Build job history from past applications (the calibration seed)

Ask for URLs to jobs they've applied to before — explicitly say it's fine, even useful, to include ones they were rejected from or that never called back. Rejections are some of the highest-signal data available: they tell you what looked right on paper but wasn't. For each URL:
1. Open it (or search for the title + company if the original link is dead) and extract title, responsibilities, and stated requirements/prerequisites.
2. Ask the person: did you apply because you wanted it, or because it seemed like a reasonable stretch? What happened? In hindsight, was it actually a good fit?

Use these answers to draft the persona's Section 4 (Disqualifiers & Positive reinforcement) — this is where a generic rubric turns into a personal one. A pattern like "every clinical-operations-flavored role got rejected and in hindsight the person didn't want it anyway" becomes a standing disqualifier. A pattern like "the two roles this person got most excited about were both platform-building roles at early-stage companies" becomes a positive-reinforcement note.

Also use this step to backfill the tracker (Step 6) so the first live scan doesn't re-surface anything already in this list.

## Step 4 — Refine target levels and titles

Show the person the career-arc summary from Step 2 and ask directly: what seniority are you targeting (Staff? Principal? Director? Head of? VP?) — and which of those would you take a half-step down for if the role were exceptional? Then work with them to produce a concrete list of title variants worth searching (e.g., if "Director of X" is in scope, so is "Head of X," "Senior Director X," possibly "VP X" — don't assume these are equivalent to the person, ask). Write this into persona Section 2 (target role families) and Section 5 (search title seeds).

A common failure mode worth watching for explicitly: title-based search under-surfaces a level the person actually wants because the search seeds only used one phrasing. If Director-level (or whatever their target level is) is in scope, make sure the seed list includes that level's variants explicitly, not just the more senior titles that happen to come to mind first.

## Step 5 — Choose and lock in job boards

Boards are only as useful as the account behind them, so this step ties back to Step 1's confirmed email. Work through it in order and don't lock the list until the person has had a chance to customize it.

1. **Confirm the boards run under the same job-search account.** Remind the person that every board should be logged in under the same email confirmed in Step 1 — mismatched accounts are a common source of weak or wrong results.
2. **Enumerate the boards that will be referenced.** Propose the generic core from `assets/default_job_boards.md`, plus the industry-specific board(s) for their field (ask — there's almost always one, and it's frequently where the best senior, salary-transparent roles live). List the full proposed set back to them explicitly so they can see exactly what will be scanned.
3. **Ask them to log in and set up / complete a profile on each board they're keeping** (if they haven't already). A complete, current profile meaningfully improves targeting and, on some boards, unlocks positions that aren't visible to logged-out or bare accounts. Prompt them to do this and **trust their confirmation** — you don't need to verify each login yourself.
4. **Let them customize the list before you lock it.** Explicitly invite them to: (a) **add** any boards not in the defaults — including paid/subscription or niche boards they use (these are read through their own logged-in session, never via credentials you handle); and (b) **remove** any boards they don't want scanned. Capture the exact search-results URL for any board where they already have a saved/filtered search.
5. **Lock in the final list.** Write it into persona Section 5, and update persona Section 0 (Environment & Access) to reflect the locked board set (including user adds/removes). For any paid/login-gated board, note in Section 5 that it is read via the person's logged-in session only, with a "needs sign-in" fallback.

## Step 6 — Build the tracker

Read `assets/tracker_template_spec.md`, then use the `xlsx` skill to build a new spreadsheet with that structure. Name it something the person will recognize, e.g. `JobTracker.xlsx`, and save it wherever they keep career-related files (ask if unclear). Backfill it with the job history gathered in Step 3.

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

Summarize what was built (persona file including its Environment & Access section, tracker, scheduled task and its cadence) and say plainly that the first couple of runs are a tuning period — encourage the person to correct anything the scan gets wrong (a missed role, an over-eager match, a disqualifier that should exist) and note that feedback belongs in the persona file's Calibration Log (Section 10) so it sticks. Remind them of the two things that keep the scan working run-to-run: stay signed in to their boards under the confirmed account in the browser Claude drives, and (if they use a spreadsheet app) keep the tracker file closed when a scan is scheduled to run so it can be written. Offer to run the first scan immediately rather than waiting for the schedule, so they get a concrete result to react to right away.
