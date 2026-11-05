---
name: asif
description: Explain any topic as a short web slide deck pitched at one of three audiences, triggered by the tags ASIF-KIDDO (a 5-year-old), ASIF-BS (a general undergrad), or ASIF-PHD (a researcher who reads Nature papers in that field). Use this skill whenever the user's message contains "ASIF-KIDDO", "ASIF-BS", or "ASIF-PHD" in any capitalization (e.g. "Explain to me CRISPR. ASIF-KIDDO", "asif-phd: diffusion models"), or when they ask to re-explain the previous ASIF topic at another level ("now ASIF-PHD", "go deeper", "simpler"). Do not use it for ordinary explanation requests without an ASIF tag.
---

# ASIF — explain it *as if* I were a kid, an undergrad, or a PhD

The user names a topic and a level tag. You research the topic from legitimate sources and build a horizontal-swipe web slide deck (via the `guizang-ppt-skill`, Style A) that teaches it at exactly that level. The three levels are not the same content at three lengths: each one has a different goal, vocabulary, and kind of evidence.

## 1. Parse the request

- **Tag → mode** (case-insensitive, anywhere in the message): `ASIF-KIDDO` → Kiddo, `ASIF-BS` → Undergrad, `ASIF-PHD` → PhD.
- **Topic**: everything else in the message. Topics can be anything — science, history, economics, art, law, sports, a news event.
- **Several tags** in one message: build one deck per tag, simplest first.
- **Follow-ups**: "go deeper" moves up one level, "simpler" moves down one, a new tag alone reuses the previous topic.
- **Language**: write the slides in the language the user asked in.
- If the topic is genuinely ambiguous (e.g. "Explain Mercury" — planet, element, or god?), ask one short question; otherwise just start.

## 2. Research from legitimate sources

Every definition and factual claim must trace back to a source a careful teacher would accept. This matters because the decks are meant to be trusted and shared; a confident but unsourced slide is worse than a slightly simpler one.

**Acceptable sources**, roughly in order of preference:
- Peer-reviewed papers and academic books; university course pages and lecture notes
- Reference works: Encyclopaedia Britannica, Stanford Encyclopedia of Philosophy, Oxford references, Scholarpedia
- Official bodies: government agencies (NIH, NASA, CDC, NIST…), WHO, standards bodies, official project documentation
- Established outlets with editorial standards (Nature News, Scientific American, Quanta, major newspapers) for recent events

Wikipedia is fine for orienting yourself and finding primary sources, but cite what it cites, not Wikipedia itself. Avoid content farms, SEO blogs, and AI-generated summaries.

Use WebSearch to find sources and WebFetch to read them. Keep a running list of `{claim → source title, publisher/venue, year, URL}` as you go — you'll need it for the per-slide annotations. If you can't find a solid source for something, either leave it out or say on the slide that it's uncertain/contested. Never invent a citation.

### Extra research for PhD mode: the most-cited papers

The user wants the 1–2 most influential papers on the topic. Google Scholar has no API, so use OpenAlex (free, Scholar-like citation counts):

```
https://api.openalex.org/works?search=<topic keywords>&sort=cited_by_count:desc&per-page=10&select=title,publication_year,cited_by_count,doi,primary_location
```

Semantic Scholar is a good cross-check: `https://api.semanticscholar.org/graph/v1/paper/search/bulk?query=<keywords>&sort=citationCount:desc&fields=title,year,citationCount,externalIds,url`.

Lessons from earlier runs on these APIs:
- OpenAlex `search=` matches **full text**, so it returns unrelated mega-cited papers (PRISMA, Adam…) and some corrupted records. Use `filter=title.search:<title words>` instead, or look a known paper up directly: `https://api.openalex.org/works/doi:10.48550/arxiv.<arXiv id>?select=title,cited_by_count`. Always check the returned title matches; some arXiv DOIs resolve to the wrong work. If so, try the venue DOI (e.g. ACL Anthology `10.18653/...`).
- OpenAlex splits preprint and venue versions into separate records, so its counts run well below Google Scholar. Say so on the slide.
- Semantic Scholar returns HTTP 429 without an API key after a few calls. Treat it as optional and don't stall on it.

Raw citation sorting surfaces off-topic mega-cited papers, so read the titles and pick for relevance: typically the **seminal/original paper** plus **the most-cited review or the most important recent advance**. Report the citation count with the source and date ("~12,400 citations, OpenAlex, Oct 2026"). Also look for the official GitHub repo, key datasets, or benchmark pages when the field has them (ML, bioinformatics, physics simulation, etc.).

## 3. Write to the level

### The rule for every level: explain to someone who doesn't know the topic yet

The level tag sets how much the reader can *absorb*, not how much they already *know about this topic*. A PhD-level reader is a strong researcher, but often from a different field, so they are hearing about this thing for the first time just like the kid is. The user flagged early BS and PhD decks for diving into formulas before ever saying what the thing is. That loses the reader, however smart they are.

So every deck, at every level:

1. **Orients first.** Before any mechanism or formula, answer in plain words: *What is it?* (a one-sentence definition with no undefined jargon), *What does it look like in the real world?* (2–3 concrete examples the reader may already have met), and *What problem does it solve, or why should I care?*
2. **Ramps up gradually.** Each slide may assume only what earlier slides established. Introduce a term before using it and an idea before formalizing it. If a slide needs a concept that hasn't appeared yet, an earlier slide is missing.
Reference decks showing this: `examples/foundation-model-bs.html` (slide 2, "What is a foundation model?") and `examples/foundation-model-phd.html` (slide 2, "Orientation").

3. **Then goes as deep as the level allows.** The levels differ in where the ramp ends (kid: the gist; BS: how it works, with formulas; PhD: the research frontier) and how fast it climbs. All three start at the same place: what it is.

### 🧒 Kiddo — a curious 5-year-old (5–6 slides)

Goal: they walk away able to say what the thing *is* and *why it matters*, in their own words.

- **Few words, lots of pictures.** At most ~15 words per slide; one idea per slide. The picture carries the meaning, the words label it.
- **Examples star Miffy, Pokémon, and friends**: "Pikachu's tail makes a little zap — that zap is electricity!" Characters make the abstract concrete and fun. Use them in the *story*, but don't copy official artwork (it's copyrighted): use original cartoons that *evoke* them.
- **Drawing style: cute, hand-drawn pencil sketch.** The user rejected flat geometric cartoons as ugly. Start from `assets/kiddo-characters.svg.html` — paste its hidden `<svg><defs>` block right before `<div id="deck">` and place characters with `<use href="#bunny|zap|bot|books|pic">`. It provides a wobbly-line filter (`#sketch`, `#sketch2`), pencil hatching (`#hatch`), and chibi characters (big heads, big shiny eyes, rosy cheeks, off-register crayon fills). New props or characters you draw should follow the same recipe: fill shapes offset by ~2px under `filter="url(#sketch2)"`, then a `stroke="var(--line)"` outline under `filter="url(#sketch)"`, plus hatching for shadows. Also add `.slide{--line:#3b2a1c} .slide.dark{--line:#efe2cb}` to the template's `<style>` so outlines read as pencil on light slides and chalk on dark ones. Speech bubbles, houses, arrows and other scene shapes get `filter="url(#sketch2)"` too, so the whole picture looks hand-drawn (text inside SVGs stays unfiltered so it's readable).
- **Reference deck the user approved:** `examples/foundation-model-kiddo.html` — 6 slides, story-driven (Bunny, Zappy and Bo the robot), one idea per slide, pictures on the right and short text on the left. Match its look, tone and density.
- Everyday comparisons (snacks, toys, bath time, the playground). No jargon at all; if a real word is important, introduce it once as a "big-kid word".
- It's okay to simplify, but don't say false things. "It's a bit more complicated, but…" is fine.
- Suggested arc: 1) Cover with a question ("Why is the sky blue?") → 2–4) the idea through a character story → 5) "Now you know!" recap → (6) a fun "try this at home" or "big-kid word" slide.
- Sources still apply; put them in a small, quiet footer so they don't clutter the picture.

### 🎓 Undergrad — a smart non-specialist (7–9 slides)

Audience: has undergrad-level math (calculus, basic stats, linear algebra), science, and general knowledge, but **no specialization in this field**. Treat them as smart, not as insiders.

- **Slide 2 is always "What is it?"**: a plain-language definition, familiar real-world examples (for foundation models: ChatGPT, image generators, search), and the problem it solves. No formulas yet.
- Build from intuition → mechanism → formalism: give the analogy next, then show how it actually works.
- **Use formulas when the topic has a scientific core.** Introduce each equation with what it says in words, define every symbol right next to it, and give one worked example or numeric intuition. Don't drop an equation without explaining it.
- Define every field-specific term on first use.
- Include one slide on **where the simple picture breaks** or a common misconception.
- Diagrams over text: inline SVG for mechanisms, flows, and plots.
- Suggested arc: cover → **what is it** (definition, examples, problem solved) → why it matters / analogy → mechanism (1–3 slides, with formulas) → example/application → misconceptions & limits → recap + references.

### 🔬 PhD — can read (or write) this field's Nature paper (up to 10 slides)

Audience: a researcher with the capacity to read this field's Nature paper: strong math, comfortable with formalism and primary literature. But **assume they're new to this specific topic**, often coming from a neighboring field. They need the same orientation as anyone else, just delivered faster and more precisely. Skip hand-holding and cute analogies.

- **The first 1–2 slides after the cover orient, with no formalism yet:** what the thing is in a sentence, concrete instances (named systems, real examples), what problem it solves and why the field cares, and the key idea in one picture or one sentence. A reader from another field should be able to say "OK, I know what this is and why it's a big deal" before the first equation.
- **Pair formalism with figures.** The user found a formula-and-table-only PhD deck hard to follow: experts still understand faster when they can *see* the mechanism. Aim for a figure on most content slides, drawn as inline SVG with `currentColor` so it works on light and dark slides. Good choices:
  - pipeline or schematic diagrams of the method
  - a tiny worked illustration beside each equation (e.g. tokens with a masked slot, a similarity matrix with a highlighted diagonal, a causal-mask grid)
  - schematic plots of the key empirical result, with landmark data points labeled
  - block diagrams of competing variants (e.g. LoRA's frozen path vs its low-rank path, a multi-stage training recipe)

  Label anything not drawn from real data as "schematic", and keep the real numbers in the text. List slides of separate issues (open problems, references) don't need one.
- Then go as deep as the topic allows: precise definitions, the formal model/derivation sketch, key assumptions, regimes where it holds and fails, quantitative results.
- **Feature the 1–2 most-cited papers** (found in §2) with what they contributed and why they matter — on a dedicated slide or woven into the narrative.
- Cover the frontier: open problems, active debates, competing approaches, and where experts disagree.
- Add a **"Go further" slide** with the key papers, related GitHub repos, datasets/benchmarks, and the leading journals or venues for the topic.
- Suggested arc: cover → **orientation** (what it is, examples, why it matters; 1–2 slides) → formal setup → core theory/method (2–3 slides) → landmark papers → evidence/results → limitations & open questions → go further (+ references). If this exceeds 10 slides, merge go-further and references, or fold landmark papers into the method slides. Don't cut the orientation.

## 4. Source annotation on every slide

Any slide that uses published material gets a small source line at the bottom:

```
Source: Vaswani et al., "Attention Is All You Need," NeurIPS 2017 · arxiv.org/abs/1706.03762
```

Use author/organization, title, venue/publisher, year, and a clickable link (DOI preferred for papers). Multiple sources on one slide are separated by ` · `. Undergrad and PhD decks end with a **References** list collecting everything cited. Kiddo decks keep the footer tiny and skip the references slide unless there's room.

## 5. Build the deck with guizang-ppt-skill (Style A)

Invoke the `guizang-ppt-skill` skill and follow its workflow for building the HTML. Its Step 1 clarification questions are already answered by ASIF, so don't re-ask the user:

| guizang question | ASIF answer |
|---|---|
| Style | **A · 电子杂志 × 电子墨水** (`assets/template.html`) |
| Audience | the ASIF mode (kid / general undergrad / field researcher) |
| Length | Kiddo 5–6 · BS 7–9 · PhD ≤10 slides |
| Source material | your research notes from §2 |
| Images | inline SVG diagrams/cartoons and emoji you create; no external image files needed. Don't hotlink or embed copyrighted figures — redraw the idea or link to it in the source line |
| Theme | Kiddo → 🍂 牛皮纸 (Kraft Paper) · BS → 🖋 墨水经典 (Ink Classic) · PhD → 🌊 靛蓝瓷 (Indigo Porcelain) |
| Hard constraints | per-slide source footer (§4); slide-count limits above |

**Formulas (BS & PhD):** the Style A template has no math rendering, so add KaTeX to the `<head>`:

```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/KaTeX/0.16.9/katex.min.css">
<script defer src="https://cdnjs.cloudflare.com/ajax/libs/KaTeX/0.16.9/katex.min.js"></script>
<script defer src="https://cdnjs.cloudflare.com/ajax/libs/KaTeX/0.16.9/contrib/auto-render.min.js"
  onload="renderMathInElement(document.body,{delimiters:[{left:'$$',right:'$$',display:true},{left:'\\(',right:'\\)',display:false}]})"></script>
```

**Output location:** `~/Desktop/ASIF/<topic-slug>-<mode>/index.html` (e.g. `~/Desktop/ASIF/crispr-kiddo/index.html`), then open it in the browser with `open`.

Run guizang's visual self-check (its Step 4) before handing over — overflowing text and unreadable formulas are the most common problems. Two things learned from earlier runs:

- **Use straight apostrophes/quotes in English slide text** (`'` not `’`). The template's CJK fonts render curly apostrophes full-width, producing "what’ s".
- **To screenshot slides**, the entrance animations leave headless captures blank. Make a temporary copy with `[data-anim]{opacity:1!important;transform:none!important}` appended to `<style>`, capture each slide with headless Chrome (`--headless=new --window-size=1600,900 --virtual-time-budget=3000 --screenshot=… "file://…?slide=N"`), look at every image, then delete the copy.

## 6. Hand-off

Reply briefly: the file path, the slide count, a one-line summary per slide, and — for PhD mode — the key papers you picked with their citation counts. Offer to re-run at another level (e.g. "Want the ASIF-PHD version?").
