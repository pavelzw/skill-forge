# Textual Events Reference

Complete reference for the Textual event system, messages, and reactivity.

## Event Handling

### Handler Naming Convention

Handler methods are named `on_<snake_case_event>`:

```python
from textual.events import Key, Mount, Click

class MyWidget(Widget):
    def on_mount(self) -> None:
        """Called when widget is mounted."""

    def on_key(self, event: Key) -> None:
        """Called on key press."""

    def on_click(self, event: Click) -> None:
        """Called on mouse click."""
```

For widget messages, use `on_<widget>_<message>`:

```python
class MyApp(App):
    def on_button_pressed(self, event: Button.Pressed) -> None:
        """Handle any Button.Pressed message."""

    def on_input_changed(self, event: Input.Changed) -> None:
        """Handle any Input.Changed message."""
```

### The `@on` Decorator

Filter handlers with CSS selectors:

```python
from textual import on
from textual.widgets import Button, Input

class MyApp(App):
    @on(Button.Pressed, "#save")
    def handle_save(self) -> None:
        """Only handles Button.Pressed from #save."""

    @on(Button.Pressed, "#cancel")
    def handle_cancel(self) -> None:
        """Only handles Button.Pressed from #cancel."""

    @on(Input.Changed, ".search-field")
    def handle_search(self, event: Input.Changed) -> None:
        """Only handles Input.Changed from widgets with .search-field class."""
```

The decorator matches against the widget that sent the message, not the handler's widget.

### Async Handlers

Handlers can be sync or async:

```python
class MyApp(App):
    async def on_mount(self) -> None:
        await self.load_data()

    def on_key(self, event: Key) -> None:
        self.notify(event.key)
```

## Message Bubbling

Messages bubble up through the DOM tree (child -> parent -> grandparent -> screen -> app).

```python
class MyWidget(Widget):
    def on_click(self, event: Click) -> None:
        event.stop()             # Stop bubbling to parent
        event.prevent_default()  # Suppress default behavior
```

## Custom Messages

```python
from textual.message import Message

class FileList(Widget):
    class FileSelected(Message):
        """Emitted when a file is selected."""

        def __init__(self, path: str) -> None:
            super().__init__()
            self.path = path

        @property
        def control(self) -> "FileList":
            """The FileList that sent this message."""
            return self._sender  # type: ignore

    def select_file(self, path: str) -> None:
        self.post_message(self.FileSelected(path))
```

**Message class variables:**

| Variable | Default | Purpose |
|----------|---------|---------|
| `bubble` | `True` | Message bubbles up DOM |
| `verbose` | `False` | Verbose message (filtered in some contexts) |
| `no_dispatch` | `False` | Cannot be handled by client code |
| `namespace` | `""` | Namespace for handler name disambiguation |

## Lifecycle Events

### App/Widget Lifecycle

| Event | When | Bubbles |
|-------|------|---------|
| `Load` | App running, before terminal mode | No |
| `Mount` | Widget added to DOM | No |
| `Show` | Widget first displayed | No |
| `Hide` | Widget hidden (removed, scrolled away, display=False) | No |
| `Unmount` | Widget removed from DOM | No |
| `Resize` | App or widget resized | No |

**Lifecycle order:** Load -> Mount -> Show -> (Focus) -> ... -> (Blur) -> Hide -> Unmount

### Resize Event

```python
from textual.events import Resize

def on_resize(self, event: Resize) -> None:
    event.size           # New size
    event.virtual_size   # Scrollable size
    event.container_size # Container size
```

## Focus Events

| Event | When | Bubbles |
|-------|------|---------|
| `Focus` | Widget receives focus | No |
| `Blur` | Widget loses focus | No |
| `DescendantFocus` | Child widget focused | Yes |
| `DescendantBlur` | Child widget blurred | Yes |
| `AppFocus` | App gains terminal focus | No |
| `AppBlur` | App loses terminal focus | No |

```python
from textual.events import Focus, Blur

def on_focus(self, event: Focus) -> None:
    self.add_class("focused")

def on_blur(self, event: Blur) -> None:
    self.remove_class("focused")
```

## Key Events

```python
from textual.events import Key

def on_key(self, event: Key) -> None:
    event.key          # Key string (e.g., "ctrl+s", "a", "enter")
    event.character    # Printable character or None
    event.is_printable # Whether it's a printable character
    event.name         # Key name suitable for Python identifier
    event.aliases      # List of key aliases
```

### Key Methods

Alternative to `on_key`, handle specific keys:

```python
class MyWidget(Widget):
    def key_space(self) -> None:
        """Handle space key."""

    def key_ctrl_s(self) -> None:
        """Handle Ctrl+S."""
```

## Mouse Events

| Event | When | Bubbles |
|-------|------|---------|
| `Enter` | Mouse moves over widget | Yes |
| `Leave` | Mouse moves away from widget | Yes |
| `MouseMove` | Mouse moves while over widget | Yes |
| `MouseDown` | Mouse button pressed | Yes |
| `MouseUp` | Mouse button released | Yes |
| `Click` | Widget clicked | Yes |

### Click Event

```python
from textual.events import Click

def on_click(self, event: Click) -> None:
    event.x, event.y               # Relative coordinates
    event.screen_x, event.screen_y # Absolute coordinates
    event.button                    # Button index (1=left, 2=middle, 3=right)
    event.shift, event.ctrl, event.meta  # Modifier keys
    event.chain                     # Click count (1=single, 2=double, 3=triple)
```

### Mouse Capture

Force all mouse events to a specific widget:

```python
self.capture_mouse()    # Start capturing
self.release_mouse()    # Stop capturing
```

## Screen Events

| Event | When | Bubbles |
|-------|------|---------|
| `ScreenSuspend` | Screen is no longer active | No |
| `ScreenResume` | Screen becomes active | No |

## Other Events

### Paste

```python
from textual.events import Paste

def on_paste(self, event: Paste) -> None:
    event.text  # Pasted text
```

### Print

Capture `print()` output:

```python
self.begin_capture_print()

def on_print(self, event: Print) -> None:
    event.text    # Printed text
    event.stderr  # True if stderr
```

## Key Bindings

### Binding Class

```python
from textual.binding import Binding

BINDINGS = [
    # Simple tuple: (key, action, description)
    ("q", "quit", "Quit"),
    ("ctrl+s", "save", "Save"),

    # Full Binding for more control
    Binding("f1", "help", "Help", show=True),
    Binding("ctrl+z", "undo", "Undo", show=False),
    Binding("tab", "focus_next", priority=True),  # Priority: handled before focused widget
    Binding("escape", "dismiss", key_display="Esc"),  # Custom footer display
]
```

**Binding parameters:**

| Parameter | Type | Default | Purpose |
|-----------|------|---------|---------|
| `key` | `str` | required | Key to bind (comma-separated for multiple) |
| `action` | `str` | required | Action method name (without `action_` prefix) |
| `description` | `str` | `""` | Description shown in footer |
| `show` | `bool` | `True` | Show in footer |
| `key_display` | `str \| None` | `None` | Custom display text in footer |
| `priority` | `bool` | `False` | Handle before focused widget |
| `tooltip` | `str` | `""` | Tooltip in footer |

### Action Methods

Actions are methods prefixed with `action_`:

```python
class MyApp(App):
    BINDINGS = [("s", "save('draft')", "Save draft")]

    def action_save(self, mode: str = "final") -> None:
        """Parameters passed from action string."""
        self.notify(f"Saved as {mode}")
```

### Dynamic Actions

Control binding visibility/availability at runtime:

```python
class MyApp(App):
    def check_action(self, action: str, parameters: tuple) -> bool | None:
        """Return True=enabled, False=disabled (grayed), None=hidden."""
        if action == "save":
            if not self.has_changes:
                return False  # Show as disabled
        return True
```

### Built-in Actions

| Action | Description |
|--------|-------------|
| `quit` | Exit the app |
| `bell` | Terminal bell |
| `focus_next` | Focus next widget |
| `focus_previous` | Focus previous widget |
| `toggle_dark` | Toggle dark mode |
| `screenshot` | Take screenshot |
| `command_palette` | Open command palette |
| `maximize` | Maximize focused widget |
| `minimize` | Restore maximized widget |

### Action Namespaces

```python
# In CSS links or action strings:
"app.quit"        # Call action on the App
"screen.dismiss"  # Call action on the current Screen
"focused.delete"  # Call action on the focused widget
```

## Reactive System

### reactive vs var

```python
from textual.reactive import reactive, var

class MyWidget(Widget):
    # reactive: triggers repaint on change (default)
    name: reactive[str] = reactive("default")

    # var: no automatic repaint
    count: var[int] = var(0)
```

### Reactive Options

```python
# All options
value: reactive[int] = reactive(
    0,                    # Default value (or callable for mutable defaults)
    layout=False,         # Recalculate layout on change
    repaint=True,         # Repaint widget on change
    init=False,           # Call watcher on mount
    always_update=False,  # Call watcher even if value unchanged
    recompose=False,      # Rebuild compose tree on change
    bindings=False,       # Refresh bindings on change
)
```

### Watchers

Called when a reactive changes:

```python
class MyWidget(Widget):
    count: reactive[int] = reactive(0)

    # 0 args
    def watch_count(self) -> None:
        self.refresh()

    # 1 arg (new value)
    def watch_count(self, new_value: int) -> None:
        self.log(f"Count is now {new_value}")

    # 2 args (old and new)
    def watch_count(self, old_value: int, new_value: int) -> None:
        self.log(f"Count: {old_value} -> {new_value}")
```

Watchers can also be async:

```python
async def watch_query(self, query: str) -> None:
    results = await self.search(query)
    self.update_results(results)
```

### Dynamic Watcher

Watch a reactive from outside the class:

```python
self.watch(widget, "count", self.on_count_changed)
```

### Validators

Called before a reactive is set:

```python
class MyWidget(Widget):
    count: reactive[int] = reactive(0)

    def validate_count(self, value: int) -> int:
        """Clamp count to 0-100."""
        return max(0, min(100, value))
```

### Compute Methods

Derived reactive values:

```python
class MyWidget(Widget):
    first_name: reactive[str] = reactive("")
    last_name: reactive[str] = reactive("")
    full_name: reactive[str] = reactive("")

    def compute_full_name(self) -> str:
        return f"{self.first_name} {self.last_name}"
```

### Data Binding

Bind parent reactive to child:

```python
class Child(Widget):
    value: reactive[str] = reactive("")

class Parent(Widget):
    value: reactive[str] = reactive("hello")

    def compose(self) -> ComposeResult:
        yield Child().data_bind(Parent.value)
```

### Mutable Reactives

For mutable types (lists, dicts), use `mutate_reactive`:

```python
class MyWidget(Widget):
    items: reactive[list] = reactive(list)

    def add_item(self, item: str) -> None:
        self.items.append(item)
        self.mutate_reactive(MyWidget.items)  # Trigger watchers
```

### set_reactive

Set a reactive without triggering watchers:

```python
self.set_reactive(MyWidget.count, 42)
```
