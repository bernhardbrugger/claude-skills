# claude-skills

> The open skill manager for Claude Code — install reusable AI behaviors from **any** GitHub repo with a single command.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-compatible-blueviolet)](https://claude.ai/code)
[![GitHub topics](https://img.shields.io/badge/community-claude--skills-orange)](https://github.com/topics/claude-skills)

---

## Why this exists

Claude Code skills are just GitHub repos. Anyone can write one — your company's coding conventions, your personal workflow shortcuts, domain knowledge Anthropic will never publish. The problem is there's no built-in way to install them.

**claude-skills** fixes that with one slash command:

```
/install @frontend-design              ← official Anthropic skill
/install wshobson/agents               ← any GitHub repo with a CLAUDE.md
```

If a repo has a `CLAUDE.md` (or `SKILL.md`), it's installable. No approval, no registry, no copy-paste.

---

## vs. the official approach

| | Manual copy-paste | Official registry only | **claude-skills** |
|---|:---:|:---:|:---:|
| Install official Anthropic skills | — | ✓ | ✓ |
| Install **any** GitHub repo as a skill | — | ✗ | ✓ |
| Short `@name` syntax for official skills | — | — | ✓ |
| Browse available skills | — | — | ✓ |
| Update skills to latest version | — | — | ✓ |
| Remove a skill cleanly | — | — | ✓ |
| No approval / no registry needed | — | ✗ | ✓ |

---

## Install the skill manager

```bash
mkdir -p ~/.claude/commands && curl -fsSL https://raw.githubusercontent.com/bernhardbrugger/claude-skills/master/commands/install.md -o ~/.claude/commands/install.md
```

That's it. Restart Claude Code and `/install` is available in every session.

---

## Usage

### Install a skill

**Official Anthropic skills** — use the `@name` shorthand:

```
/install @frontend-design
/install @claude-api
/install @webapp-testing
```

**Any community skill** — just needs a `CLAUDE.md` in the repo root:

```
/install wshobson/agents
```

Both fetch the skill content and append it to your `~/.claude/CLAUDE.md`.

### Browse available skills

```
/install search
```

```
Official skills  (install with /install @<name>)

  @frontend-design       Production-grade UI — web components, pages, apps with distinctive aesthetics
  @claude-api            Build and optimize apps with the Claude API / Anthropic SDK
  @webapp-testing        Test local web apps with Playwright — screenshots, logs, UI verification
  ...

Community skills  →  https://github.com/topics/claude-skills
```

Filter by keyword:

```
/install search document
```

### List installed skills

```
/install list
```

```
Installed skills:
  1. anthropics/skills/skills/frontend-design
  2. wshobson/agents
```

### Update skills

```
/install update            ← update all installed skills
/install update @pdf       ← update one skill
```

```
✓ Updated: anthropics/skills/skills/frontend-design
⚠ No changes: wshobson/agents
```

### Remove a skill

```
/install remove @frontend-design
/install remove wshobson/agents
```

---

## Official Anthropic skills

All installable with `/install @<name>`:

| `@name` | What it does |
|---|---|
| `@frontend-design` | Production-grade UI with distinctive aesthetics — websites, dashboards, React components |
| `@claude-api` | Build, debug, and optimize apps with the Claude API / Anthropic SDK (incl. prompt caching) |
| `@webapp-testing` | Test local web apps with Playwright — UI verification, screenshots, browser logs |
| `@web-artifacts-builder` | Build complex multi-component claude.ai HTML artifacts with React, Tailwind, shadcn/ui |
| `@mcp-builder` | Create MCP (Model Context Protocol) servers in Python (FastMCP) or TypeScript |
| `@pdf` | Read, create, merge, split, rotate, watermark, and OCR PDF files |
| `@docx` | Create, read, and edit Word documents (.docx) with full formatting |
| `@xlsx` | Create, read, edit, and convert spreadsheets (.xlsx, .csv, .tsv) |
| `@pptx` | Create, edit, and parse PowerPoint presentations (.pptx) |
| `@doc-coauthoring` | Structured workflow for writing docs, proposals, and technical specs collaboratively |
| `@canvas-design` | Create original visual designs as PNG/PDF — posters, art pieces, layouts |
| `@algorithmic-art` | Generative art with p5.js — flow fields, particle systems, seeded randomness |
| `@brand-guidelines` | Apply Anthropic's official brand colors and typography to any artifact |
| `@theme-factory` | Style artifacts (slides, docs, HTML pages) with 10 preset themes or a custom one |
| `@internal-comms` | Write internal communications — status reports, newsletters, incident reports, FAQs |
| `@slack-gif-creator` | Generate animated GIFs optimized for Slack |
| `@skill-creator` | Create and optimize new Claude Code skills, run evals to test performance |

---

## Community skills

Skills tagged [`claude-skills`](https://github.com/topics/claude-skills) on GitHub are findable by anyone. To add yours:

1. Push a repo with a `CLAUDE.md` at the root
2. Add the `claude-skills` topic to your repo (Settings → Topics)
3. Share the install path: `/install your-username/your-repo`

> Want to be listed here? Open a PR or issue.

---

## Publish your own skill

Add a `CLAUDE.md` to any public GitHub repo:

```markdown
# My Team's Coding Standards

When working in any project:
- Use TypeScript strict mode
- Write tests before implementing (TDD)
- Prefer composition over inheritance

When creating a new module, scaffold:
- A `.ts` file with the implementation
- A co-located `.test.ts` file
- An export in the nearest `index.ts`
```

That's the whole contract. Push it, share the path:

```
/install your-github-username/your-repo
```

No registration. No approval. No waiting.

---

## How it works

1. **`/install @name`** expands to `anthropics/skills/skills/name` — shorthand for all official skills.
2. **`/install author/repo`** fetches `CLAUDE.md` from the repo root (`main`/`master`, root and `.claude/`).
3. **`/install author/repo/path/to/skill`** fetches from a subdirectory — tries `SKILL.md` first, then `CLAUDE.md`. YAML frontmatter is stripped automatically.
4. Content is wrapped in tagged delimiters and appended to `~/.claude/CLAUDE.md`:
   ```
   <!-- skill: anthropics/skills/skills/frontend-design -->
   ...skill content...
   <!-- /skill: anthropics/skills/skills/frontend-design -->
   ```
5. **Deduplication** — installing the same skill twice is a no-op.
6. **`/install update`** re-fetches and replaces skill blocks in place, leaving everything else untouched.
7. **`/install remove`** surgically removes the tagged block.

---

## License

MIT
