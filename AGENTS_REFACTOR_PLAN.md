# Plan: Rewrite AGENTS.md to Standard Format

## Overview

Transform your current AGENTS.md from "documentation about agents" to "actual system prompt for agents" following https://agents.md standard.

## Detailed Plan with Small Reviewable Steps

---

## Step 1: Read Current AGENTS.md

**What**: Read entire AGENTS.md file

**Why**: Understand current structure and content to identify what to keep/change

**Lines to review**: ~800 lines (current file)

**Output**: Summary of current sections and what needs transformation

**Estimated time**: 5 minutes

**Files affected**: None (read-only)

---

## Step 2: Create New AGENTS.md Draft - Project Overview & Setup

**What**: Write project overview, tech stack, and setup commands sections

**Why**: Establishes foundation of system prompt

**Content**:

- Project Overview (2-3 paragraphs)
- Tech Stack (bullet list)
- Setup Commands (code block with bash commands)

**Estimated time**: 3 minutes

**Files created**: AGENTS.md (new file)

**Lines added**: ~30-40 lines

**Reviewable**: Yes, can see all new content

---

## Step 3: Add Development Workflow & Code Style Sections

**What**: Write workflow instructions and code style rules

**Why**: Guides agent on HOW to work, not just WHAT to do

**Content**:

- Development Workflow (5-6 steps)
- Code Style Rules (5-7 specific rules)
- Architecture Principles (4-6 principles)

**Estimated time**: 4 minutes

**Files modified**: AGENTS.md (append)

**Lines added**: ~40-50 lines

**Reviewable**: Yes, flat list format

---

## Step 4: Add Critical Rules & Git Workflow

**What**: Write the most important rules and git conventions

**Why**: These are non-negotiable constraints that must never be violated

**Content**:

- Critical Rules (5-6 rules, starting with Human-in-the-Loop)
- Git Workflow (commit format, quality checks, no push rule)
- When to Stop and Ask User (4-5 scenarios)

**Estimated time**: 4 minutes

**Files modified**: AGENTS.md (append)

**Lines added**: ~40-50 lines

**Reviewable**: Yes, clear do/don't format

---

## Step 5: Add Project Structure & Key Documentation

**What**: Write folder structure and important file references

**Why**: Help agent understand project organization

**Content**:

- Project Structure (tree diagram)
- Key Documentation (4-5 files)
- Safety Rules (3-4 rules)

**Estimated time**: 3 minutes

**Files modified**: AGENTS.md (append)

**Lines added**: ~30-40 lines

**Reviewable**: Yes, visual tree + list

---

## Step 6: Review and Refine - Remove Unnecessary Content

**What**: Review new AGENTS.md and remove anything non-essential

**Why**: Keep file concise (target: 150-200 lines max per standard)

**Content to remove**:

- Meta-discussion about what agents are
- Research citations and links
- Explaining philosophy
- References to external methodologies

**Estimated time**: 5 minutes

**Files modified**: AGENTS.md (edit)

**Lines removed**: ~100+ lines

**Result**: Clean, concise system prompt (~150-200 lines)

**Reviewable**: Yes, clear diff showing what was removed

---

## Step 7: Final Verification

**What**: Verify that new AGENTS.md follows standards

**Why**: Ensure quality before replacing original file

**Checks**:

- [ ] File is 150-200 lines (not 800+)
- [ ] Contains direct instructions (not meta-discussion)
- [ ] All sections are flat hierarchy (no deep nesting)
- [ ] Compatible with AGENTS.md standard format
- [ ] Includes Human-in-the-Loop rule prominently
- [ ] Lists when to ask user for approval

**Estimated time**: 2 minutes

**Files affected**: None (verification only)

**Output**: Pass/fail checklist with explanation

**Reviewable**: Yes, clear checklist format

---

## Step 8: Replace Original AGENTS.md

**What**: Backup original file and replace with new version

**Why**: Implement improved system prompt

**Actions**:

- Backup: `cp AGENTS.md AGENTS.md.backup`
- Replace: Write new content to AGENTS.md
- Delete backup: `rm AGENTS.md.backup`

**Estimated time**: 1 minute

**Files modified**: AGENTS.md (complete replacement)

**Lines**: ~150-200 (final)

**Reviewable**: Yes, can compare backup if needed

---

## Total Summary

| Step                | Time       | Lines            | Reviewable? | Files |
| ------------------- | ---------- | ---------------- | ----------- | ----- |
| 1. Read current     | 5 min      | -                | 0           |
| 2. Overview & Setup | 3 min      | ✅ Yes           | 1 new       |
| 3. Workflow & Style | 4 min      | ✅ Yes           | 1 mod       |
| 4. Rules & Git      | 4 min      | ✅ Yes           | 1 mod       |
| 5. Structure & Docs | 3 min      | ✅ Yes           | 1 mod       |
| 6. Refine (remove)  | 5 min      | ✅ Yes           | 1 mod       |
| 7. Verify           | 2 min      | ✅ Yes           | 0           |
| 8. Replace          | 1 min      | ✅ Yes           | 1 mod       |
| **Total**           | **27 min** | **7 of 8 steps** | **1 file**  |

---

## Ready to Proceed?

Each step is:

- ✅ Small (<150 lines of output, <5 minutes review)
- ✅ Independent (can stop/continue after each)
- ✅ Reviewable (you can see exactly what changes)
- ✅ Safe (backup original file before replacement)

**Shall I start with Step 1: Read Current AGENTS.md?**
