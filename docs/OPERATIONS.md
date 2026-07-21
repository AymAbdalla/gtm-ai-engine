# Operations

How the system runs day to day. [ARCHITECTURE.md](ARCHITECTURE.md) covers how it's built; this covers the operating rhythm — what happens at session open, before any draft, overnight, and at end of day.

> Same sanitization as the rest of the docs: structures are production-real, company-specific parameters and identifiers are abstracted.

---

## Session bookends

Every working session opens and closes the same way, because the chat itself is disposable and the durable layers are not.

**Open:** Claude reads the constitution file, the operator profile, the writing rules, and the session journal, then searches memory — entity-centric, on whatever account or project is in play — and checks the structured book's current-week tab. The session starts already knowing what April knew.

**Close:** a dated entry is appended to the session journal (worked on / decided / next), and a structured summary is written to the memory service. Mid-session capture is automatic — the intel logger writes account intel as it appears, so session close is a summary, not a scramble.

---

## The pre-outreach intel check

Mandatory before drafting, rewriting, or grading any outreach for a named account or contact. In order:

1. **The account's sheet row** — trigger tags, enrichment intel, operator notes, freshness stamp.
2. **Memory** — entity-centric search on both the account and the contact.
3. **The local account page**, if one exists.
4. **The freshness gate** — if the account's data is more than 7 days old, or it was never deep-scanned, run a live research refresh *first*. Never trust "triggers: none" on a stale page.

Then: lead with the highest-priority trigger (M&A beats new HR leader beats new CFO beats funding beats expansion, with a persona-pain fallback), surface a one-line intel summary in chat *before* showing any copy, and confirm which committee seat the contact holds.

This ritual exists because of one specific miss: in June, a draft went out referencing an account's old identity while the live trigger was its just-announced acquisition. The postmortem turned "never draft blind" from a habit into law. It has been enforced structurally ever since — an incident became a permanent control, which is the pattern for how this system hardens.

## Drafting one message (the full chain)

A single "draft an email to this person" fires the whole stack, in order:

1. The **intel logger** captures whatever was shared (a profile, a screenshot, verbal context) to the account's records.
2. The **sequence tracker** checks memory for existing sequence state — if the contact is mid-sequence, the draft must be the *next* step, and the prior-touch summary is surfaced first.
3. The **pre-outreach intel check** runs (above).
4. The **outreach engine** confirms which committee seat the contact holds and the account's angle.
5. The **copy router** reads the title, resolves department + seniority, and hands authority to the matched persona protocol.
6. The **persona protocol** writes under its rules and requests exactly one band-matched proof point from the **evidence bank**.
7. The draft passes the universal self-grade, then the persona checklist (including the verified-stats trace).
8. The intel summary and draft land in chat; the logger writes the activity record; the tracker marks the step DRAFTED.
9. **The operator sends manually.** Claude never sends.

## Working an account (the enterprise motion)

New priority account → the scorer qualifies it (fit grade, win heat, secondary-product flag, closest won customer) → the outreach engine builds the plan and maps the committee, targeting 8–12 seats with a confidence rating and source on every name. Two hard rules: an account with fewer than two sourced seats is flagged **single-threaded and not cleared to launch**, and committee coverage is a tracked percentage reported every time.

Sequencing then runs two tracks: a pain-led practitioner track over roughly two weeks, and a patient executive track over four to five weeks that opens warm after the practitioner thread is moving. Multi-thread order: start at the champion seat, build a coach, go up to the economic buyer warm, then open the finance and IT gates. Never cold-blast all seats on day one.

## List triage

A list arrives — a territory cut, an unassigned pull, a departed teammate's book — and the scorer runs it in batches of 50 with live research on every row. Output: the color-coded workbook (green keep / yellow qualify / red remove), a grouped doc with discovery questions under every borderline account, and the remove list. Good fits feed the book and the weekly attack buckets; borderlines get qualified on calls; non-fits get removed honestly.

---

## The morning cascade

Five scheduled tasks run unattended on weekday mornings, in a deliberate order so each feeds the next. The operator sits down around 8:00 to a fresh, reconciled picture instead of two hours of research.

| Order | Task | What it does |
|---|---|---|
| Mon only, first | **Intent radar** | Finds ~10 net-new in-market accounts *not* in the book, refreshes the current priority set, surfaces low-hanging fruit (aged MQLs, under-worked top accounts), and scores everything with the scorer's own rubric. |
| Mon only, second | **Weekly attack pack** | Drafts the week's 20-account attack list from the segment buckets, pre-maps 5–6 committee seats per account (reusing verified contacts before searching; every web-sourced name marked UNVERIFIED), writes a value hypothesis, phone opener, discovery questions, and multi-thread order per account — then **waits for approval**. Draft, never launch. |
| Daily | **Inbox sync** | Scans the work inbox for hard bounces, departure auto-replies (capturing named replacements), OOOs, unsubscribes, and real replies. Writes contact statuses to the sheet; logs replies and departures to memory. An OOO is never marked as a bounce. |
| Daily | **Enrichment** | Takes the ~15 stalest accounts by freshness stamp, runs the standard trigger queries per account, maintains the dated segment buckets with auto-expiry windows, writes its columns under the column contract, and renders a styled HTML report with a conversation angle attached to every signal. |
| Daily | **Contact verification** | Verifies ~30 contacts per run against the live web, senior-first. One of five statuses per contact, written back through a small Python engine that uses the workbook itself as the state store — the task needs no memory of prior runs. **Self-terminating:** when the queue hits zero it recommends disabling itself. |

Design notes worth stealing:

- **The segment buckets are the connective tissue.** Enrichment maintains them, the attack pack consumes them, the radar cross-checks them. Tasks coordinate through shared, dated tags in the data layer — not through each other.
- **The verification task exists because of a measurement.** The first verification batch found roughly half of senior contacts stale — departed, moved, or retitled. That number justified a daily task; the task now deletes its own reason to exist.
- **Boundaries are in every prompt.** Connectors and web search only; no screen control; integration failure = stop and report; nothing sends email. All five tasks are draft-and-notify.

## End of day: reconciliation

The operator pastes the day's activity report from the sequencing platform. The sequence tracker parses it and reconciles against memory: DRAFTED entries confirmed in the report become SENT; sends with no prior draft get logged; drafts missing from the report are surfaced as gaps ("Step 2 for [contact] was drafted on the 12th but never confirmed sent — sent or skipped?"). A daily reconciliation summary lands in memory.

This is the human half of the state machine the blocked API made necessary. The morning inbox sync handles the passive side (bounces, departures, replies); the EOD report handles the active side (what actually went out). Between the two, sequence state stays true without any integration.

## The weekly rhythm

Deep-work days bookend the week: approve or swap the attack pack, verify UNVERIFIED committee names, build account plans, run list triage, clean the CRM. Office days in between are call blocks and meetings, with per-call kits built before each block. The weekly activity engine stays fixed; spare capacity redirects into committee depth on existing accounts — never into more accounts.
