---
name: gate-design
description: The design, architecture, dependency inversion and abstraction-layer checks of a stage review, loaded by the gate-design subagent.
---

# Gate: design

This gate checks the general design: the architecture patterns, the direction of dependencies, the layers of abstraction, and where each rule lives. It does not check concurrency, security or the API contract: other gates do.

## Running this gate

- You are the `gate-design` subagent: go to the next section. Your prompt says how you work and how you report.
- You are in the main session, and the user asked for this gate: do not review the commits yourself. The agent that wrote the work does not review it. Send the `gate-design` subagent with `task`. Give it the north star (the stage's, or the need in one line), the base commit (the stage's, in the notebook; without one, `git merge-base HEAD <default branch>`), and the commits since. Give the user your synthesis of its report, not the raw report. An issue the user wants fixed is a new commit, in the normal cycle; never amend.

## Doctrine

- Design is the user's decision. You do not redesign. You report where the code works against the design the stage chose, or against the design the project already has. A better design you would prefer is not an issue.
- The project's conventions come first. A pattern the project uses everywhere is not an issue in this stage, even if you know a better one.
- Every issue names the cost it causes: what change becomes hard, what test becomes impossible, what rule can now be broken without anyone noticing. An issue with no cost is not an issue.
- A new structure added beside an existing one, in place of reworking the existing one, is an issue.
- A compensation (a workaround that hides a problem left in place) is an issue: name the cause it hides.

## What you check

1. **Dependency rule.** The domain does not import the framework, the ORM, the HTTP client or the settings. Dependencies point inward: the endpoint knows the use case, the use case knows the domain, the domain knows nothing outside it. Look at the imports of each changed file in the domain and use case packages.
2. **Dependency inversion.** When the domain or a use case needs the outside world (a database, a payment provider, the clock), it defines the interface it needs (a port, often a `Protocol`), and an adapter outside implements it. The use case receives the adapter; it does not build it.
3. **Thin edges.** An endpoint, a consumer or a command parses the input, calls one use case, and maps the result to a response. A business rule in an endpoint, a serializer, a SQL query or a template is an issue.
4. **One home for each rule.** A business rule is written once, in the domain. The same rule written in two places will drift. Look for the same condition (`if order.status == "shipped"`) in several files.
5. **Layers of abstraction.** Each function stays at one level of abstraction. A use case reads like the business story: "load the order, refund it, save it, notify the customer". A function that mixes the story with SQL, string formatting or HTTP details is an issue. Load `clean-code-python` for back-end code, `clean-code-ts` for front-end code.
6. **States as types.** A status as a free string, money as `float`, an id as a bare `int` passed among other ints (primitive obsession). A state that should not exist but can be built (a refund with no amount, a shipped order with no address). Report the one that lets a rule be broken.
7. **Names.** The code uses the user's domain words. A new domain word in the code that the user never said is a question for the user. A name that says how, not what, is an issue.
8. **Speculative design.** An interface with one implementation and no reason to exist, a base class, a factory, a configuration option, a parameter that no caller uses yet. Code built for a later stage is inventory: an issue.
9. **Coupling.** A change that forces edits in many unrelated files. A new global, a singleton, a module imported for one constant. A cycle between modules.

## Examples of issues

```
c7d8e9f feat(refunds): refund an order
  issues:  refunds/domain/refund.py:3 imports `sqlalchemy`. The domain now depends on the database:
           its tests need a database, and a change of ORM touches the business rules.
           The repository port could live in the domain, its SQLAlchemy adapter outside.
           api/refunds.py:27 checks `order.status == "shipped"` in the endpoint; the same rule is in
           refunds/domain/refund.py:40. Two homes for one rule: they will drift.
```

## Not in this gate

- Transactions, locks and migrations: `gate-data-integrity`.
- Authentication, authorization, input validation: `gate-security`.
- URLs, methods, status codes, response shapes: `gate-api`.
