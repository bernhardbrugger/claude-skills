You are executing the `/install` command for Claude Code skill management.

The user's input is: $ARGUMENTS

---

## How to handle each subcommand

Parse `$ARGUMENTS` to determine the action:

### `/install author/repo` — Install a skill

1. Parse the argument as `author/repo` (e.g. `anthropics/frontend-design`).
2. Try fetching the CLAUDE.md from these URLs **in order**, stopping at the first success:
   - `https://raw.githubusercontent.com/{author}/{repo}/main/CLAUDE.md`
   - `https://raw.githubusercontent.com/{author}/{repo}/main/.claude/CLAUDE.md`
   - `https://raw.githubusercontent.com/{author}/{repo}/master/CLAUDE.md`
   - `https://raw.githubusercontent.com/{author}/{repo}/master/.claude/CLAUDE.md`
   Use the `WebFetch` tool for each attempt. A 404 or error means try the next path.
3. If **no CLAUDE.md is found** in any location, print:
   ```
   ✗ No CLAUDE.md found in author/repo
   ```
   and stop.
4. Read `~/.claude/CLAUDE.md` (create it as an empty file first if it doesn't exist).
5. **Duplicate check**: Search the existing `~/.claude/CLAUDE.md` content for a line containing `<!-- skill: author/repo -->`. If found, print:
   ```
   ⚠ Skill already installed: author/repo
   ```
   and stop.
6. Append the following block to `~/.claude/CLAUDE.md`:
   ```
   
   <!-- skill: author/repo -->
   {fetched CLAUDE.md content}
   <!-- /skill: author/repo -->
   ```
7. Print:
   ```
   ✓ Skill installed: author/repo
   ```

---

### `/install list` — List installed skills

1. Read `~/.claude/CLAUDE.md`. If it doesn't exist, print:
   ```
   No skills installed yet. Use /install author/repo to add one.
   ```
   and stop.
2. Scan the file for all lines matching `<!-- skill: ... -->` (opening tags only).
3. Extract the `author/repo` from each tag and print them as a numbered list:
   ```
   Installed skills:
     1. author/repo-one
     2. author/repo-two
   ```
   If none are found, print:
   ```
   No skills installed yet. Use /install author/repo to add one.
   ```

---

### `/install remove author/repo` — Remove a skill

1. Parse the argument after `remove` as `author/repo`.
2. Read `~/.claude/CLAUDE.md`. If the file doesn't exist or the skill tag is not found, print:
   ```
   ✗ Skill not installed: author/repo
   ```
   and stop.
3. Remove the entire block from `<!-- skill: author/repo -->` through `<!-- /skill: author/repo -->` (inclusive, including the blank line before the opening tag).
4. Write the updated content back to `~/.claude/CLAUDE.md`.
5. Print:
   ```
   ✓ Skill removed: author/repo
   ```

---

## Important rules

- Always use `~/.claude/CLAUDE.md` as the target file (expand `~` to the actual home directory).
- Never modify anything outside of `~/.claude/CLAUDE.md` during install/remove.
- Be precise with the skill block delimiters — they are what makes deduplication and removal reliable.
- When fetching URLs, treat any non-200 HTTP response or network error as "not found" and try the next path.
