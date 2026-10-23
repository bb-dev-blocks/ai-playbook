# Design Decisions

## Seed VC audience with appendix depth
- level: design-choice
- chosen: write for generalist Indian seed investors; technical depth moved to Q&A appendix.
- why: user target is "average investor"; drafts too dense and defence-jargon heavy.
- alternatives:
  - grant-body audience (iDEX/TDF): narrower; kept as one secondary section.
  - strategic OEM audience: no named OEM relationships yet.

## Two funding scenarios
- level: design-choice
- chosen: lean ~₹12 Cr (MVP 1 + MVP 2, MPW tape-out on mature node) and full ~₹30 Cr (MVP 1–3, dedicated tape-out with Apara, more boards, certification); both keep ~30/70 software/chip split.
- why: user range ₹12–30 Cr; scenarios show what each amount buys.
- alternatives:
  - single number: hides trade-off between amount and scope.

## Claim tagging
- level: design-choice
- chosen: tags Demonstrated / Target spec / Roadmap / Placeholder; placeholders as `[TBD: …]`.
- why: drafts mixed unverified numbers with facts; investors punish one bad claim; greppable placeholders ease user fill-in.
- alternatives:
  - keep draft numbers: unsourced power, thermal, SLM claims risk credibility.

## MVP 1 on existing Ajit silicon
- level: design-choice
- chosen: MVP 1 builds drone carrier board around existing Ajit chips; FPGA prototypes carry HITL; new SoC tape-out in MVP 2.
- why: Ajit chips and FPGA prototypes exist; flights possible before new silicon (month 12–15).
- alternatives:
  - wait for new SoC: no flight evidence for >1 year.

## Defence-first plus dual-use positioning
- level: design-choice
- chosen: India defence first with commercial dual-use track; Europe as component supplier to OEMs via EDF.
- why: broadens investor pool; earlier revenue; avoids pure-defence procurement dependence.
- alternatives:
  - defence-only (drafts): narrows investors, slow procurement.

## Document layout
- level: design-choice
- chosen: keep drafts; new `investor-brief.md` (slide-order narrative) and `investor-qna.md` (Q&A + appendix); optional pandoc PDF.
- why: history preserved; brief mirrors deck so slides derive from it.
- alternatives:
  - edit drafts in place: loses comparison baseline.

## Reference folder git policy
- level: implementation-detail
- chosen: commit `reference/README.md` index; git-ignore downloaded files.
- why: avoid tens of MB in repo; URLs allow re-download.
- alternatives:
  - commit all PDFs: repo bloat.

## Deck theme
- level: design-choice
- chosen: Metropolis base, custom navy/saffron/grey palette, Fira Sans, XeLaTeX, 16:9, TikZ/pgfplots visuals, local theme file.
- why: tooling installed; Metropolis flat base easy to restyle into VC look.
- alternatives:
  - default Beamer themes: "nerdy" look user rejects.
  - from-scratch theme: more effort, no gain.

## Tape-out nodes per scenario
- level: design-choice
- chosen: flight-control SoC on SCL Mohali 180 nm MPW (both scenarios); full scenario adds Apara AI co-processor on a 65 nm MPW (Europractice UMC 65 nm benchmark €43,750 per 4×4 mm) plus engineering lot of flight SoC.
- why: draft DAP 2026 makes semiconductors from Indian fabs "projects of National importance" with mandatory procurement [R05]; Ajit already proven at SCL 180 nm [R81]; C2S supports SCL MPW [R69]. Edge AI needs denser node.
- alternatives:
  - dedicated full-mask tape-out: no public price; unneeded for field-test volumes.
  - foreign 65 nm for flight SoC: loses Indian-fab procurement hook.
- replaces: Two funding scenarios

## Europe entry via EU entity or partner
- level: design-choice
- chosen: Europe year 2+ through commercial component sales to EU OEMs and an EU-based partner or subsidiary for EDF 2027+ calls.
- why: EDF 2026 topics closed 29 Sep 2026; Indian parent ineligible as EDF recipient; EU subsidiary needs Member-State guarantees [R34, R47].
- alternatives:
  - direct EDF 2026 application (drafts): closed and ineligible.

## Funding totals by agent judgement
- level: design-choice
- chosen: lean ₹16 Cr (MVP 1 + 2, ~22 months), full ₹36 Cr (MVP 1–3, 24 months); 30/70 software/chip split applies to engineering only; separate operations and go-to-market bucket (11–13%); ~10% contingency on chip and software (12% chip in full); FPGA boards and 5 DGX Spark (~₹50 L) included.
- why: user freed totals; first draft lacked leadership/BD/legal costs, compute, and realistic first-silicon contingency.
- alternatives:
  - keep ₹12 / ₹30 Cr: forces cutting contingency or field-test controllers.
  - fold operations into 30/70: hides go-to-market spend investors expect to see.

## Indicative currency rates
- level: implementation-detail
- chosen: ₹95/US$, ₹110/€, ₹130/£ in footnote at first ₹ figure.
- why: user-specified; international audience.
- alternatives:
  - live rates: churn; not needed for initial take.

## Single 28 nm-class SoC at 800 MHz
- level: design-choice
- replaces: Tape-out nodes per scenario
- choice: one SoC (dual-core Ajit + Apara, `APARAJIT Micro` shape from `docs/ajit.txt`) on 28 nm-class shared-wafer run abroad; 800 MHz target, −40 to +85 °C.
- why: 800 MHz impossible at SCL 180 nm; `docs/ajit.txt` projects 750 MHz at 28 nm, 1.2 GHz at 16 nm; one tape-out cheaper than two.
- alternatives: flight chip SCL 180 nm + Apara 28 nm (keeps Indian-fab DAP hook, two tape-outs); single 65 nm SoC (800 MHz unlikely).
- consequence: pitch = Indian design and IP, foreign fab now; Indian-fab variant later; draft-DAP Indian-fab clause no longer a headline. 800 MHz above 28 nm projection → Target spec. Benchmark: Europractice GF 28 nm SLPe €12,430/mm² [R40]; TSMC 28 nm on request.

## Apara before EW
- level: design-choice
- replaces: Two funding scenarios
- choice: MVP 1 It flies (M0–9), MVP 2 It sees and it's ours (Apara SoC, M6–~20), MVP 3 It survives (EW, security, certification, M12–24). Lean = MVP 1 + 2; full adds MVP 3, field-volume run, EU groundwork.
- why: user priority; one SoC carries Apara, so tape-out and vision land together.
- consequence: lean has no anti-jamming capability; EW pitched as funded only in full.

## AP board before Apara tape-out
- level: design-choice
- choice: Agam designs Ajit flight-controller module for Pixhawk Autopilot Bus baseboard; v1 on existing Ajit chip/FPGA in MVP 1, v2 on new SoC in MVP 2.
- why: flights and ideaForge trials do not wait for silicon; module respin cheap (₹6–14 L NRE per board, `docs/agam-robotics.md`); baseboard reused.
- alternatives: design board only after SoC returns (one design, but no flights until ~M18).

## Lean AI-assisted teams
- level: major-implementation-detail
- choice: 5 chip design/verification + 3 software in both scenarios; full adds 2 software (EW/security) M12–24; board design outsourced to Agam (≈₹0.5 Cr).
- why: user direction; AI-assisted engineering lowers headcount; Apara already on FPGA so team integrates rather than designs.

## 750 MHz clock target
- level: design-choice
- replaces: Single 28 nm-class SoC at 800 MHz
- choice: same single 28 nm-class Ajit + Apara SoC, clock target 750 MHz (IITB 28 nm projection); timings at 750 MHz.
- why: user; matches published projection, so no longer above it. Still Target spec until silicon.
