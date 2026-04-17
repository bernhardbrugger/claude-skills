You are executing the `/install` command for Claude Code skill management.

The user's input is: $ARGUMENTS

---

## Step 0 — Resolve `+name` shorthand

Before parsing the subcommand, check if any argument (other than `list`, `search`, `update`, `remove`) starts with `+`. Only expand it if the part after `+` is a **simple identifier**: letters, digits, hyphens, and underscores only — no spaces, colons, slashes, dots, or URL-like patterns.

- `+name` → `anthropics/skills/skills/name`

Examples:
- `/install +frontend-design` → install `anthropics/skills/skills/frontend-design`
- `/install update +pdf` → update `anthropics/skills/skills/pdf`
- `/install remove +mcp-builder` → remove `anthropics/skills/skills/mcp-builder`

If the `+`-prefixed token does **not** match a simple identifier, leave it unchanged and continue parsing.

Apply this expansion before any further parsing below.

---

## How to handle each subcommand

Parse `$ARGUMENTS` to determine the action:

---

### `/install author/repo` or `/install author/repo/path/to/skill` — Install a skill

1. Parse the argument to extract components:
   - If the argument has **exactly 2 slash-separated parts** (e.g. `author/repo`): `author=author`, `repo=repo`, `subpath=` (empty).
   - If the argument has **3 or more slash-separated parts** (e.g. `anthropics/skills/skills/frontend-design`): `author=anthropics`, `repo=skills`, `subpath=skills/frontend-design`.

2. Try fetching the skill content from these URLs **in order**, stopping at the first success.

   **If subpath is empty** (simple `author/repo` format):
   - `https://raw.githubusercontent.com/{author}/{repo}/main/CLAUDE.md`
   - `https://raw.githubusercontent.com/{author}/{repo}/main/.claude/CLAUDE.md`
   - `https://raw.githubusercontent.com/{author}/{repo}/master/CLAUDE.md`
   - `https://raw.githubusercontent.com/{author}/{repo}/master/.claude/CLAUDE.md`

   **If subpath is set** (extended `author/repo/path` format):
   - `https://raw.githubusercontent.com/{author}/{repo}/main/{subpath}/SKILL.md`
   - `https://raw.githubusercontent.com/{author}/{repo}/main/{subpath}/CLAUDE.md`
   - `https://raw.githubusercontent.com/{author}/{repo}/master/{subpath}/SKILL.md`
   - `https://raw.githubusercontent.com/{author}/{repo}/master/{subpath}/CLAUDE.md`

   Use the `WebFetch` tool for each attempt. A 404 or error means try the next path.

3. If **no content is found** in any location, print:
   ```
   ✗ No CLAUDE.md or SKILL.md found in {original argument}
   ```
   and stop.

4. **Strip YAML frontmatter**: If the fetched content starts with a line containing only `---`, remove everything from that opening `---` up to and including the next line containing only `---`, plus any blank lines immediately following it. This cleans up the metadata header that official skill registries (like `anthropics/skills`) add to their files.

5. Read `~/.claude/CLAUDE.md` (create it as an empty file first if it doesn't exist).

6. **Duplicate check**: Search the existing `~/.claude/CLAUDE.md` content for a line containing `<!-- skill: {original argument} -->`. If found, print:
   ```
   ⚠ Skill already installed: {original argument}
   ```
   and stop.

7. Append the following block to `~/.claude/CLAUDE.md`:
   ```
   
   <!-- skill: {original argument} -->
   {fetched content, after frontmatter stripping}
   <!-- /skill: {original argument} -->
   ```

8. Print:
   ```
   ✓ Skill installed: {original argument}
   ```

---

### `/install info author/repo` or `/install info @name` — Preview a skill before installing

1. Resolve `@name` shorthand if used (same rule as Step 0).
2. Parse the argument using the same author/repo/subpath logic as the install step.
3. Fetch the skill content using the same URL priority order as install. Do **not** write anything to disk.
4. If no content is found, print:
   ```
   ✗ No CLAUDE.md or SKILL.md found in {identifier}
   ```
   and stop.
5. Strip YAML frontmatter. If frontmatter was present and contained a `description:` field, extract it.
6. Print a preview:
   ```
   Skill: {identifier}
   Source: {URL that succeeded}

   {description line if available, otherwise first 15 lines of content}

   Install with: /install {identifier}
   ```
7. Also note if the skill is already installed:
   ```
   ⚠ Already installed — use /install update {identifier} to refresh.
   ```

---

### `/install list` — List installed skills

1. Read `~/.claude/CLAUDE.md`. If it doesn't exist, print:
   ```
   No skills installed yet. Use /install author/repo to add one.
   ```
   and stop.
2. Scan the file for all lines matching `<!-- skill: ... -->` (opening tags only).
3. Extract the identifier from each tag and print them as a numbered list:
   ```
   Installed skills:
     1. author/repo-one
     2. author/repo/path/two
   ```
   If none are found, print:
   ```
   No skills installed yet. Use /install author/repo to add one.
   ```

---

### `/install search [query]` — Browse available skills

Display skills from the official `anthropics/skills` registry. If a `query` is provided, filter by skills whose name or description contains the query (case-insensitive). If no query, show all.

Format output as:

```
Official skills  (install with /install +<name>)

  +algorithmic-art       Generative art with p5.js — flow fields, particle systems, seeded randomness
  +brand-guidelines      Apply Anthropic's brand colors and typography to any artifact
  +canvas-design         Create original visual designs as PNG/PDF (posters, art, layouts)
  +claude-api            Build and optimize apps with the Claude API / Anthropic SDK
  +doc-coauthoring       Structured workflow for writing docs, proposals, and specs
  +docx                  Create, read, and edit Word documents (.docx)
  +frontend-design       Production-grade UI — web components, pages, apps with distinctive aesthetics
  +internal-comms        Write internal comms: status reports, newsletters, incident reports
  +mcp-builder           Build MCP (Model Context Protocol) servers in Python or TypeScript
  +pdf                   Read, create, merge, split, rotate, watermark, and OCR PDFs
  +pptx                  Create, edit, and parse PowerPoint presentations (.pptx)
  +skill-creator         Create and optimize Claude Code skills, run evals
  +slack-gif-creator     Generate animated GIFs optimized for Slack
  +theme-factory         Style artifacts with 10 preset themes or generate a custom one
  +web-artifacts-builder Build complex claude.ai HTML artifacts with React, Tailwind, shadcn/ui
  +webapp-testing        Test local web apps with Playwright — screenshots, logs, UI verification
  +xlsx                  Create, read, edit, and convert spreadsheets (.xlsx, .csv, .tsv)

Community skills  →  https://github.com/topics/claude-skills
```

If a query is provided and nothing matches, print:
```
No skills found matching "{query}".
Try /install search to see all available skills.
```

---

### `/install update` or `/install update author/repo[/path/to/skill]` — Update skills

1. Parse the argument:
   - If **no argument**: update ALL installed skills.
   - If an **identifier is given**: update only that specific skill.

2. For each skill to update:
   a. Extract its full identifier from the `<!-- skill: ... -->` tag in `~/.claude/CLAUDE.md`.
   b. Re-fetch the content using the same URL logic as the install step (using the identifier to determine author/repo/subpath and trying all path variants in order).
   c. Strip YAML frontmatter from the fetched content (same rule as install).
   d. Compare the fetched content to the existing block content (between the open and close tags).
   e. If content differs: replace the old block with the new content. Print:
      ```
      ✓ Updated: {identifier}
      ```
   f. If content is identical: print:
      ```
      ⚠ No changes: {identifier}
      ```
   g. If the fetch fails (no CLAUDE.md or SKILL.md found): print:
      ```
      ✗ Could not fetch: {identifier}
      ```
      and leave the existing block untouched.

3. If updating all skills and none are installed, print:
   ```
   No skills installed yet. Use /install author/repo to add one.
   ```

---

### `/install remove author/repo` or `/install remove author/repo/path/to/skill` — Remove a skill

1. Parse the argument after `remove` as the full skill identifier (may contain more than 2 slash-separated parts).
2. Read `~/.claude/CLAUDE.md`. If the file doesn't exist or the skill tag is not found, print:
   ```
   ✗ Skill not installed: {identifier}
   ```
   and stop.
3. Remove the entire block from `<!-- skill: {identifier} -->` through `<!-- /skill: {identifier} -->` (inclusive, including the blank line before the opening tag).
4. Write the updated content back to `~/.claude/CLAUDE.md`.
5. Print:
   ```
   ✓ Skill removed: {identifier}
   ```

---

## Important rules

- Always use `~/.claude/CLAUDE.md` as the target file (expand `~` to the actual home directory).
- Never modify anything outside of `~/.claude/CLAUDE.md` during install/remove.
- Be precise with the skill block delimiters — they are what makes deduplication and removal reliable.
- When fetching URLs, treat any non-200 HTTP response or network error as "not found" and try the next path.
