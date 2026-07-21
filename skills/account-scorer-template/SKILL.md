---
name: account-scorer
description: |
  The ICP cleanup + account-qualification engine — the single source of truth for "is this account a fit and should we work it." For any rep on the team. Trigger when a rep: uploads an Excel/CSV account list (territory, unassigned pool, or a teammate's book) and wants it triaged into go-after / nurture / DQ / claim; asks "which accounts should I work," "score/prioritize my list"; asks about ONE account — "is X a good fit," "is X in our ICP"; is prepping a discovery/intro call with a borderline or unknown account and asks "what should I ask," "help me qualify/DQ this"; or shares new intel to re-score an account. Scores against the configured ICP (core-product fit + secondary-product fit), labels Good Fit / Borderline / Non-Fit, ranks by Priority, matches accounts to real won customers, and generates discovery questions that uncover latent fit — not just DQ. Ignores all CRM/intent-vendor scores.
---

<!--
  TEMPLATE RECREATION of a production skill. Structure is production-real;
  every value inside a [CONFIGURE] block is either blank or marked
  "illustrative." Fill the [CONFIGURE] blocks from your own ICP, won-customer
  base, and product before installing. See docs/REPLICATION.md.
-->

# ICP Cleanup + Account Qualification Engine

The single source of truth for account fit. Any rep can run it on their own book. For one account or a whole list, it answers:

1. **Is it a fit?** Good Fit / Borderline / Non-Fit — graded against the ICP, grounded in the customers we actually win.
2. **How strong is the win, and how soon?** Triggers, displaceable stack, in-market intent, reachable buyer.
3. **Does the secondary product fit?** A separate yes/no lens that must never affect the core grade.
4. **If it's borderline or non-fit — what do I ask** to disqualify it cleanly OR uncover latent fit before walking away.

> Four non-negotiables, baked in below: (1) **live-research every account before grading** — never score off the uploaded file; (2) **keep the two ICP lenses separate** — the secondary lens never drags down core fit; (3) **always populate Lookalike + Closest customer**, band-matched and never repeated; (4) **DQ the clear non-fits** instead of parking them in "qualify." Default output is **lean**: color-coded FIT + Win heat + working columns, no numeric score columns surfaced.

## THREE MODES

1. **List Triage** — upload a list (territory / unassigned pool / teammate book) → every account labeled + ranked + verdict (Go-after / Nurture / DQ-remove / Claim).
2. **Single-Account Qualify** — one account (often meeting prep) → fit assessment + tailored discovery/DQ questions.
3. **Update** — new intel on an account → re-score, revise the play.

## CRITICAL RULES

- **Live-research every account BEFORE grading — this is the gate.** The uploaded file's employee count and industry label are *hints, not truth*. Confirm each via live web research: current headcount, what they actually do + workforce type, current stack, recent triggers (funding / M&A / new executives / growth / layoffs), and **ownership** (acquired? a subsidiary whose buying is run by the parent?). Almost every bad grade traces to trusting the file. If a freshness gate applies (data older than 7 days), refresh before scoring.
- **Verify headcount — flag mis-sized and junk records.** Real size often differs wildly from the file (a "5,700-EE" row that's really ~120; a duplicate; a defunct entity). Score on the *confirmed* number and surface the correction.
- **Ignore all CRM / intent-vendor / legacy ICP scores.** Build fresh, against evidence.
- **Two fit lenses, scored SEPARATELY — never collapse them.** (A) Core-product ICP and (B) secondary-product ICP are independent questions. An account can be a strong core fit with a hard secondary-product DQ; surface both, and never let one grade drag the other.
- **Fit is grounded in real won customers** (lookalike match), not abstract points.
- **Lookalike + Closest customer are ALWAYS populated — never omit, never repeat, never fabricate.** Every scored account names its matched lookalike profile(s) and one real, band-matched won customer, matched on pain + size band + industry. Don't paste the same generic customer across unrelated rows, and never name a customer from a smaller band as proof to a larger prospect. If no honest band-matched customer exists, write "no band-matched match" rather than inventing or recycling one.
- **Rank by Priority** (Fit + Win) but always surface the axes separately. A Non-Fit never becomes a priority just because it's in-market — flag the fit problem.
- **Team-generic.** Written for "the rep." No personal hardcoding.

---

## THE TWO ICP LENSES

### Lens A — Core-product ICP

```
[CONFIGURE — from your won-customer analysis, not from opinion:
  Sweet-spot size band:        ___ – ___ employees
  Strong band / stretch band:  ___ – ___ / ___ – ___
  Out-of-band (DQ unless carve-out): < ___ or > ___
  Strong-fit industries:       (list)
  Strong-fit signals:          (replaceable incumbent stack, growth/funding
                                events, multi-country footprint, hiring velocity…)
  Weak / non-fit:              (workforce types, sectors, and states your
                                product genuinely does not serve — be honest;
                                this list is what makes DQ defensible)]
```

### Lens B — Secondary-product ICP (separate)

```
[CONFIGURE — the attach product's own independent criteria, e.g. an
 eligibility ceiling on a sub-population, a complexity rule, a hard
 exclusion list. If you have no attach product, delete this lens and
 every reference to it.]
```

The lens rides on its own dimension (illustrative: an attach product gated on *US-based* headcount rides on the US number, not global size — a 1,500-EE global company can still qualify). Flag it; never gate core fit on it.

---

## EXPECTED INPUT

An Excel/CSV export (any rep's book, an unassigned list, or a single account name). Typical columns: Account Name, Domain, Employee Count (verify), Industry (confirm — don't trust), Owner, Last Activity. Ignore any legacy score columns. Work with what's there; flag gaps.

---

# MODE 1 — LIST TRIAGE

## Two-pass research

**Pass 1 (every account):** confirm employee count + **confirm industry** (live search — don't trust the label); **leadership scan** (the buying-committee-relevant C-suite — anyone in-seat <90 days is a trigger); news/trigger pass; lookalike classification; estimate the secondary-lens population; assign all tags + Segment + Fit + Win + secondary flag.
**Pass 2 (top focus accounts):** incumbent-stack detection, funding/growth, intent deep-dive; refine. (Committee mapping itself lives in the outreach engine — separation of powers.)

## Segment flag

```
[CONFIGURE — your segment bands and the motion each gets. Illustrative shape:]
```

| Employees | Size_Band | Segment | Motion |
|---|---|---|---|
| < *(floor)* | Too-Small | Out-of-band (DQ unless strategic) | — |
| *(low bands)* | SMB / Mid-Market | MM | primary + 1 multi-thread |
| *(core band)* | Enterprise-Core | Enterprise (sweet spot) | committee 8–12 |
| *(stretch band)* | Enterprise-Stretch | Enterprise (qualify hard) | committee 8–12 |
| > *(ceiling)* | Too-Big | Out-of-band (DQ unless division/carve-out) | — |

## Tags (Pass 1; refresh Pass 2)

`Industry_Tag` · `Size_Band` · `Segment_Tag` · `Trigger_Tag` (New_[Buyer]_Leader · New_Finance_Leader · New_C-Suite · Funding · Growth · M&A · Intl_Expansion · Dept_Hiring · Stack_Pain · Competitor_Eval · Re-engage · No_Signal) · `Stack_Tag` (incumbent system) · `Intent_Tag` (🔴 Active_Buyer / 🟠 Researching / 🟡 Mild / ⚪ None) · `Lookalike_Tag` · `Secondary_Fit` (✅ / ⚠️ verify / ❌) · `Fit_Grade` · `Win_Strength` (🔥/🟠/❄️)

## AXIS 1 — FIT → Good / Borderline / Non-Fit (score /50)

"How much do they look like the customers we win, at a band the product serves well?"

- **Band fit (0–20):** `[CONFIGURE point ladder — core band scores full marks, stretch bands step down, out-of-band scores near zero]`
- **Industry fit (0–10):** `[CONFIGURE point ladder from your won-customer verticals]`
- **Lookalike match (0–20) — the heart:** 2+ profiles (compound) = 20 · 1 strong = 14 · 1 weak = 8 · none = 2. *(Illustrative weights; the structural rule is that lookalike match is the largest single component — fit is resemblance to won customers, not a demographic checklist.)*
- **Grade:** **Good Fit** = 40–50 (A) or 30–39 (B) · **Borderline** = 20–29 (C) · **Non-Fit** = <20 (D). Always name the matched profile(s) + closest real customer.

### The lookalike profiles (match in Pass 1)

```
[CONFIGURE — 6-10 pre-purchase "states" reverse-engineered from your own
 won customers. Each profile needs: a name, how to spot it from the outside,
 which real customers were in that state, and the angle. Illustrative examples:]
```

1. **Outgrown_Starter_Stack** — on an SMB-grade incumbent at ~2x the headcount they bought it at.
2. **Spreadsheet_Ceiling** — no system of record at a size where peers all have one; the tool shows up in their job posts.
3. **Headcount_Rocket** — >30% growth in 6 months; recent funding round.
4. **Tool_Sprawl** — 3+ point tools where one platform should sit.

A compound match (2+ profiles) = strong Fit and a moderate intent signal in itself.

## AXIS 2 — WIN-STRENGTH → 🔥 Hot / 🟠 Warm / ❄️ Cool (score /50)

"Even if they fit, how strong is the opening now?"

- **Triggers (0–20):** `[CONFIGURE a point value per trigger type — new buyer-side executive <90 days should top the ladder; recent funding close behind; take the top 1-2, cap at 20]`
- **Displaceable stack (0–15):** a beatable incumbent **+ a pain/migration signal**. `[CONFIGURE your displacement ladder: which incumbents are beatable at which segment, what "no system at size X" scores, and which incumbent = automatic DQ (your own product).]` A legacy incumbent with *no* pain signal scores low — displacement needs evidence, not just presence.
- **Intent (0–10, with a ×1.0–1.5 multiplier on the Win subtotal):** Active_Buyer = 10 · Researching = 6 · Mild = 3. Committee-wide intent (two seats researching independently) is the strongest signal.
- **Access (0–5):** verified warm path = 5 · past opp / reachable buyer = 3 · cold but mappable = 2 · none = 0.
- **Heat:** 🔥 Hot 35–50 · 🟠 Warm 20–34 · ❄️ Cool <20 (after the multiplier).

## PRIORITY + VERDICT

`Priority = Fit + (Win × intent multiplier)` → sort descending. Verdict per account:

| | 🔥 Hot | 🟠 Warm | ❄️ Cool |
|---|---|---|---|
| **Good Fit** | **Go-after now (P1)** | Go-after (P2) | Nurture + trigger-watch |
| **Borderline** | Qualify fast (P2) | Qualify (P3) | Nurture |
| **Non-Fit** | Discovery-to-DQ (verify) | Nurture/DQ | **DQ-remove** |

The matrix is the safety rail: an in-market Non-Fit caps at discovery-to-DQ. Heat never overrides fit.

For an **unassigned list**, the verdict reads as **Claim** (Good Fit, any heat) / **Watch** (Borderline) / **Skip** (Non-Fit).

## DISQUALIFIERS — grade these 🔴 Non-Fit, don't park them in "qualify"

A clear DQ is a Non-Fit, not a maybe — removing them *is* the cleanup.

```
[CONFIGURE your hard-DQ taxonomy. The production categories, genericized:
 already a customer · acquired/absorbed with no independent buying authority
 (verify autonomy on any subsidiary before keeping) · workforce type the
 product doesn't serve · excluded sectors · junk/mis-sized/defunct records ·
 out-of-band size (unless a division/carve-out play) · signed a competitor
 within the entrenchment window (cap at Cool) · recent large layoffs (timing
 hold, usually Borderline).]
```

The one case to keep as Borderline rather than DQ: genuine doubt about a workable *segment* play inside an otherwise-DQ account — that's what discovery-to-DQ questions are for.

## OUTPUT (list mode)

1. **Color-filled Excel (.xlsx)** — one row per account, the **FIT cell shaded 🟢 green (Keep) · 🟡 yellow (Qualify) · 🔴 red (Remove/DQ)**. Columns: Account · Domain · Employees (confirmed; note the file's figure if it was wrong) · Segment · Industry / Workforce · **FIT** · **Win** (🔥/🟠/❄️) · Action · Why (1 line) · Current stack · **Why-now trigger** · **Lookalike** · **Closest customer** · **Secondary_Fit** · Discovery Questions (for 🟡/🔴) · Data flag (mis-sized / duplicate / absorbed). Sort 🟢→🟡→🔴, hottest first within each color. **Lean schema:** the FIT color and Win icon carry the grade — score the /50 rubrics internally but don't surface numeric score columns. Add a Summary tab with keep/qualify/remove counts + segment split.
2. **Grouped doc** — 3-line summary up top (counts), then accounts grouped by color, with **4–6 discovery questions** under every 🟡 Borderline and a one-line DQ reason under each 🔴.
3. **In-chat summary** — segment split, fit distribution, secondary-eligible count, and the remove list.

---

# MODE 2 — SINGLE-ACCOUNT QUALIFY

For one account — often meeting prep for a borderline or unknown account. Score it (same two lenses), then generate the questions that confirm, DQ, or flip it.

## Assessment output

```
ACCOUNT FIT: [Good Fit / Borderline / Non-Fit]   |   Secondary Fit: [✅ / ⚠️ verify / ❌]
Why:
- [signal 1 — confirmed, with source]
- [signal 2]
- [signal 3]
Unknowns that would change this:
- [e.g., "Current stack unknown — if legacy/none, fit goes up"]
Recommended next action:
- [Good → lead with the band-matched proof]
- [Borderline/Non-Fit → use the discovery questions below before investing]
```

## Discovery & DQ question bank

Pick **5–8**, matched to *what makes this account borderline*. The goal is to surface the signal that would flip it, or DQ cleanly — the questions are qualification tools, not filler.

```
[CONFIGURE question sets per borderline condition. The production structure,
 with illustrative questions:]
```

**Size above band — find the segment play:** "Any business units or recently acquired entities that run somewhat independently of the corporate stack?" · "Are there teams the current system underserves?"
**Industry borderline:** "What's the split between [served] and [unserved] workforce types?" · "Is there a corporate/HQ segment with different needs than the rest?"
**Looks static:** "What's driving this conversation right now — any event or timeline?" · "Headcount changes planned in the next 12–18 months?" · "Any recent or planned M&A?"
**Current stack unknown:** "What do you use today — one platform or a few stitched together?" · "How happy is the team with it, 1–10 — what would raise it?"
**Entrenched modern incumbent — find dissatisfaction:** "How much team time goes into maintaining it vs. the actual work?" · "When you want a change, how long does it take?" · "What do you wish it did better?"
**Budget/freeze:** "Exploratory, or an active initiative with budget?" · "What needs to be true internally to green-light this in 6 months?"
**Secondary-product qualification:** `[CONFIGURE the attach product's confirm-on-discovery questions — any one red answer = secondary DQ; pivot to core value.]`

## Post-discovery reassessment

After the call, re-grade on what surfaced. Latent fit appeared → bump the grade and hand to the outreach engine for committee mapping. No fit despite good questions → confirm Non-Fit and record why.

---

# MODE 3 — UPDATE

Trigger: "update [account]", "I called [account]…". Open the account's page; add activity → Outreach Log, facts → Updated Intel; re-score both axes if material; rewrite the play; **never overwrite the Outreach Log or Updated Intel** — they are append-only history.

---

## WHAT NOT TO DO

- Never use CRM/intent-vendor scores · never skip the industry confirm, leadership scan, or lookalike match.
- Never leave Segment, Fit, Win, or Secondary_Fit blank on a scored account.
- Never name a smaller-band customer as proof to a bigger prospect; never invent a stat or logo.
- Never let a Non-Fit rank as a priority because it's in-market — flag the fit gap and switch to discovery-to-DQ questions.
- Never grade off the uploaded file's numbers without live verification.
- Never let the secondary lens change the core FIT grade.
- Never repeat the same proof across unrelated accounts.
- Never park a clear DQ in "qualify" — that's a 🔴 Non-Fit.
