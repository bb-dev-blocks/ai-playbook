# Conventions

## Claims and sources
- Every claim carries one tag: Demonstrated / Target spec / Roadmap / Estimate / TBD placeholder.
- Partner names (Agam Robotics, ideaForge) as planned partners only; no signed-partnership or certification claims (e.g. "Pixhawk-compliant target", not "Pixhawk-certified").
- Market or policy metric needs a source in `docs/pitch-deck-2026/reference/README.md`; else drop or soften to range.
- No fabricated, future-dated, or unfindable citations.
- Placeholders exactly `[TBD: <what>]` so `rg '\[TBD:'` finds all.

## Investor prose
- Plain language; define jargon once in one line; headline sentence first.
- Factual defence tone; no combat rhetoric.
- ₹ primary currency; crore notation; footnote rates ₹95/US$, ₹110/€, ₹130/£ at first ₹ figure.
- No ASCII art in documents; tables in Markdown.

## Single source of numbers
- Cost model and MVP timeline defined once in `investor-brief.md`; Q&A and deck copy from it.
- Any number change updates brief, Q&A, deck together.

## Deck style
- One headline message per slide; ≤ ~3 bullets or one visual.
- Theme colours and fonts only via local theme file; no ad-hoc colours in slides.
- Rendered pages reviewed visually for overflow before handoff.
- Parameterised TikZ/pgfplots styles (`#1`) live in `deck/beamerthemeajitpitch.sty`; `#` inside a frame body breaks beamer.
- Fonts declared with `Ligatures=TeX` so `--` renders as an en dash.

## Setup, build, test and install notes
- cwd: `docs/pitch-deck-2026/`
- toolchain: TeX Live 2026 (Homebrew) with `xelatex`, `latexmk`, `beamertheme-metropolis`, `pgfplots`, `tcolorbox`, `fontawesome5`; Fira Sans font; `pandoc`; `pdftoppm` (poppler) for slide rendering.
- build deck: `deck/build.sh` (`deck/build.sh clean` removes aux files)
- re-download references: `python3 tools/fetch_refs.py [--only R01,R31] [--force]`
- render slides for review: `pdftoppm -r 60 -png deck/main.pdf /tmp/deck-review/slide`
- export docs: `pandoc investor-brief.md -o investor-brief.pdf --pdf-engine=xelatex -V mainfont=FiraSans -V mainfontoptions="Extension=.otf,UprightFont=*-Regular,BoldFont=*-SemiBold,ItalicFont=*-Italic,BoldItalicFont=*-SemiBoldItalic" -V geometry:margin=2cm -V fontsize=10pt -V colorlinks=true` (same for `investor-qna.md`). Fira is in the TeX tree, not system fonts, so name it by file stem. PDFs are git-ignored (`docs/.gitignore` `*.pdf`).
- Markdown lists need a blank line before them or pandoc merges them into the paragraph.
- check placeholders: `rg -n '\[TBD:' investor-brief.md investor-qna.md deck/`
- check docs: `python3 tools/check_docs.py`
