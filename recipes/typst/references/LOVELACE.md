# Lovelace (Pseudocode)

Lovelace formats pseudocode algorithms with customizable styling. Named after Ada Lovelace.

## Quick Start

```typst
#import "@preview/lovelace:0.3.0": *

#pseudocode-list[
  + *Input:* array $A$ of length $n$
  + *for* $i = 1$ to $n - 1$ *do*
    + *for* $j = 0$ to $n - i - 1$ *do*
      + *if* $A[j] > A[j+1]$ *then*
        + swap $A[j]$ and $A[j+1]$
  + *Output:* sorted array $A$
]
```

## With Line Numbers

```typst
#pseudocode-list(
  line-numbering: "1",
)[
  + Initialize $x = 0$
  + *while* $x < 10$ *do*
    + $x = x + 1$
  + *return* $x$
]
```

## With Indentation Guides

```typst
#pseudocode-list(
  indentation-guide-stroke: 1pt + gray,
)[
  + *function* Factorial($n$)
    + *if* $n <= 1$ *then*
      + *return* $1$
    + *else*
      + *return* $n times$ Factorial($n - 1$)
]
```

## With Title and Frame

```typst
#figure(
  kind: "algorithm",
  supplement: [Algorithm],
  pseudocode-list(
    booktabs: true,
    title: [Binary Search],
  )[
    + *Input:* sorted array $A$, target $x$
    + $"left" = 0$, $"right" = n - 1$
    + *while* $"left" <= "right"$ *do*
      + $"mid" = floor(("left" + "right") / 2)$
      + *if* $A["mid"] = x$ *then*
        + *return* $"mid"$
      + *else if* $A["mid"] < x$ *then*
        + $"left" = "mid" + 1$
      + *else*
        + $"right" = "mid" - 1$
    + *return* $-1$
  ],
  caption: [Binary search algorithm],
)
```

## Low-Level Control

```typst
#pseudocode(
  line-numbering: "1",
  [Initialize],
  indent(
    [Process step 1],
    [Process step 2],
  ),
  [Finalize],
)
```

## Line Labels for References

```typst
#pseudocode-list(
  line-numbering: "1",
)[
  + $x = 0$ #line-label(<init>)
  + *while* $x < 10$ *do*
    + $x = x + 1$ #line-label(<increment>)
]

See line @init for initialization and @increment for the loop body.
```

## Configuration Options

| Option | Description |
|--------|-------------|
| `line-numbering` | Numbering format ("1", "i", "I", etc.) |
| `indentation-guide-stroke` | Stroke for indent guides |
| `line-gap` | Gap between lines (default: 0.8em) |
| `indent-size` | Indentation width (default: 1em) |
| `booktabs` | Add frame decoration |
| `title` | Algorithm title |

## Resources

- [Typst Universe](https://typst.app/universe/package/lovelace)
