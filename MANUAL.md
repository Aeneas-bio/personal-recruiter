# job-scan-toolkit Manual

Comprehensive documentation on architecture, storage backends, persona design, and advanced usage.

## Overview

job-scan-toolkit is a personalized, autonomous job-search scanning system built as Claude skills. It automates the repetitive parts of job hunting (finding postings, filtering for fit, deduplication) while keeping the human in the loop for final decisions, application tuning, and preference calibration.

The system is built around three core concepts:

1. **Persona** - A structured profile of what you're looking for: target titles, seniority, problems of interest, disqualifiers, scoring weights, and board preferences.
2. **Job Tracker** - A persistent record of every posting found, whether it was a match, and what action you took (applied, skipped, bookmarked, etc.).
3. **Skills** - Claude skills that handle setup, recurring scans, and preference tuning.

## Skills

### `job-scan-setup`

**Purpose**: One-time onboarding to build your persona and initialize your job tracker.

**What it does**:
- Interviews you through a guided conversation to understand your target role, seniority, problems of interest, disqualifiers, location scope, and preferred job boards.
- Builds a persona file (structured YAML or JSON) that encodes your preferences and scoring rubric.
- Initializes a job tracker (in your chosen storage backend) with columns for posting ID, title, company, link, board source, posting date, your score, and action taken.
- Sets up a recurring scheduled task to run `job-scan` on your chosen cadence (e.g., weekly).
- Optionally integrates `persona-review` into the workflow for mid-search calibration.

**When to run**:
- Once per job search, at the start
- Re-run only if you want to start over from scratch (warning: this overwrites your persona and resets your tracker)

**Inputs**:
- Your conversational answers to calibration questions
- Storage backend choice and credentials (Google Drive, Notion, GitHub, Jira, or local file)
- Job board selections (LinkedIn, Indeed, AngelList, Blind, etc.)

**Outputs**:
- Persona file (stored in your chosen backend)
- Initialized job tracker (stored in your chosen backend)
- Scheduled task configuration (stored locally in Claude or your scheduler)

### `job-scan`

**Purpose**: Recurring engine that scans job boards, scores postings, and updates your tracker.

**What it does**:
- Reads your persona to understand what you're looking for
- Sweeps your job tracker for stale postings (removed from boards, past deadline)
- Calls your configured job board APIs or scraping endpoints to fetch current postings
- Scores each posting against your personal rubric (title match, problem fit, location, company disqualifiers, etc.)
- Dedupes against your tracker (skips roles you've already seen)
- Appends new qualified leads to your tracker with scores and metadata
- Optionally uses Claude to draft tailored CV and cover letter variants for top-fit roles
- Delivers a summary briefing: how many new leads, standout matches, trends, and next actions

**When to run**:
- On a schedule (weekly, bi-weekly, as configured in `job-scan-setup`)
- On-demand when you want to check for new postings mid-cycle

**Inputs**:
- Your persona file
- Your job tracker
- Job board credentials (API tokens, scraping endpoints, etc.)

**Outputs**:
- Updated job tracker (new leads appended, stale postings marked)
- Summary briefing (text or formatted document)
- Optional application drafts (tailored CV and cover letter for top matches)

### `persona-review`

**Purpose**: On-demand calibration of your persona without hand-editing.

**What it does**:
- Guides you through a structured conversation to review and refine your persona preferences
- Grounds the conversation in real evidence from your job tracker (e.g., "You've scored highly on roles with X skill; should we prioritize that more?")
- Lets you adjust target seniority, problems of interest, disqualifiers, positive reinforcement signals, scoring thresholds, board selections, and location scope
- Validates changes against your tracker to ensure they're realistic
- Saves refined persona back to your storage backend

**When to run**:
- Anytime your job search focus shifts
- After several scans, if you notice patterns (too many false positives, too many false negatives)
- Mid-search to recalibrate weights based on what you've learned

**Inputs**:
- Your existing persona file
- Your job tracker (for grounding evidence)
- Your refinement preferences (conversational guidance)

**Outputs**:
- Updated persona file (refined preferences)
- Calibration summary (what changed and why)

## Storage Backends

job-scan-toolkit uses an abstraction layer (`PersonaStore` / `JobStore`) to work with any storage backend. This lets you choose the tool that best fits your workflow without changing the skill logic.

### Supported Backends

#### Google Drive (Recommended for simplicity)

**Best for**: Users who want minimal setup and are already in Google Workspace.

**Setup**:
1. Grant Claude access to your Google Drive (typically done once during Claude setup)
2. During `job-scan-setup`, select "Google Drive" as your backend
3. The skill creates a "job-scan-toolkit" folder in your Drive root and initializes persona and tracker files there

**Storage format**:
- Persona: `job-scan-toolkit/persona.yaml` (structured YAML)
- Tracker: `job-scan-toolkit/job-tracker.csv` (tabular CSV with posting metadata)

**Pros**:
- No extra credentials needed (uses your existing Google Drive access)
- Easy to view, edit, or export tracker data
- Free tier has plenty of space for years of job search data

**Cons**:
- Requires active Google account
- Slower for very large trackers (10k+ rows) compared to database backends

---

#### Notion (Best for integration with other workflows)

**Best for**: Users who already use Notion for planning, notes, or application tracking.

**Setup**:
1. Create a Notion workspace (if you don't have one)
2. Create a new database for your job tracker (or link to an existing one)
3. Create an integration in Notion and generate an API token
4. During `job-scan-setup`, select "Notion" as your backend and paste your API token
5. Grant the integration read/write access to your tracker database

**Storage format**:
- Persona: Notion page in a dedicated "Job Search" folder
- Tracker: Notion database with rows for each posting, columns for score, status, notes, etc.

**Pros**:
- Easy to view and manually edit tracker entries
- Can embed custom properties (salary range, interview stage, etc.)
- Notion's UI makes it easy to filter or sort postings by score or status
- Integrates with other Notion workflows (calendar, notes, etc.)

**Cons**:
- Requires Notion account and API token setup
- Rate limits on free tier if scanning very frequently
- Notion API is slower than local file access

---

#### GitHub (Best for developers who want version history)

**Best for**: Technical users who want to track changes over time and integrate with their workflow.

**Setup**:
1. Create a GitHub repo (e.g., `job-search-private`)
2. Make it private (to keep your job hunt confidential)
3. Generate a personal access token with repo read/write permissions
4. During `job-scan-setup`, select "GitHub" as your backend and paste your token
5. Specify the repo and folder path for your tracker

**Storage format**:
- Persona: `job-search/persona.yaml` in the repo
- Tracker: `job-search/job-tracker.csv` in the repo
- Each scan run commits updates with a timestamp

**Pros**:
- Full version history of all changes (can revert if needed)
- Works completely offline (clone the repo locally)
- Git makes it easy to see what changed between scans
- Integrates with your development workflow

**Cons**:
- Requires GitHub account and token management
- Less visual than Notion or spreadsheets; you must edit YAML or CSV manually
- Privacy depends on repo permissions; accidental public repos expose your job search

---

#### Jira + Confluence (Best for enterprise workflows)

**Best for**: Users in organizations with Jira/Confluence instances who want centralized tracking.

**Setup**:
1. Get access to your organization's Jira and Confluence instances
2. Generate an API token for your account
3. During `job-scan-setup`, select "Jira+Confluence" as your backend
4. Paste your API token and specify your Jira project and Confluence space
5. The skill creates a Confluence page for your persona and a Jira project board for your tracker

**Storage format**:
- Persona: Confluence page in your specified space
- Tracker: Jira issues (one per posting) with custom fields for score, company, etc.

**Pros**:
- Leverages existing enterprise tools
- Jira board view makes it easy to see posting status at a glance
- Full audit trail of all changes
- Can link postings to other Jira work (e.g., upcoming interviews, hiring panel notes)

**Cons**:
- Requires Jira/Confluence instance access (not everyone has this)
- Setup and permissions can be complex
- Slower than local file systems due to API overhead

---

#### Local File (Best for offline, no-dependency approach)

**Best for**: Users who want zero external dependencies or work in air-gapped environments.

**Setup**:
1. During `job-scan-setup`, select "Local File" as your backend
2. Specify a local directory path (e.g., `~/job-search/`)
3. The skill initializes persona and tracker files there

**Storage format**:
- Persona: `~/job-search/persona.yaml`
- Tracker: `~/job-search/job-tracker.csv`

**Pros**:
- No external accounts, tokens, or setup needed
- Fast (local disk I/O)
- Complete privacy; data stays on your machine
- Easy to manually edit or version with local git

**Cons**:
- Limited to the machine where Claude runs
- No cloud backup (you must manage backups yourself)
- Can't sync across devices
- Less discoverable than cloud-based backends

---

### Choosing a Backend

| Use Case | Recommended Backend |
|----------|---------------------|
| Just getting started; minimal setup | Google Drive |
| Already use Notion; want integrated tracking | Notion |
| Developer; want version history | GitHub |
| Enterprise workflow; use Jira/Confluence | Jira + Confluence |
| Offline work; zero external dependencies | Local File |

You can also change backends mid-search. `persona-review` can help you migrate your persona and tracker to a new backend if your needs change.

## Persona Design

Your persona is the heart of job-scan-toolkit. It encodes what you're looking for and how to score postings.

### Persona Structure

A typical persona includes:

```yaml
name: "Alice's Job Search"
target_titles:
  - "Product Manager"
  - "Senior Product Manager"
  - "Director of Product"
seniority_level: "mid"  # "entry", "mid", "senior", "exec"
problems_of_interest:
  - "AI/ML infrastructure"
  - "biotech data pipelines"
  - "healthcare interoperability"
location_scope:
  - "US (remote preferred)"
  - "Bay Area"
  - "Boston"
disqualifiers:
  - "Finance-heavy roles"
  - "No remote"
  - "Seed-stage only"
board_preferences:
  - "LinkedIn"
  - "AngelList"
  - "Blind"
scoring_rubric:
  title_match: 0.3       # Weight for job title alignment
  problem_fit: 0.4       # Weight for problem domain match
  location_fit: 0.2      # Weight for location preference
  company_signal: 0.1    # Weight for company reputation/stage
thresholds:
  qualified_lead: 65     # Minimum score to add to tracker
  drafting_candidate: 80 # Minimum score to draft application
positive_reinforcement:
  "startup stage": "+10 to score"
  "data-heavy": "+15 to score"
```

### Persona Tuning

After a few scans, you'll notice patterns:
- **Too many false positives**: Tighten disqualifiers or raise thresholds
- **Too few leads**: Relax disqualifiers or broaden problem domain
- **Missing companies you care about**: Add them to `board_preferences` or adjust company scoring
- **Roles you love aren't scoring high enough**: Adjust `scoring_rubric` weights to match your real preferences

Use `persona-review` to make these adjustments conversationally rather than hand-editing YAML.

## Advanced Usage

### Tailored Application Documents

If a posting scores high enough (default: 80+), `job-scan` optionally uses Claude to draft a tailored CV and cover letter for that role. These drafts:
- Extract key requirements from the job posting
- Customize your CV to highlight relevant experience and skills
- Draft a cover letter that speaks to the company's mission and the specific role

You review and edit these drafts before sending.

### Engagement Tracking and Reporting

Over time, your job tracker becomes a valuable dataset:
- Which roles you've applied to and outcomes
- Which boards yield the most qualified postings
- Which problems/domains have the most opportunity
- Trends in hiring (e.g., remote roles increasing or decreasing)

You can export your tracker to analyze these trends or share anonymized data with mentors or advisors.

### Re-calibration Signals

`job-scan` watches for calibration signals as you interact with results:
- **High scores you skip**: Maybe your persona is missing context about what you really want
- **Low scores you love**: Adjust your rubric; you're not scoring what matters to you
- **Repeated false positives from certain boards**: Consider removing them from `board_preferences`

`persona-review` uses this feedback to suggest improvements.

### Scheduled Runs and Notifications

`job-scan-setup` creates a recurring task to run scans on your chosen cadence. You can also:
- Run scans manually anytime with `@job-scan`
- Adjust the schedule mid-search (e.g., increase frequency as you get closer to a job change)
- Set up notifications or webhooks to alert you when qualified leads are found

### Exporting and Sharing

Your job tracker (stored in your backend) can be:
- Exported to CSV for analysis in Excel or Python
- Shared with a mentor, recruiter, or advisor (anonymized if desired)
- Mirrored to multiple backends if you want redundancy

## Troubleshooting

### Setup Issues

**Q: I started setup, then lost the conversation. Do I have to start over?**
A: Yes, `job-scan-setup` creates your persona and tracker as it goes. If the conversation is interrupted, the partial files may be left behind. You can either delete them and re-run, or manually complete the persona file and run `job-scan-setup` again to pick up where you left off.

**Q: My storage backend credentials aren't working.**
A: Double-check:
- Google Drive: Ensure your Claude instance has Google Drive access granted
- Notion: Your API token is current and the integration has read/write permissions on your database
- GitHub: Your personal access token has repo permissions and hasn't expired
- Jira/Confluence: Your API token is current and your user has access to the specified project/space
- Local file: The directory path exists and is writable

### Scan Issues

**Q: No new leads are being found.**
A: Possible causes:
- Your scoring thresholds are too strict. Use `persona-review` to loosen disqualifiers or lower the `qualified_lead` threshold.
- Your target titles or problem domains don't match postings on your chosen boards.
- The boards you selected don't have active postings in your location/seniority.

Try running `persona-review` to adjust your rubric or add additional boards.

**Q: I'm seeing duplicate postings.**
A: This can happen if:
- A posting is re-posted on multiple boards (the deduplication logic checks within your configured boards, but postings can appear on multiple boards)
- Your tracker wasn't properly initialized. Manually check for duplicates in your tracker and mark one as a duplicate before the next scan.

**Q: The drafted application documents don't feel personal enough.**
A: The drafts are starting points. Edit them before sending to add specific examples, your voice, or company-specific details. As `job-scan` runs more scans, the drafting logic learns from the postings it sees and can improve its templates.

### Persona Issues

**Q: I changed my mind about what I want mid-search. Should I start over?**
A: No. Use `persona-review` to adjust your persona. This is exactly what it's designed for. Your tracker is preserved, so you keep the history of roles you've already evaluated.

**Q: My persona file got corrupted or deleted.**
A: Depending on your backend:
- **Google Drive / Notion / GitHub / Jira**: The file is recoverable from your backend's history or recycle bin
- **Local file**: You'll need to restore from a backup or re-run `job-scan-setup` to start fresh

To avoid this, back up your persona regularly.

**Q: How often should I recalibrate my persona?**
A: Every 2-4 weeks, or whenever you notice your scan results don't feel right. `persona-review` makes recalibration quick and interactive.

## Architecture and Extensibility

### High-level Flow

```
job-scan-setup (run once)
  ↓
  [Interview user, build persona, init tracker, schedule recurring task]
  ↓
job-scan (run on schedule or on-demand)
  ↓
  [Read persona, scan boards, score postings, update tracker, draft docs]
  ↓
  [User reviews results]
  ↓
persona-review (run as needed)
  ↓
  [Adjust persona based on feedback or tracker evidence]
  ↓
  [Go back to step 2]
```

### Backend Abstraction

The `PersonaStore` and `JobStore` interfaces abstract away backend details:

```
PersonaStore.read() → returns persona dict
PersonaStore.write(persona) → saves persona
JobStore.read() → returns list of job postings
JobStore.append(new_postings) → adds to tracker
```

Each backend (Google Drive, Notion, GitHub, etc.) implements these interfaces. The `job-scan` skill calls `PersonaStore.read()` and `JobStore.append()` without knowing which backend you chose.

This design makes it easy to add new backends or swap backends mid-search.

### Adding a New Backend

To add a new storage backend (e.g., Airtable, Monday.com, etc.):

1. Create a new backend module implementing `PersonaStore` and `JobStore` interfaces
2. Add it to the backend registry in `assets/storage_backends.md`
3. Update `job-scan-setup` to prompt for the new backend
4. Update the backend factory to instantiate the correct backend based on user choice

See `assets/storage_backends.md` for the interface definitions and examples.

## FAQ

**Q: Can I search multiple job boards at once?**
A: Yes. During setup, you select all boards you want to scan. Each `job-scan` run hits all of them.

**Q: Can I run scans on multiple computers?**
A: If your backend is cloud-based (Google Drive, Notion, GitHub, Jira), yes. Your tracker syncs. If you're using local files, you must manually sync or use local git.

**Q: What happens if I mark a role as "applied" but the posting gets removed from the board?**
A: Your tracker preserves your action (applied). In the next scan, the skill marks that posting as stale/dead so it won't re-appear in future searches.

**Q: Can I use this for other types of searches (internships, freelance gigs, contracts)?**
A: Yes. The persona framework is flexible. You can define target titles, problem domains, and scoring rubrics for any role type.

**Q: How do I know if I'm scoring too strictly or too leniently?**
A: Track your outcomes. If you apply to a role and get interviews, that suggests your scoring is good. If you skip high-scoring roles and later regret it, you're scoring too strictly. Use `persona-review` to adjust based on these signals.

**Q: Can I export my tracker to see trends over time?**
A: Yes. Your tracker is stored as CSV (Google Drive, GitHub, local file) or a database (Notion, Jira) that you can export. You can analyze it in Excel, Python, or visualization tools.

## Support and Contribution

For questions, bug reports, or contributions, open an issue on the repo or reach out to your Claude admin or Aeneas.bio team.
