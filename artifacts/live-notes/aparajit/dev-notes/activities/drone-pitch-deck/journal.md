# Journal

## Pause (Active → Paused)

- why: user paused after committing package
- done: brief, Q&A, 28-page deck (incl. Meet Ajit / Meet Apara), speaker notes, reference index + `tools/fetch_refs.py`, `deck/build.sh`; all in commit `8af36fd` (unpushed); `check_docs.py` passes
- next: ask user what the ResNet-50 "2–4 s" figure measured; reword brief, then Q&A, deck, notes
- watch: `docs/resnet50-metrics.md` estimates ~120 s at 500 MHz (reference kernels, Ajit SIMD unused); Q&A and deck sign-off still pending

## Resume (Paused → Active)

- done: state unchanged since pause; package in `8af36fd`
- next: `replan-work` requested; reopened to Planning to take new user details

## Pause (Active → Paused)

- why: user paused after replan v2 was committed and pushed
- done: brief v2 (signed off), Q&A (40 Qs), deck and speaker notes rebuilt from it; ₹14–21 Cr, 28 nm Ajit + Apara at 750 MHz, MobileNetV3, Agam + ideaForge, Pixhawk target; commits `dbecb9c`, `dcf5de2` (93 cycles/MAC, 0.23 s on Apara); `check_docs.py` passes
- review: singham found no number conflicts; fixed A4 secure boot → MVP 3, notes slide refs, A5 rows, Q4 metric
- next: milestone 10, end-to-end consistency pass of the package
- watch: TBDs (team, IP relationship, ideaForge terms, valuation); partners are planned, confirm naming before external use
