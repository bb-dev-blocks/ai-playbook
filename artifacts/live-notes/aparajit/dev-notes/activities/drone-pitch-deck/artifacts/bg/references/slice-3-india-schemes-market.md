# Slice 3: India schemes, Ajit, market sizing, cost benchmarks

Researched 2026-10-05. Files in `docs/pitch-deck-2026/reference/`. R83 was used for a link-only entry after its download failed (TDF FAQ returned 404 to curl).

## Index rows

| id | title | publisher/date | url | file | status | notes |
|---|---|---|---|---|---|---|
| R60 | Guidelines for Design Linked Incentive (DLI) Scheme | MeitY, 30.12.2022 | https://www.meity.gov.in/static/uploads/2024/02/Guidelines-for-Design-Linked-Incentive-DLI-Scheme-30.12.2022.pdf | R60-dli-scheme-guidelines-2022.pdf | downloaded | 16 pp; incentive caps on p10. Semicon 1.0 era; see R71 for the 2026 terms |
| R61 | Union Budget 2022-23 speech | Ministry of Finance, 01 Feb 2022 | https://www.indiabudget.gov.in/doc/bspeech/bs202223.pdf | R61-union-budget-2022-23-speech.pdf | downloaded | Kisan Drones para 33 (PDF p10); Drone Shakti para 45 (PDF p12) |
| R62 | Namo Drone Didi backgrounder | PIB, Nov 2024 | https://static.pib.gov.in/WriteReadData/specificdocs/documents/2024/nov/doc2024114428601.pdf | R62-pib-namo-drone-didi-backgrounder-2024.pdf | downloaded | 15,000 SHGs, 2024-25 to 2025-26 |
| R63 | Overview: Chips to Start-up (C2S) Programme | PIB, Jan 2026 | https://static.pib.gov.in/WriteReadData/specificdocs/documents/2026/jan/doc2026118760301.pdf | R63-pib-c2s-overview-2026.pdf | downloaded | ₹250 Cr, 85,000 professionals |
| R64 | AJITx1x1x32 Processor (datasheet) | IIT Bombay EE (M. Desai), 23 Feb 2023 | https://www.ee.iitb.ac.in/~madhav/1x1x32.kc705.pdf | R64-ajitx1x1x32-processor-iitb-2023.pdf | downloaded | SPARC-V8, 7-stage in-order pipeline, 32KB I/D caches, 256-entry TLB, FPU, FPGA KC705 prototype |
| R65 | An evaluation of a microprocessor with two independent hardware execution threads coupled through a shared cache | M. P. Desai, arXiv:2305.17773v1, 28 May 2023 | https://arxiv.org/abs/2305.17773 | R65-arxiv-2305.17773-ajit-dual-thread.pdf | downloaded | Dual-thread AJIT; FPGA benchmarks at 80 MHz; mentions a 65 nm GNSS SoC that uses AJIT |
| R66 | A Drone Strategy 2.0 for Europe, COM(2022) 652 | European Commission, 29 Nov 2022 | https://transport.ec.europa.eu/system/files/2022-11/COM_2022_652_drone_strategy_2.0.pdf | R66-eu-drone-strategy-2.0-com-2022-652.pdf | downloaded | €14.5 bn drone services by 2030 (para 16, p5) |
| R67 | Making India the drone hub of the world | EY & FICCI, Aug 2022 (PDF metadata) | https://www.ey.com/content/dam/ey-unified-site/ey-com/en-in/insights/government-public-sector/documents/ey-ficci-drones-report-final.pdf | R67-ey-ficci-drones-report-2022.pdf | downloaded | Manufacturing potential: US$4.2 bn by 2025, US$23 bn by 2030 (p5). Industry body plus consultancy, so advocacy-leaning |
| R68 | Media briefing: Semicon 2.0 / mobile manufacturing scheme | PIB/MeitY, 15 Jul 2026 | https://static.pib.gov.in/WriteReadData/specificdocs/documents/2026/jul/doc2026715921601.pdf | R68-pib-semicon-2.0-media-briefing-2026.pdf | downloaded | 16 MB slide deck; ₹1,27,500 Cr; six pillars |
| R69 | Lok Sabha Unstarred Q 3959: Chips to Startup Programme | MeitY, answered 12.08.2026 | https://sansad.in/getFile/lsapps/loksabhaquestions/annex/188/AU3959_kAN7c2.pdf | R69-loksabha-c2s-answer-AU3959.pdf | downloaded | >68,000 trained; 254 tape-outs (175 at SCL 180 nm) |
| R70 | Annual salary trends report 2024-25 | Randstad India, 2024 | https://info.randstad.in/hubfs/Thought%20leadership%20reports/Annual%20salary%20trends%20report%202024-%2025.pdf | R70-randstad-salary-trends-2024-25.pdf | downloaded | Function averages (IT-Tech) on PDF p26; no VLSI- or embedded-specific rows |
| R71 | Semicon 2.0: Scheme for development of Semiconductor Design and Manufacturing Ecosystem (Gazette) | MeitY, Gazette of India Extraordinary, 31 Aug 2026 | https://egazette.gov.in/WriteReadData/2026/275860.pdf | R71-gazette-semicon-2.0-scheme-2026.pdf | downloaded | Bilingual, 18 pp. P-DLI on p13, DLI 9% on p14. Supersedes the R60 terms for new applications (assumption; see Gaps) |
| R72 | TDF funding enhanced to ₹50 Cr per project from ₹10 Cr | PIB/MoD, 08 Jun 2022 | https://www.pib.gov.in/PressReleasePage.aspx?PRID=1832042 | R72-pib-tdf-enhanced-50cr-2022.html | downloaded | Raised in **2022**, not 2024 |
| R73 | RM launches ADITI scheme | PIB/MoD, 04 Mar 2024 | https://www.pib.gov.in/PressReleasePage.aspx?PRID=2011171 | R73-pib-aditi-launch-2024.html | downloaded | ADITI up to ₹25 Cr; iDEX Prime raised assistance from ₹1.5 Cr to ₹10 Cr |
| R74 | Cabinet approves Programme for Development of Semiconductors and Display Manufacturing Ecosystem | PIB, 15 Dec 2021 | https://www.pib.gov.in/PressReleasePage.aspx?PRID=1781723 | R74-pib-semicon-programme-76000cr-2021.html | downloaded | ₹76,000 Cr; ISM set up |
| R75 | PLI scheme for drones: press conference | PIB/MoCA, 16 Sep 2021 | https://www.pib.gov.in/PressReleasePage.aspx?PRID=1755452 | R75-pib-drone-pli-2021.html | downloaded | PLI ₹120 Cr; manufacturing turnover ₹60 Cr (2020-21) to >₹900 Cr (FY24); "global drone hub by 2030" |
| R76 | Namo Drone Didi operational guidelines | PIB, 01 Nov 2024 | https://www.pib.gov.in/PressReleasePage.aspx?PRID=2070029 | R76-pib-namo-drone-didi-guidelines-2024.html | downloaded | 14,500 SHGs, 2024-25 to 2025-26 (the other 500 supplied by fertilizer companies) |
| R77 | Cabinet approves Semicon 2.0 | PIB, 15 Jul 2026 | https://www.pib.gov.in/PressReleasePage.aspx?PRID=2284784 | R77-pib-semicon-2.0-cabinet-2026.html | downloaded | ₹1,27,500 Cr |
| R78 | C2S Programme press note | PIB/MeitY, 18 Jan 2026 | https://www.pib.gov.in/PressReleasePage.aspx?PRID=2215755 | R78-pib-c2s-2026.html | downloaded | Launched 2022, ₹250 Cr over 5 yrs, 85,000 professionals |
| R79 | MoD concludes emergency procurement contracts worth nearly ₹2,000 Cr | PIB/MoD, 24 Jun 2025 | https://www.pib.gov.in/PressReleasePage.aspx?PRID=2139102 | R79-pib-emergency-procurement-1981cr-2025.html | downloaded | 13 contracts, ₹1,981.90 Cr; includes RPAVs, loitering munitions, drones, counter-drone systems |
| R80 | Measures taken for the drone sector (Parliament reply) | PIB/MoCA, 14 Dec 2023 | https://www.pib.gov.in/PressReleasePage.aspx?PRID=1986162 | R80-pib-drone-sector-measures-2023.html | downloaded | "Mission 'Drone Shakti'" announced in Budget, 01 Feb 2022 |
| R81 | Welcome AJIT, a 'Made in India' Microprocessor | IIT Bombay R&D, ~May 2019 (Hindu coverage dated 09 May 2019) | https://rnd.iitb.ac.in/node/1137 | R81-iitb-ajit-press-2019.html | downloaded | 70-120 MHz, 1 instruction per clock, 180 nm at SCL, ₹100 target price |
| R82 | iDEX FAQ | iDEX/DIO (MoD), accessed Oct 2026 | https://www.idex.gov.in/faq | R82-idex-faq.html | downloaded | DISC/OC up to ₹1.5 Cr; ADITI up to ₹25 Cr; financial FAQ: "₹1.5 Cr/10 Cr/25 Cr or 50% of PDB whichever lower" |
| R83 | Drone sector revenue Rs 60 Cr to Rs 4,000 Cr (DFI president Smit Shah) | NewKerala (wire), 25 Sep 2026 | https://www.newkerala.com/news/a/rs-60-cr-rs-4000-cr-revenue-drone-365.htm | — | link-only | Industry-body quote, not audited; the article also quotes "₹3,000 to 4,000 crore" |
| R84 | DLI Scheme FAQ | chips-dli.gov.in (C-DAC), accessed Oct 2026 | https://www.chips-dli.gov.in/DLI/Faq | R84-chips-dli-faq.html | downloaded | Same 50%/₹15 Cr and 6-4%/₹30 Cr as R60 (portal not yet updated to Semicon 2.0?) |
| R85 | ADITI details | DDP, MoD, accessed Oct 2026 | https://www.ddpmod.gov.in/en/offerings/schemes-and-services/aditi_details | R85-ddp-aditi-details.html | downloaded | Outlay ₹750 Cr (2023-24 to 2025-26); up to 50% of PDB, max ₹25 Cr |
| R86 | Semicon 2.0 notifications issued (media address) | PIB/MeitY, 31 Aug 2026 | https://www.pib.gov.in/PressReleasePage.aspx?PRID=2304963 | R86-pib-semicon-2.0-notification-2026.html | downloaded | Applications open 3 yrs; projects up to 6 yrs; ISM is nodal agency, with C-DAC for design categories |
| R87 | Namo Drone Didi status (Parliament reply) | PIB, 17 Dec 2024 | https://www.pib.gov.in/PressReleasePage.aspx?PRID=2085177 | R87-pib-namo-drone-didi-status-2024.html | downloaded | First 500 drones 2023-24; 3,090 SHGs targeted in FY25 phase 1 |
| R88 | Cabinet approves Central Sector Scheme for providing drones to women SHGs | PIB, 29 Nov 2023 | https://www.pib.gov.in/PressReleasePage.aspx?PRID=1980683 | R88-pib-namo-drone-didi-cabinet-2023.html | downloaded | ₹1,261 Cr; 15,000 SHGs; 80% CFA up to ₹8 lakh |
| R89 | Distribution of agricultural drones under Namo Drone Didi | PIB, 13 Feb 2026 | https://www.pib.gov.in/PressReleasePage.aspx?PRID=2227743 | R89-pib-namo-drone-didi-distribution-2026.html | downloaded | As of Jan 2026: 1,094 drones distributed by fertilizer companies, of which 500 under the scheme |
| R90 | Mission Drone Shakti: India plans ₹1,800 crore scheme | Mint, 17-18 Mar 2026 | https://www.livemint.com/industry/mission-drone-shakti-drones-rs-1800-crore-scheme-local-manufacturing-china-imports-drone-components-sales-11773714939259.html | — | link-only | **Proposed** ₹1,600-1,800 Cr over 5 yrs (₹500-600 Cr R&D); flight controllers named as an imported component; awaiting Finance Ministry and Cabinet approval |
| R91 | Drone industry turnover up to ₹15,000 Cr by 2026: Scindia | The Hindu (PTI), 16 Sep 2021 | https://www.thehindu.com/news/national/indian-drone-industry-will-have-a-turnover-of-up-to-15000-crore-by-2026-jyotiraditya-scindia/article36494392.ece | — | link-only | "$1.8 billion ... approximately ₹12,000-₹15,000 crore" (hardware, software and services combined). Not in PIB R75 |
| R92 | India-US MQ-9B contract coverage | The Hindu 15 Oct 2024 ($3.5 bn); Janes 15 Oct 2024 (citing DD News ₹32,000 Cr / US$3.8 bn); ET (₹34,500 Cr incl. MRO) | https://www.thehindu.com/news/national/india-to-procure-31-predator-long-endurance-drones-from-us/article68755738.ece | — | link-only | MoD did **not** officially disclose the value (BusinessLine). PIB year-end review confirms the contract only: https://www.pib.gov.in/PressReleasePage.aspx?PRID=2088180 |
| R93 | Post-Op Sindoor Army drone orders | TOI ~Jun 2025 (₹295 Cr: Solar ₹158 Cr / ideaForge ₹137 Cr); ET Dec 2025 (>₹5,000 Cr indigenous drones); ET/ANI May 2025 (~₹40,000 Cr EP powers) | https://economictimes.indiatimes.com/news/defence/army-places-orders-worth-rs-5000-crore-for-indigenous-drones-after-op-sindoor-trials/articleshow/125965350.cms | — | link-only | Press, source-attributed ("sources said"); screening for Chinese-origin parts was mandatory |
| R94 | Drone Market Size and Growth 2025-2030 | Drone Industry Insights (DII), 2025 | https://droneii.com/project/drone-market-size-and-growth-2025-2030 | — | link-only | Full report paywalled; free summary only. Commercial and recreational, **excludes military** |
| R95 | Military UAS sector study (2024/2025 forecast); 2023/24 press release | Teal Group (Zaloga) | https://www.tealgroup.com/index.php/teal-group-media-news-briefs-2/teal-group-news-media/item/military-uas-sector-study-released | — | link-only | Paywalled report; free summary. PR Newswire 03 Jan 2024: https://www.prnewswire.com/news-releases/teal-group-predicts-worldwide-military-uas-spending-of-258-7-billion-over-the-next-decade-302025631.html |
| R96 | Military drone market and India drone market press releases | MarketsandMarkets (PR Newswire), 27 May 2025 and 23 Aug 2025 | https://www.prnewswire.com/news-releases/military-drone-market-worth-22-81-billion-in-2030---exclusive-report-by-marketsandmarkets-302465561.html | — | link-only | **Publisher bias**: paid market-research marketing. India release: https://www.prnewswire.com/news-releases/india-drone-uav-market-worth-1-39-billion-by-2030---exclusive-report-by-marketsandmarkets-302536553.html |
| R97 | Drone flight controller system market press releases | TBRC/Research and Markets via EIN Presswire 11 Jun 2026; KBV Research 15 Feb 2024; Strategic Market Research Dec 2025 | https://www.giiresearch.com/report/tbrc1991267-drone-flight-controller-system-global-market.html | — | link-only | **Publisher bias, low reliability**: estimates range US$10.2-14.7 bn by 2030 with inconsistent scope definitions |
| R98 | Crowd-sourced VLSI salary data | AmbitionBox (updated 12 May 2025); Glassdoor Chennai (Sep 2026); semiconductor.cv blog (2026) | https://www.ambitionbox.com/profile/vlsi-design-engineer-salary | — | link-only | **Low reliability** (self-reported, small samples). Glassdoor: https://www.glassdoor.com/Salaries/chennai-india-vlsi-design-engineer-salary-SRCH_IL.0,13_IM1067_KO14,34.htm |
| R99 | EDA licence cost indications | EDAboard forum (undated user posts); Vendr marketplace (2026); chipinsights.net (OpenROAD) | https://www.vendr.com/marketplace/synopsys | — | link-only | **No official list prices exist.** All figures are anecdotal or aggregator estimates. Forum: https://www.edaboard.com/threads/what-are-the-typical-costs-of-eda-tools.98833/ ; OpenROAD: https://chipinsights.net/p/eda-deep-dive-part-2-open-source |
| — | adhuliya/ajit-toolchain | GitHub | https://github.com/adhuliya/ajit-toolchain | — | link-only | Exists (HTTP 200); "Toolchains for AJIT Processor IIT Bombay" |
| — | TDF FAQ | DRDO TDF portal | https://tdf.drdo.gov.in/public/faq | — | exists-but-claims-differ | Search snippet shows the FAQ still says cost "will not exceed INR 10 crore", contradicting R72 (₹50 Cr). curl returned 404 |
| — | "Mission Drone Shakti 2.0" | — | — | — | not found | Only blogs (blitzindiabusiness, insidefpv). No government source |

## Draft citation verdicts

- **DLI "50% of eligible expenditure, cap ₹15 Cr; deployment 6-4% of net sales, cap ₹30 Cr"**: verified exactly for the 2022 DLI guidelines (R60 p10, R84). **Superseded for the Semicon 2.0 era** (R71 gazette, 31 Aug 2026):
  - Product design support (P-DLI), startups and MSMEs: "Milestone-linked advance Seed funding amounting to 50% of the project cost or ₹15 crore, whichever is lower", plus equity co-investment beyond ₹15 Cr for VC/PE-backed firms (p13).
  - Deployment incentive (DLI): "Reimbursement at the rate of 9% on net sales of target segments for a period of five (5) years, subject to a maximum incentive ceiling of ₹30 crore per application and an aggregate ceiling of ₹120 crore per company" (p14).
  - The deck should cite R71, not R60.
- **C2S "85,000 engineers, ~₹250 Cr"**: verified (R78, R63). ₹250 Cr over 5 yrs, launched 2022. Progress: >68,000 trained, 254 tape-outs (R69, Aug 2026).
- **ISM ₹76,000 Cr**: verified (R74, 15 Dec 2021). Add: Semicon 2.0 at ₹1,27,500 Cr approved 15 Jul 2026 (R77) and notified 31 Aug 2026 (R86, R71).
- **iDEX ₹1.5 Cr / iDEX Prime ₹10 Cr / ADITI ₹25 Cr**: verified (R82, R73, R85). Grants are capped at 50% of the product development budget (R82 financial FAQ, R85).
- **TDF "raised from ₹10 Cr to ₹50 Cr in 2024"**: amount verified, **date wrong**: raised 08 Jun 2022 (R72). Covers up to 90% of project cost. The TDF portal FAQ still says ₹10 Cr (exists-but-claims-differ).
- **Drone Shakti / Kisan Drones (Budget 2022-23)**: verified (R61 paras 33 and 45).
- **"Mission Drone Shakti 2.0"**: not found in any official source. A separate **proposed** "Mission Drone Shakti" (₹1,600-1,800 Cr, MoCA, FY27) is reported by Mint (R90). No Cabinet approval or PIB release found as of Oct 2026. Present it as "proposed, per press".
- **Namo Drone Didi "15,000 drones, ₹1,261 Cr, 2023"**: verified (R88, Cabinet 29 Nov 2023).
  - PIB itself gives the scheme period inconsistently: "2024-25 to 2025-26" (R88) vs "2023-24 to 2025-26" (R87, R89).
  - Execution gap: only 500 drones under the scheme as of Jan 2026 (R89).
- **Ajit "India's first indigenous microprocessor"**: this is IIT Bombay's own claim: "first ever microprocessor to be conceptualised, designed, developed and manufactured in India" (R81). Attribute it to IITB; it is not an independent finding.
- **Ajit "~2018"**: the public announcement is ~May 2019 (R81; The Hindu 09 May 2019). Work started 2015. Fabrication date is not stated.
- **Ajit "70-120 MHz"**: verified as IITB's statement (R81). No measured silicon clock or power was published.
- **Ajit "65 nm / multi-core"**: no 65 nm Ajit standalone chip found. R65 says single-thread AJIT "has been used in the implementation of a GNSS base-band receiver system-on-chip (in 65nm technology)". The cited preprint (Desai & Pandey 2023) was not found publicly. The dual-thread AJIT was evaluated on FPGA only (R65). No multi-core silicon found.
- **"Drone industry ₹12,000-15,000 Cr by 2026"**: this is a minister's projection (Scindia, 16 Sep 2021, R91), not a measurement. **"₹1,800 Cr manufacturing" is a misreading**: the source says **US$1.8 bn** total (= ₹12-15k Cr). The manufacturing target was >₹900 Cr by FY24 (R75).
- **"Global drone hub by 2030"**: verified as a MoCA objective (R75).
- **"MQ-9B $3.5-4 bn (2024)"**: contract signed 15 Oct 2024 for 31 drones (PIB year-end review). The value comes only from press (₹32,000-34,500 Cr, $3.5-4 bn); MoD did not disclose it.

## Verified metrics

| metric | value | source id | exact quote or page | confidence |
|---|---|---|---|---|
| DLI 2022 product incentive | up to 50% of eligible expenditure, ≤₹15 Cr/application | R60 | p10: "Milestones linked reimbursement of up to 50% of the eligible expenditure subject to a ceiling of ₹15 Crore per application" | high |
| DLI 2022 deployment incentive | 6% (Y1-2), 5% (Y3-4), 4% (Y5) of net sales, ≤₹30 Cr | R60 | p10: "Reimbursement of 6% to 4% of net sales ... over 5 years subject to a ceiling of ₹30 Crore" | high |
| DLI 2022 net-sales threshold | ₹1 Cr/yr (startup/MSME); ₹5 Cr/yr (others) | R60 | §3.1 table | high |
| DLI 2022 manpower cap | manpower ≤50% of total incentive approved | R60 | §2.5.1 | high |
| Semicon 2.0 P-DLI | seed funding = min(50% of project cost, ₹15 Cr) per application (startups/MSMEs) | R71 | p13 | high |
| Semicon 2.0 DLI | 9% of net sales × 5 yrs; ≤₹30 Cr/application; ≤₹120 Cr/company | R71 | p14 §3.3.4 | high |
| Semicon 2.0 outlay | ₹1,27,500 Cr | R77 | "total budget outlay of Rs.1,27,500 crore" (15 Jul 2026) | high |
| Semicon India 1.0 outlay | ₹76,000 Cr (>US$10 bn) | R74 | "Rs.76000 crore (>10 billion USD)" | high |
| C2S outlay / target | ₹250 Cr over 5 yrs; 85,000 professionals | R78 | "total outlay of ₹ 250 crore over five years"; "85,000 industry-ready professionals" | high |
| C2S progress | >68,000 trained; 254 tape-outs (175 at SCL 180 nm, 79 overseas); EDA access at 332 institutions | R69 | answer (i)-(iii), 12.08.2026 | high |
| ISM design ecosystem stats | 105 startups given EDA tools; 24 funded; 15 VC-funded | R68 | slide "Talent development" | medium (slide deck) |
| iDEX grant | up to ₹1.5 Cr (DISC/OC) | R82 | "Grant support of up to ₹1.5 crore is provided to DISC winners" | high |
| iDEX Prime | up to ₹10 Cr | R73 | "assistance increasing from Rs 1.5 crore to Rs 10 crore" | high |
| ADITI | up to ₹25 Cr, ≤50% of PDB; outlay ₹750 Cr (2023-24 to 2025-26) | R85, R73 | "grants of up to 50 per cent of the Product Development Budget (PDB) with a maximum limit of Rs 25 crore" | high |
| iDEX scheme budget | ₹498.78 Cr, 2021-22 to 2025-26 | DDP iDEX page (link-only: https://www.ddpmod.gov.in/en/offerings/schemes-and-services/idex) | "budgetary support of Rs. 498.78 crore for a period of 5 years" | high |
| TDF cap | ₹50 Cr/project (from ₹10 Cr); up to 90% of project cost | R72 | 08 Jun 2022 | high |
| Drone Shakti | startups for drone applications and DrAAS; ITI skilling courses | R61 | para 45, PDF p12 | high |
| Namo Drone Didi | ₹1,261 Cr; 15,000 SHGs; 80% CFA ≤₹8 lakh | R88 | 29 Nov 2023 | high |
| Namo Drone Didi delivery | 500 drones under the scheme (1,094 total by fertilizer companies) as of Jan 2026 | R89 | 13 Feb 2026 | high |
| Drone PLI | ₹120 Cr over 3 FYs; 20% of value addition | R75 | 16 Sep 2021 | high |
| Drone manufacturing turnover target | ₹60 Cr (2020-21) to >₹900 Cr (FY24) | R75 | "may grow from INR 60 crore in 2020-21 ... to over INR 900 crore in FY 2023-24" | high (as a target) |
| Drone industry projection | US$1.8 bn ≈ ₹12,000-15,000 Cr by 2026 | R91 | Scindia quote (PTI) | medium (projection, press) |
| DFI-reported sector revenue | ~₹4,000 Cr in 2025 (from ₹60 Cr in 2021); "₹19,000-20,000 Cr in coming years" | R83 | Smit Shah quote | low-medium (industry body, unaudited) |
| EY-FICCI manufacturing potential | US$4.2 bn (2025), US$23 bn (2030) | R67 | p5 | low-medium (advocacy estimate) |
| MQ-9B | 31 drones (15 Navy, 8 IAF, 8 Army); signed 15 Oct 2024; ~$3.5-4 bn / ₹32,000-34,500 Cr | R92 | value per press; MoD did not disclose it | high (contract) / medium (value) |
| Post-Sindoor EP (Army) | 13 contracts, ₹1,981.90 Cr of ₹2,000 Cr sanctioned (incl. drones, loitering munitions, counter-drone systems) | R79 | 24 Jun 2025 | high |
| Army EP drone contracts | ₹295 Cr (450 Nagastra-1R ₹158 Cr; ideaForge mini-UAV ₹137 Cr) | R93 | TOI | medium (press) |
| Army indigenous drone orders | >₹5,000 Cr being placed (Dec 2025); Chinese-parts screening mandatory | R93 | ET | medium (press, "sources") |
| Global drone market (commercial + recreational) | US$40.6 bn (2025) to US$57.8 bn (2030), 7.3% CAGR; hardware US$6.7 bn (2025) | R94 | DII free summary | medium (specialist firm; excludes military) |
| EU drone services market | €14.5 bn by 2030, 12.3% CAGR, 145,000 jobs | R66 | para 16, p5 | high (EC figure; services only) |
| Global military UAS spend | ~US$14.9 bn (FY25) to ~US$28.6 bn (FY34), RDT&E plus procurement, 7.5% CAGR | R95 | Teal Group summary | medium (paywalled source; excludes O&M) |
| Global military drone market | US$15.80 bn (2025) to US$22.81 bn (2030) | R96 | MarketsandMarkets PR | low (publisher bias) |
| India drone market (MnM) | US$0.47 bn (2025) to US$1.39 bn (2030) | R96 | MarketsandMarkets PR | low (publisher bias) |
| Drone flight-controller market | US$10.2-14.7 bn by 2030 (varies by publisher) | R97 | TBRC: $8.07 bn (2025) to $14.7 bn (2030); KBV: $10.2 bn | low (publisher bias, scope inconsistent) |
| Ajit clock | 70-120 MHz (IITB statement) | R81 | "can operate at clock speeds between 70-120MHz" | medium (claim; no published silicon data) |
| Ajit process | 180 nm, SCL Chandigarh/Mohali | R81 | "manufactured in ... SCL, Chandigarh ... 180 nanometers" | high |
| Ajit ISA / microarchitecture | SPARC-V8; single-issue, in-order, 7-stage; FPU; 32KB 4-way I/D caches; SPARC ref MMU with 256-entry TLB | R64 | p1 | high |
| Ajit FPGA benchmarks | 2.4 DMIPS/MHz; 2.42 CoreMark/MHz; 0.27 MFLOPS/MHz (Linpack); 0.4 MWIPS/MHz; at 80 MHz on FPGA | R65 | Fig. 4, §3.3 | high (FPGA, not silicon) |
| Ajit software | boots Linux 3.16.1 | R65 | §3 | high |
| Ajit target price | "as less as ₹100" at volume (~10 lakh units) | R81 | claim | low (aspirational) |
| Ajit funding/partners | MeitY and IIT Bombay; Powai Labs as industry partner; SAMEER planned NavIC receiver use | R81 | press text | high |
| Ajit power | not found | — | — | — |

## Cost benchmarks

| item | value | source id | note |
|---|---|---|---|
| VLSI design engineer, India (all levels <5 yrs) | ₹2.8-40 LPA | R98 (AmbitionBox, 335 salaries, May 2025) | crowd-sourced, very wide range; low reliability |
| VLSI design engineer, Chennai (Glassdoor) | median ₹6.79 LPA; 25th-75th percentile ₹5.6-8.6 LPA | R98 | n=6 salaries; very low reliability |
| VLSI by level (blog aggregate) | fresher ₹5-12 LPA; mid (3-5 yrs) ₹12-25; senior (6-10 yrs) ₹25-50; MNC freshers ₹14-30 LPA | R98 (semiconductor.cv 2026) | blog aggregating Glassdoor/AmbitionBox/Naukri; low reliability |
| IT-Tech function average CTC (proxy only) | junior (0-5 yrs) ₹7.85 L; middle (6-14 yrs) ₹18.35 L; senior (≥15 yrs) ₹31.26 L | R70 p26 | Randstad survey; not VLSI-specific; medium reliability |
| Bengaluru average CTC, all functions | junior ₹7.22 L; middle ₹19.04 L; senior ₹36.46 L | R70 p13 | Randstad 2024-25 |
| Commercial EDA (Synopsys/Cadence), mid-size team | ~US$300k-1M+/yr; startup discounts "50-70%" | R99 (Vendr) | aggregator estimate; not vendor-published; low reliability |
| Single advanced P&R seat | "probably about $750,000" per year (SoC Encounter GXL) | R99 (EDAboard forum) | anecdotal, old, undated; flag as anecdote only |
| Vendor licensing model | mostly 2-3 yr time-based licences | R99 (Open iT, citing Cadence/Synopsys filings) | qualitative |
| Gov-provided EDA | free national EDA tool grid via C2S (332 institutions) and DLI/Semicon 2.0 design infrastructure support (startups) | R69, R60 §6.1.1, R71 p13 | main cost lever for an Indian startup |
| Open-source flow | OpenROAD (DARPA, 2018) / OpenLane: RTL-to-GDS; used in real tape-outs; advanced proprietary PDKs need certified commercial tools | R99 (chipinsights) | secondary source; mature-node and open-PDK only |
| SCL 180 nm MPW via ChipIN | free to C2S participating institutions; ~9 months from GDS to packaged parts (Feb to Dec 2025 shuttle) | https://c2s.gov.in/SCL_MPW2.jsp (link-only) | academia only; startup access via DLI design infrastructure support |

## Gaps

- It is unclear whether 2022 DLI (R60) terms still apply to pending or new applications after the Semicon 2.0 notification. A third-party summary says detailed Semicon 2.0 guidelines "are to follow separately"; this was not verified in the gazette text. The chips-dli portal (R84) still shows the 2022 terms.
- Mission Drone Shakti: no Cabinet approval or PIB release found as of Oct 2026. Final outlay and flight-controller eligibility are unknown.
- Ajit: no published silicon power, measured clock, die area, or temperature grade. No public 65 nm Ajit or multi-core silicon. The Desai & Pandey 65 nm GNSS SoC preprint was not found. Exact fabrication year not public.
- TDF FAQ portal (₹10 Cr) contradicts the PIB ₹50 Cr figure. Could not fetch the page (404 to curl); a TDF guidelines PDF was not located.
- No public iDEX or TDF guideline PDFs downloaded (FAQ pages saved instead).
- MQ-9B value: no official MoD figure.
- Totals for post-Sindoor drone procurement beyond the PIB ₹1,981.90 Cr EP are press-only. The ~₹40,000 Cr EP powers and >₹5,000 Cr drone orders are source-attributed press.
- Indian defence drone market size: no government figure found.
- European military drone market: no free EU or EDA figure found. Teal Group calls Europe the "second largest market" (qualitative).
- Flight-controller and drone-avionics segment: only biased market-research press releases. No credible independent figure.
- Salary data: no Michael Page, NASSCOM, or Randstad VLSI/embedded-specific figures found (Michael Page guide only on Scribd, blocked).
- EDA costs: no official or credible public price. All figures anecdotal.
