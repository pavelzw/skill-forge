---
name: presentation-design
description: >-
  Slide design and storytelling guidelines for presentations.
  Use when creating, reviewing, or improving presentation slides.
  Focuses on visual clarity, audience attention, and narrative structure
  rather than any specific presentation tool.
license: BSD-3-Clause
---

# Presentation Slide Design Guidelines

This skill covers **design principles for effective presentations**. It is tool-agnostic — apply these guidelines whether you are generating slides in Typst/Polylux, LaTeX/Beamer, PowerPoint, Google Slides, reveal.js, or any other format.

## Start with a Story, Not Slides

**Draft a structured outline before creating any slides.** A presentation is a narrative, not a document. Before touching any slide tool, write a markdown outline that captures:

- The core message (one sentence — what should the audience remember?)
- The narrative arc: setup, tension/problem, resolution/insight, takeaway
- Key transitions between sections

**Present this outline to the human for review.** Humans are far better at judging narrative flow, pacing, and what will resonate with a specific audience. The outline is the most important artifact — get it right before designing any slides.

The structure of the outline should fit the presentation's topic and purpose — there is no single correct format. Here is one example to use as a starting point:

```markdown
# Outline: [Presentation Title]

**Core message:** [One sentence the audience should remember]

## 1. Opening — Hook
- [Surprising fact, question, or story that draws the audience in]

## 2. Problem / Context
- [What challenge or situation are we addressing?]

## 3. Key Insight / Solution
- [The main idea, broken into 2-3 sub-points]

## 4. Evidence / Demo
- [Data, examples, or live demonstration]

## 5. Takeaway / Call to Action
- [What should the audience do or remember?]
```

Adapt the sections freely — not every presentation needs all of these, and many will need different ones.

**One idea per slide.** If a slide makes two points, split it into two slides. The audience processes one concept at a time. More slides with less content each is always better than fewer dense slides.

## Minimize Text

**Slides are a visual aid, not a script.** The presenter speaks; the slides support. If the audience is reading your slides, they are not listening to you.

**Target no more than 15-25 words per slide.** Anything beyond that and the audience will read instead of listen. Headlines and short phrases work better than sentences. If you need full sentences, they belong in speaker notes, not on the slide.

**Use short, punchy headlines that state the takeaway** — not a topic label. The headline should tell the audience what to think, not what the slide is about.

- Weak: "Q3 Sales Results"
- Strong: "Q3 sales grew 40% after the pricing change"

**Never put paragraphs on slides.** If content requires detailed explanation, the presenter should say it out loud. The slide shows only the key phrase, number, or visual that anchors the point.

## Make Text Legible

**Use a large font size.** Body text should be at minimum 24pt, ideally 28-32pt. Headlines should be 36-44pt. If text needs to be smaller to fit, there is too much text — cut it down.

**Use a clean, sans-serif font.** Fonts like Inter, Helvetica, Source Sans, or the tool's default sans-serif work well. Avoid decorative or script fonts.

**Ensure high contrast.** Dark text on a light background or light text on a dark background. Avoid low-contrast combinations like light grey on white or yellow on light backgrounds.

**Left-align text.** Centered text is harder to scan, especially for multi-line content. Left alignment creates a clean reading edge.

## Prefer Visuals Over Text

**Images and diagrams always take precedence over text.** When you can show something visually, do it. A well-chosen image, diagram, or chart communicates faster and is more memorable than any sentence.

**Use full-bleed images** when an image carries the message. Let the image fill the entire slide with a short overlay headline if needed. A powerful image with minimal text is far more effective than a small image surrounded by bullet points. **If a slide has only one image or graphic, it should cover the entire slide** — a small centered image with empty space around it looks unfinished. Scale it up, use it as a background, or crop it to fill the frame.

**Replace bullet lists with visual layouts.** Instead of a list of 3-4 items, arrange them as cards, columns, or icons with short labels placed side by side. Each item gets an icon or small image, a bold label, and optionally one short line of description.

For example, instead of:
```text
Benefits:
- Fast
- Reliable
- Easy to use
```

Use a three-column layout where each column has an icon at the top, a bold keyword, and one short phrase beneath it. This is more scannable and visually engaging.

**Use diagrams for processes and relationships.** Flowcharts, architecture diagrams, timelines, and comparison tables communicate structure far better than prose.

**Use charts for data.** Follow standard data visualization principles: choose the right chart type, label data directly, use color intentionally, and write a headline that states the takeaway (not the chart type).

## Control Audience Attention

**Reveal content progressively.** Do not show an entire slide at once if it contains multiple elements. Use incremental reveals (animations/builds) so each point appears when the presenter is ready to discuss it. This keeps the audience focused on what is being said right now rather than reading ahead.

**One visual focus per slide.** The audience's eye should be drawn to exactly one place. If everything is equally prominent, nothing stands out. Use size, color, or position to create a clear focal point.

**Use contrast to direct attention.** Grey out or dim elements that are not the current focus. Highlight the active element with a bold accent color or larger size. This is especially effective when walking through a list or process step by step.

**Pause on key slides.** Important points deserve their own slide with generous white space — not a packed slide that rushes past. Give the audience time to absorb the message.

## Design for Clarity

**Use consistent styling.** Pick one color palette, one font family, and one layout grid and stick to them throughout. Consistency looks professional and avoids distracting the audience with visual novelty.

**Use white space generously.** Empty space is not wasted space — it directs attention and reduces cognitive load. Resist the urge to fill every corner of a slide.

**Limit your color palette.** Use 1-2 accent colors against a neutral background. More colors create visual noise. Use the accent color only to highlight what matters most.

**Avoid clutter.** Remove decorative elements, unnecessary logos on every slide, ornamental borders, and drop shadows. Every element on a slide should earn its place by contributing to the message.

**Use high-quality images.** Blurry, stretched, or watermarked images undermine credibility. If you don't have a good image, use a clean diagram or no image at all rather than a bad one.

## Structure and Pacing

**Start with a hook.** The opening slide should grab attention — a surprising statistic, a provocative question, a compelling image, or a short story. Do not start with an agenda slide or a title slide that just states the topic.

**Use section dividers.** Between major sections, use a simple slide with just the section title or a key question. This gives the audience a mental break and signals a shift in topic.

**End with a clear takeaway.** The final slide should reinforce the core message. A strong closing slide might restate the key insight, pose a call to action, or end with a memorable image. Avoid ending on a generic "Thank you" or "Questions?" slide — combine the takeaway with the invitation for questions.

**Keep presentations short.** Aim for fewer slides than you think you need. A 20-minute talk rarely needs more than 15-20 slides. Cut ruthlessly — if a slide doesn't directly support the core message, remove it.

## Common Pitfalls

- **Wall of text.** If a slide has more than 3 lines of text, rethink it.
- **Reading slides aloud.** The presenter should add context and narrative, not read what's on screen.
- **Bullet point overload.** Lists of 5+ items are hard to process. Group, prioritize, or visualize them instead.
- **Inconsistent design.** Mixing fonts, colors, or layouts across slides looks unprofessional.
- **Too many animations.** Subtle builds are good; flying, spinning, or bouncing transitions are distracting.
- **Data without context.** A chart without a headline that explains the takeaway leaves the audience guessing.
- **Small or low-contrast text.** If you have to squint, the audience in the back row has no chance.
