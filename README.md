# job-scan-toolkit

A personalized, autonomous job-search scanning system built as Claude skills.

## What is job-scan-toolkit?

job-scan-toolkit automates your job search by continuously scanning job boards for roles that match your unique profile, goals, and preferences. Instead of manually hunting through job listings, you define your target once, and the system does recurring scans, filters candidates intelligently against your personal rubric, and surfaces only the roles worth your time.

It's built as a set of three Claude skills that work together: one for setup, one for recurring scans, and one for tuning your preferences as your search evolves.

## Core Features

- **Personalized scanning** - Define your target seniority, problem domain, location scope, and disqualifiers once. The system remembers and uses them.
- **Multi-board coverage** - Scans your chosen job boards (LinkedIn, Indeed, AngelList, Blind, etc.) in a single run.
- **Intelligent filtering** - Scores each posting against your personal rubric and your stated goals, not just keyword matching.
- **Deduplication** - Tracks what you've already seen and skips repeats across scan runs.
- **Tailored documents** - Optionally auto-drafts CV and cover letter variants for standout matches based on each role.
- **Flexible storage** - Works with Notion, Jira+Confluence, GitHub, Google Drive, or local files for your job tracker.
- **On-demand calibration** - Refine your preferences mid-search through guided conversations grounded in real tracker data, not hand-editing.

## Core Outputs

Each scan run produces:

- **Summary briefing** - High-level report of what was scanned, how many new qualified leads were found, and highlights of standout matches.
- **Updated job tracker** - New leads appended, stale postings marked dead, your running record stays current.
- **Optional application drafts** - For top-fit roles, ready-to-edit CV and cover letter tailored to that position.
- **Engagement log** - Records which roles you've actioned, skipped, or marked as interested for future reference.

## Workflow Logic

### 1. Setup (one-time)
Run `job-scan-setup`. You'll be guided through:
- Your target job titles and seniority level
- The problems or domains you want to solve
- Disqualifiers (roles or companies you want to avoid)
- Which job boards to scan
- Location preferences
- Where to store your tracker (Notion, GitHub, Google Drive, etc.)
- Calibration thresholds and scoring weights

The skill creates your persona file, initializes your job tracker, and schedules the recurring scan task.

### 2. Recurring Scans (automated or on-demand)
Run `job-scan` on a cadence (weekly, bi-weekly, or ad-hoc). Each run:
- Reads your persona to understand what you're looking for
- Sweeps the tracker for postings that have gone stale (removed from boards, past deadline)
- Scans your configured job boards for new listings
- Scores each posting against your rubric
- Dedupes against your tracker (skips roles you've already seen)
- Appends qualified new leads to your tracker
- Optionally drafts tailored application documents for top matches
- Delivers a summary of what was found

### 3. Calibration (as needed)
Run `persona-review` anytime to refine your preferences. Instead of hand-editing your persona file or waiting for mid-run corrections, you get a guided conversation grounded in real evidence from your tracker. Update your target seniority, problems of interest, disqualifiers, scoring weights, or which boards to scan.

## Quick Setup Guide

See **[SETUP_GUIDE.md](SETUP_GUIDE.md)** for step-by-step instructions on installing and running job-scan-toolkit in your Claude environment.

## In-Depth Documentation

For comprehensive details on architecture, storage backends, persona tuning, and advanced usage, see **[MANUAL.md](MANUAL.md)**.

## Status

- Storage backend abstraction (`PersonaStore` / `JobStore`) is designed and wired into both `job-scan-setup` and `job-scan`.
- Persona-review skill is built and integrated into the setup and scan workflows.
- All three skills are production-ready.
