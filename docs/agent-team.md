# Agent team

I'm using **GitHub Copilot CLI** in a **Codespace** to coordinate a team of custom agents that will build Mona's **Project Pulse** dashboard. Each agent is defined in the `.github/agents/` folder.

| Agent | Model | Responsibility | Definition |
| --- | --- | --- | --- |
| **Orchestrator** | Claude Opus 5.5 | Splits the request into phases, hands work to the specialists with an explicit file scope for each, checks that the combined result fits together, and reports progress. It doesn't write code itself and doesn't run any git operations. | `.github/agents/orchestrator.agent.md` |
| **Planner** | Claude Opus 5.5 | Researches the repository and relevant docs, then writes an implementation plan covering phases, which files each agent owns, dependencies, edge cases, and risks. It doesn't write code. | `.github/agents/planner.agent.md` |
| **Designer** | Gemini 3.7 Flash | Owns the UI/UX direction: usability, accessibility, information hierarchy, responsive layout, and visual clarity. | `.github/agents/designer.agent.md` |
| **Coder** | GPT-5.5 | Builds the static app files within its assigned file scope, keeps the code clear and testable with explicit error handling, and checks the changes work before reporting back. | `.github/agents/coder.agent.md` |

## How the team builds Project Pulse

1. I send the Project Pulse request to the **Orchestrator** from Copilot CLI.
2. The Orchestrator asks the **Planner** for a phased plan that assigns files to agents and lists dependencies.
3. The Orchestrator turns the plan into phases. Tasks run in parallel only when their file scopes don't overlap and neither depends on the other's output. Otherwise they run one after the other.
4. The **Designer** sets the dashboard experience and styling direction, and the **Coder** implements the app files within its assigned scope.
5. The Orchestrator checks that the combined result works and summarizes the outcome. I handle staging, committing, and pushing myself with Copilot CLI prompts.
