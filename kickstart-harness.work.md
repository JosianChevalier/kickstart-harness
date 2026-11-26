---
ongoing_work_file: "The kickstart harness for OpenCode, built from the base harness: empty when the gate skills are written."
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
- Every commit leaves the branch working; the tests come with the change they prove. In a row or under autopilot, the user runs the tests once at the end: a commit inside is not tested on its own, and that is accepted.
- The message follows the Angular commit convention, the users' own; the body tells everything a reviewer needs: what and why, how, the details, what was dropped and why it lost.
- On a branch, not trunk-based: no shippable-branch or continuous-delivery discipline.
- A commit lands automatically once the user validates it; under autopilot, with no validation.
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
- No TDD, no coverage: practices brought in later.
- Every new element, prompt, rule, skill or agent, is written in plain, basic, terse English: short sentences, common words, no idiom. The users are French: a word with a Latin root (rival, concept, restate) is plain to them; a phrasal verb or a Germanic idiom is not. Not every user is fluent in English, and GPT writes in the register of its context, so the instructions are the example. Attractors stay precise, always: the field's terms and its names (cognitive load, race condition, dependency inversion, Tufte) are never simplified.

## Elements

Each is walked before it is built: added, removed or updated. The user settles only what changes the harness's behaviour; the wording and the localization are the agent's, done without the user's review. Origin: `=` copied as is, `~` copied then localized (Claude tool names, `φ` rationale lines, dead references), `*` new.

| Element | Origin | Source in `~/.claude` | To settle |
|---|---|---|---|
| prompts and wiring: system, doctrine, mecha-suit, autopilot, back-of-the-napkin, Extended mind, `opencode.json` | = | `prompts/`, `output-styles/extended-mind.md`, `opencode.template.json` | copied as is, the `φ` Rationale lines cut, the Groveling Worm voice left out, the doctrine without "A question is a question" and "A remark is said for a reason"; `opencode.json` adds the `backend-coach` primary, the default, its chain the mecha suit's with `prompts/backend-coach.md` after `mecha-suit.md` |
| backend coach prompt | * | `skills/strategic-design`, spec §12 | written in `prompts/backend-coach.md`; it holds "a commit lands only once the user validates it, except under autopilot", later in the chain than `mecha-suit.md`'s "a turn that changed a file closes with its commit" |
| rule: documents | ~ | `rules/documents.md` | copied, the store ladder moved in from the doctrine. kept whole otherwise: Extended mind's figure of the whole is for answers, not documents |
| session design | ~ | `rules/session-design.md` | merged into `prompts/mecha-suit.md` with its "one beat per turn" paragraph, no rule file: the robot does not get it |
| rule: delegation | ~ | `opencode-rules/delegation.md` | copied; the session trusts the report and does not check the work again, "independent review" cut so GPT does not skip the cycle's reviewers |
| gates | * | spec §15; `~/.claude/philosophy/kairos-gates.md`; the checklist of the first `stage-reviewer` draft | walked, drafted. A kairos gate, a step of the loop where the practice is applied at the moment it applies; at every stage end, all at once, on the stage's commits, so the agent that wrote the work does not review it; a flaw is fixed before the stage closes. Each gate is a subagent `gate-<concern>` declared in `opencode.json`, its prompt `prompts/stage-reviewer.md` chained, and a skill of the same name that it loads, not inlined: the user can run a gate too, the skill then sending its subagent. A skill, not a rule: the corporate harness holds the security rules. The application runs offline: the security gate checks no authentication, no authorization, no trust in the client. Four, the coach's concerns: `data-integrity` (race conditions, atomicity, persistence), `design`, `security`, `api`. The stage sends every agent named `gate-*`. Open: a design review opening each stage, on the approach before the code |
| skill: study | ~ | `skills/study-making` | walked. Copied, the aggregate-identification dispatch left out. Prototyping is left out: it is a practice used while coding, to be brought in later with an explanation for the users |
| skill: plan | ~ | `old-claude/skills/write-plan`, `safe-rollout` | walked. The plan records no unresolved question: GPT writes down everything without asking. Every leaf stage is a session (1.1, 1.2); a group is not. A stage needs only its north star, on top, shown to the user; the rest is what helps the agent. A done stage is one line where it stands |
| skill: stage execution | ~ | `old-claude/skills/mob-from-plan` | drafted in `skills/stage/`. The navigator speaks at their level, the agent cuts the commits: a series below that level is shown in one look, validated once, landed in a row, and stops at the first commit bringing a new behaviour, domain word or design choice. Autopilot is kept, its commits landing unvalidated; the stage end stays the same. A small blocker is the next commit, a large one a new stage before. A stage may run without a plan, the need as north star, a branch proposed on the default branch. |
| rule: git | * | Claude Code's git lines; `old-claude/skills/atomic-commit` | written in `rules/git.md`, no commit skill: the only source of truth for git, against GPT's reflex and OpenCode's shipped prompt that check the tree again and again. The agent is the only one in the repository; git is for committing and reading history, never for checking. When to commit stays with the primaries and the stage. The convention is Angular's, written in the rule: nothing is asked |
| skill: review walkthrough | ~ | `old-claude/skills/review-walkthrough` | drafted in `skills/review-walkthrough/`; proposed at every stage end, never required; the gates' issues open it |
| skill: testing | ~ | `old-claude/skills/test`, `in-memory-adapters`, `old-claude/rules/test-types.md`, `no-mockito.md` | walked, drafted in `skills/testing/`, Python, pytest, in-memory adapters, no mocks of your own code. The tested cases are co-designed: the agent proposes them as examples, asks only about the open ones; the how is left to emerge with the users. The suite before a commit is the users' pre-commit hooks. No "seen failing" step: no TDD for now. The doctrine applies to new tests; an existing test is never rewritten in passing, its assertions follow a changed behaviour, a rewrite may be suggested in one line. The agent never runs tests, nor the type checker once the project has one: it asks the user (`AGENTS.md`). A new test dependency is the user's call |
| skill: clean code, Python | = | `skills/clean-code-python` | copied |
| skills: ongoing-work, handover | = | `skills/` | copied; the breadcrumb example uses a refunds domain, "CLAUDE.md" is "AGENTS.md" |
| skill: kaizen | ~ | `skills/kaizen` | copied, the philosophy layer and the transcript analyzer cut. The agent gathers the evidence itself |
| skill: ascii | * | a third in `rules/writing.md`; spec §17 | drafted in `skills/ascii/`, standalone, as subagents do not load the napkin prompt; examples in a refunds domain; the same marks in every figure (`[x] [ ] [?]`, `<-- YOU ARE HERE`, `⚠`) |
| agents: plan reviewer, stage reviewer | * | checklist in `write-plan` phase 5; spec §10 | both read-only, neither runs tests. The plan reviewer, in `agents/`, checks order and slicing only. The stage reviewer is the prompt the gates share, `prompts/stage-reviewer.md`: one gate each, issues back to the main session, before the walkthrough |
| `AGENTS.md` | * | | the harness cannot run tests: the user runs them, the agent asks which ones |
| tools: `run`, `grab-notebook`, private tmpdir plugin | = | `tools/`, `plugins/private-tmpdir.ts` | copied, checked running |

The spec is the user's first brief (`brief.md`), kept as the source of the new elements; where it and the model above disagree, the model wins.


OpenCode wiring, as the base does it: primaries declared in `opencode.json` as chains of `{file:./prompts/…}`, built-in `build` and `plan` disabled; subagents as markdown in `agents/`; rules through `instructions` globs, reaching subagents too; verified facts in `~/.claude/opencode-agents.work.md`.

## Steps


