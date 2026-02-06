# Drafting (Margin Notes)

Drafting provides margin comments and notes for document review.

## Setup

```typst
#import "@preview/drafting:0.2.2": *

#set page(margin: (left: 2cm, right: 4cm))
#set-page-properties()
```

## Margin Notes

```typst
// Basic margin note
#margin-note[This is a comment in the margin.]

// On specific side
#margin-note(side: left)[Left margin note]
#margin-note(side: right)[Right margin note]

// With styling
#margin-note(
  stroke: red,
  fill: yellow.lighten(80%),
)[Important note!]
```

## Inline Notes

```typst
#inline-note[This note appears inline with text flow.]
```

## Highlighting with Notes

```typst
#note-highlight[highlighted phrase][
  Comment about this phrase
]
```

## Reviewer-Specific Notes

```typst
#let alice-note = margin-note.with(
  stroke: blue,
  fill: blue.lighten(90%),
)

#let bob-note = margin-note.with(
  stroke: green,
  fill: green.lighten(90%),
)

#alice-note[Alice's comment]
#bob-note[Bob's comment]
```

## Hide Notes for Print

```typst
#set-margin-note-defaults(hidden: true)
```

## Key Features

- Automatic collision avoidance for overlapping notes
- Customizable stroke, fill, and positioning
- Rule grid for precise placement
- Absolute positioning anywhere on page

## Resources

- [Typst Universe](https://typst.app/universe/package/drafting)
