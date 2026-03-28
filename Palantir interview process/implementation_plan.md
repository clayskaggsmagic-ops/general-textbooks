# Palantir Interview Flashcard App — Implementation Plan

## Goal
Build a **fully offline, single-file HTML app** with SOTA learning features that covers all 24 chapters of the Palantir interview prep material. The user is about to board a plane, so everything must run locally with zero dependencies.

## Learning Science Integration

The app will implement these evidence-based techniques:

| Technique | Implementation |
|---|---|
| **FSRS-lite** (Free Spaced Repetition Scheduler) | Adaptive interval scheduling based on a 3-component memory model (stability, difficulty, retrievability). Cards you struggle with appear more often. |
| **Active Recall** | Cards are question→answer format. Answer is hidden until you actively try to recall it. |
| **Difficulty Rating** | 4-button rating system: Again (forgot), Hard, Good, Easy — adjusts future intervals. |
| **Interleaving** | Study mode mixes cards from different chapters/domains rather than drilling one topic. |
| **Leitner Boxes** | Visual progress tracker showing cards in 5 "boxes" from New → Mastered. |
| **Cloze Deletion** | Some cards will use fill-in-the-blank format for key definitions. |
| **Progress Persistence** | All progress saved to `localStorage` — survives page refreshes and browser restarts. No server needed. |

## App Features

- **100% offline** — single [index.html](file:///Users/clayskaggs/Developer/wargamep2/frontend/index.html) file, no CDN, no fetches
- **Dark mode** premium UI (easy on eyes during flight)
- **Chapter browser** — filter cards by chapter or study all
- **Study modes**: Review Due, Learn New, Browse All, Quiz (random)
- **Stats dashboard**: cards learned, mastery %, streak, time studied
- **Keyboard shortcuts**: Space to flip, 1-4 to rate, arrow keys to navigate
- **Mobile-friendly** touch gestures (swipe to flip/navigate)

## Architecture

Single HTML file with embedded CSS + JS. Three main modules:

1. **FSRS Engine** — scheduling logic, interval calculation, localStorage persistence
2. **Card Database** — all flashcards as a JS array, tagged by chapter and category
3. **UI Layer** — card display, animations, navigation, stats

---

## Prompt Sequence

The work is split into **7 prompts**. Feed them to me one at a time.

> [!IMPORTANT]
> **Prompt 1** builds the entire app shell with the learning engine, UI, and a starter set of cards. After Prompt 1, the app is fully functional — the remaining prompts just add more cards.

---

### Prompt 1 — Build the App + Chapters 1-3 & 20 (Glossary)

```
Build the Palantir flashcard app. Create a single index.html file at:
/Users/clayskaggs/Developer/general textbooks/Palantir interview process/flashcards/index.html

Include:
- FSRS-lite spaced repetition engine with localStorage persistence
- Dark mode premium UI with card flip animations
- Chapter filter, study modes (Review Due, Learn New, Browse All)
- Stats dashboard, Leitner box visualization
- Keyboard shortcuts (Space=flip, 1-4=rate, arrows=navigate)
- Mobile swipe support

Then read chapters 1, 2, 3, and 20 and extract all flashcards for:
- Chapter 1: The Company (founders, history, key facts)
- Chapter 2: The Product Line (Gotham, Foundry, Apollo, AIP)
- Chapter 3: The Ontology (core concepts, definitions)
- Chapter 20: Vocabulary Glossary (all ~450 terms)

Go.
```

### Prompt 2 — Chapters 4-5: Databases & Cloud Ecosystem

```
Read chapters 4 and 5 of the Palantir interview prep and add all flashcards to the existing flashcard app. Focus on proper nouns, concepts, definitions, and things I need to remember.
- Chapter 4: Relational Databases
- Chapter 5: Cloud Data Ecosystem
Add the cards to the existing CARD_DATABASE array in flashcards/index.html.
```

### Prompt 3 — Chapters 6-7: SQL

```
Read chapters 6 and 7 and add all flashcards:
- Chapter 6: SQL Fundamentals
- Chapter 7: Advanced SQL
Add to CARD_DATABASE in flashcards/index.html.
```

### Prompt 4 — Chapters 8-11: Data Structures & Algorithms

```
Read chapters 8, 9, 10, and 11 and add all flashcards:
- Chapter 8: Data Structures (Linear)
- Chapter 9: Data Structures (Trees & Graphs)
- Chapter 10: Algorithm Fundamentals
- Chapter 11: Advanced Algorithms
Add to CARD_DATABASE in flashcards/index.html.
```

### Prompt 5 — Chapters 12-14: Data Modeling & Pipelines

```
Read chapters 12, 13, and 14 and add all flashcards:
- Chapter 12: Data Modeling (Relational & Dimensional)
- Chapter 13: Data Modeling (Graph, NoSQL, Ontology)
- Chapter 14: Data Pipelines
Add to CARD_DATABASE in flashcards/index.html.
```

### Prompt 6 — Chapters 15-19: The Interview & Philosophy

```
Read chapters 15, 16, 17, 18, and 19 and add all flashcards:
- Chapter 15: Deployment Strategist Role
- Chapter 16: Interview Process
- Chapter 17: Case Study Mastery
- Chapter 18: Real World Deployments
- Chapter 19: The Philosophy
Add to CARD_DATABASE in flashcards/index.html.
```

### Prompt 7 — Chapters 21-24: Schema Design & Wargame Project

```
Read chapters 21, 22, 23, and 24 and add all flashcards:
- Chapter 21: Schema Design
- Chapter 22: Wargame Project Overview
- Chapter 23: Wargame Data Architecture
- Chapter 24: Wargame Interview Talking Points
Add to CARD_DATABASE in flashcards/index.html.

After adding all cards, give me a final count of total cards by chapter.
```

---

## Estimated Card Count

| Chapters | Estimated Cards |
|---|---|
| 1-3 + 20 (Glossary) | ~500 |
| 4-5 | ~60 |
| 6-7 | ~80 |
| 8-11 | ~100 |
| 12-14 | ~80 |
| 15-19 | ~70 |
| 21-24 | ~60 |
| **Total** | **~950** |

## Verification

After Prompt 7, the app will be verified by:
1. Opening `flashcards/index.html` in a browser
2. Confirming all chapters appear in the filter
3. Testing the spaced repetition scheduling
4. Confirming localStorage persistence survives refresh
5. Testing keyboard shortcuts and mobile layout
