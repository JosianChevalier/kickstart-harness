---
name: review-walkthrough
description: Trigger — "review", "walkthrough", "walk me through", "what moved", "what should I compare"; also proposed at the end of each stage. NOT an exhaustive review, NOT a study.
---

# Review walkthrough

## Doctrine

- The author explains; the reviewer judges. Here you are the author: you drove the change. The user is the reviewer of the walk.
- The walk is a guided walk, not a report given all at once. The user can stop, ask, take a side path and return. You keep the route and the current position visible, so the user does not have to remember them.
- The walk gives the concepts first, so the code pass afterwards is recognition: every hunk already known, what it does, why, how. If the user has to open the code during the walk, the walk failed.
- Progress means the user had the chance to understand and judge a beat, not that you printed it.
- A commit is the unit the backend reviewer reads. The walk follows concepts, not commits; it still covers every commit, so the user can face that reviewer.
- A walk done after autonomous work stays a walk: one beat per turn, never a monologue.

## The walk

```
build the map ──▶ opening ──▶ beat ──▶ beat ──▶ … ──▶ all beats visited
(silent)          verdict,    one per turn,          ledger offered
                  doubts,     the user steers:       for the code pass
                  map         next, a number,
                              back, map, a question
```

## Build the map, in silence

1. Concept. From the target (the stage's north star; the diff when there is none), draw the change as a graph of beats in domain order: the flow the change lives in, the core that realizes the north star, the ripples it forced, the frictions. Edges carry their kind: forced by, flows to, friction with, named as the change needs. A beat may be purely conceptual when understanding needs it.
2. Anchor. Bind the beats to the code where code exists. Every hunk belongs to exactly one beat. Name its commit for each hunk. Cover every commit, against its parent: a change made in one commit and changed again in a later one is hidden in the total diff, and it still belongs to a beat. For an uncommitted change, name the base it is compared to; do not invent intermediate states for a split that was only proposed.
3. Group. Moves safe by construction go together into one closing beat.
4. Route. A default order of telling: the verdict first, then domain order. Say each thing you know when it becomes relevant. The route follows concepts and dependencies, not file names or the raw `git log`. Commit labels link the walk to a review commit by commit; they do not dictate the order. Inside a beat, show how an earlier commit prepares what a later one uses.
5. Persist. The map, the route, the hunk ledger and the visited beats go in `$TMPDIR/walkthrough-<branch>.md`. Read it again at each turn: it is the walk's state. Show it when asked.

The input, for a stage: the commits since the stage's base commit (in the notebook), and the stage reviewer's findings when it ran.

Hunks that fit no beat: the change was not one batch, or the story is incomplete. Interpret, recommend (split, revert), and show it in the opening turn. The user decides.

No domain flow (a cross-cutting change): the graph is the core and what it forced.

## Opening turn

- The verdict and the north star.
- ⚠ doubts on the whole change: hunks with no beat, several concerns in one change, the stage reviewer's findings.
- The map, at a scale that fits a few lines. Cluster it when it is wide. Use a clustered list when the graph is large.
- The beat count and the first stop. Stop.

## Each turn: one beat

    Beat 3 of 6 · Discount

    READ: behaviour changes as asked

    Finance needs the discount to shrink the taxable base.

    ⚠ Rounding now happens twice; totals may differ by a cent.

    <snippet>

      ← forced by   Pricing (core)
      → forces      1. Invoice total   2. Tax report
      ~ friction    3. Rounding policy

    next: Invoice total

The parts of a beat:

- Tag: what the user does with the beat.
  - `GLANCE`: safe by construction (rename, extract, inline, move to a file). Confirm the shape: every occurrence changed, nothing else moved. Give the anchors Old → New; a set moved together is one anchor.
  - `READ`: the behaviour changes as the target asked.
  - These are archetypes. A beat that fits neither says so plainly, and is read. A move that needs meaning to validate is `READ`. An uncertain change is read, even when called a refactor.
- Context: one piece of information the snippet cannot carry: the cause, the domain reason, what was there before.
- ⚠ and a plain reason, on any beat, wherever there is doubt: the behaviour may differ beyond what the target asked, or you are not sure.
- Snippet: verbatim, cut with `…`, the smallest part that lets the user recognize the code later. The context comes before it. Reverse the order only when the question is whether the code speaks for itself, and say so.
- Test: the test that proves the beat, and what it does not prove.
- Links: the beat's direct neighbours, by edge kind, numbered.
- Commits below the author's level: say what the beat protects and why the backend reviewer will want it, in the user's words. Show the code only as something to recognize later.
- Beats are named by concept. File locations live in the map file, and are given when asked.
- A blank line between the parts.

Stop after each beat. Do not add the rest of the tour to save turns. The user says:

- `next`: follow the route.
- a number: follow a link.
- `back`: return to the previous beat.
- `map`: show the whole, with the you-are-here.
- a question: answer it before you continue.

After a side path, `next` resumes at the first beat on the route not yet visited. When the user loses the relation to the whole, zoom out, then return to the same position. A question or a correction is not permission to move on.

The user corrects a domain assumption: update the map file at once, and drop the doubts it makes stale.

All beats visited: say so, and offer the ledger for the code pass.
