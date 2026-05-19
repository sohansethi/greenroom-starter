# Implementation Notes for Sohan

These notes are for your personal understanding and interview prep. They explain what was built, how it works, and what to say if you get technical questions.

---

## What files were changed or created

### New files

**`lib/interpretDealNotes.ts`**
This is the brain of the feature. It takes two things as input: the deal notes freetext (that paragraph Mariana writes), and the structured deal fields from the database (guarantee amount, percentage, caps, etc). It parses the notes using pattern matching and returns a structured result with extracted terms, source labels, confidence levels, and flags.

**`app/shows/[id]/settle/DealNotesInterpreter.tsx`**
This is the visual component. It's what the user actually sees. It receives the result from the extraction function and renders it as a series of cards: raw note, extracted terms table, review flags, tour manager summary, and the "mark reviewed" button.

**`AI_DEAL_NOTES_INTERPRETER_MEMO.md`**
The product memo for the case study submission.

**`LOOM_SCRIPT.md`**
The script for the Loom recording.

**`CASE_STUDY_SUBMISSION_NOTES.md`**
Handoff notes covering what was built, how to run it, and what to demo.

**`IMPLEMENTATION_NOTES_FOR_SOHAN.md`**
This file.

### Modified files

**`app/shows/[id]/settle/page.tsx`**
Two small changes:
1. Added an import for `DealNotesInterpreter` at the top of the file.
2. Inside the `UnsupportedDeal` function (which handles vs deals, percentage-of-net deals, etc.), added the interpreter component right after the "What the system has" card. It only renders if there are deal notes to parse.

---

## Why each file was changed

**`page.tsx`** needed to be modified because that's where the settlement page lives and where the interpreter needs to appear. The change is minimal — just an import and a render call. The rest of the page is untouched.

**`interpretDealNotes.ts`** was created as a separate utility file instead of putting the logic inside the component. This is a common pattern in this codebase (see `dealMath.ts`, `settlementStage.ts`). It keeps the logic testable and separate from the UI.

**`DealNotesInterpreter.tsx`** needed to be a "use client" component (a Next.js concept meaning it runs in the browser, not on the server) because it needs two things only available in the browser: the clipboard API (for the copy button) and React state (for the "mark reviewed" toggle).

---

## How the feature works, step by step

1. The settlement page loads for a show with a vs deal.

2. The `calculateSettlement` function returns `{ supported: false }` because vs deals aren't supported by the calculator.

3. The page renders the `UnsupportedDeal` section, which shows the amber warning, the inputs, and the deal notes.

4. If deal notes exist, the `DealNotesInterpreter` component is also rendered.

5. The component calls `interpretDealNotes(notes, deal)`. This function:
   - Looks for dollar amounts near "guarantee" or "vs" in the notes
   - Looks for percentage patterns like "80% of net"
   - Checks if those values match the structured deal fields
   - Looks for cap amounts, bonus clauses, recoup mentions, dispute flags
   - Assigns a source tag (notes/field/both) and confidence (high/medium/low) to each finding
   - Generates a tour manager summary paragraph
   - Returns all of this as a structured object

6. The component renders each section using this returned data.

7. The "Mark terms reviewed" button uses React's `useState` hook to toggle between confirmed and unconfirmed. It's purely in the browser — nothing is saved to the database.

8. The "Copy" button uses `navigator.clipboard.writeText()` to put the tour manager summary on the user's clipboard.

---

## What is mocked versus real

**Mocked (fake but functional):**
- The extraction logic in `interpretDealNotes.ts`. It uses regex pattern matching instead of calling a real AI model. The patterns were written to work well against the seed data (especially the Coastal Spell deal).
- The "mark reviewed" state. It's local React state. Refresh the page and it resets.

**Real (actually works):**
- The UI renders based on real data from the database. The notes and deal fields shown are whatever's in the actual database for that show.
- The copy button actually copies to clipboard.
- The source badges and confidence levels are computed from the real comparison between notes and structured fields.
- All existing settlement functionality is unchanged.

---

## How to run and test it

```bash
npm run db:reset   # clears and re-seeds the database
npm run dev        # starts the app at localhost:3000
```

To see the interpreter in action:

1. Go to `http://localhost:3000/shows`
2. Find a show with a "Vs deal" badge — there are many
3. Click into the show, then click the "Settle" link
4. The interpreter appears below the "What the system has" card

The best show to demo: search for "Coastal Spell" — it's the March 2025 show with a dispute history. The notes have a guarantee, a percentage, expense cap, hospitality cap, a bonus, a recoup, and a dispute note all in one paragraph. The interpreter extracts most of them.

To test with different deals, you can look at any vs deal or percentage-of-net deal. Door deals will also show the interpreter if they have notes.

---

## What to say if asked about technical tradeoffs

**"Why is this a client component?"**
The feature needs browser-only capabilities: the clipboard API for the copy button, and `useState` for the mark-reviewed toggle. In Next.js, you can only use these in "client components." Server components can't run JavaScript in the browser — they just render HTML on the server. The rest of the settlement page is a server component because it just shows data.

**"Why put the extraction logic in a separate file?"**
It's cleaner and follows the pattern already in the codebase. The other business logic files (`dealMath.ts`, `settlementStage.ts`) are also separate from the page components. It also makes it easier to swap in a real AI call later — you just change the function body and nothing else has to change.

**"Why is it inside `UnsupportedDeal` instead of a separate page?"**
Because the interpreter is an enhancement to an existing workflow, not a replacement for it. The existing "unsupported deal" cards stay — they show the actual numbers Mariana would use. The interpreter adds interpretation on top of that, in the same place where Mariana is already looking. It felt wrong to put it somewhere separate.

**"What happens if the notes don't match any patterns?"**
The function returns fewer extracted terms. The UI still renders — you'd see the raw note, maybe just the deal type term, and whatever flags match (like if the word "disputed" appears). It degrades gracefully rather than crashing.

---

## What to say if asked why there is no real AI API integration

Short version: "The product decisions I wanted to demonstrate are independent of whether the extraction is done by regex or an LLM. The UX, the audit trail, the review flow, the source attribution — those are all identical either way. I chose not to add API latency, cost, and infrastructure complexity to a prototype where the point is the product judgment."

Longer version if they push: "In production, I'd replace the parsing function with a structured Claude prompt. The prompt would say: here's a deal note, here's the structured deal data, return me a JSON object with these fields — extracted terms, source reasoning, confidence levels, flags. The schema coming back is exactly what `interpretDealNotes.ts` already returns. The entire UI works without any change. The reason to use a real model is for coverage on notes that don't match common patterns — unusual phrasing, contracts pasted in as prose, etc. The regex works well on the seed data, but it would break on enough edge cases that you'd want a model in production."

---

## Things to know before the Loom recording

1. The Coastal Spell vs deal is the best show to demo. It has the richest notes.

2. The interpreter renders below the deal notes freetext block. You'll need to scroll a bit.

3. When you click "Mark terms reviewed," the card switches to brand green confirmed state. That's the visual payoff — make sure to let it land on camera.

4. The copy button shows a checkmark for 2 seconds after you click it, then resets. Small detail but looks polished on video.

5. If someone asks "why not just build the full vs calculator?" — the answer is in the memo. The short version: "The calculator fails on uncertain inputs. If I don't know whether the guarantee is $5,000 or $3,500 because the notes were renegotiated, computing 80% of net on top of that doesn't help. The interpreter solves the step before the math."
