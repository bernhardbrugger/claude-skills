# Git Conventions

When writing commit messages:
- Use conventional commit format: `type(scope): description`
  - **feat**: new feature
  - **fix**: bug fix
  - **docs**: documentation only
  - **refactor**: code restructure with no behavior change
  - **test**: adding or correcting tests
  - **chore**: maintenance, dependencies, build
- Keep the subject line under 72 characters
- Write in imperative mood — "Add feature" not "Added feature"
- Reference issue numbers when relevant: `fix(auth): handle token expiry (#42)`
- Never use `--no-verify` to skip hooks unless the user explicitly asks

When writing pull request titles and descriptions:
- Title: same conventional commit format as above, under 72 characters
- Description: lead with the WHY, not just what changed
- Call out breaking changes explicitly
- Include a brief test plan or verification steps

When reviewing a commit message or PR title, flag anything that:
- Doesn't follow conventional commit format
- Exceeds 72 characters
- Uses past tense instead of imperative mood
- Is vague ("fix bug", "update stuff", "changes")
