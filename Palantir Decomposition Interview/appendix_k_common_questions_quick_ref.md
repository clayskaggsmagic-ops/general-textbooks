# Appendix K: Common Interview Questions — Quick Reference Card

> Use this as a last-minute refresher. For each question type, scan the key moves and the chapter where the full answer framework lives.

---

## Technical Comprehension (Chapter 19)

| They Ask... | Your Move |
|-------------|-----------|
| "I'm going to teach you [concept]. Ready?" | Listen fully. Restate in your own words. Ask a boundary question before solving. |
| "Walk me through this SQL query." | Read it clause-by-clause. Start with FROM, then WHERE, then SELECT. Name the output. |
| "What does this window function do?" | Explain PARTITION BY (the grouping) and ORDER BY (the sorting). Trace through 2-3 rows. |
| "Can you optimize this?" | Get brute force working first. Then ask: "Would you like me to optimize for speed or readability?" |
| "I'm stuck / I don't know" | Name what's blocking you: "I'm not sure about [specific thing]. Can I get a hint about [specific aspect]?" |

**Golden rule:** Think out loud. Silence is your enemy.

---

## Open-Ended Decomposition (Chapters 21–22)

| They Ask... | Your Move |
|-------------|-----------|
| "Help [organization] solve [vague problem]." | Restate. Ask 4-6 clarifying questions. Identify stakeholders, data, constraints. |
| "What data would you need?" | Name specific Object Types, Properties, and Links. Use Ontology language. |
| "What would the user see on Day 1?" | Describe the Workshop app: what's on the screen, what buttons exist, what actions they trigger. |
| "What if you don't have that data?" | Propose a proxy. Design for resilience. "If we can't get X, we could use Y as an approximation." |
| "That's too complicated." | Simplify the user surface. Keep the depth underneath. "For v1, I'd show just [subset]." |
| "What would you do first?" | "Data audit + stakeholder interviews. I need to understand what exists before I build anything." |
| "How would you know if this is working?" | North Star Metric → Usage metrics → Data health metrics. Name leading and lagging indicators. |
| "How would you roll this out?" | Phase 1 (prove it at one site) → Phase 2 (expand with actions) → Phase 3 (org-wide + AI). |

**Golden rule:** You're designing an operating system someone uses daily, not writing a recommendation deck.

---

## Analytical Problem Solving (Appendix J)

| They Ask... | Your Move |
|-------------|-----------|
| "How many [X] are there in [Y]?" (Fermi) | Break → Estimate each piece → Multiply → Sanity-check. State assumptions. |
| "How would you measure the success of [system]?" | North Star + 3-level metrics stack (data health → usage → outcome). |
| "This data shows [pattern]. What do you see?" | Describe first (no conclusions). Generate 3+ hypotheses. Say what data you'd need to distinguish. Recommend. |
| "What would you do with a $500K budget?" | Name 2-3 options with trade-offs. Recommend one with rationale. Acknowledge what you're giving up. |
| "This metric went up/down — why?" | Don't jump to one explanation. "Three possible causes: [A], [B], [C]. I'd check [data] to distinguish." |

**Golden rule:** Structure beats precision. A well-organized estimate beats a lucky guess.

---

## Behavioral / "Tell Me About..." Questions

| They Ask... | Your Move |
|-------------|-----------|
| "Why Palantir?" | Consequence + proximity. "I want to solve hard, real-world problems, and the forward-deployed model puts me closest to the impact." |
| "Tell me about a time you failed." | Own it. What happened → what you did → what you learned. No blame. |
| "How do you handle ambiguity?" | "I impose structure: break the problem down, prioritize, propose a direction, and iterate on feedback." |
| "What if users refuse to adopt your tool?" | Shadow their workflow → quick win in Week 1 → co-design with power users. (See Ch04, §4.10) |
| "What questions do you have for me?" | Ask about their deployment. "What's been the hardest part of your current engagement?" |

**Golden rule:** Every answer should end with what you *learned* or what you'd *do differently*.

---

## The 10-Second Pre-Interview Checklist

```
Before you join the call:

□ Review the Five Lenses (Ch00): Users, Data, Logic, Surfaces, Deployment
□ Know your Ontology vocabulary: Object Types, Properties, Links, Actions
□ Have 2-3 "Tell me about a time..." stories ready
□ Remember: Think out loud. Ask questions. Take hints. Name trade-offs.
□ Breathe. They want you to succeed.
```
