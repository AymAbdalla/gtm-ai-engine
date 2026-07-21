# Architecture

How the GTM AI Engine is built. This is the system-design layer of the documentation; [OPERATIONS.md](OPERATIONS.md) covers how it runs day to day, and [REPLICATION.md](REPLICATION.md) covers how a new rep or team adopts it.

> Sanitization note: company-specific parameters (ICP bands, scoring weights, customer names, internal identifiers) are abstracted throughout. The structures are exactly as they run in production; the numbers are not published. The `skills/` folder contains full template recreations of the core skills with the same abstraction applied.

---

## What the system is

An operating layer for one enterprise BDR job, built inside Claude's desktop and cloud environment. It has five jobs:

1. **Qualify** — decide which accounts deserve work at all, graded against the real ICP and real won-customer evidence, never CRM intent scores.
2. **Map** — turn each worthwhile account into a plan: the 8–12 person buying committee, who to contact in what order, and the "why now" angle.
3. **Write** — produce cold outreach governed by a strict, benchmark-backed style law that adapts per buyer persona.
4. **Remember** — persist every piece of intel, every draft, every sequence step across sessions and machines, because chat conversations are disposable.
5. **Automate the grunt work** — scheduled unattended sessions handle enrichment, inbox hygiene, weekly target lists, and contact verification before the workday starts.

One design philosophy runs through all of it: **the human sends, the system drafts.** Nothing is ever emailed, published, or deleted autonomously. And honesty is enforced structurally — the system is built to refuse accounts the product cannot serve, refuse stats that cannot be traced to a source, and refuse customer name-drops that would not survive scrutiny.

---

## The seven layers

The system orders itself by permanence. Everything lives in exactly one layer, and the layers interact in one dominant direction: instructions govern skills, skills read the reference library and memory, automations write to the data layer, and every deliverable lands in outputs.

| Layer | What lives here | Changes how often |
|---|---|---|
| 1. Instructions | The constitution file + operator profile + writing rules | Rarely; edited deliberately, with dated backups |
| 2. Skills (law) | ~12 custom skills with explicit triggers and precedence | Only through deliberate re-installs |
| 3. Reference library | Playbooks, verified-stat banks, case-study indexes | When new enablement or data ships |
| 4. Data / state | Structured account book, contact sheet, per-account pages | Daily, by automations and by hand |
| 5. Memory | Semantic memory service (cross-session) + session journal | Every session |
| 6. Automation | Scheduled unattended tasks | Runs weekday mornings |
| 7. Work product | One output folder per deliverable | Every deliverable |

The rule that makes the layering work: **the chat is not the system of record.** Conversations are treated as scratch space. If something matters — a decision, a piece of intel, a sequence state — it must land in layer 4, 5, or 7 before the conversation ends, because the conversation itself will be gone. This was an explicit early design decision, made after watching context die between sessions on a sales cycle that runs 6–18 months.

## The instruction layer

A constitution file is read at the start of every session. It defines required per-session reading, the operator's context, the core rules (communication style, safety, the automation boundary, honesty), which skill governs which context, and explicit anti-patterns. Two details worth copying:

- **Dated backups.** Every edit to a core instruction file leaves a `.bak.YYYYMMDD` beside the original. Instructions are versioned like code because they behave like code.
- **Segment guardrails.** When the operator moved from Mid-Market to Enterprise mid-year, every number changed — book size, ICP bands, cadence lengths, threading rules. Old constants are sticky: they survive in older skills, prompts, and habits. So the constitution carries an explicit guardrail block listing the retired values, with an instruction to flag any of them that resurfaces. Config drift gets caught by rule, not by luck.

---

## The skill system

A skill is a markdown-defined behavior with an explicit trigger, a jurisdiction, and declared hand-offs. The system runs on ~12 custom skills plus stock document-generation skills. Three structural ideas make the ecosystem work:

**Supreme-law scoping.** Exactly one skill owns final authority over any given output type. Cold email copy to an HR buyer is governed by the HR persona protocol and nothing else; account fit is decided by the scorer and nothing else. Style and standards never drift by committee.

**Separation of powers.** Strategy skills supply the angle and structure. Copy skills supply the words. Data skills supply the evidence. State skills track state. No skill is allowed to do another's job — the outreach engine must defer copy to the persona protocol, which must pull proof from the evidence bank. The same account can get an HR email and a CFO email on the same day, governed by different supreme laws.

**Deterministic precedence.** Which skill governs is written down in a jurisdiction table in the constitution, not inferred per conversation. Conflicts resolve by hierarchy.

| Context | Governing skill |
|---|---|
| Cold email / DM craft + routing | Copy engine core (universal craft law + persona router) |
| HR / Finance / IT / Ops buyer copy | The matched persona protocol (supreme per department) |
| Account research, committee mapping, outreach strategy | Outreach engine |
| Customer proof points | Evidence bank (pure reference data; writes no copy) |
| Account fit, qualification, DQ | Account scorer (single source of truth) |
| Secondary-product fit | Standalone qualifier lens |
| Sequence step state | Sequence tracker |
| Background intel capture | Intel logger (always on, silent) |
| Everything else | The default writing rules |

The consolidation discipline matters as much as the skills: the current set is the survivor of an audit that collapsed 11 overlapping skills into this set, absorbing two teammates' variants, retiring dead ones, and converting one into a scheduled task. Single responsibility was enforced retroactively.

### The scorer (qualification engine)

Three modes: full-list triage, single-account qualification, and re-scoring on new intel. The design decisions that carry it:

- **A live-research gate.** No account is ever graded off the uploaded file. Headcount, industry, workforce type, current stack, and ownership are verified against live sources first. Most bad grades trace to trusting the file.
- **Two independent fit lenses.** Core-product fit and secondary-product fit are scored separately and never allowed to drag each other down — they answer different questions.
- **Two visible axes.** FIT (resemblance to customers actually won) and WIN (triggers, displaceable stack, intent, access) are scored separately, then combined into priority through a verdict matrix. An in-market non-fit can never rank high on heat alone; the matrix caps it at discovery-to-DQ.
- **Lookalike grounding.** Every scored account is matched against a small set of profiles distilled from real won customers, and must name its closest real customer, band-matched. "No honest match" is a valid output; fabricating or recycling a match is not.
- **DQ as a first-class outcome.** A hard disqualification taxonomy (absorbed subsidiaries, wrong workforce type, junk records, out-of-band size) removes accounts instead of parking them in "maybe." Removing them *is* the cleanup.

The current standard was hardened through a three-way bake-off — three competing scoring approaches run against the same books, best mechanics consolidated. See `skills/account-scorer-template/` for the full structure.

### The copy engine (router + persona protocols)

The most distinctive part of the system. All cold email and DM copy is governed by a two-level family grounded in [Lavender's](https://www.lavender.ai/) published Cold Email Benchmark Report (231,818 cold emails analyzed) — an external, published evidence base rather than taste.

- **A core craft engine** holds the universal law every message inherits: strict word-count ceilings, one idea per message, low reading level, mobile-first formatting, curiosity over information, problem-first openers, a call-to-conversation instead of a meeting ask, short internal-sounding subject lines. Each rule traces to a benchmark stat.
- **A router** reads the recipient's department and seniority, then hands authority to the matched persona protocol — HR, Finance, IT, or Operations — each of which is supreme law for its buyer. The protocols differ because the benchmark data differs: finance has the lowest A-grade rate but the biggest quality lift; IT rewards credibility over polish; operations punishes presumptive openers. Within each protocol, an altitude ladder adapts the message to seniority (executives get strategy and a delegate path; directors get one use case with proof; managers get one workflow; ICs get the shortest, warmest notes).
- **Checklist gates.** Every draft passes the core self-grade, then the persona protocol's own checklist, before it reaches the operator. One checklist item hard-requires that any stat or customer mention trace to the verified-stats file — zero exceptions.
- **An anti-AI-tell layer** bans the punctuation and vocabulary that read as machine-written (em dashes, double hyphens, the usual AI-tell word list), with a final test: a sharp peer would actually type this.

See `skills/outreach-copy-router-template/` for the full pattern.

### The state and capture skills

- **The sequence tracker** exists because the sequencing platform's API is blocked in this environment. It rebuilds the state machine the API would have provided: a memory-backed registry of where every prospect sits in every sequence, inferred from drafting requests and reconciled daily against the operator's end-of-day activity reports. Its central invariant: **DRAFTED ≠ SENT** — no draft is treated as delivered until a human activity report confirms it. See `skills/sequence-state-tracker-template/`.
- **The intel logger** runs silently under every interaction: every profile read, call disposition, reply, and shared document is captured to the account's page and the structured book, with a shared tag taxonomy the other skills consume, and a source + date on every fact. It runs before the outreach skills produce output, so their inputs are always fresh.

---

## The grounding library

Skills stay thin; deep knowledge lives in a reference library the skills point into. The anti-fabrication mechanics live here:

- **A verified-stats file** is the only legal source for any number cited in outreach. The copy checklists point at it by name.
- **A product-claims file** built from official product pages carries the rule at the top: if a claim isn't in this file, it doesn't go in an email.
- **A case-study corpus** (~85 source documents) is distilled into structured indexes — the skills read the indexes, not the PDFs.
- **A trigger-stats playbook** maps each trigger type (M&A, funding, new executive, expansion…) to a named third-party stat, a matching customer story, and channel routing — with the rule that stats are proof, never the hook.
- **Provenance rules for data fields.** Job titles follow a source-of-truth cascade (live profile beats CRM; every title carries a source stamp; a contact not findable on the live network is presumed departed). The current-stack field on an account may only ever be written from a live prospect conversation, never from web inference.

## The data layer

Structured state lives in a spreadsheet book plus per-account markdown pages, under an explicit division of labor: **the sheet holds short structured values, docs hold narrative, memory holds what must survive the chat.**

The sheet's column contract is enforced on the automations at column level: the operator's own notes column is off-limits to automation; the enrichment column is replace-only; the trigger-tags column is append-only; the freshness timestamp is machine-maintained and gates downstream behavior. Least privilege, applied to agents on shared state.

## The memory architecture

A governed, multi-agent memory layer shared between Claude and a second personal agent:

- **The semantic memory service** holds durable knowledge only. Every write carries scope and type metadata; writes are intentional-only (no automatic conversation ingestion); explicit never-store rules keep prospect data, bulk records, and PII out of semantic memory entirely.
- **Work records** (the sheet and account pages) hold prospect and account state, deliberately routed away from semantic memory.
- **A canon repo** of markdown files is the master copy. Memory services are treated as rebuildable indexes — the whole layer was migrated live from a retired memory product to the current one, with a scheduled 30-day checkpoint verifying the cutover.

Retrieval is entity-centric by rule (account and contact names, not "what happened last week") because entity queries hit the memory graph and temporal ones miss.

---

## Trust boundaries

The automation is trusted because it is bounded, and every boundary is written into the prompts and skills rather than assumed:

- **Draft, never send.** Email drafts are created as drafts. Nothing is sent, published, or deleted autonomously — ever.
- **Connectors and search only.** Scheduled tasks use API connectors and web search. Never screen control, never browser takeover.
- **Stop and report.** An integration failure ends the run with a report. Automations never improvise around a broken dependency.
- **Column-level write permissions** on shared data, as above. Append-only where history matters.
- **Dead dependencies are fenced.** When a data vendor was deauthorized mid-year, every skill and task was rewritten with an explicit "never call it" rule — so no automation could quietly regress to a dead connector when it briefly reappeared.

## Design rationale

**Because chats die and enterprise cycles don't.** A 6–18 month, 8–12 stakeholder pursuit cannot live in a chat thread; state is pushed into durable layers by rule.

**Because the sequencer API was blocked.** A missing integration became a human-in-the-loop state machine (draft history + daily reconciliation) rather than a workaround. Nothing routes around sanctioned tooling; the system is engineered to perform within it.

**Because a data source vanished mid-flight.** The enrichment stack was rebuilt source-agnostic in days — web research primary, community-signal tool secondary — and re-absorbed the vendor when it returned, with no downstream behavior change.

**Because one bad draft is expensive.** A single stale-context draft in June became a postmortem, which became permanent law: a mandatory pre-outreach intel check and a freshness gate on every account (see OPERATIONS.md).

**Because credibility is the whole game upmarket.** Band-matching, verified-stats-only, one proof point per touch, and honesty guardrails about where the product genuinely loses exist so nothing said to a C-suite buyer can be embarrassed by a follow-up question. The scorer will DQ revenue-looking accounts on purpose.

**Because style drifts unless exactly one law governs.** The supreme-law model came from watching a single one-size rulebook mis-style copy for other departments. One protocol per buyer, a router deciding, checklists gating.

**Because trust in automation is earned by boundaries.** Draft-never-send, connectors-only, stop-and-report, off-limits columns: the morning automations run unattended precisely because they cannot embarrass their owner.
