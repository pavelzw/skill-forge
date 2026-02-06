# Typst Syntax Reference

Complete reference for Typst's markup, math, and code syntax.

## Markup Mode

Markup mode is the default mode for writing document content.

### Text and Paragraphs

```typst
Regular text flows naturally.

A blank line starts a new paragraph.
Line breaks within text are treated as spaces.

Force a line break with \\
or using #linebreak()
```

### Headings

```typst
= Level 1 Heading
== Level 2 Heading
=== Level 3 Heading
==== Level 4 Heading
```

### Emphasis and Formatting

| Syntax | Result | Notes |
|--------|--------|-------|
| `_italic_` | _italic_ | Underscores |
| `*bold*` | **bold** | Asterisks |
| `_*bold italic*_` | _**bold italic**_ | Combine them |
| `` `code` `` | `code` | Backticks |
| `~` | Non-breaking space | Keeps words together |

### Lists

```typst
// Unordered list
- First item
- Second item
  - Nested item (2 spaces indent)
  - Another nested

// Numbered list
+ First item
+ Second item
  + Nested numbered

// Term list (definitions)
/ Term: Definition goes here
/ Another term: Its definition
```

### Links and References

```typst
// Auto-detected URL
https://typst.app

// Explicit link with text
#link("https://typst.app")[Visit Typst]

// Labels and references
= Introduction <intro>

See @intro for details.

// Reference with supplement
@intro[Chapter]
```

### Raw Text and Code

```typst
// Inline code
`let x = 5`

// Code block with syntax highlighting
```rust
fn main() {
    println!("Hello!");
}
`` `

// Raw block (no highlighting)
```
Plain text here
`` `
```

### Escape Sequences

| Sequence | Result |
|----------|--------|
| `\\` | Backslash |
| `\#` | Hash |
| `\*` | Asterisk |
| `\_` | Underscore |
| `\`` | Backtick |
| `\$` | Dollar sign |
| `\<` | Less than |
| `\>` | Greater than |
| `\@` | At sign |
| `\u{1F600}` | Unicode character |

### Comments

```typst
// Single-line comment

/* Multi-line
   comment */
```

## Math Mode

Enter math mode with dollar signs.

### Inline vs Display

```typst
// Inline math (no spaces inside)
The equation $x^2 + y^2 = z^2$ is famous.

// Display math (spaces inside)
$ x^2 + y^2 = z^2 $

// Multi-line display
$ a &= b + c \
  &= d + e $
```

### Subscripts and Superscripts

```typst
$x^2$           // Superscript
$x_1$           // Subscript
$x_1^2$         // Both
$x^(a+b)$       // Grouped superscript
$x_(i,j)$       // Grouped subscript
```

### Fractions

```typst
$a/b$           // Simple fraction
$(a+b)/(c+d)$   // Grouped
$1/2 + 1/3$     // Multiple fractions
```

### Greek Letters

| Letter | Syntax | Letter | Syntax |
|--------|--------|--------|--------|
| α | `alpha` | ν | `nu` |
| β | `beta` | ξ | `xi` |
| γ | `gamma` | π | `pi` |
| δ | `delta` | ρ | `rho` |
| ε | `epsilon` | σ | `sigma` |
| ζ | `zeta` | τ | `tau` |
| η | `eta` | υ | `upsilon` |
| θ | `theta` | φ | `phi` |
| ι | `iota` | χ | `chi` |
| κ | `kappa` | ψ | `psi` |
| λ | `lambda` | ω | `omega` |
| μ | `mu` | | |

Capital letters: `Alpha`, `Beta`, `Gamma`, etc.

### Common Operators

```typst
$+$, $-$, $times$, $div$
$<=$ or $lt.eq$
$>=$ or $gt.eq$
$!=$ or $eq.not$
$approx$, $equiv$, $prop$
$in$, $subset$, $supset$
$and$, $or$, $not$
```

### Functions

```typst
$sin(x)$, $cos(x)$, $tan(x)$
$log(x)$, $ln(x)$, $exp(x)$
$lim_(x->0)$
$max$, $min$, $sup$, $inf$
```

### Integrals and Sums

```typst
$integral$
$integral_0^infinity f(x) dif x$

$sum$
$sum_(i=0)^n i^2$

$product_(i=1)^n$
```

### Matrices and Vectors

```typst
// Column vector
$vec(a, b, c)$

// Row vector
$vec(a, b, c)^T$

// Matrix
$mat(
  1, 2, 3;
  4, 5, 6;
  7, 8, 9;
)$

// Matrix with delimiters
$mat(delim: "[",
  a, b;
  c, d;
)$
```

### Brackets and Delimiters

```typst
$(a)$, $[a]$, ${a}$
$|a|$               // Absolute value
$||a||$             // Norm
$floor(a)$          // Floor
$ceil(a)$           // Ceiling
$lr(angle.l a angle.r)$  // Angle brackets
```

### Text in Math

```typst
$x "for all" x in RR$
$"let" x = 5$
```

### Accents

```typst
$accent(x, -)$      // Bar: x̄
$accent(x, hat)$    // Hat: x̂
$accent(x, tilde)$  // Tilde: x̃
$accent(x, dot)$    // Dot: ẋ
$accent(x, dot.double)$  // Double dot
$arrow(x)$          // Arrow over x
$vec(x)$            // Vector notation
```

## Code Mode

Enter code mode with `#` prefix.

### Basic Expressions

```typst
#let x = 5
#let name = "Alice"
#let items = (1, 2, 3)
#let dict = (key: "value", num: 42)

The value is #x.
Hello, #name!
```

### Function Calls

```typst
// Positional arguments
#text(blue)[Blue text]

// Named arguments
#rect(width: 100pt, height: 50pt, fill: blue)

// Mixed
#image("photo.jpg", width: 80%)

// Content block argument
#block[
  This is block content.
]

// Trailing content
#emph[emphasized text]
```

### Control Flow

```typst
// Conditionals
#if x > 0 [
  Positive
] else if x < 0 [
  Negative
] else [
  Zero
]

// For loops
#for item in items [
  Item: #item \
]

#for (key, value) in dict [
  #key: #value \
]

// While loops
#let i = 0
#while i < 3 [
  Count: #i \
  #(i = i + 1)
]
```

### Blocks

```typst
// Code block (returns last expression)
#{
  let x = 1
  let y = 2
  x + y
}

// Content block
#[
  This is content that can span
  multiple lines.
]
```

## Identifiers

Valid identifier characters:
- Start with: letter, underscore
- Continue with: letter, number, underscore, hyphen

```typst
#let my-variable = 1     // Kebab-case (recommended)
#let my_variable = 1     // Snake-case
#let myVariable = 1      // Camel-case
```

## Units and Lengths

| Unit | Description |
|------|-------------|
| `pt` | Points (1/72 inch) |
| `mm` | Millimeters |
| `cm` | Centimeters |
| `in` | Inches |
| `em` | Relative to font size |
| `%` | Percentage of container |
| `fr` | Fractional unit |

```typst
#rect(width: 2cm, height: 1in)
#h(1em)           // Horizontal space
#v(12pt)          // Vertical space
```

## Colors

```typst
// Named colors
#text(fill: red)[Red]
#text(fill: blue)[Blue]

// RGB
#text(fill: rgb("#ff0000"))[Red]
#text(fill: rgb(255, 0, 0))[Red]
#text(fill: rgb("ff0000"))[Red]

// With alpha
#text(fill: rgb(255, 0, 0, 50%))[Semi-transparent]

// Color operations
#let light-blue = blue.lighten(50%)
#let dark-blue = blue.darken(30%)
```
