# claude-skills

> The open skill manager for Claude Code — install reusable AI behaviors from **any** GitHub repo with a single command.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-compatible-blueviolet)](https://claude.ai/code)

---

## Why this exists

The official `anthropics/skills` registry ships 17 curated skills. That's a good start — but the real value of Claude Code skills is the long tail: your company's coding conventions, your personal workflow shortcuts, community-built domain skills that Anthropic will never publish.

**claude-skills** gives you one command to install any of them:

```
/install alice/my-coding-conventions
/install anthropics/skills/skills/frontend-design
/install yourcompany/claude-standards
```

If the repo has a `CLAUDE.md` (or `SKILL.md`), it's installable. No approval process, no registry submission, no copy-paste.

---

## vs. the official approach

| | Manual copy-paste | Official `anthropics/skills` | **claude-skills** |
|---|:---:|:---:|:---:|
| Install official Anthropic skills | — | ✓ | ✓ |
| Install any GitHub repo as a skill | — | ✗ | ✓ |
| Update skills to latest version | — | — | ✓ |
| Remove a skill cleanly | — | — | ✓ |
| No approval / no registry needed | — | ✗ | ✓ |
| Works today, zero config | ✓ | — | ✓ |

---

## Install the skill manager

```bash
mkdir -p ~/.claude/commands && curl -fsSL https://raw.githubusercontent.com/bernhardbrugger/claude-skills/master/commands/install.md -o ~/.claude/commands/install.md
```

That's it. The `/install` command is now available in every Claude Code session.

---

## Usage

### Install a skill

**Any community skill** — just needs a `CLAUDE.md` in the repo root:

```
/install alice/tdd-conventions
/install yourcompany/coding-standards
```

**Official Anthropic skills** from the [anthropics/skills](https://github.com/anthropics/skills) registry:

```
/install anthropics/skills/skills/frontend-design
/install anthropics/skills/skills/claude-api
/install anthropics/skills/skills/webapp-testing
```

YAML frontmatter is automatically stripped from official skills — you get clean instruction content.

### List installed skills

```
/install list
```

```
Installed skills:
  1. alice/tdd-conventions
  2. anthropics/skills/skills/frontend-design
```

### Update skills

Re-fetch all installed skills from their source repos:

```
/install update
```

Or update a single skill:

```
/install update anthropics/skills/skills/frontend-design
```

```
✓ Updated: anthropics/skills/skills/frontend-design
⚠ No changes: alice/tdd-conventions
```

### Remove a skill

```
/install remove alice/tdd-conventions
```

```
✓ Skill removed: alice/tdd-conventions
```

---

## Browse available skills

### Official Anthropic skills

All installable via `/install anthropics/skills/skills/<name>`:

| Skill | What it does |
|---|---|
| `frontend-design` | Production-grade UI with distinctive aesthetics, avoids generic AI look |
| `claude-api` | Build apps with the Claude/Anthropic SDK |
| `webapp-testing` | Web application testing workflows |
| `web-artifacts-builder` | Build shareable web artifacts and mini-apps |
| `mcp-builder` | Create Model Context Protocol (MCP) servers |
| `pdf` | Work with PDF files |
| `docx` | Work with Word documents |
| `xlsx` | Work with Excel spreadsheets |
| `pptx` | Work with PowerPoint presentations |
| `doc-coauthoring` | Collaborative document writing |
| `canvas-design` | Design work in canvas environments |
| `algorithmic-art` | Generative and algorithmic art creation |
| `brand-guidelines` | Apply brand guidelines consistently |
| `theme-factory` | Create and apply design themes |
| `internal-comms` | Internal communication and writing |
| `slack-gif-creator` | Create GIFs for Slack |
| `skill-creator` | Build new Claude Code skills |

### Community skills

> Publishing your skill here? Open a PR or an issue.

---

## Publish your own skill

Add a `CLAUDE.md` to any public GitHub repo. That file should describe what behaviors, conventions, or workflows the skill adds:

```markdown
# My Team's Standards

When working in any project, always:
- Use TypeScript strict mode
- Write tests before implementing (TDD)
- Prefer composition over inheritance

When creating a new module, scaffold:
- A `.ts` file with the implementation
- A co-located `.test.ts` file
- An export in the nearest `index.ts`
```

Once pushed, anyone can install it with:

```
/install your-github-username/your-repo
```

No registration. No approval. Just push and share the path.

---

## How it works

1. **`/install author/repo`** fetches `CLAUDE.md` from the GitHub repo root (tries `main`/`master` branches and `.claude/` subdirectory).
2. **`/install author/repo/path/to/skill`** fetches from a subdirectory — tries `SKILL.md` first (official Anthropic format), then `CLAUDE.md`. YAML frontmatter is stripped automatically.
3. The content is wrapped in tagged delimiters and appended to `~/.claude/CLAUDE.md`:
   ```
   <!-- skill: author/repo -->
   ...skill content...
   <!-- /skill: author/repo -->
   ```
4. **Deduplication**: installing the same skill twice is a no-op.
5. **`/install update`** re-fetches and replaces skill blocks in place, preserving everything else in your CLAUDE.md.
6. **`/install remove`** surgically removes the tagged block, leaving the rest of your CLAUDE.md intact.

---

## File structure

```
claude-skills/
├── commands/
│   └── install.md    # The /install slash command
├── skills/           # Reserved for bundled skills
└── README.md
```

---

## License

MIT
