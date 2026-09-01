# Local Setup Path for job-scan-setup: Claude Cowork Walkthrough

**A note on sources**: This document combines Anthropic's published documentation (support.claude.com, cited inline) with direct observation of the actual interface from two screen recordings of a live claude.ai Cowork session. Where the two disagreed, the recording wins, and the correction is noted. An earlier version of this guide contained invented UI details (a fictional "Local Project vs. Cloud Project" toggle, fictional Settings panels); that version was discarded, not patched.

---

## What "local setup" actually means

There is no separate "Claude Desktop project" system distinct from Cowork. The real picture, combining docs and the recordings:

- **Claude Desktop** is the app you install. It has a **Chat** tab and a **Cowork** tab, selected right in the message composer (a two-button toggle: "Chat" | "Cowork").
- **Cowork** is where local file access, skills/plugins/connectors, and **scheduled recurring tasks** live. Cowork requires a paid plan (Pro, Max, Team, or Enterprise). It's also available on web and mobile, not just Claude Desktop — the composer's Chat/Cowork toggle appears the same way on claude.ai in a browser.
- **Projects** are selected from a dropdown directly beneath the composer (labeled "Project" until one is chosen). Clicking it opens a **"Search projects"** picker showing pinned projects, other projects, and **"Create new project"** / **"View all projects"** at the bottom — this is the actual entry point, not a separate top-level "Projects" settings page. Some projects in the picker show a people icon (shared/team projects); at least one observed project showed a plain folder icon, suggesting a personal, non-shared project — the distinction matters for whether job-search materials might be visible to teammates, so confirm which kind you're creating.
- **Frequency/scheduling** for the current task has its own dropdown right beside the Project one (labeled "Manual" by default) — this is where the recurring cadence gets set inline, in addition to (or instead of) the separate top-level "Scheduled" page.
- **Skills** (including `job-scan-setup`) are reached from the **"+"** button beside the composer → **Skills** → an alphabetical list of everything already available in the account, with **"Manage skills"** and **"Browse skills"** at the bottom for installing anything not already there. The same "+" menu also has **Connectors** (Box, Gmail, Google Calendar, Google Drive, Linear, n8n, and more, each with an on/off toggle) and **Plugins** (installed plugins listed by name, e.g. one seen was "Scout," plus "Manage plugins" / "Browse plugins").
- **Devices**, also in that "+" menu, shows **"Run tasks on"** followed by a list of registered machines — in the recording, two entries both labeled "Claude Desktop (Windows)" with last-active timestamps ("21m ago," "3d ago"). This is the mechanism that actually ties a task to a specific local computer, separate from (and likely underlying) the "Require this computer" toggle seen on the scheduled-task creation form.
- **Scheduled tasks** are also created from the **"Scheduled"** item in the left sidebar. A scheduled task that needs local files only runs while a specific registered device is on and Claude Desktop is open there.

This means the job-scan-setup onboarding on the LOCAL path is really: **install/update Claude Desktop -> confirm the job-scan-setup skill is available via the "+" → Skills menu -> pick or create a Cowork project for job search -> pick a workspace folder -> consolidate existing resumes/cover letters into it -> test file access -> run onboarding -> create the scheduled task, tied to this device, via the Scheduled tab.**

Sources:
- Schedule recurring tasks in Claude Cowork -- support.claude.com/en/articles/13854387
- Organize your tasks with projects in Claude Cowork -- support.claude.com/en/articles/14116274
- Use skills in Claude -- support.claude.com/en/articles/12512180
- Use plugins in Claude -- support.claude.com/en/articles/13837440
- Browse skills, connectors, and plugins in one directory -- support.claude.com/en/articles/14328846
- Direct observation from two screen recordings of a live claude.ai Cowork session (2026-08-31), confirming the "+" menu structure, the Project picker, and the Devices/"Run tasks on" list

---

## Pre-Setup Checklist

- [ ] User is on a paid Claude plan (Pro, Max, Team, or Enterprise) -- Cowork and scheduled tasks require this
- [ ] User has a Mac or Windows machine to run Claude Desktop on
- [ ] Claude Desktop is installed and updated to the latest version (Cowork projects explicitly require "the latest version")
- [ ] User has write permission to wherever they'll put their job-search workspace folder
- [ ] User has access to their job board accounts (LinkedIn, Indeed, etc.) in their regular browser

---

## Phase 1: Install / Update Claude Desktop

1. **Ask**: "Do you already have Claude Desktop installed, and is it up to date?"
2. **If not installed**: download from claude.com/download (Windows) or the macOS download link on the same page.
3. **If installed but possibly old**: Cowork projects require the latest version -- have the user check for updates (the app typically prompts, or check within the app's own update mechanism) before proceeding. If Cowork or the Scheduled tab doesn't appear as described below, this is the first thing to check.
4. **Launch Claude Desktop** and confirm you can see both a **Chat** tab and a **Cowork** tab. If there's no Cowork tab at all, the account may not be on a plan that includes it (Pro, Max, Team, Enterprise) -- confirm plan level before troubleshooting further.

---

## Phase 2: Confirm or Install the job-scan-setup Skill

**Confirmed from a screen recording of the actual interface** (claude.ai web, Cowork tab): skills are reached from the **"+" attach button** to the left of the message composer, not a separate top-level Customize page. Clicking "+" opens a menu with **Add files or photos, Take a screenshot, Skills, Connectors, Plugins, Devices**. Hovering **Skills** opens a submenu listing every skill currently available in the account, alphabetically, followed by **"Manage skills"** and **"Browse skills"** at the bottom.

1. Click the **"+"** button next to the composer (works in both Chat and Cowork).
2. Hover or click **Skills**.
3. Look down the list for **`job-scan-setup`**. In the account this was recorded from, it's already there, listed right next to `job-scan`, `import-memory`, `morning`, `fto-search`, and others — if your organization has already distributed it, this is likely all you need to do to confirm it.
4. If it's **not** in the list, click **"Browse skills"** at the bottom of the submenu to open the shared/marketplace directory, search for "job-scan-setup," and install it from there. If it still doesn't turn up, it may be distributed as an org-specific plugin instead — check with your Aeneas Bio admin rather than guessing at a workaround.

**Verify installation**: skills listed here are available in both Chat and Cowork sessions. Starting a new Cowork task and describing a job-search-setup task in plain language should trigger `job-scan-setup` automatically, or you can reference it directly.

(An earlier version of this document described a "Customize → Skills → + → Browse skills" path based on Anthropic's written docs alone. The screen recording shows skills are actually reached from the "+" button beside the composer — the docs' "Customize" area may be a settings page for managing what's already installed, distinct from where you add things in the moment. This section has been corrected to match what was actually observed.)

---

## Phase 3: (Optional) Create or Choose a Cowork Project for Job Search

This step is optional but recommended if the user wants job-search context (files, instructions, memory) kept separate from other Cowork work. **Confirmed from the recording**: this doesn't require a separate top-level "Projects" page — the entry point is the **"Project"** dropdown directly beneath the message composer (next to the "Manual"/frequency dropdown). Clicking it opens a **"Search projects"** picker showing:
- A **Pinned** section at top, if any projects are pinned
- A flat list of other projects the account has access to
- **"Create new project"** and **"View all projects"** at the bottom

1. Click the **Project** dropdown beneath the composer.
2. If a job-search project already exists, search for it or scroll to find it in the list, and select it.
3. If not, click **"Create new project"**.
4. When creating: per Anthropic's written docs (not directly captured in the recording, which only showed the picker, not the creation flow), you'll be offered **Start from scratch**, **Import from a Claude project**, or **Use an existing folder on your computer** — the last option is the natural choice if you're about to pick a workspace folder in Phase 4 below: do that first, then point the new project at it.
5. Name the project (e.g., "Job Search") and confirm creation.

**A distinction worth noting from the recording**: projects in the picker showed two different icons — a people icon on several (e.g., "Training & Workshops," "Platform Integrations," "Marketing," "'spec") suggesting shared/team projects, and a plain folder icon on at least one ("Yobs") suggesting a personal, non-shared project. Confirm which kind you're creating or selecting for job-search work — if the account has both personal and shared/team projects available, a job search almost certainly belongs in a personal one, not a shared team project other people can see into.

What a project gives you, per Anthropic's docs: its own **Instructions**, its own **Context** (a folder, imported chat-project files, or a linked URL), its own **Memory** (scoped to the project only, not shared with other projects), and its own **Scheduled tasks**.

**Note the real limitations** (from Anthropic's docs):
- Projects are Cowork-only (not in Claude Code).
- Projects created via "Use an existing folder" are desktop-only and local, tied to that folder — no cloud sync for that variant.
- Archiving a project removes it from the UI but does not delete local files.

If the user would rather skip a dedicated project and just run job-scan-setup as a plain Cowork task pointed at a folder, that's fine too -- a project isn't required, just a convenience for keeping things organized and for scoping memory.

---

## Phase 4: Choose the Workspace Folder, Consolidate Materials, and Test

The workspace is the actual folder on disk holding `persona.md`, `tracker.xlsx`, `materials/`, `logs/`, and `drafts/`. This is separate from any Cowork project -- a project can point at this folder as its context, but the files themselves live wherever you put them.

**Why this folder needs to hold everything**: a Cowork scheduled task (Phase 5) is given exactly one folder to run against. If the person's resumes and cover letters are scattered elsewhere -- Desktop, Downloads, an old project folder, email attachments -- the scheduled task can't see them, no matter how well the persona describes them. Everything the system needs has to physically live inside one folder.

1. **Ask the user to choose a location**, e.g.:
   - `~/Documents/JobSearch/` (macOS) or `C:\Users\<name>\Documents\JobSearch\` (Windows) -- recommended, easy to find and back up
   - An existing project/work folder they specify
   - Desktop -- works, but not recommended for anything long-lived

2. **Have the user create the folder** (Finder/Explorer, or ask Claude to do it if you're already in a Cowork task with file access to that location) with subfolders:
   ```
   [workspace-root]/
   ├── materials/   (all existing resumes and cover letters go here -- see step 3)
   ├── logs/
   └── drafts/
   ```

3. **Consolidate existing resumes and cover letters into `materials/`**. Ask the user directly: "Do you have resumes or cover letters saved anywhere on this computer right now -- Desktop, Downloads, a different folder, email attachments you've saved?" Have them copy or move every version they have (old and current) into `[workspace-root]/materials/`. This matters even for old/outdated resumes -- a resume that changed across 3 jobs tells a fuller trajectory story than the latest version alone, and this material is what the main SKILL.md's Step 2 persona interview and Step 7's auto-tailoring feature both draw from. Once consolidated, note the exact filenames (you'll reference them again during persona-building).

4. **Run the file-connection test** (this part doesn't depend on any Cowork-specific UI -- it's just file I/O against the chosen folder):
   - **Write test**: create `test_connectivity_[timestamp].txt` with a short marker string; confirm the user sees it appear in the folder.
   - **Read test**: read it back; confirm content matches exactly, no encoding issues.
   - **Format test**: create a minimal `.xlsx` (via the `xlsx` skill) with a couple of test cells; read it back and confirm it parses cleanly.
   - **Materials check**: confirm Claude can list and read at least one file from `materials/` (e.g., open one resume and confirm the text is legible, not confirm just that the file exists).
   - **Cleanup**: delete the two connectivity/format test files; confirm the folder is back to `materials/`, `logs/`, and `drafts/` (plus whatever the user put in materials/).
   - **Report**: tell the user plainly whether all steps passed. If anything failed, it's almost always a path typo or a permissions issue -- check that the folder exists and is writable (macOS: `chmod 755 <path>`; Windows: right-click -> Properties -> Security -> grant your user Full Control) and retry.

Once this passes, proceed to the main SKILL.md Step 1 onward (persona, tracker, boards, etc.), using this workspace path consistently everywhere a file needs to be saved, and pointing directly at `materials/` when reading resumes and cover letters for the persona interview.

---

## Phase 5: Create the Scheduled Task in Cowork

This is the piece that makes job-scan-setup actually recurring.

1. Click **"Scheduled"** in the left sidebar to land on the **Scheduled tasks** page. (If you created a Cowork project in Phase 3, you can also create a task scoped to that project -- the docs confirm each project has its own scheduled tasks.)
2. Click **"New task"** in the upper right. A dropdown shows two options:
   - **Create with Claude** -- pre-fills a prompt asking Claude to set the task up conversationally; Claude may ask you multiple-choice questions, then shows the name/cadence/description it's about to create for you to confirm by clicking "Schedule."
   - **Set up manually** -- opens the **Create scheduled task** form directly. This is the more predictable choice here, since job-scan needs a specific set of instructions and a specific folder to get right.
3. If using **Set up manually**, the form has these fields:
   - **Name** (required) -- e.g., "Job Scan"
   - **Instructions** (required) -- the actual task prompt, in a plain text box. Use something like:
     > Run my recurring job scan using the `job-scan` skill. Persona file: `[workspace-root]/persona.md`. Tracker file: `[workspace-root]/tracker.xlsx`. Materials (resumes/cover letters) are in `[workspace-root]/materials/`. Run autonomously; only pause if a strong match is genuinely ambiguous on fit.
   - **"Work in a project or folder"** (a dropdown-style control below the Instructions box) -- select the **workspace root** from Phase 4, not just the `materials/` subfolder. The task needs `persona.md`, `tracker.xlsx`, and `materials/` all reachable together, and this field only accepts one location, so it has to be the parent folder that contains all three.
   - **Default model** (same row as the folder control) -- leave as-is unless there's a specific reason to change it.
   - **Frequency** -- a dropdown defaulting to "Manual." Change it to whatever cadence was agreed in the main SKILL.md's optional-features step (e.g., weekly or twice-weekly).
   - **Permissions** -- a dropdown defaulting to "Manually approve." Keep this on manual approval for the first few runs while the persona is still being tuned; loosen it once the person trusts the scan's judgment.
   - **"Require this computer (Claude Desktop [Windows/macOS])"** -- a toggle, **off by default**. The app's own description underneath it is worth reading to the user directly: *"Only runs while your computer is awake. Gives Claude access to the folders you've allowed on this computer."* **Turn this ON.** Without it, the task has no path to the workspace folder at all -- this toggle is what actually grants local file access, not the folder selector above by itself.
4. Click **Save** (or confirm via "Schedule" if using Create with Claude).

#### Example Dialogue

> **You**: "Let's set up the recurring scan. Click Scheduled in the sidebar, then New task, then Set up manually."
>
> [User opens the form]
>
> **User**: "I see Name, Instructions, then a row with 'Work in a project or folder' and 'Default model', then Frequency and Permissions below that."
>
> **You**: "Good, that's the right form. For Name, put 'Job Scan'. For Instructions, paste this: [gives the prompt above]. Now click 'Work in a project or folder' and select your JobSearch folder -- the same one we tested earlier."
>
> [User selects it]
>
> **User**: "Done. What about Frequency and Permissions?"
>
> **You**: "Set Frequency to whatever cadence we agreed on -- twice a week, was it? And leave Permissions on 'Manually approve' for now, since the persona still needs tuning over the first few runs."
>
> **User**: "Okay. I also see a toggle at the bottom: 'Require this computer (Claude Desktop Windows)'. It's off."
>
> **You**: "Turn that on. That's what actually gives the task access to your JobSearch folder -- it says right underneath: 'Only runs while your computer is awake. Gives Claude access to the folders you've allowed on this computer.' Without it, the folder you picked above won't actually be reachable."
>
> [User toggles it on]
>
> **User**: "Toggled on. Saving now."
>
> **You**: "Perfect. That's the scheduled task set up."

**On the Scheduled tasks page itself**: new tasks appear with a "New" badge, and the page can be sorted (e.g., "Sort by Next run"). Because the "Require this computer" toggle is on, this task depends on Claude Desktop being installed and awake on this machine at the scheduled time -- it is not a fully remote/cloud-only task, unlike tasks that don't need local files.

**Related mechanism worth knowing about**: the "+" attach menu beside the composer also has a **Devices** entry showing **"Run tasks on"** followed by every machine registered to the account (observed: two entries, both "Claude Desktop (Windows)," each with a last-active time). This is a separate view of the same underlying device-registration system that "Require this computer" taps into for a specific task. If a scheduled task isn't reaching local files as expected, checking this Devices list (is the right machine listed, and does its last-active time look current?) is a useful diagnostic alongside the toggle itself.

**Managing the task afterward**: from the Scheduled tasks page you can search across tasks, sort by next run, and click into any task to view upcoming/past runs, edit the instructions or cadence, pause, resume, delete, or run on demand -- useful for testing the first run without waiting for the schedule.

**Important operational note carried over from the main skill**: keep `tracker.xlsx` closed in any spreadsheet app when a run is due -- an open file can block the write. This isn't a Cowork-specific quirk, just normal file-locking behavior on both platforms.

---

## Phase 6: First Run and Handoff

1. From the Scheduled tasks page, use **"Run now"** (or equivalent on-demand trigger) to fire the task immediately rather than waiting for the schedule, so the user gets a concrete result to react to right away.
2. Confirm the run completed and check the tracker/persona for the expected updates.
3. Remind the user: the first couple of runs are a tuning period. Corrections belong in the persona file's calibration log so they stick.
4. Confirm the user knows where to go to check on/pause/edit the task later: the **Scheduled** sidebar item.

---

## Troubleshooting

**"I don't see a Cowork tab at all"**
-> Confirm the account is on Pro, Max, Team, or Enterprise. Cowork isn't available on plans below that.

**"I see Cowork but no 'Scheduled' item in the sidebar, or no Projects"**
-> Update Claude Desktop to the latest version -- both features are explicitly gated on "the latest version of Claude Desktop" per Anthropic's docs.

**"job-scan-setup doesn't show up when I search Skills"**
-> It may not be in the public directory. Check with your Aeneas Bio admin whether it's distributed as an internal plugin (via Organization settings -> Plugins) or a manual install, rather than assuming it should be in the general marketplace.

**"I saved the scheduled task but forgot to check 'Require this computer'"**
-> Open the task from the Scheduled tasks page, find the toggle, and turn it on, then save again. Without it, the task has no access to local files regardless of what folder was selected in "Work in a project or folder" -- the folder selector and the toggle are two separate things that both need to be set correctly.

**"The scheduled task ran but couldn't read/write my files"**
-> First, check the **"Require this computer"** toggle is actually ON -- this is the most common cause, since it's off by default and easy to miss. Then confirm the "Work in a project or folder" selection still points at the exact workspace path, that the path still exists, and that no spreadsheet app has `tracker.xlsx` open. Also confirm Claude Desktop was open/awake on that machine at the scheduled time -- with the toggle on, the task only runs while this computer is available, not from Anthropic's cloud.

**"Task didn't fire when expected"**
-> Check the task isn't paused, check the Frequency setting, and -- if "Require this computer" is on -- check whether Claude Desktop was running and the machine was awake at that time.

---

## Summary Checklist

| Item | Status after this walkthrough |
|---|---|
| Claude Desktop | Installed, updated, Cowork tab visible |
| job-scan-setup skill | Confirmed available via "+" → Skills (or installed from "Browse skills" / org plugin) |
| Cowork project (optional) | Created, pointed at workspace folder |
| Workspace folder | Chosen, created, materials/ populated with all existing resumes and cover letters, file-connection test passed |
| Persona + tracker | Built per main SKILL.md Steps 1-7 |
| Scheduled task | Created in Cowork's Scheduled tab, folder set, cadence set |
| First run | Triggered manually via "Run now," verified |

**Next**: return to the main SKILL.md at Step 1 (Pre-flight: browser, authentication & account hygiene) if you haven't already built the persona and tracker, or proceed to normal recurring use if setup is complete.
