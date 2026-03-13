# Textual Screens Reference

Complete reference for screen management, modals, and modes.

## Screen Basics

A Screen is a top-level widget that occupies the entire terminal. Apps have a screen stack; only the topmost screen is visible.

```python
from textual.screen import Screen
from textual.app import ComposeResult
from textual.widgets import Static, Button

class SettingsScreen(Screen):
    CSS = """
    SettingsScreen {
        align: center middle;
    }
    """

    BINDINGS = [("escape", "dismiss")]

    def compose(self) -> ComposeResult:
        yield Static("Settings")
        yield Button("Close", id="close")

    def on_button_pressed(self) -> None:
        self.dismiss()
```

### Screen Class Variables

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `CSS` | `str` | `""` | Inline CSS for screen |
| `CSS_PATH` | `str \| list` | `None` | CSS file path(s) |
| `BINDINGS` | `list[BindingType]` | `[]` | Key bindings |
| `AUTO_FOCUS` | `str \| None` | `"*"` | CSS selector for auto-focus on mount |

## Screen Stack

### Push Screen

Add a screen to the top of the stack:

```python
class MyApp(App):
    def action_settings(self) -> None:
        self.push_screen(SettingsScreen())

    # With callback for result
    def action_confirm(self) -> None:
        self.push_screen(ConfirmDialog(), callback=self.on_confirm)

    def on_confirm(self, result: bool) -> None:
        if result:
            self.notify("Confirmed!")
```

### Push Screen and Wait

Await the result directly:

```python
class MyApp(App):
    async def action_confirm(self) -> None:
        result = await self.push_screen_wait(ConfirmDialog())
        if result:
            self.notify("Confirmed!")
```

### Pop Screen

Remove the top screen and return to the previous one:

```python
self.pop_screen()
```

### Switch Screen

Replace the top screen (no stacking):

```python
self.switch_screen(NewScreen())
```

### Dismiss

Pop the current screen from within the screen itself, optionally returning a result:

```python
class MyScreen(Screen):
    def action_dismiss(self) -> None:
        self.dismiss()           # Pop with no result

    def confirm(self) -> None:
        self.dismiss(result=42)  # Pop and return value to callback
```

## Named Screens

Pre-register screens by name:

```python
class MyApp(App):
    SCREENS = {
        "settings": SettingsScreen,
        "help": lambda: HelpScreen("guide.md"),
    }

    def action_settings(self) -> None:
        self.push_screen("settings")
```

### Install/Uninstall Screens

```python
self.install_screen(SettingsScreen(), name="settings")
self.uninstall_screen("settings")
```

## Modal Screens

Modal screens block interaction with screens below. Use `ModalScreen` base class.

```python
from textual.screen import ModalScreen

class ConfirmDialog(ModalScreen[bool]):
    """A modal that returns True/False."""

    CSS = """
    ConfirmDialog {
        align: center middle;
    }

    #dialog {
        width: 40;
        height: auto;
        padding: 1 2;
        border: thick $accent;
        background: $surface;
    }
    """

    def compose(self) -> ComposeResult:
        with Vertical(id="dialog"):
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
```

### Type-Safe Results

`ModalScreen` is generic. Specify the result type:

```python
class TextInputDialog(ModalScreen[str]):
    def confirm(self) -> None:
        value = self.query_one(Input).value
        self.dismiss(value)

# Usage:
def action_get_name(self) -> None:
    self.push_screen(TextInputDialog(), callback=self.set_name)

def set_name(self, name: str) -> None:
    self.title = name
```

## Modes

Modes allow multiple independent screen stacks. Each mode has its own base screen.

```python
class MyApp(App):
    MODES = {
        "dashboard": DashboardScreen,
        "settings": SettingsScreen,
        "editor": lambda: EditorScreen("default.txt"),
    }

    BINDINGS = [
        ("1", "switch_mode('dashboard')", "Dashboard"),
        ("2", "switch_mode('settings')", "Settings"),
        ("3", "switch_mode('editor')", "Editor"),
    ]

    def on_mount(self) -> None:
        self.switch_mode("dashboard")  # Set initial mode
```

Each mode maintains its own screen stack. Switching modes preserves the stack of the previous mode.

### Mode Management

```python
# Switch to a mode
self.switch_mode("editor")

# Add a mode dynamically
self.add_mode("preview", PreviewScreen)

# Remove a mode
self.remove_mode("preview")
```

## Screen Lifecycle Events

| Event | When |
|-------|------|
| `Mount` | Screen is mounted (first time or reinstalled) |
| `ScreenSuspend` | Screen is no longer the active screen |
| `ScreenResume` | Screen becomes the active screen again |
| `Unmount` | Screen is removed from the DOM |

```python
class MyScreen(Screen):
    def on_screen_resume(self) -> None:
        """Called when this screen becomes active again."""
        self.refresh_data()

    def on_screen_suspend(self) -> None:
        """Called when another screen covers this one."""
        self.pause_updates()
```

## Screen Opacity

Screens can have transparent backgrounds, allowing the screen below to show through:

```python
class OverlayScreen(ModalScreen):
    CSS = """
    OverlayScreen {
        background: rgba(0, 0, 0, 0.5);
        align: center middle;
    }
    """
```

## Common Patterns

### Confirmation Dialog

```python
class ConfirmScreen(ModalScreen[bool]):
    def __init__(self, message: str) -> None:
        self.message = message
        super().__init__()

    def compose(self) -> ComposeResult:
        with Vertical(id="dialog"):
            yield Static(self.message)
            with Horizontal():
                yield Button("OK", id="ok", variant="primary")
                yield Button("Cancel", id="cancel")

    @on(Button.Pressed, "#ok")
    def confirm(self) -> None:
        self.dismiss(True)

    @on(Button.Pressed, "#cancel")
    def cancel(self) -> None:
        self.dismiss(False)

# Usage:
async def action_delete(self) -> None:
    if await self.push_screen_wait(ConfirmScreen("Delete item?")):
        self.delete_item()
```

### Screen with Navigation

```python
class DetailScreen(Screen):
    BINDINGS = [
        ("escape", "app.pop_screen", "Back"),
        ("n", "next", "Next"),
        ("p", "previous", "Previous"),
    ]

    def __init__(self, item_id: int) -> None:
        self.item_id = item_id
        super().__init__()

    def compose(self) -> ComposeResult:
        yield Header()
        yield Static(f"Item {self.item_id}")
        yield Footer()

    def action_next(self) -> None:
        self.app.switch_screen(DetailScreen(self.item_id + 1))
```
