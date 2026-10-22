# Towards Visually Faithful Document Reconstruction — M.S. Thesis Defense

Slide deck for Tallapragada Shanmukha Sreevatsa's M.S. thesis defense (IIIT Hyderabad · CVIT, 23 October 2026).
It extends the ICDAR 2026 REPLICA oral deck, re-ordered into chapters
(FID-HTML → REPLICA → VFDR-Bench & metrics → Results, the ICDAR storyline). `PLAN.md` is the full spec.

- `index.html` — the deck (one file, inline CSS + JS, no build step).
- `site.html` — copy of the REPLICA project page; the deck iframes four live figures from it
  (`site.html?embed=teaser|fidhtml|demo|downstream`). Its own qualitative section is intentionally
  trimmed (only the four pairs the deck uses are in `static/images/qual/`), so opening `site.html`
  directly shows placeholders there and logs a few harmless 404s.
- `assets/` — deck-only images; `assets/receipt/` holds the images of the SROIE receipt that the synthetic-generation
  slide inlines (so that slide also works when `index.html` is opened from disk).
- `vendor/` — Font Awesome 6.5.2 (`vendor/fa/`) and Inter + JetBrains Mono woff2 (`vendor/fonts/`), for offline use.
- `static/` — trimmed copy of the website's assets (≈ 224 MB, every file < 100 MB, no LFS).

## Presenting from GitHub Pages

Repo → Settings → Pages → *Deploy from a branch* → `main`, `/ (root)`. The deck is then at

    https://<user>.github.io/Vatsa-MS-Defense-Slides/

All paths are relative, so the sub-path works. First deploy takes a minute or two; later pushes a few
seconds to a minute (hard-refresh to bust the cache).

**Ritual:** open the Pages URL ten minutes early in Chrome, press `F`, step through every slide once with
`↓` (this loads all iframes and the qualitative pages into cache), press `Home`. If the room's network
dies after that, the loaded page keeps working.

**Fallback:** a local clone, served over http (not `file://` — the iframes and the qualitative probes need it):

    python3 -m http.server 8099

then open http://localhost:8099.

The deck makes no network requests: Font Awesome 6.5.2 and the Inter / JetBrains Mono web fonts are
vendored under `vendor/` (≈ 1.4 MB, licences in the file headers), and the unused KaTeX tags were removed
from `site.html`. Only the "Paper" / dataset links on the project page itself point off-site.

## Keys

| Key | Action |
|---|---|
| `→` `Space` `PageDown` | next beat / slide (stops at Thank you) |
| `←` `PageUp` | previous beat / slide |
| `↓` / `↑` | next / previous slide, skipping beats; `↓` from Thank you enters the backups |
| `Home` / `End` | title / last backup |
| `O` | overview (backups shown as B1…B11) |
| `N` | speaker notes (every main slide ends with a timing cue) |
| `V` | view the current slide's HTML source |
| `F` | fullscreen |
| `B` | blank screen |
| `?` | help |
| `Space` (walkthrough) | play / pause; arrows still step |
| `P`, `⇧←` `⇧→` (VFDR-Bench carousel) | pause / step pages; click a page to enlarge |
| `S` (synthetic document generation, main slide and B9) | fill the template with the next variant |

Presenter remotes send PageUp / PageDown and work out of the box.

## Backups

After Thank you: B1 full format comparison (Table 3.1, live) · B2 robustness & reliability (§5.6) ·
B3 agent & tool reliability, every component (Table 5.3) · B4 how the ground truth was made ·
B5–B7 per-stage formalism (Segment, Localize, Assemble + Refine) · B8 the complete REPLICA pipeline figure ·
B9 synthetic generation on an identity-card mock (`S` cycles variants) · B10 M3Grounder data engine
(FID-HTML → pixel masks) · B11 fine-grained benchmark analysis (Tables 4.5–4.8).
