---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

Load plan, review critically, execute tasks in batches, report for review between batches.

**Core principle:** Batch execution with checkpoints for architect review.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

## The Process

### Step 1: Load and Review Plan
1. Read plan file
2. Review critically - identify any questions or concerns about the plan
3. **Scan for parallelization** — are any upcoming tasks independent? If 2+ tasks have no dependencies between them, note them for parallel execution (Step 2b)
4. If concerns: Raise them with your human partner before starting
5. If no concerns: Create TodoWrite and proceed

### Step 2: Execute Batch
**Default: First 3 tasks**

For each task:
1. Mark as in_progress
2. Follow each step exactly (plan has bite-sized steps)
3. Run verifications as specified
4. **Post-write quality gate** (after code is written for each task):
   - Dispatch `feature-dev:code-reviewer` to review the changes
   - If reviewer finds issues: fix them before proceeding
   - If task is the last in a batch: also dispatch `code-simplifier:code-simplifier` on the batch's changes
5. Mark as completed

### Step 2b: Parallel Execution (when applicable)
When 2+ tasks are independent (no shared files, no dependency between them):

**Simple parallel dispatch** (independent, no coordination needed):
- Dispatch one `general-purpose` agent per task via the Agent tool
- All run concurrently in isolated worktrees (`isolation: "worktree"`)
- Review and integrate results when all complete

**Team mode** (interdependent, agents need to coordinate):
- Use `TeamCreate` to spin up a team
- Assign tasks to teammates who can message each other
- Useful when tasks share contracts (e.g., frontend + backend agreeing on API shape)
- See `using-superpowers` skill for team composition examples

### Step 3: Report
When batch complete:
- Show what was implemented
- Show verification output
- Show code-reviewer results (issues found and resolved)
- Say: "Ready for feedback."

### Step 4: Continue
Based on feedback:
- Apply changes if needed
- Execute next batch
- Repeat until complete

### Step 5: Verify and Complete

After all tasks complete:

**REQUIRED: Run verification before any completion claim.**
- **REQUIRED SUB-SKILL:** Use superpowers:verification-before-completion
- Run ALL verification commands (tests, build, lint)
- Read full output, confirm zero failures
- Only after evidence confirms success: proceed to completion

Then:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use superpowers:finishing-a-development-branch
- Follow that skill to verify tests, present options, execute choice

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker mid-batch (missing dependency, test fails, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- Verification fails repeatedly

**Ask for clarification rather than guessing.**

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- Partner updates the plan based on your feedback
- Fundamental approach needs rethinking

**Don't force through blockers** - stop and ask.

## Remember
- Review plan critically first
- Follow plan steps exactly
- Don't skip verifications
- **Dispatch code-reviewer after every task that writes code** — not optional
- **Look for parallelization opportunities** — independent tasks should run concurrently
- **Never claim completion without running verification** — evidence before assertions
- Reference skills when plan says to
- Between batches: just report and wait
- Stop when blocked, don't guess
- Never start implementation on main/master branch without explicit user consent
- For UI tasks: invoke `frontend-design:frontend-design` skill before implementing

## Integration

**Required workflow skills:**
- **superpowers:using-git-worktrees** - REQUIRED: Set up isolated workspace before starting
- **superpowers:writing-plans** - Creates the plan this skill executes
- **superpowers:verification-before-completion** - REQUIRED: Verify before claiming done
- **superpowers:finishing-a-development-branch** - Complete development after all tasks

**Agents used during execution:**
- **feature-dev:code-reviewer** - After each task that writes code
- **code-simplifier:code-simplifier** - After each batch, on reviewed code
- **general-purpose** - For parallel task execution (with `isolation: "worktree"`)
