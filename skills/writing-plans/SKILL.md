---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** This should be run in a dedicated worktree (created by brainstorming skill).

**Save plans to:** `docs/plans/YYYY-MM-DD-<feature-name>.md`

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## Research Phase: Plan → Architect Pipeline

Before writing the plan, gather strategic and structural intelligence using agents:

### Step 0a: Strategy (Plan agent)
Dispatch a `Plan` agent (subagent_type: `Plan`) with the design doc / requirements. It returns:
- Recommended implementation sequence and step ordering
- Key risks and trade-offs
- Critical architectural decisions that need resolution
- Dependencies between components

### Step 0b: Blueprint (code-architect agent)
Dispatch a `feature-dev:code-architect` agent with the Plan agent's strategy output. It returns:
- Specific files to create/modify (with paths)
- Existing patterns to follow (found by analyzing codebase)
- Component designs and data flow
- Build sequence matching the Plan agent's recommended order

**Run these sequentially** — the architect needs the Plan agent's strategy (approach, sequence, trade-offs) to produce a useful blueprint. If the design doc already locks in the approach completely, they can run in parallel, but this is rare.

Use the combined output (strategy + blueprint) to write the plan below. The Plan agent tells you WHAT order and WHY. The architect tells you WHICH files and HOW.

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## Task Structure

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

**Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

**Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

**Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

**Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## Remember
- Exact file paths always
- Complete code in plan (not "add validation")
- Exact commands with expected output
- Reference relevant skills with @ syntax
- DRY, YAGNI, TDD, frequent commits

## Plan Review Loop

After writing the complete plan:

1. Dispatch a single plan-document-reviewer subagent (see plan-document-reviewer-prompt.md) with precisely crafted review context — never your session history
   - Provide: path to the plan document, path to spec document
2. If Issues Found: fix the issues, re-dispatch reviewer for the whole plan
3. If Approved: proceed to execution handoff

**Review loop guidance:**
- Single-pass review — if approved, move on. Don't re-review "just in case"
- Only flag issues that would cause real problems during implementation
- Minor wording and stylistic preferences are not blocking issues

## Execution Handoff

After saving the plan, offer execution choice:

**"Plan complete and saved to `docs/plans/<filename>.md`. Three execution options:**

**1. Subagent-Driven (this session)** - I dispatch fresh subagent per task, review between tasks, fast iteration

**2. Parallel Session (separate)** - Open new session with executing-plans, batch execution with checkpoints

**3. Team Mode (this session, parallel)** - Spin up a coordinated team of agents. Best for plans with 3+ tasks that have interdependencies or need coordination (e.g., frontend + backend, or review loops)

**Which approach?"**

**If Subagent-Driven chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development
- Stay in this session
- Fresh subagent per task + code review

**If Parallel Session chosen:**
- Guide them to open new session in worktree
- **REQUIRED SUB-SKILL:** New session uses superpowers:executing-plans

**If Team Mode chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:dispatching-parallel-agents (Team Mode section)
- Set up team, create tasks from plan, spawn teammates, let agents coordinate
