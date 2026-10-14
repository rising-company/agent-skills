# Forms catalogue

Each form lists: the **hook** it serves, its **encodings**, how to build it,
its **failure modes**, and a public example to study (look, don't copy).
Examples live under `https://informationisbeautiful.net/visualizations/<slug>/`.

## Contents

1. Area squares
2. Balloon race
3. Quadrant bubbles
4. Mountain timeline
5. Annotated ranked bars / dot strip
6. Matrix and colour wheel
7. Concept map, Venn and spectrum
8. Hierarchy (pyramid, tree, nested)
9. Icon or tile grid taxonomy
10. Labelled map
11. Single-stat card
12. Long timeline / scale ladder

---

## 1. Area squares

- **Hook:** context for large amounts. "Is $500bn a lot?" Put it beside
  things the reader already knows.
- **Encodings:** square or rectangle **area** = amount. Colour = category
  (spending, earning, giving, cost of a problem). An inner square = a part of
  the whole (profit inside revenue, the US share inside the global total).
- **Build:** a squarified treemap (`d3.treemap().tile(d3.treemapSquarify)`)
  over a flat list sorted by value. Put the amount in large type and the label
  below it. Nest parts as a second, smaller square anchored bottom-right inside
  the parent. Template: `templates/area-squares.html`.
- **Failure modes:** mixing units or years. Comparing stocks (wealth) with
  flows (annual spend) without saying so. Tiny items with unreadable labels:
  group them as "other" or move them to a hover tooltip.
- **Study:** `billions-2`, `trillions`.

## 2. Balloon race

- **Hook:** what does the evidence say, and how does it compare with the hype?
- **Encodings:** **y** = evidence strength on an ordinal ladder (from "no
  evidence" up to "strong"), with a visible "worth it" line. **Size** =
  popularity or interest (search volume, sales, studies). **Colour** =
  category (condition, body system). **x** = grouping or jitter only.
- **Build:** band the y-axis by evidence grade. Use a force simulation with
  `forceY` to the grade, `forceX` to the category column and `forceCollide`
  on the radius. Label bubbles above a size threshold. Template:
  `templates/balloon-race.html`.
- **Failure modes:** treating the grade as continuous (it's a ladder of
  levels). Hiding the grading method. One bubble standing for several claims;
  give each claim its own bubble.
- **Study:** `snake-oil-scientific-evidence-for-nutritional-supplements-vizsweet`,
  `snake-oil-superfoods`, `snake-oil-cannabis-the-scientific-evidence-for-medical-marijuana`.

## 3. Quadrant bubbles

- **Hook:** two measures that trade off, where the corners carry meaning
  ("deadly but rarely spreads" vs "mild but spreads everywhere").
- **Encodings:** **x** and **y** = the two measures (use a log scale when
  values span orders of magnitude). **Size** = a third measure (cases, cost).
  **Colour** = type. Label each quadrant in plain words.
- **Build:** use a scatter with `sqrt`-scaled radii and draw quadrant lines at
  a meaningful threshold (the median, or a domain constant such as R0 = 1).
  Add a toggle when the axes have alternative metrics. Template:
  `templates/quadrant-bubbles.html`.
- **Failure modes:** quadrant lines at arbitrary points. Overlapping labels
  (nudge them, use leader lines, or label only the outliers).
- **Study:** `the-microbescope-infectious-diseases-in-context`,
  `the-rise-of-generative-ai-large-language-models-llms-like-chatgpt`,
  `caffeine-and-calories`.

## 4. Mountain timeline

- **Hook:** attention vs reality. Media panics rise and fade, and peaks of
  fear do not match actual harm.
- **Encodings:** **x** = time. Each topic is a stacked or overlapping smooth
  **area** ("mountain") of attention. **Colour** = topic. Optional toggles
  "scale by deaths" or "align to baseline" re-size the mountains by a reality
  measure.
- **Build:** use `d3.area().curve(d3.curveBasis)`, one path per topic, in
  shared or small-multiple baselines. Annotate the peak of each mountain with
  its trigger headline.
- **Failure modes:** an attention metric from a single source treated as
  truth (say what it measures). Stacking that hides the individual shapes.
- **Study:** `mountains-out-of-molehills`.

## 5. Annotated ranked bars / dot strip

- **Hook:** a ranking where the order or the gap is the surprise.
- **Encodings:** **position** along one axis = value. Sort by value. Highlight
  colour on the 2–3 items the story is about. Use a dot strip (one row of
  dots, one per item) when you show a distribution, such as the age at which
  achievements peak.
- **Build:** a plain D3 or SVG bar chart. Place annotations to the right of
  bars, and write a ranking as a sentence where possible ("#1 by a factor of
  4").
- **Failure modes:** decoration that hides the ranking. Truncated axes on bars.
- **Study:** `major-llms-ranked-by-performance`, `who-old-are-you`,
  `worlds-biggest-data-breaches-hacks-2` (bubbles by size and year, a
  variation on this form).

## 6. Matrix and colour wheel

- **Hook:** one category set × another category set, and where they agree.
- **Encodings:** rows × columns, with the cell filled by colour or icon. A
  wheel is a matrix bent into rings, where rings = groups (cultures) and
  segments = items (meanings).
- **Build:** use a CSS grid or SVG `rect` cells for the matrix. For the wheel,
  use `d3.arc()` with ring index → inner/outer radius. Use a legend of
  meanings rather than raw colour names.
- **Failure modes:** too many cells to scan; sort rows and columns so clusters
  appear.
- **Study:** `colours-in-cultures`, `taste-buds`, `horoscoped`.

## 7. Concept map, Venn and spectrum

- **Hook:** ideas and their relations, not numbers. Examples are what
  separates left from right, or which beliefs overlap.
- **Encodings:** spatial **position** = meaning. A spectrum puts two poles at
  each end and items between them. A Venn shows shared traits in overlaps. A
  concept map shows labelled links.
- **Build:** usually hand-placed SVG positions in the data
  (`{label, x, y, group}`) rather than an algorithm. Use a force layout only
  as a starting draft.
- **Failure modes:** implied precision (the reader assumes position is
  measured). Say "positions are editorial".
- **Study:** `left-vs-right-world`, `what-makes-a-good-data-visualization`,
  `ikigai-japanese-concept-to-enhance-work-life-sense-of-worth`,
  `what-is-consciousness`.

## 8. Hierarchy (pyramid, tree, nested)

- **Hook:** levels, branches or containment, such as sects within
  religions or distractions by intrusiveness.
- **Encodings:** vertical position or nesting = level, and width = count or
  importance.
- **Build:** `d3.tree()` / `d3.cluster()` for lineages, `d3.pack()` for
  containment, or a stacked trapezoid SVG for a pyramid.
- **Study:** `islamic-sects-schools-branches-movements`,
  `the-hierarchy-of-digital-distractions`.

## 9. Icon or tile grid taxonomy

- **Hook:** a field guide, an exhaustive set of named things grouped by
  type (fallacies, relationship advice, myths).
- **Encodings:** tile = item, group = colour band, and order = family. Each
  tile holds an icon, a name and a one-line example.
- **Build:** a responsive CSS grid of cards from JSON, with a filter by group.
- **Failure modes:** the hook gets lost. Add a headline insight ("the most
  common one is…") and highlight it.
- **Study:** `rhetological-fallacies`, `common-mythconceptions`,
  `good-relationtips-most-commonly-given-relationship-advice`.

## 10. Labelled map

- **Hook:** one fact per place ("every country is best at something").
- **Encodings:** geography = position, and the label is the content. Colour
  is only for the theme.
- **Build:** `d3.geoNaturalEarth1()` with a TopoJSON world file from a pinned
  CDN. Labels go at centroids, with leader lines for small countries.
- **Failure modes:** choropleths of raw counts that just show population.
  Normalise the counts, or label the places instead.
- **Study:** `because-every-country-is-the-most-beautiful-at-something`,
  `worlds-largest-empires`.

## 11. Single-stat card

- **Hook:** one striking trend or number, often good news nobody reports.
- **Encodings:** a huge number, a one-line meaning, and a sparkline or mini
  bar for the trend. Add a source line.
- **Build:** HTML/CSS card. Use an inline SVG sparkline from the time series.
  Cards collect into a grid ("most beautiful news of the year").
- **Study:** `beautiful-news`, `most-beautiful-news-of-the-year-2025`.

## 12. Long timeline / scale ladder

- **Hook:** extreme ranges, such as the far future, radiation doses or
  frequencies.
- **Encodings:** a log-scaled axis with landmark items the reader knows
  ("a banana", "a chest X-ray") anchoring the unfamiliar ones.
- **Build:** `d3.scaleLog()` vertical axis, labelled ticks at each order of
  magnitude, and items as labelled dots or blocks. Break the axis if needed,
  and say so.
- **Study:** `timeline-of-the-far-future`, `radiation-dosage-chart`,
  `per-second-vibrations-cycles-waves-rate-frequency`.
