# Team Builder - Claude Code Plugin

Analyzes any idea or project and intelligently builds a Claude Code agent team with optimized roles, prompts, and task dependencies.

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
/team-builder:team-builder

# With argument - goes straight to analysis
/team-builder:team-builder a React Native finance app with charts and goals
```

## What It Does

1. **Understands your idea** - asks 2-3 clarifying questions only if needed
2. **Analyzes context** - reads your codebase, detects stack, identifies patterns
3. **Builds the team** - creates optimized agent roles with clear prompts and deliverables
4. **Shows a visual plan** - dependency diagram before executing
5. **Asks for confirmation** - you approve or adjust before the team is created

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
