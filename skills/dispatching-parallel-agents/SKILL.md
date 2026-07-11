---
name: dispatching-parallel-agents
description: Use when facing 2+ tasks that can benefit from concurrent execution — includes parallel dispatch, subagent relay, and team mode
---

# Dispatching Parallel Agents

## Overview

You delegate tasks to specialized agents with isolated context. By precisely crafting their instructions and context, you ensure they stay focused and succeed at their task. They should never inherit your session's context or history — you construct exactly what they need. This also preserves your own context for coordination work.

When you have multiple tasks, running them sequentially wastes time. Choose the right parallelization strategy based on how much coordination the tasks need.

**Core principle:** Match the coordination mechanism to the task dependencies.

## Three Tiers of Parallelization

| Tier | Mechanism | Coordination | Best for |
|------|-----------|-------------|----------|
| **Parallel dispatch** | Multiple Agent calls | None — fire and wait | Independent tasks, no shared state |
| **Subagent relay** | Fresh agent per task, you relay | You pass context between | Light dependencies, assembly-line |
| **Team mode** | TeamCreate + persistent agents | Agents message each other directly | Interdependent work, review loops, shared contracts |

### Decision heuristic:
```
Are tasks independent?
  Yes → Parallel dispatch (simple, fast)
  No → Do agents need to talk to EACH OTHER?
    No  → Subagent relay (you pass context)
    Yes → Team mode (agents coordinate directly)
```

## When to Use

```dot
digraph when_to_use {
    "Multiple failures?" [shape=diamond];
    "Are they independent?" [shape=diamond];
    "Single agent investigates all" [shape=box];
    "One agent per problem domain" [shape=box];
    "Can they work in parallel?" [shape=diamond];
    "Sequential agents" [shape=box];
    "Parallel dispatch" [shape=box];

    "Multiple failures?" -> "Are they independent?" [label="yes"];
    "Are they independent?" -> "Single agent investigates all" [label="no - related"];
    "Are they independent?" -> "Can they work in parallel?" [label="yes"];
    "Can they work in parallel?" -> "Parallel dispatch" [label="yes"];
    "Can they work in parallel?" -> "Sequential agents" [label="no - shared state"];
}
```

**Use when:**
- 3+ test files failing with different root causes
- Multiple subsystems broken independently
- Each problem can be understood without context from others
- No shared state between investigations

**Don't use when:**
- Failures are related (fix one might fix others)
- Need to understand full system state
- Agents would interfere with each other

## The Pattern

### 1. Identify Independent Domains

Group failures by what's broken:
- File A tests: Tool approval flow
- File B tests: Batch completion behavior
- File C tests: Abort functionality

Each domain is independent - fixing tool approval doesn't affect abort tests.

### 2. Create Focused Agent Tasks

Each agent gets:
- **Specific scope:** One test file or subsystem
- **Clear goal:** Make these tests pass
- **Constraints:** Don't change other code
- **Expected output:** Summary of what you found and fixed

### 3. Dispatch in Parallel

Issue all three subagent dispatches in the same response — they run in parallel:

```text
Subagent (general-purpose): "Fix agent-tool-abort.test.ts failures"
Subagent (general-purpose): "Fix batch-completion-behavior.test.ts failures"
Subagent (general-purpose): "Fix tool-approval-race-conditions.test.ts failures"
# All three run concurrently.
```

Multiple dispatch calls in one response = parallel execution. One per response = sequential.

### 4. Review and Integrate

When agents return:
- Read each summary
- Verify fixes don't conflict
- Run full test suite
- Integrate all changes

## Agent Prompt Structure

Good agent prompts are:
1. **Focused** - One clear problem domain
2. **Self-contained** - All context needed to understand the problem
3. **Specific about output** - What should the agent return?

```markdown
Fix the 3 failing tests in src/agents/agent-tool-abort.test.ts:

1. "should abort tool with partial output capture" - expects 'interrupted at' in message
2. "should handle mixed completed and aborted tools" - fast tool aborted instead of completed
3. "should properly track pendingToolCount" - expects 3 results but gets 0

These are timing/race condition issues. Your task:

1. Read the test file and understand what each test verifies
2. Identify root cause - timing issues or actual bugs?
3. Fix by:
   - Replacing arbitrary timeouts with event-based waiting
   - Fixing bugs in abort implementation if found
   - Adjusting test expectations if testing changed behavior

Do NOT just increase timeouts - find the real issue.

Return: Summary of what you found and what you fixed.
```

## Common Mistakes

**❌ Too broad:** "Fix all the tests" - agent gets lost
**✅ Specific:** "Fix agent-tool-abort.test.ts" - focused scope

**❌ No context:** "Fix the race condition" - agent doesn't know where
**✅ Context:** Paste the error messages and test names

**❌ No constraints:** Agent might refactor everything
**✅ Constraints:** "Do NOT change production code" or "Fix tests only"

**❌ Vague output:** "Fix it" - you don't know what changed
**✅ Specific:** "Return summary of root cause and changes"

## When NOT to Use

**Related failures:** Fixing one might fix others - investigate together first
**Need full context:** Understanding requires seeing entire system
**Exploratory debugging:** You don't know what's broken yet
**Shared state:** Agents would interfere (editing same files, using same resources)

## Real Example from Session

**Scenario:** 6 test failures across 3 files after major refactoring

**Failures:**
- agent-tool-abort.test.ts: 3 failures (timing issues)
- batch-completion-behavior.test.ts: 2 failures (tools not executing)
- tool-approval-race-conditions.test.ts: 1 failure (execution count = 0)

**Decision:** Independent domains - abort logic separate from batch completion separate from race conditions

**Dispatch:**
```
Agent 1 → Fix agent-tool-abort.test.ts
Agent 2 → Fix batch-completion-behavior.test.ts
Agent 3 → Fix tool-approval-race-conditions.test.ts
```

**Results:**
- Agent 1: Replaced timeouts with event-based waiting
- Agent 2: Fixed event structure bug (threadId in wrong place)
- Agent 3: Added wait for async tool execution to complete

**Integration:** All fixes independent, no conflicts, full suite green

**Time saved:** 3 problems solved in parallel vs sequentially

## Key Benefits

1. **Parallelization** - Multiple investigations happen simultaneously
2. **Focus** - Each agent has narrow scope, less context to track
3. **Independence** - Agents don't interfere with each other
4. **Speed** - 3 problems solved in time of 1

## Verification

After agents return:
1. **Review each summary** - Understand what changed
2. **Check for conflicts** - Did agents edit same code?
3. **Run full suite** - Verify all fixes work together
4. **Spot check** - Agents can make systematic errors

## Real-World Impact

From debugging session (2025-10-03):
- 6 failures across 3 files
- 3 agents dispatched in parallel
- All investigations completed concurrently
- All fixes integrated successfully
- Zero conflicts between agent changes

## Team Mode (Tier 3)

When tasks are **interdependent** and agents need to coordinate directly, use team mode.

### When to Use Team Mode
- 3+ tasks where agent output feeds into another agent's work
- Review-fix loops (reviewer finds issue → implementer fixes → re-review)
- Multiple agents working on shared contracts (frontend + backend agreeing on API shape)
- Full-stack features where changes need to stay synchronized

### Team Setup
```
TeamCreate → define team purpose
TaskCreate → add tasks from plan
Agent(team_name, name) → spawn teammates
TaskUpdate(owner) → assign tasks
SendMessage → coordinate between agents
```

### Example: Full-Stack Feature Team
```
Lead (you) → creates tasks, reviews progress, resolves conflicts
├── backend-dev (general-purpose) — API endpoints, data layer
├── frontend-dev (general-purpose) — UI components, state
├── reviewer (general-purpose, prompted with skills/requesting-code-review/code-reviewer.md) — reviews both, flags contract mismatches
└── simplifier (code-simplifier) — cleans up after reviews pass
```

Backend and frontend can message each other about types/API shape. Reviewer watches both and catches drift.

### Example: Debug Investigation Team
```
Lead (you) → synthesizes findings, decides fix approach
├── explorer-happy (Explore) — traces the working path
├── explorer-error (Explore) — traces the failing path
└── reviewer (general-purpose, prompted with skills/requesting-code-review/code-reviewer.md) — cross-references both findings
```

### Team Mode Rules
- Always assign a clear scope to each teammate
- Use TaskCreate for all work items so progress is visible
- Teammates communicate via SendMessage, not through you (that's the whole point)
- Shut down teammates when done: SendMessage with type "shutdown_request"
- Clean up with TeamDelete after all agents are done
