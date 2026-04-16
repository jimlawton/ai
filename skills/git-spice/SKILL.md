---
name: git-spice
description: >-
  ALWAYS use this skill when the user mentions git-spice, stacked branches,
  stacked PRs, stacked pull requests, stacked merge requests, branch stacks,
  or managing multiple dependent branches. Also use when the user asks about
  restacking, submitting a stack, syncing after merges, navigating a branch
  stack, splitting a branch into stacked PRs, coordinating worktrees with
  stacked branches, or stacking on top of someone else's PR. Use even if
  the user doesn't say "git-spice" explicitly -- any mention of branches
  stacked on each other, dependent PRs, or pushing multiple related branches
  with correct base targets should trigger this skill. This skill provides
  the complete git-spice CLI reference, workflow patterns, safety rules
  (never git push or git rebase on tracked branches), worktree coordination
  for multi-agent setups, and squash-merge reconciliation. If in doubt
  whether git-spice is relevant, consult this skill anyway -- it is better
  to check and not need it than to give incorrect git-spice advice.
---

# git-spice -- Stacked Branch Management

git-spice is a CLI tool for managing stacked Git branches. It tracks
parent-child relationships between branches, rebases dependent branches when a
base changes, and creates/updates PRs across entire stacks with a single command.

## Command Name: `git-spice`

The CLI command is **`git-spice`**, not `gs` or `gsp`. Always use `git-spice` in
all commands, examples, and instructions you provide. While some installations
alias it to `gs`, the canonical command name is `git-spice` and that is what
should appear in every code block, inline reference, and explanation.

Examples of correct usage:
- `git-spice repo init` (not `gs repo init`)
- `git-spice branch create` / `git-spice bc` (not `gs bc`)
- `git-spice stack submit` / `git-spice ss` (not `gs ss`)

## Prerequisites

- **Git 2.38+** installed
- **git-spice** installed (`brew install git-spice` or `go install go.abhg.dev/gs@latest`)
- Repository initialized with `git-spice repo init`
- Authenticated with `git-spice auth login` (for PR operations)

Check installation: `git-spice --version`. Check auth: `git-spice auth status`.

## Key Concepts

- **Trunk**: The main/default branch (e.g., `main`). Root of all stacks.
- **Stack**: A chain of branches where each builds on the one below.
- **Upstack**: Branches above the current branch (descendants).
- **Downstack**: Branches below the current branch, excluding trunk (ancestors).
- **Restacking**: Rebasing tracked branches onto updated bases to keep the stack linear.
- **Tracked branches**: git-spice only manages branches it tracks. Branches created with `git-spice bc` are auto-tracked. Track existing branches with `git-spice bt`.
- **Change Request (CR)**: git-spice's term for PRs/MRs. Each tracked branch corresponds to one CR on GitHub/GitLab.

```
trunk (main)
  +-- feature-a       <- bottom of stack
       +-- feature-b
            +-- feature-c  <- top of stack
```

When on `feature-b`: upstack = `feature-c`, downstack = `feature-a`.

## Critical Rules

### Use git-spice commands instead of git equivalents

When git-spice is initialized in a repository, prefer `git-spice` commands:

- **Use `git-spice commit create`** (`git-spice cc`) instead of `git commit` -- auto-restacks upstack
- **Use `git-spice commit amend`** (`git-spice ca`) instead of `git commit --amend` -- auto-restacks upstack
- **Use `git-spice branch create`** (`git-spice bc`) instead of `git checkout -b` -- tracks the branch
- **Use `git-spice branch delete`** (`git-spice bd`) instead of `git branch -d` -- rebases upstack onto next downstack

### NEVER use `git push` on tracked branches

**Always use `git-spice stack submit` (`git-spice ss`) or `git-spice branch submit` (`git-spice bs`) to push.**

`git push` bypasses git-spice's tracking and breaks PR linkage -- branches pushed with `git push` will not have their PRs tracked, and stack-wide operations like `git-spice ss` will skip them. If this happens accidentally, run `git-spice ss` or `git-spice bs` to repair the linkage.

### NEVER use `git rebase` or `git push --force` on tracked branches

These bypass git-spice's dependency tracking and leave the stack in an inconsistent state. Use `git-spice upstack restack` / `git-spice stack restack` instead. If the stack gets out of sync, run `git-spice stack restack` to reconcile.

### Never run destructive commands without explicit user confirmation

Commands like `git-spice stack delete`, `git-spice upstack delete`, or `git-spice repo init --reset` require `--force` and explicit user approval.

## Essential Command Quick Reference

| Action                    | Command                     | Shorthand |
| ------------------------- | --------------------------- | --------- |
| Initialize repo           | `git-spice repo init`              | `git-spice ri`   |
| Sync with remote          | `git-spice repo sync`              | `git-spice rs`   |
| Restack all tracked       | `git-spice repo restack`           | `git-spice rr`   |
| Create stacked branch     | `git-spice branch create <name>`   | `git-spice bc`   |
| Track existing branch     | `git-spice branch track`           | `git-spice bt`   |
| Checkout (fuzzy search)   | `git-spice branch checkout`        | `git-spice bco`  |
| Delete branch             | `git-spice branch delete`          | `git-spice bd`   |
| Move branch to new base   | `git-spice branch onto <base>`     | `git-spice bo`   |
| Move branch + upstack     | `git-spice upstack onto <base>`    | `git-spice uo`   |
| Rename branch             | `git-spice branch rename`          | `git-spice brn`  |
| Split branch              | `git-spice branch split`           | `git-spice bsp`  |
| Squash branch             | `git-spice branch squash`          | `git-spice bsq`  |
| Fold into base            | `git-spice branch fold`            | `git-spice bfo`  |
| Edit branch (rebase)      | `git-spice branch edit`            | `git-spice be`   |
| Commit + auto-restack     | `git-spice commit create -m "msg"` | `git-spice cc`   |
| Amend + auto-restack      | `git-spice commit amend`           | `git-spice ca`   |
| Fixup commit              | `git-spice commit fixup`           | `git-spice cf`   |
| Cherry-pick into branch   | `git-spice commit pick`            | `git-spice cp`   |
| Split last commit         | `git-spice commit split`           | `git-spice csp`  |
| Submit current branch     | `git-spice branch submit`          | `git-spice bs`   |
| Submit entire stack       | `git-spice stack submit`           | `git-spice ss`   |
| Submit upstack            | `git-spice upstack submit`         | `git-spice uss`  |
| Submit downstack          | `git-spice downstack submit`       | `git-spice dss`  |
| Restack current stack     | `git-spice stack restack`          | `git-spice sr`   |
| Restack upstack           | `git-spice upstack restack`        | `git-spice ur`   |
| Restack single branch     | `git-spice branch restack`         | `git-spice br`   |
| Edit stack order          | `git-spice stack edit`             | `git-spice se`   |
| Track downstack chain     | `git-spice downstack track`        | `git-spice dst`  |
| View stack (short)        | `git-spice log short`              | `git-spice ls`   |
| View stack (detailed)     | `git-spice log long`               | `git-spice ll`   |
| Navigate up               | `git-spice up`                     |           |
| Navigate down             | `git-spice down`                   |           |
| Jump to top               | `git-spice top`                    |           |
| Jump to bottom            | `git-spice bottom`                 |           |
| Return to trunk           | `git-spice trunk`                  |           |
| Continue after conflict   | `git-spice rebase continue`        | `git-spice rbc`  |
| Abort rebase              | `git-spice rebase abort`           | `git-spice rba`  |

Add `--all` to `git-spice ls` / `git-spice ll` to show all stacks. Add `--json` for machine-readable output.

## Submit Flags

These flags apply to all submit commands (`git-spice bs`, `git-spice ss`, `git-spice uss`, `git-spice dss`):

| Flag                     | Purpose                                     |
| ------------------------ | ------------------------------------------- |
| `--fill`                 | Populate PR title/body from commit messages |
| `--draft` / `--no-draft` | Set draft status                            |
| `--reviewer` / `-r`      | Assign reviewers                            |
| `--assignee` / `-a`      | Assign assignees                            |
| `--label` / `-l`         | Add labels                                  |
| `--web` / `-w`           | Open browser after submit                   |
| `--dry-run`              | Preview without submitting                  |
| `--update-only`          | Only update existing CRs, don't create new  |
| `--no-verify`            | Bypass pre-push hooks                       |
| `--template`             | Use a specific PR template                  |

## Core Workflows

### 1. Initialize and Authenticate (once per repo)

```bash
git-spice repo init --trunk main --remote origin
git-spice auth login
```

### 2. Create a Stack

```bash
git-spice trunk
# ... make changes, stage them ...
git-spice branch create feature-part-1 -m "Add part 1"    # git-spice bc
# ... make more changes, stage them ...
git-spice branch create feature-part-2 -m "Add part 2"    # git-spice bc
git-spice log short                                         # git-spice ls
```

### 3. Navigate the Stack

```bash
git-spice up        # Move up one branch
git-spice down      # Move down one branch
git-spice top       # Jump to top of stack
git-spice bottom    # Jump to bottom of stack (above trunk)
git-spice trunk     # Return to trunk
git-spice bco       # Interactive branch picker (fuzzy search)
```

### 4. Edit Mid-Stack and Restack

After modifying a lower branch, update everything above it:

```bash
git-spice down                                  # navigate to the branch
# ... edit files ...
git add <files>
git-spice commit create -m "fix: address feedback"  # git-spice cc (commits + auto-restacks upstack)
```

Or with a normal git commit followed by explicit restack:

```bash
git add <files>
git commit -m "fix: address feedback"
git-spice upstack restack                       # git-spice ur
```

### 5. Submit PRs

```bash
git-spice stack submit --fill              # git-spice ss (submit entire stack, auto-fill PR info)
git-spice stack submit --fill --draft      # submit as drafts
git-spice branch submit                    # git-spice bs (current branch only)
git-spice upstack submit                   # git-spice uss (current + descendants)
git-spice downstack submit                 # git-spice dss (current + ancestors)
git-spice stack submit --update-only       # update existing CRs only
```

### 6. Check for Remote Changes Before Submitting

`git-spice stack submit` and `git-spice branch submit` force-push branches, which can overwrite remote changes made by CI auto-fixes, collaborators, or bots.

```bash
git fetch origin
git log HEAD..origin/<branch-name> --oneline
# If remote-only commits exist:
git pull --rebase origin <branch-name>
git-spice stack restack    # git-spice sr (propagate to upstack)
git-spice stack submit     # git-spice ss
```

### 7. Sync After Merges

```bash
git-spice repo sync                # git-spice rs (pull trunk, delete merged branches)
git-spice stack restack            # git-spice sr (rebase remaining)
git-spice stack submit             # git-spice ss (update PRs)
```

Or combined: `git-spice repo sync --restack`

### 8. Handle Conflicts During Restack

1. Resolve conflicts in the affected files
2. Stage resolved files: `git add <resolved-files>`
3. Continue: `git-spice rebase continue` (`git-spice rbc`)
4. To abort instead: `git-spice rebase abort` (`git-spice rba`)

### 9. Adopt Existing Branches

Track a single existing branch:

```bash
git checkout feature-branch
git-spice branch track --base main    # git-spice bt
```

Track an existing chain (run from the topmost branch):

```bash
git checkout <top-branch>
git-spice downstack track             # git-spice dst
```

### 10. Insert a Branch Mid-Stack

```bash
# On feature-a (which has feature-b stacked on top)
git-spice branch create --insert hotfix -m "Fix critical bug"
# Creates: trunk -> feature-a -> hotfix -> feature-b
```

### 11. Move Branches

```bash
git-spice branch onto main            # git-spice bo (move only current branch)
git-spice upstack onto other-feature  # git-spice uo (move current + all upstack)
```

### 12. Stack Surgery

```bash
git-spice stack edit                  # git-spice se (reorder branches interactively)
git-spice branch split                # git-spice bsp (split branch at commit boundaries)
git-spice branch squash               # git-spice bsq (squash all commits in branch)
git-spice branch fold                 # git-spice bfo (merge branch into its base)
git-spice commit split                # git-spice csp (split last commit into two)
```

### 13. Multiple Independent Stacks

```bash
# Stack 1
git-spice trunk
git-spice bc auth-models -m "Auth models"
git-spice bc auth-api -m "Auth API"

# Stack 2
git-spice trunk
git-spice bc reporting-models -m "Reporting models"
git-spice bc reporting-ui -m "Reporting UI"

# View all stacks
git-spice ls --all
```

## Squash-Merge Reconciliation

When a PR is squash-merged, commit hashes change and upstack branches become stale:

```bash
git-spice repo sync        # Detects merged branches, deletes them
git-spice stack restack    # Rebases remaining onto updated trunk
git-spice stack submit     # Updates remaining PRs
```

## Common Mistakes and Red Flags

| Mistake | Why it's wrong | Correct approach |
|---------|----------------|------------------|
| Using `git rebase` on tracked branches | Breaks git-spice's dependency tracking | `git-spice upstack restack` / `git-spice stack restack` |
| Using `git push` / `git push --force` | Bypasses PR tracking, breaks linkage | `git-spice branch submit` / `git-spice stack submit` |
| Rebasing children onto trunk after parent merges | Breaks tracked relationships, causes conflicts | `git-spice repo sync --restack` |
| Using `git-spice stack submit` when you meant "just children" | Submits ancestors too (sometimes surprising) | `git-spice upstack submit` |
| Forgetting to initialize the repo | Commands fail with unclear errors | Run `git-spice repo init` once per repo |
| Assuming restack happens automatically | Stacks drift after manual `git commit` | Use `git-spice cc` / `git-spice ca`, or explicitly `git-spice ur` |
| Merging a stacked PR with `gh pr merge --delete-branch` | Branch may be deleted before GitHub can retarget dependents, closing them | Let GitHub delete branches via repo settings, or use `git-spice repo sync` to recover |

### Red flags -- stop and reconsider:

- You're about to run `git rebase` or `git push --force` on a tracked branch.
- You're not sure whether you want `upstack` vs `stack` vs `downstack` scope.
- You're running `git-spice stack submit` with an external (untracked) base branch in the stack.

When in doubt:

```bash
git-spice log short
git-spice <command> --help
```

## Configuration

git-spice uses `git config` for settings. Common useful options:

```bash
# Prefix branch names (e.g., with username)
git config spice.branchCreate.prefix "myname/"

# Always create PRs as drafts
git config spice.submit.draft true

# Set default reviewers
git config spice.submit.reviewers "teammate1,teammate2"

# Add default labels
git config spice.submit.label "stacked,needs-review"

# Navigation comments: only when multiple CRs in stack
git config spice.submit.navigationComment multiple

# Open browser when CR is created
git config spice.submit.web created

# Show all stacks by default
git config spice.log.all true

# Repo sync: ignore closed (unmerged) CRs instead of prompting
git config spice.repoSync.closedChanges ignore
```

Shell completion:

```bash
eval "$(git-spice shell completion bash)"    # Bash
eval "$(git-spice shell completion zsh)"     # Zsh
eval "$(git-spice shell completion fish)"    # Fish
```

## Best Practices

1. **Atomic branches**: Each branch should be one logical, reviewable change.
2. **Stack size**: Keep stacks to 3-5 branches maximum.
3. **Stack from trunk**: Build stacks starting from main/master.
4. **Restack often**: Keep branches rebased on latest changes.
5. **Submit together**: Use `git-spice ss --fill` to create linked PRs.
6. **Sync regularly**: Use `git-spice rs` to stay current with upstream.
7. **Merge from bottom to top**: After each merge, sync with `git-spice rs`.
8. **Check stack state**: Run `git-spice ls` before and after operations.
9. **Use shorthands**: `git-spice cc`, `git-spice ss`, `git-spice rs` for efficiency.

## Agent / Claude Code Integration Notes

- **Always write `git-spice` as the command name.** Never use `gs` or `gsp` in commands, examples, or explanations -- even though `gs` is a common alias, the user expects `git-spice`.
- **Always use `git-spice ss` or `git-spice bs` to push. Never `git push`.**
- **Never run interactive commands** (`git-spice bco`, `git-spice bsp`, `git-spice se`, `git-spice csp`) without user confirmation -- these require TTY interaction.
- **Use `--fill` on first submit** to auto-populate PR descriptions.
- **Run `git-spice repo sync` before starting new work** to ensure a clean state.
- **After submitting**, share the PR URLs with the user.
- **If `git-spice` is not installed**, suggest: `brew install git-spice`.
- **If repo is not initialized**, run `git-spice repo init` first.
- **Require a clean working tree** before `restack`, `submit`, `sync`, `edit`, or delete operations.

## Reference Documentation

For detailed information on specific topics, see:

- **[CLI Reference](references/cli-reference.md)** -- Complete command documentation with all flags, shorthands, and configuration keys
- **[Workflows](references/workflows.md)** -- Advanced workflow patterns (12 workflows)
- **[Worktrees and Agents](references/worktrees-and-agents.md)** -- Using git-spice with Git worktrees for parallel multi-agent development
- **[Tracking External Branches](references/tracking-external-branches.md)** -- Safely stacking on someone else's PR branch without accidentally modifying it
- **[PR Status and Stack Views](references/pr-status-and-stack-views.md)** -- Combining `git-spice ls --json` with `gh pr view` for rich stack views with review/CI status

## External Documentation

- [git-spice Official Docs](https://abhinav.github.io/git-spice/)
- [CLI Reference](https://abhinav.github.io/git-spice/cli/reference/)
- [Configuration](https://abhinav.github.io/git-spice/cli/config/)
- [GitHub Repository](https://github.com/abhinav/git-spice)
