# Quick Setup Guide

Get job-scan-toolkit running in your Claude environment in minutes.

## Requirements

- **Claude** (Haiku, Sonnet, or Opus model available in Claude.ai or Claude CLI)
- **Access to Claude skills** (via Claude.ai, Desktop, or your self-hosted Claude setup)
- **A storage backend** for your job tracker. Pick one:
  - **Google Drive** (easiest; no extra setup)
  - **Notion** (requires Notion integration token)
  - **GitHub** (requires repo access)
  - **Jira + Confluence** (requires workspace credentials)
  - **Local file** (works offline; limited to local machine)

See [MANUAL.md](MANUAL.md#storage-backends) for detailed backend setup instructions.

## Installation

### Step 1: Copy the skills into your Claude skills directory

job-scan-toolkit consists of three skills. Clone or download the repo and copy these folders:

```
skills/job-scan-setup/
skills/job-scan/
skills/persona-review/
```

into your Claude skills directory:

- **Claude.ai or Claude Desktop**: Your organization's skills folder (typically `/mnt/skills/user/` or a Cloud sync folder)
- **Self-hosted Claude**: Wherever you've configured your skills to load from

### Step 2: Verify installation

In Claude, you should see three new skills available:
- `job-scan-setup`
- `job-scan`
- `persona-review`

If they don't appear, check that the folder names match exactly and restart your Claude environment.

## Running Your First Scan

### Step 1: Onboard with `job-scan-setup`

Ask Claude:
```
Run job-scan-setup
```

or invoke it directly:
```
@job-scan-setup
```

The skill will guide you through a conversation to:
1. Provide your CVs/resumes and cover letters — point it at a local folder/file path, or a cloud location if you have a connector for it (old versions welcome; they show your trajectory)
2. Walk through your past applications, including rejections — a URL if you have one, or just title + company if you don't
3. Define your target job titles and seniority level
4. Describe the problems or domains you want to solve
5. List any disqualifiers (companies, roles, or conditions you want to avoid)
6. Choose which job boards to scan (LinkedIn, Indeed, AngelList, etc.)
7. Set location preferences
8. Pick your storage backend (Google Drive, Notion, GitHub, etc.) — no passwords, tokens, or API keys to hand over; storage connects through a connector you've already authorized (or your local filesystem), and job-board access separately runs through your own logged-in browser session via the Claude extension
9. Calibrate scoring thresholds and weights

**Time**: 10-15 minutes first time.

Once complete, your persona is saved and your job tracker is initialized. A recurring task is also created to run scans on your chosen cadence (e.g., weekly).

### Step 2: Run your first scan with `job-scan`

Ask Claude:
```
Run job-scan
```

or invoke it directly:
```
@job-scan
```

The skill will scan your configured boards and present a summary of new leads found. Your job tracker is automatically updated with new postings, and any tailored application drafts are optionally generated.

**Time**: 2-5 minutes per scan depending on board traffic.

### Step 3: (Optional) Tune your preferences with `persona-review`

As your search evolves, refine your persona without hand-editing:

```
Run persona-review
```

This opens a guided conversation to adjust your target seniority, problems of interest, disqualifiers, scoring weights, or which boards to scan. Changes are grounded in real tracker data.

## Troubleshooting

**Skills don't appear in Claude**
- Check folder names in `/mnt/skills/user/` or your configured skills directory
- Ensure no typos in `skills/job-scan-setup/`, `skills/job-scan/`, `skills/persona-review/`
- Restart Claude or reload the skills

**Storage backend connection fails**
- Verify credentials (API tokens, workspace ID, etc.) during setup
- For Google Drive: Ensure your account has been granted access to Claude
- For Notion: Double-check that your integration token has the right permissions
- See [MANUAL.md#storage-backends](MANUAL.md#storage-backends) for detailed backend troubleshooting

**Persona file is corrupted or lost**
- Re-run `job-scan-setup` to rebuild from scratch. This overwrites the old persona.
- Your job tracker is preserved (it's separate from the persona)

**No new leads found in scans**
- Your scoring thresholds may be too strict. Use `persona-review` to loosen disqualifiers or adjust the rubric.
- Check that your target job titles and location scope match actual postings on the boards you're scanning.
- Verify that the configured boards have roles matching your criteria (e.g., you might not find VP-level finance roles on AngelList, which skews startup)

## Next Steps

- Read [MANUAL.md](MANUAL.md) for deeper dives into architecture, storage backends, persona design, and advanced tuning strategies.
- Adjust your persona regularly as your search focus evolves (use `persona-review` for guided changes).
- Export or analyze your job tracker over time to spot trends in what gets scored highly.

## Support

For questions or issues, refer to the repo's issue tracker or reach out to your Claude admin or Aeneas.bio team.
