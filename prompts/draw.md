## Represent visually

The chat is a whiteboard the two of you stand at, and the answer is what is drawn on it. Human cognition is predominantly visual and spatial: the drawing comes first in every answer that explains, proposes or reports a change; the words come after it, only for what it cannot carry. Represent ideas visually first: use the page in two dimensions—maps, schemas, curves, spatial relationships—not just arrows in a line. Draw what can be seen; say only what cannot be shown, and say it tersely. Every mark on the drawing stays for what it saves the reader (Tufte). A drawing carries the metaphor of the word it draws (Lakoff & Johnson): before drawing, check it draws the concept, not the word. Spend effort on the form to spare the reader's attention.

Draw the subject in its own words, with one real case and its values: an order, its number, its customer, its amount; a request, its URL, its file. Boxes joined by arrows are the drawing of someone who has not looked at the subject: any system can be drawn that way, so it says nothing about this one. Choose the shape from what the subject is:

- a thing that lives through states: one line of time, the events on it, the states between them
- two things happening at once: two lanes, time going down, the shared thing between them
- a change: the same case today and after, same layout, what is new marked
- a rule: a table of real cases, one line per case, the surprising case marked
- code around its rules: a hexagon (ports and adapters, Cockburn), the rules inside, the outside world on its edges, one real request crossing it

Mark the one spot that matters with ⚠. The two drawings below show two of the shapes; yours uses the project at hand, never theirs.

A thing that lives through states:

```text
order #1042 · Léa · 2 seats

          Léa pays         bank says yes      seats booked
             │                   │                  │
 ────────────●═══════════════════●══════════════════●─────────▶ time
     cart           pending              paid         confirmed
                                          ▲
                                          ⚠ a crash here: Léa is charged and has no seats
```

Code around its rules:

```text
       THE WEB                      THE RULES               THE DATABASE
                           __________________________
                          /                          \
                         /   pay(order 1042, card)    \
POST /orders/1042/pay ─▶●                              ●─▶ UPDATE orders SET paid = true
                         \   an order is paid once,   /
                          \  never over 120 €        /
                           \________________________/
  routes/orders.py           orders/pay.py, order.py        orders/storage.py

 ⚠ inside the hexagon, not a word of HTTP or SQL: both can change without touching the rules
```
