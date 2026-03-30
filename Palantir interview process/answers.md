# Palantir FDE Behavioral Interview — Answer Guide

> **Interview: Tomorrow. Total time: ~45 minutes.** Read through these, practice aloud, internalize the bullets. The two most important answers are **Why Palantir** and **Tell Me About a Time You Used Data** — those will each get 5+ minutes.

---

## 🔴 TO-DO BEFORE INTERVIEW

- [ ] **Go on the Palantir website → Foundry** — click around the product demos, play with the ontology builder if there's a sandbox, screenshot anything useful for "I actually explored Foundry and..."
- [ ] **Read through the redesigned data system (Chapters 28-31)** — especially Chapter 29 (the optimal redesign with the 10-table schema, ETL pipeline, pgvector). You need to be able to speak to this fluently as if you built it, since the "data" answer references the redesign. Know the table names, the document-to-SQL flow, and the key trade-offs
- [ ] **Review system design chapters** — Chapter 29 system design building blocks, Chapter 30 system design practice. Focus on being able to whiteboard the dual-store diagram from memory
- [ ] **Practice the "data" answer aloud** — time yourself, target 4-5 minutes, practice interruption recovery
- [ ] **Review Foundry vocabulary** — object types, link types, transforms, datasets, ontology, Workshop, Slate, AIP, OSDK

---

## 1. Why Palantir? ⭐ (Most Important — 3-5 min)

**Level 1 — The personal, visceral reason:**

- When I was at the Pentagon doing intelligence work, I was collecting information about something genuinely timely and important
- I spent **three weeks** physically running around the Pentagon, knocking on doors, trying to get the right people to share the data I needed
- What should have taken **one day** took **three weeks** — because data was siloed across offices, classifications, and bureaucracies
- I've watched, firsthand, smart people spend the **majority of their time** navigating information silos instead of doing the actual analysis
- Palantir eliminates that. I know people who use Maven every day. It makes their work **more fulfilling** — they spend time on the mission, not on data wrangling
- I want to be part of the team that gives those people superpowers

**Level 2 — The strategic, big-picture reason:**

- Making disparate data intuitive and actionable isn't just a productivity tool — it's a **revolution in military affairs**
- Historically, the nature of warfare changes only a handful of times: gunpowder, the railroad, nuclear weapons, precision-guided munitions
- What Palantir does — giving commanders a unified, real-time picture of the entire battlespace — enables something that's been impossible since Napoleon: **true unitary command**
- In World War II, multi-theater warfare meant strategy, operations, and tactics were separated by distance, time, and information lag. Commanders made decisions on stale data
- When you can see everything, integrate everything, and act on it in real time, the nature of command itself changes. That's not incremental — that's a paradigm shift
- I want to work at the company that's actually building that

---

## 2. Tell Me About Yourself (~2 min)

- From **LA**. Interested in national security since reading "Make Your Bed" by Admiral McRaven in **eighth grade**
- Went to **Yale** — studied **History** and **Chinese**, did **Air Force ROTC** all four years
- Currently an **Air Force Intelligence Officer** in the Massachusetts Air National Guard (part-time)
- After school, briefly contracted for the **Office of Net Assessment** (the Pentagon's internal think tank) doing war games
- That turned into my own startup — building AI tools for military planners and war fighters
- On a personal level: extremely curious, obsessive when I lock onto a problem. I exercise, I'm an amateur magician, and I have an inexplicable love for the US Postal Service

---

## 3. Why Your Majors / School? (~2 min)

- **History** because it's the most **methodologically comprehensive** way we have of understanding the world
  - There's anthropological history, evolutionary history, technological history, economic history
  - It teaches you to **look left and right** at adjacent causes and contexts, instead of just looking straight down a narrow disciplinary lane
  - That cross-domain thinking is exactly what an FDE needs — understanding a client's business, their data, their politics, their constraints
- **Yale specifically** because its history department is arguably the best in the country
  - Every major decision I've made has been about **following people I respect and want to learn from**
  - Several Yale history professors became close mentors who still advise me today
- **Chinese** because I believed — and still believe — the US-China relationship is the defining geopolitical question of our generation
  - Spent a summer in **Taiwan** on a language immersion program (see "3 Defining Moments")
  - That language + regional expertise directly feeds into the wargame simulation work

---

## 4. What Do You Hope to Gain From Your First Job? (~2 min)

**Two things:**

- **Getting buy-in from skeptical stakeholders.** From what I understand, FDEs walk into organizations where people may not want you there — they're skeptical, they're protective of their workflows, they might even see you as a threat. Learning to read a room, understand what someone actually needs (not what they say they need), and earn their trust — that's the skill I most want to develop. I've started learning it (see the Stakeholders answer) but I want to do it at scale, across many different organizations
- **Stacking wins.** I want to go into whatever I do next — whether that's staying at Palantir for 10 years or building something — and be able to point back and say "I built that. That was mine." From what I've seen, Palantir specifically gives people enormous autonomy and ownership over real deployments. You're not a cog — you own a thing end to end. That's exactly the kind of responsibility I'm looking for

---

## 5. What Is Your Coding Experience? (~2 min)

- I build with AI IDEs — and I know Palantir has an **internal AI IDE** and a huge push toward **agentic AI** internally. That's exactly the workflow I've been living in for the past year
- I architect systems, I design the data models and the component structure, and I use AI as the implementation layer. Then I **read, debug, and iterate** manually — I can look at a Python backend, a SQL schema, or a TypeScript component and tell you exactly what it does and why it's wrong
- I also used AI to **write me custom textbooks** explaining codebases and concepts — basically teaching myself data structures, systems design, database schemas, and API patterns through ad-hoc generated curriculum
- What I've built:
  - **Wargame simulation engine** — Python, FastAPI, LangGraph, SQLite → the full data architecture I just described
  - **Siren** — a smart focus app with Next.js, Supabase/PostgreSQL, Gemini embeddings, real-time page classification
  - **Flashcard study platform** — single-file HTML app with spaced repetition (FSRS algorithm)
- Beyond coding: I have a **ton of experience designing AI systems** — multi-agent architectures, prompt engineering, embedding pipelines, LLM orchestration with LangGraph. That maps directly to what Palantir is building with **AIP** — agentic AI that sits on top of the ontology and reasons over structured data. I've been living in that world already
- The point is: I'm not someone who memorized LeetCode. I'm someone who **builds real AI-powered systems** and has deep experience thinking about how AI agents interact with data. That's the direction Palantir is going with AIP, and it's where I already am

---

## 6. Tell Me About a Time You Used Data ⭐ (Most Important — 5 min)

> **This is the Chapter 30 narrative. Deliver it from the bullet points below — Chapter 30 has the fully written version if you need to re-read it on the plane.**

**Context (30s):**
- Built a **geopolitical crisis simulation engine** — a system where AI-controlled world leaders make strategic decisions independently
- You give it a scenario — "China blockades Taiwan" — and 5 AI leaders each decide actions based on intelligence briefings, memory of past rounds, and their real-world documented personas
- Ran **batches of 100 simulations** of the same crisis in parallel (called Monte Carlo batches) to build statistical distributions of outcomes — not just "what happened once" but "what happens on average"
- Each batch produces ~**60,000-70,000 individual data points** (5 agents × 10 rounds × dozens of actions/memories/briefings per round)

**The Unique Data Challenges (30s — weave in naturally, don't make it its own section aloud):**
- All the data is **AI-generated** — there's no clean CSV or API. Every action, every briefing, every round summary is produced by an LLM. That means the data is inherently messy
- The LLM sometimes produces **malformed or broken JSON** — missing closing brackets, extra commas, fields in the wrong format. I built a whole set of **fallback parsing mechanisms**: try strict JSON parsing first, if that fails try to repair common errors, if that fails extract what you can and flag the rest. You can't just crash a 2-hour batch run because round 7 of simulation 43 had a bad comma
- I also had to think ahead about **making every run analyzable after the fact** — that meant generating **synthetic metadata** on each simulation: what was the peak escalation level? How did each country end up (better off, worse off, catastrophic)? Was there a nuclear event? These are labels that the LLM generates as part of the analysis, and I promote them to first-class tagged fields so I can filter and group by them later. Without those tags, every simulation would just be a blob of text you'd have to read manually to understand what happened

**The Problem (30s):**
- Version 1 used **one database (SQLite) for everything** — both the live simulation state AND the post-game analytics
- The core problem: I had a single text column (`final_state`) that stored the **entire simulation as one giant JSON string** — sometimes **100KB** of nested data just dumped into one cell
- I could tell you a simulation finished, and I could tell you the final escalation number — but I couldn't answer basic analytical questions
- "How many military actions did China take across 100 simulations?" — I'd have to **load every single JSON blob into Python, parse it, loop through all the nested data manually**. No database query could answer it directly
- There was literally **no separate table for actions**. All the actions were buried inside that JSON blob. I couldn't sort them, filter them, or count them with SQL

**My Analysis (1 min):**
- Stepped back and realized: I was trying to serve **two fundamentally different workloads** from the same database
- **The live simulation** ("hot path"): each sim is its own isolated world. Deeply nested data, changing rapidly every round, and each of the 100 parallel simulations is completely independent. This needs a flexible format, fast saves, and the ability for many simulations to write at the same time without stepping on each other
- **The post-game analysis** ("cold path"): after simulations finish, I want to ask cross-cutting questions across hundreds of sims — count actions by type, compare escalation patterns, find outliers. This needs a structured, relational database where each entity (action, briefing, memory) is its own queryable row
- I was forcing both through a database that can **only handle one write at a time** (SQLite's single-writer lock). That's why I'd had to build all this band-aid infrastructure: a special write mode that lets people read while someone writes (WAL mode), a rule that says "if the database is busy, wait up to 60 seconds before giving up" (busy timeout), and a retry system that backs off and tries again with increasing delays (exponential backoff)
- All that retry logic was a **symptom, not the solution** — I was treating an architectural problem with operational duct tape

**The Redesign (1 min):**
- Designed a **two-database architecture** with a transformation pipeline (ETL) connecting them
- **For the live simulation** ("hot path"): each simulation's state is just **one standalone document** — a JSON file on my laptop, or a MongoDB document in production. Since each sim is its own file, **100 concurrent sims = 100 independent writes**. Zero fighting over a shared database
- **For the analytics** ("cold path"): after a simulation finishes, a transformation pipeline (ETL) takes that one big document and **splits it apart into 10 separate, properly structured database tables** — one for simulations, one for rounds, one for individual actions, one for briefings, one for agent states, etc. Each table is fully normalized (3NF — meaning no duplicated or nested data, every piece of information exists in exactly one place)
- Now that "China military actions" question is just one line of SQL: `SELECT action_type, COUNT(*) FROM actions WHERE actor_code = 'CHN' GROUP BY action_type` — instant answer
- Also added **meaning-based search** (pgvector): round summaries and action descriptions get turned into mathematical representations of their meaning (768-number vectors). An index lets you search by meaning, not just keywords — "find rounds most similar to nuclear brinkmanship" — and combine that with normal database filters in one query
- Built it locally on my laptop with JSON files + SQLite for the analytics side, but designed the code behind a clean interface (a `SimulationStore` protocol) so swapping to MongoDB + PostgreSQL in production is a config change, not a rewrite

**Results (30s):**
- **Writes**: zero contention. Removed ~80 lines of retry/wait/backoff code because it was no longer necessary
- **Queryability**: for the first time, every single entity — every action, every briefing, every memory — is independently queryable. Millisecond SQL queries replaced Python blob-parsing loops
- **Semantic search**: can ask natural-language questions against simulation narratives, combining meaning-similarity with relational filters

**Trade-offs (30s):**
- **More moving parts**: two databases instead of one — mitigated during development by keeping both local (JSON files + local PostgreSQL)
- **Slight delay**: there's a brief window after a simulation completes but before the transformation pipeline has run where the analytics database is stale. Acceptable because nobody needs analytics on a simulation that finished 200 milliseconds ago
- **Double the storage**: the data exists in both stores temporarily — mitigated by archiving the raw documents after the transformation runs
- This mirrors **Palantir's own philosophy**: capture data in the source system's native format, transform it asynchronously into a clean ontology, and serve analytics from the structured layer

---

## 7. Tell Me How You Convince Critical Stakeholders (~3 min)

**Story A — The Colonel and MIT Lincoln Labs:**

- Approached a **Colonel** who ran the Air Force AI Accelerator at a conference, tried to pitch him directly
- He **brushed me off** — politely but firmly. I was a nobody
- Emailed him once after — brushed off again
- Realized: **hierarchy matters** in the military. Going straight to the top as an unknown is presumptuous
- Changed strategy: found his **entire team** of subordinates — captains, majors, civilian researchers
- Spent **months** building relationships with them individually, getting them excited about the idea, earning their trust
- Eventually they **invited me to speak at MIT Lincoln Labs** — the colonel's own venue
- When I met him there, he was **genuinely happy to see me**: "Good job"
- **Lesson**: Sometimes people don't want to be hounded. Show you can earn your way in through the hard door. They'll respect you more for it

**Story B — The Missile Adjudication Tool at Telemus:**

- At Telemus (my last job), I was starting to use AI to build tools. Introduced ideas I thought were great — digital wargame boards, AI-assisted analysis
- Team was **not excited**. They didn't see the value
- Changed approach: instead of pitching my ideas, I asked them **"What's the worst part of your job? What's most tedious?"**
- One analyst said: **"Adjudicating missiles against ships"** — dozens of factors (radar horizon, missile size, ship height, radar type, countermeasures)
- I said: "Let me build that." They were busy, so I built the **entire tool from scratch**
- We then **used it during a real wargame** and it actually worked
- **Lesson**: Don't pitch solutions — **find their pain**, solve it, bring it back working. Let the tool speak for itself

---

## 8. Three Defining Moments (~3 min)

> Frame: "The three most defining moments were the times I was most vulnerable — and those were the periods of most intense growth."

**1. Air Force Boot Camp (Failure + Growth):**
- I'd gotten everything I'd ever wanted my entire life. Never faced a real challenge. Was told my whole childhood I was special
- Showed up to field training and **utterly failed**. The skills that mattered — yelling at people, marching, performing under chaos — I had zero experience with
- Realized the only way to survive was to **get the people around me to like me and help me**
- My roommate would stay up late teaching me things I didn't know how to do — became one of my closest friends
- **Takeaway**: raw talent means nothing without humility and the ability to build trust fast

**2. Taiwan (Loneliness + Gratitude):**
- Language immersion program. Completely alone in a foreign country with foreign cultures
- Time difference made it nearly impossible to call friends or family
- Was the **loneliest I'd ever been** — felt like nobody would notice if I disappeared
- That isolation made me **profoundly grateful** for every person I care about
- **Takeaway**: I came back with a deep appreciation for relationships and a commitment to never take them for granted

**3. Released from Active Duty (Forced Reinvention):**
- Was supposed to go active duty Air Force after graduation. Very last minute, got released into the **Air National Guard** — part-time only
- Suddenly needed a full-time career I'd never planned for
- Most people decide their career path during college — I never had to because I "knew" I was going active duty
- This forced period of reflection made me ask: **What are the hardest problems in the world? What do I actually want to solve?**
- Led directly to the AI + national security work I'm now pursuing
- **Takeaway**: the best career decisions come from being forced to actually think, not from following the default path

---

## 9. Tell Me About a Failure (~3 min)

- Quit my job at Telemus because one of my bosses encouraged me to leave and said he could help me get a **DARPA contract**
- I thought I had a clear path: take this contract, build something important, succeed
- The deal **fell through completely**. All of a sudden I was **unemployed** in a field I knew very little about — AI defense contracting
- Felt like a complete failure. No job, no contract, no network, no idea how defense contracting actually worked
- What I did: **built the network from scratch**
  - Reached out to every mentor I had and asked for introductions
  - Their friends taught me: this is how defense contracting works, this is what OSD does, this is how you talk to program managers, these are the people who control the money
  - Over months, I built a network of people who **wanted** to work with us
- Eventually assembled a strong pipeline of potential contracts and partnerships
- **The key lesson**: if I'd gotten that first DARPA contract, I would never have built the muscle of **building a network, understanding the ecosystem, learning to sell**
- That "failure" gave me skills that are 10x more valuable than the contract would have been

---

## 10. What App Do You Use Often and How Would You Improve It? (~3 min)

**Primary Answer — Cold Turkey Blocker → Smart Focus Blocking (Siren):**

- I use **Cold Turkey**, a website blocker, every day for focus. You put YouTube, Twitter, etc. on a block list
- The problem: Google sometimes uses YouTube for **login and authentication**. So I block YouTube for focus, and then I can't log into Google services because the login page redirects through YouTube and gets blocked
- **My insight**: blocking should be based on **what you're doing**, not on domain names. A YouTube video about database design is productive. A YouTube meme is not. But domain-based blocking treats them the same
- So I actually **built this** — I call it **Siren**
- The idea: you tell it what you're working on, and it figures out whether each page you visit matches that intent
  - It takes each webpage and converts it into a mathematical representation of its meaning (an embedding), then compares that to what you said you're focused on
  - If the page is about what you're supposed to be working on → **allowed**. If it's not → **blocked**
  - So a YouTube tutorial on SQL → allowed. A random YouTube rabbit hole → blocked
- But it's not always obvious. Some pages are ambiguous — is Stack Overflow a tool right now or a distraction?
  - So I built a **database of verified tool pages** — pages that have been confirmed as productive
  - Users can **challenge** a block — if they think a page was wrongly blocked, it triggers a deeper analysis using an LLM (which is more expensive than the quick embedding check) that actually reads the page and reasons about whether it's relevant
  - Over time the system learns what's a tool and what's a distraction for that specific user and focus area

**Backup ideas (don't expand, just have ready):**
- Vision model idea for accessibility
- Kalshi prediction market social features

---

## 11. 8 Minutes of Questions for the Interviewer

> Ask 3-4 of these depending on time. Lead with the ones most important to you.

1. **"Is there any appetite at Palantir for the kind of AI wargaming / geopolitical simulation work I'm building? If I could bring a government contract for this tool, how would that work?"**

2. **"I want to build deep technical skills. How feasible is it for an FDE to internally transition to a Delta or Dev role over time?"**

3. **"If I came in with an existing tool and a contract — is there any version of an acqui-hire, or would it be better to keep the tool as a separate entity that partners with Palantir?"**

4. **"When you're hiring FDEs, are you looking for raw clay you can mold, or the finished statue of David? What matters more — raw potential or existing skills?"**

5. **"Why is there a New York USG office? I would have expected all government work to run through DC."**

6. **"What's your honest take on Palantir's culture? What surprised you about it — positively or negatively?"**

7. **"Do you have any feedback for me on how I should proceed in the application process? Anything I should emphasize or prepare for in the next round?"**

8. **"What's the most interesting problem you've personally worked on at Palantir?"**
