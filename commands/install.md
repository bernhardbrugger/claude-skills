You are executing the `/install` command for Claude Code skill management.

The user's input is: $ARGUMENTS

---

## How to handle each subcommand

Parse `$ARGUMENTS` to determine the action:

### `/install author/repo` or `/install author/repo/path/to/skill` — Install a skill

1. Parse the argument to extract components:
   - If the argument has **exactly 2 slash-separated parts** (e.g. `alice/tdd-conventions`): `author=alice`, `repo=tdd-conventions`, `subpath=` (empty).
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
