# Replication

How the system was packaged so other reps and regions could adopt it — and what it takes to run the same play on a different team, territory, or product. This is the part of the project that turned a personal workflow into org infrastructure: adopted across an 18-rep US BDR org and handed to EMEA BDR management for region-tailored rollout.

> Sanitized from the real handoff artifacts: the v1.0 suite package, its setup guide, and the rep onboarding doc. Structures and adoption mechanics are as-shipped; company parameters are abstracted.

---

## The packaging principle

A workflow that lives in one person's head doesn't replicate; a workflow that lives in versioned skill files does. The core skills were written **team-generic** from the consolidation onward — addressed to "the rep," no personal hardcoding — so they could be handed over as files rather than explained as folklore.

The shipped unit is a **sales suite**: nine skills (the scorer, the outreach engine, the evidence bank, the secondary-product qualifier, and the five-skill copy family — core router plus four persona protocols), packaged **both ways at once**:

- **One `.plugin` bundle** — the whole suite in a single install that stays in sync as one unit, publishable to a team plugin marketplace so reps adopt it in one click.
- **An `individual-skills/` folder** — the same nine as separate `.skill` files for cherry-picking.

A plain-text README and a setup guide ride along, plus one documented fallback that demystifies the whole format: *a skill is just a folder with a SKILL.md* — unzip and drop it in manually if the UI path ever fails.

Two packaging decisions matter most:

**Degrade gracefully by construction.** The suite's deep reference library — playbooks, case-study corpus, customer list — is deliberately *not* shipped; every skill carries its core logic inline and runs on Claude plus web search alone. Connectors and reference files make the skills **richer, not functional**. This is what makes the package portable: nothing breaks in a new environment that lacks the author's workspace, data vendors, or connectors, and the receiving team rebuilds the reference layer from *their own* market's evidence (which they should do anyway — see localization).

**Ship the sales core, not the whole operating system.** The workspace layer — the constitution file, persistent memory, scheduled automation, living account pages — is explicitly excluded from the pack, with a documented offer to help a team stand up the full system separately. Skills replicate cleanly through files; an operating layer replicates through guided setup. The packaging respects the difference.

## Install paths and the first ten minutes

**Path A — whole team, one install (recommended):** add the `.plugin` under the app's capabilities settings, or publish it to the team marketplace. All nine skills land together.

**Path B — cherry-pick:** install individual `.skill` files. The guide names the fastest "wow" pair to start with: the scorer plus the evidence bank.

Adoption lives or dies on the first run, so the first run is scripted. The setup guide ships three **smoke tests** — if the right skill fires on each, the install works:

1. *"Score these accounts for fit"* + a short CSV → Good Fit / Borderline / Non-Fit, a priority rank, and discovery questions.
2. *"Map the buying committee at [company]"* → 8–12 seats with titles and who to call first.
3. *"Draft a cold email to a VP People at [company]"* → a visible routing line (HR → the HR protocol) followed by a short, one-idea email.

The third test is the diagnostic gem: the rep *watches the router fire* before the copy appears, which teaches the governance model in one prompt.

A starter-prompt table follows — two or three natural phrasings per skill — because skills trigger from what a rep says, not from commands, and new users need to see the phrasings that fire them.

## The customization path: interview and rewrite

For a rep or team that wants the system fitted rather than installed: attach the `.skill` files to a conversation and have Claude **interview the adopter** — territory, segment bands, verticals, proof stories, voice — and rewrite the skills for their book before installing.

This is the more interesting engineering idea in the handoff: **the skills are self-customizing distribution.** Because a skill is markdown, the same agent that runs it can rewrite it, and the interview extracts exactly the parameters the architecture needs. No one maintains a fork by hand; the fork writes itself against the parameter list below.

## What a team customizes vs. what stays invariant

The EMEA handoff formalized this as a localization worksheet in the setup guide — an explicit table of what the pack assumes and what a region changes. Generalized:

**Parameters (swap per team / region / product):**

- ICP and territory definition: size bands, target geographies, language
- Scoring weights and trigger values inside the two axes
- The lookalike profiles — re-derived from *that region's* won customers, never copied
- The evidence bank — regional logos lead; a proof that lands in one market can be unknown in another
- The secondary-product lens — in the real handoff this was the piece marked *not applicable* for the receiving region (its equivalent is country-specific), which is exactly why it's a separate lens the architecture can drop without touching core fit
- The competitive set — same category, re-weighted, plus local players
- Language, outreach norms, and compliance regime (the receiving region's guide flags data-protection rules on all enrichment and outreach)
- Cosmetics down to the sample signatures in the protocol examples

**Invariants (the architecture — don't fork these):**

- The live-research gate: no account graded off the uploaded file
- Two independent fit lenses; two visible axes; the verdict matrix capping in-market non-fits
- DQ as a first-class outcome
- Lookalike grounding with "no honest match" as a legal output; band-matched proof only
- Supreme-law copy routing with persona protocols and checklist gates
- The verified-stats trace rule: no number in an email without a source on file
- DRAFTED ≠ SENT and the reconciliation loop
- Draft-never-send and the automation boundaries

A region that swaps the parameters and keeps the invariants gets the same system. A region that forks the invariants gets a different, probably worse system. The handoff documentation ships the invariants as law and the parameters as a worksheet.

## Onboarding non-technical adopters

The rep onboarding doc assumes its reader has only ever used a chat box, and its framing choices are why it landed:

- **It opens with the mental shift, not the tooling:** a chat box answers and forgets; this is *an operating system you build around your job* — it knows your accounts, remembers across weeks, runs jobs before you sit down, and writes in your voice. "That's not a chatbot. That's a coworker who shows up before you do and remembers everything."
- **It teaches a simplified layer model** — instructions, memory, workspace, connectors, skills — a deliberately compressed version of the production architecture, because five layers teach and seven overwhelm.
- **Setup is zero-to-first-workflow in five steps**, and the centerpiece is a **minimum-viable constitution file of ~150 words** (who I am, how to talk to me, what I do, what NOT to do, my day pattern) with explicit advice: don't aim for perfect, write it, iterate. The safety rules are in the starter template from day one — never send without showing the draft, never delete without asking, don't pretend to know data you can't verify.
- **Restraint is taught as a feature:** connect five tools, not fifteen ("bloat slows things down"); add skills when a repeated task earns one; "treat it like a smart intern, not an autonomous agent — always read what it's about to do before approving."
- **A Common Mistakes section names the failure modes** observed in real adopters: treating it like a chat box, skipping the constitution, skipping memory, running unsupervised on real systems, installing everything at once, day-1 perfectionism.
- **A first-week table** paces the ramp day by day, ending with the rep running one skill on one real prospect — then the team stack installs on top once the basics feel natural.

The sequencing is the point: the generic operating skills come first, the team's tuned stack second, and the full workspace layer only when the rep wants it. Nobody is handed the whole machine on day one.

## Rollout, in practice

The sequence that worked: run it yourself until the numbers move → package the skills team-generic → demo to the immediate team on each rep's own book, live → hand the suite + setup guide to adjacent teams → present the workflow upstream to RevOps and BDR leadership, arguing the same trigger criteria belong in account assignment itself → ship the full package to EMEA management with the localization worksheet and the invariants marked.

The upstream step matters most: a prospecting workflow that proves account data is fixable at the rep level is also the case for fixing it at the source. Replication isn't just horizontal (more reps); it's vertical (into the process that hands reps their books).
