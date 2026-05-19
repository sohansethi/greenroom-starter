# Case Study Submission Notes

## What was built

**Feature: AI Deal Notes Interpreter**

An AI-assisted deal terms interpreter built into the Greenroom settlement page. It reads free-text deal notes, extracts key settlement terms, shows where each value came from, flags things that need human confirmation, and produces a copyable explanation for the tour manager.

The feature appears on the settlement page for any deal type the in-app calculator doesn't support (vs deals, percentage-of-net, door deals) — which is about 82% of The Crescent's shows.

---

## Where the main files are

| File | What it does |
|------|--------------|
| `lib/interpretDealNotes.ts` | Core extraction logic. Parses deal notes and structured deal fields, returns extracted terms, confidence levels, flags, and the tour manager summary. |
| `app/shows/[id]/settle/DealNotesInterpreter.tsx` | Client component. Renders all five sections of the interpreter UI. Handles copy-to-clipboard and the "mark reviewed" toggle. |
| `app/shows/[id]/settle/page.tsx` | Settlement page. Modified to import and render `DealNotesInterpreter` inside the `UnsupportedDeal` section when deal notes exist. |
| `AI_DEAL_NOTES_INTERPRETER_MEMO.md` | Product memo (1-2 pages). Covers slice choice, problem, design decisions, what was cut, AI/human-in-the-loop reasoning, metrics, and next steps. |
| `LOOM_SCRIPT.md` | Full walkthrough script for the Loom recording. |
| `CASE_STUDY_SUBMISSION_NOTES.md` | This file. |
| `IMPLEMENTATION_NOTES_FOR_SOHAN.md` | Plain-English notes explaining every file and design decision. |

No other files were modified. No new dependencies were added. No database migrations were needed.

---

## How to run the app

```bash
# From the repo root
npm install           # if you haven't already
npm run db:reset      # resets and re-seeds the database (takes ~10 seconds)
npm run dev           # starts Next.js on localhost:3000
```

Then go to:

1. `http://localhost:3000/shows` — the shows list
2. Click any show with a "Vs deal" badge, or find a show for "Coastal Spell" (it has the most interesting deal notes)
3. Click "Settle" in the show detail page
4. The AI Deal Notes Interpreter appears below the "What the system has" inputs card

If you want to see the interpreter with the richest data, look for the Coastal Spell show from March 14, 2025. It has a vs deal with a dispute note, a marketing recoup, and a bonus clause.

---

## Assumptions made

1. The interpreter is read-only for this prototype. In production, Mariana should be able to edit extracted terms before marking reviewed. That's called out in the memo as the next step.

2. The "mark terms reviewed" state is local React state. It resets on page refresh. In production this would write to the settlement record.

3. The extraction logic was tuned on vs deal patterns from the seed data. Other deal types (door deals, straight percentage-of-net) will still show the interpreter if they have notes, but fewer fields may extract successfully.

4. The interpreter appears for any deal type where `calc.supported === false`. That's currently vs, percentage_of_net, and door.

---

## Known limitations

- No live LLM call. Extraction is deterministic regex parsing. Notes that deviate from common patterns may produce fewer extracted terms. This is intentional for a prototype — see the memo for the rationale.
- The copy button uses `navigator.clipboard`, which requires HTTPS or localhost. It won't work in some server-render contexts or old browsers.
- The "mark reviewed" toggle is UI-only. Nothing is written to the database.
- If deal notes are very short or non-standard, the interpreter may only extract deal type and leave most fields blank.

---

## What to show in the Loom

**In order:**

1. Start on the shows list (`/shows`). Filter or scroll to find a vs deal show.
2. Click into the show, then click "Settle" to go to the settlement page.
3. Show the amber warning card — "The in-app tool can't settle a vs deal yet." This is the existing state.
4. Scroll down to the deal notes block — show the raw freetext.
5. Keep scrolling to the AI Deal Notes Interpreter section.
6. Walk through each section: raw note, extracted terms (explain the color badges), confidence flags, tour manager summary.
7. Click "Copy for tour manager" — show the clipboard feedback.
8. Click "Mark terms reviewed" — show the card switch to confirmed state.
9. Close by explaining what you cut and why, and what you'd ship next.

The Coastal Spell show is the best one to demo — it has the richest notes with a dispute, a bonus, and a marketing recoup all in one note.

---

## Talk track if asked why you chose this slice

"Settlement at The Crescent fails before the math. Mariana can't confidently settle a show when the actual deal terms live in prose and the structured fields are incomplete or stale. I chose the interpreter because it addresses that root cause: making the notes legible, auditable, and reviewable. Once the terms are confirmed, the calculator problem is much more straightforward to solve. The interpreter is the higher-leverage slice because it unblocks the whole workflow, not just the arithmetic."
