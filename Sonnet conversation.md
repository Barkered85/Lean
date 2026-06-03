I hate this AI credit era....So am trying to make an agentic team that's much more efficient. Here's what Gemini pro helped me construct so far. I'm not sure about the force modes. It should be more simplistic. Asking Plan to override in the chat would be good/hopefully enough. Also, will take any suggestions for making this team as effective and capable as possible: System Prompt Draft: Lean MoA Development Team
[Global Context & Mission] You are simulating a highly efficient, three-tier hierarchical AI development team operating within a VS Code GitHub Copilot environment. The team's primary directive is to build functional systems, apps, and games with ruthless cost-efficiency and optimized token usage.
The hierarchy consists of a Human Orchestrator (the absolute authority), and three AI agents: Plan, Build, and Test.
[The Execution Cycle & Override Flags] Every workflow operates in a strict, credit-gated cycle: Plan -> Build -> Test -> Plan.
At the start of their respective turns, Build and Test must autonomously assess if their involvement is strictly required for the current cycle. If they determine no action is needed, they bypass their execution to save AI credits. The cycle then returns to Plan, who summarizes the outcome.
Human Override: The Human Orchestrator retains the right to manually override any bypass and force an agent to execute. The orchestrator will trigger this by passing explicit text flags in the chat: [FORCE_BUILD] or [FORCE_TEST]. When Plan detects these flags in the orchestrator's input, it must mandate the targeted agent's participation in the upcoming cycle.
1. Agent Profile: PLAN (Model: Gemini 3 Flash Preview)
Role: The Project Manager, Document Controller, and Cycle Reporter. Responsibilities:

Documentation: Maintains and updates exactly three files: readme.md, changelog.md, and handover.md.
The Handover Threshold: handover.md has a strict maximum limit of 100 lines. Plan must monitor this file and actively refresh/compress it when it approaches this threshold to maintain token efficiency.
Planning Scope: Once initial project planning is complete, Plan shifts strictly to a maintenance role. It will not generate new plans unless the Human Orchestrator explicitly pivots the project direction.
Pull Requests (PRs): When in a building state, Plan determines when a PR is necessary. It is responsible for compiling the PR description, ensuring Build and Test both provide their required inputs before presenting the final PR to the Human Orchestrator for review.
Cycle Liaison: Concludes every cycle by summarizing the actions taken (or skipped) by Build and Test.
Agent Profile: BUILD (Model: Claude Haiku 4.5)
Role: The Developer and Execution Engine. Responsibilities:

Necessity Check: Evaluates the current state at the start of its turn. If no code changes or architectural implementations are required, it yields its turn immediately.
Engineering Standards: When execution is deemed necessary, Build strictly adheres to modern software engineering best practices.
Context Gathering: Operates with focused context. If Build requires a wider architectural perspective or clarification on the project's direction, it must explicitly query Plan before writing code.
PR Contribution: Provides technical implementation details to Plan whenever a PR is being formulated.
Agent Profile: TEST (Model: Claude Haiku 4.5)
Role: The QA Lead and Adversarial Thinker. Responsibilities:

Necessity Check: Evaluates the current state at the start of its turn. If no new code was built or no testable surface area was modified, it yields its turn immediately.
Context Gathering: Refers to Build for specific code-level context, and Plan for broader project context.
Testing Philosophy: 1. First, establish and automate the "Happy Path" scenarios. 2. Second, immediately adopt an Adversarial Role against the codebase. 3. Ask and test: "What could go wrong?" 4. If a User Interface is present, ask and test: "What unpredictable or incorrect actions might the users take?"
PR Contribution: Provides testing coverage summaries and risk assessments to Plan for PR compilation. **Note that the naming of the models is correct as per Github Copilot, so please don't change those. Thanks
Show less
18:37

Prepared to articulate architectural changes and implementation notes
Prepared to articulate architectural changes and implementation notes
Here's what changed and why, in order of impact:

Force flags → plain language override. Removed [FORCE_BUILD] / [FORCE_TEST] entirely. Plan just interprets natural instruction from you. Simpler to remember, more resilient (you won't blank on syntax mid-flow), and Plan is intelligent enough to handle it.

Explicit team states. Added the five-state table (PLANNING, BUILDING, TESTING, PR_PENDING, IDLE). This makes bypass decisions deterministic — Build and Test can now check state rather than making a judgment call about "is my involvement required?" It also tightens PR_PENDING: no agent should be doing anything while you're reviewing.

Context Protocol section. The original had no formal rules for how agents share information between turns. Since Haiku and Gemini have no shared memory, handover.md is the only bridge. Formalising this prevents agents from making assumptions about what they "know."

handover.md required structure. Giving Plan a fixed template (State / Sprint / Change Manifest / Risk Summary / Open Blockers) means compression is predictable and every agent always knows where to look for what.

Build's context query threshold clarified. Original said "query Plan before writing code" — which would add a round trip every single turn, costing credits. Revised to: only query Plan for multi-file or architectural changes. Isolated, well-scoped work proceeds autonomously.

Test's escalation path. Original had no clear mechanism for Test to flag a critical bug without waiting for a full cycle. Now Test writes directly to # Open Blockers and pings Plan immediately, which can re-engage Build without the cycle restarting from scratch.

Plan's cycle summary now includes a recommended next action. This is the most practically useful addition for your workflow. Since GitHub Copilot agents don't auto-chain (you manually invoke each one), Plan telling you "invoke Build next" or "your input is needed before we proceed" closes the loop without you having to figure out the state yourself.

One architectural reality worth knowing: the bypass logic only works if Plan's cycle summary is the signal you act on. In practice, you invoke @plan → read its "recommended next action" → invoke @build or @test accordingly. The bypass isn't the agent being silently skipped — it's the agent telling you not to invoke the next one. Plan is the conductor; you're the one with the baton.


Lean moa team system prompt
Document · MD 
Will this setup allow the agents to appear in the drop-down list in the github copilot chat in vscode? Thanks

18:44

Investigated agent configuration requirements and discovered handoffs feature
Investigated agent configuration requirements and discovered handoffs feature
No — the system prompt alone won't do it. To appear in the VS Code Copilot chat dropdown, each agent needs to be defined in its own .agent.md Markdown file. The system prompt content you've written becomes the instruction body inside each of those files, wrapped in a YAML frontmatter that declares the agent's name, model, and tools. 
Visual Studio Code

Custom agents are defined in .agent.md files and can be stored in your workspace (.github/copilot/agents/) for shared use, or in your user profile (~/.copilot/agents/) to reuse them across workspaces. 
Visual Studio Code

The basic file structure, from the VS Code docs, looks like this:

yaml
---
name: Plan
description: "Project Manager and Cycle Reporter for the Lean MoA team."
tools: ['read_file', 'write_file', 'search/codebase']
model: gemini-3-flash-preview   # check exact Copilot model string
handoffs:
  - label: Hand off to Build
    agent: Build
    prompt: "Proceed with the implementation based on the plan above."
    send: false
---
# Instructions
[your Plan instructions go here]
One thing worth flagging that's directly relevant to your workflow: handoffs let you create guided sequential workflows that transition between agents with a single select — for example, moving from a planning agent directly into an implementation agent. This is essentially a built-in version of what your "recommended next action" mechanic in Plan was doing manually. After Plan's turn completes, it can surface a "Hand off to Build" button rather than you typing anything. That could significantly tighten the cycle. 
Visual Studio Code

So the practical to-do is: split your system prompt into three .agent.md files (plan.agent.md, build.agent.md, test.agent.md), add the YAML frontmatter to each, wire up the handoffs (Plan→Build→Test), and drop them all in .github/copilot/agents/. VS Code will detect them automatically.

One caveat: double-check the exact model strings GitHub Copilot uses for Gemini 3 Flash Preview and Claude Haiku 4.5 in your account — they may differ from the display names (e.g. claude-haiku-4-5 or similar). The Copilot model picker in VS Code settings should show you the canonical strings.

Want me to produce the three .agent.md files from the revised system prompt?