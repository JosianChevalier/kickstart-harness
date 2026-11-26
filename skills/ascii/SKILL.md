---
name: ascii
description: Drawing a figure in plain text: a flow, a trust boundary, states, concurrent requests, a plan tree, a review route. In the chat, a study, a plan, a review or a kaizen round.
---

# ASCII figures

## Doctrine

- A figure is a tool to think with, not decoration. Draw the model while it is discussed: the user corrects a relation on the figure before it becomes code.
- The form follows the information, and each element stays only for what it saves the reader (Tufte). A shape is a figure. A comparison is a table. A verdict is a sentence. Not every sentence needs a box.
- A figure is for a human reader. A brief to an agent is prose.
- Draw the subject itself: one real case, in the project's words, with its values. Boxes joined by arrows say nothing: any system can be drawn that way.
- A figure carries the metaphor of the word it draws (Lakoff & Johnson). Before drawing, check that it draws the concept, not the word.
- A clean figure is not evidence. What was observed in the code and what is proposed never look the same.
- The model is the user's. After you draw their model, ask whether it is theirs. When they correct it, redraw it.

## Choose the shape

The relation to understand picks the shape.

| What must be understood | The figure |
|---|---|
| a request, data or control flow | arrows with labels, and the branch gates shown |
| ownership or trust | boxes and boundaries, each crossing labeled |
| states and legal transitions | state nodes, each transition named |
| concurrent requests | one lane per request, aligned on time |
| a plan and its prerequisites | a tree or a dependency graph, with the current position |
| a review route | the route, the concepts next to it, the side paths |
| feedback and improvement | a loop, showing where evidence changes the next action |
| many items sorted into a few homes | one figure, the homes as headings, one line per item |

A comparison across dimensions is a table, not a forced diagram. The one item that decides gets a mark, inside the figure or the table.

## Draw

- One concept per box, named in the user's words. Words and numbers go inside the figure, not in the text under it.
- Label an arrow when its direction alone does not say the relation. Causation, dependency and chronology are three different arrows: do not mix them in one figure.
- Use both dimensions of the page: lanes for concurrency, nesting for containment, branches for alternatives. Do not flatten every model into one long chain of arrows.
- The whole first, then zoom on one region. Keep names and orientation stable across redraws, and across the study, the plan, the stage and the review: the same thing has the same name and the same place.
- The same marks everywhere, with a legend line when the figure needs one:

  ```text
  [x] done    [ ] to do    [?] open decision    <-- YOU ARE HERE    ⚠ hazard
  ```

- Real dispatch points only. Before drawing a flow, find in the code where it really branches; a branch does not pass through setup that only another branch runs.
- A figure goes in a fenced `text` block. Portable ASCII (`+`, `-`, `|`, `>`, `v`); Unicode line drawing is optional, never needed to read the figure. Width: about 80 columns, so it fits the chat.

## Check

- Read the figure in monospace before you send it: alignment, every arrow connected and with its head, every branch labeled, the reading order clear.
- A crowded figure becomes two: an overview and one detail. Never shorten names into abbreviations to make it fit.
- Under the figure, only what it cannot show. Do not repeat each box in prose.
- The same point asked again: redraw it shorter. Do not explain more.
- The test: a reader with no file open finds the subject, the relations, where we are, the open choice and the next step.

## Examples

The examples show the form. Your figure uses the project at hand, its names and its values.

A flow across a trust boundary, with its branch gates:

```text
 browser, untrusted    |  API                                 |  database
                       |                                      |
 POST /orders/4821/    |                                      |
   refund  12.50 EUR --+--> parse: 12.50 EUR, user 77         |
                       |      |                               |
                       |      user 77 owns order 4821?        |
                       |      no ----> 403, nothing written   |
                       |      | yes                           |
                       |      12.50 <= 40.00 paid?            |
                       |      no ----> 422, nothing written   |
                       |      | yes                           |
                       |      v                               |
                       |      one transaction ----------------+--> refund row
                       |                                      |    4821 refunded
```

Two concurrent requests, one lane each, a race condition:

```text
 time   request A, order 4821       request B, order 4822       stock SKU-118
 ----   -------------------------   -------------------------   -------------
  t1    read stock: 1                                                 1
  t2                                read stock: 1                     1
  t3    1 >= 1, write stock 0                                         0
  t4                                1 >= 1, write stock 0             0

 ⚠ two orders, one unit: both read 1 before either one wrote
```

A plan tree, with the current position:

```text
 refunds for paid orders
 +-- [x] 1. a refund is a value: amount, reason, order id
 +-- [ ] 2. POST /orders/{id}/refund                <-- YOU ARE HERE
 |   +-- [x] only the owner, else 403
 |   +-- [ ] amount <= paid, else 422
 |   +-- [?] one refund per order, or several partial ones?
 +-- [ ] 3. the refund reaches the payment provider
```
