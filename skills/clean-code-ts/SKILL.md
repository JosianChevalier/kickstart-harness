---
name: clean-code-ts
description: Always load when writing or reviewing TypeScript code.
---

# Clean code (TypeScript)

## Doctrine

- SLAP, always: one level of abstraction per function (Composed Method, Beck). The tool is extracting a function, never naming a variable.
- Model-driven design (Evans), stratified (Abelson & Sussman): each abstraction layer is a zoom level with its own model, built from the words of the layer beneath.
- Intention-revealing interfaces (Evans): a reader digs into a function to see how, never to understand what.
- Stepdown rule: each layer reads as a coherent English sentence, like well-written prose (Booch).
- Idiomatic TypeScript: the method pipeline, extended to the `Option` and `Result` monads, carries the flow; partial application hides the syntax.
- Behaviour lives on the object that owns the data; a collection with domain meaning is a class.
- Illegal states are unrepresentable: a state is a type, a transition returns the next state's type.
- Everything is immutable: a change returns a new instance.
- Domain expressiveness wins over TypeScript convention: a convention that loses information, absence or partiality included, is wrong here.
- Object Calisthenics, loosely.

## Know-how

### Composition

- Read aloud: "filter, reduce, for" is implementation, "remove inactive, sum totals" is intent.
- A comment separating sections, or an "and" in the description, names the functions to extract. Bodies of one to five lines.
- Early return over nesting.

```ts
// before
function process(customerId: CustomerId): void {
  // Fetch current state
  const orders = orderRepository.findPending(customerId);
  const payments = paymentRepository.findByCustomer(customerId);

  // Detect failed payments
  const failedPayments: FailedPayment[] = [];
  for (const payment of payments) {
    if (payment.isDeclined()) failedPayments.push(FailedPayment.declined(payment));
  }

  // Reconcile
  for (const order of orders) {
    if (failedPayments.some(f => f.matches(order))) order.cancel();
  }
}

// after
function process(customerId: CustomerId): void {
  const pendingOrders = fetchPendingOrders(customerId);
  const failedPayments = detectFailedPayments(customerId);

  reconcile(pendingOrders, failedPayments);
}
```

- A pipeline, `.map`/`.flatMap` on a `Result` or an array, has a named concept at each step. A lambda holding logic becomes a function returning a function, partial application; a bare delegation such as `p => p.isListed()` stays inline.
- Chaining over unwrapping: `map`, `flatMap`, `orElse`, never `isEmpty` then `get`. A type without a chain is handled imperatively, each non-trivial branch extracted.

```ts
return expectedTimestamps
  .filter(notCoveredBy(covered))
  .map(asMissing(unit));

const notCoveredBy = (covered: Set<Instant>) => (timestamp: Instant): boolean =>
  !covered.has(timestamp);
```

### Names

- Intent, in the domain's language; no abbreviation in code: `quantity`, `repository`.
- A factory is imported by name when the call reads without its qualifier, `customerId('alice')`, `success(result)`; the qualifier stays when it does not, `CustomerId.of('alice')`.

### Placement

- A service is a closure: context captured once, one behaviour returned. Several closures over the same context, calling each other or evolving together, are a class, the repository shape being the archetype. Never a module of unrelated exported functions.
- A function using `this` or part of the public API is a member; a pure helper is a non-exported function below the class, the module being the privacy boundary. It composes point-free: `array.map(helper)` works where `array.map(this.helper)` loses its binding.

### Types

- A state is a class with a literal `state` field, a discriminated union over classes. The transition a state lacks does not compile; a consumer demands the one state it handles.

```ts
export class OrderReady extends Order {
  readonly state: OrderState.Ready = OrderState.Ready;

  cancel(): OrderCancelled { return OrderCancelled.from(this, new Date()); }
  process(): OrderExpedited { return OrderExpedited.from(this, new Date()); }
}

export class OrderExpedited extends Order {
  readonly state: OrderState.Expedited = OrderState.Expedited;
  // no cancel(): an expedited order cannot be cancelled, the compiler says so
}

const processOrder = (order: OrderReady): Promise<ProcessOrderOutput> => ...
```

- A value object is nominal: a class when it has behaviour (`Euros.add`), a branded type with a factory when it is id-like, through one branding utility for the whole project.
- Absence is modelled, `Option<Price>`, never a required field filtered upstream.

### Errors: the ladder

Failure handling is a design decision, not a reflex. Pick the lowest rung that fits. **If the caller must react, make failure visible in the type system; if not, let it propagate.**

#### 1. Tagged Union Return — outcomes the caller decides on

When failure is a *domain outcome* the caller inspects and acts upon, return it as data.

```ts
export type ProcessOrderOutput =
  | { result: 'SUCCESS'; events: ProcessingEvents }
  | { result: 'FAILURE'; events: ProcessingEvents; cause: string };

export type InvalidPrice =
  | { type: 'EMPTY_ORDER'; cause: string }
  | { type: 'INVALID_PRICE'; cause: string }
  | { type: 'UNKNOWN_PRODUCT_TYPE'; cause: string };
```

Match exhaustively on the discriminant; `assertNever` makes new cases fail to compile:

```ts
switch (error.type) {
  case 'EMPTY_ORDER': return ...;
  case 'INVALID_PRICE': return ...;
  case 'UNKNOWN_PRODUCT_TYPE': return ...;
  default: return assertNever(error);
}

function assertNever(value: never): never {
  throw new Error(`Unreachable: ${JSON.stringify(value)}`);
}
```

#### 2. Railway (`Result`) — recoverable failures in a flow

When a multi-step transformation can fail midway and later steps must be skipped, compose with `Result.map`/`flatMap` — the first failure short-circuits, and the error type is part of the signature.

```ts
fullPrice(): TotalPriceResult {
  return Result.success(this.orderLines)
    .flatMap(validateNonEmpty)
    .map(mapArray(addVat))
    .flatMap(Result.combine)
    .flatMap(ensureHigherThan(this.deliveryFees))
    .valueOrError();
}
```

Use it when it's cheaper than try-catch at every step — almost always true once the flow has more than two failure points.

#### 3. Local try-catch — recovery is local and imperative

When the recovery logic lives right next to the risky call and the surrounding code is already imperative, a local try-catch is simpler than threading a `Result` through.

```ts
function parseCachedRates(raw: string): TaxRates {
  try {
    return taxRates(JSON.parse(raw));
  } catch {
    return DEFAULT_TAX_RATES;   // recovery is one line, right here
  }
}
```

Do **not** let a try-catch interrupt a functional pipeline with imperative logic — that's rung 2's job.

#### 4. Throw — unrecoverable

Bugs, broken invariants, unreachable states: throw and let it propagate to the boundary (the left adapter translates to HTTP 500, NACK, etc.). The domain never catches these.

```ts
class DiscountRate {
  constructor(readonly value: number) {
    if (value < 0 || value > 100) throw new Error(`Discount rate must be 0-100, got ${value}`);
  }
}
```

#### Checklist

1. Caller decides on the outcome? → tagged union return
2. Multi-step flow with recoverable failure? → `Result` railway
3. One-line local recovery amid imperative code? → local try-catch
4. No recovery possible? → throw; never catch it in the domain

### Comments are code smells

A comment is an admission that the code failed to express itself. The goal is **zero comments**, reached by iterating on the code until each comment becomes redundant.

#### The Rule

1. See a comment explaining *what* or *how* → **iterate on the code** until the comment is redundant
2. Delete the redundant comment
3. Only keep comments that explain *why* something non-obvious exists

#### Process: Eliminate a Comment

For every comment, ask: **"Can I change the code so this comment is unnecessary?"**

##### Rename to replace the comment

```ts
// BAD
// Check if the cart has not been updated for more than 24 hours
if (hoursBetween(cart.lastUpdated, now) > 24) {

// ITERATION 1 — extract to function, delete comment
if (isInactiveSince(cart, now, hours(24))) {

// ITERATION 2 — push behavior into the object
if (cart.isAbandoned()) {
```

##### Extract function to replace the comment

```ts
// BAD — comment separating a section
function replenish(warehouseId: WarehouseId): void {
  const products = productRepository.findByWarehouse(warehouseId);

  // Filter products running short
  const shortages = products.filter(p => p.quantityInStock < 10 || p.hasBackorders());

  shortages.forEach(reorder);
}

// GOOD — the function name IS the comment
function replenish(warehouseId: WarehouseId): void {
  const products = productRepository.findByWarehouse(warehouseId);
  const shortages = detectShortages(products);

  shortages.forEach(reorder);
}
```

##### Introduce a domain concept to replace the comment

```ts
// BAD
// Fewer than 10 units is considered low stock
if (product.quantityInStock < 10) {

// GOOD — the domain concept speaks for itself
if (product.isLowOnStock()) {
// Inside Product, the threshold is a named constant or part of a StockQuantity value object
```

##### Replace conditional comment with expressive boolean

```ts
// BAD
// Only sell the product if it is listed and not out of stock
if (product.status === Status.Listed && !outOfStockProducts.has(product.id)) {

// GOOD
const isPurchasable = product.isListed() && !product.isOutOfStock();
if (isPurchasable) {
```

#### The Only Surviving Comments

Comments that explain **why** — a business rule, a workaround, a non-obvious constraint. These cannot be expressed in code.

```ts
// DST transitions produce 23h or 25h days, so we use calendar days, not duration
const dayBoundaries = computeCalendarDayBoundaries(zone);

// Legal requirement: invoices must be retained for 10 years even after payment
const retentionCutoff = now.minusYears(10);
```

If you can turn the "why" into a well-named function or constant, do it. The comment only survives when it truly can't be expressed otherwise.

#### Checklist

For every comment in the code:

1. **Is it explaining how/what?** → Rename, extract, or restructure until it's redundant, then delete it
2. **Is it separating sections?** → Each section becomes its own function
3. **Is it explaining why?** → Keep it, but try once more to express it in code
4. **Is it a TODO?** → Acceptable as short-lived markers. Handle each TODO before moving to another task, in its own commit
