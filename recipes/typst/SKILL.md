---
name: typst
description: Create beautifully typeset documents with Typst, a modern markup-based typesetting system. Handles academic papers, scientific documents, mathematical equations, tables, and document layouts. Use when creating PDFs, typesetting documents, or when the user mentions typst, document formatting, or mathematical typesetting.
license: Apache-2.0
---

# Typst Skill

Typst is a modern markup-based typesetting system designed for scientific documents. It serves as an alternative to LaTeX with faster compilation (milliseconds, not seconds) and simpler syntax.

## When to Use This Skill

Use Typst when:
- Creating academic papers, theses, or scientific documents
- Writing documents with mathematical equations
- Need faster compilation than LaTeX
- Want a simpler markup syntax than LaTeX
- Creating documents with complex tables and layouts
- Building reusable document templates

## Three Syntactical Modes

Typst operates in three modes:

| Mode | Purpose | Switching |
|------|---------|-----------|
| **Markup** | Document structure (default) | Default mode |
| **Math** | Mathematical formulas | `$...$` for inline, `$ ... $` with spaces for display |
| **Code** | Scripting features | Prefix with `#` |

## Escaping Special Characters

CRITICAL: When writing literal text in markup mode, you MUST escape special characters with a backslash (`\`). An unescaped `#` is the most common source of compilation errors — it tells Typst to enter code mode, so any literal `#` in text (e.g. "C#", "issue #42") must be written as `\#`.

### Markup Mode Special Characters

| Character | Meaning | Escape | Example |
|-----------|---------|--------|---------|
| `\` | Escape character / line break | `\\` | `path\\to\\file` |
| `#` | Code expression | `\#` | `Tweet at us \#ad`, `C\#`, `\$1.50 per \#item` |
| `*` | Bold text | `\*` | `5 \* 3 = 15` |
| `_` | Italic text | `\_` | `file\_name\_here` |
| `` ` `` | Raw text / code | `` \` `` | |
| `$` | Math mode | `\$` | `costs \$1.50` |
| `<` | Label | `\<` | `a \< b in text` |
| `@` | Reference / citation | `\@` | `email\@example.com` |
| `~` | Non-breaking space | `\~` | |
| `'` | Smart quote | `\'` | |
| `"` | Smart quote | `\"` | |

Note: `=`, `-`, `+`, `/` only have special meaning at the start of a line (headings, lists, term lists). They do not need escaping mid-sentence.

### Math Mode Special Characters

Inside `$...$`, these characters have special meaning and need `\` to use literally:

| Character | Meaning | Escape |
|-----------|---------|--------|
| `_` | Subscript | `\_` |
| `^` | Superscript | `\^` |
| `/` | Fraction | `\/` |
| `&` | Alignment | `\&` |
| `#` | Code/variable access | `\#` |

### Symbols and Special Characters

Typst has built-in named symbols for common characters. Prefer these over `\u{...}` unicode escapes:
```typst
// Dashes
En dash: --    Em dash: ---

// Emojis via #emoji namespace
#emoji.avocado #emoji.face.happy

// Named symbols via #sym namespace
#sym.arrow.r  #sym.checkmark  #sym.euro

// Direct UTF-8 characters also work
Ü, ñ, ö — just type them directly
```

### Common Mistakes

```typst
// WRONG — # starts code mode, causes compilation error:
I love C# programming

// CORRECT:
I love C\# programming

// WRONG — $ enters math mode:
The price is $10

// CORRECT:
The price is \$10

// WRONG — @ tries to find a reference:
Contact me @twitter

// CORRECT:
Contact me \@twitter
```

## Basic Markup Syntax

```typst
= First-Level Heading
== Second-Level Heading

Regular paragraph text.
A blank line creates a new paragraph.

_italics_ and *bold* text.

- Unordered list item
- Another item

+ Numbered list item
+ Another numbered item

/ Term: Definition for term lists

`inline code` and code blocks:

```python
def hello():
    print("Hello!")
`` `

Link: https://typst.app or #link("https://typst.app")[Typst App]

#image("diagram.png", width: 70%)
```

## Mathematical Typesetting

```typst
Inline math: $x^2 + y^2 = z^2$

Display math (note the spaces):
$ integral_0^infinity e^(-x^2) dif x = sqrt(pi) / 2 $

Greek letters: $alpha$, $beta$, $gamma$, $omega$

Fractions: $a/b$ or $(a + b) / c$

Subscripts and superscripts: $x_1$, $x^2$, $x_1^2$

Vectors and matrices:
$ vec(a, b, c) quad mat(1, 2; 3, 4) $

Aligned equations:
$ a &= b + c \
  &= d + e $
```

## Functions and Content

```typst
// Image with caption
#figure(
  image("chart.png", width: 80%),
  caption: [Monthly revenue growth],
) <fig:chart>

See @fig:chart for details.

// Bibliography
#bibliography("refs.bib")
Citation: @einstein1905

// Tables
#table(
  columns: 3,
  [Header 1], [Header 2], [Header 3],
  [Row 1 A], [Row 1 B], [Row 1 C],
  [Row 2 A], [Row 2 B], [Row 2 C],
)
```

## Set Rules (Global Styling)

Apply styles throughout your document:

```typst
// Font and text settings
#set text(font: "Libertinus Serif", size: 11pt)
#set par(justify: true, leading: 0.65em)

// Page setup
#set page(
  paper: "us-letter",
  margin: (x: 2.5cm, y: 3cm),
  numbering: "1",
)

// Heading numbering
#set heading(numbering: "1.1")

// Document metadata
#set document(
  title: "My Document",
  author: "Author Name",
)
```

## Show Rules (Element Transformation)

Customize how elements are displayed:

```typst
// Style all level-1 headings
#show heading.where(level: 1): it => [
  #set text(blue)
  #block(smallcaps(it.body))
]

// Style specific text
#show "Typst": name => box[
  #text(blue)[#name]
]

// Combine show with set
#show heading: set text(navy)
```

## Variables and Functions

```typst
// Define variables
#let title = "My Document"
#let version = 2

// Define functions
#let highlight(body, color: yellow) = {
  box(fill: color, inset: 3pt, body)
}

// Use them
#highlight[Important text]
#highlight(color: red)[Warning!]
```

## Document Templates

Create reusable templates:

```typst
// template.typ
#let article(
  title: none,
  authors: (),
  abstract: none,
  body
) = {
  set document(title: title)
  set page(margin: 2cm)
  set text(font: "Libertinus Serif", size: 11pt)

  // Title
  align(center)[
    #text(17pt, weight: "bold")[#title]
    #v(1em)
    #authors.join(", ")
  ]

  // Abstract
  if abstract != none [
    #heading(outlined: false)[Abstract]
    #abstract
  ]

  body
}

// main.typ
#import "template.typ": article
#show: article.with(
  title: "My Paper",
  authors: ("Alice", "Bob"),
  abstract: [This paper explores...],
)

= Introduction
Content here...
```

## Tables

```typst
// Basic table with styling
#table(
  columns: (auto, 1fr, auto),
  align: (right, left, center),
  fill: (_, y) => if y == 0 { gray.lighten(50%) },
  stroke: 0.5pt,

  table.header[ID][Name][Score],
  [1], [Alice], [95],
  [2], [Bob], [87],
  [3], [Carol], [92],
)

// Cell spanning
#table(
  columns: 3,
  table.cell(colspan: 2)[Merged],  [Single],
  [A], [B], [C],
)
```

## Page Layout

```typst
// Two-column layout
#set page(columns: 2)

// Headers and footers
#set page(
  header: [Document Title #h(1fr) Draft],
  footer: context [
    #h(1fr)
    Page #counter(page).display()
  ],
)

// Page breaks
#pagebreak()

// Column breaks
#colbreak()
```

## Popular Packages

Import packages from Typst Universe with `#import "@preview/package:version"`.

### Presentations

```typst
#import "@preview/touying:0.6.1": *   // Full-featured slides with themes
#import "@preview/polylux:0.4.0": *   // Lightweight beamer-style slides
```

### Drawing and Visualization

```typst
#import "@preview/cetz:0.4.2"         // TikZ-inspired drawing
#import "@preview/lilaq:0.5.0" as lq  // Scientific plots and charts
```

### Code and Algorithms

```typst
#import "@preview/codly:1.3.0": *     // Enhanced code blocks
#import "@preview/lovelace:0.3.0": *  // Pseudocode formatting
```

### Document Utilities

```typst
#import "@preview/drafting:0.2.2": *  // Margin notes and comments
```

See individual package guides for detailed usage:
- [Touying](references/TOUYING.md) - Presentations with themes
- [Polylux](references/POLYLUX.md) - Lightweight slides
- [CeTZ](references/CETZ.md) - Drawing library
- [Lilaq](references/LILAQ.md) - Scientific plots
- [Codly](references/CODLY.md) - Code blocks
- [Lovelace](references/LOVELACE.md) - Pseudocode
- [Drafting](references/DRAFTING.md) - Margin notes

## CLI Usage

```bash
# Compile to PDF
typst compile document.typ

# Watch for changes
typst watch document.typ

# Specify output
typst compile document.typ output.pdf

# Add font paths
typst compile --font-path ./fonts document.typ
```

## Quick Reference: LaTeX to Typst

| LaTeX | Typst |
|-------|-------|
| `\section{Title}` | `= Title` |
| `\textbf{bold}` | `*bold*` |
| `\textit{italic}` | `_italic_` |
| `\begin{itemize}` | `- item` |
| `\begin{enumerate}` | `+ item` |
| `$x^2$` | `$x^2$` |
| `\frac{a}{b}` | `$a/b$` |
| `\usepackage{...}` | `#import "@preview/..."` |
| `\documentclass` | Template with show rule |

## Additional Resources

- [Syntax Reference](references/SYNTAX.md)
- [Scripting Guide](references/SCRIPTING.md)
- [Styling Guide](references/STYLING.md)
- [Tables Guide](references/TABLES.md)
- [Official Documentation](https://typst.app/docs)
