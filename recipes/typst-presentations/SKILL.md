---
name: typst-presentations
description: >-
  Create visually appealing presentations with Typst. Combines Typst typesetting
  knowledge with presentation design principles. Use when creating slide decks,
  talks, or presentations with Typst, Touying, or Polylux.
license: BSD-3-Clause
---

# Typst Presentations

Create visually appealing presentations with Typst by combining typesetting skills with presentation design principles.

## Prerequisites

This skill builds on two foundational skills:
- **typst** - For Typst syntax, packages (Touying, Polylux), and CLI usage
- **presentation-design** - For general presentation design principles (slide layout, typography, visual hierarchy, storytelling)

Refer to both skills before starting. Use the typst skill's Touying or Polylux references for slide-specific syntax.

If you don't have a Typst presentation template, set up a 16:9 page format.

## Visual Feedback Loop

**This is the most critical part of creating presentations with Typst.**

You cannot judge a slide from its source code alone. After editing each slide, you MUST compile it to a PNG image and visually inspect it. This is a non-negotiable step in the workflow.

### Compile and inspect a single slide

After editing slide N, run:

```bash
pixi run typst compile <main.typ> <output.png> --format png --pages <N>
```

Then read the output PNG to visually inspect the result.

### What to check on every slide

- **Text overflow** - Does text run off the slide or overlap with other elements?
- **Alignment** - Are elements properly aligned and balanced on the slide?
- **Font sizes** - Is text large enough to read from the back of a room (minimum ~20pt for body text)?
- **White space** - Does the slide feel cramped or cluttered?
- **Visual hierarchy** - Is it immediately clear what the most important element is?
- **Consistency** - Does this slide match the style of previous slides?

### Fix and re-render

If anything looks wrong, fix the issue in the source, recompile, and inspect again. Repeat until the slide is clean. Common fixes:

- **Text overflow** - Reduce content, decrease font size, or split into multiple slides
- **Cramped layout** - Increase margins, reduce content, use whitespace deliberately
- **Poor alignment** - Use `align`, `grid`, or `stack` for precise positioning
- **Inconsistent styling** - Extract repeated styles into `set` rules or variables

**Do not move on to the next slide until the current slide passes visual inspection.**
