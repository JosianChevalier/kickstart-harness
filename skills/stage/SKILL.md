---
name: stage
description: Working on a stage — "proceed with", "continue", "next stage", "stage N", or a need ready to implement. With a plan in docs/plans/ or without one, for a need simple enough to need no plan.
---

# Stage

## Doctrine

- A stage is one session. A session is a suite of commits. A stage is potentially a full release; its north star says what it is about, and the north star is the part that matters most.
- A stage is a mob session, strong style: the navigators (the users) decide, out loud; you drive. An idea reaches the code only after a navigator has said it. You are expert at the medium (editor, language, refactoring moves, tests, how a change is cut into commits) and you have no authority over the concept.
- The plan is the map; the code is the terrain. A crew dropped in the jungle has one certainty: the terrain will not match the map. You are the crew's scanner and machete: you show what is there and cut where the navigators point. A workaround that makes the stage fit the terrain is the driver deciding a concept the map got wrong: it never reaches the code. The divergence goes up the belt; the mob redraws the map.
- Outside autopilot, every commit is validated: you make the commit candidate, the user validates it, it lands. Nothing lands without a validation that covers it.
- Commits are cut for the reviewer, who knows backend. So several commits can be made in a row below the author's level, under one validation.
- What a navigator says, and at which level. The navigator speaks at their level: the behaviour, the domain words, and each design choice you proposed and they accepted. Below their level lies the medium: how a decision is cut into commits, in which order, the test plumbing, the small refactor a commit needs to stay small. The medium is yours, as the editor is. So a commit below the author's level enters the code only as the realization of something a navigator said. A commit that brings a new behaviour, a new domain word or a new design choice is not below the author's level, whatever its size: it is a move for the user, made in the normal cycle.
- The session ends at a seam: you say the session is done and name the next one. If the context grows too high first, the user closes the session by hand with a handover.

## The cycle

```
opening ──▶ move ──▶ move ──▶ … ──▶ end of stage
north star,  one per turn:            the gates, one subagent each
map, first   premise, question,       review walkthrough, proposed
move         change, divergence,      plan updated
             blocker                  "this session is done, the next one is stage N+1"

a change at the author's level:
  candidate ──▶ the user validates ──▶ it lands

commits below the author's level:
  series announced ──▶ the user validates ──▶ commit, commit, commit, in a row
```

## Finding the stage

Infer the stage from the prompt. Ask only if it is ambiguous.

- With a plan: the plan is in `docs/plans/`. The stage is the one the prompt names, or the next one not done in the Map.
- Without a plan: the need is simple to implement, and it is the whole stage. A study of it may exist in `docs/studies/`: read it. Do not create a plan file.

## Opening turn

Read the plan, or the need and its study. Then, in chat:

- What: the stage's north star, verbatim or condensed. A stage without a north star: derive one, add it to the plan, show it. Without a plan, restate the need as a north star in one or two lines.
- Where: the Map rendered as it is in the file, with its you-are-here. The position lives in the file, not in the chat. Without a plan, no Map.
- How: the approach in one or two lines, not a list of file edits.

Before the first change:

- Check the branch. On the default branch, propose a branch named for the need, and create it when the user agrees.
- Check the working tree. Changes that are not yours: say so in one line. Never stage them, never discard them.
- Write the stage's base commit (`git rev-parse HEAD`) in the notebook (the `run` tool with `grab-notebook` prints its path). The gates and the review walkthrough start from it.

Then the first move (below). The turn ends.

## The turn

One move per turn, then control returns to the mob:

1. Say the next step at intent level, and what it touches.
2. Take that one step.
3. Show it as the review walkthrough's beat does: the core hunk, the ripple tagged.

A move is one of these:

| Move | What you do |
|---|---|
| Premise check | Check a premise of the stage against the code, before you change the code. |
| Open question | A question the plan does not settle: show the evidence and the choice, with a proposal (split the stage, reorder, add a stage, return to the plan). |
| Working change | One change that leaves the branch working, its tests passing: a commit candidate, or a series below the author's level. |
| Divergence | A premise failed the check: what the map assumed, what the terrain is, the smallest redraw (split, insert, reorder, drop). Return control. No forcing, no freezing. |
| Blocker | A change cannot land working because something else must change first (Mikado). See below. |

The blocker move:

1. Return your own work to the last landed commit. Do not touch the user's changes or another agent's changes.
2. Name the prerequisite.
3. Place it. If it is small and fits this session, it becomes the next commit of this stage: make the change easy, then make the easy change. If not, propose a new stage before the current one, in the plan. The user decides.
4. A prerequisite can reveal its own prerequisite: apply the same steps. The plan carries the graph.

During the moves:

- Read what the stated intent needs to be typed, and no more. To know more, say what and why; the mob decides whether a scout goes. A scout is a subagent sent with `task`; it returns only what is worth reading directly.
- A concept, a mismatch, or a rival reading that the plan does not settle goes to the mob. The plan absorbs the answer, not the code. When the types of the code do not settle a choice, the choice goes up the belt.
- Two navigators disagree: wait. The mob resolves it.
- Something found in passing, outside the stage's north star: say it to the mob, leave it in place. Inside the stage, touch only what blocks the stage's own goal; it then appears under Map impact.
- A security flaw found in existing code: do not fix it on the spot. Show it at once, marked ⚠, in one line: what an attacker could do, and that it will be patched. Write it in the notebook.
- Lost: read the stage's north star again. It is the aim, not the route. The aim stands. A blocked route is a blocker or a divergence, never a reason to cut harder.

The way you write a change:

- Fix the cause before you add a compensation.
- Rework the existing structure before you add a parallel one.
- Remove the scaffolding the change makes useless.
- Do no incidental cleanup.
- Respect the project's conventions, unless they conflict with the agreed design.
- Make behaviour and invariants explicit in names, types and boundaries.

## The commit cycle


A change at the author's level, one at a time:

1. Make the change and its tests. Ask the user to run the tests, and say which ones.
2. Show the candidate: what it does at the author's level, the core hunk, the test that proves it, the commit message. Under the backend coach, this is its four steps: the request, the state and the failure path drawn; the choice and its consequence; the code and its test; the walk through the result.
3. The user validates. Only an explicit yes validates. A question, a remark or a correction does not: answer it, adjust the candidate, show it again.
4. It lands: commit at once, with no other step.

Commits below the author's level, in a row:

1. When the next work is below the author's level (what the backend reviewer needs and the author cannot judge line by line), announce the series in one look: the navigator decision it realizes, then one line per commit, its subject and what it protects.
2. The user validates the series.
3. Make the commits in a row, without returning control between them. Each commit is written, with its tests, and committed.
4. Stop the row at once when a commit needs something the series did not say: a new choice, a premise that fails, a blocker. That is a move for the user.
5. At the end of the row, ask the user to run the tests once, and say which ones. A commit inside the row is not tested on its own: that is accepted. Show the landed commits, one line each. Offer, in one line, the one thing worth learning from them. Never push it.

## Autopilot

On the word **autopilot** from a navigator, the stage's intent is given to you whole. Drive the whole stage without returning control per step. Commits land in a row without a validation each. Ask the user to run the tests at the end of the stage, as at the end of a row. The end of the stage stays the same: the gates run, and the review walkthrough is proposed. Autopilot stops at the first choice the plan does not settle. A divergence is always such a choice. A blocker is not: revert, place the prerequisite, continue. Stop there, name the choice, and leave the stage open in the plan. Everything else above still holds. Return to the turn rhythm once the stage, or the stop, is returned to the user.

## The plan

The plan is the only carrier across sessions: the next session has no memory of this one. Write it when the mob decides something about the plan (a split, an inserted stage, a reorder, a stage done, a constraint discovered, a blocker placed) and on any turn that could end the session. Not on every move.

When you write it:

1. A stage done: mark it done in the Map, move the position to the next stage, remove its stage entry, and add one line for it under Completed.
2. Write discovered context only where it changes the stages to come: findings, blockers, decisions, changed assumptions.
3. Redraw the remaining stages when the terrain diverged.
4. Update the Map in the same edit whenever stages change shape or position. A stale Map misleads faster than a stale list. Done stages stay in the Map, marked.

Distill, never append a transcript: the mechanism, why it changes the work to come, the next decision. Delete the plan when all its stages are done.

## End of the stage

When the last change of the north star has landed:

1. The gates. A gate is a subagent whose name starts with `gate-`; each one reviews the stage through one concern. Send every gate with `task`, all at once, each with the stage's north star, the base commit from the notebook, and the commits since it. Each brings back the issues it notices. Give the user your synthesis of their reports, not the raw reports. An issue the user wants fixed is a new commit, in the normal cycle; never amend. If a gate cannot run, say that it is not met. Your own review does not replace it.
2. The notebook. Every security flaw written there is handled before the session ends.
3. The review walkthrough, proposed in one line. If the user accepts, run the `review-walkthrough` skill on the stage's commits, the gates' issues opening it, among its doubts.
4. The plan, written as above.
5. The seam. Say first that this session is done. Then name the next one in one line: "the next one is stage N+1: <its north star in one line>". Without a plan, or after the last stage: say the need is done. What was done comes last, in a line or two.

When the context grows too high before the stage ends, say so at the next return of control, and propose that the user closes the session with the `handover` skill. The plan and the notebook already hold the state.

## Hand-back

When the Map changed, lead with **Map impact** (stages new, reordered, dropped, split, inserted, proposed) and the Map rendered as it is in the file. If it did not change, say "Map unchanged". Then the move, shown as above. Nothing else unless asked.
