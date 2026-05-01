# FAM Skills

Domain-specific instruction sets for AI agents running on the Familiar Agentic Matrix (FAM).

Skills are modular context packages — structured knowledge that an agent loads on demand to handle a specific category of work. Each skill contains a primary `SKILL.md` and optional reference files the agent loads as needed during the task.

## How skills work

An agent loads a skill when the task matches its description. The skill injects structured instructions, workflows, and reference material into the active session. The agent follows the skill's protocol for the duration of that task.

Skills are agent-agnostic — they work with OpenCode, Claude Code, or any runtime that supports file-based context loading.

## Available skills

| Skill | Description |
|-------|-------------|
| [figma-to-web](./figma-to-web/) | Implement web interfaces from Figma designs with 1:1 visual fidelity |
| [figma-to-swift](./figma-to-swift/) | Implement iOS SwiftUI interfaces from Figma designs with 1:1 visual fidelity |

## Skill structure

Each skill follows this layout:

```
skill-name/
  SKILL.md          # Primary skill file — the agent loads this
  references/       # Supporting docs — loaded on demand per phase
    *.md
```

The `SKILL.md` is the entry point. It defines when the skill applies, what tools it requires, the workflow phases, and which reference files to load at each stage.

## Using a skill

### OpenCode

Add the skill to your `.opencode/skills/` directory (or the global `~/.config/opencode/skills/`), then reference it in your agent config or load it inline.

### Claude Code

Add the skill to your `~/.claude/skills/` directory and wire it through your agent instructions file.

### Manual

Paste the `SKILL.md` content (and relevant reference files) into your system prompt or context window at the start of a session.

## Contributing

Skills in this repo are maintained by [A Damn Fine Co.](https://github.com/adamnfineco). If you've built a skill that holds up in production, open a PR.

Structure requirements:
- `SKILL.md` must include a `name`, `description`, and clear `USE FOR` / `DO NOT USE FOR` boundaries
- Reference files should be focused and loadable independently (not require each other)
- No hardcoded project paths or personal config — skills should be portable

## Authors

Built by Mark Progano and Solin — [damnfine.xyz](https://damnfine.xyz)

## License

MIT
