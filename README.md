# job-scan-setup Skill Family: Deployment Package

## What's in this folder

- **SKILL.md** — the updated `job-scan-setup` skill (onboarding). Deploy to `/mnt/skills/user/job-scan-setup/SKILL.md`.
- **job-scan-setup-assets/persona_template.md** — the updated persona template (Section 3 location scoping, Section 4 rank-based rubric weighting). Deploy to `/mnt/skills/user/job-scan-setup/assets/persona_template.md`.
- **job-scan-SKILL.md** — the updated `job-scan` skill (the recurring engine). Deploy to `/mnt/skills/user/job-scan/SKILL.md`. Changes: Step 8's calibration note now hands off to `persona-review` for anything beyond a single on-the-spot correction; Step 4 now reads the person's rank-based category weights fresh each run instead of assuming a fixed assignment.
- **persona-review/SKILL.md** — **new skill**. On-demand calibration: lets the person revise their persona through a guided conversation, grounded in real tracker evidence, without hand-editing the file or waiting for a scan to surface a correction. Supports re-ranking the fit-score categories and changing location scope, in addition to the original disqualifier/threshold/board edits. Deploy to `/mnt/skills/user/persona-review/SKILL.md`.
- **local-setup-walkthrough.md** — companion guide for the LOCAL (Claude Desktop + Cowork) deployment path, referenced from SKILL.md's Step 0 and Step 8. The CLOUD path is documented directly in SKILL.md Step 8 (short enough not to need its own file).

## Why persona-review exists

Before this, there were exactly two ways to change a persona: hand-edit the markdown file directly, or wait for `job-scan`'s Step 8 to notice something felt off during a run and offer to log it. Neither is a real review mechanism — the first requires the person to understand the file's structure well enough to edit it correctly (and to know which section a given change belongs in), and the second only fires as a side effect of a scan actually running, never on the person's own initiative.

`persona-review` closes that gap. It's a dedicated skill for exactly one job: sit down with the person, pull real evidence from the tracker (which rows scored high vs. low, what they actually engaged with vs. ignored), and turn a vague complaint ("it's showing me junk" / "it missed something good") into a specific, durable edit to the right section of `persona.md` — disqualifiers, positive-reinforcement, thresholds, target level, boards, whatever actually needs to change — with a dated log entry explaining why. This is the mechanism that keeps the whole system's core value proposition (a rubric that gets sharper over time, not one that drifts back to generic keyword-matching) actually true in practice, not just in the persona template's own framing.

## How the fit-score rubric works, and what a person can customize

This is worth documenting explicitly since it's easy to get wrong (an earlier design treated the five category weights as fixed and hardcoded — this pass changed that).

**The five categories and five point values never change.** Every persona uses the same five scoring categories — Title & seniority match, Domain overlap, Skill/keyword overlap, Strategic/leadership fit, Mission alignment — and the same five point values: **30, 25, 20, 15, 10**, always summing to 100.

**What's personal is the assignment, chosen by rank, not by typed-in numbers.** During onboarding (Step 4c) and in `persona-review`, the person ranks the five categories from most to least important to them. Rank 1 gets 30 points, rank 2 gets 25, and so on down to rank 5 getting 10. This is deliberately a **ranking exercise, not a free-numbers exercise** — the person orders the categories; they don't propose their own point values or a different total. That constraint is intentional: it keeps every persona in this system scoring on the same 0–100 scale with the same five-value "shape," so tracker Fit Scores stay comparable across the system's users, and it keeps `job-scan`'s job simple (read the rank table, apply the matching point value, proportionally derive score bands) rather than having to handle arbitrary weight schemes.

If a person asks for something the mechanism doesn't support — e.g., "I want domain overlap to be worth 40 points" or "can two categories tie" — the correct response, in both `job-scan-setup` and `persona-review`, is to say plainly that the rubric currently only supports reordering the same five fixed values, not custom totals or ties. Don't quietly approximate a request like that with a same-shape re-ranking that doesn't actually reflect what was asked; say what the system can and can't do and let the person decide whether the closest supported option is good enough.

**Location scoring is now scoped to the person's real situation, not forced into one shape.** The old default was always a full 6–7-location ranked table (1–8 scale). Onboarding's Step 4b now asks first whether that shape even applies: a person who is remote-only doesn't need a ranked table at all (remote scores 1; onsite/hybrid becomes a low score or disqualifier, by their choice); someone with only one or two real locations gets a short table sized to just those; only someone with a genuine range of acceptable places gets the full 6–7-slot ranking. `persona-review` can move a person between these shapes later (e.g., a fully remote worker who becomes open to relocating) — same section, different shape, chosen the same way onboarding chooses it the first time.

**Net effect on customization surface**: everything that was already customizable (threshold, disqualifiers, positive-reinforcement, salary floor, boards, target level, search seeds) still works exactly as before. Two things that were previously fixed or one-size-fits-all are now genuinely personal: which category the rubric weights most heavily (via ranking, not free numbers), and how much location-ranking structure a person's persona actually carries.

## Reliance on the Cowork Project structure (hardened this pass)

Earlier drafts described the Cowork Project as optional for the LOCAL path — "a convenience for keeping things organized." That undersold it. With `persona-review` now in the picture, there are three separate skills (`job-scan-setup`, `job-scan`, `persona-review`) that all need to land on the exact same persona file, tracker, and materials every time they're invoked — not "probably the same folder" but *guaranteed* the same one. A bare folder pointed at by an ad-hoc Cowork task doesn't guarantee that; a named Cowork Project does, because it's a stable, selectable unit with its own persistent Instructions, Memory, and Context binding.

SKILL.md's Step 0 now treats Project creation as a required step on both paths (not just CLOUD, where it was already load-bearing since there's no local disk to fall back on), and Step 9's wrap-up explicitly tells the person: every future session — scheduled scans, manual scans, and `persona-review` check-ins — needs to run inside that same named Project. `persona-review` itself opens by confirming it's running in the right Project before touching anything, for the same reason.

## Revision history (read this first)

**First draft** described a fabricated Claude Desktop UI — a "Local Project vs. Cloud Project" toggle, generic Settings panels, OS-level task schedulers, and third-party integrations (n8n, Zapier, IFTTT) this skill doesn't have. Discarded, not patched.

**Second draft**, after a screenshot of the real "Create scheduled task" form, corrected the field names (Name, Instructions, Work in a project or folder, Default model, Frequency, Permissions) and added the **"Require this computer"** toggle — off by default, and the thing that actually grants local file access, separate from the folder selector.

**This draft**, after a second screenshot showing the same form *without* the toggle at all (folder control reads "Work in a project," no local-machine option), corrected a real gap in the second draft: it had wrongly treated the no-toggle case as "manual-only, no real automation." It isn't. Per Anthropic's docs, a scheduled task with no local-machine tie-in runs as **Cowork in the cloud** — "Claude's work runs on Anthropic's servers instead of your computer... scheduled tasks run with no device online." That's genuine unattended automation, just backed by a Cowork Project's cloud storage instead of a folder on disk.

**This draft**, after two screen recordings of a live claude.ai Cowork session, corrected the described path to skills, connectors, and plugins — reached from the **"+"** button beside the composer (Skills / Connectors / Plugins / Devices), not "Customize → Skills" as the written docs alone had suggested. Also newly documented: the **Project** dropdown beneath the composer (a live picker with pinned/other projects, "Create new project," "View all projects"), the **Frequency** dropdown beside it (defaults "Manual"), and a **Devices → "Run tasks on"** list showing registered machines — a mechanism related to, but distinct from, the "Require this computer" toggle on the scheduled-task form.

Sources cited throughout SKILL.md and local-setup-walkthrough.md:
- Schedule recurring tasks in Claude Cowork — support.claude.com/en/articles/13854387
- Organize your tasks with projects in Claude Cowork — support.claude.com/en/articles/14116274
- Use Claude Cowork on web, desktop, and mobile — support.claude.com/en/articles/15520349
- Upload files to Claude — support.claude.com/en/articles/8241126
- Use skills in Claude — support.claude.com/en/articles/12512180
- Use plugins in Claude — support.claude.com/en/articles/13837440
- Browse skills, connectors, and plugins in one directory — support.claude.com/en/articles/14328846

## The two real deployment paths

**LOCAL** — Claude Desktop + Cowork, "Require this computer" toggled ON. Files live in a real folder on the person's disk (`persona.md`, `tracker.xlsx`, `materials/` for resumes and cover letters, `logs/`, `drafts/`). The scheduled task only runs while that specific machine is awake with Claude Desktop open.

**CLOUD** — Cowork on web, mobile, or desktop with no local-machine toggle set. Files live inside a **Cowork Project's cloud storage** (not the person's disk, and not the same as a regular Claude Project's read-only file-upload knowledge base — a Cowork Project's storage is read/write and built for exactly this). The scheduled task runs on Anthropic's servers with no device needing to be online.

Both are genuine, verified, unattended-scheduling paths. Neither is a fallback to the other. A third option — not using Cowork at all — has no scheduling mechanism and is manual-only by necessity, not by limitation of this skill.

## What actually changed in the skill

**Two-path Step 0**: distinguishes LOCAL from CLOUD upfront, each with its own file-storage location and its own version of the file-connection test (local folder I/O for LOCAL; Cowork Project file I/O for CLOUD).

**Consolidated materials**: since a scheduled task — local or cloud — only has access to one folder/project, existing resumes and cover letters get consolidated into that single location (`materials/` locally, or uploaded into the Cowork Project for cloud) during Step 2, before persona-building. This is what lets the persona interview, tracker backfill, and Step 7's auto-tailored drafts actually reach the person's real documents.

**Step 8 scheduled task creation**: walks through the real "Scheduled" → "New task" → "Set up manually" (or "Create with Claude") flow for both paths — LOCAL fills in "Work in a project or folder" plus turns "Require this computer" ON; CLOUD fills in "Work in a project" (no toggle exists in that form) and points at the Cowork Project instead.

## Deployment checklist

- [ ] Read local-setup-walkthrough.md in full before facilitating a LOCAL-path onboarding
- [ ] Confirm the account has Cowork access for whichever path is chosen (Pro/Max/Team, or Enterprise with the relevant admin toggle enabled)
- [ ] For LOCAL: confirm Claude Desktop is updated to the latest version
- [ ] For CLOUD: confirm Cowork on web/mobile is actually available for this account (it's in beta / admin-gated per Anthropic's docs, so don't assume it's on)
- [ ] Verify `job-scan-setup`, `job-scan`, and `persona-review` all show up under the **"+"** button beside the composer → **Skills** in your environment, or find out from your Aeneas Bio admin how they're distributed internally if not
- [ ] Deploy `persona-review/SKILL.md` alongside the other two — it's a new skill, not an edit to an existing one, so it needs its own directory under `/mnt/skills/user/`
- [ ] Deploy the updated `job-scan-setup-assets/persona_template.md` to `/mnt/skills/user/job-scan-setup/assets/persona_template.md` — without this, `job-scan-setup` still references the old fixed-weight template and the rank-based rubric change won't actually take effect
- [ ] Run through onboarding's Step 4c once and confirm the resulting rank table sums to 100 (30+25+20+15+10) — a quick sanity check that the ranking, not free-typed numbers, actually produced the table
- [ ] Confirm the onboarding flow actually results in one named Cowork Project per person, and that the person is told its name explicitly before the session ends — this is now load-bearing for all three skills to find the same files
- [ ] Run through Step 0–9 once yourself for whichever path you're deploying, then run a `persona-review` session against that test setup, before using this with a real user — UI details drift over time, so re-verify against docs.claude.com/support.claude.com periodically

## Known gaps / things not independently verified

- Whether a Cowork Project's own scheduled-task creation flow differs from the top-level Scheduled tasks page.
- Whether "Require this computer" reads "(Claude Desktop (macOS))" on Mac, matching the confirmed Windows wording — not independently confirmed.
- The exact mechanism for getting existing local resumes/cover letters into a Cowork Project's cloud storage when the person is on the CLOUD path but the files currently sit on a computer they're not using Cowork from (e.g., they want CLOUD on their phone but the resumes are on a laptop) — SKILL.md flags this as something to ask the person about directly rather than assuming a specific transfer mechanism.
- The actual project-creation flow (the "Start from scratch" / "Import from a Claude project" / "Use an existing folder" choice) was described in Anthropic's written docs but not directly captured on video — the recordings only showed the picker used to select an *existing* project, not the creation dialog itself. Verify this step against the live app before relying on it.
- Whether the "+" → Skills submenu's ordering/content is per-account (showing only skills that account has enabled) or something closer to a full directory — the recording showed a relatively short, clearly-account-specific list (contractor-lifecycle, design, explain-usage, fto-search, import-memory, job-scan, job-scan-setup, morning...), consistent with "already enabled for this account" rather than "everything that exists."
- `persona-review` is newly written, not observed or tested against a live session — its structure follows the same conventions as `job-scan` and `job-scan-setup` (same tracker column references, same persona section numbering) and should be run once against a real or test persona/tracker before relying on it with an actual user, per the deployment checklist above.

## Confirmed via direct evidence (screenshots and screen recordings, not just docs)

**From the two "Create scheduled task" screenshots:**
- **Local form**: "Name," "Instructions," a row with "Work in a project or folder" + "Default model," then "Frequency" (defaults "Manual") and "Permissions" (defaults "Manually approve"), and a separate "Require this computer (Claude Desktop (Windows))" toggle — off by default, described as "Only runs while your computer is awake. Gives Claude access to the folders you've allowed on this computer."
- **Cloud form** (same dialog, different context): identical fields except the folder row reads "Work in a project" (no "or folder" option) and there is no "Require this computer" toggle at all.

**From the two screen recordings of a live Cowork session:**
- The composer (in Chat or Cowork mode) has a **Chat | Cowork** toggle, and — specific to Cowork — a **Project** dropdown and a **Frequency** dropdown (defaulting to "Manual") sitting directly beneath it, letting a project and cadence be set for the current task without visiting a separate page.
- Clicking **Project** opens a "Search projects" picker: a Pinned section, other projects (some with a people icon suggesting shared/team projects, at least one with a plain folder icon suggesting a personal project), then "Create new project" and "View all projects."
- The **"+"** attach button beside the composer opens: Add files or photos, Take a screenshot, then **Skills**, **Connectors**, **Plugins**, **Devices** as expandable submenus.
  - **Skills** submenu: an alphabetical list of skills already available to the account (observed: contractor-lifecycle, design, explain-usage, fto-search, import-memory, job-scan, job-scan-setup, morning, morning-briefing...), then "Manage skills" / "Browse skills."
  - **Connectors** submenu: toggleable list (observed: Box, Gmail, Google Calendar, Google Drive, Linear, n8n, Scholar Gateway, each with an on/off switch), plus "Add connector" / "Manage connectors."
  - **Plugins** submenu: installed plugins by name (observed: "Scout"), plus "Manage plugins" / "Browse plugins."
  - **Devices** submenu: "Run tasks on" followed by registered machines (observed: two entries, both "Claude Desktop (Windows)," with last-active times "21m ago" and "3d ago").
