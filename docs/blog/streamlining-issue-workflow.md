# Streamlining Your Issue Workflow with Skills and Dossiers

You pick up a new issue. Before writing a single line of code, you're:

- Tab-switching to GitHub to grab the issue title
- Deciding if it's `bug/` or `feature/` (what were the labels again?)
- Mentally slugifying "Add user preferences page" → "add-user-preferences-page"
- Remembering where worktrees go in this repo (`../`? `.worktrees/`? next to `main/`?)
- Running `git branch`, `git worktree add`
- Creating yet another `PLANNING.md` from scratch
- Trying to remember the team's PR conventions

This takes 5-10 minutes. You do it multiple times a day. And everyone on your team does it slightly differently.

**What if you could just say: "start working on issue 123"?**

---

## The Solution: Skills + Dossiers

Two tools that work together:

| Component | What it does | Where it lives |
|-----------|--------------|----------------|
| **Skill** | Natural language trigger | Local (`.claude/skills/`) |
| **Dossier** | Versioned workflow engine | Registry (shared) |

- **Skills** teach Claude Code to respond to phrases like "start issue" or "work on issue #123"
- **Dossiers** contain the actual workflow logic—versioned, signed, and shareable

Combined: You say "start working on issue 123" → Claude triggers the skill → skill runs the dossier → everything happens automatically.

---

## The Skill File

A Claude Code skill is a markdown file that teaches Claude to respond to natural language triggers.

### What is a Skill?

- Lives in `.claude/skills/<skill-name>/SKILL.md`
- Auto-discovered by Claude Code
- The `description` field tells Claude when to trigger
- The body contains instructions for what to do

### The Skill Content

Create `.claude/skills/start-issue/SKILL.md`:

```yaml
---
name: start-issue
description: Set up a GitHub issue for development with branch, worktree, and planning doc.
  Use when user says "start issue", "work on issue #X", "set up issue", or "begin issue".
---

# Start Issue Workflow

When the user wants to start working on a GitHub issue:

## Prerequisites

Ensure dossier-tools is installed:
```bash
pip install dossier-tools
```

If not installed, help the user install it first.

## Steps

1. Extract the issue number from their request
2. Run the setup workflow:
   ```bash
   dossier run imboard-ai/development/git/setup-issue-workflow
   ```
3. When prompted for issue number, provide the extracted number
4. Confirm successful setup with the user

## What This Creates

- A properly named branch (`feature/123-title` or `bug/123-title`)
- A git worktree for isolated development
- A `PLANNING.md` file to track implementation
```

---

## The Dossier File

The skill triggers a dossier from the registry. You can inspect it first:

```bash
# View metadata
dossier get imboard-ai/development/git/setup-issue-workflow

# Download locally
dossier pull imboard-ai/development/git/setup-issue-workflow
```

This is an example dossier—you can create and modify your own for your team's conventions.

<details>
<summary><strong>Full Dossier Content (click to expand)</strong></summary>

```yaml
---
authors:
- name: Yuval Dimnik <yuval.dimnik@gmail.com>
checksum:
  algorithm: sha256
  hash: 1d499eb35feb239776ef52f791f77380703e59718993aa2eef88bc426cc41ccc
name: setup-issue-workflow
objective: Create a workflow for GitHub issues that fetches issue details, creates
  appropriately named branches, sets up git worktrees, and generates PLANNING.md files
  for structured development
schema_version: 1.0.0
signature:
  algorithm: ed25519
  public_key: rwZMHabZOn44qGc9tIRVPjFsHpoB3KxbsLhoULI5Xrw=
  signature: r06cktM+V2EDSB2EqYB9l18D7dmSt+WtUr2coeq1/Q9qntiQJ+9m3hntY8r5mK1jNOquR1trcfhRqGMiYbZeBQ==
  signed_by: Yuval Dimnik <yuval.dimnik@gmail.com>
  timestamp: '2025-12-07T06:36:07.194861+00:00'
status: draft
title: Setup Issue Workflow
version: 1.0.0
---

# Setup Issue Workflow

## Objective

Create a workflow for GitHub issues that fetches issue details, creates appropriately named branches, sets up git worktrees, and generates PLANNING.md files for structured development.

## Prerequisites

- [ ] Git is installed and configured
- [ ] GitHub CLI (gh) is installed and authenticated
- [ ] You are in a git repository with GitHub as a remote
- [ ] You have push access to create branches

## Context to Gather

### 1. Issue Information

Collect the GitHub issue number from user input. Then fetch:
- Issue title
- Issue labels (especially "bug" or "feature")
- Issue body/description
- Issue assignees (optional)

### 2. Worktree Location Discovery

Discover where to create the worktree using this priority order:

1. **Check documentation first** (avoid trial-and-error):
   - Look in `WORKTREES.md`, `.claude.md`, `agents.md`, `README.md`, `CONTRIBUTING.md` for worktree instructions
   - These files should have explicit guidance on worktree location patterns
   - **Critical**: Some repos use nested main worktree (e.g., `repo/main/`) where worktrees should be siblings to `main/`, not to `repo/`

2. **Check for sibling worktrees** (most common pattern):
   ```bash
   git worktree list
   ```

3. **Check for nested worktrees directory**:
   - Look for `../worktrees/` or `.worktrees/` patterns in existing worktrees
   - If found, use that directory

4. **If unclear**: Prompt user for location

## Actions to Perform

### Step 1: Get Issue Number

Prompt the user for the GitHub issue number if not already provided.

### Step 2: Fetch Issue Details

Use the GitHub CLI to fetch issue information:

```bash
gh issue view <ISSUE_NUMBER> --json title,labels,body,assignees
```

### Step 3: Determine Branch Type

Based on issue labels:

1. **If "bug" label present**: Use `bug/` prefix
2. **If "feature" label present**: Use `feature/` prefix
3. **If both or neither**: Prompt user to choose

### Step 4: Create Branch Name

Generate the branch name:

1. **Slugify the title**:
   - Convert to lowercase
   - Replace spaces with hyphens
   - Remove special characters
   - Truncate to reasonable length (50 chars max)

2. **Construct branch name**:
   ```
   {type}/{issue-number}-{slugified-title}
   ```

   Examples:
   - `bug/123-fix-login-redirect-issue`
   - `feature/456-add-user-dashboard`

### Step 5: Discover Worktree Location

Follow the discovery process from "Context to Gather" section.

### Step 6: Create Git Branch

```bash
git branch <branch-name>
```

### Step 7: Create Git Worktree

```bash
git worktree add <worktree-path> <branch-name>
```

### Step 8: Generate PLANNING.md

Create the PLANNING.md file in the worktree root with this structure:

```markdown
# Issue #<NUMBER>: <TITLE>

## Type
<bug|feature>

## Problem Statement
<Issue body content here>

## Implementation Checklist
- [ ] Understand the issue and gather context
- [ ] Identify files that need modification
- [ ] Implement the changes
- [ ] Add/update tests
- [ ] Update documentation if needed
- [ ] Self-review the changes
- [ ] Create pull request

## Files to Modify
<!-- List the files you expect to modify -->

## Testing Strategy
<!-- Describe how you will test the changes -->

## Notes
<!-- Additional notes, questions, or considerations -->

## Related Issues/PRs
- Issue: #<NUMBER>
```

### Step 9: Display Results

Show a summary of what was created:

```
Issue workflow setup complete!

Issue:      #<NUMBER> - <TITLE>
Type:       <bug|feature>
Branch:     <branch-name>
Worktree:   <worktree-path>
Planning:   <worktree-path>/PLANNING.md

Next steps:
1. Navigate to the worktree: cd <worktree-path>
2. Review and update PLANNING.md
3. Start working on the issue!
4. When done, create a PR: gh pr create
```

## Validation

- [ ] Issue details were successfully fetched from GitHub
- [ ] Branch name follows the convention: `{type}/{number}-{slug}`
- [ ] Git branch was created successfully
- [ ] Git worktree was created at the correct location
- [ ] PLANNING.md was created in the worktree root
- [ ] PLANNING.md contains all required sections
- [ ] User was shown clear next steps
```

</details>

---

## Implementation Steps

### Prerequisites

- [Claude Code](https://claude.ai/code) CLI installed
- [GitHub CLI](https://cli.github.com/) (`gh`) installed and authenticated

### Step 1: Install dossier-tools

```bash
pip install dossier-tools
```

Or with uv:

```bash
uv add dossier-tools
```

GitHub: [github.com/liberioai/dossier-tools](https://github.com/liberioai/dossier-tools)

### Step 2: Download the Skill File

```bash
mkdir -p .claude/skills/start-issue
curl -o .claude/skills/start-issue/SKILL.md \
  https://raw.githubusercontent.com/liberioai/dossier-tools/main/examples/skills/start-issue/SKILL.md
```

Or copy it manually from the repo: [examples/skills/start-issue/SKILL.md](https://github.com/liberioai/dossier-tools/tree/main/examples/skills/start-issue/SKILL.md)

### Step 3: (Optional) Pull the Dossier Locally

```bash
dossier pull imboard-ai/development/git/setup-issue-workflow
```

### Step 4: Test It

In Claude Code, say:

> "start working on issue 123"

---

## Testing & Validation

Verify the workflow works:

- [ ] Skill triggers when you say "start issue", "work on issue #X", etc.
- [ ] Branch is created with correct naming (`feature/123-title` or `bug/123-title`)
- [ ] Worktree is created in the right location
- [ ] `PLANNING.md` is generated with issue details
- [ ] Summary shows all created resources

---

## Summary

| Before | After |
|--------|-------|
| 5-10 minutes of manual setup | One natural language command |
| Inconsistent naming across team | Same conventions every time |
| Context switching to GitHub | Automated fetching |
| Creating PLANNING.md from scratch | Generated with issue details |

### What You Get

- **Time saved**: Manual multi-step process → one command
- **Consistency**: Same workflow, same naming, every time
- **Shareability**: Publish your own dossiers for your team

### Next Steps

- Browse more dossiers: `dossier list`
- Create your own: `dossier new`
- Learn more: [What is a Dossier?](../what-is-a-dossier.md)

---

## Links

- [dossier-tools on GitHub](https://github.com/liberioai/dossier-tools)
- [Claude Code](https://claude.ai/code)
- [GitHub CLI](https://cli.github.com/)
