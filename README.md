# claude-skills

> A skill manager for [Claude Code](https://claude.ai/code) — install reusable AI behaviors from any GitHub repo with a single command.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-compatible-blueviolet)](https://claude.ai/code)

---

## What is this?

Claude Code reads `~/.claude/CLAUDE.md` on every session start. Any instructions in that file shape how Claude behaves across all your projects.

**claude-skills** adds a `/install` slash command so you can pull reusable Claude behaviors ("skills") from any public GitHub repository — no manual copy-paste, no drift.

A "skill" is just a GitHub repo with a `CLAUDE.md` at its root (or in `.claude/`). That's it.

---

## One-liner install

```bash
cp "$(gh repo clone bernhardbrugger/claude-skills --json path -q '.path' 2>/dev/null || git clone https://github.com/bernhardbrugger/claude-skills /tmp/claude-skills && echo /tmp/claude-skills)/commands/install.md" ~/.claude/commands/install.md
```

Or manually:

```bash
# 1. Clone the repo
git clone https://github.com/bernhardbrugger/claude-skills
# 2. Copy the command
mkdir -p ~/.claude/commands
cp claude-skills/commands/install.md ~/.claude/commands/install.md
```

That's it. The `/install` command is now available in every Claude Code session.

---

## Usage

### Install a skill

**Community skill** (any public GitHub repo with a `CLAUDE.md`):

```
/install alice/tdd-conventions
```

**Official Anthropic skill** from the [anthropics/skills](https://github.com/anthropics/skills) registry:

```
/install anthropics/skills/skills/frontend-design
```

Both fetch the skill content and append it to your `~/.claude/CLAUDE.md`. Future sessions will automatically include those instructions.

For official skills, the command automatically handles the nested path (`anthropics/skills` repo → `skills/frontend-design/` subdirectory) and strips the YAML frontmatter that the official registry adds.

### List installed skills

```
/install list
```

```
Installed skills:
  1. alice/tdd-conventions
  2. anthropics/skills/skills/frontend-design
```

### Remove a skill

```
/install remove alice/tdd-conventions
/install remove anthropics/skills/skills/frontend-design
```

```
✓ Skill removed: alice/tdd-conventions
✓ Skill removed: anthropics/skills/skills/frontend-design
```

---

## How it works

1. `/install author/repo` fetches `CLAUDE.md` from the GitHub repo (tries `main` and `master` branches, root and `.claude/` subdirectory).
2. `/install author/repo/path/to/skill` fetches from a subdirectory within the repo — tries `SKILL.md` first (for official registries like `anthropics/skills`), then `CLAUDE.md`. YAML frontmatter is automatically stripped.
3. The content is wrapped in tagged delimiters and appended to `~/.claude/CLAUDE.md`:
   ```
   <!-- skill: author/repo -->
   ...skill content...
   <!-- /skill: author/repo -->
   ```
4. Duplicate installs are detected via the tag — installing the same skill twice is a no-op.
5. `/install remove` surgically removes the tagged block, leaving the rest of your CLAUDE.md intact.

---

## Making your repo skill-compatible

Just add a `CLAUDE.md` to your repository root (or `.claude/CLAUDE.md`). That file should describe:

- What behaviors or conventions this skill adds
- Any setup steps Claude should know about
- Instructions in natural language — Claude Code reads this before every session

**Example `CLAUDE.md` for a skill:**

```markdown
# My Skill Name

When working in this project, always:
- Use TypeScript strict mode
- Prefer functional components in React
- Write tests before implementing features (TDD)

When asked to create a new component, scaffold it with:
- A `.tsx` file
- A co-located `.test.tsx` file
- Exported from the nearest `index.ts`
```

Once your repo has a `CLAUDE.md`, anyone can install your skill with:

```
/install your-github-username/your-repo
```

---

## File structure

```
claude-skills/
├── commands/
│   └── install.md    # The /install slash command
├── skills/           # Reserved for future bundled skills
└── README.md
```

---

## License

MIT
