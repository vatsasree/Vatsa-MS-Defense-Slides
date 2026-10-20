# MS Thesis Defense Deck — Implementation Plan (v2, all questions resolved)

**Candidate:** Tallapragada Shanmukha Sreevatsa · **Advisor:** Dr. Ravi Kiran Sarvadevabhatla · IIIT Hyderabad / CVIT · BharatGen
**Thesis title:** *Towards Visually Faithful Document Reconstruction* · **Defense date: 23 October 2026**
**Deliverable:** a self-contained HTML slide deck in this folder, pushed to a GitHub repo and presented from **GitHub Pages**. It is an **extension of the ICDAR 2026 oral deck** (`replica-agents.github.io/slides.html`): every interactive figure kept, re-ordered to the thesis chapters, extended with the thesis-only slides from the public-presentation `.pptx`, cross-checked against the thesis PDF (`Sreevatsa_MSThesis_compressed.pdf`, in this folder).

This document is written for the implementing agent (Opus). Sections: priorities (§0), source inventory with exact slide indices (§1), design decisions already taken (§2), slide-by-slide spec (§3), repo layout and GitHub Pages (§4), phased implementation with code-level guidance and gotchas (§5), QA (§6), resolved questions and remaining flags for Sreevatsa (§7), data appendix (§8).

---

## 0. How to read this plan — and what to build first

The defense is **two days** after this plan was written (21 Oct → 23 Oct 2026). Build in priority order and deploy early; everything marked **P0** must be live on Pages before anything **P1** starts.

| Tier | What | Rough agent effort |
|---|---|---|
| **P0** | Scaffold + Pages deploy of the ICDAR deck unchanged → re-skin (title, roadmap, six dividers, eyebrows) → reorder to thesis chapters → the two table slides (Table 4.1; baselines-only + reveal) → ladder callouts → human-agreement, metrics-formally, future-directions, conclusion and thank-you edits → re-deploy | ~½ day |
| **P1** | Taxonomy slide · ground-truth slide · robustness & reliability slide · **synthetic data engine slide (visual, new)** · 4th limitation card · GroundingDocQA frag · backups B1–B3, B6 · speaker notes on every slide | ~½ day |
| **P2** | Offline vendoring of Font Awesome / fonts · backups B4, B5, B8, B9 · polish (future-direction icons, variant shuffle key) | as time allows |

Conventions:
- **"ICDAR N"** = slide index as the ICDAR deck counts it (`slides.html`: title = 0, counter `N / 20`). Map in §1.2.
- **"pptx N"** = PowerPoint slide number in `Thesis Slides Sreevatsa-2.pptx` (1–92, `markitdown` numbering). Map in §1.3.
- **"thesis p.N / Table X.Y"** = the thesis PDF (75 pages; chapter map in §1.4).
- **Keep** = copy the ICDAR slide verbatim; only eyebrow/notes change. **New** = build from the components the ICDAR CSS already defines (`.card`, `.grid3/4`, `.stats/.stat`, `.rtab`, `.math`, `.stagecard`, `.stage-strip`, `.pagebox`, `.codecard`, `.takeaway`, `.section-slide`, `.frag`).
- One `index.html`, inline CSS + inline JS + a flat list of `<section class="slide">`. No framework, no build step.

---

## 1. Source inventory

### 1.1 Locations

| What | Path |
|---|---|
| ICDAR deck (the base) | `/Users/bharatgen-hyd-2025-33/Downloads/replica-agents.github.io/slides.html` (6,301 lines, 226 KB) |
| Project website the deck iframes for 4 live figures | `/Users/bharatgen-hyd-2025-33/Downloads/replica-agents.github.io/index.html` (357 KB, self-contained; `localize_data.js` is **not** loaded by it — ignore) |
| Assets both use | `/Users/bharatgen-hyd-2025-33/Downloads/replica-agents.github.io/static/` (350 MB, 198 files; see §4 for what to copy) |
| Public-presentation deck | `/Users/bharatgen-hyd-2025-33/Downloads/Thesis Slides Sreevatsa-2.pptx` (92 slides) |
| Thesis PDF | `/Users/bharatgen-hyd-2025-33/Downloads/Vatsa-MS-Defense-Slides/Sreevatsa_MSThesis_compressed.pdf` (75 pp, 20 MB — **git-ignore it**, see §4) |
| Dev-server config to copy | `replica-agents.github.io/.claude/launch.json` → `python3 -m http.server 8099` |
| Website internals reference | `replica-agents.github.io/WEBSITE_HANDOFF.md` |

Tooling on this machine: **no `node`, no LibreOffice, no `pdftoppm`**. `python3` (miniconda) has `python-pptx` and `PyMuPDF` (`fitz`) — run it **without `-I`**. Extract pptx pictures with `shape.image.blob` (script in §8.5); render thesis pages with `fitz` if a figure is needed.

### 1.2 ICDAR deck — slide map

| ICDAR | id / data-embed | Slide | Mechanism | Fate |
|---|---|---|---|---|
| 0 | `.title-slide` | REPLICA title, venue chip, roadmap chips, authors, logos | — | **Adapt** → thesis title |
| 1 | `#ladderSlide` | "What is VFDR?" — live fidelity ladder (invoice rebuilt L0→L3) | 3 frags; clones `#ladderSrc`, toggles `.L1/.L2/.L3` | **Keep + ❌ callouts** |
| 2 | — | Requirements for VFDR (4 axis cards + web-native / machine-readable) | 2 frags | Keep |
| 3 | `data-embed="teaser"` | Where the field stands — Fig 1A live, baselines only (`?only=marker,gpt5,qwen`) | `data-nav`, 2 frags | Keep |
| 4 | — | **Limitations of existing representations** — bottleneck wall SVG (`.btl`) | 2 frags | Keep |
| 5 | `data-embed="teaser"` | The same page, rebuilt (`?only=replica`) | 1 frag | Keep |
| 6 | `data-embed="fidfig"` | Six aspects of FID-HTML (`?embed=fidhtml&part=figure`) | `data-nav`, 6 frags | Keep |
| 7 | — | **Comparison with existing representations** — the wall with a second door (`.btl.fmtwall`) | 4 frags | Keep |
| 8 | — | Defining agents and tools (`.at-grid` + `.atfig`) | 1 frag | Keep |
| 9 | `#stagesSlide` | The agentic framework, stage by stage (4 `.stagecard`) | 4 frags | Keep |
| 10 | `data-embed="pipeline"` | Walkthrough (`?embed=demo&doc=newspaper&rate=1.5`) | `replica:demo` / `replica:demoState` | Keep |
| 11 | `#vfdrSlide` | VFDR-Bench carousel — 30 pages, 6 groups, autoplay | `caro`; Shift+arrows, P | Keep |
| 12 | — | Which metric measures what — radar (`.mradar`) + cards | 3 frags | Keep |
| 13 | `#resSlide` | Quantitative results — native 16-method table (`.rtab`) | 5 frags; `RES_GROUPS`, `renderResults`, `fitResults` | Keep + **generalise** (second instance for pptx 35) |
| 14 | `#ablSlide` | Ablation chart (`#ablChart`) | 3 frags; `ABL` | Keep |
| 15 | `#qualSlide` | Source scan vs live FID-HTML render | 4 frags; `QUAL_SET`, `fitQual`, `measureDoc` | Keep **as is (4 pairs)** |
| 16 | — | Limitations — 3 cards with `.limfig` SVGs | — | Keep + 4th card |
| 17 | `data-embed="down"` | What FID-HTML unlocks (`?embed=downstream`) | `data-nav`, 6 frags | Keep |
| 18 | — | Supervision for grounded document models (`.relsvg`) | 2 frags | Keep + 1 frag |
| 19 | `data-eyebrow="09 · Conclusion"` | Representation → Method → Benchmark | — | Keep, reorder, fix numbers |
| 20 | `.thanks` | Thank you — QR wall (REPLICA + M3Grounder + DoCoG + DocLayout-VL) + logos | — | Adapt |

Archived inside `slides.html` between `<!##` … `##>` (≈ lines 4724–4826): three per-stage formal slides (`.math` rows). **Restore as backup B3.** Also commented out: `.section-slide` divider markup and the Table-1 embed `?embed=fidhtml&part=table` (backup B1).

### 1.3 pptx — slide map and what is taken

| pptx | Content | Decision |
|---|---|---|
| 1 | Title, advisor, candidate | Take text → slide 00 |
| 2 | "The road ahead" — six sections + "represent → measure → generate" | Take → slide 01 |
| 3, 9, 20, 25, **36**, 61, 78, 80 | Section dividers (the "slide 36" pattern Sreevatsa meant) with motivating questions | Take questions → six `.section-slide` dividers |
| 4–6, 8 | OCR primer; content/structure/style triad | **Drop** (ladder level 0 is OCR; ICDAR 2 supersedes the triad) |
| 7 | Invoice twice + four ❌ callouts | **Take callouts** → live on the ladder (slide 03). Same Meridian & Co. invoice the ladder draws. |
| 10–19 | Related-work taxonomy | Condense → one interactive slide (07); text in §8.1 |
| 21–24 | Generic vs FID-HTML; Table 3.1 image; Fig 3.1; fidelity scale | Drop / backup B1 |
| 26 | Why a new benchmark | Fold into divider 04 + notes |
| 27 | **Table 4.1** (image) | Take → native interactive table (slide 14; data §8.2, verified against thesis p.29) |
| 28–29 | Stats, source datasets, mosaic | Already ICDAR 11; add the thesis source list to the kicker |
| 30 | Ground-truth creation | Take → short slide 16, **full human annotation** wording (see §7 flag F2) |
| 31–32 | Metric list; standard vs new | Already ICDAR 12 |
| 33 | Metrics formally | Take → slide 18 (formulas verified against thesis §4.4) |
| 34 | Human agreement 0.87 / 0.82 / 0.85 / 0.89 | Take → slide 19, **plus per-annotator Table 4.3** (§8.3) |
| 35 | 17-method baseline table | Take as **second instance of the results table, Table 5.1 numbers** (slide 20). The pptx-35 numbers are thesis **Table 4.4** (a different run, 17 methods) — kept only as optional backup B9 (§8.4) |
| 37–60 | Method slides | Already ICDAR 8–10; Fig 4 static → backup B6 |
| 62–64 | Headline, Table 5.1 image (numbers = ICDAR `RES_GROUPS`, verified), Fig 5.1 | ICDAR 13 / 3 / 5 |
| 65–74 | Qualitative pairs | ICDAR 15, **4 pairs, unchanged** |
| 75 | Robust across 15 languages & 10+ doc types; baselines drop up to 20 % beyond en/zh | Take → slide 29 (left half) |
| 76 | Ablation | ICDAR 14 |
| 77 | Agent & tool reliability tiles | Take → slide 29 (right half); full Table 5.3 → backup B2 |
| 79 | Limitations incl. high-resource skew | ICDAR 16 + 4th card (matches thesis §6.2's four) |
| 81–87 | Applications | ICDAR 17 (6 apps live) · **synthetic document generation → new visual slide 31b** · M3Grounder/GroundingDocQA → ICDAR 18 frag + backup B5 |
| 88–89 | Summary; future directions | ICDAR 19; new slide 34 using thesis §6.4's list |
| 90–92 | ICDAR card, thanks, blank | ICDAR 20 adapted |

### 1.4 Thesis PDF — chapter map (page numbers are PDF pages)

Ch.1 Introduction p15 (problem definition p16: the four requirements; contributions p17–18) · Ch.2 Background p19 (foundations / doc-to-markup / benchmarks / motivation) · Ch.3 FID-HTML p24 (Table 3.1 p25) · Ch.4 VFDR-Bench p28 (**Table 4.1** p29; Table 4.2 capability matrix p30; Fig 4.1 mosaic p31; construction p30–32; metrics §4.4 p32–35; **Table 4.3** human alignment p35; **Table 4.4** 17-method sweep p36; Tables 4.5–4.8 per-language / per-domain / column-layout / scanned-vs-digital p37–39; limitations p40) · Ch.5 REPLICA p41 (validation §5.6 p55–56; **Table 5.1** p57; **Tables 5.2, 5.3** p63; limitations p64) · Ch.6 Conclusion p65 (unified limitations p65–66; **§6.3.1 synthetic document generation p66–67; §6.3.2 GroundingDocQA p67–68**; future directions §6.4 p68).

### 1.5 Website embed API (what the iframe slides rely on)

`site.html?embed=<sectionId>` strips the website to one `<section>` (`body.embed-mode`). Ids used: `teaser`, `fidhtml`, `demo`, `downstream` (also `results`, `vfdrbench`, `qualitative`). Params: `part=` (`figure` / `table` / `compare`), `only=` (teaser subset), `hide=`, `doc=<gov|magazine|financial|newspaper>`, `rate=`, `speed=`, `autoplay=1`, `grade=1`.
Deck → iframe: `{type:"replica:demo", action: step|back|play|reset}`; `{type:"replica:navto", frag, frags}`. Iframe → deck: `{type:"replica:demoState", atStart, atEnd, playing}`.
Iframes are sized via `EMBED_CFG[data-embed]` — **any new embed slide needs a key there** (`fmt` already exists for B1).
Website externals: Google Fonts (Inter, JetBrains Mono), Font Awesome 6.5.2 (cdnjs — icons in the walkthrough), KaTeX 0.16.9 (jsdelivr; only the website abstract uses it).

### 1.6 Asset status
Everything referenced by `slides.html` and `index.html` exists on disk except `static/images/favicon.png` (harmless).

---

## 2. Decisions (all confirmed by Sreevatsa on 21 Oct 2026)

1. **Thesis order:** Represent (FID-HTML) → Measure (VFDR-Bench & metrics) → Generate (REPLICA) → Results. Chapter 04 ends on "the bar to beat"; chapter 06 reopens with the same table plus the REPLICA row.
2. **No OCR-history slides.** The ladder's level 0 is OCR; pptx 7's four ❌ callouts live on the ladder and flip to ✓ as levels repair them.
3. **Background = one interactive taxonomy slide** between the live "field fails" teaser and the bottleneck wall.
4. **FID-HTML slides (ICDAR 4, 5, 6, 7) untouched.** Stages slide and walkthrough untouched.
5. **Numbers:** both results tables use **Table 5.1** (= ICDAR `RES_GROUPS`). Thesis Table 4.4 (17 methods, different run) is *not* shown in the main deck.
6. **Six chapter dividers** in the existing `.section-slide` style (pptx 36 pattern).
7. **Title:** date 23 October 2026; **no collaborators line.**
8. **Thank-you:** keep the three sister-paper QRs + project QR + email.
9. **Ground truth slide:** full human annotation wording (see §7 F2 for the thesis-text caveat).
10. **Synthetic document generation gets its own visual slide** (31b), built as a live HTML mock, not a static image.
11. **Qualitative stays at 4 pairs** — only those four have real FID-HTML behind them.
12. **Hosting:** GitHub Pages is the primary presentation surface; offline vendoring demoted to P2 insurance; local `http.server` clone as fallback.

---

## 3. The final deck — slide-by-slide spec

Target: **37 main slides ≈ 36–40 min**, plus backups. [time] per slide; tier in bold. Eyebrows are exact strings (two CSS rules select on them, §5 gotchas).

### Front matter

**00 · Title — P0** (adapt ICDAR 0) [1:00]
- `.venue-chip`: `M.S. Thesis Defense <span class="oral">23 October 2026</span>`; second line in `.affil`-style under the chip: `IIIT Hyderabad · CVIT`.
- `<h1 class="thesis">Towards Visually Faithful<br>Document Reconstruction</h1>` with `.title-slide h1.thesis{font-size:84px;line-height:1.08}`.
- `.subtitle`: *Turning scanned pages and PDFs into HTML that doesn't just read like the original — it looks like it.*
- `.roadmap`: **Represent** FID-HTML → **Measure** VFDR-Bench + GPS · LPS · VFS → **Generate** REPLICA.
- `.authors`: `Tallapragada Shanmukha Sreevatsa`; `.affil`: `Advisor: Dr. Ravi Kiran Sarvadevabhatla`. No collaborators line.
- `.title-logos`: `cvit.png`, `iiith_logo.jpeg`, `bgen_logo.png`.
- Notes: "One-line thesis: turn documents into HTML that looks exactly like the original, not just reads like it. Three contributions, in thesis order: represent, measure, generate."

**01 · The road ahead — P0** (New, pptx 2) [0:50]
- eyebrow `Overview`; `<h2>How this talk is organised</h2>`. Six rows (flex column of card-rows, padding 18px 26px): `.num-badge` 01–06 · section name (serif navy 30px) · one-liner (ink-2 21px) · right-aligned mono chip on rows 03–06: `represent` / `measure` / `generate` / `results`. All visible, no frags. `.takeaway`: *The order follows the thesis: define the target, measure against it, then build the system that beats existing methods on that measure.*
- Rows: 01 Introduction — the problem, and why appearance matters · 02 Background & Related Work — how people have tried, and what they miss · 03 FID-HTML — a representation designed to be faithful · 04 VFDR-Bench & Metrics — a benchmark and metrics that measure fidelity · 05 REPLICA — an agentic framework that generates it · 06 Results, Limitations & Applications — evidence, honest edges, downstream impact.

### Chapter 01 · Introduction

**02 · Divider 01 — P0** [0:15] eyebrow `01 · Introduction`; `<h1>Introduction</h1>`; `.q` = §8.7 Q1.

**03 · What is Visually Faithful Document Reconstruction? — P0** (ICDAR 1 + callouts) [3:00]
- Keep the ladder entirely. Add a third grid column with four `.lnote` chips (markup/CSS/JS in §5 Phase 3): *OVERDUE stamp lost* (fix 3) · *VOID marker lost — reads as an active charge* (fix 3) · *Columns collapsed — which cell goes where?* (fix 2) · *TOTAL DUE no longer stands out* (fix 3). ❌ red until `frag >= fix`, then ✓ green, struck through. `data-frags` stays 3.
- Notes: ICDAR 1 notes; after level 0 add pptx 7: "Every word survived. A person reading the right-hand version could pay a cancelled charge, miss an overdue notice and misread what they owe. Style is data." At level 3: "and only now do all four turn green."

**04 · Requirements for VFDR — P0** (ICDAR 2, keep) [1:30] eyebrow `01 · Introduction`. Notes may cite thesis p.16: the four requirements are the thesis's formal problem definition.

### Chapter 02 · Background & Related Work

**05 · Divider 02 — P0** [0:15] `<h1>Background &amp; Related Work</h1>`; `.q` = §8.7 Q2.

**06 · Where the field stands — P0** (ICDAR 3, keep) [2:00] eyebrow `02 · Background`.

**07 · A taxonomy of prior work — P1** (New, pptx 10–19; thesis Ch.2) [2:30]
- `data-frags="4"`; `.grid4` of `.card`s, each `.frag` 1–4: `.tag` family → `<h3>` → `.chips` of systems → red "where it breaks" line → bottom **emits** chip (`Markdown` / `plain text` / `generic HTML` / `md · txt references`), the hand-off to slide 08. Text in §8.1. `.takeaway` on frag 4: *Logical methods recover content & hierarchy; physical methods chase pixels in narrow domains. Nobody delivers web-native HTML faithful in content, structure, style and rendered appearance.*
- Notes: pptx 10, 17 (DREAM answer), 18 ("the field cannot improve what it does not measure").

**08 · Limitations of existing document representations — P0** (ICDAR 4, keep) [2:00] eyebrow `02 · Background`.

### Chapter 03 · FID-HTML

**09 · Divider 03 — P0** [0:15] `<h1>FID-HTML</h1>`; `.q` = §8.7 Q3.
**10 · The same page, rebuilt — P0** (ICDAR 5) [1:00] · **11 · Six aspects of FID-HTML — P0** (ICDAR 6) [2:30] · **12 · Comparison with existing representations — P0** (ICDAR 7) [2:00] — all keep, eyebrow `03 · FID-HTML`.

### Chapter 04 · VFDR-Bench & Metrics

**13 · Divider 04 — P0** [0:20] `<h1>VFDR-Bench &amp; Metrics</h1>`; `.q` = §8.7 Q4. Notes: pptx 26 — a target with no ruler can't be optimised.

**14 · Against existing benchmarks (Table 4.1) — P0** (New native table, pptx 27 / thesis p.29) [1:30]
- `id="benchSlide"`, `data-frags="4"`, `<h2>Broader coverage — and the <em>only</em> one measuring layout &amp; fidelity</h2>`.
- `.rwrap > .rfit > table.rtab#benchTable`; columns Benchmark · Images · Languages · Domains · Format · TE · LS · PS · VF; rows §8.2 (VFDR-Bench last, `tr.ours`, hidden until beat 4). ✓ = `td.ok` (`#2f9e63`), ✕ = `td.no` (`--ink-3`).
- Beats: 1 light Images/Languages/Domains; 2 light Format (md / txt / LaTeX,HTML vs FID-HTML); 3 light TE·LS·PS·VF with every baseline PS/VF cell tinted `g0` — "two empty columns for the whole field"; 4 reveal the VFDR-Bench row with four green `best` ticks.
- `.takeaway`: *The only document benchmark annotated in FID-HTML — and the only one that scores physical structure and visual fidelity.* Lede wording per §8.2 note (READoc has more languages, OCRBench-v2 more images; the claim is combined breadth + PS/VF).

**15 · VFDR-Bench — 5,000 documents, annotated in FID-HTML — P0** (ICDAR 11, keep) [1:30] Kicker addition (thesis p.30–32): *sourced from IndicDLP · RVL-CDIP · PubLayNet · DocLayNet · M6Doc · IDL · PRImA · D4LA · SROIE · FUNSD · OJ4OCRMT · CCpdf + web-scraped pages · 20+ layout categories*.

**16 · Ground truth — how 5,000 pages were annotated — P1** (New, pptx 30) [0:45]
- `.grid3` of `.card`s with `.num-badge` 1/2/3: **Annotate** — trained annotators author ground-truth FID-HTML for every page (position, style, hierarchy); **Verify** — a second pass checks the rendered HTML against the source, with particular care for complex layouts and fine detail; **Result** — 100 % expert-annotated references, scorable on all four axes. `.takeaway`: *You cannot score position or style against a Markdown reference — so the reference had to be FID-HTML.* (Caveat §7 F2.)

**17 · Which metric measures what — P0** (ICDAR 12, keep) [2:00] eyebrow `04 · VFDR-Bench & Metrics`.

**18 · The metrics, formally — P0** (New, pptx 33; thesis §4.4) [1:30]
- `data-frags="4"`, `.grid2` of four `.card`s with `.mathrow/.math/.mathnote`:
  1. **Position (PS)** — `GPS(B_gt, B_pred) = |B_gt ∩ B_pred| / |B_gt|` over block boxes (layout models on source vs rendered HTML, manually refined); `LPS` identical at **line** level (Hi-SAM boxes); `PS = (GPS + LPS) / 2`. Note: *averaged over all boxes; penalises missed and hallucinated regions alike.*
  2. **Logical structure (LS)** — `NTED` between standardised DOM trees (`1 − TED / max(|T_gt|,|T_pred|)`); `TEDS` for tables, `CDM` for formulae, list-structure penalty for `<li>`; `LSS` = VLM-as-judge on the DOM tree + source image when normalisation is impossible.
  3. **Visual fidelity (VF)** — `VFS`: Gemini 2.5 Pro judges paired renders on layout consistency, font attributes, styling, image placement, overall appearance.
  4. **Text & overall** — `TE = (CRR + WRR) / 2`; `OS = (TE + LS + PS + VF) / 4`; CRR/WRR clipped per sample so OCR blow-ups don't dominate multilingual aggregates.

**19 · Do these metrics match human perception? — P0** (New, pptx 34; thesis Table 4.3) [1:00]
- `data-frags="2"`. `.lede`: *Seven expert annotators (document design, typography, layout analysis) ranked reconstructions of 100 documents from five systems; Pearson correlation of their rankings with each metric.*
- Beat 1: `.stats` — **0.87** VFS · **0.82** GPS · **0.85** LPS · **0.89** Overall (gold).
- Beat 2: a compact `.rtab` (`.graded`) of Table 4.3: rows VFS / GPS / LPS / OS × columns A1…A7 · Avg (data §8.3), every cell ≥ 0.80 → all `g3` green.
- `.takeaway`: *Every proposed metric exceeds 0.8 for every annotator; Overall aligns best.*

**20 · The bar to beat — 15 existing methods on VFDR-Bench — P0** (second instance of the results table, pptx 35 with **Table 5.1 numbers**) [2:00]
- `id="baseSlide"`, `table#baseTable`, built by `buildResults(tb, RES_GROUPS, {withOurs:false})`; `data-frags="5"`: TE → LS → PS → VF → graded (ICDAR beats, no green `best`).
- `<h2>The benchmark is <em>hard</em>, and it discriminates</h2>`; `.takeaway` on graded beat: *The strongest system tops out at OS 0.58. Text is largely solved; position and style collapse for the whole field. This is the bar REPLICA has to clear.*

### Chapter 05 · REPLICA

**21 · Divider 05 — P0** [0:15] `<h1>REPLICA</h1>`; `.q` = §8.7 Q5.
**22 · Defining agents and tools — P0** (ICDAR 8) [1:30] · **23 · The agentic framework, stage by stage — P0** (ICDAR 9) [3:00] · **24 · Walkthrough — P0** (ICDAR 10, `doc=newspaper&rate=1.5`) [3:30] — keep; eyebrows `05 · REPLICA`, `05 · REPLICA · Demo`.

### Chapter 06 · Results, Limitations & Applications

**25 · Divider 06 — P0** [0:15] `<h1>Results</h1>`; `.q` = §8.7 Q6; sub-line: *All numbers that follow are on VFDR-Bench, with the metric suite of Chapter 04.*

**26 · Quantitative results — all 16 methods — P0** (ICDAR 13 + reveal) [2:30]
- `id="resSlide"`, `data-frags="6"`. Beat 0: table as slide 20 left it (graded, `tr.ours.hide`). Beat 1: REPLICA row slides in (gold), grading and focus off. Beats 2–5: TE/LS/PS/VF axis walk with `best` green (ICDAR beats 1–4). Beat 6: graded board + legend (ICDAR beat 5).
- Notes: "This is the board you saw at the end of chapter four. One row is new." then ICDAR 13 notes.

**27 · Ablation studies — P0** (ICDAR 14, keep) [2:00] · **28 · Source scan vs live FID-HTML render — P0** (ICDAR 15, keep, **4 pairs**) [1:30]

**29 · Can we trust it? Robustness and reliability — P1** (New, pptx 75 + 77; thesis p.56, Table 5.3) [1:15]
- Left `.card` **Across languages and document types**: `.stats` OS_all **0.86** · OS_en **0.93** · OS_zh **0.89**; `.mathnote`: *REPLICA sustains performance across 15 languages and 10+ document types with minimal variance; baselines drop up to 20 % in overall and structural accuracy beyond English/Chinese (GPT-5: 0.62 → 0.50).* Right `.card.accent` **Agent & tool reliability** (trace-based): routing accuracy **0.97** · sub-HTML validity **0.96** · tool-call success **99.5 %** · reflection gain **+0.06** · reading-order accuracy **0.98**. `.takeaway`: *Exactly what the 22-language, 17-domain design of VFDR-Bench was built to surface.*

**30 · Limitations — P0 (3 cards) / P1 (4th card)** (ICDAR 16 + thesis §6.2's fourth) [1:30] eyebrow `06 · Limitations`. Add `card.lim` **High-resource skew**: small `.limfig` — script bars (Latin, CJK tall navy; Indic, Hebrew, Arabic, handwritten short red), caption *coverage leans to high-resource scripts and clean layouts; handwritten and degraded archival pages are under-represented*. Switch `.grid3.limgrid` → `.grid4`, `h3` 30 px.

**31 · What FID-HTML unlocks — P0** (ICDAR 17, keep) [1:30] eyebrow `06 · Applications`.

**31b · Synthetic document generation — FID-HTML as a template engine — P1** (New, visual; thesis §6.3.1, pptx 84) [2:00]
- `id="synthSlide"`, `data-frags="4"`, eyebrow `06 · Applications`, `<h2>Synthetic document generation — <em>FID-HTML as a template engine</em></h2>`.
- **Top strip** (reuse `.stage-strip`/`.st` with `.active/.done`): `Real document` → `REPLICA → FID-HTML` → `Localize mutable slots` → `Fill from schema` → `Render`. Lights up with the beats.
- **Body**: two `.pagebox.sized` boxes (same 462×625 rectangle idiom as the ladder) with a narrow `.codecard` between them.
  - Left box **Template** — an HTML mock of a **bilingual identity card** (mirrors pptx 84 without any real person): emblem placeholder (SVG), header `IDENTITY CARD` + a second-script line, photo placeholder (grey silhouette SVG), a seal circle, and a field grid: *Elector's name*, *Parent's name*, *Sex*, *Age as on 1.1.1995*, *ID No.* — each value wrapped as `<span class="slot" data-slot="name">…</span>` etc. Alternative mock if preferred: a payslip (name, employee id, month, earnings table, net pay). Build the card in HTML/CSS the way the ladder invoice is built; no raster.
  - Right box **Synthetic** — a clone of the left card (same `cloneNode` trick as the ladder) whose slot values are replaced from `SYNTH_VARIANTS` (3–4 variants: names in Telugu / Hindi / Latin script, parent name, sex, age 18–90 consistent with the date, ID in the same `AA/00/000/00000` pattern, photo silhouette recoloured). Layout, fonts, spacing, seal and emblem are byte-identical because only slot text changes.
  - Middle `.codecard` **Schema** (mono): `name: person · script=te` · `parent: person` · `sex: {M,F}` · `age: 18–90` · `id: ^[A-Z]{2}/\d{2}/\d{3}/\d{5}$` · `photo: portrait · synthetic`.
- **Beats**: 0 — left card only, right box shows a dashed "— " placeholder. 1 — *Localize*: slots get gold dashed outlines with a tiny mono label (`::after{content:attr(data-slot)}`); immutable regions take a faint navy tint ("fixed"). 2 — *Fill*: schema card appears; right card appears with variant 1, values flip (`opacity` swap, 300 ms). 3 — *Render & diff*: a diff overlay dims unchanged regions and tints changed values green; `.takeaway`: *Edits happen in FID-HTML, so typography, alignment, spacing and tables survive by construction — one template, unlimited privacy-safe variants.* 4 — use-case chips `medical reports · academic transcripts · bank statements · payslips · identity cards` + `→ pre-training / fine-tuning data for document VLMs where real corpora are blocked by privacy or law`.
- **Live during Q&A**: key `S` on this slide cycles `SYNTH_VARIANTS` on the right card (register in the keydown handler guarded by `slides[cur].id === "synthSlide"`).
- Optional beat 5 (**only if the source card is from a public dataset**): the real before/after pair from pptx 84 (`assets/pptx84_1739.png`, `pptx84_1740.png`) in two `.imgbox`es. The original shows a real person's face, name and ID number; GitHub Pages is public and permanent, so default is **off**.
- Notes (thesis p.66–67): "Curated real documents are converted to FID-HTML; the DOM hierarchy and semantic tags let us localise mutable regions — names, dates, account numbers, diagnoses, tabular values, addresses, identifiers — and parameterise them into slots. A generation engine fills them from domain schemas and constrained value generators. Because edits operate on FID-HTML, the output preserves typography, alignment, spacing and tables and renders back to images or PDFs. This matters exactly where real data is scarce for legal, financial or privacy reasons: medical reports, transcripts, bank statements, payslips, ID cards. When sensitive regions are mutated we inpaint photorealistically rather than black-bar, so downstream models cannot learn shortcut artefacts."

**32 · Supervision for grounded document models — P0 (keep) / P1 (frag)** (ICDAR 18 + 1 frag) [1:30] `data-frags="3"`: frag 3 adds under the gold arrow **200,000 documents · 2,000,000 pixel-grounded QA pairs** (GroundingDocQA, thesis §6.3.2). Notes: "FID-HTML's nested DOM maps onto phrase ⊂ line ⊂ block, so regions project to precise masks rather than boxes."

**33 · Conclusion — Represent → Measure → Generate — P0** (ICDAR 19 reordered) [1:00] eyebrow `07 · Conclusion`, give it `id="concSlide"` and rewrite the two eyebrow-keyed CSS selectors. Cards: FID-HTML (Representation) · VFDR-Bench + GPS·LPS·VFS (Benchmark & Metrics — **22 languages, 17 domains**) · REPLICA (Method, `.accent`). Stats: **0.93** VFS (gold) · **0.86** OS · **+39 %** over Qwen3-VL-30B at 3.75× smaller.

**34 · Future directions — P0** (New; thesis §6.4) [0:45] `.grid4` of `.card`s: **Multi-page reconstruction** with document-level memory (cross-page tables, section carry-over) · **Interactive human-in-the-loop refinement** — editing FID-HTML in web-native interfaces while preserving validity · **Large-scale document simulators** — the synthesis engine as training data for OCR, layout and multimodal systems · **Documents as editable, grounded, agent-compatible web resources** — reconstruction + synthesis unified. Notes also list pptx 89's items (font identification at scale, handwritten/degraded archives, proxy rendering) as the limitation-mirroring answers.

**35 · Thank you — P0** (ICDAR 20 adapted) Keep `.qrwall` (REPLICA large; M3Grounder / DoCoG / DocLayout-VL) and `.thanks-logos`. Replace the commented `.sesscard` with: `tallapragada.s@research.iiit.ac.in` · `replica-agents.github.io` · `REPLICA — ICDAR 2026 Oral` (small `icdar_logo.png`).

### Backup (after Thank you; `class="slide backup"`, eyebrow `Backup`)

- **B1 (P1)** Full format comparison — Table 3.1 / paper Table 1: `data-embed="fmt"`, `<iframe src="site.html?embed=fidhtml&part=table">` (markup exists, commented).
- **B2 (P1)** Agent & tool reliability, full Table 5.3 (§8.3b) in `.rtab`.
- **B3 (P1)** Per-stage formalism ×3 — un-archive the `<!## ##>` block; fix the double-escaped `&amp;lt;` in the Localize code card.
- **B4 (P2)** Synthetic generation — real before/after pair (pptx 84 pictures) *if cleared for publication*.
- **B5 (P2)** M3Grounder data engine figure (pptx 82 picture `1708`): FID-HTML DOM → masks at phrase ⊂ line ⊂ block; inpainting instead of black bars.
- **B6 (P1)** Complete REPLICA pipeline (Fig 4, `static/images/replica-agents-maindiag-final.png`, `.imgbox`).
- **B8 (P2)** Fine-grained benchmark analysis (thesis Tables 4.5–4.8) — four one-line findings with the small SC/MC and scanned/born-digital tables: position scores higher on single-column; fidelity higher on born-digital (no background restoration in baselines); Marker and Gemini most stable across languages; forms, IDs and receipts hardest for structure.
- **B9 (P2, optional)** Chapter-4 baseline sweep — Table 4.4, 17 methods (§8.4) in `.rtab`, **only** if Sreevatsa wants to pre-empt "why does GPT-5 read 0.51 in Table 4.4 and 0.58 in Table 5.1" (§7 F1).

---

## 4. Repository layout and GitHub Pages

```
Vatsa-MS-Defense-Slides/
├── index.html          ← the deck (from slides.html); Pages serves it at the repo URL
├── site.html           ← verbatim copy of replica-agents.github.io/index.html, iframe source only
├── static/             ← trimmed copy of the website's static/ (one shared copy; see list below)
├── assets/             ← deck-only additions (pptx-extracted figures for B4/B5, new SVG/PNG)
├── vendor/             ← P2: offline Font Awesome / fonts
├── .claude/launch.json ← {"name":"static","runtimeExecutable":"python3","runtimeArgs":["-m","http.server","8099"],"port":8099}
├── .nojekyll           ← Pages serves everything as-is (no Jekyll)
├── .gitignore          ← .DS_Store, **/.DS_Store, Sreevatsa_MSThesis_compressed.pdf
├── PLAN.md · README.md
└── Sreevatsa_MSThesis_compressed.pdf   (local reference only, ignored by git)
```

**What to copy into `static/`** (keeps the repo ≈ 220 MB instead of 350 MB; every file < 100 MB so plain git works, no LFS — Pages cannot serve LFS objects):
- Keep whole dirs: `images/demo/` (all four presets — the walkthrough's doc picker exists even in embed mode), `images/vfdr/`, `images/teaser/`, `images/downstream/`, `images/logos/`, `images/qr/`, `images/stages/`.
- Keep files: `images/applications.png`, `images/replica-agents-maindiag-final.png`, `images/replica_maindiag.png`, `images/qr_projectpage.png`.
- Keep from `images/qual/`: the four `QUAL_SET` pairs (`IndicDLP_1`, `X51005621482`, `00000087`, `LoRaLay_1`: `.png` + `_corrected.html`) **and** `20005.png` + `20005_html.html` (the website's Fid-HTML figure renders it).
- Drop: `static/paper/` (24 MB; set `PAPER_URL` in `site.html` to `https://replica-agents.github.io/static/paper/CameraReady-REPLICA_AGENTS__ICDAR_2026.pdf`), `Doc2Web-Pipeline-ExplainingFid-HTML.png` (24 MB, unused), `Teaser_v2.*`, `fidhtml_diag_horizontal.*`, the other `qual/` pairs. Side effect: `site.html`'s own (non-embedded) qualitative section shows placeholders if opened directly — acceptable; note it in README.
- Iframe `src` values: `index.html?embed=` → `site.html?embed=` (4 live + B1).

**GitHub Pages:** repo → Settings → Pages → *Deploy from branch* `main`, root. URL will be `https://<user>.github.io/Vatsa-MS-Defense-Slides/`; all paths are relative, so the sub-path works. First deploy takes a minute or two; later pushes a few seconds to a minute (hard-refresh to bust cache). **Presentation ritual:** open the Pages URL ten minutes early in Chrome, press `F`, step through every slide once with `↓` (this loads all five iframes and the qualitative probes into cache), press `Home`. If the room's network dies after that, the loaded page keeps working. Keep a local clone + `python3 -m http.server 8099` as a second fallback.

---

## 5. Implementation phases (P0 first; each ends in a browser check at 1920×1080 served over http)

### Phase 0 — Scaffold, smoke test, first deploy (P0)
1. Create the layout in §4; `cp slides.html index.html`; `cp index.html site.html`; copy the `static/` subset; copy `.claude/launch.json`; write `.gitignore`, `.nojekyll`, README.
2. In `index.html`: `src="index.html?embed=` → `src="site.html?embed=`. In `site.html`: `PAPER_URL` → absolute URL.
3. Serve; step through all 21 ICDAR slides: ladder, teaser arrows, Fid-HTML highlights, walkthrough step/play, carousel autoplay, results axis walk, ablation sweep, qualitative pairs in one rectangle, downstream zoom, QR wall. This is the regression baseline.
4. `git init`, commit, let Sreevatsa create the GitHub repo and enable Pages → verify the unchanged ICDAR deck works at the Pages URL **before** any content work.

### Phase 1 — Global re-skin (P0)
1. Title slide and roadmap slide per §3.
2. Six `.section-slide` dividers with §8.7 questions.
3. Renumber every `data-eyebrow`: `01 · Introduction`, `02 · Background`, `03 · FID-HTML`, `04 · VFDR-Bench & Metrics`, `05 · REPLICA`, `05 · REPLICA · Demo`, `06 · Results`, `06 · Limitations`, `06 · Applications`, `07 · Conclusion`, `Thank you`, `Backup`.
4. **Gotcha:** `.slide[data-eyebrow="09 · Conclusion"] …` appears in two CSS blocks (card sizing, `.stats .v` 140 px). Give the conclusion slide `id="concSlide"` and rewrite those selectors as `#concSlide …`.

### Phase 2 — Reorder into thesis chapters (P0)
1. Move `<section>` blocks into §3 order; `applyFrags()` dispatches on `id`, so order is free.
2. **Counter/progress:** compute `const mainCount = slides.filter(s => !s.classList.contains("backup")).length - 1;` for `counter` and `progress`; backups show `B1…` and a full bar; overview labels them `B1…`.
3. `next()` must stop at the Thank-you slide (do not drift into backups); backups reachable via `↓`, `End`, `O`. `prev()` symmetrical.

### Phase 3 — Ladder callouts (P0)
Markup after `.ladder-out` inside `.ladder-wrap`:
```html
<div class="ladder-notes">
  <div class="lnote" data-fix="3"><b>OVERDUE</b> stamp lost</div>
  <div class="lnote" data-fix="3"><b>VOID</b> marker lost — reads as an active charge</div>
  <div class="lnote" data-fix="2">Columns collapsed — which cell goes where?</div>
  <div class="lnote" data-fix="3"><b>TOTAL DUE</b> no longer stands out</div>
</div>
```
CSS: `.ladder-wrap{grid-template-columns:462px 700px 330px;column-gap:64px}` (1620 < 1744) · `.ladder-notes{grid-column:3;grid-row:1;align-self:center;display:flex;flex-direction:column;gap:16px}` · `.lnote{position:relative;border:2px solid var(--red);background:#fbe9e7;color:#8c2d22;border-radius:10px;padding:12px 16px 12px 46px;font-size:19px;line-height:1.35;transition:all .4s}` · `.lnote::before{content:"\2716";position:absolute;left:16px;color:var(--red);font-weight:800}` · `.lnote.ok{border-color:var(--green);background:#eaf5f0;color:var(--green);text-decoration:line-through}` · `.lnote.ok::before{content:"\2713";color:var(--green)}`.
JS in `renderLadder()`: `document.querySelectorAll("#ladderSlide .lnote").forEach(n => n.classList.toggle("ok", frag >= +n.dataset.fix));`

### Phase 4 — Generalise the results table (P0; slides 20 and 26)
```js
function buildResults(tb, groups, { withOurs })   // ICDAR body, skip g.ours when !withOurs
function renderResults(tb, n, { withOurs, reveal }) // step = RES_AXES[n - (reveal ? 2 : 1)];
                                                   // reveal: n===0 graded-no-ours, n===1 show ours row
function fitResults(slideEl)                        // query .rwrap/.rfit inside slideEl
```
`applyFrags`: `baseSlide` → `renderResults(baseTable, frag, {withOurs:false})`; `resSlide` → `renderResults(resTable, frag, {withOurs:true, reveal:true})`; both call `fitResults(slide)`. `RES_BEST` computed once from the full `RES_GROUPS`. Legend per slide (`.rlegend` inside each slide). `tr.ours.hide{display:none}`; reveal via a one-shot translateY transition. Keep `fitResults` on `resize` and `document.fonts.ready` for both slides.

### Phase 5 — Table 4.1 (P0; slide 14)
`const BENCH_ROWS` from §8.2; build `#benchTable` with `.rtab` classes; cells carry `data-c` 1–8; a small `renderBench(frag)` applies `.lit/.q`, `g0` tint on baseline PS/VF at beat 3, and un-hides `tr.ours` at beat 4. Reuse `fitResults(slide)`.

### Phase 6 — Remaining P0 slides, then P1 slides
P0: 01, 18, 19, 34, conclusion edits (33), thanks edits (35). P1: 07, 16, 29, 30 (4th card), 31b, 32 frag. Any new SVG `<pattern>`/`<marker>` needs a **unique id** (defs inside a `display:none` slide don't resolve — that is why `#hatch` and `#hatch2` both exist).

**31b build notes:** reuse `.pagebox.sized` (462×625) for both cards; author the mock card in HTML inside the left box with `<span class="slot" data-slot="…">`; `buildSynth()` clones it into the right box; `renderSynth()` toggles `.synth.L1/.L2/.L3/.L4` on a wrapper and writes variant values into the clone's slots (`querySelectorAll('[data-slot]')`). CSS: `.synth.L1 .slot{outline:2px dashed var(--gold);outline-offset:2px;background:var(--gold-soft)}` `.synth.L1 .slot::after{content:attr(data-slot);position:absolute;top:-16px;left:0;font:700 10px var(--mono);color:#8a6510;letter-spacing:.1em}` (slots `position:relative`) · `.synth.L3 .clone :not(.changed){opacity:.45}` `.synth.L3 .clone .slot.changed{background:#d3efdb;color:#0d5c32}`. Keydown: `if (slides[cur].id==="synthSlide" && (k==="s"||k==="S")) { nextVariant(); return; }`.

### Phase 7 — Backups (P1: B1, B2, B3, B6 · P2: B4, B5, B8, B9)
B1 uncomment + fix src; B3 delete `<!##`/`##>`, fix `&amp;lt;`; B4/B5 extract pictures with §8.5 into `assets/`.

### Phase 8 — Speaker notes (P1)
Every slide gets `data-notes`; keep ICDAR notes verbatim on kept slides; use §3 drafts and the pptx notes (§1.3) for new ones; end each with a timing cue.

### Phase 9 — Offline insurance (P2)
`site.html`: delete the three KaTeX tags (verify none of the embedded sections use `\(`, `$$`); vendor Font Awesome 6.5.2 (`vendor/fa/css/all.min.css` + `vendor/fa/webfonts/fa-solid-900.woff2`, `fa-regular-400.woff2`, `fa-brands-400.woff2` and `.ttf`); vendor Inter 400–900 and JetBrains Mono 400–600 as `woff2` with a local `fonts.css`, or accept system fallback. Test with Wi-Fi off.

### Phase 10 — Git hygiene
Commit per phase; do not push (Sreevatsa pushes); README: Pages URL, serve command, key map, presentation ritual, "site.html's own qualitative section is intentionally trimmed".

### Gotchas (from reading the ICDAR code)
- Stage is a fixed **1920×1080** scaled by `fit()`; slide body 1744 × 906.
- `data-frags="N"` overrides counted `.frag`s — keep in sync when adding beats (slides 03, 14, 19, 26, 31b, 32).
- `svg .frag{transform:none !important}` — keep SVG frags as `<g class="frag">`.
- `rewindEmbed()` resets embeds on leave; `syncNav()` sends the absolute beat on entry — new `data-nav` slides need both.
- `renderResults` grades from `td.textContent` — keep two-decimal text.
- `fitQual` forces both panes into one rectangle; `QUAL_SET` unchanged (4 pairs).
- `history.replaceState("#"+cur)` — slide hashes shift when slides are inserted; recheck any `#n` link at the end.
- The carousel runs only while `#vfdrSlide` is current — keep the id.
- Title `.roadmap` and the conclusion `<h2>` are the only two places with the ICDAR order hard-coded; both become Represent → Measure → Generate.

---

## 6. QA checklist
- [ ] Every slide at 1920×1080 with nothing clipped; 16:10 window letterboxes in the slide's own colour.
- [ ] `→` walks every beat, `←` reverses, `↓/↑` skip, `Home/End`, `O` overview (main + `B1…`), `N` notes on every slide, `?` help accurate.
- [ ] Slides 06, 10, 11, 24, 31 (and B1): iframes load from `site.html`, respond to arrows, rewind on re-entry.
- [ ] Slide 24: Space plays/pauses; arrows step; the slide ends at the last beat.
- [ ] Slide 15: autoplay, `P`, Shift+arrows, click-zoom.
- [ ] Slides 20 and 26: pixel-identical header/columns/row heights at the last beat of 20 vs beat 0 of 26.
- [ ] Slide 31b: slots outline at beat 1, variant fills at beat 2, diff at beat 3, `S` cycles variants; left and right cards stay the same size.
- [ ] Numbers consistent: 5,000 · 22 languages · 17 domains; VFS 0.93 · OS 0.86 · OS_en 0.93 · OS_zh 0.89; Pearson 0.87/0.82/0.85/0.89; 15 baselines + REPLICA = 16 methods; GroundingDocQA 200,000 / 2,000,000.
- [ ] Pages URL works from a clean clone; no reference to `../replica-agents.github.io`; repo < 300 MB; no file > 100 MB.
- [ ] Full run-through on the Pages URL in Chrome, fullscreen: 36–40 min.

---

## 7. Resolved questions and remaining flags

**Resolved (21 Oct 2026):** Q1 divider pattern = pptx 36 (typo for 34) ✔ · Q2 use Table 5.1 numbers for both tables ✔ · Q3 thesis order ✔ · Q4 date 23 Oct 2026, no collaborators line ✔ · Q5 keep three sister QRs + project + email ✔ · Q6 full human annotation ✔ · Q7 synthetic data engine as a visual slide (31b) ✔ · Q8 qualitative stays at 4 ✔ · Q9 thesis PDF provided and cross-checked ✔ · Q10 GitHub Pages primary, offline as insurance ✔.

**Flags to prepare for (no action needed from the implementer):**
- **F1 — Two GPT-5 numbers in the thesis.** Table 4.4 (p.36, 17 methods) reports GPT-5 OS 0.51 / VFS 0.73, Gemini 0.41 / 0.44, Marker 0.50 / 0.76; Table 5.1 (p.57) reports 0.58 / 0.86, 0.47 / 0.82, 0.53 / 0.74. The deck uses Table 5.1 throughout, so the slides are consistent — but a committee member who read both chapters may ask. Have the one-sentence reason ready (different run / protocol / subset). Backup B9 exists if you want to show Table 4.4 and explain it proactively.
- **F2 — Ground truth wording.** Thesis p.32 says every page was "first processed through our REPLICA pipeline to produce a high-fidelity reference HTML … manually refined by trained annotators", and p.40 says "Reference HTMLs, generated by REPLICA …". Slide 16 says full human annotation per your answer. Be ready to reconcile ("REPLICA produced a first pass; every page was then fully reviewed and corrected by trained annotators, so the released references are human-verified") — or tell the implementer to switch card 1 to the REPLICA-first-pass story.
- **F3 — "15 languages" vs "22 languages".** The benchmark has 22 languages; the robustness claim (thesis p.56) is "15 languages and 10+ document types". Slide 29 uses the thesis wording; slide 33 says 22 for the benchmark. Both are correct; don't let them blur.
- **F4 — pptx 84 pictures** show a real voter-ID card (face, name, number). 31b uses an HTML mock by default; include the real pair (B4 / beat 5) only if the card comes from a public dataset you are allowed to republish.

---

## 8. Appendix — data to paste

### 8.1 Taxonomy slide (07) card text
- **Foundations** · OCR, layout analysis & document VLMs · chips: Tesseract, PaddleOCR 3.0, Surya, GOT-OCR 2.0, DeepSeek-OCR, DocLayout-YOLO, LayoutLMv3, DiT, Donut, Nougat, SmolDocling, Dolphin · breaks: *both OCR paradigms optimise text; detectors give coarse blocks with no DOM; VLM markup stays semantic-first — every failure mode is a visual one* · emits `plain text · boxes · loose markup` · note: REPLICA's Segment stage builds on this line (DocLayout-YOLO + IndicDLP ensemble + Hi-SAM).
- **Doc → Markup** · chips: Marker, MinerU, Docling, Unstructured, Nanonets-OCR-s, OCRFlux · pdf2htmlEX, PyMuPDF, Pandoc · Qwen2.5/3-VL, olmOCR, Logics-Parsing · breaks: *linearisation is a design goal for them and a failure mode for us; PDF→DOM needs born-digital streams; end-to-end markup has no deterministic control over geometry* · emits `Markdown · JSON · generic HTML`. (Thesis Table 4.2 p.30 is the capability matrix — HTML/CSS/semantic tags/absolute position/font size/attributes/visual stitching — if a committee member wants detail.)
- **Physical reconstruction & screenshot-to-HTML** · chips: DREAM, PDFMathTranslate, image-to-LaTeX · WebCode2M, UICopilot, Design2Code, render-&-reflect loops · breaks: *domain-restricted — scientific two-column PDFs or text-light UIs; neither yields web-native, style-complete HTML for dense multilingual pages* · emits `LaTeX · UI HTML`. Notes: "DREAM is pixel-faithful but autoregressive and domain-narrow; screenshot-to-HTML is web-native but distribution-shifted from text-dense documents. REPLICA sits on both axes — logical and physical — in web-native HTML/CSS."
- **Benchmarks** · chips: OmniDocBench, OCRBench v2, READoc, KITAB-Bench, olmOCR-Bench, CC-OCR, DP-Bench · Image2Struct, WebCode2M, Design2Code · breaks: *text and logical structure only, annotated in plain text or Markdown — no axis for physical structure, rendered layout, font/style fidelity or figure/background reintegration* · emits `md / txt references`. Notes: "extending OmniDocBench is impossible: its references are Markdown, so there is no ground truth for position or style."

### 8.2 Table 4.1 rows (slide 14) — thesis p.29
| Benchmark | Images | Languages | Domains | Format | TE | LS | PS | VF |
|---|---|---|---|---|---|---|---|---|
| OmniAI OCR Benchmark | 1000 | 1 | NA | md | ✓ | ✓ | ✕ | ✕ |
| OmniDocBench | 981 | 3 | 9 | txt | ✓ | ✕ | ✕ | ✕ |
| CC-OCR | 800 | 10 | 6 | txt | ✓ | ✕ | ✕ | ✕ |
| olmOCR-Bench | 1403 | 1 | NA | md | ✓ | ✓ | ✕ | ✕ |
| OCRBench-v2 | 10000 | 2 | 26 | md | ✓ | ✓ | ✕ | ✕ |
| READoc | 3576 | 27 | 6 | md | ✓ | ✓ | ✕ | ✕ |
| dp-bench | 200 | 1 | 4 | md | ✓ | ✓ | ✕ | ✕ |
| KITAB-Bench | 917 | 1 | 9 | md | ✓ | ✓ | ✕ | ✕ |
| Image2Struct | 2100 | 1 | 3 | LaTeX, HTML | ✕ | ✓ | ✕ | ✕ |
| **VFDR-Bench (Ours)** | **5,000** | **22** | **17** | **FID-HTML** | ✓ | ✓ | ✓ | ✓ |

Lede: *Broadest combined coverage (22 languages × 17 domains), the only FID-HTML-annotated set, and the only one scoring PS and VF* (READoc has more languages, OCRBench-v2 more images — don't claim "most" on either alone).

### 8.3 Human alignment — Table 4.3 (slide 19, beat 2)
| Metric | A1 | A2 | A3 | A4 | A5 | A6 | A7 | Avg |
|---|---|---|---|---|---|---|---|---|
| VFS | 0.87 | 0.86 | 0.88 | 0.90 | 0.85 | 0.89 | 0.87 | **0.87** |
| GPS | 0.82 | 0.80 | 0.83 | 0.84 | 0.81 | 0.85 | 0.82 | **0.82** |
| LPS | 0.85 | 0.84 | 0.86 | 0.87 | 0.83 | 0.88 | 0.85 | **0.85** |
| Overall (OS) | 0.88 | 0.86 | 0.89 | 0.91 | 0.86 | 0.92 | 0.88 | **0.89** |

### 8.3b Reliability — Table 5.3 (backup B2; slide 29 uses the five bold ones)
Segment agent: AT-F1 0.98 · AV-F1 0.99 · Hierarchy consistency 0.98 · **Router: routing accuracy 0.97** · **Localize: sub-HTML validity 0.96** · **Reflection gain +0.06** · **Tools: tool-call success 0.995** · **reading-order accuracy 0.98** · font-fit success 0.97 · background-stitch success 0.96.

### 8.4 Table 4.4 — Chapter-4 baseline sweep (backup B9 only; **not** the main-deck numbers)
`[WRR, CRR, NTED, LPS, GPS, VFS, OS]` — Marker 0.65 0.80 0.80 0.25 0.35 0.76 0.50 · Docling 0.38 0.47 0.86 0.12 0.26 0.48 0.31 · RolmOCR 0.54 0.70 0.95 0.26 0.07 0.47 0.35 · smolDocling 0.34 0.40 0.90 0.13 0.24 0.45 0.28 · olmOCR-7B 0.54 0.67 0.93 0.28 0.10 0.28 0.32 · Nanonets-OCR-s 0.41 0.54 0.95 0.25 0.10 0.48 0.31 · GOT-OCR-2.0 0.42 0.48 0.93 0.17 0.25 0.60 0.33 · OCRFlux-3B 0.45 0.57 0.88 0.25 0.25 0.37 0.34 · Logics-Parsing 0.64 0.77 0.84 0.32 0.42 0.65 0.49 · Qwen2.5-VL-7B 0.50 0.61 0.80 0.14 0.24 0.37 0.34 · Qwen3-VL-30B-A3B 0.47 0.54 0.78 0.15 0.33 0.29 0.33 · Gemma-3-27b-it 0.57 0.62 0.85 0.16 0.25 0.32 0.34 · InternVL3-14B 0.28 0.46 0.83 0.22 0.29 0.27 0.28 · Gemini-2.5-Pro 0.63 0.69 0.81 0.19 0.31 0.44 0.41 · GPT-5 0.64 0.77 0.80 0.26 0.44 0.73 0.51 · WebCoder-1.3B 0.33 0.42 0.82 0.20 0.27 0.34 0.29 · WebSight-VLM-7B 0.14 0.42 0.85 0.19 0.29 0.16 0.23.

### 8.5 Extracting pictures from the pptx (B4, B5)
```python
# python3 (NOT -I) extract.py "/Users/bharatgen-hyd-2025-33/Downloads/Thesis Slides Sreevatsa-2.pptx" 84 assets/
import sys; from pptx import Presentation
prs = Presentation(sys.argv[1]); n = int(sys.argv[2]); out = sys.argv[3]
for sh in list(prs.slides)[n-1].shapes:
    if sh.shape_type == 13:
        open(f"{out}/pptx{n:02d}_{sh.shape_id}.{sh.image.ext}", "wb").write(sh.image.blob)
```
Known pictures: pptx 82 → `1708` (M3Grounder pipeline, 2048×781); pptx 84 → `1739` (original card) and `1740` (synthetic card, Telugu); pptx 90 → `1815` (ICDAR Vienna 2026 wordmark).

### 8.6 Source datasets line (slide 15 kicker) — thesis p.30–32
IndicDLP · RVL-CDIP · PubLayNet · DocLayNet · M6Doc · IDL · PRImA · D4LA · SROIE · FUNSD · OJ4OCRMT · CCpdf · web-scraped pages — 22 languages, 20+ layout categories.

### 8.7 Divider questions (verbatim from pptx)
Q1 — *Can we convert arbitrary scanned or digital documents into editable, web-native representations that preserve both semantic meaning and rendered appearance?* · Q2 — *Existing pipelines recover portions of text and hierarchy — but which signals do they preserve, and which do they quietly discard?* · Q3 — *What representation could preserve a page's text, hierarchy, geometry, and style all at once — and still be web-native and machine-readable?* · Q4 — *We have a target representation. But how do we systematically evaluate whether a reconstruction truly preserves both the semantics and the appearance of the source?* · Q5 — *How can arbitrary scanned or digital documents be reconstructed into FID-HTML with both semantic correctness and near pixel-level visual fidelity?* · Q6 — *Does structured agentic design actually beat brute-force model scaling on visually faithful reconstruction?*
