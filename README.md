# pr-best-practices

A Claude Code skill for authoring strong pull requests. It guides you through writing clear PR titles and descriptions, preparing changes before you open a review, choosing reviewers, and responding to feedback well.

## What this skill does

This skill helps you or Claude produce pull requests that reviewers can actually act on. It covers:

- Writing a title and body that lead with *why*, not just *what*
- Sizing the description to match the change (no ceremony on a one-liner)
- Reviewing your own diff before anyone else sees it
- Keeping the change small and single-purpose
- Waiting for CI before requesting review
- Choosing the right reviewers without over-requesting
- Responding to comments in a way that closes every thread and improves the code

It works across GitHub, GitLab, and Bitbucket and applies to any language or stack.

## Trigger phrases

Claude activates this skill when you say things like:

- "Help me write a PR description"
- "Write a pull request for these changes"
- "Is my PR ready to open?"
- "How should I describe this change?"
- "Can you draft a merge request title and body?"
- "Is this PR too big?"
- "How do I respond to this review comment?"
- "Summarize my changes for a reviewer"

It also activates right after a `git push` when the natural next step is opening a PR.

## When to use it

Use this skill any time you are:

- Opening or drafting a pull request or merge request
- Writing or improving a PR title or description
- Deciding whether to split a large change
- Polishing a PR before requesting review
- Addressing feedback and closing review threads

## File structure

```
SKILL.md                  # Main skill instructions
references/
  examples.md             # Before/after PR examples across small, medium, and large changes
```

## Core principles

**Lead with why.** Reviewers can read the diff to see what changed line by line. What they cannot see is the bug, the user need, or the business reason that motivated the change. The summary's job is to supply that context.

**Size to the change.** A one-line fix gets one sentence. A 600-line refactor needs more room. Adding `## Summary` and `## Test plan` headers to a typo fix is noise the reviewer has to read past.

**Be your own first reviewer.** Reading your diff outside the editor you wrote it in surfaces leftover debug code, half-finished logic, and "why did I do it that way" moments before anyone else sees them.

**Keep it small.** Small, single-purpose PRs get reviewed faster and more thoroughly. There is a well-known dynamic where 10 lines of code draw 10 comments while 500 lines get "looks good to me," because a reviewer cannot hold a large diff in their head.

**Close every thread.** Leaving review comments unanswered is disrespectful of the reviewer's time and leaves the conversation unresolved. Acknowledge, push back with reasoning, or make the change - but close each loop.

## Default PR template

For anything beyond a trivial change, the skill defaults to a lean two-part structure:

```markdown
## Summary
<1-4 sentences: what this change does and, more importantly, why. Lead with the
motivation or problem, then the approach. Flag anything a reviewer should scrutinize.>

## Test plan
<How you verified it. Concrete and honest - the commands you ran, the cases you checked,
what a reviewer can do to confirm. A checklist works well for several steps.>
```

For tiny changes (a typo fix, a version bump, a one-line patch), skip the headers entirely and write one sentence saying why.

See [`references/examples.md`](references/examples.md) for worked before/after examples at each size.

## Example output

**Weak:**
```
Title: Fixed bug

## Summary
Modified cache.py to fix the issue.

## Test plan
Tested locally.
```

**Good:**
```
Title: Fix race condition in session cache eviction

## Summary
Concurrent requests could evict a session entry that another request had just
refreshed, causing sporadic logouts under load. The eviction check now compares the
entry's refreshed-at timestamp under the same lock used to write it, closing the window.

Closes #482

## Test plan
- pytest tests/cache/test_eviction.py (added a test that reproduces the race with
  50 concurrent refresh/evict pairs - fails on main, passes here)
- Ran the staging load test (200 rps for 5 min); no spurious logouts vs. ~12/min before
```

## Installation

### Via Claude Code

1. Download `pr-best-practices.skill` (the packaged version of this repo).
2. In Claude Code, run:
   ```
   /install-skill pr-best-practices.skill
   ```
3. The skill is now available in your session.

### Manual setup

Clone this repo and point your Claude Code project at it, or copy `SKILL.md` into your project's `.claude/skills/` directory.

## Security notes

This skill requires no shell execution or filesystem write permissions. It reads context you provide (a diff, a description, a commit log) and produces text output. No tools beyond standard read access are needed.

## Guidelines for contributors

- **One skill, one purpose.** This skill focuses on PR authorship. Do not expand it to cover unrelated workflows.
- **Specific trigger words.** Any changes to the description frontmatter should include clear trigger phrases so Claude can activate this skill accurately.
- **Progressive disclosure.** Keep `SKILL.md` concise. Add auxiliary context and advanced examples to files under `references/`, not to the main script.
- **Concrete examples.** Show real PR text rather than abstract pseudocode or placeholder prose.
- **Minimal permissions.** Do not add bash or shell execution to this skill. It does not need them.

## Related skills

- [pr-presubmit](https://github.com/julianbesonen/pr-presubmit) - Run a structured pre-submit checklist before opening a PR
- [pr-review](https://github.com/julianbesonen/pr-review) - Write thoughtful, collaborative code review comments
