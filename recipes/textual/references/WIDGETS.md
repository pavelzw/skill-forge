# Textual Widgets Reference

Complete reference for all built-in Textual widgets.

## Display Widgets

### Static

Display static text or Rich renderables.

```python
from textual.widgets import Static

yield Static("Hello, World!")
yield Static("Rich [bold]markup[/bold] supported")
```

- **Focusable:** No | **Container:** No
- **Key method:** `update(content)` to change displayed content

### Label

Display simple text. Subclass of Static.

```python
from textual.widgets import Label

yield Label("Username:")
```

- **Focusable:** No | **Container:** No

### Digits

Display numbers in large multi-line characters. Supports 0-9, A-F, +, -, ^, :, x.

```python
from textual.widgets import Digits

yield Digits("12:45")
```

- **Focusable:** No | **Container:** No
- **Key method:** `update(text)` to change display
- Respects `text-align` CSS

### Pretty

Display pretty-formatted Python objects.

```python
from textual.widgets import Pretty

yield Pretty({"key": [1, 2, 3]})
```

- **Focusable:** No | **Container:** No
- **Key method:** `update(obj)` to update the displayed object

### Rule

Visual separator line.

```python
from textual.widgets import Rule

yield Rule()                         # Horizontal
yield Rule(orientation="vertical")   # Vertical
```

- **Focusable:** No | **Container:** No
- **Reactive:** `orientation` (horizontal/vertical), `line_style` (solid/dashed/double/heavy/etc.)

### Markdown

Render markdown content.

```python
from textual.widgets import Markdown

yield Markdown("# Hello\n\nSome **markdown** content.")
```

- **Focusable:** No | **Container:** No
- **Events:** `Markdown.TableOfContentsUpdated`, `Markdown.TableOfContentsSelected`, `Markdown.LinkClicked`
- **Key method:** `update(markdown_str)` to change content

### MarkdownViewer

Markdown with optional table of contents and browser-like navigation.

```python
from textual.widgets import MarkdownViewer

yield MarkdownViewer("# Hello\n\nContent here.")
```

- **Focusable:** Yes | **Container:** No
- **Reactive:** `show_table_of_contents` (bool, default: `True`)

### Sparkline

Visual representation of numerical data as bars.

```python
from textual.widgets import Sparkline

yield Sparkline([1, 4, 2, 8, 5, 3])
```

- **Focusable:** No | **Container:** No
- **Reactive:** `data` (Sequence[float]), `summary_function` (Callable, default: `max`)

### LoadingIndicator

Animated dots shown during data loading.

```python
from textual.widgets import LoadingIndicator

yield LoadingIndicator()

# Or use the loading reactive on any widget:
widget.loading = True
```

- **Focusable:** No | **Container:** No

### Placeholder

Layout placeholder for prototyping. Click to cycle through variants (default/size/text).

```python
from textual.widgets import Placeholder

yield Placeholder("Sidebar")
```

- **Focusable:** No | **Container:** No
- **Reactive:** `variant` (default/size/text)

## App Chrome

### Header

Display app title and subtitle at top of app. Click to toggle tall/short.

```python
from textual.widgets import Header

yield Header()  # Shows App.title and App.sub_title
```

- **Focusable:** No | **Container:** No
- **Reactive:** `tall` (bool, default: `True`)

### Footer

Display available key bindings at bottom of app.

```python
from textual.widgets import Footer

yield Footer()
```

- **Focusable:** No | **Container:** No
- **Reactive:** `compact` (bool), `show_command_palette` (bool)
- Auto-displays bindings for the focused widget

## Input Widgets

### Button

Clickable button with semantic variants.

```python
from textual.widgets import Button

yield Button("Submit", variant="primary", id="submit")
yield Button("Delete", variant="error", disabled=True)
```

- **Focusable:** Yes | **Container:** No
- **Constructor:** `label` (str), `variant` (default/primary/success/warning/error), `disabled` (bool)
- **Events:** `Button.Pressed`
- **Reactive:** `label`, `variant`, `disabled`

### Input

Single-line text input with validation.

```python
from textual.widgets import Input
from textual.validation import Number

yield Input(placeholder="Enter name...", id="name")
yield Input(type="integer", max_length=5)
yield Input(validators=[Number(minimum=0, maximum=100)])
yield Input(password=True, placeholder="Password")
```

- **Focusable:** Yes | **Container:** No
- **Constructor:** `value`, `placeholder`, `type` (text/integer/number), `password` (bool), `restrict` (regex), `max_length`, `validators`, `validate_on` (changed/submitted/blur), `valid_empty` (bool)
- **Events:** `Input.Changed`, `Input.Submitted`
- **Reactive:** `value`, `placeholder`, `password`, `restrict`, `type`, `max_length`, `cursor_blink`

### MaskedInput

Text input with template mask for formatted input.

```python
from textual.widgets import MaskedInput

yield MaskedInput(template="(999) 999-9999")  # Phone
yield MaskedInput(template="9999-99-99")       # Date
yield MaskedInput(template="HH:HH:HH:HH:HH:HH")  # MAC address
```

- **Focusable:** Yes | **Container:** No
- **Template characters:** `A`/`a` = letter (required/optional), `N`/`n` = alphanumeric, `9`/`0` = digit, `H`/`h` = hex, `>` = uppercase, `<` = lowercase
- **Events:** `MaskedInput.Changed`, `MaskedInput.Submitted`

### TextArea

Multi-line text editor with optional syntax highlighting.

```python
from textual.widgets import TextArea

yield TextArea("Initial content", language="python", theme="monokai")
yield TextArea.code_editor("print('hello')", language="python")
```

- **Focusable:** Yes | **Container:** No
- **Reactive:** `language`, `theme`, `show_line_numbers`, `indent_width`, `soft_wrap`, `read_only`, `cursor_blink`
- **Events:** `TextArea.Changed`, `TextArea.SelectionChanged`
- **Key methods:**
  - Content: `text` property, `replace()`, `insert()`, `delete()`, `clear()`
  - Cursor: `cursor_location`, `move_cursor()`, `move_cursor_relative()`
  - Selection: `selection`, `select_line()`, `select_all()`
  - Undo: `undo()`, `redo()`, `history.checkpoint()`
- **Languages:** python, javascript, markdown, json, yaml, etc. (requires `textual[syntax]`)
- **Themes:** css, dracula, github_light, monokai, vscode_dark

### Checkbox

Boolean toggle control.

```python
from textual.widgets import Checkbox

yield Checkbox("Enable notifications", value=True)
```

- **Focusable:** Yes | **Container:** No
- **Constructor:** `label` (str), `value` (bool, default: `False`)
- **Events:** `Checkbox.Changed`
- **Reactive:** `value`

### RadioButton / RadioSet

Mutually exclusive selection.

```python
from textual.widgets import RadioButton, RadioSet

# With strings
yield RadioSet("Small", "Medium", "Large")

# With RadioButton objects
with RadioSet():
    yield RadioButton("Option A", value=True)
    yield RadioButton("Option B")
```

- **RadioSet:** Container for radio buttons with mutual exclusivity
- **Events:** `RadioSet.Changed` (with `pressed` and `index` attributes)
- **Reactive:** `RadioButton.value`

### Switch

On/off toggle control.

```python
from textual.widgets import Switch

yield Switch(value=False)
```

- **Focusable:** Yes | **Container:** No
- **Events:** `Switch.Changed`
- **Reactive:** `value` (bool)

### Select

Compact dropdown for selecting one option.

```python
from textual.widgets import Select

yield Select(
    [("Small", "s"), ("Medium", "m"), ("Large", "l")],
    prompt="Choose size",
)
yield Select.from_values(["Red", "Green", "Blue"])
```

- **Focusable:** Yes | **Container:** No
- **Constructor:** options (list of (display, value) tuples), `prompt`, `allow_blank`, `value`
- **Events:** `Select.Changed`
- **Reactive:** `value`, `expanded`
- **Key methods:** `set_options()`, `clear()`, `is_blank()`
- **Generic:** `Select[int]` for type-safe values. `Select.NULL` for blank state.

### SelectionList

Multi-select list with checkboxes.

```python
from textual.widgets import SelectionList

yield SelectionList[str](
    ("Python", "py", True),     # (label, value, selected)
    ("JavaScript", "js"),
    ("Rust", "rs"),
)
```

- **Focusable:** Yes | **Container:** No
- **Events:** `SelectionList.SelectedChanged`, `SelectionList.SelectionToggled`, `SelectionList.SelectionHighlighted`
- **Reactive:** `highlighted` (int | None)
- **Key properties:** `selected` (list of selected values)

### OptionList

Vertical list of Rich renderable options.

```python
from textual.widgets import OptionList
from textual.widgets.option_list import Option, Separator

yield OptionList(
    "Option 1",
    Option("Option 2", id="opt2"),
    Separator(),
    Option("Option 3", disabled=True),
)
```

- **Focusable:** Yes | **Container:** No
- **Events:** `OptionList.OptionHighlighted`, `OptionList.OptionSelected`
- **Reactive:** `highlighted` (int | None)

## Data Display Widgets

### DataTable

Interactive tabular data display with cursor, sorting, and selection.

```python
from textual.widgets import DataTable

class MyApp(App):
    def compose(self) -> ComposeResult:
        yield DataTable()

    def on_mount(self) -> None:
        table = self.query_one(DataTable)
        table.add_columns("Name", "Age", "City")
        table.add_rows([
            ["Alice", 30, "NYC"],
            ["Bob", 25, "LA"],
        ])
```

- **Focusable:** Yes | **Container:** No
- **Reactive:** `show_header`, `show_row_labels`, `zebra_stripes`, `cursor_type` (cell/row/column/none), `show_cursor`, `fixed_rows`, `fixed_columns`
- **Events:** `CellHighlighted`, `CellSelected`, `RowHighlighted`, `RowSelected`, `ColumnHighlighted`, `ColumnSelected`, `HeaderSelected`
- **Key methods:**
  - `add_columns(*labels)` / `add_column(label, key=None, width=None, default=None)`
  - `add_rows(rows)` / `add_row(*cells, key=None, label=None, height=None)`
  - `update_cell(row_key, column_key, value)` / `update_cell_at(coordinate, value)`
  - `remove_row(key)` / `remove_column(key)` / `clear(columns=False)`
  - `sort(*keys, reverse=False)` / `sort(key, key=lambda row: row[0])`
  - `coordinate_to_cell_key(coordinate)`
- **Keys:** Rows and columns can be identified by key (string) instead of index

### Tree

Hierarchical tree structure with expandable nodes.

```python
from textual.widgets import Tree

tree = Tree("Root")
node = tree.root.add("Branch 1")
node.add_leaf("Leaf A")
node.add_leaf("Leaf B")
tree.root.add("Branch 2").add_leaf("Leaf C")
yield tree
```

- **Focusable:** Yes | **Container:** No
- **Reactive:** `show_root`, `show_guides`, `guide_depth`
- **Events:** `Tree.NodeCollapsed`, `Tree.NodeExpanded`, `Tree.NodeHighlighted`, `Tree.NodeSelected`
- **TreeNode methods:** `add(label, data=None)`, `add_leaf(label, data=None)`, `remove()`, `toggle()`, `expand()`, `collapse()`

### DirectoryTree

Tree view for filesystem navigation.

```python
from textual.widgets import DirectoryTree

yield DirectoryTree("/path/to/dir")
```

- **Focusable:** Yes | **Container:** No
- **Events:** `DirectoryTree.FileSelected` (with `path` attribute)
- **Key method:** `filter_paths(paths)` override for custom filtering

### ListView / ListItem

Scrollable list of selectable items.

```python
from textual.widgets import ListView, ListItem

yield ListView(
    ListItem(Static("Item 1")),
    ListItem(Static("Item 2")),
    ListItem(Static("Item 3")),
)
```

- **Focusable:** Yes (ListView) | **Container:** Yes (ListView)
- **Events:** `ListView.Highlighted`, `ListView.Selected`
- **Reactive:** `index` (currently highlighted index)
- **Key methods:** `append(item)`, `clear()`, `insert(index, items)`, `pop(index)`, `remove_items(indices)`

## Container Widgets

### TabbedContent / TabPane

Tabs with associated content panels.

```python
from textual.widgets import TabbedContent, TabPane

# Simple string labels
with TabbedContent("Settings", "Logs", "About"):
    yield Static("Settings content")
    yield Static("Logs content")
    yield Static("About content")

# Or with TabPane for more control
with TabbedContent(initial="logs"):
    with TabPane("Settings", id="settings"):
        yield Input(placeholder="Name")
    with TabPane("Logs", id="logs"):
        yield RichLog()
```

- **Focusable:** Yes | **Container:** Yes
- **Events:** `TabbedContent.TabActivated`, `TabbedContent.Cleared`
- **Reactive:** `active` (str - ID of active tab)
- **Key methods:** `add_pane(pane)`, `remove_pane(pane_id)`, `clear_panes()`

### Tabs

Standalone row of selectable tabs (used internally by TabbedContent).

```python
from textual.widgets import Tabs, Tab

yield Tabs(
    Tab("First", id="first"),
    Tab("Second", id="second"),
)
```

- **Focusable:** Yes | **Container:** No
- **Events:** `Tabs.TabActivated`, `Tabs.Cleared`
- **Reactive:** `active` (str - active tab ID)
- **Key methods:** `add_tab(tab)`, `remove_tab(tab_id)`, `clear()`

### Collapsible

Expandable/collapsible container with title.

```python
from textual.widgets import Collapsible

with Collapsible(title="Advanced Settings", collapsed=True):
    yield Input(placeholder="API Key")
    yield Switch(value=False)
```

- **Focusable:** Yes | **Container:** Yes
- **Events:** `Collapsible.Toggled`
- **Reactive:** `collapsed` (bool), `title` (str)

### ContentSwitcher

Show one child at a time, switching between them by ID.

```python
from textual.widgets import ContentSwitcher

with ContentSwitcher(initial="page1"):
    yield Static("Page 1 content", id="page1")
    yield Static("Page 2 content", id="page2")

# Switch with:
self.query_one(ContentSwitcher).current = "page2"
```

- **Focusable:** No | **Container:** Yes
- **Reactive:** `current` (str | None - ID of visible child)

## Log Widgets

### Log

Append-only text log (text only, no Rich formatting).

```python
from textual.widgets import Log

log = self.query_one(Log)
log.write_line("Event occurred")
log.write_lines(["Line 1", "Line 2"])
log.clear()
```

- **Focusable:** Yes | **Container:** No
- **Reactive:** `max_lines` (int | None), `auto_scroll` (bool)

### RichLog

Scrollable log with Rich formatting support.

```python
from textual.widgets import RichLog
from rich.table import Table

log = self.query_one(RichLog)
log.write("Hello [bold red]World[/bold red]")
log.write(Table(...))  # Any Rich renderable
log.clear()
```

- **Focusable:** Yes | **Container:** No
- **Reactive:** `highlight` (bool), `markup` (bool), `max_lines` (int | None), `wrap` (bool), `min_width` (int, default: 78)
- **Key method:** `write(content)` accepts strings and Rich renderables

## Misc Widgets

### Link

Clickable link that opens URL in browser.

```python
from textual.widgets import Link

yield Link("Visit Textual", url="https://textual.textualize.io")
```

- **Focusable:** Yes | **Container:** No
- **Reactive:** `text`, `url`

### ProgressBar

Progress indicator with optional percentage and ETA.

```python
from textual.widgets import ProgressBar

bar = ProgressBar(total=100, show_eta=True)
yield bar

# Update progress:
bar.advance(10)
bar.update(progress=50)
bar.update(total=200)  # Indeterminate if total is None
```

- **Focusable:** No | **Container:** No
- **Reactive:** `progress` (float), `total` (float | None), `percentage` (float | None, read-only)
- **Constructor:** `total`, `show_percentage` (bool), `show_eta` (bool)

### Toast

Notification popup. Not used directly; created via `App.notify()` or `Widget.notify()`.

```python
self.notify("File saved!", title="Success", severity="information")
self.notify("Disk full!", severity="error", timeout=10)
```

- **Severities:** `information`, `warning`, `error`
