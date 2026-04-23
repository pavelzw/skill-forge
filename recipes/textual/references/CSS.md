# Textual CSS Reference

Complete reference for Textual CSS (TCSS) properties, selectors, and types.

## Selectors

### Basic Selectors

| Selector | Syntax | Example | Matches |
|----------|--------|---------|---------|
| Type | `TypeName` | `Button` | All Button widgets |
| ID | `#id` | `#sidebar` | Widget with `id="sidebar"` |
| Class | `.class` | `.error` | Widgets with `error` CSS class |
| Universal | `*` | `*` | All widgets |

### Combinators

| Combinator | Syntax | Example | Matches |
|-----------|--------|---------|---------|
| Descendant | `A B` | `Screen Button` | Buttons anywhere inside Screen |
| Child | `A > B` | `Horizontal > Button` | Direct child Buttons of Horizontal |
| Nesting | `&` | `& > .child` | Used inside nested rules |

### Pseudo-classes

| Pseudo-class | Matches |
|-------------|---------|
| `:hover` | Mouse is over widget |
| `:focus` | Widget has input focus |
| `:focus-within` | Widget or descendant has focus |
| `:disabled` | Widget is disabled |
| `:enabled` | Widget is enabled |
| `:dark` | App is in dark mode |
| `:light` | App is in light mode |
| `:even` / `:odd` | Even/odd children |
| `:first-child` / `:last-child` | First/last child of parent |
| `:blur` | Widget does not have focus |
| `:can-focus` | Widget is focusable |
| `:inline` | App is running inline |

### CSS Nesting

```css
Button {
    background: $surface;

    &:hover {
        background: $accent;
    }

    &.primary {
        background: $primary;
    }

    & > Static {
        color: $text;
    }
}
```

### Specificity

From lowest to highest: type < class < ID. Later rules win on equal specificity. Use `!important` to override.

## CSS Variables

```css
/* Theme variables (built-in) */
Screen {
    background: $surface;
    color: $text;
    border: solid $accent;
}

/* Custom variables */
$sidebar-width: 30;
#sidebar {
    width: $sidebar-width;
}
```

**Theme color variables:** `$primary`, `$secondary`, `$accent`, `$foreground`, `$background`, `$surface`, `$panel`, `$boost`, `$warning`, `$error`, `$success`

Each has shades: `$primary-lighten-1`, `$primary-lighten-2`, `$primary-lighten-3`, `$primary-darken-1`, `$primary-darken-2`, `$primary-darken-3`

**Text variables:** `$text`, `$text-muted`, `$text-disabled`

## CSS Types

### `<scalar>` - Length / Size

| Unit | Description | Example |
|------|-------------|---------|
| (number) | Cell units | `width: 30;` |
| `%` | Percentage of parent | `width: 50%;` |
| `fr` | Fraction of remaining space | `width: 1fr;` |
| `vw` | Viewport width % | `width: 50vw;` |
| `vh` | Viewport height % | `height: 50vh;` |
| `w` | Container width % | `width: 50w;` |
| `h` | Container height % | `height: 50h;` |
| `auto` | Automatic | `height: auto;` |

### `<color>` - Color Values

| Format | Example |
|--------|---------|
| Named | `red`, `blue`, `green`, `white`, `black` |
| Hex | `#FF0000`, `#F00` |
| Hex RGBA | `#FF000080` |
| RGB | `rgb(255, 0, 0)` |
| RGBA | `rgba(255, 0, 0, 0.5)` |
| HSL | `hsl(0, 100%, 50%)` |
| HSLA | `hsla(0, 100%, 50%, 0.5)` |
| Variable | `$accent`, `$primary` |
| Auto | `auto` (automatic contrast) |

### `<border>` - Border Styles

`ascii`, `blank`, `dashed`, `double`, `heavy`, `hidden`, `hkey`, `inner`, `none`, `outer`, `panel`, `round`, `solid`, `tall`, `thick`, `vkey`, `wide`

### `<text-style>` - Text Decorations

`none`, `bold`, `italic`, `reverse`, `strike`, `underline` (can combine: `bold italic`)

### Other Types

| Type | Values |
|------|--------|
| `<horizontal>` | `left`, `center`, `right` |
| `<vertical>` | `top`, `middle`, `bottom` |
| `<overflow>` | `auto`, `hidden`, `scroll` |
| `<position>` | `relative`, `absolute` |
| `<text-align>` | `left`, `center`, `right`, `justify` |
| `<integer>` | Whole numbers (e.g., `5`, `-2`) |
| `<number>` | Decimal numbers (e.g., `0.5`, `3.14`) |
| `<percentage>` | Number with `%` (e.g., `50%`) |
| `<name>` | Identifier (e.g., `my-layer`) |
| `<hatch>` | `cross`, `horizontal`, `left`, `right`, `vertical`, or character |
| `<keyline>` | `none`, `thin`, `heavy`, `double` |
| `<pointer>` | `default`, `pointer`, `text`, `crosshair`, etc. |

## Layout Properties

### layout

Set how children are arranged.

```css
/* CSS */                           /* Python */
layout: vertical;                   widget.styles.layout = "vertical"
layout: horizontal;                 widget.styles.layout = "horizontal"
layout: grid;                       widget.styles.layout = "grid"
```

### display

Show or hide a widget.

```css
display: block;                     widget.display = True
display: none;                      widget.display = False
```

### dock

Fix widget to an edge of its container.

```css
dock: top;                          widget.styles.dock = "top"
dock: bottom;                       widget.styles.dock = "bottom"
dock: left;                         widget.styles.dock = "left"
dock: right;                        widget.styles.dock = "right"
```

### align / content-align

Align children or content within a widget.

```css
align: center middle;               widget.styles.align = ("center", "middle")
content-align: right top;           widget.styles.content_align = ("right", "top")

/* Individual axes */
align-horizontal: center;
align-vertical: middle;
```

## Sizing Properties

### width / height

```css
width: 50;                          widget.styles.width = 50
width: 50%;                         widget.styles.width = "50%"
width: 1fr;                         widget.styles.width = "1fr"
width: auto;                        widget.styles.width = "auto"
height: 100vh;                      widget.styles.height = "100vh"
```

### min-width / max-width / min-height / max-height

```css
min-width: 20;                      widget.styles.min_width = 20
max-width: 80;                      widget.styles.max_width = 80
min-height: 5;                      widget.styles.min_height = 5
max-height: 30;                     widget.styles.max_height = 30
```

### box-sizing

```css
box-sizing: border-box;             /* Default: padding/border included in size */
box-sizing: content-box;            /* Padding/border added to size */
```

## Spacing Properties

### margin

Space outside the widget. Values: 1 (all), 2 (vertical horizontal), or 4 (top right bottom left).

```css
margin: 1;                          widget.styles.margin = (1, 1, 1, 1)
margin: 1 2;                        widget.styles.margin = (1, 2, 1, 2)
margin: 1 2 3 4;                    widget.styles.margin = (1, 2, 3, 4)
```

### padding

Space inside the widget around content. Same value syntax as margin.

```css
padding: 1 2;                       widget.styles.padding = (1, 2, 1, 2)
```

### offset

Move widget relative to its normal position.

```css
offset: 5 3;                        widget.styles.offset = (5, 3)
offset-x: 10;
offset-y: -5;
```

## Color Properties

### background / color

```css
background: $surface;               widget.styles.background = "blue"
background: red 50%;                /* With alpha */
color: $text;                       widget.styles.color = "white"
color: auto;                        /* Automatic contrast */
```

### tint / background-tint

Blend a color over the widget or its background.

```css
tint: red 20%;                      widget.styles.tint = "red 20%"
background-tint: blue 10%;
```

### opacity / text-opacity

```css
opacity: 0.5;                       widget.styles.opacity = 0.5
opacity: 50%;
text-opacity: 75%;                  widget.styles.text_opacity = "75%"
```

### visibility

Hide widget while reserving its space.

```css
visibility: visible;                widget.visible = True
visibility: hidden;                 widget.visible = False
```

## Border Properties

### border / outline

Border draws outside content; outline draws over content.

```css
border: solid $accent;              widget.styles.border = ("solid", "$accent")
border: heavy red;
border-top: dashed blue;
border-left: solid green;

outline: round white;               widget.styles.outline = ("round", "white")
```

### Border Title

```css
border-title-align: center;         /* left (default), center, right */
border-title-color: $accent;
border-title-background: $surface;
border-title-style: bold;

border-subtitle-align: right;
border-subtitle-color: $text-muted;
```

Set title text in Python:

```python
widget.border_title = "My Title"
widget.border_subtitle = "Subtitle"
```

## Text Properties

### text-align

```css
text-align: left;                   /* Default */
text-align: center;
text-align: right;
text-align: justify;
```

### text-style

```css
text-style: bold;
text-style: italic underline;
text-style: none;
```

### text-wrap / text-overflow

```css
text-wrap: wrap;                    /* Default: wrap text */
text-wrap: nowrap;                  /* No wrapping */

text-overflow: fold;                /* Default: wrap to next line */
text-overflow: ellipsis;            /* Truncate with ... */
text-overflow: clip;                /* Hard clip */
```

## Grid Properties

```css
/* Define grid */
layout: grid;
grid-size: 3 2;                     /* 3 columns, 2 rows */
grid-columns: 1fr 2fr 1fr;          /* Column widths */
grid-rows: auto 1fr;                /* Row heights */
grid-gutter: 1 2;                   /* Vertical horizontal spacing */

/* Cell spanning */
column-span: 2;                     /* Span 2 columns */
row-span: 3;                        /* Span 3 rows */
```

### Grid Example

```css
#container {
    layout: grid;
    grid-size: 3;
    grid-gutter: 1;
    grid-columns: 1fr 2fr 1fr;
}

#header {
    column-span: 3;
}

#sidebar {
    row-span: 2;
}
```

## Layer Properties

Control z-ordering with layers.

```css
/* Define layers on container (last = topmost) */
Screen {
    layers: below default above;
}

/* Assign widgets to layers */
#dialog {
    layer: above;
}

#background {
    layer: below;
}
```

## Scrollbar Properties

### overflow

```css
overflow: auto auto;                /* Default: show when needed */
overflow-x: hidden;                 /* Never show horizontal scrollbar */
overflow-y: scroll;                 /* Always allow vertical scrolling */
```

### scrollbar-gutter

```css
scrollbar-gutter: auto;             /* Default: no reserved space */
scrollbar-gutter: stable;           /* Reserve space for scrollbar */
```

### scrollbar-size

```css
scrollbar-size: 2 1;                /* horizontal vertical */
scrollbar-size-horizontal: 2;
scrollbar-size-vertical: 1;
```

### scrollbar-color / scrollbar-background

```css
scrollbar-color: $accent;
scrollbar-background: $surface;
scrollbar-color-hover: $accent-lighten-1;
scrollbar-background-hover: $surface;
scrollbar-color-active: $accent-lighten-2;
scrollbar-background-active: $surface;
scrollbar-corner-color: $surface;
```

## Link Properties

Style Textual action links in markup.

```css
link-color: $accent;
link-background: transparent;
link-style: underline;

link-color-hover: white;
link-background-hover: $accent;
link-style-hover: bold underline;
```

## Special Properties

### hatch

Fill background with repeating pattern.

```css
hatch: cross red;
hatch: "T" blue 80%;
hatch: right green 50%;
```

### keyline

Draw lines around child widgets in a grid.

```css
keyline: thin green;
keyline: heavy $accent;
```

### pointer

Change mouse cursor shape (requires Kitty protocol).

```css
pointer: pointer;
pointer: grab;
```

### position

```css
position: relative;                  /* Default: offset from normal position */
position: absolute;                  /* Offset from container origin */
```
