# Dossier HTML — structure spec

Canonical structure for `<session>/dossier.html`. Rendered from `<session>/dossier.md` by the `/prep` orchestrator (step 5). One pass, single self-contained HTML file. Same look-and-feel across companies.

> **Why this lives in `templates/`:** so the rendering rules are auditable and consistent across runs. If you change the design, change it here once.

## 1. Output contract

- File path: `<session>/dossier.html`
- Self-contained (CLAUDE.md convention). External: Tailwind via CDN, Chart.js via CDN, nothing else. No images. No fonts beyond what Tailwind ships.
- Must work offline-friendly: if CDNs fail, the document still reads as plain text.
- Print-friendly: a `Print / PDF` button in the nav, `@media print` hides the nav and inserts a page break before the "Prep cards" section.
- Length target: ~500–700 lines. Density over completeness.

## 2. CDN includes (use exactly these)

```html
<script src="https://cdn.tailwindcss.com"></script>
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
```

## 3. Design system

**Palette** — clean medical / tech, neutral on white:
- Primary teal: `teal-600` / `teal-100` background tints
- Secondary indigo: `indigo-500` / `indigo-100`
- Warning amber: `amber-50` / `amber-200` borders for gaps and unknowns
- Danger rose: `rose-50` / `rose-200` for "things to avoid" or dissonance
- Neutral slate: `slate-50` background, `slate-900` text, `slate-200` borders
- Hero gradient: `from-teal-50 via-white to-indigo-50` + radial accents at 6% opacity

**Reusable components:**
- `.chip` — pill, padding `.25rem .6rem`, rounded-full, font-semibold-ish
- `.ring-soft` — `box-shadow: 0 0 0 1px rgba(15,23,42,.06), 0 1px 2px rgba(15,23,42,.04)` (cards live on this, not on heavy borders)
- Sticky `<nav>` with anchor links, backdrop-blur, hidden on print
- Numbered timeline items: vertical guideline + colored ring around a numbered circle
- Founder cards: gradient avatar circle with initials (no real photos)

## 4. Required sections (in order)

Always include all sections. If the source dossier has nothing for a section, render the section anyway with a placeholder card stating "Not found in research" — the candidate should *see* the gap.

| # | Section            | Anchor      | Source mapping                                                |
| - | ------------------ | ----------- | ------------------------------------------------------------- |
| 1 | Hero + TL;DR       | (top)       | dossier frontmatter + Company snapshot top bullets            |
| 2 | Company snapshot   | `#snapshot` | dossier `## Company snapshot`                                 |
| 3 | What they sell     | `#product`  | dossier `## Products / business model`                        |
| 4 | Momentum (timeline)| `#funding`  | dossier `## Recent challenges, news, or strategic moves` + funding from snapshot |
| 5 | Culture cheat sheet| `#culture`  | dossier `## Engineering culture` (split: declared / signals / dissonance) |
| 6 | Tech stack         | `#stack`    | dossier `## Tech stack` + JD frontmatter `tech_stack`         |
| 7 | Founders           | `#founders` | dossier `## Founders & key people` (or pull from interview process if absent) |
| 8 | Interview process  | `#process`  | dossier `## Known interview process`                          |
| 9 | Prep cards         | `#prep`     | **Derived** — synthesized by the renderer (see §5)            |
| 10| Known unknowns     | `#gaps`     | dossier `_log.md` confidence gaps + any "single-source" markers in the dossier |
| 11| Sources            | `#sources`  | dossier `## Sources`, in a `<details>` block                  |

Section 9 ("Prep cards") is the only section the renderer **generates** rather than transcribes. Everything else maps from existing dossier content.

## 5. Prep cards — the only generated section

Four cards, all derived from the dossier + JD. Same shape every time:

1. **Things to bring up naturally** (teal accent, ✓ icon)
   - 3–5 specific facts from the dossier the candidate can casually reference: a public number ("$2M seed"), a recent move ("Marshall pilot"), a brand-history detail ("ex-Soma Lab"), a founder origin story.
   - Selection rule: prefer items that are **specific** and **researched** over generic praise. The whole point is to signal "I read past the careers page."

2. **Questions you can ask** (indigo accent, `?` icon)
   - 3–5 questions that probe the most interesting parts of the dossier: a technical detail, a strategic move, a team-shape question.
   - Selection rule: never ask anything answered on the careers page. Each question should pull from a specific dossier finding and require the interviewer to think.

3. **Avoid** (rose accent, ✗ icon)
   - 3–5 anti-patterns specific to **this** company / role.
   - Selection rule: pull from `_log.md` confidence gaps (e.g., "don't claim hands-on with X — you'd be bluffing") and JD signals (e.g., "don't lead with comp", "don't treat in-person as negotiable").

4. **Role logistics** (slate-900 dark card, `i` icon)
   - Tabular `<dl>` of the JD frontmatter highlights: title, location, comp, hard filter, work auth, conversion path. Pure transcription, no synthesis.

If you don't have evidence for a card (e.g., no logistics data in JD), state "not enough data to populate" rather than inventing.

## 6. Hero specifics

- Two-color gradient title using `gradient-ink` style (teal→indigo).
- Tagline: one sentence, ≤ 25 words, capturing what the company actually does. Pull a verifiable claim from the dossier (don't paraphrase a marketing line).
- Stat grid: 5 stats, picked from {founded year, total raised, headcount range, reach (customers/users), HQ/role location}. If a stat isn't in the dossier, replace it with another concrete number (revenue, customer count, age in months) — never leave a stat blank.
- TL;DR card: 4 bullets, each ≤ 20 words. These are the four things the candidate must remember. Prefer signal-rich differentiators (voice-first, B2B, founder origin, founder-led interviews) over generic facts.

## 7. Momentum / timeline

- Vertical timeline with 4–7 entries, ordered chronologically.
- Each entry: date pill, headline, one-line description.
- If the dossier has funding rounds, render a small Chart.js doughnut to the right with the funding mix (round names + amounts).
- Add a green "no headwinds detected" callout when the dossier says no layoffs / pivots / leadership departures. If headwinds exist, swap to an amber "headwinds" callout that lists them.

## 8. Culture cheat sheet

Three columns side-by-side:
1. **Declared values** (teal header) — bullets straight from the dossier "declared values" sub-section.
2. **Verifiable signals** (indigo header) — bullets that are independently confirmable (team size, founder involvement, public artifacts).
3. **Dissonance / gaps** (rose header) — bullets calling out where declared values can't be triangulated (no Glassdoor/Blind reviews, no public engineer posts, etc.). This column is mandatory; it forces the candidate to face the gap.

## 9. Tech stack

Two cards side-by-side:
- **Confirmed** (emerald chip in header, dark chips for items): things named in the JD itself.
- **Inferred** (amber chip in header, light chips with ring): things derived by the researcher from product architecture but not directly stated.

Plus a small grid of "architecture-relevant truths" — 4 lessons the candidate should internalize about how the product shape constrains the stack (e.g., "voice-first → streaming and latency are first-class").

## 10. Founders

One card per named founder, plus an explicit placeholder card for any founder whose identity wasn't surfaced (use a dashed amber border). Each card has:
- Initial-only avatar with a teal→indigo or indigo→rose gradient
- Name + role
- 3–5 bullets pulled from the dossier
- A small footer card naming the round they likely interview for (matched to the dossier's interview-process info)

After the founder grid, render a dark slate-900 card with a one-paragraph "why this matters" that states the implication of the founder structure on the interview style (e.g., "all rounds with founders → trust-bar hire, not skills-bar hire").

## 11. Print rules

- Hide the sticky nav (`.no-print`).
- Insert a `page-break` before `#prep`. The first page should be the company overview; the second page is the prep cards.
- Targets ~11pt body text on print. Avoid full-bleed gradients on print pages.

## 12. Graceful degradation

- If a dossier section is missing entirely, render its anchor section with a single muted card: "Not found in research — see `_log.md` for why."
- If the dossier was generated in degraded mode (no Perplexity), render a banner above the hero: "⚠ Dossier was not generated — this view is JD-only. Re-run /prep with Perplexity MCP for full research."
- If `dossier.md` doesn't exist at all when the renderer runs, abort and tell the user: "No `dossier.md` found in the active session. The HTML render needs the markdown first."

## 13. Reference implementation

The first instance of this template was generated for `simcare-20260509-v2/dossier.html`. Use it as the visual reference when rendering any new company. The structure does not vary; only the content does.
