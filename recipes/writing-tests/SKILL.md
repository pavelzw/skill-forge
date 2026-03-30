---
name: unit-testing
description: >
  Write high-quality unit and integration tests for Python code following
  value-driven testing principles. Use this skill whenever the user asks you to
  write tests, add test coverage, create a test suite, write unit tests,
  integration tests, or review existing tests for quality. Also trigger when
  the user mentions pytest, testing strategy, mocking best practices, test
  refactoring, test design, TDD, or asks "how should I test this?". This skill
  ensures every test you write is valuable, maintainable, and resistant to
  refactoring. Always consult this skill before writing any test code.
---

# Unit Testing Skill

## Scope

This skill governs how you write **tests only**. Do not refactor or modify
the user's production code. If you identify production code that would benefit
from refactoring (e.g., to improve testability), mention it as a suggestion
but do not make the change yourself.

Not all tests are equal. A test suite full of low-value tests is worse than
a small suite of high-value ones because every test has a maintenance
cost. Write tests that maximize value while minimizing that cost.

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

### Parameterized Tests

Group related scenarios that differ only by input. Keep positive and
negative cases together only when it's obvious which is which.

```python
@pytest.mark.parametrize("days_from_now", [-1, 0, 1])
def test_detects_an_invalid_delivery_date(days_from_now): ...

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

### Prefer Spies at Boundaries

Handwritten spy classes encapsulate assertion logic and improve
readability over raw `Mock()` calls.

```python
class BusClientSpy:
    def __init__(self):
        self.sent_messages: list[str] = []

    def send(self, message: str) -> None:
        self.sent_messages.append(message)

    def should_have_sent(self, count: int) -> "BusClientSpy":
        assert len(self.sent_messages) == count
        return self

    def with_email_changed_message(self, user_id, email) -> "BusClientSpy":
        expected = f"Type: USER EMAIL CHANGED; Id: {user_id}; NewEmail: {email}"
        assert expected in self.sent_messages
        return self
```

### Don't Mock Between Your Own Classes

Use the classical school: isolate tests from each other, not classes
from each other. Use real collaborators. A "unit" is a unit of behavior,
possibly spanning multiple classes. London-style mocking (mock all
collaborators) couples tests to implementation and makes them brittle.

## Step 5: Review Against the Four Pillars

Every test must score well on all four. If any is zero, the test's value
is zero.

1. Protection against regressions — Does it exercise meaningful,
   complex code paths?
2. Resistance to refactoring — Will it break if implementation changes
   but behavior stays the same? (Non-negotiable — treat as binary.)
3. Fast feedback — Does it run in milliseconds?
4. Maintainability — Is it short, readable, free of complex setup?

The trade-off is between (1) and (3). Never compromise on (2).

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
