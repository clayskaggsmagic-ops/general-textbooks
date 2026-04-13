# PROMPT: Restructure the Palantir DS Interview Textbook

You are restructuring a Palantir Deployment Strategist interview textbook. The original folder is at:
`/Users/clayskaggs/Developer/general textbooks/Palantir interview process/`

Create a **new folder** at:
`/Users/clayskaggs/Developer/general textbooks/Palantir Decomposition Interview/`

**Do NOT modify** the original folder. Copy files to the new folder, renaming and updating headers as specified below.

---

## Context: The Three Interview Rounds

From the recruiter email, the DS interview has three rounds:

1. **Technical Comprehension** — Interviewer teaches a new technical concept, you solve problems of increasing complexity. SQL basics recommended.
2. **Open-Ended Decomposition** — Abstract problem, break it down, design a solution. No coding. You make assumptions about data.
3. **Analytical Decomposition** — Same structure but with data constraints and rails. May require pseudocode.

> **CRITICAL**: The recruiter says: "These are NOT consulting case studies — we are not looking for you to apply consulting frameworks."

---

## File Mapping

For each file: copy to new folder → rename → update the `# Chapter XX:` header line inside the file to the new number → update any internal cross-references ("see Chapter XX") to point to the correct new numbers.

### Part I: Palantir & The Role

| New Filename | Source Filename |
|---|---|
| `chapter_01_the_company.md` | `chapter_1_the_company.md` |
| `chapter_02_the_product_line.md` | `chapter_2_the_product_line.md` |
| `chapter_03_the_ontology.md` | `chapter_3_the_ontology.md` |
| `chapter_04_deployment_strategist_role.md` | `chapter_15_deployment_strategist_role.md` |

### Part II: Data Foundations

| New Filename | Source Filename |
|---|---|
| `chapter_05_relational_databases.md` | `chapter_4_relational_databases.md` |
| `chapter_06_cloud_data_ecosystem.md` | `chapter_5_cloud_data_ecosystem.md` |
| `chapter_07_sql_fundamentals.md` | `chapter_6_sql_fundamentals.md` |
| `chapter_08_advanced_sql.md` | `chapter_7_advanced_sql.md` |
| `chapter_09_data_structures_linear.md` | `chapter_8_data_structures_linear.md` |
| `chapter_10_data_structures_trees_graphs.md` | `chapter_9_data_structures_trees_graphs.md` |
| `chapter_11_algorithms_fundamentals.md` | `chapter_10_algorithms_fundamentals.md` |

### Part III: Data Modeling & Architecture

| New Filename | Source Filename |
|---|---|
| `chapter_12_data_modeling_relational_dimensional.md` | `chapter_12_data_modeling_relational_dimensional.md` |
| `chapter_13_data_modeling_graph_nosql_ontology.md` | `chapter_13_data_modeling_graph_nosql_ontology.md` |
| `chapter_14_data_pipelines.md` | `chapter_14_data_pipelines.md` |

### Part IV: System Design

| New Filename | Source Filename |
|---|---|
| `chapter_15_system_design_fundamentals.md` | `chapter_28_system_design_fundamentals.md` |
| `chapter_16_system_design_building_blocks.md` | `chapter_29_system_design_building_blocks.md` |
| `chapter_17_system_design_practice.md` | `chapter_30_system_design_practice.md` |

### Part V: The Three Interview Rounds

| New Filename | Source Filename |
|---|---|
| `chapter_18_interview_process.md` | `chapter_16_interview_process.md` |
| `chapter_19_technical_comprehension.md` | **NEW — write from scratch (see below)** |
| `chapter_20_seven_step_framework.md` | `chapter_17_case_study_mastery.md` |
| `chapter_21_open_ended_decomposition_playbook.md` | `chapter_26_decomposition_playbook.md` |
| `chapter_22_open_ended_decomposition_practice.md` | `chapter_27_decomposition_practice_lab.md` |
| `chapter_23_analytical_decomposition.md` | `chapter_21_schema_design.md` |
| `chapter_24_the_philosophy.md` | `chapter_19_the_philosophy.md` |
| `chapter_25_real_world_deployments.md` | `chapter_18_real_world_deployments.md` |

### Part VI: Your Projects & Portfolio

| New Filename | Source Filename |
|---|---|
| `chapter_26_wargame_overview.md` | `chapter_22_wargame_project_overview.md` |
| `chapter_27_wargame_data_architecture.md` | `chapter_23_wargame_data_architecture.md` |
| `chapter_28_wargame_talking_points.md` | `chapter_24_wargame_interview_talking_points.md` |
| `chapter_29_siren_system_architecture.md` | `chapter_24_siren_system_architecture.md` |
| `chapter_30_siren_classification_engine.md` | `chapter_25_siren_classification_engine.md` |

### Appendices

| New Filename | Source Filename |
|---|---|
| `appendix_a_glossary.md` | `chapter_20_vocabulary_glossary.md` |
| `appendix_b_consulting_case_fundamentals.md` | `chapter_25_consulting_case_fundamentals.md` |
| `appendix_c_consulting_case_playbook.md` | `chapter_32_consulting_case_strategy.md` |
| `appendix_d_wargame_data_audit.md` | `chapter_28_wargame_data_layer_forensic_audit.md` |
| `appendix_e_wargame_data_redesign.md` | `chapter_29_wargame_optimal_data_redesign.md` |
| `appendix_f_wargame_narrative.md` | `chapter_30_wargame_interview_narrative.md` |
| `appendix_g_wargame_followup.md` | `chapter_31_wargame_followup_qa.md` |
| `appendix_h_siren_frontend.md` | `chapter_26_siren_frontend_data_lifecycle.md` |
| `appendix_i_siren_database.md` | `chapter_27_siren_database_layer.md` |

### Files to SKIP (leave in original folder only)

- `chapter_11_algorithms_advanced.md` — out of scope for DS
- `answers.md`, `prompts.md`, `questions.md`, `todo.md`, `later.md` — scratch files
- `flashcard_prompts.md`, `flashcards/` — tooling
- `implementation_plan.md`, `wargame_data_redesign_prompts.md` — old planning
- `restructure_prompt.md` — this file

---

## New Chapter to Write: Chapter 19 — Technical Comprehension

Write `chapter_19_technical_comprehension.md` from scratch. This chapter prepares for the Technical Comprehension interview round. Structure:

### What the Round Is
- Interviewer introduces a technical concept you've never seen
- You solve problems of increasing complexity using that concept
- You're NOT expected to know the material in advance
- It's a collaborative discussion, not a test of memorized knowledge

### The Learning Ladder Approach
1. **Listen actively** — let the interviewer explain without interrupting
2. **Restate in your own words** — "So if I understand correctly, [X] works by..."
3. **Ask boundary questions** — "What happens at the edges? What if [extreme case]?"
4. **Build a mental model** — draw/sketch the concept, create analogies
5. **Apply to simple case first** — solve the easiest version before scaling up
6. **Extend to harder cases** — layer on complexity incrementally

### How to Ask Great Questions
- "Can you give me an example of how this would work with [concrete scenario]?"
- "Is this similar to [known concept]? Where does the analogy break down?"
- "What's the expected behavior when [edge case]?"
- "Am I on the right track, or should I reconsider [assumption]?"

### When to Pivot Approaches
- If your current approach stalls for >2 minutes, say so: "I think I'm stuck on this approach — let me try a different angle"
- There are usually multiple valid solutions
- Pivoting shows intellectual flexibility, not weakness

### SQL as the Likely Technical Domain
- The recruiter recommends SQL basics
- Review: SELECT, WHERE, JOIN, GROUP BY, ORDER BY, aggregate functions
- Be ready to learn a *new* SQL concept live (window functions, CTEs, recursive queries) and apply it
- Practice: "Here's a concept called a CTE (Common Table Expression). It lets you... Now use it to solve this problem."

### Practice Format
Include 2-3 exercises in this format:
> "You've never seen [concept X]. Here's how it works: [brief explanation]. Now solve this problem using it."

Use the same tone and depth as the rest of the textbook — plain English first, technical terms in parentheses, concrete examples, no fluff.

---

## Cross-Reference Update Map

When updating internal cross-references, use this old→new mapping:

| Old Reference | New Reference |
|---|---|
| Chapter 1 | Chapter 1 |
| Chapter 2 | Chapter 2 |
| Chapter 3 | Chapter 3 |
| Chapter 4 | Chapter 5 |
| Chapter 5 | Chapter 6 |
| Chapter 6 | Chapter 7 |
| Chapter 7 | Chapter 8 |
| Chapter 8 | Chapter 9 |
| Chapter 9 | Chapter 10 |
| Chapter 10 | Chapter 11 |
| Chapter 12 | Chapter 12 |
| Chapter 13 | Chapter 13 |
| Chapter 14 | Chapter 14 |
| Chapter 15 | Chapter 4 |
| Chapter 16 | Chapter 18 |
| Chapter 17 | Chapter 20 |
| Chapter 18 | Chapter 25 |
| Chapter 19 | Chapter 24 |
| Chapter 20 | Appendix A |
| Chapter 21 | Chapter 23 |
| Chapter 22 | Chapter 26 |
| Chapter 23 | Chapter 27 |
| Chapter 25 (consulting) | Appendix B |
| Chapter 26 (decomposition) | Chapter 21 |
| Chapter 27 (decomposition) | Chapter 22 |
| Chapter 28 (system design) | Chapter 15 |
| Chapter 29 (system design) | Chapter 16 |
| Chapter 30 (system design) | Chapter 17 |
| Chapter 32 | Appendix C |

---

## Execution Order

1. Create the new folder
2. Copy and rename all files per the mapping tables above (use a shell loop)
3. Update the `# Chapter XX:` header in each copied file
4. Update internal cross-references using the mapping table
5. Write Chapter 19 from scratch
6. Create a `README.md` table of contents in the new folder
7. Verify: count files (30 chapters + 9 appendices + README = 40 files), grep for stale chapter references

**Do this in batches to manage context.** Start with Parts I–III (copy + rename + header update), then Parts IV–VI, then appendices, then Chapter 19, then cross-references, then verification.
