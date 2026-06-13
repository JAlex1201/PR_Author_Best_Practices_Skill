# PR Author Best Practices Skill

> A Claude Code skill for authoring strong pull requests - titles, descriptions, scoping, reviewer selection, and responding to feedback.

A pull request is two things at once: a message to a busy reviewer, and the start of a dialogue about someone's work. Most of what makes a PR good is consideration for the person on the other side. They have limited time and attention, and review is asynchronous, so the context you would get from body language or a quick back-and-forth is gone. This skill optimizes for the reviewer's time and for a productive conversation, because that is how a PR actually gets merged.

Works across GitHub, GitLab, and Bitbucket. Applies to any language or stack.

## What this skill does

This skill helps you or Claude produce pull requests that reviewers can actually act on. It covers:

- Writing a title and body that lead with *why*, not just *what*
- Sizing the description to fit the change (no ceremony on a one-liner, enough room for a large refactor)
- Reviewing your own diff before anyone else sees it
- Keeping the change small and single-purpose
- Waiting for CI before requesting review
- Choosing the right set of reviewers without over-requesting
- Responding to review comments in a way that closes every thread and improves the code
- Maintaining a collaborative tone throughout the review dialogue

## When to use it

Use this skill any time you are:

- Opening or drafting a pull request or merge request
- Writing or improving a PR title or description
- Deciding whether to split a large change into smaller pieces
- Polishing a PR before requesting review
- Responding to review feedback and closing comment threads
- Summarizing a set of changes for a reviewer or team

It also activates right after a `git push` when the natural next step is opening a PR.

## Trigger phrases

Claude activates this skill when you say things like:

- "Help me write a PR description"
- "Write a pull request for these changes"
- "Draft a merge request title and body"
- "How should I describe this change?"
- "Is my PR ready to open?"
- "Is this PR too big to review?"
- "How should I respond to this review comment?"
- "Summarize my changes for a reviewer"
- "Polish this PR before I request review"
- "Should I split this into multiple PRs?"

**Not triggered by:** requests to review someone else's code (use [pr-review](https://github.com/JAlex1201/pr-review)), pre-submit checklists (use [pr-presubmit](https://github.com/JAlex1201/pr-presubmit)), or CI/CD pipeline configuration.

## File structure

```
SKILL.md                                              # Main skill instructions loaded by Claude
references/
  examples.md                                         # Before/after PR examples at each size
.github/
  skills/
    pr-best-practices/
      SKILL.md                                        # GitHub Copilot discovery path
      references/
        examples.md                                   # Bundled reference for Copilot agents
```

The root `SKILL.md` is used by Claude Code. The `.github/skills/` path follows the GitHub Copilot agent skill convention for cross-agent discoverability.

## Core principles

**Lead with why.** Reviewers can read the diff to see what changed line by line. What they cannot see is the bug, the user need, or the business reason that motivated the change. The summary's job is to supply that context. "Webhook retries were dropping events during deploys, so this adds backoff" is more useful than "Added a retry loop." The retry loop is visible in the diff; the outage that motivated it is not.

**Size the description to the change.** A one-line fix gets one sentence. A 600-line refactor needs more room. Adding `## Summary` and `## Test plan` headers to a typo fix is noise the reviewer has to read past. Only reach for structure once the change is big enough that a reviewer would actually benefit from it.

**Be your own first reviewer.** Reading your diff outside the editor you wrote it in surfaces leftover debug code, half-finished logic, and "why did I do it that way" moments before anyone else sees them. The cheapest review comment is the one you catch yourself.

**Keep it small.** Small, single-purpose PRs get reviewed faster and more thoroughly. There is a well-known dynamic where 10 lines of code draw 10 comments while 500 lines get "looks good to me," because a reviewer cannot hold a large diff in their head. If a change is sprawling, look for a way to split it into a sequence of independently reviewable PRs.

**Wait for green.** If there is CI, let it pass before requesting review. You do not want the first comment to be "the build is red."

**Close every thread.** Leaving review comments unanswered is disrespectful of the reviewer's time and leaves the conversation unresolved. Acknowledge, push back with reasoning, or make the change - but close each loop.

**Clarify the code before defending it.** If a reviewer did not understand something, future readers probably will not either. The first move should be to make the code clearer or add a comment explaining why it is there, not to write an explanation that lives only in the review thread.

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

For tiny changes (a typo fix, a version bump, a one-line patch), skip the headers entirely and write one sentence saying why. Adding ceremony to a one-line change is the most common over-templating mistake.

For large or risky changes, add sections as they earn their place: `## Motivation` when the why needs more room, `## Breaking changes`, `## Rollback plan`, `## Screenshots`, `## Out of scope`. Each section should give the reviewer something the diff cannot.

See [`references/examples.md`](references/examples.md) for worked before/after examples at each size: tiny change, medium feature, and large refactor.

## Before/after example

**Weak:**
```
Title: Fixed bug

## Summary
Modified cache.py to fix the issue. Updated the eviction logic and
changed how timestamps are compared.

## Test plan
Tested locally.
```

**Strong:**
```
Title: Fix race condition in session cache eviction

## Summary
Concurrent requests could evict a session entry that another request had just
refreshed, causing sporadic logouts under load. The eviction check now compares
the entry's refreshed-at timestamp under the same lock used to write it,
closing the race window.

Closes #482

## Test plan
- pytest tests/cache/test_eviction.py (added a test that reproduces the race
  with 50 concurrent refresh/evict pairs - fails on main, passes here)
- Ran the staging load test (200 rps for 5 min); no spurious logouts vs.
  ~12/min before the fix
```

Why the second one is better: the title is scannable in a list of fifty PRs, the summary leads with the problem not the solution, the test plan is specific and honest about what was verified, and the issue link closes the loop automatically on merge.

## Choosing reviewers

Be deliberate but not excessive. Ask: who is impacted by or has context on this change, who is qualified to give the best feedback, and who actually has time?

Too many reviewers creates its own failure mode - either a flood of conflicting comments or diffusion of responsibility where everyone assumes someone else will review it. A good pattern is a small core set for substantive feedback, plus anyone who just needs visibility. If part of the change is outside your area of expertise or the obvious reviewers', pull in someone who can actually judge that piece.

## Responding to review comments

A review is a dialogue and it should reach a clear conclusion.

Answer every comment. Acknowledge, push back with reasoning, or make the change - but close each loop. Leaving threads dangling is disrespectful and the unresolved thread is a source of confusion later.

Stay open; do not take it personally. Critique of the code is an attempt to help and to protect a codebase you both share. Engage with the reasoning, assume good intent, and evaluate your code as if someone else wrote it. When you disagree, many decisions are genuine judgment calls, so open a conversation about the tradeoffs rather than treating either side as obviously right.

Do not chase perfection forever. Balance code quality against shipping. If a comment surfaces a big refactor, the right move is usually a follow-up task linked from the PR, not an ever-growing diff that risks merge conflicts and drains everyone's motivation.

## Tone guidelines

Whether you are writing the PR description or responding to comments, keep it respectful and collaborative:

- Ask, explain, suggest rather than command, judge, or blame. "What do you think about keying this on account id?" lands better than "Do not do this."
- Use inclusive language: "our code," "should we," "what if we" rather than "your code" or "you did," which reads as finger-pointing.
- Offer genuine appreciation when something is done well. Reviews skew toward problems, and a "nice, this is much cleaner" costs nothing and means a lot.
- Back suggestions with references (a doc link, a code sample, a prior PR) so they are easy to act on and do not come across as mere opinion.
- If a thread accumulates many back-and-forth replies, that is a signal to switch to a real-time conversation, then post a short summary of what was agreed.

## Installation

### Claude Code (recommended)

Add this repo as a skill source in your Claude Code project settings, or copy `SKILL.md` into your project's `.claude/skills/pr-best-practices/` directory. Claude will automatically discover and activate the skill based on the trigger phrases in the frontmatter description.

### GitHub Copilot agents

The skill is also available at `.github/skills/pr-best-practices/SKILL.md` following the GitHub Copilot agent skill directory convention. Clone or fork this repo and your Copilot agent will pick it up from that path.

### Manual

Copy `SKILL.md` and the `references/` folder to wherever your AI agent looks for skill instructions. The skill has no dependencies and requires no configuration beyond the file being readable.

## Security notes

This skill requires no shell execution or filesystem write permissions. It reads context you provide (a diff, a description, a commit log) and produces text output. The `allowed-tools` field in the frontmatter is explicitly empty. Do not modify this skill to grant shell or bash access - it does not need them.

## Guidelines for contributors

- **One skill, one purpose.** This skill focuses on PR authorship. Do not expand it to cover code review, pre-submit checks, or deployment workflows.
- **Specific trigger words.** Any changes to the description frontmatter must include clear trigger phrases and negative triggers so the agent can activate this skill accurately and avoid false positives.
- **Progressive disclosure.** Keep `SKILL.md` focused on principles and structure. Add worked examples and auxiliary context to `references/examples.md` rather than growing the main instruction file.
- **Concrete examples.** Show real PR text rather than abstract pseudocode or placeholder prose. Weak/strong pairs are more useful than rules alone.
- **No em dashes in prose.** Use hyphens, colons, or restructure the sentence.
- **Minimal permissions.** Do not add bash or shell execution to this skill. It does not need them.

## License

Apache 2.0. See [LICENSE](LICENSE).

## Related skills

- [pr-presubmit](https://github.com/JAlex1201/pr-presubmit) - Run a structured pre-submit checklist before opening a PR
- [pr-review](https://github.com/JAlex1201/pr-review) - Write thoughtful, collaborative code review comments
