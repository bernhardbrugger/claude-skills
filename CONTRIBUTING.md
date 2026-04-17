# Contributing

## Add your skill to the community list

If you've published a Claude Code skill and want it listed in the README:

1. Make sure your repo has a `CLAUDE.md` at the root (or in `.claude/`)
2. Add the `claude-skills` GitHub topic to your repo (Settings → Topics)
3. Open an issue or PR with:
   - Your install path: `author/repo`
   - One-line description of what the skill does
   - Confirmation that it's general-purpose (not project-specific config)

## What makes a good community skill

- **Focused**: does one thing well, not a dump of personal config
- **General-purpose**: useful to someone on a completely different project
- **Instructive**: the `CLAUDE.md` contains clear behavioral instructions, not just documentation

## Improve the install command

The core logic lives in `commands/install.md`. It's a Claude Code slash command — plain markdown that Claude executes as instructions.

To propose a change:
1. Fork and edit `commands/install.md`
2. Test it in a Claude Code session by copying the file to `~/.claude/commands/install.md`
3. Open a PR describing what you tested and what the before/after behavior is

## Report a bug

Open an issue with:
- The exact `/install` command you ran
- What happened vs. what you expected
- Your OS and Claude Code version
