# Chapter 26: The Decomposition Playbook — Executing Palantir's Signature Interview

> *"The decomposition round isn't a case study. It's closer to architecture — you're designing a building from a vague brief, and the interviewer is the client who keeps changing the requirements."*

---

## Why This Chapter Matters

Palantir's Decomposition round is the interview that no other company does. It's not a McKinsey profitability case. It's not a Google system design question. It's something specific to the DS role: you receive an sprawling, deliberately ambiguous real-world problem, and you must break it into tractable pieces, design a data-powered solution, and navigate constant pushback — all in 45 minutes.

Chapter 25 gave you the consulting foundations. Chapter 17 gave you the 7-Step Framework and worked examples. This chapter goes deeper. It teaches you the *execution layer* — the minute-by-minute playbook for how to actually *perform* a decomposition under live interview conditions. It covers timing, how to draw on a whiteboard, how to handle the five types of interviewer pushback, and the specific cognitive traps that kill candidates.

If you only read one chapter before your Palantir interview, make it this one.

---

## 26.1 What Makes Decomposition Different from a Consulting Case

Let's be precise about the differences, because candidates who treat decomposition like a McKinsey case fail:

| Dimension | McKinsey Case | Palantir Decomposition |
|---|---|---|
| **Starting point** | A specific business question ("Why did profits drop?") | A vague, systems-level problem ("Help this agency reduce veteran homelessness") |
| **Data** | The interviewer provides data when you ask the right questions | You must *imagine* what data would exist and design around it |
| **Endpoint** | A recommendation ("We recommend X because Y") | An operational system ("Here's a Workshop app that lets caseworkers do X daily") |
| **Framework** | Borrow a known framework (profit tree, 3C's) and adapt it | Build a bespoke decomposition from scratch each time |
| **Math** | Often requires calculations (market sizing, break-even) | Math is secondary; system thinking is primary |
| **Pushback** | Interviewer redirects you toward the "right" answer path | Interviewer stress-tests your design — there is no right answer |
| **Product knowledge** | Not needed | Expected — you should reference Ontology, Workshop, Actions |
| **Time horizon** | The analysis | The deployment lifecycle — build, ship, iterate, scale |

### The Meta-Difference

A consulting case asks: **"What should they do?"**
A decomposition asks: **"What would you build, and how would people use it every day?"**

This is not a semantic distinction. It changes *everything* about how you structure your answer. In consulting, you're done when you have a recommendation. In decomposition, you're done when you've designed a system that a real person would sit down in front of every morning and use to make better decisions.

---

## 26.2 The Anatomy of 45 Minutes

Here's how to allocate your time. This is not a rigid script — adapt to the conversation — but if you have no plan, you'll either rush or run out of time.

```
┌─────────────────────────────────────────────────────────────┐
│                    45-MINUTE DECOMPOSITION                    │
│                                                              │
│  MINUTES 0-3:   LISTEN AND RESTATE                           │
│  ────────────   Hear the prompt. Restate in your own words.  │
│                 Confirm the problem boundary.                │
│                                                              │
│  MINUTES 3-8:   SCOPE AND CLARIFY                            │
│  ────────────   Ask 4-6 targeted questions.                  │
│                 Identify stakeholders, constraints, data.    │
│                 Verbally set up your structure.               │
│                                                              │
│  MINUTES 8-15:  DECOMPOSE                                    │
│  ─────────────  Break the problem into 3-5 major             │
│                 sub-problems. Prioritize which to tackle.    │
│                 Draw or describe the high-level architecture.│
│                                                              │
│  MINUTES 15-30: BUILD THE SOLUTION                           │
│  ──────────────  Deep-dive into 2-3 sub-problems.            │
│                 Define data entities, pipelines, and          │
│                 the user experience. Name specific fields.   │
│                 This is where the Ontology framing lives.    │
│                                                              │
│  MINUTES 30-40: HANDLE PUSHBACK AND ITERATE                  │
│  ──────────────  The interviewer challenges your design.      │
│                 You adapt, revise, and strengthen.           │
│                 Show flexibility, not defensiveness.          │
│                                                              │
│  MINUTES 40-45: CLOSE                                        │
│  ──────────────  Summarize what you designed.                 │
│                 Name what you'd explore further.             │
│                 State the phased deployment plan.             │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### The Critical Pivot — Minute 8

Most candidates spend too long clarifying and never get to building. Others jump straight to building and never establish context. **Minute 8 is your pivot point.** By minute 8, you should have:

1. Restated the problem ✓
2. Asked your clarifying questions ✓
3. Announced your structure ✓

If you haven't done all three by minute 8, you're behind. Speed up the clarification. If the interviewer seems eager to move forward, follow their cue — they're telling you the scoping is sufficient.

---

## 26.3 The Five Moves — Decomposition-Specific Techniques

These are the moves that specifically work in decomposition. They go beyond the general consulting techniques in Chapter 25.

### Move 1: The Stakeholder Map

Before touching data or systems, identify **who** in the organization would use what you build, and **what decision** they make with it. This is the single most important move in decomposition, and the one most candidates skip.

```
EXAMPLE PROMPT: "A state Department of Education wants to
reduce the high school dropout rate."

STAKEHOLDER MAP:

┌─────────────────────────────────────────────────────┐
│                                                      │
│  TIER 1 — DAILY USERS                               │
│  ─────────────────────                               │
│  School counselors: "Which of my 300 students are     │
│  at risk THIS WEEK, and what specific intervention    │
│  should I prioritize?"                                │
│                                                      │
│  TIER 2 — WEEKLY DECISION-MAKERS                     │
│  ──────────────────────────────                       │
│  Principals: "How is my school performing on           │
│  retention? Where do I need to add resources?"        │
│                                                      │
│  TIER 3 — STRATEGIC LEADERSHIP                       │
│  ────────────────────────────                        │
│  Superintendent / State officials: "Which districts   │
│  need intervention? How do we allocate state funding   │
│  for the biggest impact?"                             │
│                                                      │
└─────────────────────────────────────────────────────┘
```

**Why this works:** The interviewer instantly sees that you think about the *human operating system*, not just the technology. Different stakeholders need different views of the same data. The counselor needs a student-level alert list. The superintendent needs a district-level trend dashboard. This naturally leads to a multi-layer solution design.

**How to say it:**

```
"Before I design anything, let me think about who would
use this system daily. I see three tiers of users. First,
school counselors — they're the front line. They need a
student-level early warning system: which kids are showing
risk signals right now, and what's the recommended
intervention? Second, principals — they need a school-level
dashboard to allocate counseling resources. Third, state
leadership — they need the aggregate view to direct
funding. I'd design for the counselor first, because
that's where the direct intervention happens."
```

---

### Move 2: The Data Inventory Audit

In consulting, the interviewer gives you data when you ask. In decomposition, **you must hypothesize what data would exist** in the client's world and assess how accessible it is.

```
THE DATA INVENTORY GRID

For each potential data source, assess:

┌──────────────┬─────────────┬──────────────┬───────────────┐
│ Data Source   │ Likely      │ Quality      │ Integration   │
│              │ Exists?     │ Expectation  │ Difficulty    │
├──────────────┼─────────────┼──────────────┼───────────────┤
│ Student       │ ✅ Yes      │ High — it's  │ Low — comes   │
│ enrollment   │ (in SIS)    │ the core     │ from one      │
│ records      │             │ system       │ system (SIS)  │
├──────────────┼─────────────┼──────────────┼───────────────┤
│ Attendance   │ ✅ Yes      │ Medium —     │ Medium — may  │
│ records      │ (in SIS)    │ manual entry │ have gaps     │
│              │             │ errors       │               │
├──────────────┼─────────────┼──────────────┼───────────────┤
│ Grade data   │ ✅ Yes      │ Medium —     │ Medium —      │
│              │ (LMS/SIS)   │ varies by    │ multiple LMS  │
│              │             │ school       │ platforms     │
├──────────────┼─────────────┼──────────────┼───────────────┤
│ Discipline   │ ✅ Yes      │ Low — very   │ High —        │
│ records      │ (separate   │ inconsistent │ different     │
│              │   system)   │ reporting    │ systems by    │
│              │             │ standards    │ district      │
├──────────────┼─────────────┼──────────────┼───────────────┤
│ Home         │ ⚠️ Maybe    │ Low —        │ High — may    │
│ environment  │ (surveys,   │ incomplete,  │ not be        │
│ data         │  social     │ sensitive    │ digitized     │
│              │  services)  │              │               │
├──────────────┼─────────────┼──────────────┼───────────────┤
│ Economic     │ ✅ Yes      │ High — govt  │ Low — public  │
│ indicators   │ (Census,    │ data         │ APIs          │
│ by ZIP code  │  BLS)       │              │               │
└──────────────┴─────────────┴──────────────┴───────────────┘
```

**Why this works:** It shows the interviewer you understand that *data availability constrains solution design*. You're not designing in a vacuum. You're designing for the reality of messy, fragmented, sometimes-nonexistent data.

**How to say it:**

```
"Let me think about what data would actually exist here.
Student enrollment and attendance — that's in the Student
Information System, pretty reliable. Grades — probably in
a separate LMS, quality varies by school. Discipline records —
this is where it gets messy, because different schools code
incidents differently. And here's the hard one: home life
factors — poverty, family instability, food insecurity —
these are the strongest dropout predictors, but they're
the hardest to get because the data lives in social services
systems that may not integrate with the school system at all.

So my design needs to work even without that last category.
I'll build the core model on attendance, grades, and
discipline — the data we know we have — and design the
architecture so we can layer in socioeconomic data later
if we get access."
```

---

### Move 3: The Decision Architecture

This is the technique that separates "data analyst thinking" from "systems designer thinking." Instead of asking "what analysis should we run?", ask: **"What decision does this person make, and what would they need to see to make it well?"**

```
THE DECISION ARCHITECTURE TEMPLATE

For each stakeholder, define:

DECISION: _________________
FREQUENCY: ________________
CURRENT STATE: _____________
DESIRED STATE: _____________

Example — School Counselor:

DECISION: "Which students should I check on today,
           and what type of outreach should I do?"
FREQUENCY: Every morning
CURRENT STATE: Counselor relies on gut feeling, teachers
               stopping them in the hallway, or a student
               missing 3+ weeks before anyone notices.
DESIRED STATE: Counselor opens a dashboard at 8 AM showing
               a ranked list of at-risk students, each with
               a risk score, the contributing factors, and
               a recommended intervention type.
```

**Why this works:** It forces your design to be *operationally useful*, not just analytically interesting. The interviewer hears you describing a real workflow — a counselor at 8 AM opening an app to decide how to spend their day — and thinks: *"This person understands deployment."*

---

### Move 4: The Progressive Disclosure Stack

You can't cover everything in 45 minutes. The trick is to **design in layers** — present a lean core, then progressively disclose complexity as the interviewer asks.

```
LAYER 1 — THE MVP (always present this)
──────────────────────────────────────
The simplest version that delivers value.
"We build a risk score for each student based on
attendance + grades + discipline. Counselors see a
ranked list. They review the top 20 each week."

LAYER 2 — THE ENRICHED VERSION (present if asked)
──────────────────────────────────────────────────
Add sophistication but same core.
"We add temporal trending — is this student's risk
increasing or stable? We add peer comparison — how
does this student compare to similar demographics?
We add intervention tracking — what was tried before?"

LAYER 3 — THE FULL PLATFORM (present if time allows)
─────────────────────────────────────────────────────
The strategic vision.
"Long-term, this becomes a state-wide early warning
network. Districts share anonymized patterns. We can
identify systemic factors — like a new principal change
correlating with risk spikes. And we build an
intervention effectiveness feedback loop: which
interventions actually reduced dropout rates?"
```

**Why this works:** You demonstrate that you understand phased deployment — the core Palantir delivery model. You don't try to build the Death Star on day one. You ship value fast, then iterate.

**How to say it:**

```
"Let me propose this in phases. Phase 1 — the first
two weeks — we build the core risk score and the
counselor dashboard. That's the MVP that delivers
immediate value. Phase 2 — month two — we layer in
intervention tracking and outcome measurement. Phase 3 —
quarter two — we scale to the district level with
aggregate views for principals. I'll focus my design
on Phase 1 right now, but I want you to know the
architecture supports all three."
```

---

### Move 5: The Ontology Close

Every decomposition answer must end with an Ontology framing. This is your "Palantir signature" — the thing that makes your answer a DS answer rather than a generic consulting answer.

**The formula:**

```
"Let me map this to the Ontology.

OBJECT TYPES:
• Student (student_id, name, grade_level, school_id,
  enrollment_date)
• School (school_id, name, district_id, principal,
  student_count)
• RiskAssessment (assessment_id, student_id, risk_score,
  risk_tier, contributing_factors[], assessment_date)
• Intervention (intervention_id, student_id, counselor_id,
  type, date_initiated, status, outcome)
• AttendanceRecord (record_id, student_id, date,
  status [present/absent/tardy])

LINKS:
• Student → enrolled at → School
• RiskAssessment → for → Student
• Intervention → targets → Student
• Intervention → assigned to → Counselor

ACTIONS:
• 'Schedule Check-In' → creates an Intervention,
  assigns to the counselor, adds a calendar event
• 'Escalate to Principal' → changes risk_tier to
  CRITICAL, creates a notification for the principal
• 'Log Outcome' → updates the Intervention status,
  feeds back into the risk model for future scoring
• 'Request Home Visit' → triggers a workflow to
  coordinate with social services, creates a
  linked case in the external system

The Actions are what make this an operating system,
not just a dashboard."
```

---

## 26.4 The Five Types of Interviewer Pushback

The interviewer will challenge you. This is not optional — it's the most important part of the round. How you handle pushback is the single biggest predictor of your score.

Here are the five pushback types, what they're testing, and exactly how to respond:

### Pushback Type 1: "What if you don't have that data?"

**What they're testing:** Can you design around constraints? Do you panic when your assumptions break?

```
YOU: "I'd use attendance records as the primary
risk signal —"

INTERVIEWER: "What if the school system doesn't
track attendance digitally? What if it's paper-based
in half the schools?"

BAD RESPONSE:
"Well, we'd need to digitize it first."
(This is technically correct but shows no creativity)

GOOD RESPONSE:
"That's a real constraint. If half the schools are
paper-based, I'd take a dual approach. For the digital
schools, we proceed as planned. For the paper schools,
I'd look for proxy data — are there grade records that
are digital? Grade drops often correlate with attendance
issues. Or can we use a simpler signal like 'teacher
referrals' — a counselor creates a manual entry when
a teacher flags a concern? The system should work
with imperfect data. Meanwhile, we can make the
Phase 2 roadmap include digitizing attendance data
for the remaining schools."
```

**The pattern:** Acknowledge → Propose a proxy or alternative → Show that your architecture is resilient to missing data.

---

### Pushback Type 2: "That's too complicated. The client won't use it."

**What they're testing:** Can you simplify? Do you understand that an unused system is a failed deployment?

```
YOU: "The dashboard would show risk scores with
contributing factors, trend lines, peer comparisons,
intervention histories, and predictive confidence
intervals —"

INTERVIEWER: "That's a lot. The counselors have
5 minutes between students. They're not data
scientists. Can you simplify?"

BAD RESPONSE:
"We could remove the confidence intervals."
(This is trivially responsive — you're not engaging
with the real concern)

GOOD RESPONSE:
"Fair point — I'm over-designing for the user. Let
me rethink the counselor view. They need exactly
three things: the student's name, a red/yellow/green
risk indicator, and ONE recommended action. That's it.
Everything else — the trend data, the contributing
factors, the prediction confidence — lives on a
detail page they can drill into if they choose. But
the primary view is a simple action list: 'Check on
these 8 students today. Here's what to do for each.'

The richness still exists in the system for principals
and analysts. But the front-line counselor gets
simplicity."
```

**The pattern:** Validate the concern → Redesign for the actual user → Show that simplicity at the surface doesn't mean simplicity underneath.

---

### Pushback Type 3: "What would you do FIRST?"

**What they're testing:** Can you prioritize ruthlessly? Do you understand that deployment is sequential, not simultaneous?

```
INTERVIEWER: "You've outlined a lot. If you could
only do ONE thing in the first week, what is it?"

BAD RESPONSE:
"I'd start building the risk model."
(You're jumping to the solution without understanding
the problem on the ground)

GOOD RESPONSE:
"Week one, I'd do a data audit. I would sit with
the school IT team and understand exactly what data
exists, in what format, with what quality issues.
I'd interview two or three counselors to understand
their current workflow — how do they identify at-risk
students today? What tools do they currently use?

The reason this is first: I've seen deployments fail
because the team designed a system based on data they
*assumed* existed. Week one is about reality-testing
my assumptions. The engineering can wait a week; the
understanding cannot."
```

**The pattern:** Show that your first move is *understanding*, not building. This is the core DS instinct — and it mirrors what Palantir calls "the first two weeks of a deployment."

---

### Pushback Type 4: "That wouldn't work because [domain-specific objection]."

**What they're testing:** Can you take new information, update your model, and produce a revised design? Do you get defensive or collaborative?

```
INTERVIEWER: "You mentioned using discipline records
as a risk signal. But in practice, discipline data is
racially biased — Black students are disciplined at
much higher rates for the same behaviors. Your model
would over-flag them and under-flag white students."

BAD RESPONSE:
"That's a good point, we'd need to control for that."
(Vague, no specifics, sounds like hand-waving)

GOOD RESPONSE:
"That's a critical point and it changes my design.
If discipline data reflects systemic bias rather than
student risk, then using it raw would perpetuate
that bias through the model. I'd take three steps:

First, I'd weight discipline data lower in the risk
score — or remove it entirely and rely on attendance
and grade trajectories, which are less subjective.

Second, I'd add a fairness audit to the model: do
risk scores distribute proportionally across
demographics? If not, something is wrong.

Third — and this is important — I'd make the
contributing factors visible to the counselor. If
the system says 'Student X is high-risk because of
3 discipline incidents,' the counselor can apply
human judgment about whether those incidents are
truly predictive.

The system should augment human judgment, not
replace it. Especially in domains where the data
itself carries bias."
```

**The pattern:** Take the feedback seriously → Revise your design specifically (not vaguely) → Show that you think about ethical dimensions of data systems.

---

### Pushback Type 5: "What if the scope doubles?"

**What they're testing:** Did you design for scalability? Is your architecture rigid or extensible?

```
INTERVIEWER: "What if the state asks you to expand
this from 200 high schools to all 2,000 K-12 schools?"

BAD RESPONSE:
"We'd just scale up the infrastructure."
(Says nothing about what actually changes)

GOOD RESPONSE:
"The Ontology design handles this well because we
already modeled Schools as their own Object Type with
a level attribute. Adding K-8 schools is adding rows,
not changing the schema. The risk model would need
adjustment — dropout risk signals look different for
a 3rd grader vs. an 11th grader, so we'd need
age-appropriate risk indicators.

The bigger challenge is data heterogeneity. 2,000
schools means more variation in data quality, more
integration points, and probably different SIS
vendors across districts. I'd handle this with a
standardized data ingestion layer — a pipeline that
maps each district's schema to our canonical model.
The first 200 schools teach us the patterns; scaling
to 2,000 is configuration, not redesign.

But I'd be honest with the state: scaling 10x
requires investment in data pipelines and support
staff. The Ontology scales technically; the
organizational readiness is the bottleneck."
```

**The pattern:** Show that your design choices were intentional and scale-friendly → Identify the *real* scaling constraint (hint: it's usually organizational, not technical).

---

## 26.5 The Cognitive Traps — What Kills Candidates

### Trap 1: The Architecture Astronaut

Designing a beautiful, complex system that nobody asked for.

```
❌ "We'd build a microservices architecture with a
   Kafka event bus, a feature store, a model registry,
   and a CI/CD pipeline for model retraining..."

   The interviewer asked how to reduce dropouts.
   You're designing infrastructure.

✅ "The counselor opens the app at 8 AM and sees a
   list of 12 students, sorted by risk. For each one:
   a risk score, the top 3 contributing factors, and
   a button that says 'Schedule Check-In.' They click
   the button, it creates a calendar event and logs
   the outreach in the system."

   Start with the human experience. The architecture
   serves the experience, not the other way around.
```

---

### Trap 2: The Monologue Death Spiral

Talking for 8+ minutes without pausing. The interviewer disengages. You lose the collaborative dynamic.

**The rule:** Never talk for more than 3 minutes without checking in.

```
CHECK-IN PHRASES:
• "Does this direction resonate, or should I adjust?"
• "I'm about to go deeper on the data model — is
  that where you'd like me to focus, or would you
  rather I talk about the user experience?"
• "I've been outlining the first phase. Should I
  continue with Phase 2, or zoom into a specific
  part of Phase 1?"
• "What would you push back on so far?"
```

---

### Trap 3: The Abstraction Trap

Staying at a high level and never getting concrete.

```
❌ "We'd integrate the data, build a model, and
   create a dashboard."

   This describes every data project ever. It says
   nothing.

✅ "We'd pull attendance_records — one row per
   student per day, with a status field: present,
   absent, tardy, excused. We'd compute a rolling
   30-day absence rate per student. Any student
   above 15% absence rate in a 30-day window gets
   flagged. That threshold comes from research
   showing that missing more than 2 days per month
   increases dropout probability by 40%."

   Now you've said something specific and defensible.
```

**The test:** If you could replace the company and industry in your answer with any other company and industry and it would still make sense, you're too abstract. Get specific.

---

### Trap 4: The Kitchen Sink

Trying to cover everything instead of going deep on a few things.

Remember: **you will not finish.** The problem is deliberately too large. The interviewer knows this. What they want to see is *depth of thinking on the areas you choose to focus on*.

```
45-MINUTE BUDGET:
─────────────────
You have time to go deep on 2-3 sub-problems.
You do NOT have time for 7 sub-problems at surface level.

BETTER TO SAY:
"I've identified five major sub-problems here, but in the
interest of depth, let me focus on what I think is the
highest-impact one: the early warning model for counselors.
I can come back to the others if we have time."
```

---

### Trap 5: The Defensive Reaction

Getting rattled when your design is challenged.

```
❌ INTERVIEWER: "I don't think that approach would
   work because..."
   YOU: "Well, actually, if you think about it..."
   (You're fighting the interviewer)

✅ INTERVIEWER: "I don't think that approach would
   work because..."
   YOU: "That's helpful — tell me more about what
   you're seeing. [Listens.] Okay, that makes sense.
   Let me revise: instead of X, what if we..."
   (You're collaborating with the interviewer)
```

**The reframe:** Every pushback is the interviewer *giving you new information*. It's a gift, not an attack. The strongest candidates treat pushback as collaborative input that improves their design.

---

## 26.6 The Whiteboard: Visual Decomposition Techniques

If you have screen-share or a whiteboard (most virtual onsites allow it), **use it**. Candidates who diagram consistently score higher on communication.

### Technique 1: The Problem Tree

Draw the decomposition visually as you talk.

```
START WITH:
┌──────────────────────┐
│  Reduce HS Dropout   │
└──────────┬───────────┘
           │

THEN BRANCH:
┌──────────────────────┐
│  Reduce HS Dropout   │
└──────────┬───────────┘
           │
   ┌───────┼───────────┐
   │       │           │
┌──▼───┐ ┌─▼────┐ ┌───▼────┐
│Identi│ │Inter-│ │Measure │
│fy    │ │vene  │ │& Track │
│Risk  │ │Early │ │Outcomes│
└──┬───┘ └──┬───┘ └───┬────┘
   │        │         │
 ┌─▼──┐  ┌──▼──┐   ┌──▼──┐
 │Data│  │What │   │Feed-│
 │    │  │types│   │back │
 │    │  │of   │   │loop │
 └────┘  │inter│   └─────┘
         └─────┘
```

As you fill in each branch, you're showing the interviewer your structure in real time. They can redirect you to a branch they want to explore further.

---

### Technique 2: The System Flow Diagram

Draw the data flow from sources to user.

```
DATA SOURCES          PLATFORM              USERS
────────────         ─────────             ──────

SIS (enrollment) ─┐
                   ├──→ Cleanse ──→ ┌──────────┐
LMS (grades) ─────┤     Merge      │ Ontology  │──→ Counselor
                   ├──→ Score  ──→ │          │    (alert list)
Attendance sys ───┤               │ Student   │
                   │               │ School    │──→ Principal
Discipline sys ───┘               │ Risk      │    (dashboard)
                                   │ Interv.   │
Census data ──────────────────→   │          │──→ State HQ
                                   └──────────┘    (aggregate)
```

---

### Technique 3: The Entity Relationship Sketch

Show the Ontology as a simple ER diagram.

```
┌─────────┐    enrolled at    ┌─────────┐
│ Student │─────────────────→│  School  │
└────┬────┘                   └─────────┘
     │
     │ has
     ▼
┌───────────────┐    targets    ┌──────────────┐
│RiskAssessment │◄──────────────│ Intervention │
└───────────────┘               └──────┬───────┘
                                       │
                                 assigned to
                                       │
                                ┌──────▼───────┐
                                │  Counselor   │
                                └──────────────┘
```

Even a rough sketch communicates more clearly than five minutes of verbal description.

---

## 26.7 The Opening Script — Your First 3 Minutes

Here's a template for how to start any decomposition. Adapt the specifics, keep the structure.

```
INTERVIEWER: "A large urban hospital network is seeing
emergency department wait times of 6+ hours. The CEO
wants to use data to fix it. Walk me through your approach."

YOUR FIRST 3 MINUTES:

[RESTATE — 30 seconds]
"So the core problem is excessive ED wait times — 6+ hours
suggests patients are waiting significantly beyond what's
clinically appropriate. Before I design a solution, I want
to make sure I understand: are we trying to diagnose WHY
wait times are long, build a system that REDUCES them
going forward, or both?"

[CLARIFY — 90 seconds, 3-4 questions]
"A few quick questions to scope this:
• How many hospitals in the network, and how many EDs?
• Is the 6-hour figure an average, or a worst case?
  Because the distribution matters — if 80% of patients
  are seen in 2 hours and 20% wait 12, that's a very
  different problem than a flat 6-hour average.
• What's the primary bottleneck in their view — is it
  beds, is it staffing, or is it flow (patients waiting
  for imaging, labs, or specialist consults)?
• Who would be the primary user of whatever we build —
  the ED charge nurse, the hospital COO, or both?"

[SIGNPOST — 60 seconds]
"Here's how I'd structure this. I see three major
sub-problems: First, understanding the flow — where in
the patient journey does the waiting actually happen?
Second, predicting demand — can we forecast arrival
volumes by hour, day, and acuity level? Third,
optimizing resource allocation — given what we predict,
how should we staff and manage bed turnover?

I'm going to start with the first one — understanding
the flow — because you can't fix what you can't see.
Then I'll move to the predictive layer if we have time.
Stop me if you want to go in a different direction."
```

**What happened in those 3 minutes:**
1. You restated the problem (shows listening)
2. You asked smart questions (shows scoping ability)
3. You announced a structure (shows organized thinking)
4. You prioritized (shows judgment)
5. You invited collaboration (shows awareness that this is a conversation)

Now the interviewer knows where you're headed and can steer.

---

## 26.8 Worked Decomposition — Full 45-Minute Simulation

Let's walk through a complete decomposition to illustrate execution at every minute. This is longer than the other examples in the book because you asked for more detail.

---

### The Prompt

**"A mid-sized city's police department wants to reduce gun violence. They have 911 call data, arrest records, and social services referral data. You're the DS. How would you approach this?"**

> This is one of the hardest decomposition prompts because it involves ethical complexity, data sensitivity, and stakeholder tensions. If you can do this one, you can do any of them.

---

**Minutes 0-3: Listen, Restate, Set the Stage**

```
"This is a high-stakes, sensitive problem — let me make
sure I understand the goal. The city wants to reduce gun
violence, not just track it. So we're designing an
operational system, not a reporting dashboard. A few
questions before I structure my approach:

1. When you say 'reduce gun violence,' is the focus on
   prevention — stopping incidents before they happen —
   or response — getting resources to the right place
   faster — or both?

2. What's the political landscape? Is there public trust
   in the police department to use this kind of system?
   Because the best model in the world doesn't matter if
   the community rejects it.

3. The data sources you mentioned — 911 calls, arrests,
   social services — are there other data sources we
   should consider? Hospital trauma records? ShotSpotter
   or other gunshot detection? Community organization data?

4. Who are the intended users? Beat officers? Precinct
   commanders? Violence intervention workers? City
   council? Because each of these groups would use the
   system very differently and has very different trust
   relationships with the community."
```

---

**Minutes 3-8: Decomposition and Structure**

```
"Based on what I know, I'd decompose this into four
sub-problems:

┌──────────────────────────────────────────────────┐
│           REDUCE GUN VIOLENCE                     │
│                                                   │
│  1. UNDERSTAND THE PATTERNS                       │
│     Where, when, and among whom does gun violence  │
│     concentrate? What are the spatial and temporal │
│     clusters?                                     │
│                                                   │
│  2. IDENTIFY THE DRIVERS                          │
│     What factors predict elevated risk? Disputes?  │
│     Gang proximity? Economic stress? Reentry from  │
│     incarceration? Prior victimization?            │
│                                                   │
│  3. ENABLE INTERVENTION                            │
│     How do we route the right resources to the     │
│     right place at the right time? This includes   │
│     BOTH policing AND community-based intervention │
│     (violence interrupters, social workers).       │
│                                                   │
│  4. MEASURE OUTCOMES                               │
│     How do we know if what we're doing is working? │
│     What's the feedback loop?                     │
│                                                   │
└──────────────────────────────────────────────────┘

I want to focus on sub-problems 1 and 3 — understanding
the patterns and enabling intervention — because those
are where the system delivers the most operational value.
Does that prioritization resonate, or would you steer me
differently?"
```

---

**Minutes 8-20: Deep-Dive into Sub-Problem 1**

```
"Let me start with understanding the patterns.

DATA ENTITIES I'd define:

• Incident (incident_id, type [shooting/robbery/assault],
  lat/lon, timestamp, severity [fatal/injury/shots_fired],
  resolution_status)

• Location (location_id, address, lat/lon, zone,
  land_use [residential/commercial/public],
  nearby_features [school/park/bar/transit])

• Person (person_id — anonymized, demographics,
  relationship_to_incident [victim/suspect/witness],
  prior_involvement_flag, social_services_contact)

• Call (call_id, timestamp, caller_type [911/ShotSpotter],
  location_id, response_time_minutes, responding_units)

GRAIN:
• The Incident table is the core fact table — one row
  per violent event.
• Each Incident links to a Location and zero-or-more Persons.
• Each Incident may be preceded by related Calls.

ANALYSIS LAYER:

First, spatial analysis — I'd compute incident density
by grid cell (maybe 500m × 500m). This immediately shows
hot spots. But I'd go beyond a heat map: I'd look at
temporal clustering. Are hot spots persistent (same
corners, every weekend) or migrating (violence shifts
when policing shifts)?

Second, temporal analysis — time of day, day of week,
seasonal patterns, and — critically — event-driven spikes.
Does violence increase after certain types of incidents
(retaliatory patterns)? After weather changes?

Third, network analysis — this is where it gets powerful
but sensitive. Using arrest and call data, can we identify
connected groups? If person A was involved in incident X,
and person B was involved in incident Y, and A and B have
co-appeared in prior records — there may be a group dynamic
that a violence interrupter could address.

But — and this is important — the network analysis raises
serious ethical questions. I'd want to flag this explicitly."
```

---

**Minute 20: Interviewer Pushback**

```
INTERVIEWER: "You mentioned ethical concerns. What are they,
and how would you handle them?"

"Three big ones. First, racial bias in the data. Arrests
are not an unbiased measure of criminal behavior — they
reflect policing patterns. If the system learns from
arrest data, it will over-predict risk in communities
that are over-policed. I'd mitigate this by focusing on
OUTCOMES (shootings, which are less discretionary to
report) rather than INPUTS (arrests, which are discretionary).

Second, privacy. Building a network graph of individuals
based on government data is sensitive. I'd anonymize
person-level data for the spatial analysis entirely, and
restrict the network analysis to specific, court-authorized
investigations — not general patrol.

Third, community trust. If the community perceives this
system as surveillance, it will undermine the very
cooperation that reduces violence. I'd design the system
so that community-based organizations — violence
interrupters, social workers — are primary users alongside
police. The system should enable HELP, not just enforcement.
A use case: 'Person X was recently released from
incarceration and has indicators of elevated risk. Route
this to a reentry coordinator, not a patrol unit.'

This changes who the stakeholders are. It's not just a
police tool — it's a public safety ecosystem."
```

---

**Minutes 25-35: Deep-Dive into Sub-Problem 3 — Intervention**

```
"Let me now design the intervention layer — Sub-Problem 3.

STAKEHOLDER MAP:

• Precinct commander: 'Which zones need increased presence
  this weekend based on pattern analysis?'
• Violence interrupter (community org): 'Which individuals
  in my network are at elevated risk of being involved in
  violence? How can I reach them before something happens?'
• Social worker: 'Which recently released individuals in
  my caseload are in high-risk zones and need a check-in?'
• City leadership: 'Are our intervention programs working?
  Are we allocating resources to the right neighborhoods?'

WORKSHOP APPLICATION:

For the precinct commander:
• Map view showing predicted violence risk by zone for
  the next 72 hours
• Risk factors driving each zone's score (recent incident,
  event in the area, weather forecast, historical pattern)
• Resource deployment overlay: where are units now vs.
  where should they be?
• Action: 'Adjust Deployment' — creates a staffing
  recommendation for the shift commander

For the violence interrupter:
• Case list: individuals in their network, ranked by risk
• Each case shows: last contact, recent life events
  (housing change, incarceration release, family incident),
  recommended outreach type
• Action: 'Log Outreach' — records the contact, updates
  the individual's risk profile
• Action: 'Request Resources' — connects the individual
  to housing, employment, or mental health services

ONTOLOGY:

Object Types: Zone, Incident, Person, Intervention,
Organization, Resource
Links:
  Zone → contains → Incident
  Person → involved in → Incident
  Intervention → targets → Person
  Intervention → conducted by → Organization
  Resource → deployed to → Zone
Actions:
  'Adjust Deployment' → modifies resource allocation
  'Log Outreach' → creates Intervention record
  'Request Resources' → creates referral to social services
  'Escalate' → flags a pattern for command review"
```

---

**Minutes 35-40: Interviewer Pushback #2**

```
INTERVIEWER: "What's the very first thing you'd do on
Day 1 of this deployment?"

"Day 1, I'd interview front-line stakeholders — a beat
officer, a violence interrupter, and a social worker.
Not their bosses — the people doing the actual work. I'd
ask them: 'Walk me through your day. Where do you get
your information? What decisions do you make? What do
you wish you knew that you don't?'

Then I'd do a data audit. I'd pull a sample of the 911
call data and the incident reports and assess: are the
location fields geocodable? Are the timestamps consistent?
Is the severity field filled in reliably? Because the
entire system depends on data quality, and I've learned
that the gap between 'we have the data' and 'the data is
usable' is often enormous.

I would NOT start building anything on Day 1. The first
week is listening and understanding."
```

---

**Minutes 40-45: Close**

```
"Let me summarize what I've designed:

A public safety ecosystem — not just a police tool — that:
1. Maps spatial and temporal violence patterns using incident
   data, 911 calls, and sensor data
2. Provides predictive risk scores by zone to inform
   resource deployment
3. Supports community-based intervention by routing at-risk
   individuals to violence interrupters and social workers,
   not just patrol
4. Measures outcomes through an intervention tracking loop

What I'd want to explore further:
• The data bias question — we'd need a fairness review
  before this goes live
• Integration with hospital trauma data — this would
  capture incidents that never become police reports
• Legal review — some of the network analysis approaches
  have civil liberties implications that need legal sign-off

Deployment would be phased: Phase 1 is the zone-level
heat map and the command dashboard. Phase 2 is the
individual-level intervention tracking. Phase 3 is the
predictive layer and the community organization portal."
```

---

## 26.9 Rapid-Fire Practice Prompts

Practice each of these using the 45-minute template. Record yourself. Review.

| # | Prompt | Key Challenge |
|---|---|---|
| 1 | "A national grocery chain wants to reduce food waste across 1,200 stores." | Supply chain data, perishability, store-level variability |
| 2 | "A state education department wants to identify schools at risk of failing standardized tests, two years in advance." | Prediction vs. self-fulfilling prophecy, political sensitivity |
| 3 | "An electric utility wants to predict which power lines will cause wildfires." | Sensor data, weather integration, catastrophic risk vs. false positive cost |
| 4 | "A humanitarian NGO wants to optimize the placement of refugee camps across a conflict zone." | Incomplete data, rapidly changing conditions, security concerns |
| 5 | "A major airline wants to reduce cascading flight delays." | Network effects, crew scheduling constraints, weather dependency |
| 6 | "A pharmaceutical company wants to detect adverse drug reactions earlier from post-market surveillance data." | Rare event detection, FDA regulatory requirements, data from multiple sources (hospitals, pharmacies, patient reports) |
| 7 | "A city transportation department wants to reduce pedestrian fatalities by 50% in 3 years." | Vision Zero framework, infrastructure vs. behavior, privacy of camera data |
| 8 | "The US Forest Service wants to optimize prescribed burn planning across 193 million acres." | Multi-constraint optimization (weather, smoke, wildlife, proximity to communities), seasonal windows |
| 9 | "A large bank wants to detect money laundering patterns in transaction data." | False positive volume, regulatory reporting requirements (SARs), adversarial behavior |
| 10 | "A state child welfare agency wants to improve the accuracy of child abuse risk assessments." | Ethical minefield: bias in reporting, false positive consequences (family separation), model interpretability requirements |

For each prompt, practice:
1. Opening with a restatement and 3-4 clarifying questions (3 min)
2. Drawing a stakeholder map (2 min)
3. Decomposing into 3-5 sub-problems and prioritizing (5 min)
4. Deep-diving on 2 sub-problems with data entities and user experience (15 min)
5. Closing with an Ontology framing and phased deployment (5 min)
6. Having a friend challenge you with pushback from the five types above (10 min)

---

## 26.10 Chapter Summary — Quick Reference

### Time Allocation (45 min)
- Minutes 0-3: Listen, restate, confirm
- Minutes 3-8: Scope, clarify, announce structure
- Minutes 8-15: Decompose into sub-problems, prioritize
- Minutes 15-30: Build solution — data entities, user experience, Ontology
- Minutes 30-40: Handle pushback, iterate
- Minutes 40-45: Summarize, name gaps, state phased plan

### The Five Moves
1. **Stakeholder Map** — Who uses it, what decision, how often?
2. **Data Inventory Audit** — What data exists, what quality, how hard to integrate?
3. **Decision Architecture** — What decision, current state, desired state?
4. **Progressive Disclosure Stack** — MVP → Enriched → Full platform
5. **Ontology Close** — Object Types, Links, Properties, Actions

### The Five Pushback Types
1. "What if you don't have that data?" → Propose a proxy, design for resilience
2. "That's too complicated" → Simplify the surface, keep depth underneath
3. "What would you do first?" → Data audit + stakeholder interviews, not engineering
4. "That won't work because [domain objection]" → Take it seriously, revise concretely
5. "What if scope doubles?" → Show your design is extensible, identify the real bottleneck

### The Five Cognitive Traps
1. **Architecture Astronaut** — Too much infra, too little user experience
2. **Monologue Death Spiral** — Check in every 3 minutes
3. **Abstraction Trap** — Name specific fields, not vague "data"
4. **Kitchen Sink** — Go deep on 2-3 things, not shallow on 7
5. **Defensive Reaction** — Treat pushback as information, not attack

### The Key Difference from Consulting
- Consulting asks: "What should they do?" → Recommendation
- Decomposition asks: "What would you build, and how would people use it daily?" → Operating system

---

*Next Chapter: Chapter 27 — Decomposition Practice Lab: Eight Complete Scenarios with Analysis →*
