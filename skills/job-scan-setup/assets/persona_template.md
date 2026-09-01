# {{USER_NAME}} Persona & Job-Match Rubric

_Built by the `job-scan-setup` skill from: CV(s), cover letter(s), LinkedIn profile/posts, an interview about skills/certifications/projects, and a review of past job applications (including rejected ones — those still say a lot about what this person actually wants). This file is the single source of truth the `job-scan` skill reads every run. It is meant to be edited by hand over time — especially the Calibration Log at the bottom — as the person gives feedback on scan results._

_Last built: {{DATE}}_

---

## 0. Environment, Access & Storage

_Captured during onboarding (Steps 1 and 6 of `job-scan-setup`). The `job-scan` skill and scheduled runs read this on every run — keep it current if the person changes email, browser, boards, or storage backend._

### Access (pre-flight — Step 1)

- **Job-search account (email):** _{{fill}}_ — the single account all board logins, profiles, saved searches, and delivered summaries key off.
- **Browser + Claude extension:** _{{confirmed installed & enabled? date}}_ — authenticated boards are read through this logged-in session; Claude never sees, types, or stores passwords/tokens.
- **Account hygiene confirmed:** _{{date}}_ — person confirmed that ONLY this account is logged in (no other browsers/profiles signed into different accounts).
- **Locked-in job boards:** see Section 5 (finalized during onboarding, including any user-added paid/niche boards and any the person opted out of).
- **Authentication rule:** sign-in happens only in the person's own browser via the extension. If a board has no active session on a run, it is reported as "needs sign-in" and skipped — never accessed via stored credentials.

### Storage (Step 6)

_(See `assets/storage_backends.md` for backend options and capabilities. `job-scan` reads this block to know where the persona and job tracker actually live, without re-asking.)_

```yaml
storage:
  persona_backend: {{backend}}
  persona_location: {{page id, file path, or equivalent}}
  job_backend: {{backend}}
  job_location: {{database id, board key, file path, or equivalent}}
```

---

## 1. Who this person is today

Fill this in from the CVs, cover letters, and LinkedIn profile. Write it the way a strong executive-recruiter brief would: specific, not generic.

**Headline:** _(one or two lines — role, years of experience, domain, what they're known for)_

**Career arc:** _(the sequence of roles/companies that got them here — enough to spot the trajectory, not a full CV dump)_

**Signature themes:** _(recurring language from their own cover letters/LinkedIn posts — their stated mission, what energizes them, how they describe their own working style. Pull actual phrases where possible; people notice when a rubric doesn't sound like them.)_

**Core competencies / hard skills:** _(the specific technical and domain skills that show up across multiple roles or documents — not a generic skills list, the ones that are actually load-bearing for their target roles)_

**Keyword bank (for title/JD matching):** _(a flat list of domain terms, tools, methodologies, credentials — used later to score keyword density in job descriptions. Build this from the CVs/cover letters/LinkedIn, not from guessing what the industry generally values.)_

---

## 2. What this person wants next

**Target seniority:** _(e.g., Staff, Principal, Senior, Director, Senior Director, VP, Head of, Executive Director — ask the person directly which titles they consider "at level," "a stretch," and "a step down they'd still take if the role were exceptional")_

**Target role families (priority order):** _(list the 3–7 role/domain families this person is actually targeting, in order of demonstrated interest — inferred from what they've applied to, not just what they say they want. If those two things disagree, ask about the gap rather than picking one.)_

**Problems of interest:** _(the specific problems — not just the domain — this person wants to spend their next role solving. "Healthcare AI" is a domain; "getting clinicians out from under documentation burden with ambient capture" is a problem. Ask directly: not what industry, but what actual problem, stated the way they'd explain it to a peer. List 2–5, each with a line on why it matters to them if that came up in the interview. These drive the Problem fit component of Section 4's rubric — a role can nail the domain and title and still be a weak match if the actual day-to-day problem it's solving isn't one of these.)_

**Employers of interest (pattern, not exhaustive list):** _(the kinds of organizations — company size, sector, mission — this person gravitates toward, drawn from their application history)_

**Salary floor:** _(a hard number or range below which a role isn't worth surfacing. Ask directly — don't infer this from past offers alone, people's floors change.)_

**Work mode & geography:** _(remote/hybrid/onsite tolerance, relocation willingness, geographies in scope)_

---

## 3. Location preference score (1–8)

Ask the person to rank their top 6–7 preferred locations (metro areas, or "remote" if that's a genuine top preference), 1 = most preferred. Everything else falls into bucket 8.

| Score | Area |
|------|------|
| 1 | _{{fill}}_ |
| 2 | _{{fill}}_ |
| 3 | _{{fill}}_ |
| 4 | _{{fill}}_ |
| 5 | _{{fill}}_ |
| 6 | _{{fill}}_ |
| 7 | _{{fill}}_ |
| 8 | Anywhere else |

Notes: For fully remote roles, score by the company's HQ/hub if one anchors the role; otherwise 8, and note "Remote-friendly" (remote is usually a plus, not a neutral). For multi-location postings, use the best (lowest) qualifying location.

---

## 4. Fit score (0–100) — push through only if above the person's chosen threshold AND the salary rule passes

Ask the person what threshold they want to start at (a reasonable default to suggest is 65–70; it can be tuned down over time the way a hiring manager tunes a job-alert filter — one earlier user of this system started at 70 and lowered it to 60 after a few rounds of "you're being too conservative" feedback).

Weighted rubric — this structure is intentionally generic; the scoring judgment underneath it is what makes it personal:

- **Title & seniority match (0–20):** Exact target family at target level = 16–20; adjacent leadership = 9–15; IC/too-junior or off-domain = 0–8.
- **Domain overlap (0–20):** How much the role's actual subject matter overlaps this person's core domain (not just adjacent buzzwords). Strong core = 16–20; partial/general = 9–15; weak = 0–8.
- **Problem fit (0–25):** Does the role's actual day-to-day work — the responsibilities, not the department name — map to one of Section 2's Problems of interest? This is the single most important signal to get right, and the easiest one to fake by skimming: a role can sit in the right domain and still spend most of its time on a problem this person explicitly isn't excited about. Genuinely working on one of the listed problems, framed the way the person framed it = 20–25; adjacent problem in the same space = 10–19; domain match but no real overlap with any listed problem = 0–9. If a role clearly maps to a problem NOT on the list but seems like a strong match anyway, still score it fairly and flag it in Notes — this might be a gap in Section 2 worth adding via the Calibration Log, not a reason to suppress the lead.
- **Skill/keyword overlap (0–15):** Density of matched terms from the Section 1 keyword bank, in the title AND the body of the responsibilities — not just the requirements list.
- **Strategic/leadership fit (0–10):** Team-building, budget/P&L ownership, strategy-setting, stakeholder management, storytelling — whatever this person's actual leadership signature is.
- **Mission alignment (0–10):** Does the role's stated purpose match what this person says drives them (Section 1 signature themes)? Distinct from Problem fit above — mission is *why* the org exists, problem fit is *what work* the role actually does day to day; a role can hit one without the other.

**The most important instruction in this whole rubric, repeated because it's the thing that goes wrong first:** score based on how the target skills are actually *applied* in the responsibilities section of the JD, not on title or keyword density alone. A title can have every right word in it and still describe work this person doesn't want (see Disqualifiers below) — and a plainer title can still be a bullseye. Read the JD like a recruiter who has to defend the shortlist, not like a keyword matcher.

### Disqualifiers & conversion penalties

This section starts empty and gets built from two sources: (1) an interview about what this person does and doesn't want to do day-to-day, even within their target domain, and (2) their past application outcomes — rejections are signal. Ask directly: "Looking at your rejected or withdrawn applications, was there a role type, company type, or requirement that, in hindsight, was never going to work? What made it not work?"

Typical categories worth asking about explicitly (fill in with this person's real answers, don't assume):
- A sub-domain within their field that looks similar on paper but is actually different day-to-day work they don't want.
- Large-company or specific-employer-type roles with a hard "must have prior X-company-type experience" gate that historically hasn't converted for this person even with referrals.
- Categories that are a stretch right now but might not be in N months (e.g., "needs prior production rollout experience they're currently building") — worth a time-boxed pause rather than a permanent exclusion.
- Hard credential gates the person doesn't hold (an advanced clinical license, a specific board certification, a security clearance, etc.) — flag as a likely-ineligible near-miss rather than silently dropping it; the person may want to see it anyway.
- Roles that use the right domain language but the actual leadership signature doesn't match (e.g., "AI Director" roles that are actually deep technical work in a *different* sub-specialty than this person's — surface language match, weak substance match).

### Positive reinforcement

Also starts empty. Fill in from the interview and from roles this person got excited about (even if they didn't get an offer) — what made those roles score obviously high? Note the pattern here so future scans recognize it without re-deriving it every time.

**Decision rule:** create a job record if `Fit ≥ [threshold]` AND (`salary range touches the floor` OR `no salary posted`), AND not in a Disqualifier/Paused category. Dedupe against existing records via `JobStore.find_by_url()`, falling back to company + similar title. If the person already applied to a role that closed and reopened, score it on merit but note that in the record's `notes` rather than skipping it.

---

## 5. Job boards to scan

_(Filled in during onboarding — see the `job-scan-setup` skill's board-selection step for generic defaults plus industry-specific suggestions.)_

- {{board 1}}
- {{board 2}}
- ...

**Search title seeds:** _(the specific title variants worth searching verbatim, built from Section 2's target role families — e.g. "Director X", "Head of X", "VP X". Explicitly include a step to run "Director <domain>" style variants if Director-level roles are in scope; title-based search tends to under-surface these otherwise.)_

**Geographies:** _(pull from Section 3)_

---

## 6. Output: job record fields

The `job-scan` skill creates one job record per qualifying lead via the `JobStore.create()` operation for the backend configured in Section 0 (see `assets/storage_backends.md` for what that looks like per backend — a database row in Notion, an issue in Jira, etc.). The record fields are the same regardless of backend:

`title, company, url, source_board, date_found, stage, fit_score, salary_min, salary_max, salary_disclosed, notes, tailored_docs_url`

Field notes (map these onto whichever backend-native property/column names the chosen backend uses):
- New leads: `stage = "sourced"`, `date_found` = the scan's run date.
- `notes` = one-line fit rationale + work mode + any gate/caveat.
- `tailored_docs_url`: populated ONLY with newly generated draft documents for roles that clear the auto-tailoring threshold (Section 8) — never point this at the person's existing, real CV/cover letter files.
- Stage transitions (applied → interviewing → offer/rejected/withdrawn) happen via `JobStore.update()` as the person's real-world process moves, not automatically by `job-scan` itself except for the initial `sourced` stage and the `closed`/`stale` stages set by Section 9's sweep and staleness flag.

---

## 7. Run protocol (what `job-scan` does each time it runs)

1. Closed-posting sweep — check open, unresolved job records with real links; retire confirmed-dead postings to `closed`. Also flag any open record untouched past the staleness window as `stale` (Section 9).
2. Scan configured boards for the search seeds (and, per Section 2's Problems of interest, supplemental description-level searches) across configured geographies.
3. Score fit + location (read responsibilities, apply disqualifiers/penalties and Problem fit); test the salary rule; flag missing salary.
4. Dedupe against existing job records.
5. Create qualifying job records.
6. For any record clearing the auto-tailoring threshold, generate draft documents (Section 8).
7. Deliver a summary (Section 8 of the `job-scan-setup` config decides where: chat channel, email draft, or just a saved report).
8. Pause and ask if a strong title/JD match is genuinely ambiguous on fit — otherwise proceed autonomously.

---

## 8. Auto-tailored CV + cover letter (optional feature)

Ask during onboarding whether this person wants this at all, and if so, at what fit threshold (85+ is a reasonable starting suggestion — high enough that it only fires for standout matches). When a role clears the threshold and is actionable (open, not already applied/closed):

1. Start from this person's real base CV and cover letter (ask which files during onboarding).
2. Tailor emphasis and language to the specific JD's responsibilities and keywords, using Section 1's real language — never invent experience, credentials, or claims this person doesn't actually have.
3. Save into a per-job folder, e.g. `_AutoTailored/{Company}_{ShortTitle}/`, with `_draft` appended to every filename.
4. Put the new draft paths in the record's `tailored_docs_url` field via `JobStore.update()`.
5. Note "auto-draft, review before use" — these are first drafts for the person to edit, not final documents.

If this feature is declined during onboarding, leave `tailored_docs_url` unset always and skip this step.

---

## 9. Closed-posting sweep and staleness flag

Two distinct checks, both optional — ask about each separately during onboarding, don't bundle them as one yes/no:

**Closed-posting sweep** (cheap and generally worth it, but does spend time following links): for every open job record (stage `sourced` or `applied`) with a real `url`, follow the link. If the posting is gone/expired/no-longer-accepting (filled or taken down), call `JobStore.retire(job_id, reason)` — sets stage `closed`, preserving any prior notes. This is a verdict about the posting, not about the person's candidacy. If live, capture the posted date if shown via `JobStore.update()`, without touching stage. Don't overwrite records that already carry a richer, current stage (e.g., `interviewing`, `rejected`) — for those, only note liveness if useful.

**Staleness flag**: ask what window counts as stale (suggest 30 days as a starting point). Any open record (`sourced` or `applied`) whose notes/stage haven't changed within that window gets set to stage `stale` via `JobStore.update()`. This is independent of the closed-posting sweep above — it's a nudge to revisit a lead, not a claim about whether the job itself is still open. A record can be `stale` and still live, or go straight to `closed` before ever going stale.

**Authentication rule:** never collect, store, type, or ask for this person's account passwords. If a link is genuinely behind a login wall with no existing session, leave that record unchanged and report it under "needs sign-in" rather than attempting credentials.

---

## 10. Calibration log

_(Empty at first. Every time this person gives feedback on a scan's results — "you missed X," "stop surfacing Y," "actually that one's a great fit, here's why" — add a dated entry here summarizing the feedback and the rule change it produced. This log is what keeps the rubric from drifting back to generic keyword-matching over time. If this system is running somewhere with a persistent-memory feature, also save durable lessons there so they survive across sessions, not just in this file. For anything beyond a quick single correction, the `persona-review` skill runs a guided session for exactly this — it's the better path for the person to use directly rather than editing this document by hand.)_

- **{{DATE}}: