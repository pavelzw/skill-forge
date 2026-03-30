---
name: unit-testing
description: >
  Write high-quality unit and integration tests for Python code using pytest,
  following value-driven testing principles. Use this skill whenever the user
  asks you to write tests, add test coverage, create a test suite, write unit
  tests, integration tests, or review existing tests for quality. Also trigger
  when the user mentions pytest, testing strategy, mocking best practices, test
  refactoring, test design, TDD, or asks "how should I test this?". This skill
  ensures every test you write is valuable, maintainable, and resistant to
  refactoring. Always consult this skill before writing any test code.
---

# Unit Testing Skill

## Scope and Principles

Only write and modify test code. Do not refactor or modify the user's
production code. If production code would benefit from changes to improve
testability, suggest the refactor but do not perform it. Always read the
production code first to classify it (Step 1) before writing any tests.

Every test has a maintenance cost. Prefer a small suite of high-value tests
over broad coverage with low-value ones. Evaluate each test against these
four pillars:

1. Protection against regressions — exercise meaningful, complex code paths.
2. Resistance to refactoring — must not break when implementation changes
   but behavior stays the same. Never compromise on this.
3. Fast feedback — keep tests fast (milliseconds).
4. Maintainability — keep tests short, readable, and free of complex setup.

When (1) and (3) conflict, make a deliberate trade-off. (2) is non-negotiable.

## Step 1: Classify the Code Before Writing Tests

| | Few collaborators | Many collaborators |
|---|---|---|
| High complexity / domain significance | Unit test extensively | Suggest refactoring first (Humble Object pattern), then test what you can |
| Low complexity / domain significance | Don't test (trivial code) | Integration test (few, happy-path + critical edges) |

- Domain logic with few collaborators = best ROI. Test these first.
- Trivial code (constructors, one-liner pass-throughs) = don't test.
- Overcomplicated code (high complexity + many collaborators) = don't
  mock everything. Suggest splitting into a pure domain piece and a thin
  orchestration piece (Humble Object pattern), but do not perform the
  refactor yourself. Write the best tests you can for the code as-is.

## Test File Organization

Place tests in a `tests/` directory mirroring the source structure. Name test
files `test_<module>.py`. For example, `src/billing/invoice.py` →
`tests/billing/test_invoice.py`.

## Step 2: Choose the Testing Style

Prefer in this order:

1. Output-based (best) — Supply input, verify return value. Shortest,
   most maintainable, most refactoring-resistant.
2. State-based (good) — Invoke operation, verify resulting state.
3. Communication-based (sparingly) — Verify the system under test called
   an external dependency correctly. Only for unmanaged dependencies.

## Step 3: Write the Test

### Structure: AAA (Arrange, Act, Assert)

```python
def test_delivery_with_a_past_date_is_invalid():
    # Arrange
    service = DeliveryService()
    delivery = Delivery(date=datetime.now() - timedelta(days=1))

    # Act
    is_valid = service.is_delivery_valid(delivery)

    # Assert
    assert is_valid is False
```

- One act per test. Multiple acts = multiple behaviors = multiple tests.
- No `if` statements in tests. Branching means two tests.
- Act should be a single call. If it takes multiple lines to invoke one
  behavior, the API may be leaking implementation details.
- Multiple assertions are fine if they verify the same unit of behavior.

### Testing Exceptions

Use `pytest.raises` to verify error paths. Name the test after the scenario,
not the exception class.

```python
def test_withdrawal_above_balance_raises_insufficient_funds():
    account = Account(balance=100)
    with pytest.raises(InsufficientFundsError):
        account.withdraw(200)
```

### Testing Async Code

For async functions, use `pytest-asyncio`:

```python
@pytest.mark.asyncio
async def test_fetching_user_returns_profile():
    client = AsyncAPIClient()
    profile = await client.get_user(user_id=42)
    assert profile.name == "Alice"
```

### Naming: Plain English Facts

Write test names as if describing the scenario to someone who knows the
domain but not the code.

```python
# GOOD
def test_delivery_with_a_past_date_is_invalid(): ...
def test_purchase_succeeds_when_enough_inventory(): ...

# BAD — rigid template, implementation-focused
def test_is_delivery_valid_invalid_date_returns_false(): ...
```

- Don't include the method name. You test behavior, not methods.
- Don't use "should be" — state facts, not wishes.
- Be specific: "past date", not "invalid date".

### Fixtures: Factory Functions, Not Shared State

```python
# BAD — shared setUp couples tests together
class TestCustomer:
    def setup_method(self):
        self.store = Store()
        self.store.add_inventory(Product.SHAMPOO, 10)
        self.customer = Customer()
```

```python
# GOOD — self-contained, readable, decoupled
def test_purchase_succeeds_when_enough_inventory():
    store = create_store_with_inventory(Product.SHAMPOO, 10)
    customer = Customer()
    success = customer.purchase(store, Product.SHAMPOO, 5)
    assert success is True

def create_store_with_inventory(product, quantity):
    store = Store()
    store.add_inventory(product, quantity)
    return store
```

Exception: shared infrastructure in `conftest.py` (DB connections) is fine.

### Fixtures vs. Factory Functions

Use `@pytest.fixture` for shared infrastructure (DB sessions, HTTP clients) or
when pytest's teardown/scoping is needed. Use plain factory functions for
creating domain objects — they're simpler, explicit, and don't hide setup.

```python
# Fixture: good for infrastructure with teardown
@pytest.fixture
def db_session():
    session = create_test_session()
    yield session
    session.rollback()

# Factory function: good for domain objects
def create_store_with_inventory(product, quantity):
    store = Store()
    store.add_inventory(product, quantity)
    return store
```

### Parameterized Tests

Group related scenarios that differ only by input. Keep positive and
negative cases together only when it's obvious which is which.

```python
@pytest.mark.parametrize("days_from_now, expected_valid", [(-1, False), (0, False), (2, True)])
def test_delivery_date_validity(days_from_now, expected_valid):
    service = DeliveryService()
    delivery = Delivery(date=datetime.now() + timedelta(days=days_from_now))
    assert service.is_delivery_valid(delivery) is expected_valid

def test_the_soonest_delivery_date_is_two_days_from_now(): ...
```

## Step 4: Apply Mocking Rules

### Mocks vs. Stubs

- Mock = emulates and verifies outgoing interactions (commands/side
  effects). Example: sending an email.
- Stub = emulates incoming interactions (queries). Example: returning
  data from a dependency.

Never assert interactions with stubs. How the system under test queries
data is an implementation detail.

### Managed vs. Unmanaged Dependencies

| Dependency type | Example | Strategy |
|---|---|---|
| Managed (only your app accesses it) | Your database | Use real instance, verify final state |
| Unmanaged (other systems observe it) | Message bus, SMTP, third-party API | Mock it, verify interactions |

> Mocking the database is one of the most common mistakes. Use a real
> database in integration tests.

### Mock at the System Edge

Mock the last type before the external call, not an intermediate wrapper.
This maximizes both protection against regressions (more code exercised)
and resistance to refactoring (you verify the actual external contract).

```python
# BAD — mocking an intermediate wrapper
bus_mock = Mock(spec=MessageBus)
bus_mock.send_email_changed.assert_called_once_with(user_id, email)

# GOOD — mocking at the edge, verifying the actual message
bus_client_mock = Mock(spec=BusClient)
bus = MessageBus(bus_client_mock)
bus_client_mock.send.assert_called_once_with(
    f"Type: USER EMAIL CHANGED; Id: {user_id}; NewEmail: {email}"
)
```

### Consider Spies at Frequently Tested Boundaries

A spy is a handwritten test double that records calls for later assertion.
Unlike `Mock()`, spies let you encapsulate assertion logic into fluent,
domain-specific methods (e.g., `spy.should_have_sent(1).with_message(...)`)
which makes tests read like specifications. This is especially valuable at
boundaries you test repeatedly — write the spy once and reuse it across
many tests. For one-off mocks, `Mock(spec=...)` is fine.

### Don't Mock Between Your Own Classes

Use the classical school: isolate tests from each other, not classes
from each other. Use real collaborators. A "unit" is a unit of behavior,
possibly spanning multiple classes. London-style mocking (mock all
collaborators) couples tests to implementation and makes them brittle.

## Anti-Patterns — Don't Do These

- Assertion-free tests — exercising code without verifying anything.
- Leaking domain knowledge — recomputing expected values in the test
  instead of hardcoding them.
- Exposing private state for testing — adding public API solely for
  test access. Tests interact with the system like production code does.
- Testing private methods directly — test them through public
  behavior. If a private method is too complex, extract it into its own
  class.
- Code pollution — adding production code (flags, test-only branches)
  solely for testing.
- Time as ambient context — inject time as an explicit value, don't
  call `datetime.now()` inside domain logic.
- Coverage targets — coverage is an indicator, not a goal. High
  coverage ≠ good tests.
- Mocking concrete classes to preserve partial behavior — this signals
  a Single Responsibility violation. Split the class instead.
