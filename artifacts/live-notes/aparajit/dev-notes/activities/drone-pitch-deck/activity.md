# Drone pitch deck 2026 (Ajit + Apara)

| Key | Value |
|---|---|
| status | Paused |
| slug | drone-pitch-deck |
| branch | none |
| ticket | none |
| notes | Replan v2 shipped: ₹14–21 Cr, 28 nm Ajit + Apara SoC at 750 MHz, MobileNetV3, Agam + ideaForge. Commits `dbecb9c`, `dcf5de2` (pushed). Milestone 10 (e2e) pending |

# Goal

Investor-ready seed package in `docs/pitch-deck-2026/`: verified references, two-scenario cost model (~70% chip / ~30% software engineering plus operations bucket), concrete MVPs, `investor-brief.md` first, then `investor-qna.md`, Beamer deck and speaker notes. Replan: single 28 nm Ajit + Apara SoC at 750 MHz, MobileNetV3-Large as AI workload, Agam Robotics (AP board) and ideaForge (field tests) as partners, Apara before EW, leaner AI-assisted teams. Every claim tagged Demonstrated / Target spec / Roadmap / Estimate / TBD.

# Scope

Aparajit is the software and silicon enablement stack around the indigenous Ajit SPARC-V8 processor (toolchain, CoRTOS, NuttX, TFLite inference). This activity turns it into a fundraising narrative for a sovereign drone autopilot (AP) chip and board with on-board AI: India defence first plus dual-use commercial, Europe (year 2+) as component supplier. Output is documents, references and a deck — no product code.

In scope: replan revision of brief (first, user sign-off), then Q&A, deck, speaker notes; new references (MobileNetV3, Agam, ideaForge, internal `docs/ajit.txt`, `docs/resnet50-metrics.md`); `tools/check_docs.py` updates. Out of scope: logos, team bios, measured power/thermal, video, sending to investors, product engineering.

# Background and Special Notes

- User-stated facts: NavIC demonstrated; NuttX and PX4 ported; PX4 SITL/HITL in JMAVSim/Gazebo within weeks; Ajit chips exist; FPGA prototypes for HITL; no drone boards yet. Ajit SIMD (64-bit vector ISA extension) exists but unused in ResNet-50 port.
- `docs/resnet50-metrics.md` (C-model, in progress): ResNet-50 int8 reference TFLM = 322.1e9 invoke instructions, CPI ≈1.13, ≈81 instructions and ≈92 cycles per executed MAC; optimized-scalar loop alone removes ~75 instructions/MAC. Final cycle totals still placeholders («TOTAL_CYCLES»).
- `docs/ajit.txt`: Ajit spec (vector ISA, 1–4 cores, 2.46 DMIPS/MHz, 750 MHz at 28 nm and 1.2 GHz at 16 nm projected, ~100 mW at 16/28 nm, 1x1x32 in IITB NavIC SoC at 65 nm); Apara 8-way VLIW, 65× int8 matmul, 200× int8 dot product vs 1x1x32; APARAJIT Micro = dual-core Ajit + 1 Apara, on FPGA, for UAV.
- `docs/agam-robotics.md`: agent-generated outreach + NRE outline (₹6–14 L per board design, excludes components). Agam V6X-RT: Pixhawk FMUv6X-RT, NXP i.MX RT1170, −40 to +85 °C, ₹26,000 module (PX4 docs, Agam site).
- ideaForge: IIT Bombay origin, listed 2023 [R14].
- Old "ResNet-50 2–4 s" claim dropped.
- Currency: ₹95/US$, ₹110/€, ₹130/£.

# Current Design

- Files: `investor-brief.md` (single source of numbers), `investor-qna.md`, `reference/README.md` (+ git-ignored downloads), `tools/check_docs.py`, `tools/fetch_refs.py`, `deck/main.tex` + `deck/beamerthemeajitpitch.sty` + `deck/build.sh`, `deck/speaker-notes.md`.
- Chip: one SoC, dual-core Ajit (with SIMD) + Apara, 28 nm-class shared-wafer run abroad, 750 MHz Target spec (IITB 28 nm projection), −40 to +85 °C, Indian design and IP; Indian-fab variant Roadmap. Ajit IP shared with partners under signed NDA (security requirement).
- AP board: targets Pixhawk Autopilot Reference Standard (FMUv6X, Pixhawk Autopilot Bus) through Agam's design, so the module fits any PX4-compatible carrier; Pixhawk Payload Bus noted for Apara camera payloads. Agam designs Ajit flight-controller module for their baseboard; v1 on existing Ajit chip/FPGA (MVP 1), v2 on new SoC (MVP 2). Board before Apara tape-out.
- MVPs:
  - MVP 1 "It flies" (M0–9): Agam module v1, PX4 HITL, flights, first ideaForge trials, MobileNetV3 + SIMD on Ajit CPU, SoC spec freeze.
  - MVP 2 "It sees, and it's ours" (M6–~20): Apara on FPGA, SoC tape-out ~M12, parts ~M16–18, module v2, ideaForge field tests, MobileNetV3 on Apara.
  - MVP 3 "It survives" (M12–24, full only): jamming/spoofing detection, GNSS-denied fallback, secure boot + hardware security, DGCA certification.
- AI timing at 750 MHz (Estimate; ≈92 cycles/MAC reference; speedups applied to reference):

  | Model | Reference | SIMD 5× | Apara 120× |
  |---|---|---|---|
  | MobileNetV3-Large (0.22 G MACs) | ≈27 s | ≈5.4 s | ≈0.22 s (~4–5 frames/s) |
  | ResNet-50 (4.0 G MACs executed) | ≈485 s | ≈97 s | ≈4.0 s |

- Power (Estimate): Ajit core ≈0.1 W [`docs/ajit.txt`]; SoC flight-only ≈0.2–0.4 W; Apara active ≈0.5–1 W; full AP board ≈1.5–3 W.
- Cost draft (Estimate; rates chip ₹32 L, software ₹25 L, security ₹30 L, leadership ₹35 L per year):
  - Lean ₹14.00 Cr, ~20 months, MVP 1 + 2: chip ≈8.48 (5 engineers 2.67, Agam board design +30% 0.65, EDA + 28 nm IP 1.20, 28 nm MPW + packaging 1.40, FPGA 0.20, lab/thermal 0.40, boards 0.55, field trials incl. ideaForge [TBD terms] 0.50, contingency 12%), software ≈3.58 (3 engineers 1.25, contract specialists — PX4 drivers, security review 0.75, 3 DGX Spark 0.30, simulation server 0.15, HITL 0.10, compliance 0.20, maintenance 0.50, contingency 10%), operations ≈1.94 (leadership 1.17, office/legal/NDA 0.37, travel 0.20, demos 0.20).
  - Full ≈₹21 Cr, 24 months, MVP 1–3: chip ≈12.04 (adds board rev 0.75, second spin / field-volume run 1.60, 300 field controllers 1.00, field trials 0.90), software ≈5.34 (adds 2 EW/security engineers M12–24 0.60, certification 0.90), operations ≈3.70 (3 leaders, EU groundwork 0.40).
  - Engineering split ≈70% chip / 30% software in both. Brief states split is indicative, renegotiated as partner terms (ideaForge, Agam quote) firm up.
- Evidence signals: `check_docs.py` passes; deck builds clean; slides reviewed.

# Current Plan

1. References: add MobileNetV3 paper (arXiv:1905.02244), Agam site + V6X-RT spec + PX4 Agam page, pixhawk.org + pixhawk.org/standards, ideaForge IITB origin source, internal `docs/ajit.txt` and `docs/resnet50-metrics.md` rows; download where possible.
2. Brief revision (user review): chip plan, Ajit spec, MobileNetV3 timing table, partners section (Agam working process + NRE, ideaForge), MVP reorder, power/temperature, cost model and ask, risks (foreign fab, 750 MHz unproven until silicon, partner costs unquoted), DAP clause demoted.
3. Update `tools/check_docs.py` (scenario totals, headline numbers, drop "2–4 s").
4. After brief sign-off: Q&A, deck, speaker notes from brief.
5. e2e check and render review.

# Milestones

1. [x] Verified reference set (original)
2. [x] Market study and cost model (original; superseded by milestone 7)
3. [x] `investor-brief.md` v1 (superseded by milestone 8)
4. [ ] `investor-qna.md` final — superseded by milestone 9
5. [x] Beamer deck v1 + speaker notes (superseded by milestone 9)
6. [ ] e2e (original) — superseded by milestone 10
7. [x] Replan references and cost model (R106–R116 added; lean 14.00, full 21.00)
   - tests:
     - MobileNetV3, Agam, ideaForge, internal Ajit/metrics rows in index with status
     - lean and full tables sum; software share 25–35%; per-MVP subtotals sum
     - user cost cuts reflected line by line
   - evidence: `python3 tools/check_docs.py --refs --costs`
8. [x] `investor-brief.md` v2 (user signed off)
   - tests:
     - no "2–4 s"; MobileNetV3 timing table at 750 MHz with Estimate tags
     - chip plan, Ajit spec, temperature, power placeholders, partners, MVP order Apara before EW
     - currency footnote; claim tags
   - evidence: `python3 tools/check_docs.py --brief`; user sign-off
9. [x] Q&A, deck, speaker notes v2 from brief (all checks pass; independent review fixes applied)
   - tests: numbers match brief; deck no overfull boxes; slides reviewed
   - evidence: `python3 tools/check_docs.py --qna --deck`; `deck/build.sh`
10. [ ] e2e: package consistent and presentable
    - tests: all checks pass; deck rebuilt; `[TBD:` list reported
    - evidence: `cd docs/pitch-deck-2026 && python3 tools/check_docs.py && deck/build.sh && pdftoppm -r 60 -png deck/main.pdf /tmp/deck-review/slide`

# Next Steps

1. User reviews `investor-brief.md` v2 (PDF via pandoc command in Conventions); apply edits to brief.
2. After sign-off: Q&A, deck, speaker notes from brief (full `check_docs.py` fails on Q&A/deck until then: old ₹16/36 and 2–4 s figures).
3. Untracked repo docs cited as internal refs (`docs/ajit.txt`, `docs/resnet50-metrics.md`, `docs/agam-robotics.md`) should be committed with the package.

# References

- `docs/pitch-deck-2026/initial-deck.md`, `initial-deck-qna.md` — copied to `artifacts/`; originals unchanged. Learned: original MVP/GTM framing; many unverified claims.
- `.dev-notes/activities/activities.md` — context-only. Learned: prior activities demonstrate NuttX, TFLite, ResNet-50 on Ajit emulator.
- `docs/resnet50-metrics.md` — context-only (untracked, in progress). Learned: C-model ≈81 instructions/MAC, CPI ≈1.13; SIMD estimate model; final totals pending.
- `docs/ajit.txt` — copied (`artifacts/ajit.txt`; source untracked). Learned: Ajit/Apara spec, process projections, APARAJIT Micro SoC.
- `docs/agam-robotics.md` — copied (`artifacts/agam-robotics.md`; source untracked, agent-generated). Learned: Agam outreach framing, NRE ranges ₹6–14 L (plan uses +30%), division of labour.
