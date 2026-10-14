---
name: editorial-infographics
description: Design an editorial infographic that turns a dataset or a body of knowledge into one surprising, sourced, readable picture, in the tradition of Information is Beautiful. Use when asked for an infographic, a visual explainer, "make this data tell a story", "put these numbers in context", "what does the evidence say", or "like Information is Beautiful". Not for dashboards or monitoring charts, and not for chart styling (palette, axes, accessibility); hand those to a dataviz skill.
---

# Editorial Infographics

An editorial infographic answers one question so that the reader gets it in a
glance and keeps exploring for minutes. It grew out of the practice
popularised by David McCandless and the Information is Beautiful studio
(<https://informationisbeautiful.net>). This skill teaches that **process**,
in our own words. Their graphics and texts are copyrighted, so everything here
is a description of the method plus original code. Link to their pages for
inspiration; never copy their images, data tables or copy into output.

The leading word is **the hook**: the single comparison, ranking or contrast
that makes a reader say "huh, I didn't know that". Every step below serves the
hook.

## How It Works

1. Find the hook.
2. Check the four elements.
3. Pick the form.
4. Shape the data.
5. Compose the page.
6. Build it.
7. Critique and ship.

### 1. Find the hook

Write the question the graphic answers, then the one-sentence answer the data
gives. Look for the hook in these places, in order:

- **Context**: an absolute number made meaningful by its neighbours (a
  billion-dollar figure placed beside other billion-dollar figures).
- **Mismatch**: perception vs reality, hype vs evidence, fear vs actual harm.
- **Rank**: who is biggest, oldest, deadliest, most common, and by how much.
- **Shape**: a hidden structure, such as a hierarchy, a spectrum, a cycle or a
  family tree of ideas.
- **Change**: a rise, a spike, a collapse, a "good news" trend nobody reports.

Done when you can state the hook as a sentence of at most 15 words that would
surprise an informed reader. If every candidate is obvious, say so to the user
and propose a sharper question or a richer dataset before drawing anything.

### 2. Check the four elements

A graphic that works needs all four. Score each one honestly:

| Element | Question | Missing it gives you |
| --- | --- | --- |
| **Information** | Is the data accurate, sourced and complete enough? | Pretty fiction |
| **Story** | Is there a hook, an angle, a reason to care? | Boring |
| **Goal** | Does it serve a reader's task (compare, choose, understand)? | Useless |
| **Visual form** | Does the shape of the picture match the shape of the data? | A spreadsheet |

Two elements make a sketch, and three make something that feels strangely
lacking. Done when every element has a one-line answer you could defend.

### 3. Pick the form

Choose by the **shape of the hook**, not by habit. Read
[`references/forms.md`](references/forms.md) for the full catalogue, with each
form's encodings, when to use it, its failure modes, and the matching
template. In short:

| Hook shape | First-choice form |
| --- | --- |
| Context for big amounts | Area squares (sized rectangles, nested parts) |
| Evidence vs popularity | Balloon race (bubbles rising by evidence strength) |
| Two measures, find the outliers | Quadrant bubble scatter |
| Hype vs reality over time | Mountain timeline (peaks of attention) |
| Ranked list with a twist | Annotated ranked bars or a dot strip |
| Categories × categories | Matrix / colour wheel |
| Concepts and their relations | Concept map, Venn, spectrum or hierarchy |
| One striking fact | Single-stat card ("good news") |

Done when you can name the form, map every data field to one encoding (size,
position, colour, order or label), and say why the form shows the hook better
than a plain bar chart. If a plain bar chart wins, use it.

### 4. Shape the data

- **Source every row.** Keep a `source` (and `url`) column. Prefer primary
  sources; when sources disagree, note the range or pick one and say which.
- **Make it comparable.** Use the same units, currency year and scope.
  Normalise when the raw number misleads (per capita, per year, share of
  total). When both raw and normalised tell different stories, offer a toggle
  rather than choosing silently.
- **Grade claims, don't just count them.** For "does X work" topics, grade
  each claim's evidence on an explicit scale (see the evidence ladder in
  [`references/editorial.md`](references/editorial.md)) and record why.
- **Cut to the hook.** Thirty well-chosen items beat three hundred. Keep the
  long tail in the published data, not in the picture.
- **Publish the datasheet.** The data lives in its own file (CSV/JSON) that
  the page loads and links to as "See the data".

Done when every value on the page traces to a row with a source, and the
units are consistent.

### 5. Compose the page

Every graphic carries the same editorial frame. Follow the anatomy and voice
rules in [`references/editorial.md`](references/editorial.md):

- a **headline** that is short and punny or pointed, plus a **subhead** that
  states the hook plainly
- a **key** that explains encodings in reader language ("bigger bubble = more
  people searching")
- **annotations** on the 3–5 marks that carry the story
- a **footer** with sources, the data link, credits, and an "updated" date
- a **changelog** of updates and corrections for living graphics

Done when a stranger could explain the hook after reading only the headline,
the key and the annotations.

### 6. Build it

Start from a template in [`templates/`](templates/). Each is a single
self-contained HTML file using D3 with inline data, plus light/dark themes:

- [`area-squares.html`](templates/area-squares.html): amounts in context,
  with nested parts
- [`balloon-race.html`](templates/balloon-race.html): evidence vs popularity
- [`quadrant-bubbles.html`](templates/quadrant-bubbles.html): two measures,
  with labelled quadrants

Copy the closest template, replace the `DATA` and `COPY` blocks, and adjust
encodings. If a dataviz or design-system skill is available, apply its
palette, type and accessibility rules on top. That skill owns how things look
in detail; this one owns what the graphic says.

Done when the page renders in a browser with your data, in both themes, at
desktop width and at 390px wide.

### 7. Critique and ship

Run the checklist in [`references/editorial.md`](references/editorial.md#critique-checklist).
Look at a screenshot, not the code. Fix every failed item, then hand over the
file, the datasheet, and a one-paragraph note: the hook, the form chosen and
why, the sources, and known gaps.

## Present Results to User

```
**<Headline>** — <subhead / hook in one line>

- Form: <form>, because <reason tied to the hook>
- Data: <n> items from <sources>; datasheet at <path>
- Gaps / judgement calls: <what you normalised, graded, or left out>
- Files: <html path>, <data path>
```

## Troubleshooting

- **There's no hook.** The dataset may be descriptive only. Ask what decision
  or curiosity the reader brings, or add a second dimension (cost, evidence,
  time, perception) that creates contrast.
- **Too many items to label.** Label only the annotated marks and expose the
  rest on hover/tap. Provide a filter or category toggle instead of shrinking
  text.
- **Sizes look wrong.** Area must scale with value (`sqrt` for radii and square
  sides). Never scale radius or side length linearly.
- **The reader asks "says who?"** The footer or the datasheet is missing a
  source. Fix that before polishing the visuals.
