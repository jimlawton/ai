# Agent prompts, rules, and skills

A collection of useful Agent prompts, rules files, and Claude Code skills/plugins.

## Skills & Plugins

### git-spice

A Claude Code plugin for managing stacked Git branches with [git-spice](https://github.com/abhinav/git-spice). Provides CLI reference, workflow patterns, safety rules, worktree coordination for multi-agent setups, and squash-merge reconciliation.

**Location:** `skills/git-spice/`

**What's included:**
- Complete command reference with all flags and shorthands
- 13 workflow patterns (creating stacks, mid-stack edits, sync after merges, etc.)
- Safety rules (never `git push` or `git rebase` on tracked branches)
- Worktree coordination for parallel multi-agent development
- Tracking external branches safely
- PR status and stack views with `gh` CLI

**Installing as a plugin from GitHub:**

```bash
# Add this repo as a plugin marketplace (one-time)
/plugin marketplace add jimlawton/ai

# Install the git-spice plugin
/plugin install git-spice@jimlawton-plugins

# Reload to activate
/reload-plugins
```

The skill is then available as `/git-spice:git-spice` and will trigger automatically when you ask about stacked branches, stacked PRs, or git-spice workflows.

**Alternative: install from a local path:**

```bash
/plugin install /path/to/this-repo/skills/git-spice
```

**Alternative: use as a standalone skill (no plugin):**

Copy the skill directory into your project's `.claude/skills/`:

```bash
cp -r skills/git-spice/skills/git-spice /path/to/your-project/.claude/skills/
```

## References

- https://www.reddit.com/r/ArtificialInteligence/comments/1kw16yi/a_comprehensive_list_of_agentrule_files_do_we/ - discussion of the various AI systems and their use of rules files.
- https://github.com/jlevy/simple-modern-uv - General and Python rules.
- https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions - Copilot rules documentation.
- https://docs.claude.com/en/docs/claude-code/memory - Claude memory documentation.
- https://github.com/steipete/agent-rules - a large collection of rules files.
- https://agents.md/ and https://github.com/openai/agents.md - AGENTS.md format.
- https://github.com/Ranrar/rustic-prompt - Rust-specific rules.
- https://microsoft.github.io/rust-guidelines/ - Microsoft Rust guidelines, also available as a [rules file](https://microsoft.github.io/rust-guidelines/agents/all.txt).
