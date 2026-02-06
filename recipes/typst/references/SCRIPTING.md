# Typst Scripting Guide

Complete guide to Typst's scripting language for automating documents and creating sophisticated styles.

## Expressions

Code is embedded in markup using the hash (`#`) prefix. An expression is introduced with `#` and normal markup parsing resumes after the expression is finished.

```typst
#let x = 5
The value is #x.

// Complex expression needs parentheses or block
The sum is #(x + 10).

// Code block for multiple statements
#{
  let a = 1
  let b = 2
  a + b
}
```

## Data Types

### Primitives

```typst
#let nothing = none          // None/null
#let yes = true              // Boolean
#let no = false
#let count = 42              // Integer
#let pi = 3.14159            // Float
#let greeting = "Hello"      // String
```

### Lengths and Sizes

```typst
#let width = 100pt           // Points
#let height = 2cm            // Centimeters
#let spacing = 1.5em         // Relative to font
#let half = 50%              // Percentage
#let portion = 1fr           // Fractional
#let angle = 45deg           // Angle
```

### Collections

```typst
// Array
#let items = (1, 2, 3)
#let first = items.at(0)     // Access by index
#let len = items.len()       // Length
#items.push(4)               // Add item (mutating)

// Dictionary
#let person = (
  name: "Alice",
  age: 30,
  active: true,
)
#let name = person.name      // Access by key
#let name = person.at("name") // Alternative access
```

### Content

```typst
// Content is a first-class value
#let warning = [*Warning:* Be careful!]
#let header = [= My Title]

// Use content in functions
#let box-it(body) = box(stroke: black, inset: 5pt, body)
#box-it[Hello]
```

## Bindings

### Let Bindings

```typst
// Simple binding
#let x = 5

// Binding with content
#let title = [My Document]

// Destructuring arrays
#let (a, b, c) = (1, 2, 3)
#let (first, ..rest) = (1, 2, 3, 4)  // rest = (2, 3, 4)

// Destructuring dictionaries
#let (name: n, age: a) = (name: "Alice", age: 30)
#let (name, ..rest) = (name: "Alice", age: 30, city: "NYC")
```

### Function Definitions

```typst
// Basic function
#let greet(name) = [Hello, #name!]
#greet("World")

// With default parameters
#let greet(name, excited: false) = {
  if excited [Hello, #name!!!]
  else [Hello, #name.]
}
#greet("World")
#greet("World", excited: true)

// With content parameter
#let highlight(color: yellow, body) = {
  box(fill: color, inset: 3pt, body)
}
#highlight[Important]
#highlight(color: red)[Critical]

// Variadic arguments
#let sum(..nums) = {
  nums.pos().fold(0, (a, b) => a + b)
}
#sum(1, 2, 3, 4)  // 10
```

## Control Flow

### Conditionals

```typst
#let x = 5

#if x > 0 [
  Positive
] else if x < 0 [
  Negative
] else [
  Zero
]

// In code blocks
#{
  if x > 10 {
    "Large"
  } else {
    "Small"
  }
}

// Ternary-style
#let result = if x > 0 { "pos" } else { "neg" }
```

### For Loops

```typst
// Iterate over array
#for item in (1, 2, 3) [
  - Item: #item
]

// With index
#for (i, item) in (1, 2, 3).enumerate() [
  #(i + 1). #item
]

// Iterate over dictionary
#for (key, value) in (a: 1, b: 2) [
  #key = #value \
]

// Iterate over string
#for char in "abc" [
  Character: #char \
]

// Iterate over range
#for i in range(5) [
  #i \
]
```

### While Loops

```typst
#{
  let i = 0
  while i < 5 {
    [#i ]
    i += 1
  }
}
```

### Loop Control

```typst
// Break
#for i in range(10) {
  if i >= 5 { break }
  [#i ]
}

// Continue
#for i in range(5) {
  if calc.rem(i, 2) == 0 { continue }
  [#i ]  // Only odd numbers
}
```

## Operators

### Arithmetic

| Operator | Description |
|----------|-------------|
| `+` | Addition |
| `-` | Subtraction |
| `*` | Multiplication |
| `/` | Division |

```typst
#let x = 10 + 5      // 15
#let y = 10 - 3      // 7
#let z = 4 * 3       // 12
#let w = 15 / 4      // 3.75
```

### Comparison

| Operator | Description |
|----------|-------------|
| `==` | Equal |
| `!=` | Not equal |
| `<` | Less than |
| `>` | Greater than |
| `<=` | Less than or equal |
| `>=` | Greater than or equal |

### Logical

| Operator | Description |
|----------|-------------|
| `and` | Logical AND |
| `or` | Logical OR |
| `not` | Logical NOT |

```typst
#let valid = x > 0 and x < 100
#let invalid = not valid
#let optional = x == none or x == 0
```

### Assignment

```typst
#let x = 5
#(x = 10)        // Reassignment
#(x += 5)        // Add and assign
#(x -= 3)        // Subtract and assign
#(x *= 2)        // Multiply and assign
#(x /= 4)        // Divide and assign
```

### String/Array

```typst
#let s = "Hello" + " " + "World"  // Concatenation
#let a = (1, 2) + (3, 4)          // Array join
#"Hello".contains("ell")          // true
#(1, 2, 3).contains(2)            // true
```

## Methods

### String Methods

```typst
#let s = "Hello World"
#s.len()                    // 11
#s.contains("World")        // true
#s.starts-with("Hello")     // true
#s.ends-with("World")       // true
#s.find("o")                // 4
#s.replace("World", "Typst") // "Hello Typst"
#s.split(" ")               // ("Hello", "World")
#s.trim()                   // Remove whitespace
#upper(s)                   // "HELLO WORLD"
#lower(s)                   // "hello world"
```

### Array Methods

```typst
#let a = (1, 2, 3, 4, 5)
#a.len()                    // 5
#a.at(0)                    // 1
#a.first()                  // 1
#a.last()                   // 5
#a.slice(1, 3)              // (2, 3)
#a.contains(3)              // true
#a.position(x => x == 3)    // 2
#a.filter(x => x > 2)       // (3, 4, 5)
#a.map(x => x * 2)          // (2, 4, 6, 8, 10)
#a.fold(0, (acc, x) => acc + x)  // 15
#a.join(", ")               // "1, 2, 3, 4, 5"
#a.sorted()                 // Sorted copy
#a.rev()                    // Reversed copy
#a.enumerate()              // ((0, 1), (1, 2), ...)
```

### Dictionary Methods

```typst
#let d = (a: 1, b: 2, c: 3)
#d.len()                    // 3
#d.at("a")                  // 1
#d.at("x", default: 0)      // 0
#d.keys()                   // ("a", "b", "c")
#d.values()                 // (1, 2, 3)
#d.pairs()                  // (("a", 1), ("b", 2), ("c", 3))
```

## Modules

### Importing Files

```typst
// Import entire file
#import "utils.typ"

// Import specific items
#import "utils.typ": greet, format-date

// Import with renaming
#import "utils.typ": greet as say-hello

// Import all items
#import "utils.typ": *
```

### Including Content

```typst
// Include renders content directly
#include "chapter1.typ"
```

### Packages

```typst
// Import from Typst Universe
#import "@preview/cetz:0.4.1"
#import "@preview/tablex:0.0.8": tablex, rowspanx

// Use package functions
#cetz.canvas({
  // Drawing code
})
```

## Error Handling

```typst
// Assertions
#assert(x > 0, message: "x must be positive")

// Panic (stops compilation)
#panic("Something went wrong")

// Optional access
#let value = dict.at("key", default: none)
#if value != none [
  Value: #value
]
```

## Type Checking

```typst
#type(5)              // integer
#type(3.14)           // float
#type("hello")        // string
#type((1, 2))         // array
#type((a: 1))         // dictionary
#type([text])         // content
#type(none)           // none

// Type checking in code
#if type(x) == int [
  x is an integer
]
```

## Calc Module

```typst
#calc.abs(-5)         // 5
#calc.pow(2, 3)       // 8
#calc.sqrt(16)        // 4
#calc.sin(calc.pi)    // ~0
#calc.cos(0)          // 1
#calc.log(100, base: 10)  // 2
#calc.min(1, 2, 3)    // 1
#calc.max(1, 2, 3)    // 3
#calc.rem(7, 3)       // 1 (remainder)
#calc.floor(3.7)      // 3
#calc.ceil(3.2)       // 4
#calc.round(3.5)      // 4
```
