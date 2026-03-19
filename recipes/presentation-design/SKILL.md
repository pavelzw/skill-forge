---
name: presentation-design
description: >-
  Slide design and storytelling guidelines for creating presentations.
  ALWAYS load this skill when the user wants to create, review, or improve
  presentation slides — regardless of tool (Typst/Polylux, Beamer, PowerPoint,
  Google Slides, reveal.js, etc.).
license: BSD-3-Clause
---

# Presentation Slide Design Guidelines

These guidelines apply to **all** presentation tools and formats.

## 1. Outline First — Mandatory

**You MUST draft a structured outline before creating any slides. Do NOT skip this step.** A presentation is a narrative, not a document. Write a markdown outline that captures:

- The **core message** — one sentence the audience should remember
- The **narrative arc** — setup, tension/problem, resolution/insight, takeaway
- Key transitions between sections

**Present this outline to the human and wait for approval before proceeding.** Do not create slides until the human has reviewed and approved the outline. Adapt the structure to fit the topic — there is no single correct format. Example:

```markdown
# Outline: [Presentation Title]

**Core message:** [One sentence]

## 1. Opening — Hook
- [Surprising fact, question, or story]

## 2. Problem / Context
- [What challenge are we addressing?]

## 3. Key Insight / Solution
- [Main idea, 2-3 sub-points]

## 4. Evidence / Demo
- [Data, examples, or demonstration]

## 5. Takeaway / Call to Action
- [What should the audience do or remember?]
```

## 2. Minimize Text — Ruthlessly

**Slides are a visual aid, not a script.** The presenter speaks; slides support.

**Absolute maximum: 10 words per slide. The best slides have zero to three words.** Headlines and short phrases only — never sentences. If you need full sentences, put them in speaker notes.

**Use short headlines that state the takeaway**, not a topic label.

- Weak: "Q3 Sales Results"
- Strong: "Sales up 40%"

**Never put paragraphs or long bullet lists on slides.** If a slide has more than two lines of text, rethink it.

**One idea per slide.** If a slide makes two points, split it.

## 3. Visuals Over Text

**Show, don't tell.** Images, diagrams, and charts communicate faster than words. When you can replace text with a visual, always do it.

**Use full-bleed images.** If a slide has one image, it should fill the entire slide. A small centered image with empty space looks unfinished. Use overlay text sparingly.

**Download product logos and relevant images from the internet** when they would strengthen the slide. Fetch official logos rather than describing them in text.

**Ask the human for screenshots** if they would help illustrate a point — screenshots of UIs, demos, dashboards, or workflows are often more effective than any diagram you could generate. It's very good practice to ask.

**Replace bullet lists with visual layouts** — cards, columns, or icons with short labels side by side.

**Use diagrams** for processes, relationships, and architecture. Use charts for data, with headlines that state the takeaway.

## 4. Control Attention

**Reveal content progressively.** Don't show everything at once — use incremental builds so each point appears when the presenter discusses it.

**One visual focus per slide.** Use size, color, or position to create a clear focal point. Grey out elements that aren't the current focus.

## 5. Design for Clarity

- **Consistent styling.** One color palette, one font family, one layout grid throughout.
- **Large fonts.** Body text minimum 24pt, headlines 36pt+. If text must shrink to fit, there's too much text.
- **High contrast.** Dark on light or light on dark.
- **White space.** Empty space directs attention — don't fill every corner.
- **1-2 accent colors** against a neutral background.
- **No clutter.** Remove decorative elements, unnecessary logos on every slide, and ornamental borders.
- **Left-align text.** Easier to scan than centered.

## 6. Structure and Pacing

- **Start with a hook** — surprising fact, question, or image. Not an agenda slide.
- **Use section dividers** — simple slides with just a section title for mental breaks.
- **End with a clear takeaway** — restate the core message. Avoid generic "Thank you" or "Questions?" slides.
- **Keep it short.** Fewer slides than you think. Cut anything that doesn't directly support the core message.
