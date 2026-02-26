# Team Builder - Claude Code Plugin

Analyzes any idea or project and intelligently builds a Claude Code agent team with optimized roles, prompts, and task dependencies — then **actually executes it**.

**v3.0** - The agent becomes the **team leader**: spawns real teammates, waits for each wave to finish, coordinates inter-agent communication, and delivers the final result.

## Requirements

- Claude Code v2.1+ with Agent Teams enabled
- `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in your settings

## Installation

Add the marketplace:

```bash
/plugin marketplace add wesleyisr4/claude-team-builder-plugin
```

Install the plugin:

```bash
/plugin install team-builder@wesley-plugins
```

## Usage

```bash
# Interactive - asks what you want to build
/team-builder

# With argument - goes straight to analysis
/team-builder a React Native finance app with charts and goals
```

## How It Works

1. **Understands your idea** - asks 2-3 clarifying questions only if needed
2. **Analyzes context** - reads your codebase, detects stack, identifies patterns
3. **Builds the team plan** - roles, prompts, deliverables, dependency graph
4. **Shows visual plan** - dependency diagram + role table for your approval
5. **You approve** - adjust or confirm before execution

Then it **executes for real**:

6. `TeamCreate` - creates structured team with shared task list
7. `TaskCreate` - creates a task per agent with detailed descriptions
8. `TaskUpdate` - sets up dependency chain (`blockedBy`)
9. `Task` (wave 1) - spawns agents with no blockers (e.g. Explorer)
10. **Waits** - team leader waits for wave 1 to finish
11. `Task` (wave 2) - spawns next agents (e.g. Architect)
12. **Repeats** - continues spawning waves as dependencies resolve
13. `Task` (final wave) - spawns QA/Reviewer to validate everything
14. **Delivers** - summary of all deliverables + cleanup

## Execution Flow

```
You ──▶ /team-builder "my project idea"
         │
         ▼
    [PLAN + CONFIRM]
         │
         ▼
    TeamCreate("my-project")
         │
         ▼
    TaskCreate × N (all tasks)
         │
         ▼
    TaskUpdate (dependencies)
         │
         ▼
    ┌─ Task: Explorer (Haiku) ──── wave 1
    │  └─ waits...
    │
    ├─ Task: Architect (Sonnet) ── wave 2
    │  └─ waits...
    │
    ├─ Task: Dev A (Sonnet) ──┐
    ├─ Task: Dev B (Sonnet) ──┤── wave 3 (parallel)
    ├─ Task: Dev C (Sonnet) ──┘
    │  └─ waits...
    │
    ├─ Task: QA (Sonnet) ──────── wave 4
    │  └─ waits...
    │
    └─ Summary + TeamDelete
```

## Supported Project Types

| Type | Team Composition |
|------|-----------------|
| Software | Explorer, Architect, Developer(s), Tester, Reviewer |
| Content | Researcher, Architect, Writer, Editor, Publisher |
| Research | Explorer A/B/C (parallel), Synthesizer, Critic |
| Automation | Researcher, Architect, Builder, Tester, Documenter |
| Refactoring | Analyzer, Architect, Migrator(s), Validator |

## Modifiers

- Say **"fast"** or **"mvp"** for 2-3 focused agents
- Say **"complete"** or **"detailed"** for 5-6 agents with full coverage
- Speaks **your language** (PT-BR, EN, ES, etc.)

## License

MIT
