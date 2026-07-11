---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** If working in an isolated worktree, it should have been created via the `superpowers:using-git-worktrees` skill at execution time.

**Save plans to:** `docs/plans/YYYY-MM-DD-<feature-name>.md`
- (User preferences for plan location override this default)

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

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

## Task Right-Sizing

A task is the smallest unit that carries its own test cycle and is worth a
fresh reviewer's gate. When drawing task boundaries: fold setup,
configuration, scaffolding, and documentation steps into the task whose
deliverable needs them; split only where a reviewer could meaningfully
reject one task while approving its neighbor. Each task ends with an
independently testable deliverable.

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

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

## Global Constraints

[The spec's project-wide requirements — version floors, dependency limits,
naming and copy rules, platform requirements — one line each, with exact
values copied verbatim from the spec. Every task's requirements implicitly
include this section.]

---
```

## Task Structure

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Interfaces:**
- Consumes: [what this task uses from earlier tasks — exact signatures]
- Produces: [what later tasks rely on — exact function names, parameter
  and return types. A task's implementer sees only their own task; this
  block is how they learn the names and types neighboring tasks use.]

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

## No Placeholders

Every step must contain the actual content an engineer needs. These are **plan failures** — never write them:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" (without actual test code)
- "Similar to Task N" (repeat the code — the engineer may be reading tasks out of order)
- Steps that describe what to do without showing how (code blocks required for code steps)
- References to types, functions, or methods not defined in any task

## Remember
- Exact file paths always
- Complete code in every step — if a step changes code, show the code
- Exact commands with expected output
- Reference relevant skills with @ syntax
- DRY, YAGNI, TDD, frequent commits

## Self-Review

After writing the complete plan, look at the spec with fresh eyes and check the plan against it. This is a checklist you run yourself — not a subagent dispatch.

**1. Spec coverage:** Skim each section/requirement in the spec. Can you point to a task that implements it? List any gaps.

**2. Placeholder scan:** Search your plan for red flags — any of the patterns from the "No Placeholders" section above. Fix them.

**3. Type consistency:** Do the types, method signatures, and property names you used in later tasks match what you defined in earlier tasks? A function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug.

If you find issues, fix them inline. No need to re-review — just fix and move on. If you find a spec requirement with no task, add the task.

## Execution Handoff

After saving the plan, **choose the best execution approach based on the plan's structure**, then present it for approval.

### Decision Logic

```dot
digraph execution_choice {
    "Plan saved" [shape=doublecircle];
    "3+ tasks with interdependencies\nor cross-cutting coordination?" [shape=diamond];
    "Tasks mostly independent?" [shape=diamond];
    "Team Mode" [shape=box];
    "Subagent-Driven" [shape=box];
    "Subagent-Driven (default)" [shape=box];

    "Plan saved" -> "3+ tasks with interdependencies\nor cross-cutting coordination?";
    "3+ tasks with interdependencies\nor cross-cutting coordination?" -> "Team Mode" [label="yes"];
    "3+ tasks with interdependencies\nor cross-cutting coordination?" -> "Tasks mostly independent?" [label="no"];
    "Tasks mostly independent?" -> "Subagent-Driven" [label="yes"];
    "Tasks mostly independent?" -> "Subagent-Driven (default)" [label="sequential"];
}
```

| Signal | Approach |
|--------|----------|
| 3+ tasks with interdependencies needing coordination (e.g., frontend + backend, review loops) | **Team Mode** |
| Tasks are independent or mostly sequential | **Subagent-Driven** |

Subagent-Driven is the default — it covers most plans well (sequential or independent tasks, review between each).

### Present Recommendation

**"Plan complete and saved to `docs/plans/<filename>.md`. I recommend **[chosen approach]** because [1-sentence reason based on plan structure]. OK to proceed?"**

### Execution by Approach

**Subagent-Driven:**
- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development
- Stay in this session
- Fresh subagent per task + code review

**Team Mode:**
- **REQUIRED SUB-SKILL:** Use superpowers:dispatching-parallel-agents (Team Mode section)
- Set up team, create tasks from plan, spawn teammates, let agents coordinate
