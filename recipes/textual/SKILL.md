---
name: textual
description: >-
  Build terminal user interfaces (TUIs) with Textual, a Python framework for
  creating rich, interactive applications in the terminal. Handles widgets,
  layouts, CSS styling, events, screens, and async workers. Use when building
  TUI apps or when the user mentions textual.
license: MIT
---

# Textual Skill

Textual is a Python framework for building rich terminal user interfaces (TUIs). It provides a widget toolkit with CSS-based styling, an event system with message passing, reactive data binding, screen management, and async worker support.

## When to Use This Skill

Use Textual when:
- Building interactive terminal applications
- Creating dashboards, forms, or data browsers in the terminal
- Need rich UI with layout, styling, and input handling
- Want CSS-like styling for terminal apps
- Building apps with multiple screens or modal dialogs

## Architecture

```
App (application root)
 └── Screen (layered views, one active at a time)
      └── Widget (UI components, nested in a DOM tree)
           └── Child Widgets...
```

- **App** is the top-level container. It manages screens, themes, key bindings, and the event loop.
- **Screen** is a full-screen layer within the app. Screens stack; only the top screen is visible.
- **Widget** is a UI component. Widgets form a DOM tree, can have children, and are styled with CSS.

## Creating an App

```python
from textual.app import App, ComposeResult
from textual.widgets import Header, Footer, Static

class MyApp(App):
    """A minimal Textual application."""

    CSS = """
    Screen {
        align: center middle;
    }
    #greeting {
        width: 40;
        padding: 1 2;
        border: solid green;
        text-align: center;
    }
    """

    BINDINGS = [
        ("q", "quit", "Quit"),
        ("d", "toggle_dark", "Toggle dark mode"),
    ]

    def compose(self) -> ComposeResult:
        yield Header()
        yield Static("Hello, World!", id="greeting")
        yield Footer()

    def action_toggle_dark(self) -> None:
        self.theme = "textual-light" if self.theme == "textual-dark" else "textual-dark"

if __name__ == "__main__":
    MyApp().run()
```

### App Class Variables

| Variable | Type | Purpose |
|----------|------|---------|
| `CSS` | `str` | Inline CSS rules |
| `CSS_PATH` | `str \| list[str]` | Path(s) to `.tcss` CSS files |
| `BINDINGS` | `list[BindingType]` | Key bindings |
| `TITLE` | `str` | App title (shown in Header) |
| `SUB_TITLE` | `str` | App subtitle |
| `SCREENS` | `dict[str, Callable]` | Named screen factories |
| `MODES` | `dict[str, str \| Callable]` | App modes with base screens |
| `COMMANDS` | `set[type[Provider]]` | Command palette providers |
| `ENABLE_COMMAND_PALETTE` | `bool` | Enable command palette (default: `True`) |

### Key App Methods

```python
# Run the app
app.run()

# Exit with optional return value
app.exit(result=value)

# Screen management
app.push_screen(screen, callback=None)        # Push screen onto stack
await app.push_screen_wait(screen)             # Push and await dismiss result
app.pop_screen()                               # Pop top screen
app.switch_screen(screen)                      # Replace top screen
app.install_screen(screen, name="name")        # Install named screen

# Mode management
app.switch_mode("mode_name")

# Widget management
app.mount(widget)                              # Mount widget on current screen
app.query(selector)                            # Query DOM with CSS selector
app.query_one("#my-id", Widget)                # Query single widget
app.set_focus(widget)                          # Set focus to widget

# Notifications
app.notify("Message", title="Title", severity="information")  # information/warning/error

# Timers
app.set_timer(delay, callback)                 # One-shot timer
app.set_interval(interval, callback)           # Repeating timer
app.call_later(callback)                       # Call on next idle

# Workers
app.run_worker(async_func, thread=False)       # Run async or threaded worker

# Suspend terminal to run external process
async with app.suspend():
    os.system("vim file.txt")
```

## Widgets

### Built-in Widgets

Textual ships with 35+ widgets. See [references/WIDGETS.md](references/WIDGETS.md) for the complete reference. Key widgets:

| Widget | Purpose |
|--------|---------|
| `Header`, `Footer` | App chrome with title and key bindings |
| `Static`, `Label` | Display text |
| `Button` | Clickable button with variants |
| `Input`, `TextArea` | Text input (single/multi-line) |
| `Select`, `SelectionList` | Dropdown and multi-select |
| `DataTable` | Interactive tabular data |
| `Tree`, `DirectoryTree` | Hierarchical tree views |
| `ListView` | Scrollable list of items |
| `TabbedContent` | Tabbed content panels |
| `Markdown` | Render markdown content |
| `ProgressBar` | Progress indicator |
| `RichLog` | Scrollable formatted log |
| `Checkbox`, `RadioSet`, `Switch` | Toggle controls |

### Custom Widgets

```python
from textual.widget import Widget
from textual.reactive import reactive

class Counter(Widget):
    """A custom counter widget."""

    DEFAULT_CSS = """
    Counter {
        height: auto;
        padding: 1 2;
        border: solid $accent;
    }
    """

    BINDINGS = [
        ("up", "increment", "Increment"),
        ("down", "decrement", "Decrement"),
    ]

    can_focus = True
    count: reactive[int] = reactive(0)

    def render(self) -> str:
        return f"Count: {self.count}"

    def action_increment(self) -> None:
        self.count += 1

    def action_decrement(self) -> None:
        self.count -= 1

    def watch_count(self, old_value: int, new_value: int) -> None:
        """Called when count changes."""
        if new_value > 10:
            self.notify("Count exceeded 10!", severity="warning")
```

### Composition

Build widget trees with `compose()`. Use context managers for containers:

```python
from textual.containers import Horizontal, Vertical, Grid

def compose(self) -> ComposeResult:
    yield Header()
    with Horizontal():
        yield Button("OK", variant="primary", id="ok")
        yield Button("Cancel", id="cancel")
    yield Footer()
```

### Containers

| Container | Layout | Description |
|-----------|--------|-------------|
| `Container` | vertical | Basic expanding container |
| `Vertical` | vertical | Expanding, no scrollbars |
| `VerticalGroup` | vertical | Auto height, no scrollbars |
| `VerticalScroll` | vertical | With vertical scrollbar |
| `Horizontal` | horizontal | Expanding, no scrollbars |
| `HorizontalGroup` | horizontal | Auto height, no scrollbars |
| `HorizontalScroll` | horizontal | With horizontal scrollbar |
| `ScrollableContainer` | vertical | Full scrollbars, focusable |
| `Center` | vertical | Horizontal center alignment |
| `Middle` | vertical | Vertical middle alignment |
| `Grid` | grid | Grid layout |

### Key Widget Methods

```python
# Composition and mounting
widget.mount(child)                      # Add child widget
widget.mount(child, before=other)        # Insert before another
await widget.remove()                    # Remove from DOM
widget.remove_children()                 # Remove all children
await widget.recompose()                 # Rebuild compose tree

# Rendering
widget.refresh()                         # Request repaint
widget.refresh(layout=True)              # Request layout recalc

# Focus
widget.focus()                           # Take focus
widget.blur()                            # Release focus

# Scrolling
widget.scroll_to(x, y, animate=True)
widget.scroll_home()
widget.scroll_end()
widget.scroll_to_widget(child)
widget.scroll_visible()

# Animation
widget.animate("opacity", 1.0, duration=0.5)

# Data binding
child.data_bind(Counter.count)           # Bind reactive from parent
```

## Textual CSS

Textual uses a CSS-like language (TCSS) for styling. See [references/CSS.md](references/CSS.md) for the complete property reference.

### CSS Sources

```python
# Inline CSS on the class
class MyApp(App):
    CSS = """
    Screen { background: $surface; }
    """

# External .tcss file
class MyApp(App):
    CSS_PATH = "my_app.tcss"

# Widget-scoped CSS
class MyWidget(Widget):
    DEFAULT_CSS = """
    MyWidget { height: auto; padding: 1; }
    """
    SCOPED_CSS = True  # Default: True. Selectors only match within this widget.
```

### Selectors

| Selector | Syntax | Example |
|----------|--------|---------|
| Type | `WidgetType` | `Button { color: red; }` |
| ID | `#id` | `#sidebar { width: 30; }` |
| Class | `.class` | `.error { color: red; }` |
| Universal | `*` | `* { margin: 1; }` |
| Pseudo-class | `:state` | `Button:hover { background: $accent; }` |
| Child | `Parent > Child` | `Horizontal > Button { width: 1fr; }` |
| Descendant | `Ancestor Descendant` | `Screen Input { border: solid; }` |
| Nesting | `&` | `& > .child { ... }` (inside nested rules) |

**Pseudo-classes:** `:hover`, `:focus`, `:focus-within`, `:disabled`, `:enabled`, `:dark`, `:light`, `:even`, `:odd`, `:first-child`, `:last-child`, `:blur`, `:can-focus`, `:inline`

### Key CSS Properties

| Category | Properties |
|----------|-----------|
| **Layout** | `layout` (vertical/horizontal/grid), `display`, `dock`, `align`, `content-align` |
| **Sizing** | `width`, `height`, `min-width`, `max-width`, `min-height`, `max-height` |
| **Spacing** | `margin`, `padding`, `offset` |
| **Colors** | `background`, `color`, `tint`, `opacity`, `text-opacity` |
| **Borders** | `border`, `outline`, `border-title-align` |
| **Text** | `text-align`, `text-style`, `text-wrap`, `text-overflow` |
| **Scrolling** | `overflow`, `scrollbar-size`, `scrollbar-color` |
| **Grid** | `grid-size`, `grid-columns`, `grid-rows`, `grid-gutter`, `column-span`, `row-span` |
| **Position** | `position` (relative/absolute), `layer`, `layers` |

### Units

| Unit | Description | Example |
|------|-------------|---------|
| (number) | Cell units | `width: 30;` |
| `%` | Percentage of parent | `width: 50%;` |
| `fr` | Fraction of remaining space | `width: 1fr;` |
| `vw` / `vh` | Viewport width/height | `width: 50vw;` |
| `w` / `h` | Container width/height % | `width: 50w;` |
| `auto` | Automatic sizing | `height: auto;` |

### CSS Variables

```css
/* Use theme variables */
Screen {
    background: $surface;
    color: $text;
    border: solid $accent;
}

/* Custom variables */
$my-color: red;
.highlight { background: $my-color; }
```

## Events and Messages

### Handler Naming Convention

Event handlers are methods named `on_<event_name>`:

```python
from textual import on
from textual.events import Key, Mount
from textual.widgets import Button

class MyApp(App):
    def on_mount(self) -> None:
        """Called when app is mounted."""
        self.title = "My App"

    def on_key(self, event: Key) -> None:
        """Called on any key press."""
        self.notify(f"Key: {event.key}")

    # Widget-specific message handler (ClassName_MessageName)
    def on_button_pressed(self, event: Button.Pressed) -> None:
        self.notify(f"Button pressed: {event.button.id}")
```

### The `@on` Decorator

Use `@on` for CSS-selector-filtered handlers:

```python
from textual import on

class MyApp(App):
    @on(Button.Pressed, "#ok")
    def handle_ok(self) -> None:
        self.notify("OK pressed")

    @on(Button.Pressed, "#cancel")
    def handle_cancel(self) -> None:
        self.notify("Cancelled")

    @on(Input.Changed, "#search")
    def handle_search(self, event: Input.Changed) -> None:
        self.notify(f"Search: {event.value}")
```

### Custom Messages

```python
from textual.message import Message

class MyWidget(Widget):
    class Selected(Message):
        """Emitted when item is selected."""
        def __init__(self, item: str) -> None:
            super().__init__()
            self.item = item

    def on_click(self) -> None:
        self.post_message(self.Selected("item-1"))
```

Messages bubble up through the DOM. Use `event.stop()` to stop propagation. Use `event.prevent_default()` to suppress default behavior.

### Common Events

See [references/EVENTS.md](references/EVENTS.md) for the complete reference.

| Event | When |
|-------|------|
| `Mount` | Widget added to DOM |
| `Unmount` | Widget removed from DOM |
| `Key` | Key pressed |
| `Click` | Widget clicked |
| `Focus` / `Blur` | Widget gains/loses focus |
| `Resize` | App or widget resized |
| `Show` / `Hide` | Widget becomes visible/hidden |

## Key Bindings and Actions

```python
from textual.binding import Binding

class MyApp(App):
    BINDINGS = [
        # Simple: (key, action, description)
        ("q", "quit", "Quit"),
        ("d", "toggle_dark", "Dark mode"),

        # Full Binding object for more control
        Binding("ctrl+s", "save", "Save", show=True, priority=True),
        Binding("ctrl+z", "undo", "Undo", show=False),
    ]

    def action_save(self) -> None:
        """Action methods are prefixed with action_."""
        self.notify("Saved!")

    # Dynamic actions: return True/False/None to enable/disable/hide
    def check_action(self, action: str, parameters: tuple) -> bool | None:
        if action == "save" and not self.has_changes:
            return False  # Disable (grayed out in footer)
        return True
```

**Built-in actions:** `quit`, `bell`, `focus_next`, `focus_previous`, `toggle_dark`, `screenshot`, `command_palette`, `maximize`, `minimize`

## Reactivity

```python
from textual.reactive import reactive, var

class MyWidget(Widget):
    # reactive: triggers refresh on change
    name: reactive[str] = reactive("default")

    # var: no automatic refresh
    count: var[int] = var(0)

    # With options
    items: reactive[list] = reactive(list, layout=True, recompose=True)

    # Watcher: called when value changes
    def watch_name(self, old_value: str, new_value: str) -> None:
        self.log(f"Name changed: {old_value} -> {new_value}")

    # Validator: called before value is set
    def validate_count(self, value: int) -> int:
        return max(0, value)  # Clamp to non-negative

    # Compute: derived value
    display_name: reactive[str] = reactive("")
    def compute_display_name(self) -> str:
        return self.name.upper()
```

### Reactive Options

| Option | Default | Effect |
|--------|---------|--------|
| `repaint` | `True` | Refresh widget on change |
| `layout` | `False` | Recalculate layout on change |
| `init` | `False` | Call watcher on mount |
| `always_update` | `False` | Call watcher even if value unchanged |
| `recompose` | `False` | Rebuild compose tree on change |
| `bindings` | `False` | Refresh bindings on change |

### Data Binding

Pass reactive values from parent to child:

```python
class Child(Widget):
    value: reactive[str] = reactive("")

class Parent(Widget):
    value: reactive[str] = reactive("hello")

    def compose(self) -> ComposeResult:
        yield Child().data_bind(Parent.value)
```

## Screens

See [references/SCREENS.md](references/SCREENS.md) for the complete reference.

```python
from textual.screen import Screen, ModalScreen

class SettingsScreen(Screen):
    BINDINGS = [("escape", "dismiss")]

    def compose(self) -> ComposeResult:
        yield Static("Settings")
        yield Button("Close", id="close")

    def on_button_pressed(self) -> None:
        self.dismiss()

# Modal with result
class ConfirmDialog(ModalScreen[bool]):
    def compose(self) -> ComposeResult:
        with Vertical():
            yield Static("Are you sure?")
            with Horizontal():
                yield Button("Yes", id="yes", variant="primary")
                yield Button("No", id="no")

    @on(Button.Pressed, "#yes")
    def confirm(self) -> None:
        self.dismiss(True)

    @on(Button.Pressed, "#no")
    def cancel(self) -> None:
        self.dismiss(False)

# Usage
class MyApp(App):
    def action_settings(self) -> None:
        self.push_screen(SettingsScreen())

    def action_confirm(self) -> None:
        self.push_screen(ConfirmDialog(), callback=self.handle_confirm)

    def handle_confirm(self, confirmed: bool) -> None:
        if confirmed:
            self.notify("Confirmed!")
```

## Workers

See [references/WORKERS.md](references/WORKERS.md) for the complete reference.

```python
from textual.worker import Worker, get_current_worker
from textual import work

class MyApp(App):
    # Decorator approach
    @work(exclusive=True)
    async def fetch_data(self, url: str) -> None:
        worker = get_current_worker()
        response = await fetch(url)
        if not worker.is_cancelled:
            self.notify(f"Got {len(response)} bytes")

    # Thread worker for blocking I/O
    @work(thread=True)
    def load_file(self, path: str) -> str:
        with open(path) as f:
            return f.read()
```

## Testing

See [references/TESTING.md](references/TESTING.md) for the complete reference.

```python
import pytest
from my_app import MyApp

@pytest.mark.asyncio
async def test_app():
    async with MyApp().run_test() as pilot:
        await pilot.press("q")       # Press key
        await pilot.click("#ok")     # Click widget
        await pilot.pause()          # Wait for messages

        # Assert on app state
        assert pilot.app.query_one("#label", Static).renderable == "Hello"
```

## Additional References

- [Widgets Reference](references/WIDGETS.md) - All built-in widgets
- [CSS Reference](references/CSS.md) - All CSS properties, types, and selectors
- [Events Reference](references/EVENTS.md) - Events, messages, and reactivity details
- [Screens Reference](references/SCREENS.md) - Screen management and modes
- [Testing Reference](references/TESTING.md) - Testing with Pilot
- [Workers Reference](references/WORKERS.md) - Async workers and threading
