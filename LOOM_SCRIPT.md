# Loom Script — AI Deal Notes Interpreter
**Target length:** 6-8 minutes
**Tone:** Confident, conversational, practical. Like you're showing a colleague something you built and actually care about.

---

## [0:00 - 1:00] Problem framing

"I want to start by showing you the actual problem before I show you what I built.

This is Greenroom's settlement page for a show at The Crescent — a 650-cap venue in Nashville. This is a vs deal, which is the most common deal structure in independent music venues. Guarantee versus percentage of net, whichever is greater.

And this is what the page looks like right now."

[Show the amber warning card: "The in-app tool can't settle a vs deal yet."]

"The tool just... gives up. It tells Mariana — the lead booker — to go do this in a spreadsheet.

Now scroll down and look at the deal notes."

[Show the dealNotesFreetext block]

"This is the note Mariana actually trusts. The real deal is in here — the guarantee, the percentage split, the expense cap, the hospitality rules, a renegotiation flag, and a dispute note. It's not structured. It's not clean. But it's real.

The problem isn't just that the calculator doesn't work. It's that even before you get to the math, Mariana has to read this note at 2 a.m., figure out which numbers to use, reconcile it with whatever's in the structured fields, and translate all of that into something she can hand to a tour manager who's waiting for payment.

That's the slice I chose."

---

## [1:00 - 1:45] Why this slice and why not a full calculator

"I could have built a vs deal calculator. That would have been the obvious thing. But I made a deliberate call not to.

The settlement problem starts before the math. Before Mariana can plug numbers into any formula, she needs to be confident about what the deal actually says. And when the notes conflict with the structured fields — which happens — there's no system helping her figure that out.

An interpreter that makes the deal terms legible and auditable is higher leverage than a calculator that computes the right answer on top of uncertain inputs.

The calculator becomes much easier to build once this is solved."

---

## [1:45 - 4:30] Prototype walkthrough

"Let me show you what I built."

[Scroll to the AI Deal Notes Interpreter section]

"So right below the existing inputs card, the interpreter kicks in whenever there are deal notes to parse. It does not replace the warning — it takes action on it.

**Section one: Raw deal note.**"

[Point to the raw note card]

"I show the raw note in full, with a 'notes_freetext' source badge. Mariana needs to see what the interpreter read — no surprises.

**Section two: Extracted terms.**"

[Point to the extracted terms card]

"This is the core of the feature. Every term the interpreter found is displayed here with three pieces of information: the value it extracted, where it came from, and how confident it is.

The source badges tell the story. Sky means it found the value in both the notes and the structured deal field — those are your most reliable extractions. Amber means it came from the notes only. Green means it came from the structured field. Rose would mean it was assumed.

Look at the guarantee: $5,000, confirmed against the structured field, high confidence. Look at the artist percentage: 80%, from notes only, medium confidence — because the structured percentage field happens to be missing. Look at the expense cap: $2,500, confirmed in both places, high confidence.

**Section three: Flags.**"

[Point to the flags card]

"This is where the interpreter flags what Mariana needs to look at. There are two types: warnings and informational notes.

A warning is something that should be resolved before settlement moves forward. Here you can see things like: 'Structured percentage field is missing — using notes value, verify before final settlement.' And: 'Dispute history found in notes — confirm all contested items are resolved.'

These are not generic alerts. They're specific to what the interpreter actually found in this note."

---

## [4:30 - 5:30] Human review and audit trail

"Let me talk about the two sections that I think matter most from a product standpoint.

**The tour manager summary.**"

[Point to the tour manager summary card]

"This is a pre-generated paragraph Mariana can copy and send. It synthesizes the extracted terms into plain English. She doesn't have to write this from scratch.

But — and this is important — the summary notes where things are still pending. If there are unresolved flags, the summary says so. It doesn't pretend everything is clean.

There's a copy button. When Mariana clicks it, it copies to clipboard. Simple, but saves real time.

**The human review CTA.**"

[Point to the CTA card]

"And then this. Before settlement can move forward, Mariana explicitly marks the terms as reviewed. This is the human in the loop. The AI surfaced the interpretation — but Mariana is the one who says 'yes, this is what I agreed to.'

When she clicks it, the card switches to confirmed state, the brand green accent appears, and she gets a 'Ready for settlement review' indicator."

[Click 'Mark terms reviewed']

"In production, this would write a timestamp and her user ID to the settlement record. You'd see who reviewed the terms and when. That's a real audit trail that the GM and the artist's team can trust."

---

## [5:30 - 6:15] How the AI assistance works

"One thing I want to be transparent about: there is no live API call here. The extraction runs on deterministic parsing — regex patterns that look for dollar amounts, percentages, key phrases.

I made that call deliberately. The point of this prototype is to show the product design: what information to surface, how to attribute it, what to flag, and how to keep the human in control. That doesn't require a live LLM call to demonstrate.

In production, you'd swap the parser for a structured prompt to Claude. The prompt would say: 'Read this deal note and return JSON with these fields, confidence levels, and flags.' The output schema is identical. The UI, the review flow, the audit trail — none of that changes.

The reason to use a real model is handling notes that don't fit patterns — unusual phrasing, contracts pasted as prose, non-English text. The regex works well for the seed data, but it would break on enough edge cases that you'd want a model in production."

---

## [6:15 - 7:00] Tradeoffs

"A few tradeoffs I made and would own in an interview:

The interpreter is read-only right now. Mariana can review the extracted terms, but she can't correct them inline. In production, every field should be editable before she marks reviewed. Each edit would get a 'manual correction' source tag so the audit trail is accurate.

The 'mark reviewed' state is local React state — it doesn't persist if you refresh. That's fine for a prototype but would need to write to the settlement record in production.

I also didn't build the bridge to the calculator. The logical next step would be: once terms are confirmed, the interpreter pre-fills those values into the settlement calculation. Right now those are separate.

And I didn't handle every deal type. The interpreter appears for all unsupported deal types, but it was tuned on vs deals. A percentage-of-net deal or a door deal might produce different patterns that need their own parsing rules."

---

## [7:00 - 7:45] Validation and what I'd ship next

"If I were taking this to production, here's how I'd validate it:

First metric: adoption. How many settlement workflows for unsupported deal types include a 'mark reviewed' event? If Mariana uses the feature, it's doing its job.

Second: flag resolution rate. When the interpreter surfaces a warning, does Mariana take a confirming action? High rate means the flags are useful. Low rate means they're noise.

Third: time to settlement for vs deals. Does this reduce the wall-clock time from show-end to settlement-sent?

For next steps — in priority order:

One: real LLM call. Same schema, same UI. You gain coverage on edge cases.

Two: editable terms. Let Mariana correct the output before marking reviewed.

Three: persist the review state with timestamp and user ID.

Four: bridge to the calculator — once terms are confirmed, pre-fill the settlement draft.

The point is that this feature doesn't have to be perfect at launch to be useful. Even a version where Mariana sees her deal terms structured and attributed, reviews the flags, and copies the summary — that's saving her real time at 2 a.m. compared to the current state of 'go do it in a spreadsheet.'"

---

## [7:45 - 8:00] Close

"That's the feature. The core thesis is that settlement at The Crescent starts with an interpretation problem, not a calculation problem. This prototype addresses the interpretation step: making the deal terms legible, auditable, and reviewable before any math happens.

Thanks for watching."

---

**Notes for recording:**
- Keep the pace natural. Don't rush through the UI sections — let the audience read what's on screen.
- When you show the source badges, pause and explain what each color means. That's the key UX insight.
- The "Mark terms reviewed" click is the climactic moment — let it land.
- If you're asked about no real API: "The parsing is deterministic — same patterns you'd use to prompt a model. The product decisions are identical. I chose not to add API latency and cost to a prototype where the point is the UX, not the NLP."
