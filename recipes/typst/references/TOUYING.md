# Touying (Presentations)

Touying is a powerful presentation package for Typst. The name means "projection" in Chinese.

## Quick Start

```typst
#import "@preview/touying:0.6.1": *
#import themes.simple: *

#show: simple-theme.with(aspect-ratio: "16-9")

= Title

== First Slide

Hello, Touying!

#pause

Hello, Typst!
```

## Key Features

### Animations

```typst
// Sequential content reveals
#pause              // Pause between content
#meanwhile          // Show content simultaneously

// Math equation animations
$ f(x) &= pause x^2 + 2x + 1 \
       &= pause (x + 1)^2 $
```

### Available Themes

- `simple` - Minimal design
- `dewdrop` - Navigation bars
- `metropolis` - Modern academic style
- `university` - Institution-branded
- `aqua` - Clean, colorful
- `stargazer` - Dark theme

### Theme Usage

```typst
#import "@preview/touying:0.6.1": *
#import themes.metropolis: *

#show: metropolis-theme.with(
  aspect-ratio: "16-9",
  config-info(
    title: [My Presentation],
    author: [Author Name],
    date: datetime.today(),
  ),
)

= Section Title
== Slide Title
Content here...
```

### Speaker Notes

```typst
#slide[
  Main content visible to audience.
][
  Speaker notes only visible to presenter.
]
```

## Export Options

- Native Typst PDF output
- PPTX conversion support
- HTML export for web presentation

## Resources

- [Typst Universe](https://typst.app/universe/package/touying)
- [Documentation](https://touying-typ.github.io)
