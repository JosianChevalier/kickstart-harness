---
name: plan
description: Trigger word: "plan" — "make a plan", "plan <change>". Also when a need is not trivial to implement, or when evidence breaks the route of an existing plan.
---

# Plan

## Doctrine

- A plan is a session, not a file request. The conversation is the planning act. The file is the working artifact of the task, once the calibration has validated its shape.
- The user validates from chat, at the altitude of the map, never by reading the file. The file is for agents: the session of each stage reads it.
- A plan is a suite of stages. Each leaf stage is one session. Each stage ends potentially a release: working, its tests passing, zero regression. A stage is a suite of commits; the plan does not plan the commits.
- The rhythm is "make the change easy, then make the easy change" (Kent Beck). A stage reshapes the code without a change of behavior, or a stage changes behavior on prepared ground. One stage never does both when the reshaping is not trivial.
- A plan is read without its details. The map says what the whole is about. Each north star says what its stage is about. The north star is the part that matters most.
- The plan states intent at every level, never the how. The session of the stage owns the how.
- The plan holds the decisions the user took. A guess written in the plan becomes a decision nobody took. Never take a domain or design decision for the user: draw the choice on one real case, say which way you would take and why, and let the user decide.
- The plan is the steps to come. It is not a conspiracy board (the Pepe Silvia meme), and it does not keep score. A done stage is one line. What matters for later lives only in the stage it concerns, and leaves with it.
- The plan is direction, not a contract. Stages are rewritten, reordered, split or dropped as the work teaches.
- Small batches: the inventory between two stages is measured in hours. A big stage is a hidden big batch.

## When to plan

- A need that is not trivial to implement gets a plan, after its study when it had one. A really simple need, or a need simple to implement, is one stage, without a plan.
- Evidence that breaks the route of an existing plan reopens that plan. Rework the same file. Never write a second plan beside the first.
- Before the first response, read the goal, the nearest evidence in the code, and any study or plan already written for this need.
- An uncertainty that could change the shape of the plan returns to a study. The plan does not settle it by writing a guess.

## The session

```text
calibrate ──▶ skeleton ──▶ converge ──▶ working draft ──▶ review ──▶ checkpoint
model and     after a       decisions     the file follows    plan-reviewer  map and north
one question, user turn     land in       the shared model    subagent       stars in chat;
mic back                    chat                                             next: 1st stage
```

**1. Calibrate. Your first response, always.** Reply with:

- The working model, in 2 to 5 bullets, and a compact map.
- At most one question, and only if its answer could change the scope, the order of the stages, the architecture or the semantic contract. Name both answers and how the plan diverges between them. A question is consequential when its answer changes which concept the plan commits the code to; scope, order and architecture follow from that. The names of the stages, the style of the map and the format are medium: yours.
- Smaller consequential points as assumptions: "Assuming X unless you object."
- When nothing is consequential, say so: "No consequential unknowns; ready to write."

Your turn ends here. This response is the deliverable. Evidence in the repository resolves questions of implementation, not ambiguity of the domain. A term with several meanings in the codebase is consequential by default.

**2. Skeleton.** Write `docs/plans/YYYY-MM-DD-<topic>.md` only after a user turn that follows the calibration. An assumption the user did not object to is confirmed: do not ask it again. Start with the smallest useful skeleton: the goal, the map, the decisions taken, and the stages as a direction.

**3. Converge.** Discuss until the model settles. Everything needed to decide comes into chat. Never ask a question whose answer requires the user to open the file. Update the skeleton as each decision lands.

**4. Working draft.** Make the plan follow the shared model as it changes. Before each hand-back, update the file if a fresh session would otherwise lose a decision, a finding, an open thread or a real next step. Distill: never append a transcript.

**5. Review.** When the file is usable, run the `plan-reviewer` subagent with `task`. Give it the path of the plan file and nothing else: it reads the plan and the code, not your reasoning. It checks ordering and slicing, never scope or design. Apply its mechanical fixes (reorder, split, merge, nest) before the checkpoint. A FAIL you disagree with, or a fix that changes meaning or scope, goes to the user: never override it in silence. When the review cannot run, say so: the gate stays visibly unmet. Your own review is useful, but it is not an independent review.

**6. Checkpoint.** Render in chat: the map, each stage with its north star in one line, and the key decisions. Ask whether it matches. The user validates from chat, not by opening the file. Once validated, say that this session is done and that the next one is the first leaf stage, named. The plan stays a living artifact until the work ends.

**Standing rule: every hand-back opens with the map in chat.** From the skeleton on, every response to the user opens with the current map, drawn in the chat. Never a pointer to the file. Never "see the plan for details". The user checks direction and coherence from this map and iterates on it. Opening the file is the user's choice, never a need. Fast feedback, low cognitive load.

## The plan file

Every plan has a **goal**, with the condition that says the work is done, a **map**, and **stages**. Beyond that, add what the context needs: domain context, constraints, decisions, a diagram, references. Plans are shared context between sessions that do not share memory: concise, but precise.

### The map

The map is the human's view. It is mandatory, never skipped.

- It is a short summary of how the plan reaches the goal. It is self-sufficient: someone who has not seen the code can read it. It uses the words of the project's domain.
- It is not the list of stages at a lower resolution. It is a different level of information: it summarizes, and it may explain what the stages do not carry: why this slicing, what depends on what, where the risk is.
- Before you draw it, ask: what is the best way to represent this plan visually? An ASCII diagram, a pseudo-code sketch, a table, an arrow chain (`parse -> validate -> persist`), an indented list, or another compact shape that makes the structure clear at one glance. Do not default to arrows: use them only when the plan is truly linear. When stages are grouped, an indented list is the natural form.
- It is a you-are-here map. It holds the whole route: done, current, ahead. The current position is marked. How it is marked is yours: check marks, an arrow, a pin, whatever the form of the map carries at a glance.
- A done stage never leaves the map. It stays, marked done, and the position moves.
- When a stage is added, reordered, rewritten, dropped or done, the map changes in the same edit. A stale map is worse than none.

### A stage

A stage opens with its **north star**, at the top. It is the one required part of a stage. It is what the user is shown, so they know what the stage is about. It is one artifact that makes the aim of the stage clear at one look. Someone lost in the middle of the work, the user or an agent deep in a complex change, looks at the top of the stage and finds the direction at once.

- Ask for each stage: what is the best way to represent this stage visually? A request flow, an example interaction, a state transition, a pseudo-code snippet, a fragment of the target code, an ASCII diagram, a table, a before and after. Stages can differ: one needs pseudo-code, the next a dependency diagram, the next a before and after.
- The north star is intent, not prescription. The session of the stage owns the details.

Under the north star, a stage carries what helps the agent do it well, and nothing else: no fixed list of fields. No file lists. No step-by-step instructions.

A done stage collapses to one line, in place: what it delivered. Its north star and its details leave with it.

### Decisions

Record the decisions the user took, in the stage they concern, or at the top when they concern the whole plan.

Never record an unresolved question. Ask it in chat now. When it could change the shape of the plan, return it to a study.

## Ordering

**Stages and groups.** A leaf stage is the unit: one session, potentially a release. A functional outcome that needs several stages groups them: `1` is a group, `1.1` is a session, `1.2` is a session. A group is not a session. The group returns value and information to the user; its stages may be technical, but each one ends working, zero regression, live. One level of grouping is enough most of the time. A stage that cannot end working and live is too large, or it is sliced on the wrong axis: split it, never enlarge it.

**Stage ends are observable saved points.** The safety invariants hold at every stage end. When a stage cuts through domain state, name with the user the consistency boundary that must hold (the aggregate), and the invariant it protects.

**Slice across layers.** Do not plan "database, then domain, then API" unless each layer-only stage is a reshaping that preserves behavior. Prefer thin end-to-end increments over batches of architectural layers. A stage that touches one layer is legitimate only inside a functional outcome it serves.

**Intent at every depth.** No file lists, no step-by-step instructions, in a group as in a stage. Depth is not detail: a stage is narrow in scope and still stated as intent.

**Expose last: the keystone stage.** A public contract goes live only after the behavior behind it works and its tests prove it. A public contract is an API surface, a contract with the front end, a published name, a documentation. Documenting or advertising a contract is exposure: a plan with API documentation before the backend stages that make it real promises a behavior that does not exist. The order: behavior effective, then verified, then exposed and documented, then the old behavior removed. One exception: a goal that is explicitly contract-first, where the plan states that the contract carries no claim of implementation.

```text
make the change easy
        ↓
implement the effective behavior
        ↓
verify it
        ↓
expose or document it
        ↓
remove the obsolete behavior
```

**Follow the rhythm.** Do not mix a non-trivial reshaping with a change of behavior in the same stage. First make the change easy, with a stage that reshapes and preserves behavior. Then make the easy change, with a small behavior stage on prepared ground. When the implementation reveals new structural debt, add a cleanup stage after the behavior stage.

**Change in place.** Reshape an existing live workflow on the live path, as a chain of behavior-preserving moves: each stage transforms the code that runs, and stays live. Never plan a dark replacement of a live workflow followed by a "switch" stage: that is the big batch. An alternative flow (an adapter, expand/contract, a parallel branch behind a flag) is allowed only for the smallest atom that cannot be transformed in place without regression, and that atom goes live in the same stage once parity is proven. The keystone order (dark first, expose last) is for new behavior that has no live path yet.

**Just in time.** A stage introduces only what is on the live path by the end of that stage or the next one. No stage defines contracts, seams, interfaces or helpers for later stages to consume: that is inventory, and later stages bend to fit the guess. When a stage needs a contract, that stage creates it and uses it.

**Producers before consumers.** The implementation is ready before anything calls it.

**The coach's concerns are not planned.** Race conditions, atomicity, persistence, design, security, REST and API design run as gates once, on the diff of the stage, at its end. A design review may open each stage. The plan does not carry them stage by stage.

**Evidence informs, it never commits.** A plan downstream of a study, or of a prototype, keeps four things apart: what was observed, the decisions accepted, the semantics still under review with the user, and the stages. The code of a prototype is evidence, never a stage. When an alternative appears whose trade-offs resist discussion, return it to a study instead of arguing; the findings return to chat, and the plan records the decision.

**Mark siblings sequential or parallel.** Parallel only when their scopes do not overlap, so the user can run their sessions at the same time.

## Safe rollout on a live path

A stage end is potentially a release. Inside a stage, each commit keeps the branch working and its tests passing. When a stage modifies code on a live path, choose a strategy that keeps the existing flow safe at every stage end, and at every commit.

### Is the flow live?

- **Yes** when the code is already called from an endpoint, a scheduler, a message consumer, a command, or any other entry point of production.
- **No** when the code is new and no entry point reaches it yet. Then stubs and skeletons are fine: nothing triggers them.
- Never assume "it will not be called yet": verify the call chain.

### Strategies: pick the simplest that works

**1. No-op adapter.** When a port gains a method for observability or an optional side effect, and the flow works fine without it. Ship the no-op, then replace it with the real implementation later.

```python
class CrmNotifier:
    def customer_moved(self, customer_id: CustomerId) -> None:
        pass  # Intentionally empty: the real call comes in the next step.
```

**2. Safe default.** When a port gains a method that returns a value, and the flow needs a sensible answer.

```python
class ConfigSource:
    def fetch_config(self, customer_id: CustomerId) -> Config | None:
        return None  # Safe default until the real adapter is ready.
```

**3. Feature flag, dark launch.** When existing behavior is replaced, and the old and the new must coexist until the new path is validated.

- Deploy the new code behind a flag (a configuration value, a toggle in the database).
- Activate it in production once validated.
- Remove the flag and the old code in a later step.

**4. Parallel flow, expand/contract.** When a flow migrates and cannot switch atomically.

1. Expand: add the new flow beside the old one. Both run; the old one is authoritative.
2. Validate: compare the results, watch for discrepancies.
3. Contract: remove the old flow once the new one is proven.

A typical order to modify a port on a live flow, each step working:

1. Implement the new method of the adapter: no-op, safe default, or real.
2. Make the domain call it.
3. Remove the old error handling or fallback that is now redundant.

### Anti-patterns

- `raise NotImplementedError` on a live path.
- A `# TODO` that returns wrong data in silence.
- Old behavior removed before the new behavior is proven.
- "It will not be called yet", assumed without checking the call chain.

## Keeping the plan current

Update the plan when:

- The user gives feedback that changes the approach.
- The user takes a decision that affects how the work will be done: update it at once.
- A study, a probe or a discussion changes what the next session needs to know.
- A stage ends: it collapses to one line, the map marks it done, and the position moves to the next stage.

Every change to a stage updates the map in the same edit. When control returns to the user, the session could end: the plan already holds the current state.

## Examples

Two families of failure. **Interrogation**: the medium sent back to the user (a questionnaire, a forced question). **Silent choice**: a concept decided without the user (a silent guess).

VIOLATION, same-turn write: the agent states an assumption, then writes the file in the same turn. To disclose is not to validate: the calibration ends the turn.

VIOLATION, questionnaire: "Before I start: 1) Which format? 2) Sequential or parallel stages? 3) Any constraints?" This is not planning. It hands the triage to the user. A question you could have resolved by triage is a failure, not diligence.

VIOLATION, forced question: "Should the stages be numbered or named?" No divergence in scope, order or architecture. Asking it shows the triage was skipped, and caution performed.

VIOLATION, silent guess: the user says "plan the sync feature". The agent picks one-way sync without saying so and delivers a polished plan. A clean artifact built on an unstated interpretation is the worst outcome: it looks done, and it is wrong. Worse than any question.

VIOLATION, flat plan: "Stage 1: importer end-to-end (schema, parser, validator, diff, API)". One stage, five layers, feedback only at the end. Correct:

```text
1 the importer never corrupts the ledger
  1.1 schema and parser, live behind the existing path
  1.2 row validation, with a report of invalid rows
  1.3 dry-run diff
  1.4 commit path
```

VIOLATION, dark replacement: "Stage 1: extract the resolver seam. Stage 2: extract the matcher seam. Stage 3: extract the merger seam. Stage 4: switch the live routing to the new seams." Three stages of inventory, one big-bang integration. Correct: each stage extracts one seam from the live path and uses it live in the same stage. There is no switch stage.

VIOLATION, inventory of future contracts: "Stage 1: define the interfaces for selection, scoring and merge that stages 2 to 5 will implement." Stage 1 ships guesses about shapes nothing uses yet. Correct: stage 1 introduces only the first seam it uses; later stages create their own contracts when they consume them.

VIOLATION, stale map: a probe swapped stages 3 and 4. The stages were rewritten, but the map kept the old order. The user validated a direction from an obsolete map. The map and the stages change in the same edit, always.

VIOLATION, map as forecast: stage 1.1 is done, and the agent deleted it from the map. The next session sees a route with no origin and cannot tell where it is. Done stages stay, marked done; the position moves.

VIOLATION, guess as decision: the user did not say whether a retried payment may charge twice. The agent writes "payments are idempotent by request id" in stage 2. It reads as settled, and nobody settled it. Correct: ask it in chat now, with both answers and what changes. The plan records the answer once the user gives it.

Sharp (good): "Assuming 'sync' means a one-way export to the CRM. If it is two-way, we need conflict resolution, and the stages roughly double. Which one?" Two answers, the divergence named.

Correctly silent (good): the need fixes the API, the storage and the rollout. "No consequential unknowns; ready to write." The turn ends. The user's "go" opens the skeleton.

Good maps (choose by fit, not by habit):

```text
the importer never corrupts the ledger
fetch -> parse and validate -> dry-run diff -> commit path -> expose API
the dry run exists so the damage is visible before anything writes
```

```text
                 +------------------+
incoming file -> | quarantine parse | -> report of invalid rows
                 +------------------+
                           |
                           v
                    valid ledger diff -> reviewed commit
```

```text
for each inbound event:
  normalize it into a canonical command
  reject it if it crosses an account boundary
  save the command and its audit record in one transaction
```

A good map during the work (one way to mark the position; pick what fits the form, it only needs to say "you are here"):

```text
  1 the importer never corrupts the ledger
✓   1.1 schema and parser, live behind the existing path
✓   1.2 row validation
→   1.3 dry-run diff               damage visible before anything writes
    1.4 commit path
  2 expose the API                 after 1 is verified
```

Good north stars (different by stage):

```text
Stage: keep the current behavior while the seam moves

old handler -> adapter -> existing service
new handler -------^
```

```text
Stage: make a duplicate submission idempotent

if submission_id is already processed:
  return the stored result
else:
  validate -> charge -> persist the result
```
