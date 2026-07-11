---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

Load plan, review critically, execute all tasks, report when complete.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Note:** Tell your human partner that Superpowers works much better with access to subagents. The quality of its work will be significantly higher if run on a platform with subagent support (Claude Code, Codex CLI, Codex App, and Copilot CLI all qualify; see the per-platform tool refs in `../using-superpowers/references/`). If subagents are available, use superpowers:subagent-driven-development instead of this skill.

## The Process

### Step 1: Load and Review Plan
1. Read plan file
2. Review critically - identify any questions or concerns about the plan
3. **Scan for parallelization** — are any upcoming tasks independent? If 2+ tasks have no dependencies between them, note them for parallel execution (Step 2b)
4. If concerns: Raise them with your human partner before starting
5. If no concerns: Create todos for the plan items and proceed

### Step 2: Execute Tasks

Execute tasks continuously — don't pause between tasks to ask "should I continue?"

For each task:
1. Mark as in_progress
2. Follow each step exactly (plan has bite-sized steps)
3. Run verifications as specified
4. **Post-write quality gate** (after code is written for each task):
   - Dispatch `feature-dev:code-reviewer` to review the changes
   - If reviewer finds issues: fix them before proceeding
5. Mark as completed

**At natural checkpoints** (a plan phase/milestone completes, or a coherent set of related tasks lands):
- Dispatch `code-simplifier:code-simplifier` on the accumulated changes
- Check progress against the plan; surface anything surprising to your human partner
- Then keep executing — checkpoints are for review, not for permission to continue

If the plan has no natural checkpoints, dispatch `code-simplifier:code-simplifier` once on the full diff before final verification (Step 3).

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

### Step 3: Verify and Complete

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
- Hit a blocker (missing dependency, test fails, instruction unclear)
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
- Execute continuously — review at natural checkpoints, don't stop to ask "should I continue?"
- Reference skills when plan says to
- Stop when blocked, don't guess
- Never start implementation on main/master branch without explicit user consent
- For UI tasks: invoke `frontend-design:frontend-design` skill before implementing

## Integration

**Required workflow skills:**
- **superpowers:using-git-worktrees** - Ensures isolated workspace (creates one or verifies existing)
- **superpowers:writing-plans** - Creates the plan this skill executes
- **superpowers:verification-before-completion** - REQUIRED: Verify before claiming done
- **superpowers:finishing-a-development-branch** - Complete development after all tasks

**Agents used during execution:**
- **feature-dev:code-reviewer** - After each task that writes code
- **code-simplifier:code-simplifier** - At natural checkpoints (or once before final verification), on reviewed code
- **general-purpose** - For parallel task execution (with `isolation: "worktree"`)
