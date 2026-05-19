# Making Deal Notes Interpretable at 2 A.M.

**Feature Name:** AI Deal Notes Interpreter
---

## 1. The Slice I Chose

I built an AI Deal Notes Interpreter that reads messy free-text deal notes, extracts key settlement terms, flags ambiguous or risky clauses, and produces a copyable explanation for the tour manager — all with explicit attribution showing where each value came from.

I did not build a full settlement calculator. That distinction matters, and I'll explain why below.

---

## 2. Why This Slice

The brief is clear about where the real problem lives: "real deal terms often live in messy deal notes rather than clean structured fields." The `dealNotesFreetext` field is described as the source of truth Mariana actually trusts.

That means the problem is not just that the calculator doesn't support vs deals. It's that even if you built a perfect vs calculator, Mariana would still have to manually read ambiguous prose, figure out which numbers to trust, and translate the deal into something she can hand to a tour manager — at 2 a.m., after the show.

The settlement problem starts before the math. You can't confidently plug numbers into a formula when you're not sure which number is current.

This is why the interpreter is the right slice. It tackles the actual bottleneck: turning "notes that only Mariana understands" into something reviewable, explainable, and shareable. Once terms are confirmed, the calculator work becomes much more tractable.

---

## 3. The User Problem

Mariana settles shows late at night. Most of her deals are vs deals or percentage-of-net deals — the two types the in-app tool doesn't support. She defaults to Google Sheets because the tool can't handle her actual deal structure.

But the spreadsheet problem is downstream of the interpretation problem. Before she can do any math, she has to:

1. Find the original deal email or contract
2. Read through notes that may have been updated by phone or email with no clear versioning
3. Decide which number to use when the structured field and the prose disagree
4. Mentally reconstruct the settlement logic from prose
5. Produce a clear explanation for a tour manager who may not be in a good mood

That's where trust breaks down. Not in the math itself — in the step before it.

---

## 4. What I Built

The feature lives on the existing settlement page (`/shows/[id]/settle`) and only appears for deal types the in-app calculator doesn't support. It consists of five sections:

**Raw Deal Note** — Shows exactly what was read. No hiding the source material.

**Extracted Terms** — A structured table of what the interpreter found: deal type, guarantee, artist percentage, basis (net/gross), expense cap, hospitality cap, bonus clauses, recoup mentions. Each row shows the value, where it came from (notes, structured field, or both), and a per-field confidence level.

**Confidence Flags** — Specific warnings Mariana should check. Things like "structured percentage field is missing but notes mention 80%" or "notes mention a renegotiation — treat notes as primary." Flags are categorized as warnings (need action) versus informational (worth knowing).

**Tour Manager Summary** — A plain-English paragraph Mariana can copy and send. It synthesizes the extracted terms into something readable. If there are unresolved flags, the summary notes that items are pending confirmation.

**Human Review CTA** — A "Mark terms reviewed" button that lets Mariana confirm she's looked at this before settlement moves forward. It's local UI state — no backend needed for a prototype. In production this would write a timestamp and the reviewer's name to the settlement record.

---

## 5. Design Decisions

**Source attribution on every field.** The brief explicitly calls out that the data is messy and trust is the real issue. So every extracted term shows a badge: "Notes + field" (sky), "From notes" (amber), "Deal field" (brand green), or "Assumed" (rose). Mariana can see at a glance which values she needs to double-check.

**Field-level confidence alongside overall confidence.** Showing one overall "medium" confidence rating isn't useful. Mariana needs to know which specific fields are uncertain so she can investigate those, not reread the whole note.

**The copy button exists for a real workflow reason.** Tour managers ask for settlement breakdowns in writing. Mariana currently has to compose these by hand. A pre-generated, copyable summary that synthesizes the deal terms saves real time and reduces the chance of miscommunication.

**Using the existing amber-card "unsupported deal" section as context.** The interpreter doesn't replace the warning — it sits below it and takes action on the problem it names. This keeps the information hierarchy clear.

**No real AI API call.** The extraction uses deterministic regex-based parsing that produces the same output format a real LLM prompt would return. The UI, the review flow, and the audit trail are the product decisions being demonstrated here — not the NLP sophistication. Calling a real API would have added latency, cost, and infrastructure concerns that would have distracted from the core product judgment.

---

## 6. What I Intentionally Cut

**Full settlement calculator for vs deals.** Building the actual vs deal math would have been a large, separate feature. The interpreter is about making the terms legible; the calculator is about applying them. These are distinct problems and solving them together in one prototype would have blurred the focus.

**Dispute resolution flow.** The brief mentions disputes and the seed data has an active disputed settlement (The Coastal Spell / WME case). That's a full workflow involving back-and-forth with the agent, revised recoup items, and status transitions. Not appropriate scope here.

**Editing / correcting extracted terms.** In a real version, Mariana would be able to edit the interpreter's output before marking it reviewed. That's important for production. For the prototype, the review CTA communicates the intent clearly enough.

**Database writes for the "mark reviewed" state.** The review toggle is local UI state. In production it would need to write to the settlement record (who reviewed, when, what they confirmed) to create a real audit trail.

**Confidence scores from an actual model.** The per-field confidence levels in this prototype are heuristic (did the value appear in both places? did the notes mention a renegotiation?). A real implementation would get confidence scores from the model itself, and those would be more calibrated.

---

## 7. AI and Human-in-the-Loop Reasoning

The design principle here is: the AI surfaces, the human confirms.

Mariana should never be in a position where she's trusting a settlement because the AI said so. She needs to understand what the AI found, where it found it, and where it's uncertain. That's why source attribution and per-field confidence are load-bearing features, not decorative ones.

The "Mark terms reviewed" CTA is also deliberate. It creates a checkpoint. Settlement is a consequential action — there's money changing hands, there's a relationship with an artist and their team, and errors have real downstream effects. The right UX puts Mariana explicitly in control of the step that says "yes, these terms are what I agreed to."

One practical thing this design handles: notes that conflict with structured fields. The interpreter doesn't silently prefer one over the other. It surfaces the conflict and flags it. Mariana decides which is current. That's the right call because the system can't know if the structured field is stale from a renegotiation, or if the notes contain a typo.

---

## 8. Validation Metrics

If this were a real feature, I'd track:

**Adoption rate** — How many settlement workflows for unsupported deal types include a "mark reviewed" event? If Mariana uses it, it's working.

**Flag resolution rate** — When the interpreter surfaces a warning, how often does Mariana take a confirming action (e.g., check the recoup section, adjust a number)? A high rate means the flags are calibrated well. A low rate means they're noisy.

**Time to settlement for vs deals** — Does the feature reduce the wall-clock time from "show ends" to "settlement sent"? This is the core efficiency claim.

**Tour manager summary usage** — How often is the copy button used? If it's rarely touched, the summary format may need tuning, or tour managers are getting the info another way.

**Confidence calibration** — For fields where the interpreter says "high confidence," how often does Mariana change the value during review? If she's changing high-confidence fields, the confidence scoring is wrong.

---

## 9. What I Would Ship Next

**Step 1: Wire in a real LLM call.** Replace the regex parser with a structured prompt to Claude. The output schema stays the same — the UI doesn't change. The gain is handling notes that don't match common patterns (unusual phrasing, non-English deals, contracts pasted in as prose).

**Step 2: Editable extracted terms.** Let Mariana correct the interpreter's output inline, before marking reviewed. Each edit gets logged with a source tag of "manual correction" so the audit trail shows where human judgment overrode the extraction.

**Step 3: Persist the review state.** Write the "mark reviewed" action to the settlement record with a timestamp and user ID. This turns the CTA into a real audit trail that the GM and the artist team can trust.

**Step 4: Settlement draft generation.** Once terms are confirmed, offer to pre-fill a settlement draft with the extracted numbers. This is the bridge between "terms are legible" and "settlement is done."

**Step 5: Confidence improvement loop.** After each settlement is finalized, compare what the interpreter extracted against what Mariana confirmed or corrected. Feed that signal back to improve the prompt or the parsing heuristics over time.

---
