---
name: sequence-state-tracker
description: |
  Per-prospect sequence step tracking and end-of-day activity report ingestion. The sequencing platform's API/connector is blocked in this environment, so this skill maintains a memory-backed registry of where each prospect sits in each sequence. Triggers when the rep (a) asks to draft outreach to a contact (email, InMail, DM, call script, bump, follow-up), (b) uploads or pastes an EOD activity report, (c) asks "what step is X on" / "where am I with [contact]." Infers next step from prior draft history + the rep's sequence definitions. Reconciles against EOD reports. Surfaces step state in every future draft for the same contact so outreach is never blind to history.
---

<!--
  TEMPLATE RECREATION of a production skill. This one is nearly logic-only, so
  it survives sanitization almost intact — the [CONFIGURE] surface is small:
  your memory backend, your sequence definitions, and your copy-authority skill.

  The engineering story: enterprise IT blocked the sequencing platform's API,
  which meant no live way to read what step any prospect was on. Rather than
  work around sanctioned tooling, the missing API was rebuilt as a state
  machine over two human touchpoints the rep already produces: drafting
  requests (in) and end-of-day activity reports (out). The invariant that
  makes it safe is DRAFTED ≠ SENT.
-->

# Sequence State Tracker

> The sequencing platform is blocked from integration. This skill is the manual replacement for its sequence-stage API. **The memory layer is the system of record.**

---

## Why this skill exists

The rep runs prospects through multi-touch sequences (email + LinkedIn + call) in a sequencing platform whose API is blocked by IT. Without this skill, every "draft outreach to [contact]" request risks repeating a step, skipping a step, or pitching the wrong angle for where the prospect actually is.

This skill keeps a memory-backed per-prospect step registry, inferred from the rep's draft requests and reconciled against EOD activity reports.

---

## When to trigger

**Outreach drafting requests**
- "draft an email / follow-up / InMail / DM / call script for [contact]"
- "bump [contact]" · "next step for [contact]" · "what should I send [contact]"

**EOD activity reports**
- The rep uploads or pastes anything labeled "EOD report" / "activity report" / "today's activity"
- "log my activity" / "here's what I sent"

**Direct state queries**
- "what step is [contact] on" · "where am I in the sequence with [contact]" · "have I emailed [contact] yet"

---

## Behavior

### On an outreach drafting request

1. **Search memory** for the contact's sequence state. Entity-centric queries only: `[Contact Name] sequence state`, `[Contact Name] outreach history`. Never temporal queries.
2. **If state found:** identify the last step, the sequence, and the date. Draft the NEXT step. Surface the prior touch in chat first:
   > "Last touch: [Contact] received [Step N — type] on [date]. Drafting Step N+1."
3. **If no state found:** never guess. Flag it and ask:
   > "No prior sequence state for [Contact]. Starting fresh — confirm sequence: [list] or treat as Step 1 of [default]?"
4. **After drafting**, write the state record:
   > Tag: `sequence-state-{contact-slug}`
   > Content: `Contact · Account · Sequence · Step N (type) · Status: DRAFTED [date] · Awaiting send confirmation from EOD report · Subject · Angle · Proof used.`
5. **Never re-draft a step already marked DRAFTED or SENT** unless the rep explicitly asks for a rewrite.

### On EOD report ingestion

1. **Parse** each line for: contact, account, sequence/step, action (sent / called / connected / replied), date.
2. **Reconcile each line against memory:**
   - Matching DRAFTED entry exists → update to **SENT** with the report date.
   - No DRAFTED entry → log as SENT directly (the rep sent something Claude didn't draft — capture it, don't complain).
   - Report says skipped → mark the DRAFTED entry **SKIPPED**; surface the gap.
3. **Surface the reconciliation in chat:**
   > "X drafted-and-confirmed-sent. Y sent without prior draft (now logged). Z drafted but not in the report — sent or skipped?"
4. **Write a daily summary** under `eod-reconciliation-YYYY-MM-DD` for traceability.

### On a direct state query

Search memory; answer plainly:
> "[Contact] is on Step [N] of [Sequence]. Last touch: [type] on [date]. Next due: [Step N+1] on [date per cadence]."

If no state is found, **say so — never guess.**

---

## Memory tag conventions

- `sequence-state-{contact-slug}` — current state per contact (latest entry wins)
- `sequence-history-{contact-slug}` — full history (append-only)
- `eod-reconciliation-YYYY-MM-DD` — daily reconciliation summary
- `sequence-gap` — drafted-but-unconfirmed items flagged after an EOD report

```
[CONFIGURE: the memory backend these tags live in — any persistent,
 searchable store shared across sessions. The tag scheme is the contract;
 the backend is swappable.]
```

## Sequence definitions

```
[CONFIGURE: your sequence catalog — name, steps, channel per step, day
 offsets. Keep definitions in a reference file, not hardcoded here, so
 sequence changes don't require a skill reinstall. Example shape:
   "Practitioner Track" — Day 1 email + connect · Day 2 call + VM ·
   Day 4 bump · Day 6 DM · Day 9 proof email · Day 11 call · Day 14 breakup]
```

---

## Hard rules

1. **Never re-draft a step already marked SENT** unless explicitly asked ("rewrite the bump I sent").
2. **DRAFTED ≠ SENT.** Never assume a send without EOD confirmation. This is the invariant the whole skill exists to protect.
3. **Always lead with the prior-touch summary** before showing a new draft — the rep needs the context to trust that the draft is the right next step.
4. **Ambiguous state → ask before drafting.** A one-line clarification beats a repeated step.
5. **EOD ingestion reads the rep's uploaded report only** — inbox scanning belongs to the inbox-sync automation, not this skill.
6. **The copy-authority skill is supreme for any words this skill drafts.** `[CONFIGURE: your copy engine]` — this skill tracks state; the copy engine writes the words.

---

## Example interactions (fictional names)

**Draft request with prior state**
> Rep: "Draft a bump to Dana Whitfield"
> Skill: finds `Dana Whitfield — Step 1 InMail SENT [date]` → drafts the Day-3 bump → "Last touch: Dana received InMail Step 1 on the 22nd. Drafting Day 3 bump."

**Draft request, no state**
> Rep: "Draft a cold email to Sam Ortiz, Director of People Ops at [account]"
> Skill: no state found → "Fresh contact. Which sequence: Practitioner Track, Exec Track, or treat as Step 1 of the default?"

**EOD report**
> Rep pastes: "Sends today: D. Whitfield bump 2, S. Ortiz email 1, M. Chen connection request…"
> Skill: reconciles → "3 sends confirmed against drafts. 0 gaps. Logged to eod-reconciliation-[date]."

**State query**
> Rep: "Where am I with Sam Ortiz?"
> Skill: "Sam is on Step 2 of Practitioner Track. Step 1 email SENT on the 20th. Step 2 call due today."
