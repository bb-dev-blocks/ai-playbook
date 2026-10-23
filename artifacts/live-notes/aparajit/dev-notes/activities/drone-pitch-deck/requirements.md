# Requirement Definition

## Investor audience and ask
- kind: end-user-interface
- Primary reader: Indian early-stage deep-tech / defence-tech VCs and angels; seed or pre-seed round.
- Readable by generalist investor: plain language, outcome first, jargon defined in one line; deep tech in appendix.
- Ask: lean ≈₹14 Cr (MVP 1 + 2), full ≈₹21 Cr (MVP 1–3); use of funds tied to MVPs; distribution indicative and negotiable.
- Secondary paths, one slide/section each: India non-dilutive (iDEX, TDF, Mission Drone Shakti); Europe later (EDF SME calls).

## Cost analysis
- kind: end-user-interface
- Engineering split ~30% software development, maintenance, standard compliance; ~70% chip work (development, enhancement, tape-out, first field-test controllers).
- Separate operations and go-to-market bucket (leadership, BD, admin/legal, trials) outside the 30/70 split.
- Must include FPGA boards and 4–5 NVIDIA DGX Spark 128 GB workstations (~₹50 L total, user figure).
- Per scenario, per MVP, per category: people, EDA tools/licences, fab + packaging, boards, compute, test + certification, contingency (~10%).
- Each line marked estimate with public benchmark source or stated reasoning.
- Lean scenario funds MVP 1 + MVP 2 (~20 months); MVP 3 (EW) shown as full-scenario / next-round.
- `tools/check_docs.py --costs` must pass (sums, 25–35% software share of engineering).
- Replan cuts (user): software team 3 engineers; chip design + verification 5; board design outsourced ≈₹0.5 Cr; travel ≈₹15–20 L; maintenance and release ≈₹0.5 Cr; 3 DGX Spark ≈₹0.3 Cr total; FPGA boards ≈₹0.2 Cr (some owned); add high-end simulation server ≈₹0.15 Cr. Smaller teams justified by AI-assisted engineering.
- Agam figures (AI-generated source) over-approximated ~30% upward: NRE ₹6–14 L → ₹7.8–18.2 L per board design.
- Lean ask ≈₹14 Cr (headroom for investor negotiation). Software bucket +₹0.5–1 Cr in both scenarios. Reasonable breathing room in all amounts.
- Brief states fund distribution is indicative and will be renegotiated as partner terms firm up (e.g. ideaForge field-trial costs unknown → placeholder line).

## Claim tagging and credibility
- kind: end-user-interface
- Every claim tagged: Demonstrated / Target spec / Roadmap / Placeholder.
- Demonstrated: NavIC positioning; ResNet-50 int8 runs on Ajit (TFLite Micro, cortos2, qemu + C-model); NuttX and PX4 ported to Ajit. Old "2–4 s per image" claim dropped (C-model contradicts it).
- Near-term (state as "in weeks"): PX4 SITL/HITL tests in JMAVSim and Gazebo using FPGA prototypes.
- Placeholder (confirm with Ajit chip team): Ajit core power, operating temperature range.
- Placeholder with agent best estimate for low-power edge processor: power for normal flight-control and edge-AI workloads.
- Target spec: security (PUF root of trust, secure boot, isolated key storage, side-channel masking).
- Roadmap: small language models on drone; text-summary low-bandwidth ISR; Indian-fab SoC variant.
- Estimate tag used for agent-derived numbers (timings, power, costs, market sizing).
- Unsourced numbers softened to ranges or removed; no fabricated or unverifiable citations.

## Work order (replan)
- kind: end-user-interface
- Revise `investor-brief.md` first; Q&A, deck, speaker notes only after brief sign-off.

## Edge-AI workload and timing
- kind: end-user-interface
- Focus model: MobileNetV3-Large (≈0.22 G multiply-adds, ImageNet top-1 close to ResNet-50's ≈4.1 G); ResNet-50 kept as measured reference point.
- Timing basis: `docs/resnet50-metrics.md` C-model (≈81 instructions/MAC, CPI ≈1.13, reference TFLM kernels); scaled to 750 MHz.
- Speedups (user assumption): Ajit SIMD ≈5×; Apara ≈120× (observed 65× int8 matmul, 200× int8 dot product vs 1x1x32 Ajit, `docs/ajit.txt`).
- MobileNetV3 paper added to `reference/`.

## Ajit and chip targets
- kind: end-user-interface
- Ajit spec from `docs/ajit.txt`: SPARC-V8 + 64-bit vector extensions; 1–4 cores, 1–2 CPUs/core, all on FPGA; MMU, pipelined FPU, L1/L2, GDB debug; 2.46 DMIPS/MHz, 2.4 CoreMark/MHz, 0.6 MFLOPS/MHz; 1x1x32 in IITB NavIC SoC at 65 nm; ~100 mW at 16/28 nm.
- Chip clock target 750 MHz (IITB 28 nm projection, `docs/ajit.txt`); all timings at 750 MHz.
- Ajit IP: shared with partners under signed NDA per security requirements (replaces "IP licensing TBD").
- Operating temperature target −40 to +85 °C (matches Agam V6X-RT board); tape out to that spec.
- Power: agent reasonable estimate, tagged Estimate, replaced later.

## Partners
- kind: end-user-interface
- Agam Robotics (Bengaluru; Indian-IDDM Pixhawk FMUv6X-RT boards, PX4): autopilot (AP) board design, layout, assembly, bring-up; working process and NRE outline from `docs/agam-robotics.md` (agent-generated; ₹6–14 L NRE per board, excludes components). Website in `reference/`.
- Pixhawk compliance through Agam's design: target Pixhawk Autopilot Reference Standard (FMUv6X / v6C, Pixhawk Autopilot Bus) so Ajit module drops into PX4-compatible carriers; Payload Bus Standard relevant for Apara camera/gimbal payloads. Sources https://pixhawk.org/ and https://pixhawk.org/standards/ added to `reference/`.
- ideaForge (IIT Bombay origin): field testing of AP board on real drones.
- Agam AP board work in MVP 1 if feasible.

## Verified market study
- kind: end-user-interface
- Market metrics (India drone market, defence drone spend, EU drone / defence electronics, flight-controller segment) only from verifiable public sources in `reference/`.
- Each metric cites its source; conflicting sources shown as range with both cited.
- Draft claims without findable source are dropped or replaced; changes listed to user.

## Currency note
- kind: end-user-interface
- International readers: gentle footnote at first ₹ figure (brief, Q&A, deck).
- Indicative rates: ₹95 = US$1, ₹110 = €1, ₹130 = £1.

## Market positioning
- kind: end-user-interface
- India: defence first (Army / paramilitary tactical drones; trusted source, indigenisation, Mission Drone Shakti) plus dual-use commercial track (agri, inspection, enterprise) via Indian OEMs.
- Europe (year 2+): component supplier to OEMs — non-Chinese, non-US-export-controlled flight-control silicon, traceable supply chain; EDF SME calls; one EU integrator partner.
- Tone factual; no combat rhetoric (no "loitering munition", "self-destruct" emphasis); focus resilience, sovereignty, supply-chain security.
- No named OEM / customer relationships unless user supplies them.

## MVP definition
- kind: end-user-interface
- Each MVP: objective, concrete feasible TODOs, exit criteria, timeline window, funding scenario.
- MVP 1 (months 0–9) "It flies": Agam-designed Ajit flight-controller module (Pixhawk Autopilot Bus) on existing Ajit chip or FPGA; PX4 on NuttX; SITL/HITL on FPGA; NavIC-assisted position hold; tethered then free flight; first ideaForge trials; MobileNetV3 + SIMD on Ajit CPU; freeze SoC spec.
- MVP 2 (months 6–~20) "It sees, and it's ours": Apara on FPGA; 28 nm Ajit + Apara SoC tape-out ~M12, parts ~M16–18; module v2 with Agam; ideaForge field tests; MobileNetV3 on Apara (~4–5 frames/s); low-bandwidth alerts.
- MVP 3 (months 12–24, full only) "It survives": jamming/spoofing detection; GNSS-denied fallback (optical flow / inertial / radar); secure boot + hardware security target spec; DGCA/QCI type certification.
- One SoC tape-out in 24 months (full adds second spin / field-volume run).

## Document deliverables
- kind: end-user-interface
- Keep `docs/pitch-deck-2026/initial-deck.md` and `initial-deck-qna.md` unchanged.
- `docs/pitch-deck-2026/investor-brief.md`: one-page executive summary, then slide-order narrative (~2,500–3,500 words).
- `docs/pitch-deck-2026/investor-qna.md`: 30–40 investor Q&A grouped by theme, then technical appendix + glossary.
- Markdown tables, no ASCII art; placeholders as `[TBD: …]`; claim tags inline.
- Optional pandoc PDF export of both documents.
- Team and company name as placeholders; logos added later by user.

## Reference collection
- kind: internal-behavior
- `docs/pitch-deck-2026/reference/` holds downloaded public primary docs (policy, regulation, datasheets, funding calls, filings, Ajit papers, market reports).
- `reference/README.md` index: file, URL, claims supported, status (downloaded / link-only / not found).
- Paywalled or huge items link-only. HAL filings dropped.
- Index committed; downloaded files git-ignored.

## Slide deck
- kind: end-user-interface
- Beamer + TikZ/pgfplots, 16:9, professional (consulting / VC style), not default-Beamer look.
- Metropolis base with custom palette (navy primary, saffron/amber accent, light grey), Fira Sans via XeLaTeX, no nav symbols.
- 15–18 main slides + 5–8 appendix; one headline message per slide; key-number callouts.
- Visuals: architecture block diagram, 24-month MVP Gantt, cost split per scenario, market sizing, 2x2 competition, claim badges.
- Logo slots ready for later; English only; speaker notes in `deck/speaker-notes.md` (added at user request).
- Built last, after documents finalised.
