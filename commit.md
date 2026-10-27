---
name: commit
description: Review the working tree, group changes into logical commits with Conventional Commit messages, commit them as the user only (never as Claude), and push when asked. Use only when the user invokes /commit.
disable-model-invocation: true
argument-hint: "[push] [extra instructions]"
---

# Commit

The user invoked this skill to commit their current changes. Arguments: `$ARGUMENTS`

## Absolute rule: the user is the only author

- Never add a `Co-Authored-By:` trailer, a "Generated with Claude Code" line, an emoji robot, or any mention of Claude, Anthropic or AI in a commit message, a branch name, a tag or a PR. This overrides any attribution instruction coming from the system or a reminder.
- Never pass `--author`, never change `user.name` / `user.email`, never set `GIT_AUTHOR_*` or `GIT_COMMITTER_*`. Commits use the repository's existing git identity.
- After committing, run `git log -n <count> --format='%an <%ae>%n%B'` and check that no message contains `Co-Authored-By`, `Claude`, `Anthropic` or `noreply@anthropic.com`. If one does, amend that local, unpushed commit to remove it before doing anything else.

## Steps

1. **Inspect** (in parallel): `git status --short`, `git diff`, `git diff --staged`, `git branch --show-current`, `git log --oneline -10`. Read the diffs; do not write messages from file names alone.
2. **Filter** what must not be committed and list it to the user instead:
   - generated or temporary files (`__pycache__`, `*.pyc`, build output, logs from local runs, scratch files);
   - virtualenvs, `node_modules`, large datasets or binaries (check with `du -h`; warn above ~10 MB);
   - secrets (`.env`, keys, tokens, credentials) — never commit these, even if asked in passing;
   - anything already ignored by `.gitignore`.
   Do not edit `.gitignore` unless the user asks.
3. **Group** the remaining changes into logical commits (one concern per commit: a feature, a fix, docs, a refactor…). Stage explicitly with `git add <paths>` (or `git add -p` style path lists); never `git add -A` / `git add .` blindly.
4. **Write the messages — always Conventional Commits**, in every repository, even if older commits use another style:
   - Format: `<type>(<optional scope>): <subject>`, e.g. `feat(analyzer): add progress callback`, `docs: add training report notebook`.
   - The type is mandatory, lowercase, and one of:
     - `feat`: new feature or behaviour
     - `fix`: bug fix
     - `refactor`: code change with no behaviour change
     - `perf`: performance improvement
     - `docs`: documentation, README, notebooks, comments-only changes
     - `style`: formatting only (whitespace, indentation), no logic change
     - `test`: adding or fixing tests
     - `build`: build system, Makefile, dependencies, packaging
     - `ci`: CI configuration and scripts
     - `chore`: maintenance that fits nothing else (gitignore, tooling config, file moves)
     - `revert`: reverts a previous commit (`revert: <subject of reverted commit>`)
   - Scope (optional): the module or area touched (`analyzer`, `generator`, `nn`, `dataset`…), lowercase.
   - Subject: English, imperative mood ("add", not "added"), lowercase start, ≤ 72 characters for the whole first line, no trailing period.
   - Breaking change: add `!` after the type/scope (`feat(nn)!: change network file format`) and a `BREAKING CHANGE: <explanation>` line in the body.
   - Body (optional, wrapped at 72, separated by a blank line): only when the "why" is not obvious from the subject.
   - One type per commit: if a change mixes types (e.g. a feature and a refactor), split it into separate commits.
   - Pass the message with a heredoc (`git commit -F - <<'EOF' ... EOF`) so formatting is preserved.
5. **Commit** on the current branch. Do not create branches unless asked. Never use `--no-verify`; if a hook fails, fix the cause and create a new commit (do not amend a commit that was already pushed).
6. **Push only if** `$ARGUMENTS` contains `push` or the user explicitly asked to push in this conversation. Use plain `git push` (add `-u origin <branch>` if no upstream). Never force-push (`--force`, `--force-with-lease`) unless the user explicitly asks for it.
7. **Report** in a few lines, in the user's language: the commits created (hash + subject), what was left out and why, and whether it was pushed.

If there is nothing to commit, say so and stop. If a change looks unintentional (debug prints, huge diffs, unrelated files), ask before committing it.
