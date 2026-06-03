# Lean MoA Development Team

## Overview
This repository implements a highly efficient, three-tier agentic development team designed to operate within VS Code GitHub Copilot. The system is built for functional software development — apps, games, and tools — with a hard focus on token efficiency and credit-gated execution. Agents are autonomous but coordinated: each one reads shared state before acting, decides whether it is genuinely needed, and bypasses if not.

To get started, clone the repo, then pick Plan from the GitHub Copilot chat and start planning your project. 

---

## The Hierarchy

```
Human Orchestrator  (absolute authority)
        │
       Plan         (Project Manager, Document Controller, Cycle Reporter)
        │
       Build        (Developer, Execution Engine)
        │
       Test         (QA Lead, Adversarial Thinker)
```

The Human Orchestrator drives direction. Plan conducts. Build and Test execute — or bypass.

---

## The Agents

### Plan — Gemini 3 Flash (Preview)
Plan is the conductor of the team. It is the only agent that touches the three managed files (`readme.md`, `changelog.md`, `handover.md`) and the only agent that speaks directly to the Human Orchestrator.

**What it does each turn:**
1. Reads `handover.md` to determine current state and any new Human direction.
2. Updates the state and sprint goal in `handover.md`.
3. Manages files: compresses `handover.md` when it approaches 80 lines, archiving completed cycle detail into `changelog.md` as a timestamped entry, then resetting `handover.md` to its clean 5-section structure.
4. Compiles pull requests from Build's Change Manifest and Test's Risk Summary, presenting them to the Human Orchestrator for approval.
5. Closes every turn with a **Cycle Summary** — what ran, what was bypassed, and a clear **recommended next action**.

**Human override:** No syntax required. Plain language instructions like *"make Build run this cycle regardless"* are interpreted naturally and reflected in `handover.md`.

**Handoffs:** Plan → Build, or Plan → Test (when Build is bypassed).

---

### Build — Claude Haiku 4.5
Build is the implementation engine. It touches code and the filesystem, but never manages the team's shared documentation files — that belongs to Plan.

**What it does each turn:**
1. Reads `handover.md`. If the state is `PLANNING`, `PR_PENDING`, or `IDLE`, or there is no actionable build task, it **bypasses immediately** with a one-line notice and hands back to Plan.
2. If proceeding: checks whether the task is isolated (single file, clear scope) or architectural (multi-file, structural). Isolated tasks proceed autonomously. Architectural tasks trigger a brief direction-check with Plan before code is written — a short sync costs fewer credits than rework.
3. Implements to modern engineering standards: clean naming, appropriate error handling, no dead code or orphaned imports.
4. Concludes every active turn by writing a **Change Manifest** to `handover.md`:
   - Files modified
   - Nature of changes
   - Known risks or limitations

**Handoffs:** Build → Test (after completing work), or Build → Plan (if bypassing).

---

### Test — Claude Haiku 4.5
Test is the last line of defence before code is considered done. It is adversarial by design.

**What it does each turn:**
1. Reads `handover.md`. If the state is `PLANNING` or `IDLE`, or no Change Manifest from Build exists in the current cycle, it **bypasses immediately** with a one-line notice and returns to Plan.
2. If proceeding, it tests in strict order:
   - **Happy Path first** — does the system do what it is supposed to under normal conditions? Automate where possible.
   - **Adversarial second** — what could fail? What edge cases exist? What would a hostile or careless user do?
   - **UI-specific (if applicable)** — unpredictable inputs, navigation misuse, accessibility edge cases.
3. **Critical defect escalation:** If a blocker is found, Test writes it directly to `# Open Blockers` in `handover.md` and uses the **"Re-engage Build"** handoff immediately — it does not wait for a full cycle restart.
4. Concludes every active turn by writing a **Risk Summary** to `handover.md`:
   - Pass / Pass with caveats / Fail
   - Key risks identified
   - Whether Build re-engagement is needed

**Handoffs:** Test → Plan (normal completion), or Test → Build (critical blocker found).

---

## Team States

The team always operates in one of five states, maintained by Plan in `handover.md`:

| State        | Meaning                                              |
|--------------|------------------------------------------------------|
| `PLANNING`   | Initial scoping or a Human-triggered direction pivot |
| `BUILDING`   | Active code development underway                     |
| `TESTING`    | QA and validation in progress                        |
| `PR_PENDING` | PR compiled; awaiting Human Orchestrator approval    |
| `IDLE`       | No active work; awaiting direction                   |

Build and Test use this state as their primary bypass signal. When `PR_PENDING`, all agents stand down until the Human Orchestrator acts.

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

---

## Getting Started

1. Select **Plan** from the VS Code Chat agents dropdown.
2. Describe your goal in plain language. Plan will scope the work and set the state.
3. Use the **handoff buttons** at the end of Plan's response to transition to Build or Test.
4. Follow Plan's **recommended next action** at the close of every cycle — it tells you exactly which agent to invoke next, or whether your input is needed before the team can proceed.
