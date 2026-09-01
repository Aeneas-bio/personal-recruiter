# GitHub Workflow: Update Docs and Merge to Main

Follow these steps in PowerShell to update the three documentation files and merge to main.

## Prerequisites

- Git is installed and configured
- You have write access to https://github.com/Aeneas-bio/personal-recruiter
- You have the three new files ready: README.md, SETUP_GUIDE.md, MANUAL.md

## Step-by-Step

### Step 1: Navigate to repo and create a feature branch

```powershell
# Open PowerShell and navigate to your personal-recruiter repo
cd path\to\your\personal-recruiter

# Make sure you're on main and up to date
git status
git pull origin main

# Create a feature branch for this docs update
git checkout -b docs/restructure-readme
```

**What this does**: Creates a separate branch so your changes don't affect main until you're ready.

---

### Step 2: Update the three documentation files

You have two options for each file:

**Option A: Manual copy/paste (easiest if files are short)**

1. Open `README.md` in your repo in a text editor
2. Delete all content
3. Copy/paste the new README.md content
4. Save and close
5. Repeat for `SETUP_GUIDE.md` and `MANUAL.md`

**Option B: Copy files via PowerShell (if you have them saved locally)**

```powershell
# Copy each file into your repo (replace paths with your actual paths)
Copy-Item -Path "C:\path\to\new\README.md" -Destination ".\README.md" -Force
Copy-Item -Path "C:\path\to\new\SETUP_GUIDE.md" -Destination ".\SETUP_GUIDE.md" -Force
Copy-Item -Path "C:\path\to\new\MANUAL.md" -Destination ".\MANUAL.md" -Force
```

**What this does**: Replaces the old docs with the new versions.

---

### Step 3: Stage and commit the changes

```powershell
# See what files have changed
git status

# Stage all three files
git add README.md SETUP_GUIDE.md MANUAL.md

# Double-check they're staged
git status

# Commit with a descriptive message
git commit -m "docs: restructure readme into modular guides

- README.md: Streamlined landing page with what/why/features/workflow
- SETUP_GUIDE.md: Quick start for installation and first scan
- MANUAL.md: Comprehensive reference for architecture, backends, tuning

This addresses readability by separating quick-start from deep-dive content."
```

**What this does**: Saves your changes locally with a clear message.

---

### Step 4: Push to GitHub

```powershell
# Push your feature branch to GitHub
git push origin docs/restructure-readme
```

**What this does**: Uploads your branch to GitHub so you can create a PR.

---

### Step 5: Create a Pull Request

Go to **https://github.com/Aeneas-bio/personal-recruiter**

You should see a banner saying "docs/restructure-readme had recent pushes". Click **"Compare & pull request"**

Or manually:
1. Click **Pull requests** tab
2. Click **New pull request**
3. Set base: `main` | compare: `docs/restructure-readme`
4. Click **Create pull request**

Fill in:
- **Title**: `docs: Restructure README into modular guides`
- **Description**: Paste your commit message or add more context
- **Reviewers**: Assign someone to review (if needed)

Click **Create pull request**.

**What this does**: Asks your team to review the changes before merging.

---

### Step 6: Merge the PR

Once your PR is approved (or if you're confident):

**Option A: Merge via GitHub UI (recommended)**

1. Go back to your PR on GitHub
2. Click **Merge pull request**
3. Select merge strategy (default is fine)
4. Click **Confirm merge**
5. GitHub shows "Pull request successfully merged"
6. Click **Delete branch** to clean up

**Option B: Merge via PowerShell**

```powershell
# Switch to main locally
git checkout main

# Pull latest changes
git pull origin main

# Merge your feature branch
git merge docs/restructure-readme

# Verify the merge
git log --oneline -5

# Push main to GitHub
git push origin main

# Clean up the feature branch
git branch -d docs/restructure-readme
```

**What this does**: Combines your changes into the main branch.

---

### Step 7: Verify success

```powershell
# Confirm you're on main
git status

# Peek at the new files to confirm they're there
Get-Content .\README.md -TotalCount 20
Get-Content .\SETUP_GUIDE.md -TotalCount 20
Get-Content .\MANUAL.md -TotalCount 20
```

Done! Your docs are now live on main.

---

## Quick Command Summary

If you're comfortable with git, here's the abbreviated version:

```powershell
cd path\to\personal-recruiter
git pull origin main
git checkout -b docs/restructure-readme

# Update README.md, SETUP_GUIDE.md, MANUAL.md here

git add README.md SETUP_GUIDE.md MANUAL.md
git commit -m "docs: restructure readme into modular guides"
git push origin docs/restructure-readme

# Go to GitHub, create PR, merge when approved
```

---

## Troubleshooting

**Q: I made a typo in a file. How do I fix it?**

A: Edit the file locally, then:
```powershell
git add <filename>
git commit --amend
git push origin docs/restructure-readme -f
```

**Q: I want to delete my feature branch and start over.**

A:
```powershell
git checkout main
git branch -D docs/restructure-readme
git checkout -b docs/restructure-readme
```

**Q: The merge has conflicts.**

A: This happens if main changed while you were working. Run:
```powershell
git merge main
# Fix conflicts in your editor
git add .
git commit -m "Resolve merge conflicts"
git push origin docs/restructure-readme
```

**Q: I accidentally committed to main instead of a branch.**

A: Don't panic. Run:
```powershell
git reset --soft HEAD~1  # Undo the commit but keep changes
git checkout -b docs/restructure-readme
git commit -m "docs: ..."
git push origin docs/restructure-readme
```

---

## Next Steps

- Celebrate your cleaner docs!
- Share the SETUP_GUIDE.md with new users so they can get started quickly
- Monitor for feedback and use MANUAL.md for deeper questions
