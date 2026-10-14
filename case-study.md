# Case Study: Shade Correction for Digitally Printed Tiles

**Prepared by:** Forward Deployed Engineer, Banao Technologies
**Status:** Proposal after first discovery call. Confidential.

**In one paragraph.** The client already measures how far a tile's shade is from the master (ΔL, Δa, Δb, Δgloss). What nobody can tell them is *how much to change the printer's input file*. We propose a measure, calculate and human-approve tool that fills that gap, piloted at one India plant over about 11 weeks. Phase 1 should cut a correction from about 6.5 hours to about 2 to 3 hours. It will not reach "5 to 6 minutes". That needs an optional Phase 2, which we will only recommend if a test supports it.

---

## 1. Problem understanding

### 1.1 What they asked vs. what they need

| | Asked | Actually needed |
|---|---|---|
| System | "Video / vision analytics" | A **measure + calculate** tool. They own a spectrophotometer, so no camera is needed. |
| Output | Compare master and current tile | Tell QC **how much to change the print file** (the missing step) |
| Where it runs | Implied: on the line | **Only at batch change.** The client said that if the first sample of a batch is right, the whole batch is fine. |
| Speed | "5 to 6 minutes" | Fewer rounds and a faster fix. The kiln cannot be skipped in Phase 1. |
| Consistency | Not asked | Numbers replace eyesight, so different QC people reach the same fix |

### 1.2 Stakeholders

| Who | Status | What they want |
|---|---|---|
| Head of IT & OT Procurement | On the call | A safe, deployable purchase; confidence on hardware and OEM partners |
| Special Projects Lead, Procurement | On the call | Less wasted production, shorter cycle, **proof it can be done** |
| Plant QC team | **Not on the call** (daily users) | Fewer rounds, a suggestion they can trust and override |
| Design team | **Not on the call** (daily users; assumed role) | Master shades protected; no surprise changes to approved designs |
| Plant manager, plant IT/OT, finance sponsor | Assumed | Less rejected output; no new shop-floor network risk; clear return |

The people who buy are not the people who use it. Getting QC and design into the next call is our first ask.

### 1.3 As-is process (as we understand it)

1. Production target is set for an SKU. The printer holds the master sample and the previous run's sample.
2. One sample is printed, fired and checked in the lab (spectrophotometer: ΔL, Δa, Δb).
3. If OK, the whole batch runs.
4. If not OK, QC edits the image file in Photoshop by eye (6 primary colours, e.g. "brown +5%"), makes 6 or 7 variants, prints, fires and re-checks them. **One round is 6 to 7 hours.** Repeat until matched.
5. The line keeps running during this, so about 5,000 to 10,000 m² come out in the wrong shade.

**Pain points:** every fix is a guess; judging is visual so people differ; "99% match" is sometimes accepted; old reference files (e.g. from 2021) have drifted after many small tweaks, so nobody is sure what the true starting point is.

---

## 2. What we don't know yet

Ranked by how much the answer could change the solution or price. Full 12-question list is in Appendix A; the top 7 are here.

| # | Question for next call | We assume until answered |
|---|---|---|
| 1 | Can QC and design join the next call, and can we visit the **India plant**? | Yes, within 2 weeks |
| 2 | What tolerance (ΔE, Δgloss) counts as a match? | ΔE ≤ 1.0 vs master |
| 3 | How many SKUs, how often does batch change happen, which hurt most? | About 100 active SKUs; pilot on 6 to 8 of the worst |
| 4 | True split of the 6 to 7 hours, typical rounds, and the **unclear 5,000 to 10,000 m² figure**? | Steps: ~1h edit, 1h print, 3h kiln, 1.5h lab. Waste 5,000 m² (low end) |
| 5 | Does "5 to 6 minutes" mean the calculation only or the whole correction including firing? | The whole correction, so Phase 1 alone won't reach it, and we say so openly |
| 6 | What is the true starting reference per SKU (master, last run, or old file)? | Master is the target; latest approved print file is the start |
| 7 | Does shade shift come mainly from print or from kiln/raw material? | Mostly print-related; kiln is out of scope, as the client said |

**Decisions where the notes were unclear:** (a) the m² figure: we used the low end; (b) the line is assumed to keep running with the old file during correction, as the brief says; (c) pilot plant is India, because the client says the process is mostly the same across plants (an assumption to confirm); (d) currency is not stated, so we quote in INR with USD in brackets.

---

## 3. Solution

### 3.1 Options considered

| Option | Verdict | Reason |
|---|---|---|
| A. Inline camera / continuous vision | No | Expensive at three plants; still doesn't say how much to change; client said it need not run continuously |
| **B. Measure + calculate + human approves** | **Recommend (Phase 1)** | Fills the exact missing step; uses the device they own; fits "only at batch change" |
| C. Predict fired colour from unfired print | Phase 2, only if a test passes | Could reach "minutes", but needs a lot of paired data and kiln variation makes it risky |
| D. Fix the root cause (raw material, kiln) | No | Client themselves called it unrealistic for now |
| E. Clean up process and old reference files | **Add-on to B** | Cheap; our calculation needs a trustworthy start point. Includes one approved file per pilot SKU, a simple change log, and a written "good enough" rule |

### 3.2 Why B fits this client
It answers the real gap, reuses their hardware, runs only when needed, replaces eyesight with numbers, and keeps a person in control. It is also honest about time.

### 3.3 What we would deliberately not build
No camera or line monitoring. No root-cause analysis. No automatic push to the printer in Phase 1 (QC applies changes by hand). No promise of 5 to 6 minutes in Phase 1. No all-SKU rollout at once.

### 3.4 Phase 2 (optional, not priced)
Predict the fired colour from an unfired reading using the paired readings Phase 1 collects for free. Go only if prediction is within ΔE 1.0 of the real fired colour on at least 80% of test tiles (example numbers to agree with the client).

---

## 4. Solution engineering

### 4.1 Routine at each batch change
1. Select the SKU; the tool loads its master reading and calibration.
2. QC measures the current tile at agreed spots (ΔL, Δa, Δb, Δgloss).
3. The tool suggests a change, e.g. "brown -5%, blue -1.5%".
4. **QC lead reviews, edits if needed, and approves.**
5. QC applies it in the print file, as today.
6. Print and fire **one confirmation tile**; measure it.
7. Within tolerance → run the batch. If not → one more round with better data.
8. The result is saved, so the next batch starts better informed.

### 4.2 Inputs and data

| Input | Source |
|---|---|
| Master tile reading | Spectrophotometer, once per SKU |
| Current tile reading | Spectrophotometer, at batch change |
| Approved print file | QC / design (Option E) |
| Tolerance | Client decision (assumed ΔE ≤ 1.0) |
| Calibration table | Built by us per SKU family |

**Calibration (one-time per SKU family):** print one baseline tile, then for each of the 6 inks one tile with +step and one with -step (about ±5%). That is about 13 tiles. Fire and measure all. Result: a table of how each ink moves L, a, b and gloss. Every later confirmation tile adds a free data point (and unfired/fired pairs for Phase 2). Past Photoshop edits and lab results, if they exist, can reduce calibration work.

### 4.3 The logic (plain words, invented example)
If we know what each ink does, we can work backwards from a colour gap to the fix.

- Brown +1% moves the tile by ΔL -0.25, Δb +0.30. Blue +1% moves it by ΔL -0.10, Δb -0.40.
- The tile is too dark by 1.5 and too yellow by 1.0, so we need ΔL +1.5 and Δb -1.0.
- Solving the two equations gives **brown -5.4%, blue -1.5%**, which cancels the gap.

In the real system there are 6 inks and 4 measurements, so the computer finds the best compromise (least squares). Because firing makes the effect not perfectly straight, we calibrate near the normal operating point, cap one-step changes (e.g. ±10%), and show a **low-confidence flag** when the colour is far from anything calibrated. This is simple, explainable maths, deliberately not a black-box AI, so QC can see why a change was suggested.

### 4.4 Human sign-off
QC lead approves every suggestion. Large changes or low confidence need a second look. The confirmation tile must pass tolerance before the batch runs. The design team owns the master; the tool never changes it.

### 4.5 Fit with their tools, people and routine
- **Tools:** spectrophotometer data in as CSV; a simple screen on a **local plant laptop** (no cloud). QC applies changes in their existing Photoshop / printer software. **No printer connection in Phase 1**, so no new shop-floor network risk.
- **People:** about 1 day of QC training; we are on site for the first batch changes.
- **Routine:** replaces today's "guess in Photoshop and make 6 or 7 variants" step. Everything else stays.

---

## 5. Evaluation

A **plan**, not a result. All numbers are proposals to agree with the client.

**Baseline:** for 4 weeks before go-live QC logs each batch change on pilot SKUs (date, SKU, rounds, hours, ΔE, m² made wrong). Assumed today: 6 to 7 hrs per round, ~2 rounds, 5,000 m² wrong per change.

**Metrics:** *Primary:* first-tile pass rate (confirmation tile within tolerance); time to approved batch; wrong-shade m² per change. *Guardrail:* final ΔE to master. *Supporting:* rounds; suggested vs actual shift.

**Pass thresholds (pilot SKUs):**
1. First-tile pass rate ≥ **70%** (ΔE ≤ 1.0)
2. Median time to approved batch ≤ **3 hours**
3. Wrong-shade m² down ≥ **50%**
4. Final ΔE no worse than today's accepted tiles

**Test protocol (safest first):**
1. **Paper test:** feed old tiles and records into the tool. Would it have suggested the fix QC eventually found? No line risk.
2. **Side-by-side:** QC works as today; we record what the tool would have suggested.
3. **Live use:** QC tries the suggestion first, old method as backup.
4. Aim for **15 to 20 batch changes** in total, then review with QC and procurement.
5. **Who measures:** the client's QC team takes all readings with their own spectrophotometer, at the agreed spots. We observe and analyse, so the client owns the numbers.

**Stop if:**
- First-tile pass rate is **below 40%**, or no better than QC by eye.
- **Measurements are not repeatable:** the same tile read twice differs by more than tolerance.
- **The kiln is the main cause:** shade changes between firings of the same file exceed what a file change can fix.
- **Calibration doesn't carry over:** every batch needs full recalibration.

Stopping early is a valid, honest outcome.

---

## 6. Build plan and timeline (Phase 1: about 11 weeks, India pilot, 6 to 8 SKUs)

| Phase | What happens | Weeks |
|---|---|---|
| 0. Kickoff and access | QC and design join; visit India plant; receive tiles and records; agree tolerance and pilot SKUs | 1 to 2 |
| 1a. Baseline and setup | QC logs batch changes (parallel); check measurement repeatability; set up laptop tool; Option E starts | 2 to 5 |
| 1b. Calibration | ~13 tiles per SKU family; build tables | 4 to 6 |
| 1c. Paper test + side-by-side | Both together | 6 to 9 |
| 1d. Live use | Tool's suggestion first, old method as backup | 9 to 10 |
| 1e. Review | Compare to thresholds; expand / adjust / stop | 11 |
| 2. Optional extension | More SKUs, other plants, unfired-colour idea | Separate plan |

**Milestones the client sees:** Wk 2 kickoff done; Wk 5 baseline numbers; **Wk 7 first result** (calibration and first paper-test results); Wk 10 live use complete; **Wk 11 go / adjust / stop decision.**

**What depends on the client:**

| Dependency | Needed by | If late |
|---|---|---|
| QC and design at kickoff | Wk 1 to 2 | Risk of building the wrong thing |
| Plant access | Wk 2 | Calibration can't start |
| Master and variation tiles | Wk 2 | Paper test delayed |
| Tolerance and pilot SKUs agreed | Wk 2 | No clear pass/fail line |
| Kiln slots for calibration tiles | Wk 4 to 6 | Slips week for week |
| QC time for logging and side-by-side | Wk 2 to 10 | Weak baseline |
| Spectrophotometer CSV export | Wk 4 | Manual entry, slower |
| Procurement decision | Wk 11 | Delays expansion |

---

## 7. Cost

All rates, margin and exchange rate are **illustrative assumptions** (the notes contain no money figures). Banao's real rate card replaces them; only the rate column changes.

**Price = people × time × rate + travel + buffer + margin**

| Role | Weeks | Rate/week | Cost |
|---|---|---|---|
| FDE, part on site | 11 | ₹1,20,000 | ₹13,20,000 |
| Data/ML engineer | 7 | ₹1,20,000 | ₹8,40,000 |
| Software engineer | 4 | ₹90,000 | ₹3,60,000 |
| Project manager (part-time) | 2 | ₹80,000 | ₹1,60,000 |
| **Labour (24 person-weeks)** | | | **₹26,80,000** |

| Item | Amount |
|---|---|
| Labour | ₹26,80,000 |
| Travel and stay (3 trips, India plant) | ₹1,50,000 |
| Buffer 10% of ₹28,30,000 | ₹2,83,000 |
| **Our cost** | **₹31,13,000** |
| Margin 30% of cost | ₹9,33,900 |
| Cost + margin | ₹40,46,900 |
| **Proposed price (rounded)** | **₹40,00,000 (≈ $47,000 at ₹85 = $1)** |

Rounding down makes the effective margin about 28%. **Not included:** kiln slots, test tiles, the existing spectrophotometer, plant access (client dependencies).

**Payment (tied to results):** 30% at kickoff (wk 2) = ₹12,00,000; 40% at first result (wk 7) = ₹16,00,000; 30% at the decision review (wk 11) = ₹12,00,000. If we stop at week 7, the client has paid at most 70%.

**Running cost after Phase 1:** hosting ₹0 (local laptop); support and updates ₹1,00,000/month (~$1,180); each new SKU family calibration ₹60,000; plant laptop ~₹80,000 one-time (client buys).

**Value to the client (illustrative):**
- Wrong-shade output per batch change: 5,000 m² (low end). Target 50% less → **2,500 m² saved**.
- Loss per wrong m²: ₹150 (our guess) → 2,500 × ₹150 = **₹3,75,000 per change**.
- 4 batch changes per month on pilot SKUs (our guess) → **₹15,00,000/month**.
- **Payback:** ₹40,00,000 ÷ ₹15,00,000 ≈ **2.7 months** after go-live.
- Pessimistic case, 1 change per month: ₹3,75,000/month → payback ≈ **10.7 months**. This excludes other plants and faster shade approval.

Phase 2 and expansion are not priced; they are decided after week 11.

---

## 8. Risks

| # | Risk | L | I | Mitigation |
|---|---|---|---|---|
| 1 | QC and design don't engage | M | H | Include at kickoff; QC approves and can edit; on-site training |
| 2 | Kiln drives the shade change | M | H | Test kiln repeatability early; stop condition |
| 3 | Measurements not repeatable | M | H | Fixed spots; repeat-reading check wk 2 to 5; stop condition |
| 4 | Calibration doesn't carry over | M | M | Per-family calibration near operating point; track error; recalibrate |
| 5 | Too few batch changes to prove results | M | M | Target 15 to 20 cases; paper test; extend window |
| 6 | Client dependencies late | H | M | Dated list; slip week for week; say so early |
| 7 | Old reference files unreliable | M | M | Option E: one trusted file per SKU, change log |
| 8 | Over-promising "minutes" | H | H | State openly Phase 1 ≈ 2 to 3 hrs; Phase 2 only if test passes |
| 9 | Results differ at other plants | M | M | Pilot at India only; others priced and tested separately |
| 10 | Confidential design/process data | L | H | Local laptop, no cloud; data stays with client |

L = likelihood, I = impact (our judgement, not data). The two to say out loud: **#8** (the "minutes" expectation gap) and **#2** (the kiln may be the real cause).

---

## Appendix A: Full question list for the next call

1. Can QC and design join, and can we visit the India plant? *(assume yes, within 2 weeks)*
2. Tolerance (ΔE, Δgloss) for "match"? *(ΔE ≤ 1.0)*
3. Number of SKUs, batch-change frequency, worst SKUs? *(~100 SKUs; pilot 6 to 8)*
4. True split of the 6 to 7 hours; typical rounds; the unclear m² figure? *(step times as in Section 2; 5,000 m²)*
5. Does "5 to 6 minutes" mean calculation only or the whole correction? *(whole correction)*
6. True starting reference per SKU; are old files (e.g. 2021) usable? *(master = target; latest approved file = start)*
7. Print or kiln as the main cause of shade shift? *(mostly print)*
8. Which printer, inks, software? Can we export the print file; how do changes reach the printer? *(QC applies by hand; no integration in Phase 1)*
9. Where is measurement taken on multi-colour tiles; can data be exported as CSV; is gloss on the same device? *(fixed spots on key colour areas; CSV)*
10. Can the client supply master tiles, variation tiles and past correction records? *(only a small set at first)*
11. Do all three plants share printers, inks and kilns; where do we start? *(mostly the same; pilot India; others priced separately)*
12. Who approves a correction and owns the master; any quality standards? *(QC lead approves; design owns master)*

## Appendix B: Where the notes were unclear, and what we decided

| Unclear point | Decision | Why |
|---|---|---|
| m² wrong per batch change (recording unclear) | Use 5,000 m² | Conservative low end |
| Does the line run during correction? | Yes | Matches "lines run continuously" and the brief |
| Step times within the 6 to 7 hrs | Assumed | Not given; to confirm with QC |
| "5 to 6 minutes" meaning | Whole correction | Firing can't be removed in Phase 1 |
| Pilot plant | India | Client says process is mostly the same; easier on-site support |
| Currency | INR, USD in brackets | Not stated in notes |
| Tolerance | ΔE ≤ 1.0 | Not stated; "99% match" was accepted by eye |
| Process-flow document | Not available | Worked from the call notes only |
