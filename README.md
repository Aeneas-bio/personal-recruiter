# job-scan-toolkit

A personalized, autonomous job-search scanning system built as Claude skills.

## Skills

- **`skills/job-scan-setup/`** — One-time onboarding. Interviews the person, builds their persona/rubric file, sets up job tracker storage (see `assets/storage_backends.md` for backend options — Notion, Jira+Confluence, GitHub, local file, or Google Drive), and creates the recurring scheduled task. Run once per person; re-running overwrites earned calibration.
- **`skills/job-scan/`** — The recurring engine. Reads the persona, sweeps for stale postings, scans configured job boards, scores and dedupes leads, optionally drafts tailored application documents, and delivers a summary. Currently still written against spreadsheet-tracker mechanics in a few steps (Inputs, Step 2, Step 5, Step 6, Step 7) — pending an update to call the backend-agnostic `JobStore` operations defined in `job-scan-setup/assets/storage_backends.md`.

## Status

Storage backend abstraction (`PersonaStore` / `JobStore`) is designed and wired into `job-scan-setup`. `job-scan` itself still needs the matching pass to stop assuming spreadsheet mechanics.

Persona-review skill (for revisiting/updating an existing persona outside a scan run) is planned but not yet built.

## Installing

Copy `skills/job-scan-setup/` and `skills/job-scan/` into your Claude skills directory (e.g. `/mnt/skills/user/` in this environment, or the equivalent for wherever you're running Claude).
