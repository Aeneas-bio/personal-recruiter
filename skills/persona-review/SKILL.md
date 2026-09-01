---
name: "persona-review"
description: "On-demand calibration session for an existing job-search persona built by `job-scan-setup` — lets the person revise their persona.md (target level, disqualifiers, positive-reinforcement patterns, salary floor, boards, thresholds, location scope, and which of the five fit-score categories gets weighted highest) through a guided conversation instead of hand-editing the file or waiting for the next `job-scan` run to surface a correction. Pulls real evidence from the tracker (recent Fit Scores, Outcomes, Notes) so the conversation is grounded in what actually happened, not abstract preference-guessing. Trigger this whenever the person wants to: review or discuss recent scan results, say a scan is missing good roles or surfacing bad ones, adjust their target seniority/salary/locations/boards, add or remove a disqualifier, change their fit threshold, re-rank which scoring category matters most to them (e.g., \"domain overlap should matter more than title match\"), change their location scope (e.g., going remote-only, or narrowing to one city), or generally \"tune,\" \"fix,\" \"recalibrate,\" or \"go over\" their job search persona. Also trigger on phrases like \"why did it surface X,\" \"stop showing me Y,\" \"I'm also open to Z now,\" or \"can we sit down and go through my results.\" Requires an existing persona file and tracker built by `job-scan-setup` — if those don't exist yet, run `job-scan-setup` first. Runs inside the same Cowork Project as `job-scan-setup` and `job-scan` so it sees the same persona, tracker, and materials."
---

# Persona Review

`job-scan`'s Step 8 can log a quick calibration note as a side effect of a run, and a person can always hand-edit `persona.md` directly — but neither of those is a real review. This skill is the third option: a deliberate, guided session where the person and Claude go through what's actually happened, and the persona file gets revised properly — not by patching in whatever the person says verbatim, but by asking the same kind of follow-up questions `job-scan-setup`'s original interview asked, so a vague complaint ("it's showing me too much noise") turns into a specific, durable rule the next scan can actually apply.

This is the core mechanism that keeps calibration honest over time. A persona built once during onboarding and never revisited will drift back toward generic keyword-matching as the person's target shifts — a new disqualifier they've learned about their own preferences, a title band that opened up, a salary floor that moved. This skill is how that drift gets corrected on the person's own schedule, not just when a scan happens to prompt it.

## Before you start: locate the real files

1. **Confirm you're in the right Cowork Project.** This system's `job-scan-setup` skill sets up one dedicated Cowork Project per person (see its Step 0) containing `persona.md`, `tracker.xlsx`, and `materials/`. If you're not sure you're inside it, ask the person to confirm the project name, or check the Project selector. Running this review inside the wrong project (or no project) means editing a file that isn't the one `job-scan` actually reads on its next run — a wasted conversation.

2. **Read the full persona file, not a summary.** Same rule `job-scan` follows: load the entire file fresh every session, including Section 10's Calibration Log, so you know what's already been corrected and don't re-litigate a settled point.

3. **Read the tracker.** Pull recent rows — especially `Fit Score`, `Outcome`, and `Notes` columns (see `job-scan-setup/assets/tracker_template_spec.md` for exact column order) — for the period since the last review (or since onboarding, if this is the first review). This is your evidence base. A review grounded in "here's what actually scored high vs. what you actually engaged with" produces sharper corrections than one based purely on the person's unaided memory of a scan they skimmed once.

4. **If no persona/tracker exists yet**, stop and say so — point to `job-scan-setup` instead of trying to conduct a review with nothing to review.

## Step 1 — Set the frame

Tell the person plainly what this session will do: go through recent results together, identify what the scan got right and wrong, and turn that into specific changes to the persona file — not a vague "we talked about it," but dated, concrete edits to the actual rubric. Ask whether they want to:
- **Review a specific recent scan** (if they came in reacting to one), or
- **Do a general check-in** (no specific trigger — just want to tune things), or
- **Make one specific change** they already know they want (e.g., "raise my salary floor") without a broader review.

All three are legitimate; the depth of the conversation should match what they actually want, not default to the fullest version every time.

## Step 2 — Walk the evidence, not just the complaint

If reviewing recent results, don't just ask "how did it go?" — bring the tracker data into the conversation directly:

- **Surface specific rows.** "Since your last review, the scan added 14 new leads. 3 scored 85+ and you marked one 'Application Submitted' — the other two are still sitting at 'New Lead.' 4 scored in the 60s and got no action from you. Want to go through any of these?"
- **For a "too much noise" complaint**: pull the lowest-scoring rows that still cleared the threshold and ask specifically what was wrong with each — was it the title, the company type, the actual responsibilities, something about seniority? A generic "lower the threshold" fix often isn't the real problem; a missing disqualifier usually is, and disqualifiers are more durable than threshold tweaks because they catch the *pattern*, not just this batch.
- **For a "missing good roles" complaint**: ask if they've seen a specific role elsewhere that the scan should have caught. If so, pull it up and score it manually against the current persona — the gap between what it should have scored and what it would have scored under the current rubric tells you exactly what to fix (a missing keyword, a title variant not in the search seeds, a domain the persona undersells).
- **For engaged rows** (applied, interviewed, especially if they got excited about one): ask what specifically made it appealing. This is Section 4's "Positive reinforcement" material — as valuable as fixing what's wrong.

Push past the first answer the same way the original `job-scan-setup` interview does: "it seemed off" is not yet a rule; "the responsibilities were 80% people-management and I want to stay hands-on technical" is.

## Step 3 — Turn the conversation into specific rule changes

Map what you've learned onto the actual persona sections it belongs in — don't just append everything to the Calibration Log and call it done, since a log entry alone doesn't change scoring behavior unless the earlier sections also change:

- **New disqualifier or conversion penalty** → add to Section 4's Disqualifiers list, following the template's own format (name the pattern, not just "role at Acme Corp didn't work" — the next scan needs a rule, not an anecdote about one company).
- **New positive-reinforcement pattern** → add to Section 4's Positive reinforcement list, same way.
- **Threshold change** (fit score, salary floor) → update the actual number in Section 2 or Section 4, don't just note "lower the threshold" in the log.
- **Target level/title changes** → update Section 2's target seniority/role families and Section 5's search title seeds together — a level change that isn't reflected in the search seeds won't actually change what gets scanned.
- **Board changes** (add/remove/adjust) → update Section 5's board list directly.
- **Location scope or ranking changes** → update Section 3. This might mean re-ranking an existing table, shrinking a full ranked table down to one or two real locations (or to a remote-only rule with no table), or the reverse if their situation opened up — match the table's shape to whatever the person actually describes now, the same way onboarding's Step 4b does, rather than assuming the original structure still fits.
- **Rubric category re-ranking** (e.g., "domain overlap matters more to me than title match now") → this is a **re-ranking**, not a re-numbering. The five point values in Section 4's rank table are always exactly 30/25/20/15/10 — persona-review can change *which category holds which value*, by walking the person through the same five-category ranking question onboarding's Step 4c asks, but must never let the person assign a point value outside that set or propose a different total. If someone asks for something the ranking mechanism genuinely can't express (e.g., "I want domain overlap to count for 40 points"), say plainly that the rubric currently only supports reordering the same five fixed values, not custom point totals — don't quietly approximate their request with a re-ranking that doesn't actually reflect what they asked for.
- **Anything that doesn't cleanly map to an existing section** (a genuinely new kind of judgment call) → this is what Section 10's Calibration Log is for. Write it as a durable rule, not a transcript of the conversation: "2026-09-12: Deprioritize roles requiring on-call rotation even at otherwise-strong fit — person confirmed this is a hard no regardless of title/comp" is useful to a future scan; "talked about on-call stuff" is not.

**Always add a dated Section 10 entry summarizing what changed and why**, even when the substantive edit lives elsewhere in the file — the log is the audit trail that lets a future review (or a future instance of `job-scan` hitting an edge case) understand why the rubric looks the way it does, not just what it currently says.

## Step 4 — Confirm before writing

Read back the specific changes you're about to make — in plain language, referencing the actual section numbers — and get explicit confirmation before editing `persona.md`. This file is the rubric of record for an autonomous recurring process; a misunderstood correction here doesn't just affect this conversation, it silently reshapes every future scan until someone notices and corrects it again. A quick "so I'm going to: raise your salary floor to $X in Section 2, add a disqualifier for roles requiring on-call in Section 4, and log both changes in Section 10 — sound right?" is worth the extra turn.

## Step 5 — Make the edits and report back

Edit `persona.md` directly (not a copy, not a draft — the real file `job-scan` reads). After writing:
- Confirm the file still opens/parses cleanly (same discipline `job-scan` applies to the tracker — a malformed edit here is worse than no edit, since it's silent until the next scan run).
- Summarize exactly what changed, section by section, in the same message — don't make the person open the file to see what happened.
- If persistent memory is available in this environment, also save the durable lesson there (same as `job-scan`'s own note on this), so it survives even if this particular Calibration Log entry is later trimmed or the file is rebuilt.

## Step 6 — Offer a concrete next step

Depending on what changed, offer one of:
- **Run `job-scan` now** so the person can see the effect of the change immediately, rather than waiting for the next scheduled run — this is often the most convincing way to confirm a fix actually worked.
- **Re-score a specific role** they mentioned during the conversation, on the spot, against the newly updated rubric, so they can sanity-check the change before it goes live on autopilot.
- Just confirm the changes are saved and will apply starting with the next scheduled run, if they don't want to test it live right now.

## A note on frequency and tone

This isn't a form to fill out on a fixed cadence — some people will want this after every scan for the first few weeks, then rarely; others will want one thorough session and then only come back for a specific complaint. Match the depth to what they're asking for (Step 1), and don't manufacture urgency or suggest a review is "due." The person deciding to come back is itself the signal that something's worth revisiting.
