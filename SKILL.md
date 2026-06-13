---
name: pr-best-practices
version: 1.0.0
description: >
  Use this skill to author strong pull requests across GitHub, GitLab, and Bitbucket.
  Activate when a user is opening, drafting, splitting, or polishing a pull request or
  merge request; writing a PR title or description; wondering whether a PR is too big
  to review; addressing or responding to review feedback; or summarizing changes for a
  reviewer. Also activate right after a push when the next natural step is opening a PR.
  Do not use this skill to review someone else's code (use pr-review for that), to run
  a pre-submit checklist before opening a PR (use pr-presubmit for that), or to manage
  CI/CD pipelines or deployment workflows.
allowed-tools: []
---

# Authoring Pull Requests

A pull request is two things at once: a message to a busy reviewer, and the start of a
dialogue about someone's work. Most of what makes a PR good is consideration for the
person on the other side — they have limited time and attention, and review is
asynchronous, so the context you'd get from body language or a quick back-and-forth is
gone. Everything below optimizes for the reviewer's time and for a productive
conversation, because that's how a PR actually gets merged.

The PR description (title + body) is the concrete core of this — see
[Writing the title and body](#writing-the-title-and-body) — but a great PR also depends
on what you do *before* you open it and *after* the comments come in.

## Before you open the PR

**Be your own first reviewer.** Read your own diff start to finish before inviting
anyone. Reviewing changes outside the editor you wrote them in surfaces leftover debug
code, inconsistencies, missing pieces, and "wait, why did I do it that way" moments. The
cheapest review comment is the one you catch yourself.

**Keep it small.** Small, single-purpose PRs get reviewed faster and far more
thoroughly — there's a well-known dynamic where "10 lines of code = 10 comments, 500
lines = looks good to me," because a reviewer can't hold a huge diff in their head. If
the change is large, look for a way to split it into a sequence of smaller, independently
reviewable PRs. When you spot work that's tempting to bundle in but isn't essential to
this change (a drive-by refactor, an unrelated cleanup), prefer carving it into a
separate follow-up rather than growing the diff. If you're authoring on the user's
behalf and the change is sprawling, say so and suggest how to break it up.

**Wait for the green light.** If there's CI, let it pass before requesting review. You
don't want the first comment to be "the build's red." If you can't run CI, at least run
the tests and linter locally and say what you ran.

**Leave it a little better (the boy-scout rule).** Opening a PR is a chance to nudge the
surrounding code in the right direction — a missing test, an overlong function, a naming
inconsistency. You don't have to make it perfect; just a bit better than you found it.
Keep these improvements proportional, though — don't let them balloon the diff and
undercut "keep it small." Big cleanups belong in their own task.

## Writing the title and body

This is the part the reviewer reads first. The diff already says *what* changed line by
line — your job is to say *why*, and to point the reviewer at what matters, so they can
approve with confidence and minimal back-and-forth.

First, gather the real story so the description reflects reality, not just the prompt:
- Read the diff: `git diff <base>...HEAD` (or `git diff --stat` for the shape of it).
- Read the commit messages: `git log <base>..HEAD --oneline`.
- Note any issue/ticket it closes and how the change was verified. If something is
  genuinely unclear (why a change was made, whether tests ran), ask rather than invent —
  a confident-but-wrong description erodes reviewer trust fast.

### The title

One line, imperative mood, specific enough to scan in a list of fifty PRs.
- Good: `Fix race condition in session cache eviction`
- Weak: `Bug fix` / `Updates` / `Changes to cache` — these tell a reviewer nothing.

If the repo clearly uses a convention (Conventional Commits like `fix:`/`feat:`, or a
`[JIRA-123]` prefix), match it — check recent merged PR titles. Don't impose one the
project doesn't use.

### The body

**Size the PR before you reach for any structure.** The amount of scaffolding should
match the change, not the other way around. For a trivial one-liner (a constant bump, a
typo, a version pin), the *entire* body is one sentence saying why — no headers, no "Test
plan: N/A". Adding `## Summary` / `## Test plan` to a one-line change is noise the
reviewer has to read past; it's the most common over-templating mistake, so resist it.
Only reach for the template below once the change is big enough that a reviewer would
actually benefit from the structure.

For everything beyond trivial, default to a lean two-part structure. It's enough for the
vast majority of PRs:

```markdown
## Summary
<1-4 sentences: what this change does and, more importantly, *why*. Lead with the
motivation or problem, then the approach. Flag anything a reviewer should scrutinize.>

## Test plan
<How you verified it. Concrete and honest — the commands you ran, the cases you checked,
what a reviewer can do to confirm. A checklist works well for several steps.>
```

If the PR closes an issue, add a `Closes #123` line (or the platform's equivalent),
usually at the end of the Summary.

**What makes the Summary good:**
- **Lead with why.** "Webhook retries were dropping events during deploys, so this adds
  backoff" beats "Added a retry loop." The reviewer sees the loop in the diff; they can't
  see the outage that motivated it. This is the single highest-value thing you provide.
- **Describe behavior, not files.** Avoid a mechanical "Modified `foo.py`, updated
  `bar.py`" list — that's just the diff restated, and it ages badly. Group by what
  changed conceptually; a short bulleted list is fine when a PR touches several things.
- **Flag the risky bits.** Tradeoffs, follow-ups, deliberately out-of-scope pieces, spots
  you're unsure about — call them out. Reviewers value being pointed at what matters, and
  it's exactly the "things reasonable people can disagree about" that review is for.
- **Match length to the change.** A typo fix gets one sentence; don't pad it with
  ceremony. A 600-line refactor needs more room — give it.

**What makes the Test plan good:**
- Be specific: `pytest tests/payments/ -k webhook` and "verified the 500 path retries 3×
  then dead-letters" — not "tested locally."
- Be honest. If tests weren't run or a case is unverified, say so plainly rather than
  implying coverage that doesn't exist.
- For UI changes, note screenshots/recordings (or that they should be attached).

### Scaling the template up or down

The two-part structure is the default, not a cage:
- **Tiny PR** (typo, version bump, one-line fix): one Summary sentence, and a Test plan
  only if there's anything to verify. Skip the headers if they'd outweigh the change.
- **Large or risky PR**: add sections as they earn their place — `## Motivation`/
  `## Context` when the why needs room, `## Breaking changes`, `## Rollback plan`,
  `## Screenshots`, `## Out of scope`. Each should give the reviewer something the diff
  can't.

See `.github/skills/pr-best-practices/references/examples.md` for worked before/after examples across PR sizes.

## Choosing reviewers

When the workflow involves picking reviewers, be deliberate but not excessive. Ask: who's
impacted by or has context on this change, who's qualified to give the best feedback, and
who actually has time? Too many reviewers is its own failure mode — either a flood of
comments pulling in different directions, or diffusion of responsibility where everyone
assumes someone else will do it. A good pattern: a small set of core reviewers for the
real feedback, plus anyone who just needs visibility. If part of the change is outside
your or the obvious reviewers' expertise, pull in someone who can actually judge it.

## After the comments come in

A review is a dialogue, and it should reach a clear conclusion.

**Answer every comment.** Leaving threads dangling is both disrespectful of the
reviewer's time and leaves the conversation unresolved — and the resolved thread is a
useful record later. Acknowledge, push back with reasoning, or make the change, but close
each loop.

**Clarify the code before defending it.** If a reviewer didn't understand something, that
future readers won't either. Your first move should be to make the code clearer or add a
comment explaining *why* it's there — not to write an explanation that lives only in the
review thread and helps no one later. Only when the code genuinely can't be clarified
does a review-comment explanation make sense.

**Stay open; don't take it personally.** Critique of the code is an attempt to help and
to protect the codebase you both share — it isn't about you. Engage with the reasoning,
assume good intent, and evaluate your code as if someone else wrote it. When you disagree,
many decisions are genuine judgment calls, so open a conversation about the tradeoffs
rather than treating either side as obviously right.

**Don't chase perfection forever.** There's a threshold past which more changes aren't
worth it. Balance code quality against shipping and momentum; if a comment surfaces a big
refactor, the right move is usually a follow-up task linked from the PR, not an
ever-growing diff that risks merge conflicts and drains everyone's motivation.

## A note on tone (for both authoring and reviewing)

When this skill is used to *write* review comments or respond to them, keep it
respectful and collaborative — it's a critique of work, and tone carries the
conversation:
- Ask, explain, suggest — rather than command, judge, or blame. "What do you think about
  keying this on account id?" lands better than "Don't do this."
- Use inclusive language: "our code," "should we," "what if we" — not "your code"/"you
  did," which reads as finger-pointing.
- Offer genuine appreciation when something's done well; reviews skew toward problems, and
  a "nice, this is much cleaner" costs nothing and means a lot.
- Back suggestions with references (a doc link, a code sample, a prior PR) so they're easy
  to act on and don't come across as mere opinion.
- If a thread is spiraling into many back-and-forth replies, that's a signal to switch to
  a real-time conversation, then post a short summary of what was agreed.

## Avoid

- Marketing language and filler ("This amazing PR significantly improves…"). State it
  plainly.
- Restating the diff file-by-file.
- Claiming tests/verification that didn't happen.
- A wall of text. If the reviewer has to hunt for the point, the description failed.
