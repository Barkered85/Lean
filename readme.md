# Lean MoA Development Team

## Overview
This repository implements a highly efficient, three-tier agentic development team designed to operate within VS Code GitHub Copilot. The system is built for functional software development — apps, games, and tools — with a hard focus on token efficiency and credit-gated execution. Plan orchestrates everything. Build and Test run as subagents within Plan's turn. The human only ever talks to Plan.

---

## Getting Started

1. Clone the repo.
2. Select **Planner** from the VS Code Chat agents dropdown.
3. Describe your goal in plain language. Planner will scope the work, call Build and Test as subagents, and report back to you — all within a single turn.
4. Continue talking to Planner. It drives the full cycle.

---

## The Hierarchy

```
Human Orchestrator  (absolute authority)
        │
      Planner       (Project Manager, Document Controller, Cycle Reporter)
      ┌─┴─┐
    Build  Test     (called as subagents by Planner — not directly by the human)
```

The Human Orchestrator talks only to Planner. Planner conducts the cycle, calling Build and Test in sequence as subagents using the `runSubagent` tool. Build and Test do not hand off to each other and do not speak to the Human Orchestrator.

---

## The Agents

### Planner — Gemini 3 Flash (Preview)
Planner is the conductor and the only agent the human interacts with. It is the sole owner of the team's shared documentation files (`readme.md`, `changelog.md`, `handover.md`) and the only agent that writes the `# State:` line.

**What it does each turn:**
1. Checks `changelog.md` entry count — at 5 entries runs `/compact`; at 10+ recommends a new chat (state is safely preserved in `handover.md`).
2. Reads `handover.md` to determine current state and any new Human direction.
3. Updates state and sprint goal in `handover.md`.
4. Calls **Build** as a subagent with the task. Writes Build's Change Manifest to `handover.md`.
5. Calls **Test** as a subagent. Writes Test's Risk Summary to `handover.md`.
6. If Test finds a blocker: re-engages Build up to **2 attempts** on Haiku. On third failure, escalates to **GPT-5.3-Codex**.
7. Manages file compression: when `handover.md` approaches 80 lines, archives completed cycle detail into `changelog.md` and resets `handover.md`.
8. Closes every turn with a **Cycle Summary** — what ran, what was skipped, current state, and whether Human input is needed.

**Human override:** Plain language instructions like *"skip Test this cycle"* or *"force Build even if nothing changed"* are interpreted naturally.

---

### Build — Claude Haiku 4.5
Build is the implementation engine. It is called as a subagent by Planner. It touches code and the filesystem, but **never writes the `# State:` line** in `handover.md` — that belongs exclusively to Planner. Writing the wrong state is what causes the team to deadlock.

**What it does each turn:**
1. Reads `handover.md` and the task passed by Plan.
2. Implements to modern engineering standards: clean naming, appropriate error handling, no dead code.
3. Returns a **Change Manifest** to Plan (and writes it to `# Last Change Manifest` in `handover.md`):
   - Files modified
   - Nature of changes
   - Known risks or limitations

---

### Test — Claude Haiku 4.5
Test is the last line of defence before code is considered done. It is called as a subagent by Planner after Build completes. It is adversarial by design.

**What it does each turn:**
1. Reads `handover.md`. If there is no new Change Manifest from Build, it bypasses immediately.
2. If proceeding, tests in strict order:
   - **Happy Path first** — does the system work under normal conditions?
   - **Adversarial second** — what edge cases and failure modes exist?
   - **UI-specific (if applicable)** — invalid inputs, navigation misuse, accessibility.
3. **Critical defect reporting:** Writes blockers to `# Open Blockers` in `handover.md` with full reproduction detail. Sets `Build re-engagement needed: Yes` in its Risk Summary. **Returns to Planner — does not re-engage Build directly.** Planner controls escalation.
4. Returns a **Risk Summary** to Plan (and writes it to `# Last Risk Summary` in `handover.md`):
   - Pass / Pass with caveats / Fail
   - Key risks identified
   - Whether Build re-engagement is needed

---

## Bug Fix Escalation

When Test reports a critical blocker, Plan tracks fix attempts in `# Open Blockers`:

| Attempt | Agent | Notes |
|---------|-------|-------|
| 1 | Build (Haiku 4.5) | Standard fix attempt |
| 2 | Build (Haiku 4.5) | Second attempt with added context from attempt 1 |
| 3+ | Build (**GPT-5.3-Codex**) | Escalated — full blocker history passed |

After each attempt, Test is re-run as a subagent to validate the fix.

---

## Team States

The team always operates in one of five states, maintained by Planner in `handover.md`. **Only Planner writes this value.**

| State        | Meaning                                              |
|--------------|------------------------------------------------------|
| `PLANNING`   | Initial scoping or a Human-triggered direction pivot |
| `BUILDING`   | Active code development underway                     |
| `TESTING`    | QA and validation in progress                        |
| `PR_PENDING` | PR compiled; awaiting Human Orchestrator approval    |
| `IDLE`       | No active work; awaiting direction                   |

---

## Context Management

`/compact` reduces context within the current chat session (good for ongoing work). A **new chat** is the true clean slate — safe to do because Planner reads `handover.md` on startup and resumes from full preserved state.

| Signal | Action |
|--------|--------|
| 5 `changelog.md` entries | Planner runs `/compact` automatically |
| 10+ entries, or chat feels slow | Start a new chat; select **Planner**; describe next task |

Planner checks and acts on this at the start of every turn — you don't need to manage it.


---

## Shared Context — The Three Files

All agents share context exclusively through three files. There is no other shared memory between turns.

| File           | Owner | Purpose                                                                 |
|----------------|-------|-------------------------------------------------------------------------|
| `readme.md`    | Plan  | Project overview and setup instructions                                 |
| `handover.md`  | Plan  | Live team state: sprint goal, Change Manifest, Risk Summary, blockers   |
| `changelog.md` | Plan  | Timestamped archive of completed cycles, compressed from `handover.md`  |

### handover.md structure
```
# State: [current state]
# Current Sprint: [active goal — 1 to 3 lines max]
# Last Change Manifest: [from Build]
# Last Risk Summary: [from Test]
# Open Blockers: [critical issues requiring action, or "None"]
```

When `handover.md` approaches 80 lines, Plan snapshots the current cycle content into `changelog.md` under a dated heading (`## [YYYY-MM-DD HH:MM] — [Sprint summary]`) and resets the file to the clean structure above.

