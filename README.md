# job-scan-toolkit

A personalized, autonomous job-search scanning system built as Claude skills.

## Skills

- **`skills/job-scan-setup/`** — One-time onboarding. Interviews the person, builds their persona/rubric file, sets up job tracker storage (see `assets/storage_backends.md` for backend options — Notion, Jira+Confluence, GitHub, local file, or Google Drive), and creates the recurring scheduled task. Run once per person; re-running overwrites earned calibration.
- **`skills/job-scan/`** — The recurring engine. Reads the persona, sweeps for stale postings, scans configured job boards, scores and dedupes leads, optionally drafts tailored application documents, and delivers a summary. Calls the backend-agnostic `PersonaStore`/`JobStore` operations defined in `job-scan-setup/assets/storage_backends.md` throughout, so it works the same regardless of which storage backend (Notion, Jira+Confluence, GitHub, local file, Google Drive) was chosen during setup.

## Status

Storage backend abstraction (`PersonaStore` / `JobStore`) is designed and wired into both `job-scan-setup` and `job-scan`.

Persona-review skill (for revisiting/updating an existing persona outside a scan run) is planned but not yet built.

## Installing

Copy `skills/job-scan-setup/` and `skills/job-scan/` into your Claude skills directory (e.g. `/mnt/skills/user/` in this environment, or the equivalent for wherever you're running Claude).
