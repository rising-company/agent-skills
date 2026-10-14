# Editorial frame

The editorial frame makes a chart an infographic: the words, the sourcing,
and the honesty around the picture.

## Page anatomy

Top to bottom:

1. **Headline**: 2–6 words. It may pun or provoke, but it must still make
   sense to someone who misses the joke.
2. **Subhead**: the hook in one plain sentence, or a question the graphic
   answers.
3. **Controls** (if interactive): toggles that re-scale or filter, labelled
   with the reader's question ("scale by deaths", "show per person").
4. **Key**: each encoding in reader language. Put it near the picture, not
   in a footnote.
5. **The graphic** with 3–5 **annotations** on the marks that carry the story.
6. **Footer**: `Sources: …` · `See the data` link · `Design: …` ·
   `Updated: <Mon YYYY>`.
7. **Changelog** (living graphics): `UPDATED <Mon YYYY>:` followed by what
   changed and why. Corrections get their own `Correction <date>:` line
   that names the mistake.

## Voice

- Short, conversational and a little witty, aimed at the reader, not the
  analyst. Prefer "Spoiler: not much." to "Values are relatively low."
- Use concrete nouns and numbers. Round them to what matters ($1.7tn, not
  $1,712,340,000,000) and keep the exact value in the datasheet and tooltip.
- Titles can use wordplay (a portmanteau, a twisted idiom), but the subhead
  is always literal.
- Write annotations as findings ("3× more searched than anything with strong
  evidence"), not as labels ("Vitamin C").

## Honesty rules

- Show the grading or scoring method, or link to it.
- Say when positions are editorial rather than measured (concept maps,
  spectrums).
- Keep the date of each number. When figures come from different years, say
  so in the key or tooltip.
- When the hook depends on a choice (normalisation, threshold, exclusion),
  name the choice in the footer or offer a toggle.
- Contested topics get the strongest version of each side, and the evidence
  grade decides placement, not the author's view.

## Evidence ladder

Use this for "does X work / is X true" graphics. Grade each **claim** (a
substance or intervention × an outcome), not each substance.

| Grade | Meaning | Typical basis |
| --- | --- | --- |
| 6 Strong | Consistent benefit across high-quality trials | Multiple RCTs and a meta-analysis |
| 5 Good | Benefit shown, some caveats | At least one large RCT, or a consistent set of small ones |
| 4 Promising | Positive but limited | Small RCTs, mixed quality |
| 3 Conflicting | Studies disagree | Mixed RCT results |
| 2 Slight | Weak signals only | Observational, animal or in-vitro work |
| 1 None | No good evidence of benefit, or evidence of no effect | Null RCTs, or no studies |

Draw a "worth it" line between 4 and 5. Record in the datasheet, for each
claim: grade, a one-line rationale, key study links, and the date assessed.
Harmful effects get a separate marker, not a low grade.

## Critique checklist

Run this on a **screenshot**. Every item must pass:

- [ ] A stranger gets the hook from the headline, key and annotations alone.
- [ ] Area scales with value (`sqrt` for radii and squares). No truncated bar
      axes.
- [ ] Every number on the page traces to a sourced datasheet row, and the
      datasheet is linked.
- [ ] Units, currency year and scope are consistent, or the differences are
      disclosed.
- [ ] Labels are legible at the target size, with nothing overlapping or cut
      off.
- [ ] Colour encodes one thing and the key explains it. The page still reads
      in greyscale and for colour-blind readers.
- [ ] It works at 390px wide and in dark mode.
- [ ] Footer has sources, credits and an updated date.
- [ ] Nothing is copied from another publisher's graphic: the data is
      re-sourced, the copy is original, and the layout is your own.
