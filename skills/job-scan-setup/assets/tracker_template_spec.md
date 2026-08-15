# JobTracker.xlsx build spec

Build this with the `xlsx` skill (read its SKILL.md before creating the file — don't hand-roll the spreadsheet). This spec is what to build, not how to drive the xlsx skill itself.

## Sheet1 (the only required sheet)

Row 1 headers, exactly this order and these names (the `job-scan` skill's write logic depends on this exact order):

| # | Column | Notes |
|---|--------|-------|
| 1 | Title | |
| 2 | Company | |
| 3 | Apply Date | Date this person applied, if they did |
| 4 | Decision Date | Date of outcome (rejection, offer, or "Cold Posting" retirement date) |
| 5 | Outcome | Free text: "New Lead", "Application Submitted", "Rejected", "Cold Posting", "Paused - ...", "Reviewed - ...", etc. |
| 6 | Skillset | Matched persona themes/keywords for this role |
| 7 | Location | |
| 8 | Salary Low | Numeric, USD-normalized |
| 9 | Salary High | Numeric, USD-normalized |
| 10 | Job Link | Full URL; used for dedupe and the stale-posting sweep |
| 11 | Notes | One-line fit rationale + work mode + any gate/caveat; append rather than overwrite on later runs |
| 12 | Identify Date | Date this row was first added by a scan |
| 13 | Posted Date | Date the posting itself went live, if discoverable |
| 14 | Location Score | 1–8 per the persona file's Section 3 table |
| 15 | No Salary Info | "Y" / "N" |
| 16 | Fit Score | 0–100 per the persona file's Section 4 rubric |
| 17 | CV Link | Path to an auto-drafted CV, only if the auto-tailoring feature is enabled and this row cleared threshold |
| 18 | Cover Letter Link | Same, for the cover letter |

Formatting: bold header row, freeze row 1, reasonable column widths (Title/Notes wider, dates/scores narrower). Nothing fancier is needed — this file gets read and appended to programmatically far more often than it gets looked at as a formatted document.

## Backfilling from past applications

If the person supplied URLs to jobs they've already applied to (from the `job-scan-setup` interview step on job history), add one row per job with whatever fields are known:
- Title, Company, Job Link — always fill if the URL resolves
- Apply Date — ask the person, or leave blank if unknown
- Outcome — ask the person ("rejected", "no response", "interviewed but no offer", "still waiting," etc.) — don't guess
- Decision Date — only if the person knows/remembers it, or the sweep step later determines the posting is dead
- Fit Score / Location Score — leave blank at backfill time unless the rubric is fully built already; these can be filled on the next real scan pass rather than forcing the person to score their own history during onboarding

The goal of backfilling is dedupe safety (so the first real scan doesn't re-surface something they already applied to) and calibration material (Section 4 of the persona file leans on "what did this person actually apply to, and what happened" as signal) — it does not need to be perfectly complete before the system goes live.

## Optional second sheet: "Notes & Legend"

A short reference sheet explaining the columns and the scoring scale in plain language, so the tracker is still readable to the person (or a partner/spouse/advisor they show it to) without opening the persona file. Not required, but cheap to add and worth offering.
