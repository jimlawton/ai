# git-spice CLI Reference

Complete reference for all `git-spice` commands, flags, shorthands, and configuration options.

> Sources: [CLI Reference](https://abhinav.github.io/git-spice/cli/reference/), [Shorthands](https://abhinav.github.io/git-spice/cli/shorthand/), [Configuration](https://abhinav.github.io/git-spice/cli/config/)

---

## Command Structure

All commands follow: `git-spice <group> <action> [flags]`

Shorthands use initials: `git-spice branch create` -> `git-spice bc`, `git-spice stack submit` -> `git-spice ss`

## Global Flags

| Flag | Description |
|------|-------------|
| `-h, --help` | Show help for command |
| `--version` | Print version and exit |
| `-v, --verbose` | Enable verbose output |
| `-C, --dir=DIR` | Change directory before running |
| `--[no-]prompt` | Enable/disable interactive prompts |

---

## Navigation Commands

| Command     | Description                                     |
| ----------- | ----------------------------------------------- |
| `git-spice up`     | Check out the branch above current in the stack |
| `git-spice down`   | Check out the branch below current in the stack |
| `git-spice top`    | Check out the topmost branch in the stack       |
| `git-spice bottom` | Check out the bottommost branch (above trunk)   |
| `git-spice trunk`  | Check out the trunk branch                      |

All navigation commands support `-n` / `--dry-run` to print target branch name without checking out.

---

## git-spice repo (Repository Commands)

### git-spice repo init (git-spice ri)

Initialize git-spice for a repository. Sets trunk branch and remote.

**Flags:**

- `--trunk <branch>`: Specify trunk branch (default: prompted)
- `--remote <remote>`: Specify remote (default: prompted)
- `--reset`: Hard reset git-spice metadata (not git history)

### git-spice repo sync (git-spice rs)

Pull latest changes from remote, delete merged branches, prepare for restacking.

- Skips branches checked out in other worktrees
- Detects squash-merged PRs and cleans up
- Add `--restack` to also restack remaining branches

### git-spice repo restack (git-spice rr)

Restack all tracked branches onto their bases. Full repo rebase for tracked branches only.

---

## git-spice branch (Branch Commands)

### git-spice branch create (git-spice bc)

Create a new branch stacked on current branch.

**Flags:**

| Flag | Description |
|------|-------------|
| `-m, --message=MSG` | Commit message for initial commit |
| `-a, --all` | Stage all modified/deleted files |
| `-t, --target=BRANCH` | Base branch (default: current) |
| `--insert` | Restack upstack onto new branch |
| `--below` | Create below target branch |
| `--no-commit` | Create branch without commit |
| `--no-verify` | Skip pre-commit hooks |
| `--signoff` | Add Signed-off-by trailer |

### git-spice branch checkout (git-spice bco)

Check out a tracked branch. Without arguments, shows interactive fuzzy-searchable tree.

**Flags:**

- `-u` / `--untracked`: Also show untracked branches in the prompt

### git-spice branch track (git-spice bt)

Track an existing (untracked) branch with git-spice. Use `--base <branch>` to specify the parent.

### git-spice branch untrack (git-spice but)

Stop tracking a branch without deleting it.

### git-spice branch delete (git-spice bd)

Delete one or more branches. Rebases upstack branches onto next available downstack.

**Flags:**

- `--force` / `-f`: Force delete even if branch has unmerged changes

### git-spice branch rename (git-spice brn)

Rename a tracked branch. `git-spice brn [old] [new]`

### git-spice branch onto (git-spice bo)

Move only the current branch to a different base. Upstack stays on original base.

**Flags:**

- `--branch` / `-b`: Target a different branch (not the current one)

### git-spice branch restack (git-spice br)

Restack a single branch on top of its base.

### git-spice branch submit (git-spice bs)

Submit (create/update) a PR for the current branch only.

### git-spice branch split (git-spice bsp)

Interactively split a branch at commit boundaries into multiple branches.

### git-spice branch edit (git-spice be)

Interactive rebase of the current branch. Allows squash, fixup, reorder commits.

### git-spice branch squash (git-spice bsq)

Squash all commits in branch into one.

### git-spice branch fold (git-spice bfo)

Merge branch into its base (squash-merges branch commits into base).

---

## git-spice commit (Commit Commands)

### git-spice commit create (git-spice cc)

Commit staged changes and restack all upstack branches automatically. Equivalent to `git commit` + `git-spice upstack restack`.

**Flags:**

| Flag | Description |
|------|-------------|
| `-m, --message=MSG` | Commit message |
| `-a, --all` | Stage all tracked changes before committing |
| `--no-verify` | Skip hooks |
| `--signoff` | Add Signed-off-by |

### git-spice commit amend (git-spice ca)

Amend the last commit and restack all upstack branches automatically.

**Flags:**

| Flag | Description |
|------|-------------|
| `-m, --message=MSG` | New commit message |
| `-a, --all` | Stage all changes |
| `--no-edit` | Keep existing message |
| `--no-verify` | Skip hooks |

### git-spice commit split (git-spice csp)

Interactively split the last commit into two commits and restack upstack.

### git-spice commit fixup (git-spice cf)

Create a fixup commit for a commit below.

### git-spice commit pick (git-spice cp)

Cherry-pick a commit into the current branch.

---

## git-spice stack (Stack Commands)

### git-spice stack submit (git-spice ss)

Submit all branches in the current stack as PRs. Creates/updates linked PRs with navigation comments.

### git-spice stack restack (git-spice sr)

Restack all branches in the current stack.

### git-spice stack edit (git-spice se)

Edit the entire stack interactively. Reorder, move branches between bases.

### git-spice stack delete (git-spice sd)

Delete all branches in current stack. Requires `--force`.

---

## git-spice upstack (Upstack Commands)

### git-spice upstack submit (git-spice uss)

Submit the current branch and all upstack branches as PRs.

### git-spice upstack restack (git-spice ur)

Restack all branches upstack from current.

### git-spice upstack onto (git-spice uo)

Move the current branch AND its entire upstack to a new base. Unlike `git-spice bo` which moves only the current branch.

### git-spice upstack delete (git-spice usd)

Delete all branches above current branch.

---

## git-spice downstack (Downstack Commands)

### git-spice downstack submit (git-spice dss)

Submit the current branch and all downstack branches as PRs.

### git-spice downstack track (git-spice dst)

Traverse commit graph downward, tracking untracked branches along the way. Useful for onboarding existing branch chains.

### git-spice downstack edit (git-spice dse)

Edit the downstack interactively.

---

## git-spice log (Log/Visualization Commands)

### git-spice log short (git-spice ls)

Short view of the stack showing branch names, relationships, and CR status.

**Flags:**

- `--all` / `-a`: Show all stacks, not just the current one
- `--json`: Output as JSON (one JSON object per line)

### git-spice log long (git-spice ll)

Detailed view including individual commits for each branch.

**Flags:**

- `--all` / `-a`: Show all stacks
- `--json`: Output as JSON

---

## git-spice rebase (Rebase Commands)

### git-spice rebase continue (git-spice rbc)

Continue a git-spice operation interrupted by a rebase conflict. Run after resolving conflicts and staging files.

### git-spice rebase abort (git-spice rba)

Abort an ongoing git-spice rebase operation.

---

## git-spice auth (Authentication Commands)

### git-spice auth login

Log in to GitHub or GitLab. Supports OAuth, GitHub App, and PAT authentication.

### git-spice auth status

Check current authentication status.

### git-spice auth logout

Log out and remove stored credentials.

---

## git-spice shell (Shell Commands)

### git-spice shell completion [shell]

Generate shell completion script. Supported shells: bash, zsh, fish.

```bash
eval "$(git-spice shell completion bash)"   # Bash
eval "$(git-spice shell completion zsh)"    # Zsh
eval "$(git-spice shell completion fish)"   # Fish
eval "$(git-spice shell completion)"        # Auto-detect
```

---

## Submit Command Flags (All Submit Commands)

| Flag                     | Description                                      |
| ------------------------ | ------------------------------------------------ |
| `--fill`                 | Populate title and body from commit messages     |
| `--draft` / `--no-draft` | Mark CR as draft or not                          |
| `--dry-run`              | Print what would be submitted without submitting |
| `--no-verify`            | Bypass pre-push hooks                            |
| `--update-only`          | Only update existing CRs, don't create new ones  |
| `--nav-comment`          | Control navigation comment behavior              |
| `--web` / `-w`           | Open browser with submitted CR                   |
| `--reviewer` / `-r`      | Assign reviewers                                 |
| `--assignee` / `-a`      | Assign assignees                                 |
| `--label` / `-l`         | Add labels                                       |
| `--template`             | Use a specific PR template                       |

---

## Complete Shorthand Reference

| Shorthand | Full Command          | Category   |
| --------- | --------------------- | ---------- |
| `git-spice ri`   | `git-spice repo init`        | Repository |
| `git-spice rs`   | `git-spice repo sync`        | Repository |
| `git-spice rr`   | `git-spice repo restack`     | Repository |
| `git-spice bc`   | `git-spice branch create`    | Branch     |
| `git-spice bco`  | `git-spice branch checkout`  | Branch     |
| `git-spice bd`   | `git-spice branch delete`    | Branch     |
| `git-spice brn`  | `git-spice branch rename`    | Branch     |
| `git-spice bt`   | `git-spice branch track`     | Branch     |
| `git-spice but`  | `git-spice branch untrack`   | Branch     |
| `git-spice bo`   | `git-spice branch onto`      | Branch     |
| `git-spice br`   | `git-spice branch restack`   | Branch     |
| `git-spice bsp`  | `git-spice branch split`     | Branch     |
| `git-spice bsq`  | `git-spice branch squash`    | Branch     |
| `git-spice bfo`  | `git-spice branch fold`      | Branch     |
| `git-spice be`   | `git-spice branch edit`      | Branch     |
| `git-spice bs`   | `git-spice branch submit`    | Branch     |
| `git-spice cc`   | `git-spice commit create`    | Commit     |
| `git-spice ca`   | `git-spice commit amend`     | Commit     |
| `git-spice csp`  | `git-spice commit split`     | Commit     |
| `git-spice cf`   | `git-spice commit fixup`     | Commit     |
| `git-spice cp`   | `git-spice commit pick`      | Commit     |
| `git-spice ss`   | `git-spice stack submit`     | Stack      |
| `git-spice sr`   | `git-spice stack restack`    | Stack      |
| `git-spice se`   | `git-spice stack edit`       | Stack      |
| `git-spice sd`   | `git-spice stack delete`     | Stack      |
| `git-spice uss`  | `git-spice upstack submit`   | Upstack    |
| `git-spice ur`   | `git-spice upstack restack`  | Upstack    |
| `git-spice uo`   | `git-spice upstack onto`     | Upstack    |
| `git-spice usd`  | `git-spice upstack delete`   | Upstack    |
| `git-spice dss`  | `git-spice downstack submit` | Downstack  |
| `git-spice dse`  | `git-spice downstack edit`   | Downstack  |
| `git-spice dst`  | `git-spice downstack track`  | Downstack  |
| `git-spice ls`   | `git-spice log short`        | Log        |
| `git-spice ll`   | `git-spice log long`         | Log        |
| `git-spice rbc`  | `git-spice rebase continue`  | Rebase     |
| `git-spice rba`  | `git-spice rebase abort`     | Rebase     |

---

## Configuration Reference

All configuration uses `git config`. Supports `--global`, `--local`, `--worktree`, and `--system` scopes.

### Branch Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `spice.branchCreate.prefix` | Prefix for branch names created with `git-spice bc` | (none) |
| `spice.branchCreate.commit` | Auto-commit staged changes on branch create | `true` |
| `spice.branchCreate.generatedBranchNameLimit` | Max length of auto-generated branch names | (unlimited) |
| `spice.branchCheckout.showUntracked` | Show untracked branches in `git-spice bco` | `false` |
| `spice.branchCheckout.trackUntrackedPrompt` | Prompt to track untracked branches during checkout | `true` |
| `spice.commit.signoff` | Add Signed-off-by by default | `false` |

### Submit Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `spice.submit.publish` | Create CRs on forge (false = push only) | `true` |
| `spice.submit.draft` | Default draft status for new CRs | `false` |
| `spice.submit.web` | Open browser after submitting | `false` |
| `spice.submit.updateOnly` | Default to `--update-only` | `false` |
| `spice.submit.reviewers` | Default reviewers (comma-separated) | (none) |
| `spice.submit.reviewers.addWhen` | When to add reviewers | `always` |
| `spice.submit.assignees` | Default assignees | (none) |
| `spice.submit.label` | Default labels | (none) |
| `spice.submit.template` | Default PR template | (none) |
| `spice.submit.navigationComment` | Navigation comment behavior | `true` |
| `spice.submit.navigationCommentSync` | Sync nav comments on update | `true` |
| `spice.submit.noVerify` | Default to `--no-verify` | `false` |

### Log Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `spice.log.all` | Default to `--all` for `git-spice ls` and `git-spice ll` | `false` |
| `spice.log.crFormat` | Format for CR information in log output | (default) |
| `spice.log.crStatus` | Show CR status in log | `true` |

### Sync Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `spice.repoSync.closedChanges` | How to handle closed (unmerged) CRs during sync | (prompt) |

### Forge Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `spice.forge.github.url` | URL for GitHub Enterprise | (github.com) |
| `spice.forge.github.apiUrl` | API URL for GitHub Enterprise | (github.com) |
| `spice.forge.gitlab.url` | URL for self-hosted GitLab | (gitlab.com) |
| `spice.forge.gitlab.apiURL` | API URL for self-hosted GitLab | (gitlab.com) |

---

## Limitations

- **Squash-merge reconciliation**: After squash-merge, all upstack branches need restacking (commit hashes change)
- **Git 2.38+ required**: Older versions may have partial functionality
- **Interactive prompts**: Commands like `git-spice bco`, `git-spice bsp`, `git-spice se`, `git-spice csp` use interactive prompts that do not work in non-TTY environments
- **Single remote**: git-spice works with one remote per repository
- **Single trunk**: Only one trunk branch per repository; no per-stack trunk overrides
