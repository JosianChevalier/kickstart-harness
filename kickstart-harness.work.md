---
ongoing_work_file: "The kickstart harness for OpenCode, built from the base harness: empty when every element is walked with the user, built, and the repository is sent to its users."
rank: 1
---

# kickstart-harness

An OpenCode harness that takes a front-end developer, knowing neither Python nor backend, from a need they explain to a branch of small commits a backend reviewer can read. Built by copying the base harness (`~/.claude`, its `old-claude/` folder included), deviating as little as possible.

```text
NEED (the user explains it; no ticket, no autonomous agent)
  |
  +-- not trivial to understand? --> STUDY      one session
  +-- not trivial to implement?  --> PLAN       one session
  |                                  map + stages, a north star each
  v
STAGE 1 ............ one session, potentially a release
  commit -> the user validates -> it lands        (gate)
  commit, commit, commit   below the author's level, in a row
  review walkthrough, proposed
  "this session is done, the next one is stage 2"  --handover-->
STAGE 2 ............ a new session
```

## The model, settled with the user

### The cycle

| The need is | Study | Plan | Stage |
|---|---|---|---|
| really simple | no | no | one |
| simple to implement | yes | no | one |
| neither | yes | yes | several |

- The plan is a suite of stages, each potentially a full release, in the rhythm "make the change easy, then make the easy change". It is read without its details: the map says what the whole is about, each north star what a stage is about. The north star is the part that matters most.
- A stage is a session; a session is a suite of commits.
- A review walkthrough is proposed at the end of each stage.
- A session ends in two ways: at a seam, the agent says the session is done and names the next one; on context too high, it is closed by hand with a handover.

### Commits

- One reviewable change per commit, small: a unit of thought, a quick functional change or a quick refactor.
- Every commit leaves the branch working, its tests passing; the tests come with the change they prove.
- The subject follows Conventional Commits (`feat:`, `fix:`, `refactor:`), as the users' repositories already do.
- On a branch, not trunk-based: no shippable-branch or continuous-delivery discipline.
- A commit lands automatically once the user validates it.
- The body says why; and how, when the how is complicated.
- Two readers. The author follows what they can; the reviewer knows backend and goes deeper. Commits are cut for the reviewer, so several can be made in a row below the author's level.
- Atomic refactoring stays out; it has its own repository, offered on the side.

### Three primaries

- backend coach: the cycle above.
- mecha suit: the basic collaborative one, copied.
- robot: the autonomous one, copied.

### The backend coach

- Managing the user's cognitive load is the alpha and the omega of the harness.
- It never takes a domain or design decision for the user; it guides them through it.
- It teaches fundamentals and patterns, not syntax: low-abstraction coding may soon be gone.
- Everything is an opportunity to learn, offered, never pushed.
- The base prompts keep every idea they hold: GPT needs each one spelled out, nothing is condensed.
- A security flaw met in existing code is not fixed on the spot. It is made visible at once, marked ⚠, in one line: what an attacker could do, and that it will be patched. It goes in the notebook, and every flaw noted there is handled before the session ends.
- Concerns it checks, somewhat complete at first, tweaked by its users afterwards:
  - race conditions, atomicity, persistence
  - general design, architecture patterns, dependency inversion, layers of abstraction
  - security
  - REST and API design

### The build

- The philosophy layer stays out; the doctrine / know-how split is kept in each element.
- Know-how is not cut, and a little is added: OpenCode with GPT models needs it.
- No TDD. A full testing practice is its own later point.
- Everything that ends up in the harness, every prompt, rule, skill and agent, copied or new, is written in plain, basic, terse English: short sentences, common words, no idiom. Not every user is fluent in English, and GPT writes in the register of its context, so the instructions are the example. Attractors stay precise, always: the field's terms and its names (cognitive load, race condition, dependency inversion, Tufte) are never simplified.

## Elements

Each is walked with the user before it is built: added, removed or updated. Origin: `=` copied as is, `~` copied then localized (Claude tool names, `φ` rationale lines, dead references), `*` new.

| Element | Origin | Source in `~/.claude` | To settle |
|---|---|---|---|
| system prompt | = | `prompts/system.md` | |
| doctrine prompt | ~ | `prompts/doctrine.md` | "Cartographer" and "Whiteboard" leave for the four answer prompts; in their place, the two paragraphs below the table |
| answer prompts: map, draw, hierarchy, write-lean | * | `prompts/doctrine.md`, `rules/writing.md` | the four steps of an answer to the user, one file each in `prompts/`, chained in the three primaries. Open: the probe to rework, with the full chain in the system position and on GPT (a blank Sonnet given the four alone follows them); the word "folded" leaks into answers as jargon; two ⚠ per drawing where one is asked |
| mecha-suit prompt | ~ | `prompts/mecha-suit.md` | |
| autopilot prompt (robot) | ~ | `prompts/autopilot.md` | "go to the end" against a stop at a security concern |
| backend coach prompt | * | `skills/strategic-design`, spec §12 | walked, written in `prompts/backend-coach.md`; chained after the mecha-suit prompt |
| rules: writing, session-design, subtraction, documents | = | `rules/` | the writing rule's chat section duplicates the four answer prompts: one home to choose |
| rule: delegation | ~ | `opencode-rules/delegation.md` | it says not to commission independent review |
| rule: security and tool boundaries | * | spec §15; `prompts/doctrine.md` holds the cybersecurity posture | permissions: none declared in the base |
| skill: study | ~ | `skills/study-making`, `old-claude/skills/prototyping` | |
| skill: plan | ~ | `old-claude/skills/write-plan`, `safe-rollout` | it forbids recording unresolved questions |
| skill: stage execution | ~ | `old-claude/skills/mob-from-plan` | reworked on the commit cycle above. It settles the mecha suit's "nothing enters the code that a navigator did not say" against commits made in a row below the author's level |
| skill: commit | ~ | `old-claude/skills/atomic-commit`, `old-claude/agents/committer.md` | walked. Kept as written: what counts as one change and what does not, the red flags, the subject rules, stage by name, never amend, never push. Left out: the committer sub-agent with its wrapper script and hook, the triggers forcing TDD and atomic refactoring, "one refactoring move" as a unit, gitmoji. The gate (validated, then it lands) belongs to the stage, with the coach |
| skill: review walkthrough | ~ | `old-claude/skills/review-walkthrough` | |
| skill: testing | ~ | `old-claude/skills/test`, `in-memory-adapters`, `old-claude/rules/test-types.md`, `no-mockito.md` | Java and Kotlin, to redo for Python; evidence reporting and concurrent requests have no source |
| skill: clean code, Python | = | `skills/clean-code-python` | |
| skills: ongoing-work, handover | = | `skills/` | |
| skill: kaizen | ~ | `skills/kaizen` | depends on the philosophy scout |
| skill: ascii | * | a third in `rules/writing.md`; spec §17 | |
| agents: plan reviewer, stage reviewer | * | checklist in `write-plan` phase 5; spec §10 | whether they stay |
| acceptance of the harness | * | spec §16 | to check: drawings use the project's domain, not the ticket shop of the whiteboard's examples |
| tools: `run`, `grab-notebook`, private tmpdir plugin | = | `tools/`, `plugins/private-tmpdir.ts` | |

The doctrine paragraph naming the four steps:

> Every answer to the user is a pipeline of four steps. Each step takes what the step before returns.
>
> 1. **Filter and group**, like a map: keep what the reader needs now, group the rest into clusters.
> 2. **Represent visually**: draw all that can be seen. It reduces the extraneous cognitive load (Sweller).
> 3. **Set the hierarchy**, of information and of the page: the point first, the big labels readable alone.
> 4. **Write lean**: say what is left, in few and plain words.

The doctrine paragraph on writing for an agent, a file or a brief. Only clutter matters there, not style:

> Writing for an agent, in a file or a brief: do not clutter it. Clutter is the disease (Zinsser). Stop when there is nothing left to take away (Saint-Exupéry). Every line must fire (Chekhov's gun): the reader acts on it, or it goes. What is done is dead: the code holds it, the file does not. No file is a bureaucrat's record. No file is for keeping score.

The spec is the user's first brief (`brief.md`), kept as the source of the new elements; where it and the model above disagree, the model wins.

OpenCode wiring, as the base does it: primaries declared in `opencode.json` as chains of `{file:./prompts/…}`, built-in `build` and `plan` disabled; subagents as markdown in `agents/`; rules through `instructions` globs, reaching subagents too; verified facts in `~/.claude/opencode-agents.work.md`.

## Steps

- [ ] Walk the elements, one per turn. ← next: plain language
  - [ ] plain language: how the whole level design pulls GPT toward it, in every file. The form that works, from the draw prompt: short sentences, one idea per bullet, the attractors kept.
  - [ ] lean files: GPT writes down all that was said and done, and work files reach thousands of tokens. Each file gets a good picture, a bad figure and a rule, in its own text; the doctrine holds the attractors. "Backlog" is fine; "changelog" is the word that pulls wrong. Probes need a GPT client, none on this machine.

    | File | It is | It is not | Rule |
    |---|---|---|---|
    | notebook | your backlog, your drafts | your secret diary | do not bloat it; an item leaves as soon as it is treated |
    | work file | what the conversation crystallized | a bureaucrat's record | nothing is written for the record |
    | plan | the steps to come | a conspiracy board (the Pepe Silvia meme); keeping score | a done step is one line; what matters for later lives only in the step it concerns, and leaves with it |
    | study | the ruling, not the trial: the conclusion and its consequence | keeping score; taken down in evidence; the file kept on someone | only what has an impact stays (Chekhov's gun) |

  - [ ] testing practice
  - [ ] the others, in table order
- [ ] Build, in stages each leaving a harness that runs: skeleton with the three primaries; cycle skills; coach and review elements; kaizen; acceptance in a clean OpenCode install.
- [ ] Send the repository to its users.
