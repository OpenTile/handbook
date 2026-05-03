# Code Review Guide

This document is the foundation for everyone doing code reviews on the team. Read it once before your first review, and come back to it whenever you hit a tricky situation.

It's based on two articles by Michael Lynch: [How to Do Code Reviews Like a Human, Part 1](https://mtlynch.io/human-code-reviews-1/) and [Part 2](https://mtlynch.io/human-code-reviews-2/).

---

## TL;DR

- The goal of a review is not to find the maximum number of bugs, but to help your teammate make the code better while keeping a healthy working relationship.
- Automate everything you can (linters, formatters, CI). Style debates are settled by the [style guide]([LINK]).
- Start the review immediately, respond fast, work top-down from high-level issues to small ones.
- Frame feedback as requests, ground notes in principles rather than taste, and never say "you".
- Don't push for perfection — aim to raise the code by one or two letter grades. Approve as soon as only minor issues remain.
- A stalemate in a review is always a conflict between people, not about the code. Address it proactively.

---

## Core concepts

A review involves an **author** (the one who writes the code) and a **reviewer** (the one who reviews it). There can be more than one reviewer, but for simplicity we'll talk about one.

A **changelist** is the set of changes the author wants to merge into the main codebase. The review starts when the author sends the changelist to the reviewer.

A **round** is one complete cycle: the author sends changes → the reviewer responds with notes. A single review can have multiple rounds.

**LGTM** ("looks good to me") is the reviewer's approval. The review is complete when the reviewer gives LGTM.

---

## Part 1. Setting up the process

### 1. Let computers do the boring work

A reviewer's attention is a scarce resource. Don't spend it on things a machine does better.

| Task | Tool |
| --- | --- |
| Build and automated tests | CI: Travis, CircleCI, GitHub Actions, etc. |
| Formatting | Formatters: ClangFormat, gofmt, Prettier, Black, etc. |
| Unused imports, unused variables, common bugs | Linters: pyflakes, JSLint, ESLint, golangci-lint, etc. |

Wire these checks directly into the pipeline (pre-commit hooks, required CI checks on PRs). If the author can "forget" to run them manually, they will, and you'll end up catching the trivia by hand.

### 2. Style debates are settled by the style guide

Arguing about braces, line length, and naming in every review is a waste of time. The team has a single style guide: **[LINK]**. Read it before your first review.

How to use it during a review:

- If the author wrote something off-guide — refer to the specific rule, not your personal taste.
- If the guide doesn't cover the rule — it's probably not worth arguing about.
- If it is worth arguing about — bring it to the team and codify the decision in the guide. The style guide lives in git, and changes go through a normal review. Once it's settled, we don't reopen the discussion.

---

## Part 2. Running the review

### 3. Treat the review as a high-priority task

When you're actually reading the code and writing feedback — **don't rush, but start the review immediately**, ideally within minutes. Starting immediately creates a virtuous cycle: review turnaround becomes purely a function of the changelist's size and complexity. This pushes authors to send small, narrowly-scoped changelists. They're easier and more pleasant to review, so you review them faster, and the cycle continues.

**The hard cap on a single review round is one business day.** If a higher-priority task is blocking you and you can't complete the round in under a day, let your teammate know and give them the chance to reassign the review to someone else. If you have to decline reviews more than about once a month, that probably means the team needs to slow down to keep development practices sane.

### 4. Start high-level and work your way down

If you're worried about drowning the author in notes, **stick to high-level comments in the early rounds**. Focus on things like redesigning a class interface or splitting up complex functions. Wait until those are resolved before moving on to smaller issues like variable naming or the clarity of code comments.

Low-level notes often become moot once the author addresses the high-level ones. Deferring them to a later round saves you the work of writing carefully-worded comments and saves the author from processing notes that no longer apply. It also segments the layers of abstraction you focus on, helping both of you work through the changelist clearly and systematically.

### 5. Be generous with code examples

If you ease the author's load by **writing out some of the changes you're suggesting**, you demonstrate generosity as a reviewer.

This isn't limited to one-liners. You can spin up your own branch to show a larger proof of concept — for example, splitting a large function into pieces, or adding a unit test that covers an additional edge case.

**Limit yourself to two or three code examples per round.** If you start writing the entire changelist for the author, it signals that you don't think they're capable of writing their own code.

### 6. Never say "you"

**Make it clear you're critiquing the code, not the coder.** When the author sees "you" in a comment, their attention shifts from the code back to themselves, and the risk of taking the criticism personally goes up.

Fortunately, it's easy to rewrite feedback to avoid "you".

**Option 1: replace "you" with "we".**

Before:
> Can you rename this variable to something more descriptive, like `seconds_remaining`?

After:
> Can we rename this variable to something more descriptive, like `seconds_remaining`?

The pronoun "we" reinforces the team's shared ownership of the code. The author may move to a different company, and so might you, but the team that owns this code will keep existing in some form. It can sound a bit silly to say "we" when it's clearly the author who's going to do the work — but silly is better than accusatory.

**Option 2: drop the subject from the sentence.**

Before:
> Can you rename this variable to something more descriptive, like `seconds_remaining`?

After:
> Suggest renaming to something more descriptive, like `seconds_remaining`.

Or:
> This variable should be renamed to something more descriptive, like `seconds_remaining`.

Or:
> What about renaming this variable to something more descriptive, like `seconds_remaining`?

### 7. Frame feedback as requests, not commands

In your feedback, **err on the side of being too gentle rather than not gentle enough**. Phrase notes as requests or suggestions, not commands.

| Bad | Good |
| --- | --- |
| Move the `Foo` class to a separate file. | Can we move the `Foo` class to a separate file? |

Requests also make it easier for the author to push back politely. Maybe they have a good reason for their choice. If you frame your feedback as a command, any pushback comes across as disobedience. If you frame it as a request or a question, the author can simply respond.

### 8. Tie notes to principles, not opinions

When you send a note, explain both the suggested change and the **reason** for it. Instead of saying "We should split this class in two", say: "Right now this class is responsible for both downloading the file and parsing it. We should split it into a downloader class and a parser class, following the single responsibility principle."

Grounding your notes in principles steers the discussion in a constructive direction. When you cite a concrete reason — like "We should make this function private to minimize the class's public interface" — the author can't just reply "No, I prefer my version." Well, technically they can, but it'll look weak, because you've shown how the change satisfies a goal while they've only stated a preference.

Where possible, back your notes up with links. The best link is the relevant section of your team's style guide. You can also link to language or library documentation.

### 9. Aim to raise the code by one or two letter grades, not to perfection

**Don't try to turn D-grade code into A-grade code in one review.** Bumping it up by one or two letters is enough.

This won't drag down the codebase, even though it might feel like it would. When you help a teammate go from D to C, the next changelist they send is very likely to start at C. Within a few months, changelists will start at B and end at A by the close of the review.

**An F is reserved for code that's either functionally incorrect or so tangled that you can't be confident it's correct.** The only legitimate reason to withhold approval is if the code stays at F after several rounds.

### 10. Limit feedback on repeated patterns

If several of the author's mistakes follow the same pattern, **don't flag every single instance**. It's fine to call out two or three. Beyond that, just ask the author to fix the pattern as a whole rather than each individual occurrence.

### 11. Respect the scope of the review

Don't let a small changelist mutate into a refactor of half the project. If the reviewer starts pointing at code *near* the changes, the author fixes it, then something else nearby surfaces, then something else — and a narrowly-scoped changelist explodes into a pile of unrelated churn.

**Rule of thumb: if a line wasn't changed in this changelist, it's out of scope.**

Even if there's an awful magic number and a horrible variable name right next to the changes — out of scope. Even if the author is the same person who wrote that nearby code — still out of scope. If the surrounding code is egregiously bad, file a separate ticket or send your own changelist with a fix, but don't dump it on the author in this review.

**Exception:** the changelist affects the surrounding code without touching it directly (for example, a function got renamed from `ValidateAndSerialize` to `Serialize`, and the old call sites are now incorrect). That's in scope and needs to be fixed.

If you don't have many notes anyway and there's an easy fix just outside the scope, you can mention it — but **always** tag it as `(out of scope, optional)` so the author knows your approval doesn't depend on it.

### 12. Look for opportunities to split up large reviews

**A changelist of more than ~400 lines should be split.**

Help the author do the splitting. When the changelist touches several files independently, you can simply break it up into smaller groups of files. In trickier cases, find functions or classes at the lowest layer of abstraction. Ask the author to move them into a separate changelist, then come back to the rest of the code after the first one is merged.

### 13. Offer sincere praise

For example, imagine you're reviewing for an author who struggles with documentation, and you come across a clear, concise function comment. **Tell them they nailed it.** They'll improve faster if you let them know when they got it right, instead of only flagging the things they got wrong.

If the author is a junior developer or newly joined the team, they're likely to feel nervous or defensive during a review. **Sincere compliments ease that tension** and show that you're a supportive teammate, not a strict gatekeeper.

### 14. Approve when the remaining fixes are minor

Some reviewers mistakenly think they should withhold approval until they see fixes for every single note. **This causes unnecessary review rounds and wastes time for both author and reviewer.**

Grant approval when any of these are true:

- you have no more notes;
- the remaining notes are minor (e.g., renaming a variable, fixing a typo);
- the remaining notes are optional suggestions. Mark them clearly as optional so your teammate doesn't think your approval depends on them.

I've seen reviewers withhold approval because the author missed a period at the end of a code comment. Don't do this. It signals to the author that you don't think they're capable of adding basic punctuation without supervision.

### 15. Handle stalemates proactively

**The worst possible outcome of a review is a stalemate:** you refuse to approve the changelist without further changes, and the author refuses to make them.

Signs you're heading toward a stalemate:

- the tone of the discussion is escalating, becoming tense or hostile;
- the number of notes per round isn't going down;
- you're getting pushback on an unusually high share of your notes.

#### 15.1. Concede or escalate

Weigh the cost of just approving the changes. You can't build quality software by quietly accepting low-quality code, but you also can't reach high quality if you and your teammate fight so badly you can't work together anymore. How bad would it really be to approve this changelist? Is it code that could potentially destroy critical data? Or is it a background process that, in the worst case, will fail and require some debugging? If it's closer to the second option, consider just conceding so you can keep working with your teammate on good terms.

If conceding isn't an option, talk to the author about escalating to the team's manager or tech lead. Offer to reassign the review to someone else. If the escalation goes against you, accept the decision and move on. Continuing to fight will drag out a bad situation and make you look unprofessional.

#### 15.2. After a stalemate — fix the relationship, not the code

Messy review arguments are usually less about the code and more about the relationship between the people involved. If you reached or nearly reached a stalemate, the situation will repeat unless you address the underlying conflict.

- **Talk to your manager.** If there's a conflict on the team, your manager should know about it. Maybe the author is just hard to work with. Maybe you're contributing to the situation in ways you don't realize. A good manager will help both of you sort it out.
- **Take a break from each other.** If possible, avoid sending each other code reviews for a few weeks until things cool down.
- **Study conflict resolution techniques.**

---

## Pre-send checklist

Before you hit "send", run through this list:

- [ ] Are there notes that should have been caught by linter/formatter/CI?
- [ ] Are style debates grounded in the style guide, not personal taste?
- [ ] In the first round, is the focus mostly high-level (architecture, responsibilities) rather than nitpicks?
- [ ] Is the word "you" absent everywhere?
- [ ] Are notes phrased as requests rather than commands?
- [ ] Does every non-trivial note come with a reason (principle, link)?
- [ ] Is anything out of scope tagged as `(out of scope, optional)`?
- [ ] If the remaining issues are minor — has approval been granted?
- [ ] Is there at least one sincere "I liked this" (when there's something to praise)?
