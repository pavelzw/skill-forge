# Textual Testing Reference

Complete reference for testing Textual applications.

## Setup

Textual tests use `pytest` with `pytest-asyncio`:

```bash
pip install pytest pytest-asyncio
```

## Basic Testing

Use `App.run_test()` for headless testing:

```python
import pytest
from textual.widgets import Static, Button, Input
from my_app import MyApp

@pytest.mark.asyncio
async def test_initial_state():
    async with MyApp().run_test() as pilot:
        app = pilot.app
        assert app.title == "My App"
        label = app.query_one("#greeting", Static)
        assert "Hello" in str(label.renderable)
```

### run_test Parameters

```python
async with app.run_test(
    headless=True,          # No terminal output (default: True)
    size=(80, 24),          # Terminal size (columns, rows)
    message_hook=callback,  # Called for every message
) as pilot:
    ...
```

## Pilot API

The `Pilot` object simulates user interaction.

### Keyboard Input

```python
async with MyApp().run_test() as pilot:
    await pilot.press("a")                  # Press a single key
    await pilot.press("ctrl+s")             # Press key combo
    await pilot.press("tab", "tab", "enter") # Multiple keys in sequence
    await pilot.press("escape")
```

### Mouse Clicks

```python
async with MyApp().run_test() as pilot:
    # Click a widget by selector
    await pilot.click("#submit")

    # Click by widget type
    await pilot.click(Button)

    # Click with offset (relative to widget center)
    await pilot.click("#canvas", offset=(10, 5))

    # Click with modifiers
    await pilot.click("#item", shift=True)
    await pilot.click("#item", control=True)

    # Double/triple click
    await pilot.double_click("#text")
    await pilot.triple_click("#text")

    # Mouse button (1=left, 2=middle, 3=right)
    await pilot.click("#item", button=3)
```

### Mouse Actions

```python
async with MyApp().run_test() as pilot:
    # Hover over a widget
    await pilot.hover("#menu-item")

    # Mouse down/up separately
    await pilot.mouse_down("#drag-handle")
    await pilot.mouse_up("#drop-target")
```

### Pausing

Wait for pending messages to be processed:

```python
async with MyApp().run_test() as pilot:
    await pilot.press("a")
    await pilot.pause()  # Wait for all pending messages

    # Or with a delay
    await pilot.pause(delay=0.5)
```

### Resizing

```python
async with MyApp().run_test() as pilot:
    await pilot.resize_terminal(120, 40)
    await pilot.pause()
```

### Exiting

```python
async with MyApp().run_test() as pilot:
    await pilot.exit(return_value)
```

### Waiting for Animations

```python
async with MyApp().run_test() as pilot:
    await pilot.wait_for_animation()
    await pilot.wait_for_scheduled_animations()
```

## Querying Widgets

Use CSS selectors to find widgets in tests:

```python
async with MyApp().run_test() as pilot:
    app = pilot.app

    # Query single widget
    button = app.query_one("#submit", Button)
    assert button.label == "Submit"

    # Query multiple widgets
    buttons = app.query(Button)
    assert len(buttons) == 3

    # Check widget state
    input_widget = app.query_one("#name", Input)
    assert input_widget.value == "default"

    # Check CSS classes
    widget = app.query_one("#status")
    assert widget.has_class("active")

    # Check display/visibility
    assert app.query_one("#panel").display is True
    assert app.query_one("#hidden").visible is False
```

## Testing Patterns

### Testing Key Bindings

```python
@pytest.mark.asyncio
async def test_quit_binding():
    async with MyApp().run_test() as pilot:
        await pilot.press("q")
        # App should have exited
        assert pilot.app.return_code == 0
```

### Testing Screen Navigation

```python
@pytest.mark.asyncio
async def test_screen_push():
    async with MyApp().run_test() as pilot:
        await pilot.press("s")  # Trigger settings screen
        await pilot.pause()

        # Verify new screen
        assert isinstance(pilot.app.screen, SettingsScreen)

        await pilot.press("escape")  # Go back
        await pilot.pause()
        assert not isinstance(pilot.app.screen, SettingsScreen)
```

### Testing Input

```python
@pytest.mark.asyncio
async def test_form_input():
    async with MyApp().run_test() as pilot:
        # Focus the input
        await pilot.click("#name-input")

        # Type text
        await pilot.press("H", "e", "l", "l", "o")
        await pilot.pause()

        input_widget = pilot.app.query_one("#name-input", Input)
        assert input_widget.value == "Hello"

        # Submit
        await pilot.press("enter")
        await pilot.pause()
```

### Testing Reactive Updates

```python
@pytest.mark.asyncio
async def test_counter():
    async with MyApp().run_test() as pilot:
        counter = pilot.app.query_one(Counter)
        assert counter.count == 0

        await pilot.press("up")
        await pilot.pause()
        assert counter.count == 1

        await pilot.press("down")
        await pilot.pause()
        assert counter.count == 0
```

### Testing Data Table

```python
@pytest.mark.asyncio
async def test_data_table():
    async with MyApp().run_test() as pilot:
        table = pilot.app.query_one(DataTable)
        assert table.row_count == 5
        assert table.get_cell_at((0, 0)) == "Alice"
```

### Testing with Workers

```python
@pytest.mark.asyncio
async def test_async_loading():
    async with MyApp().run_test() as pilot:
        await pilot.press("l")  # Trigger load action
        await pilot.pause(delay=1.0)  # Wait for worker

        results = pilot.app.query_one("#results")
        assert "loaded" in str(results.renderable).lower()
```

### Testing Notifications

```python
@pytest.mark.asyncio
async def test_notification():
    async with MyApp().run_test(notifications=True) as pilot:
        await pilot.press("s")  # Trigger save
        await pilot.pause()

        # Check notifications
        assert len(pilot.app._notifications) > 0
```

## Snapshot Testing

Visual regression testing with `pytest-textual-snapshot`:

```bash
pip install pytest-textual-snapshot
```

```python
from textual.app import App

async def test_snapshot(snap_compare):
    assert snap_compare(MyApp())
```

Run with `--update-snapshot` to generate/update reference images:

```bash
pytest --update-snapshot
```

### Snapshot with Interaction

```python
async def test_snapshot_after_click(snap_compare):
    async def setup(pilot):
        await pilot.click("#submit")
        await pilot.pause()

    assert snap_compare(MyApp(), run_before=setup)
```

### Snapshot with Custom Size

```python
async def test_snapshot_large(snap_compare):
    assert snap_compare(MyApp(), terminal_size=(120, 40))
```

## Testing Tips

- Always call `await pilot.pause()` after interactions to let messages process
- Use `size` parameter in `run_test()` to test responsive layouts
- Use `message_hook` to spy on messages for debugging
- For workers, use `pilot.pause(delay=...)` with appropriate delays
- Test both the happy path and edge cases (empty data, long text, etc.)
