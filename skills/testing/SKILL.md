---
name: testing
description: Writing or reviewing Python tests (pytest); a test double, a fake or an in-memory adapter; a change to shared state under concurrent requests; a test result needed or reported.
---

# Testing

## Doctrine

- A test proves an observable behaviour, not the shape of the code. Test what the code does, not what the doubles do.
- The tests come with the change they prove, in the same commit. Every commit leaves the tests passing. A change without its tests is not done.
- Test and code are written in any order. What counts is that they land together.
- This doctrine applies to new tests. An existing test is never rewritten in passing, even when the change touches it.
- Tests are the example mapping, written in code. A reader who does not code reads a test body and understands the business rule.
- Which cases a behaviour is tested on is a domain question. The agent proposes the cases; the user stays in the choice, without carrying it.
- A test asserts enough to exclude the plausible wrong implementations.
- No mock of your own code: no `unittest.mock`, no `MagicMock`, no `patch`. A double behaves like the real thing: a fake with real state, an in-memory adapter. A mock is a tool to isolate, never the thing under test.
- Time, randomness and external effects are controlled: they enter as arguments.
- A fake is a hypothesis about the real adapter. The real adapter verifies it.
- What concurrent requests produce is a domain decision, and it belongs to the user. Only an overlap through the real application boundary, on representative storage, proves it.
- The agent never runs the tests, nor the type checker. The user runs them in their own setup. When the agent needs a result, it names the tests to run and asks the user.
- Evidence is reported as it is: passed, failed, skipped, not run, not testable in the user's setup. A static check is not runtime proof. A behavioural probe is evidence, not a universal guarantee.

## Know-how

### Asking for a test run

You do not run the tests, nor the type checker: the user runs them in their own setup. When you need a result, ask the user to run the tests, say which ones, then wait for what they saw.

Find the project's test command first: `pyproject.toml`, `Makefile`, `tox.ini`, `noxfile.py`, the CI configuration, the README. Give it as written (`uv run pytest`, `poetry run pytest`, `make test`). Do not invent another. Name the smallest selection that gives the evidence you need, as a command the user can copy:

```bash
pytest                                              # all tests
pytest tests/ordering/test_payment_monitor.py       # one file
pytest tests/ordering/test_payment_monitor.py::TestWhenCheckingPayments              # one class
pytest tests/ordering/test_payment_monitor.py::TestWhenCheckingPayments::test_cancels_order_when_payment_is_declined   # one test
pytest -k "declined"                                # tests whose name matches
pytest -x                                           # stop at the first failure
pytest -m "not integration"                         # skip tests with a marker
```

A failing test is never skipped, marked `xfail`, deleted or weakened to make the suite pass. A test that already failed before the change is reported to the user at once, not fixed in passing and not hidden.

### Choosing the cases

Propose the cases with the change, as concrete examples in the user's words: "Alice cancels before shipping: refunded. After shipping: ?". Ask only about a case whose outcome you do not know, or one the user may not have thought of. A case that follows from what they said is listed, not asked. How the cases are agreed is open: find with the user what keeps them engaged without loading them.

### Test types

| Type | It tests | Doubles | Speed |
|---|---|---|---|
| Unit | domain logic, use cases | in-memory adapters and fakes; never an intermediate domain class replaced | milliseconds |
| Integration | one adapter: repository, HTTP client, API endpoint | the real engine in a container, or a fake server with state | milliseconds to a second |
| End-to-end | a complete user journey, all components together | real database; external services faked at the HTTP level | seconds |
| Architecture | the layers and the dependency rule | none | seconds |

- **Unit.** No database, no network, no web framework. The use case is built with in-memory adapters. Files mirror the production package under `tests/`.
- **Integration, repository.** It runs against the same database engine as production: the project's existing fixture, or a container (`testcontainers`). Each test starts from a known state: a transaction rolled back, or a clean schema. SQLite in place of PostgreSQL is not the same engine.
- **Integration, HTTP client.** It runs against a fake HTTP server that keeps internal state and answers like the real service (for example `pytest-httpserver`).
- **Integration, API endpoint.** It runs through the framework's test client (FastAPI `TestClient`, Django `Client`, Flask `test_client`). The use case behind it is wired to in-memory adapters, not to a mock.
- **End-to-end.** The whole application, its real database in a container, external services faked at the HTTP level. Keep them for the critical journeys and for proof that all components work together.
- **Architecture.** `import-linter` contracts: the domain imports no infrastructure, a layer does not import the layer above it. They run in CI.

A slow test carries a marker (`@pytest.mark.integration`, `@pytest.mark.e2e`), declared in the pytest configuration, so the fast suite runs alone. Use the markers the project already has. A container starts once per test session: a fixture with `scope="session"`.

A new test dependency (`testcontainers`, `pytest-httpserver`, `import-linter`) is the user's decision. Propose it, with what it proves, and wait.

### Structure

- **Files:** `test_<module>.py`, next to a path that mirrors the production module: `ordering/payment_monitor.py` gives `tests/ordering/test_payment_monitor.py`.
- **Classes:** one class per scenario or feature, to group related tests: `TestWhenCheckingPayments`, `TestWhenShippingOrders`. pytest needs the `Test` prefix. No `__init__` in a test class.
- **Names:** `test_` then a phrase about behaviour: `test_returns_true_when_cart_has_invalid_items`, `test_raises_when_payment_method_is_not_found`, `test_ships_order_when_payment_is_confirmed`.

| Quality | Good | Bad |
|---|---|---|
| Minimal | One thing. An "and" in the name: split it. | `test_validates_email_and_domain_and_whitespace` |
| Clear | The name describes the behaviour. | `test_1` |
| Shows intent | It shows the desired API. | It hides what the code should do. |

**Fixtures** carry the wiring: a fresh fake per test, the use case built on it. A fixture has the default scope, `function`, so no state leaks from one test to the next. Shared fixtures live in `conftest.py`. The arrangement that matters to the scenario stays in the test body, where the reader sees it; a fixture never hides a Given.

**Parametrize** a table of cases of one rule, with readable ids. It is example mapping in a table:

```python
@pytest.mark.parametrize(
    ("total", "shipping"),
    [(eur(29), PAID), (eur(30), FREE), (eur(45), FREE)],
    ids=["below threshold", "at threshold", "above threshold"],
)
def test_offers_free_shipping_from_the_threshold(total, shipping):
    policy = ShippingPolicy(threshold=eur(30))

    assert policy.evaluate(total).shipping == shipping
```

### Arrange, Act, Assert

Each test has three sections: Arrange, Act, Assert, separated by one blank line.

```python
class TestWhenCheckingPayments:
    def test_cancels_order_when_payment_is_declined(self, orders, payments, monitor):
        orders.save(a_pending_order(placed_by=alice))
        payments.will_decline(alice)

        monitor.check_payments(alice)

        assert orders.find_for(alice).status == CANCELLED
```

Two blank lines between sections when a section has blank lines inside it:

```python
def test_offers_express_shipping_to_loyal_customers(orders):
    policy = (
        a_shipping_policy()
        .with_threshold(eur(30))
        .build()
    )

    orders.has_existing_orders(placed_by=alice, count=3)


    quote = policy.evaluate(eur(35), customer=alice)


    assert quote == ShippingQuote(free=True, method=EXPRESS)
```

### Assertions

- Plain `assert`: pytest shows both sides when it fails. No `self.assertEqual`, no `unittest.TestCase`.
- Compare whole values when you can: `assert quote == ShippingQuote(free=True, method=EXPRESS)` excludes more wrong implementations than `assert quote.free`.
- Exceptions: `with pytest.raises(UnknownPaymentMethod, match="card-4111"):`. The exception type is a domain type; the match checks the part of the message that matters.

### Test data

- **Builders.** Test data comes from builder functions in one place (the project's place; without one, `tests/builders.py`). A builder function takes keyword-only arguments with realistic defaults, so each dimension has a name: `an_order(placed_by=alice, status=PENDING)`. When the data combines several domain dimensions and a sentence reads better, use a fluent builder: `an_item().priced_at(eur(18)).discounted_to(eur(15))`. Never positional arguments whose meaning is unclear: `item(18.0, 15.0, EXPRESS)`.
- **Complete data.** Test data mirrors the complete real structure, with every field the system can read downstream, each with a realistic default. Partial data hides structural assumptions: the test passes, the real data breaks. A dict or a JSON payload is the main risk: a dataclass with required fields protects itself, a dict does not. Before you create test data, read the real structure in the production code.
- **Time.** The code under test receives the time: a `now: datetime` argument, or a clock function. A test passes a fixed instant: `datetime(2025, 1, 1, 10, 0, tzinfo=UTC)`. Never `patch` `datetime.now`.
- **Time names.** Name instants like a schedule: `_00h00`, `_00h30`, `_10h30`. Name days like calendar entries: `january_first`, `march_third`.
- **Domain ids.** Realistic names: `product_1`, `warehouse_a`, `customer_alice`. Not `p1`, `w1`, `c1`, `id1`, `test_id`.
- **Imports.** Import the names the data uses directly, so test lines read like domain sentences: `from shop.money import eur` and `eur(100)`, not `shop.money.eur(100)`. A module-level alias for an enum member is fine when it helps the reading: `EXPRESS = ShippingMethod.EXPRESS`.

### Readability

A test body is read aloud, without the syntax (`=`, parentheses, `self`). If it does not sound like the description of a scenario, rework it until it does.

1. **No unpacking in helper calls.** Helpers and builders take domain types. If a call needs `str(...)`, `.value` or another accessor to fit a helper, change the helper: it accepts the domain type, and a version for raw values converts once, inside the helper.

   ```python
   # bad: leaks the implementation, breaks the sentence
   a_customer().identified(str(alice.customer_id))
   # good: reads as a specification
   a_customer().identified(alice)
   ```

2. **Builders start with an article:** `a_customer_id()`, `a_product()`, `an_order()`, `a_customer()`.
3. **Variables carry the role in the scenario**, not the type: `alice`, `product_in_stock`, not `c1`, `p1`.
4. **Fake setup reads like Given clauses:**

   ```python
   catalog.having_products(product)
   inventory.having_stock_for(product_in_stock, StockEntry(AVAILABLE))
   payments.will_return_nothing()

   having.for_customer(acme).card("4111-1111-1111-1111").declined().card("5555-5555-5555-4444").accepted()
   ```

5. **Builders read like descriptions:** `an_order().placed_by(alice).with_item(a_product().identified(product_out_of_stock))`.
6. **A value used in several setup calls is one named variable.** The duplication hides the link between the steps.

   ```python
   # bad: the same instant twice, the link hidden
   create_stock_count("2025-01-01T10:30:00Z", 200)
   create_sale("2025-01-01T10:30:00Z", eur(100))
   # good: one variable; the helpers accept a datetime
   _10h30 = datetime(2025, 1, 1, 10, 30, tzinfo=UTC)
   create_stock_count(_10h30, 200)
   create_sale(_10h30, eur(100))
   ```

7. **Composite data names each dimension** (see Builders).

Checklist, to write or to review a test:

1. No unpacking: are domain types passed directly to helpers?
2. Articles: do builders start with `a_` or `an_`?
3. Intent names: do variables name the role in the scenario?
4. Given clauses: does the fake setup read as English?
5. Readable builders: does the Arrange section read like a description?

### No mocks of your own code

Do not use `unittest.mock` (`Mock`, `MagicMock`, `AsyncMock`, `patch`), `pytest-mock`'s `mocker`, or `monkeypatch.setattr` on your own code. Mocks lead to:

- tests that check calls to the mock instead of observable outcomes;
- brittle tests that break when the implementation changes and the behaviour does not;
- false confidence in coverage;
- test intent hidden under mock setup.

When a test needs a dependency:

1. Look for an existing in-memory adapter or fake.
2. If none exists, write one (see In-memory adapters).
3. If the real implementation is fast and has no external effect, use the real one.

A third-party library (an SDK, an HTTP client, a cloud API) is not mocked either. It sits behind a port of your own, a `Protocol`. The domain tests use an in-memory adapter of that port; the real adapter gets its integration test.

If a mock seems necessary, the design is the first suspect. Show the user the dependency and the design question; do not add the mock.

An existing test is never rewritten in passing, even when the change touches it: its assertions follow the new behaviour, its form stays. When an existing test breaks this skill, for example with mocks, suggest a rewrite to the user in one line.

### In-memory adapters

An in-memory adapter implements a port and **behaves like the real system**. It is not an empty stub.

- A repository stores and retrieves with a `dict` or a `list`, like a database.
- A finder returns consistent results from one call to the next.
- It enforces the constraints of the real system: uniqueness, the not-found case, the error the real one raises.
- It implements the `Protocol` of the port, so the type checker verifies that it has every method the real adapter has.

```python
class InMemoryOrderRepository:  # implements OrderRepository (Protocol)
    def __init__(self) -> None:
        self._orders: dict[OrderId, Order] = {}

    def save(self, order: Order) -> None:
        self._orders[order.order_id] = order

    def get(self, order_id: OrderId) -> Order:
        if order_id not in self._orders:
            raise OrderNotFound(order_id)
        return self._orders[order_id]

    def find_by_customer(self, customer_id: CustomerId) -> list[Order]:
        return [order for order in self._orders.values() if order.customer_id == customer_id]
```

Why: the domain is designed, tested and changed before any technical code exists, with no coupling to a database, an HTTP client or a message broker. An in-memory adapter is often the first working adapter, and often temporary: a persistent or remote adapter replaces it when the domain is stable.

| Real adapter | In-memory alternative |
|---|---|
| PostgreSQL repository (SQLAlchemy) | a `dict` or a `list` |
| HTTP client to an external API | static or configurable responses |
| Message publisher | the published messages kept in a `list` |

Location, unless the project has its own:

- An in-memory adapter the application can run with is production code: in the infrastructure package, for example `adapters/in_memory/`.
- A fake used only by tests lives in `tests/fakes/`. Its test-only helpers (`having_products`, `will_decline`) live there too, never in production code.

**A fake is proven against the real adapter.** Write the behaviour of the port once, as contract tests, and run them against both adapters with a parametrized fixture. The fake stays faithful; the real adapter gets its tests.

```python
@pytest.fixture(params=["in_memory", "postgres"])
def order_repository(request):
    ...  # returns InMemoryOrderRepository() or the PostgreSQL adapter

def test_finds_the_orders_of_a_customer(order_repository):
    ...
```

### Anti-patterns

Three laws:

1. Never test the behaviour of a double.
2. Never add a test-only method to a production class.
3. Never replace a dependency you do not understand.

**1. Testing the behaviour of a double.**

```python
# bad: checks that a call happened, not that the behaviour happened
stock_monitor.check_stock(warehouse_a)
restock_orders.save.assert_called_once()
# good: a fake with real state, an assertion on the outcome
stock_monitor.check_stock(warehouse_a)
assert restock_orders.find_all() == [a_restock_order(for_product=keyboard)]
```

The bad test passes when the call happens, right or wrong. Before any assertion on a call, ask: "Am I testing real behaviour, or only that a call was made?" If only a call: delete the assertion, or use a fake with real state.

**2. Test-only methods in production.**

```python
# bad: reset() exists only for tests, and looks like production API
class OrderLifecycleService:
    def reset(self) -> None:
        self._orders.clear()
```

It pollutes the production class, it is dangerous if production calls it, and it confuses the lifecycle of the object with the lifecycle of the entity. The fix: a fresh fake per test, from a fixture, so no reset is needed; a helper only tests need lives in the fake. Before you add a method to a production class, ask: "Is it used only by tests?" If yes, it goes in test code. Ask also: "Does this class own the lifecycle of this resource?" If no, it is the wrong class.

**3. Replacing a dependency you do not understand.**

```python
# bad: the double removes the lookup the test depends on; nothing to ship, the test passes for nothing
orders = FakeOrderRepository(always_empty=True)
payment_monitor.on_payment_accepted(payment_id)
# good: a fake with real state
orders.save(a_pending_order(placed_by=alice, paid_with=payment_id))
payment_monitor.on_payment_accepted(payment_id)
assert orders.find_pending(alice) == []
```

Before you replace a dependency, ask: what side effects does the real method have? Does the test depend on one of them? Do I understand what the test needs? If the test depends on a side effect, the double keeps it. If you are not sure, ask the user to run the test with the real implementation first, read what has to happen, then add the smallest double at the right level. Red flags: "I will replace it to be safe", "it might be slow", a double chosen without the chain of dependencies in view.

**4. Incomplete test data.** See Complete data: only the fields you think you need hides what the code reads downstream.

**5. Tests after the fact.** "The implementation is complete, the tests come next" is not complete. The tests are part of the change, in its commit.

**6. Redundant setup for default behaviour.**

```python
# bad: the error handling already gives an empty set
inventory.having_products_in_warehouse(warehouse_a, [])
# good: arrange only what the scenario needs
inventory.having_no_warehouse(warehouse_a)
```

Before you arrange an empty or default answer, ask: "What happens if I arrange nothing?" If the code under test gives the same result (for example, a 404 handled as an empty set), the setup is noise. Arrange an empty answer only when the empty case is the subject of the test.

**7. Testing what Python or the type checker guarantees.** A test that cannot fail while the type checker passes tests nothing. Do not test: dataclass field access, `isinstance` on a declared base class, the existence of an enum member, the exhaustiveness of a `match` that ends in `assert_never`. Do test: validation in `__post_init__`, computed properties, a custom `__eq__` or `__hash__`, business rules in domain types. Before you write a test, ask: "Can it fail while the type checker passes?" If no, do not write it.

**When doubles grow too complex.** Signs: the double's setup is longer than the test logic; doubles everywhere to make the test pass; the double lacks methods the real component has; the test breaks when the double changes. Ask: "Do we need a double here?" A test with the real components is often simpler.

| Anti-pattern | Fix |
|---|---|
| Assertion on a call to a double | Assert on an outcome, with a fake with state |
| Test-only method in production | Move it to test code |
| Double without understanding | Understand the dependency first; smallest double |
| Incomplete data | Mirror the real structure completely |
| Setup for defaults | Check what the code already does without it |
| Tests after the fact | Tests in the same commit as the change |
| Complex doubles | A test with real components |
| Testing guarantees | Test only what can fail while the type checker passes |

Red flags:

- an `assert_called*` or a `call_args` in a test;
- a method called only from test files;
- double setup over half of the test;
- a test that fails when you remove the double;
- a double you cannot explain;
- a double "to be safe";
- an empty answer arranged when error handling already gives it;
- a test of dataclass fields, of `isinstance` on a base class, of enum members.

### Concurrent requests

Before a change to shared backend state, agree with the user on the result of concurrent **application requests**. These are domain decisions; the user takes them, guided by a concrete case ("Tom books the same seat one second after Alice: what happens?"):

- Which requests succeed?
- What does each caller observe?
- What state remains?
- What invariant must never be violated?

```text
time ──────────────────────────────────────────────▶
Alice  POST /bookings ── read: 1 seat left ── write ── 201
Bob    POST /bookings ──── read: 1 seat left ── write ── ?
                                                 └─ invariant: never 2 bookings for 1 seat
```

Then test the overlap:

- **Through the real application boundary:** the HTTP API, or the use case entry point with its real transaction handling.
- **On representative storage:** the same database engine and the same isolation level as production. Sequential tests and in-memory stores do not prove concurrency safety: an in-memory adapter has no transactions and no locks. The GIL does not make a read then a write atomic. An async application has races too, at each `await`.
- **At the same moment:** start the requests from threads that wait on a `threading.Barrier`, or with `asyncio.gather` for an async client, each request with its own client and its own database connection.
- **Many times:** a race is probabilistic; repeat the overlap (for example 50 times) before you trust a pass.
- **Or deterministically:** with two database connections, open the first transaction, make its read or write, run the second request, then commit the first. This forces the interleaving you want to see.
- **Assert the decisions:** the status each caller received, the final state, the invariant.

```python
def test_only_one_customer_books_the_last_seat(api, concert_with_one_seat_left):
    barrier = threading.Barrier(2)

    def book(customer):
        barrier.wait()
        return api.client_for(customer).post("/bookings", json={"concert": concert_with_one_seat_left})

    with ThreadPoolExecutor(max_workers=2) as pool:
        responses = list(pool.map(book, [alice, bob]))

    assert sorted(response.status_code for response in responses) == [201, 409]
    assert api.bookings_for(concert_with_one_seat_left) == 1
```

A concurrency case not tested this way is reported as not tested, with the reason.

### Reporting evidence

Every report of test results distinguishes:

- **Passed.**
- **Failed.**
- **Skipped**, with the reason pytest gives.
- **Not run**: what the user did not run, with why when it is known.
- **Not testable in the user's setup**, with what is missing: no Docker, no network, no credentials.

Rules:

- Report what the user saw, never what you expect. Name the command the user ran and copy the numbers from pytest's summary line. If the user gives no summary line, ask for it. An error at collection is a failure.
- "The tests pass" means the user ran the full suite and it passed. When only a part ran, say which part.
- Never "it should pass". Either the user ran it, or it was not run.
- A static check (type checker, linter, `import-linter`) is not runtime proof. Say "the type checker passes", not "it works".
- A behavioural probe (one manual request, one run of a race test) is evidence for the case it ran, not a guarantee for every case.

```text
Asked: uv run pytest -m "not integration" tests/ordering
The user ran it: 42 passed, 2 skipped (need Docker), 0 failed
Not run: integration tests, the user did not run them
Static: the user reports mypy --strict passes (not runtime proof)
```
