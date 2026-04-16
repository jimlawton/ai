# Git Spice Workflows

Advanced workflow patterns for managing stacked branches.

## Workflow 1: Building a Feature Stack

### Scenario

You're implementing a large feature that should be broken into reviewable pieces.

### Steps

```bash
# 1. Start from updated trunk
git-spice trunk
git pull origin main

# 2. Create the first branch (foundation layer)
git-spice bc api-models -m "Add User and Profile models"

# Make your changes...
git add src/models/
git-spice ca  # Amend to add more changes
# Or create additional commits
git-spice cc -m "Add model validation"

# 3. Create second branch (builds on first)
git-spice bc api-endpoints -m "Add user API endpoints"

# Make changes...
git add src/api/
git-spice cc -m "Implement CRUD operations"

# 4. Create third branch (builds on second)
git-spice bc api-tests -m "Add API integration tests"

# Make changes...
git add tests/
git-spice cc -m "Add user endpoint tests"

# 5. View your stack
git-spice ll
```

### Result

```
      +-- api-tests
    +-+ api-endpoints
  +-+ api-models
  main
```

## Workflow 2: Submitting a Stack for Review

### Scenario

Your stack is ready for review. You want to create linked PRs.

### Steps

```bash
# 1. Ensure you're authenticated
git-spice auth status

# 2. View what will be submitted
git-spice ls

# 3. Submit entire stack
git-spice ss

# For draft PRs
git-spice ss --draft

# For auto-filled titles/bodies from commits
git-spice ss --fill

# 4. View PR links
git-spice ls  # Shows PR numbers next to branches
```

git-spice automatically adds navigation comments to PRs showing the stack structure.

## Workflow 3: Updating After Review Feedback

### Scenario

Reviewers requested changes to a branch in the middle of your stack.

### Steps

```bash
# 1. Check out the branch that needs changes
git-spice bco api-endpoints

# 2. Make your changes
# ... edit files ...

# 3. Either amend existing commit
git-spice ca

# Or create a new commit
git-spice cc -m "Address review: improve error handling"

# 4. Restack everything above this branch
git-spice ur  # upstack restack (already done if you used gs cc/gs ca)

# 5. Re-submit updated branches
git-spice uss  # upstack submit (force push happens automatically)
```

## Workflow 4: Syncing with Upstream

### Scenario

Trunk has new commits and your stack needs updating.

### Steps

```bash
# 1. Sync repository
git-spice rs  # repo sync
# This fetches from remote, fast-forwards trunk, deletes merged branches

# 2. Restack your branches
# Option A: Restack all tracked branches
git-spice repo restack

# Option B: Restack just current stack
git-spice sr  # stack restack

# 3. Re-submit to update PRs
git-spice ss
```

## Workflow 5: Handling Merge Conflicts

### Scenario

A restack operation encounters conflicts.

### Steps

```bash
# 1. Start restack
git-spice sr

# 2. If conflicts occur, git-spice pauses and shows conflicted files
git status

# 3. Resolve conflicts in your editor
# ... edit conflicted files ...

# 4. Stage resolved files
git add <resolved-files>

# 5. Continue the restack
git-spice rbc  # rebase continue

# If conflicts continue in other branches, repeat steps 3-5

# 6. If you need to abort
git-spice rba  # rebase abort
```

## Workflow 6: Reorganizing a Stack

### Reorder Branches

```bash
git-spice se  # stack edit - opens editor with branch list, reorder as needed
```

### Insert New Branch Mid-Stack

```bash
# Check out where you want to insert
git-spice bco api-endpoints

# Create with --insert flag (restacks upstack onto new branch)
git-spice bc api-middleware --insert -m "Add rate limiting middleware"

# Result: trunk -> api-models -> api-endpoints -> api-middleware -> api-tests
```

### Move Branch to Different Base

```bash
git-spice bo main           # Move only current branch to trunk
git-spice uo other-feature  # Move current + entire upstack to new base
```

## Workflow 7: Cleaning Up After Merge

### Scenario

PRs have been merged and you want to clean up.

### Steps

```bash
# 1. Sync to detect merged PRs
git-spice rs
# Repo sync automatically detects merged PRs, deletes local branches, updates tracking

# 2. If manual cleanup needed
git-spice bd merged-branch --force
```

## Workflow 8: Working with Draft PRs

```bash
# 1. Submit as drafts
git-spice ss --draft

# 2. Continue working and updating
git-spice bco some-branch
# ... make changes ...
git-spice ca
git-spice ur
git-spice uss  # Updates existing draft PRs

# 3. When ready, mark as ready on GitHub/GitLab web UI
```

## Workflow 9: Splitting a Large Branch

### Interactive Split

```bash
git-spice bco large-feature
git-spice bsp  # Interactive - select commits for each new branch
```

### Manual Split

```bash
git-spice bco large-feature
git-spice bc first-half --below    # Create branch below current
git-spice cp <commit-hash>          # Cherry-pick relevant commits
git-spice bco large-feature
git-spice be                        # Edit branch to remove moved commits
```

## Workflow 10: Squashing Before Merge

```bash
git-spice bsq                      # Squash all commits in branch to one
# Or use branch edit for selective squashing:
git-spice be                        # Change "pick" to "squash" in editor
```

## Workflow 11: Importing Existing Branches

### Track a single branch

```bash
git-spice bt existing-feature --base main
```

### Track a chain of branches

```bash
# Bottom-up (one at a time)
git-spice bt feature-part1 --base main
git-spice bt feature-part2 --base feature-part1
git-spice bt feature-part3 --base feature-part2

# Or use downstack track from the top (auto-walks the graph)
git checkout feature-part3
git-spice dst
```

### Adopt a big branch into a multi-branch stack

If your branch has several commits that should be separate PRs:

```bash
# Create branches at key commits
git branch pay-core   <commit1>
git branch pay-stripe <commit2>
git branch pay-ui     <commit3>

# Track the whole stack from the top
git checkout pay-ui
git-spice dst  # downstack track
```

## Workflow 12: Multiple Independent Stacks

```bash
# Stack 1: Auth feature
git-spice trunk
git-spice bc auth-models -m "Auth models"
git-spice bc auth-api -m "Auth API"

# Stack 2: Separate feature
git-spice trunk  # Return to trunk
git-spice bc reporting-models -m "Reporting models"
git-spice bc reporting-ui -m "Reporting UI"

# View all stacks
git-spice ls --all

# Navigate between stacks
git-spice bco auth-api       # Jump to auth stack
git-spice bco reporting-ui   # Jump to reporting stack
```

## Workflow 13: Recovering from GitHub Auto-Retarget Race Condition

When merging a stacked PR with `gh pr merge --delete-branch`, GitHub may delete the branch before retargeting dependents, closing them.

### Prevention

- Don't use `--delete-branch` flag -- let GitHub delete branches via repo settings
- Or manually update base branches of dependent PRs before merging

### Recovery

```bash
git-spice rs                              # Sync - detects closed PR
git-spice bt <branch> --base main         # Re-track with correct base
git-spice sr                              # Restack
git-spice ss                              # Submit - creates new PR
```

## Best Practices

### Stack Size

- Keep stacks to 3-5 branches maximum
- Each branch should be independently reviewable
- Larger features: consider multiple smaller stacks

### Commit Hygiene

- Use meaningful commit messages
- Squash fixup commits before final review (`git-spice bsq` or `git-spice be`)
- Keep commits atomic and focused

### Naming Conventions

```bash
git config spice.branchCreate.prefix "username/"
# All branches created with git-spice bc will be prefixed: username/feature-name
```

### Review Workflow

1. Submit stack as draft (`git-spice ss --draft`)
2. Get early feedback
3. Address comments branch by branch
4. Restack after changes (`git-spice sr`)
5. Mark ready when complete

### Merge Strategy

- Merge from bottom to top
- After each merge, sync (`git-spice rs`)
- Continue merging remaining branches
