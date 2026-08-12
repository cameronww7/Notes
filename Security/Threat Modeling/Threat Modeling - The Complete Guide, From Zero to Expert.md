# Threat Modeling: The Complete Guide, From Zero to Expert

_A teaching walkthrough. Plain language, real examples, diagrams, workflows, mindset training, and the tooling landscape people actually use._

---

## Part 1: What Threat Modeling Actually Is

**Overview.** Before any framework, tool, or acronym, you need to know what problem threat modeling solves that nothing else solves. That problem is design flaws: mistakes in _what_ was decided rather than _how_ it was coded. Every other security tool you own — SAST, DAST, dependency scanning, pentesting — operates on something that already exists. Threat modeling is the only practice that operates on decisions before they harden into architecture, which is exactly when they're cheapest to change.

This section builds your base vocabulary and draws the line between bugs and flaws. That line matters more than it looks. If you walk away with one thing, it's this: a scanner can tell you your lock is broken, but only a human thinking ahead can tell you that you put the door in the wrong wall. Companies get breached through wrong-wall doors constantly, and their scan reports were green the whole time.

The examples here are deliberately boring and common. Threat modeling is not about exotic nation-state attacks. It's about the ordinary, predictable ways systems get abused, caught early because someone bothered to ask.

Here's the simplest way to think about it.

Before you build a house, an architect looks at the plans and asks: "Could someone climb through that window? Is the back door visible from the street? Where would a burglar try first?" They ask this **before** the concrete is poured, because moving a door after the house is built costs a fortune.

Threat modeling is the same thing for software. It's a structured conversation, held **early**, about what could go wrong with a system before (or while) you build it. Not "did we write buggy code," but "did we design something that can be abused even if the code is perfect?"

**Key term: Threat.** Anything that could go wrong. A person, event, or condition that could cause harm to your system or data.

**Key term: Vulnerability.** A weakness that makes a threat possible. Threats are the "what could happen." Vulnerabilities are the "why it's possible."

**Key term: Mitigation (or countermeasure).** The thing you do about it. A control, a design change, a process.

**Key term: Design flaw vs. code bug.** A code bug is a mistake in _how_ something was written (a missing input check). A design flaw is a mistake in _what_ was decided (letting one service talk directly to a database it should never touch). Scanners catch code bugs. Only humans thinking ahead catch design flaws. That's the entire reason threat modeling exists.

### Why scanners can't save you

Your SAST tool, your DAST tool, your dependency scanner — they all check for _known bad patterns in code that already exists_. None of them can flag:

- "This API returns more data than the frontend needs"
- "Any authenticated user can call this admin endpoint because we assumed only admins would find it"
- "The password reset flow trusts an email address the user typed in"

All three are real, common, breach-causing flaws. All three pass every scan. All three would surface in a 30-minute threat modeling session.

---

## Part 2: The Mindset — How to Think Before You Learn to Model

**Overview.** This is the section most guides skip, and it's the one that actually makes people good at this. Frameworks are training wheels. STRIDE exists so that people who haven't developed the attacker mindset yet can still find threats systematically. Experienced practitioners barely think in letters anymore — they've internalized a way of looking at systems, and the framework just keeps them honest and complete. If you learn the mindset first, every framework in this guide becomes obvious. If you skip it, you'll run STRIDE like a checklist and produce threat models that look thorough and catch nothing interesting.

The core mental shift is this: builders think about what a system is _supposed_ to do. Attackers think about what a system _actually permits_. Those two sets are never the same, and the gap between them is where every breach lives. Training the mindset means learning to see the second set — the full space of what's possible — instead of the intended path. Nobody is born seeing it. It's a trainable habit, and this section gives you the drills.

One more thing before we start: the attacker mindset is not paranoia and it's not pessimism. It's precision. You're not assuming your teammates are malicious or your users are criminals. You're recognizing that a system serving a million people will eventually be probed by the small percentage who are curious, greedy, or hostile, and that "eventually" arrives faster than anyone expects.

### The five mental shifts

**Shift 1: From "how it works" to "what it permits."** A login form is _supposed_ to authenticate users. What it _permits_ is: unlimited guessing (no rate limit?), username enumeration ("wrong password" vs "no such user"), and being a free oracle for testing stolen credential lists. Same feature, two completely different views. Practice describing features both ways.

**Shift 2: Assume breach.** Don't ask "can an attacker get in?" Assume they're already in — one server, one set of stolen credentials, one compromised laptop — and ask "now what can they reach?" This single reframe is why concepts like least privilege, network segmentation, and blast radius exist. It converts a vague fear into a concrete map exercise.

**Shift 3: Think in paths, not vulnerabilities.** Attackers chain small things. A verbose error message (low severity) leaks an internal hostname, which reveals an admin panel (medium), which accepts a default credential (high) — and none of those three findings alone tells the story. Beginners rate findings in isolation. Experts ask "what does this let me do _next_?"

**Shift 4: Attackers are lazy and economical.** Real attackers don't use the coolest attack; they use the cheapest one that works. Nobody burns a zero-day when a phished credential works. This means your threat model should weight boring, cheap attacks (credential stuffing, exposed endpoints, misconfigurations, social engineering of the reset flow) far above exotic ones. If your threat model is full of Hollywood attacks and empty of password reuse, it's backwards.

**Shift 5: Trust is the raw material.** Every system is built on a pile of assumptions: "this input comes from our mobile app," "only the billing service calls this endpoint," "engineers won't look at customer data." Threat modeling is largely the act of writing those assumptions down and asking "what happens when this one is false?" The most dangerous assumptions are the ones nobody knows they're making — which is why the diagramming step exists: it drags assumptions into daylight.

### The questions experts actually ask

Keep this list next to you in every session. These are framework-free and work on anything:

- "What's the worst thing a _legitimate, logged-in_ user could do with this?" (abuse of intended access beats hacking almost every time)
- "Who do we trust here, and how do we know it's really them?"
- "What happens if this input is a lie?"
- "If this component gets fully compromised, what does the attacker now own?"
- "What would make headlines? Now trace backwards: what would have to fail to get there?"
- "What are we logging? Would we even know this happened?"
- "Where does data go that we'd be embarrassed to explain?"
- "What did we build for convenience that an attacker would call a feature?"

### Use cases vs. abuse cases

```
   THE SAME FEATURE, TWO LENSES

   Feature: "Users can share a document via a link"

   BUILDER'S LENS (use case)        ATTACKER'S LENS (abuse case)
   ─────────────────────────        ────────────────────────────
   • User clicks Share              • Are links guessable/sequential?
   • Gets a URL                     • Do links ever expire?
   • Sends it to a colleague        • Does the link work after the
   • Colleague opens doc              user is offboarded?
                                    • Can I enumerate links at scale?
   "Works great, ship it"           • Does search engine indexing
                                      pick these up?
                                    • Is access logged anywhere?
```

**Diagram breakdown:** Left side is the happy path — one path, the one in the ticket. Right side is the possibility space — six questions, each one a real incident that has happened to a real company (publicly guessable share links have leaked medical records, legal docs, and payroll files). Notice the abuse cases aren't clever hacks; they're just the feature, examined honestly. Training the mindset means the right column starts generating itself whenever you read a ticket. That's the skill.

### Cognitive traps to watch in yourself

- **Optimism bias:** "Nobody would bother attacking us." Attackers automate; they bother with everyone.
- **Curse of knowledge:** You know the admin URL is unlisted, so you feel it's safe. Unlisted is not protected. Attackers don't have your mental map — they have scanners that don't need it.
- **Checklist hypnosis:** Running STRIDE letter by letter and stopping when each letter has one entry. The framework is a floor, not a ceiling.
- **Threat actor fan-fiction:** Modeling the NSA while ignoring the intern with prod access and the unrotated API key from 2022.
- **Availability bias:** Over-weighting last week's headline attack and under-weighting your own boring exposures.

### How to train the mindset (actual drills)

1. **Narrate the news.** Every breach writeup you read, ask: "which question in a threat modeling session would have caught this?" Almost always there is one, and it's usually embarrassingly simple.
2. **Abuse-case your daily life.** Hotel keycard, airport kiosk, grocery app coupon flow. How would you cheat it? This is low-stakes reps for the same mental muscle.
3. **Read your own tickets adversarially.** Pick any feature ticket in your backlog and write three abuse cases before reading the design. Compare against what the design actually handles.
4. **Play Elevation of Privilege** with your team. The card game format forces everyone through all six STRIDE categories without it feeling like an audit.
5. **Do a "premortem."** Before a launch: "It's six months from now and this feature caused a breach. What happened?" People generate threats freely when framed as a story instead of criticism.

---

## Part 3: The Foundation — Shostack's Four Questions

**Overview.** Once the mindset is in place, you need a structure that turns it into a repeatable process — something a team can run consistently whether or not the most experienced person is in the room. That structure is Adam Shostack's Four Question Framework, and it's the closest thing the field has to a universal standard. Google, AWS, the US government's CMS, and the Threat Modeling Manifesto all build on the same four questions, which tells you something: the process itself is settled. The debates are all about how to execute it.

The framework's genius is that it's method-agnostic. It doesn't tell you to use STRIDE or any particular tool — it tells you what a complete threat modeling effort must answer, in what order, and (through the fourth question) that the effort never really ends. Everything else in this guide plugs into one of these four slots: DFDs answer question one, STRIDE and friends answer question two, prioritization and tracking answer question three, and validation loops answer question four.

Pay attention to the wording notes below. Shostack has written a whole whitepaper on why the exact words matter — teams that drift on the wording drift on the process. Consistency of language produces consistency of practice, which is an underrated lesson for anyone building a program across many teams.

Almost every serious threat modeling program is built on four questions:

```
┌─────────────────────────────────────────────────────────────┐
│                THE FOUR QUESTION FRAMEWORK                  │
│                                                             │
│  ┌──────────────────┐      ┌──────────────────────┐         │
│  │  1. What are we  │─────▶│  2. What can go      │         │
│  │  working on?     │      │  wrong?              │         │
│  │  (scope + map)   │      │  (enumerate threats) │         │
│  └──────────────────┘      └──────────┬───────────┘         │
│           ▲                           │                     │
│           │                           ▼                     │
│  ┌────────┴─────────┐      ┌──────────────────────┐         │
│  │  4. Did we do a  │◀─────│  3. What are we      │         │
│  │  good job?       │      │  going to do about   │         │
│  │  (validate +     │      │  it? (mitigate +     │         │
│  │  update)         │      │  assign owners)      │         │
│  └──────────────────┘      └──────────────────────┘         │
│                                                             │
│         It's a LOOP, not a checklist you finish.            │
└─────────────────────────────────────────────────────────────┘
```

**Diagram breakdown:** The arrows matter more than the boxes. Question 1 feeds Question 2 — if your scoping is weak, your threat list will be weak. Question 4 loops back to Question 1 because systems change, and a threat model of a system that no longer exists is fiction. Most failed programs run 1 → 2 → 3 once, file the doc, and never hit 4. The loop is the whole point.

Deliberate word choices worth knowing:

- **"Working on," not "building."** Keeps the framework usable on legacy systems, not just new ones. You can threat model a ten-year-old monolith.
- **"We," not "you."** Collaborative by design. Security alone can't do it well.
- **"Good job," not "perfect job."** The goal is a system more secure than it was, not a flawless model. "Did we do a good _enough_ job" is a valid and intentional variant.

---

## Part 4: The Map — Data Flow Diagrams (DFDs)

**Overview.** Question 1 ("what are we working on?") produces the single most important artifact in the whole practice: the Data Flow Diagram. If threat modeling has a secret, it's that most of the value is generated _while drawing the diagram_, before any formal threat enumeration starts. The act of putting the system on a whiteboard forces the team to state its assumptions out loud, and the room routinely discovers that three engineers hold three different mental models of the same system. Every one of those disagreements is a finding.

The concept that makes DFDs work is the **trust boundary** — the line where data crosses between things that trust each other differently. Threats cluster at boundaries the way crime clusters at borders: it's where the rules change, where identity gets checked (or doesn't), and where one side's assumptions stop being enforceable. Once you learn to see trust boundaries, you'll see them everywhere — between the internet and your app, between your app and its database, between your CI system and production, between an LLM and its tools.

A warning before you start drawing: the most common DFD mistake is drawing infrastructure instead of data flow. Nobody attacking you cares which availability zone you're in. They care where data enters, what touches it, and where the privilege lines sit. Keep the diagram about data, keep it small, and keep it honest — drawn as the system _is_, not as the architecture slide claims.

**Key term: DFD.** A drawing showing how data moves through your system — where it enters, what touches it, where it's stored, where it leaves.

**Key term: Trust boundary.** Any point where data moves between things operating at different levels of trust or privilege. This is _the_ core concept. Threats cluster at boundaries because that's where assumptions break.

### The five elements

|Element|Symbol (convention)|What it is|Example|
|---|---|---|---|
|External Entity|Rectangle|People or systems outside your control|A user, Stripe, a partner API|
|Process|Circle|Something that transforms data|Your web app, a Lambda, a queue worker|
|Data Store|Two parallel lines|Where data rests|Postgres, S3, Redis, a secrets vault|
|Data Flow|Arrow|Data in motion|HTTPS request, DB query, webhook|
|Trust Boundary|Dashed line|Where trust level changes|Internet → your app, app → cloud API|

_(Note: the ASCII diagrams in this guide flatten the notation — everything is a box. In a real tool you'd use the conventional symbols above.)_

### Example DFD: a simple web app

```
                    TRUST BOUNDARY #1 (Internet → App)
  ┌──────────┐     ╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌
  │   User   │────────▶┌─────────────┐
  │(browser) │  HTTPS  │   Web App   │──────▶┌───────────┐
  └──────────┘         │  (process)  │  SQL  │  Database │
                       └──────┬──────┘       │(data store)│
                              │              └───────────┘
                              │ API call        ▲
       TRUST BOUNDARY #2      │            TRUST BOUNDARY #3
       (App → 3rd party)      ▼            (App → its own data)
  ╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┌─────────────┐
                       │ Payment API │
                       │ (external)  │
                       └─────────────┘
```

**Diagram breakdown:** Three components, three flows, three boundaries — and each boundary is a place to ask "what can go wrong here?"

- **Boundary #1:** Everything crossing it is attacker-controlled. Every input from the user is hostile until proven otherwise. Injection, spoofing, and business-logic abuse live here.
- **Boundary #2:** You're trusting an external party. What if the payment API is slow, lies, or gets compromised? What data are you sending them that you shouldn't?
- **Boundary #3:** The app has privileged database access. Does it have _more_ access than it needs? If the app is popped, the DB goes with it — unless you scoped its credentials.

Even this toy diagram already generated real questions. That's the trick: **the diagram isn't the deliverable, it's the map you interrogate.**

Rules of thumb:

- Start small. Five or six components, four or five boundaries. Don't diagram your whole company.
- Every boundary gets interrogated. No skipping.
- Draw what _is_, not what was _intended_. Outdated diagrams produce wrong threat models.

---

## Part 5: STRIDE — The Default Threat Enumeration Framework

**Overview.** With the map drawn, Question 2 asks "what can go wrong?" — and this is where beginners freeze. Staring at a diagram waiting for threats to appear doesn't work; the mind needs prompts. STRIDE, developed at Microsoft in the late 1990s, is the most successful set of prompts ever created for this: six categories of threat, walked systematically against each piece of your diagram, so that nothing gets skipped because nobody thought to ask.

Understand what STRIDE is and isn't. It's a _mnemonic for completeness_, not a theory of all possible attacks. Its dominance comes from being teachable — you can get a room of engineers productive with it in twenty minutes — and from mapping neatly onto DFD elements, which turns an open-ended brainstorm into a bounded, finishable exercise. Its weaknesses are equally real: it was designed for deterministic software and has no vocabulary for AI systems, supply chain, or organizational risk. Mature programs use it as the default and extend past it deliberately.

The per-element mapping below is the practical key that most tutorials bury. You do not ask all six questions of everything. Learning which categories apply to which element type is the difference between a three-hour slog and a tight one-hour session — and it's the first thing to memorize cold.

|Letter|Threat|Plain-language question|Example 1|Example 2|
|---|---|---|---|---|
|**S**|Spoofing|"Can someone pretend to be someone else?"|A service fakes being a trusted internal caller|Attacker replays a stolen session cookie|
|**T**|Tampering|"Can someone change data they shouldn't?"|Modifying a JWT to escalate privileges|Editing a price field in a client-side request|
|**R**|Repudiation|"Can someone deny they did something?"|Admin deletes records; no audit log exists|Payment disputed with no signed transaction trail|
|**I**|Information Disclosure|"Can data leak to someone unauthorized?"|Stack traces in error messages|API returns full user objects including internal fields|
|**D**|Denial of Service|"Can someone make it unavailable?"|Unthrottled expensive search endpoint|File upload with no size limit fills the disk|
|**E**|Elevation of Privilege|"Can someone gain power they weren't given?"|Regular user hits an admin route directly|Container escape to the host|

### How you actually apply it

STRIDE is applied **per element and per flow**, not to the system as a whole. The web app DFD above (3 components + 3 flows) means six passes through STRIDE, not one.

```
  DFD Element          Which STRIDE categories apply?
  ─────────────        ──────────────────────────────
  External Entity  ──▶ S, R          (can it lie about identity or deny actions?)
  Process          ──▶ S,T,R,I,D,E   (all six — processes are the busiest surface)
  Data Store       ──▶ T,R,I,D       (no spoofing — a database doesn't claim identity)
  Data Flow        ──▶ T,I,D         (data in motion: intercept, alter, flood)
```

**Diagram breakdown:** This mapping is why STRIDE stays fast in practice. A database can't spoof anyone, so skip S. A network flow can't elevate privilege on its own, so skip E. Processes get the full treatment because they're where logic lives. Memorize this mapping and a session that would take three hours takes one.

### STRIDE's honest limitations

- Built for deterministic software. AI systems break its categories (Part 10).
- Doesn't cover organizational risk, insider threat, or supply chain well.
- Nothing about your CI/CD pipeline (Part 9).
- Popular because it's _teachable and systematic_, not because it's complete.

**Key term: DREAD.** An older Microsoft companion for _scoring_ found threats (Damage, Reproducibility, Exploitability, Affected users, Discoverability). Mostly retired because scoring was too subjective. Modern teams usually rate findings High/Medium/Low with business context, or CVSS-style scoring — with the caveat that CVSS scores vulnerabilities, not design flaws, so most teams rank threat model findings on simple likelihood × impact plus a judgment call.

---

## Part 6: The Rest of the Framework Landscape

**Overview.** STRIDE answers "what can go wrong with this component," but that's not the only question organizations need answered. Sometimes the question is "what does compromise cost the business?" (a board question), or "are we handling personal data lawfully?" (a privacy question), or "how would an attacker chain steps toward one specific goal?" (a red-team question), or "what breaks when software makes its own decisions?" (an AI question). Each of those questions has a framework shaped around it, and this section maps the landscape so you can pick by question rather than by habit.

Two mistakes dominate framework selection in the wild. The first is treating frameworks as competitors and picking one "winner" — in reality they nest and combine: PASTA can run STRIDE inside its analysis stages, LINDDUN runs alongside STRIDE on the same diagram. The second is choosing by familiarity: teams run STRIDE on a boardroom risk question and produce output executives can't use, or run PASTA on a sprint feature and burn a week producing a document nobody asked for.

The expert move is owning a small portfolio: STRIDE as the daily driver, one business-risk framework for leadership conversations, LINDDUN when privacy is in scope, and the AI trio (Part 10) when modeling agents. That's four tools, each matched to a question, and it covers essentially everything.

|Framework|Best for|One-line summary|Watch out for|
|---|---|---|---|
|**STRIDE**|Component-level modeling|Six threat categories per DFD element|Misses org, AI, pipeline risk|
|**PASTA**|Business-risk conversations|Seven stages from business objectives down to attack simulation|Resource-heavy, slow|
|**OCTAVE**|Org-level risk, smaller companies|Management-driven, asset-focused, operational|Not technically deep, hard to scale|
|**TRIKE**|Teams needing auditable methodology|Assigns an acceptable risk level per asset, verifies against it|Small community, thin tooling|
|**LINDDUN**|Privacy threats|STRIDE's cousin for privacy: Linking, Identifying, Non-repudiation, Detecting, Data disclosure, Unawareness, Non-compliance|Privacy only; pair with STRIDE|
|**Attack Trees**|Modeling one specific attack goal|Root = attacker goal, branches = ways to achieve it|Manual, explodes in size fast|
|**MAESTRO**|Agentic AI systems|Seven-layer decomposition of agent architectures (CSA, Feb 2025)|New, limited adoption so far|

### PASTA's seven stages, visualized

```
  BUSINESS ────────────────────────────────▶ TECHNICAL
  ┌──────────────┐
  │ 1. Define    │  "What does the business need protected?"
  │  objectives  │
  └──────┬───────┘
  ┌──────▼───────┐
  │ 2. Technical │  "What's the tech footprint in scope?"
  │  scope       │
  └──────┬───────┘
  ┌──────▼───────┐
  │ 3. Decompose │  "How does the app actually work?" (DFDs here)
  │  application │
  └──────┬───────┘
  ┌──────▼───────┐
  │ 4. Threat    │  "Who attacks systems like this, and how?"
  │  analysis    │  (STRIDE can slot in here)
  └──────┬───────┘
  ┌──────▼───────┐
  │ 5. Vuln &    │  "Where are we actually weak?"
  │  weakness    │
  └──────┬───────┘
  ┌──────▼───────┐
  │ 6. Attack    │  "Simulate the paths." (attack trees here)
  │  modeling    │
  └──────┬───────┘
  ┌──────▼───────┐
  │ 7. Risk &    │  "What does this cost the business,
  │  impact      │   and what do we do?"
  └──────────────┘
```

**Diagram breakdown:** PASTA is a funnel from business language down into technical detail and back up to business impact — which is exactly why it works in compliance-heavy and executive-facing environments where STRIDE output ("Tampering on flow 3") means nothing to the audience. Notice stages 3, 4, and 6: DFDs, STRIDE, and attack trees all slot _inside_ PASTA. That's the proof that frameworks nest rather than compete. The cost is real, though — seven stages with business stakeholder input is a multi-week effort, not a sprint ritual. Use it on crown-jewel systems and annual reviews, not feature work.

### Attack tree example (underused and great for teaching)

```
                    GOAL: Read another user's messages
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
  Steal a session       Exploit the API        Compromise
  ┌───────────┐        ┌──────────────┐       the database
  │ • XSS      │        │ • IDOR: swap │       ┌──────────┐
  │ • cookie   │        │   user_id in │       │ • SQLi   │
  │   theft on │        │   request    │       │ • stolen │
  │   shared PC│        │ • missing    │       │   creds  │
  └───────────┘        │   authz check│       └──────────┘
                       └──────────────┘
```

**Diagram breakdown:** The root is what the attacker _wants_, not a vulnerability. Each branch is an independent path to that goal, each leaf a concrete technique. Read it bottom-up as a defender: "if I fix the IDOR but leave XSS, the goal is still reachable." Attack trees force you to think in _paths_ — which is Mindset Shift 3 from Part 2 made visual. The IDOR branch is worth highlighting because it's one of the most common real-world findings and no scanner reliably catches it.

**Key term: IDOR.** Insecure Direct Object Reference — the app lets you access object #123 just by asking for #123, without checking that #123 belongs to _you_.

---

## Part 7: The General Workflow, Start to Finish

**Overview.** Everything so far has been concepts. This section is the operating procedure — what an actual threat modeling effort looks like from the moment someone says "this needs a review" to the moment fixes are validated. If you're building a program, this is the section to steal from; if you're joining one, this is the map of where you fit.

Two design decisions in this workflow carry most of its success or failure. First, **who drives each phase**: the diagram phase belongs to the engineers who built the system, because they're the only ones who know how it actually works, while security's job is facilitation — asking questions, keeping the enumeration systematic, and holding the quality bar. Programs that invert this (security draws, engineers watch) produce plausible-looking fiction. Second, **where findings land**: in the engineering backlog, as ordinary tickets with owners and severities, or they don't get fixed. Full stop. Decades of filed-and-forgotten threat model documents prove this one empirically.

The third thing to internalize is triggers. Threat modeling on a calendar ("annually, for the audit") guarantees staleness; threat modeling on _change_ (new auth flow, new integration, new data type) keeps the model tethered to reality at exactly the moments risk actually shifts. Write your triggers down. If they're not written, the loop in Question 4 never fires.

```
 PHASE          WHO'S DRIVING        OUTPUT
 ─────          ────────────         ──────
 ┌────────────────┐
 │ 0. TRIGGER     │  Anyone           "This needs a threat model"
 │  new feature,  │
 │  new integration│
 │  auth change   │
 └───────┬────────┘
         ▼
 ┌────────────────┐
 │ 1. SCOPE       │  Security +       One-paragraph scope,
 │  what's in,    │  tech lead        list of assets that matter
 │  what's out    │
 └───────┬────────┘
         ▼
 ┌────────────────┐
 │ 2. DIAGRAM     │  Engineers        DFD with trust
 │  draw the DFD  │  (they know       boundaries marked
 │  together      │  the system!)
 └───────┬────────┘
         ▼
 ┌────────────────┐
 │ 3. ENUMERATE   │  Everyone         Raw threat list,
 │  STRIDE per    │  (security        tagged to elements
 │  element/flow  │  facilitates)
 └───────┬────────┘
         ▼
 ┌────────────────┐
 │ 4. PRIORITIZE  │  Security +       Ranked list: severity,
 │  rank + assign │  eng manager      owner, deadline
 └───────┬────────┘
         ▼
 ┌────────────────┐
 │ 5. TRACK       │  Eng (in their    Tickets in Jira/GitHub,
 │  findings →    │  normal backlog)  NOT a security silo
 │  backlog       │
 └───────┬────────┘
         ▼
 ┌────────────────┐
 │ 6. VALIDATE    │  Security         Tests, pentest scope,
 │  did fixes     │                   updated model
 │  land + work?  │──────┐
 └────────────────┘      │
         ▲               │
         └───────────────┘
        (loop on next trigger)
```

**Diagram breakdown:** Three things this diagram is trying to burn in:

1. **The "who's driving" column.** Phase 2 belongs to engineers, not security. A security engineer who doesn't know the system will draw a diagram that looks right and misses the three things that would actually get you compromised. Security _facilitates_, builders _supply the truth_.
2. **Phase 5 goes into the normal backlog.** Findings in a security spreadsheet die there. Findings that become Jira tickets with owners get fixed. AWS pipes findings into automated tests and pentest scope; Google turns them into roadmap items owned by teams. Same principle at any size.
3. **Triggers, not calendars.** Auth flow changes, new external integrations, significant new data handling, relevant threat intel. Define them in writing or Phase 6's loop never fires.

### Time-boxing (what makes this sustainable)

|Session type|Time|Scope|
|---|---|---|
|Feature-level|30–60 min|One feature, existing DFD updated|
|New service|2–4 hours|Fresh DFD, full STRIDE pass|
|High-risk system (payments, identity, health data)|Multi-session|Full formal model, possibly PASTA, external review|

The 30-minute feature session is where most of the value lives, because it's the one that can actually happen every sprint.

---

## Part 8: How Different People Use It (This Is Where Expertise Lives)

**Overview.** Ask three people what threat modeling looks like and you'll get three honest, contradictory answers: a developer describes a fifteen-minute design-doc habit, a security engineer describes a facilitated half-day session with formal output, and a CISO describes a business-risk conversation feeding a risk register. All three are threat modeling. All three are correct. The expert insight is that these aren't competing definitions — they're _layers_, each with a different depth-to-coverage tradeoff, and a mature program deliberately runs all of them at once.

The fundamental tension driving all of this is arithmetic: one security engineer can facilitate maybe two or three formal sessions a week, while engineering ships fifty changes in the same window. Depth doesn't scale and coverage isn't deep. Every tooling trend in this field — dev-led templates, threat-model-as-code, and the entire agentic platform category — is an attempt to attack that arithmetic from a different angle.

As you read the three modes, notice what each optimizes and what each sacrifices. Then look at the coverage-vs-depth chart at the end: program design at the expert level is literally the act of placing each system in your portfolio onto that chart and deciding which layer it deserves. Payments get everything. The internal lunch-ordering app gets the dev checklist. Most things get something in between.

### Mode 1: Developer-led ("threat modeling as a design habit")

**What it looks like:** Lightweight, fast, embedded in existing rituals. A section in the design doc template. Fifteen minutes at the end of sprint planning. "What's the worst thing a logged-in user could do with this?"

**What devs actually use:**

- **Diagrams-as-code:** Mermaid or PlantUML committed next to the code, so the DFD lives in the repo and updates in PRs
- **pytm / Threagile:** threat-model-as-code — describe the system in Python (pytm) or YAML (Threagile), get generated diagrams and threat lists in CI
- **Design doc templates:** a "Security Considerations" section with 4–5 prompting questions
- **Elevation of Privilege card game:** gamified STRIDE for teams new to it

**Strengths:** Coverage. It happens on _everything_, not just what security had time for. **Weakness:** Depth. Devs find the obvious threats and miss the creative ones. Quality drifts without security spot-checks.

### Mode 2: Security-led ("threat modeling as a formal review")

**What it looks like:** Scheduled sessions, a facilitator, a formal DFD, full STRIDE (or PASTA for business framing), documented output, tracked findings. Reserved for high-risk systems and major architecture changes.

**What security pros actually use:**

- **Microsoft Threat Modeling Tool** — the classic free desktop tool; DFD + auto-generated STRIDE threats. Windows-only, dated, still everywhere
- **OWASP Threat Dragon** — free, open source, web-based, cross-platform DFD + threat tracking
- **AWS Threat Composer** — free, open source, structured around the four questions; great for learning and for documenting models as JSON
- **IriusRisk / ThreatModeler** (merged Jan 2026) — the commercial tier: diagram-driven enumeration, threat libraries, countermeasure suggestions, compliance mapping, MAESTRO support
- **Confluence/wiki templates + Jira** — honestly, still the most common "platform" in the wild

**Strengths:** Depth and rigor. Finds the non-obvious stuff. **Weakness:** Doesn't scale. Two or three sessions a week per facilitator, against fifty shipped changes.

### Mode 3: Product / executive ("threat modeling as risk conversation")

**What it looks like:** OCTAVE or PASTA framing. "What are our crown-jewel assets, what does compromise do to the business, what risk are we accepting?" Output feeds risk registers and budget decisions, not Jira.

### The coverage-vs-depth picture

```
 DEPTH of analysis
   ▲
   │            ┌─────────────────┐
 H │            │  Security-led   │
   │            │  formal reviews │
   │            │  (Mode 2)       │
   │            └─────────────────┘
   │   ┌─────────────────────────────────────┐
 M │   │   Agentic design-stage platforms    │
   │   │   (Prime, Clover — see Part 10)     │
   │   └─────────────────────────────────────┘
   │ ┌─────────────────────────────────────────────┐
 L │ │  Developer-led habits (Mode 1)              │
   │ └─────────────────────────────────────────────┘
   └──────────────────────────────────────────────────▶
                                        COVERAGE (% of changes reviewed)
```

**Diagram breakdown:** No single mode wins. Formal reviews go deep but cover a sliver of what ships (CrowdStrike's 2024 report: 54% of orgs admit fewer than half their code changes get _any_ security review). Developer-led habits cover everything shallowly. Agentic platforms sit in the middle — broader than formal reviews, deeper than an unaided dev checklist. A mature program deliberately runs **all three layers**: dev habits on everything, agentic/automated review on the long tail, formal sessions on the high-risk core.

---

## Part 9: The Gap Nobody Models — Your CI/CD Pipeline

**Overview.** Here's a question that exposes the blind spot in most threat modeling programs: your pipeline holds cloud credentials, signs your artifacts, and has standing access to production — so why does it never appear in a threat model? The honest answer is habit. Threat modeling grew up around applications, its examples are all web apps and APIs, and "the build system" got mentally filed under plumbing. Attackers filed it under opportunity, and the last several years of supply chain attacks are the result.

Think about what a pipeline _is_ through the Part 2 mindset: it's a machine that takes attacker-influenceable input (pull requests, dependencies, build definitions) and converts it into signed, trusted, production-deployed output — automatically, with elevated privileges, usually with less monitoring than any production service. Described that way, it's obviously the most attractive target in your company. Compromise one developer laptop and you get one credential; compromise the pipeline and you get a distribution channel wearing your company's signature.

The good news is that nothing new is required. The same four questions and the same DFD approach apply directly — you just point them at build and deploy infrastructure instead of the app. This section walks the pipeline as a chain of trust hops and shows the threat at each hop, then points you at OWASP SPVS for a structured control checklist to walk against your own config.

```
        THE PIPELINE AS A TRUST BOUNDARY CHAIN

  ┌──────┐   ┌────────┐   ┌───────┐   ┌──────────┐   ┌──────┐
  │ Dev  │──▶│ Source │──▶│ Build │──▶│ Artifact │──▶│ Prod │
  │      │PR │ Repo   │   │ (CI)  │   │ Registry │   │      │
  └──────┘   └────────┘   └───┬───┘   └──────────┘   └──────┘
     ▲           ▲            │            ▲            ▲
     │           │            ▼            │            │
  Threat:     Threat:    ┌─────────┐    Threat:      Threat:
  stolen      malicious  │ Secrets │    push a       deploy creds
  dev creds   PR to      │ in CI   │    backdoored   used to ship
              shared     │ env vars│    image        unauthorized
              workflow   └─────────┘                 changes
                              │
                         Threat: exfiltrate
                         cloud credentials
```

**Diagram breakdown:** Every arrow is a trust hop, and every hop has been exploited in the wild. Left to right:

- **Dev → Repo:** stolen developer credentials or a malicious contributor. The PR itself is attacker-controlled input to your CI.
- **Repo → Build:** the build definition (Jenkinsfile, GitHub Actions workflow) is _code that executes with privileges_. A malicious step injected via PR into a shared workflow runs with the pipeline's secrets.
- **Build:** CI environment variables holding cloud creds are the crown jewels. Exfiltrating them turns a repo compromise into a cloud compromise.
- **Build → Registry:** weak registry access controls let anyone push a backdoored image that prod will happily pull.
- **Registry → Prod:** deploy automation with standing production access means pipeline compromise = production compromise.

SolarWinds (2020) is the canonical example: attackers compromised the _build system_ and injected malicious code into _signed_ updates. The application code was fine. No application-scoped threat model would have seen it.

**How to model it:** the same four questions and DFD approach, applied to build/deploy infrastructure. For a structured control checklist, walk your pipeline against **OWASP SPVS** (Secure Pipeline Verification Standard), which organizes pipeline security controls by lifecycle stage. You don't need a full assessment — just reading the controls against your own config shows you where to focus.

And no, "we don't have a formal build system" isn't a defense. Ad-hoc pipelines have the same attack surface with less visibility.

---

## Part 10: AI Changes Both Sides of the Table

**Overview.** AI intersects threat modeling twice, and conflating the two directions is the most common confusion in current conversations. Direction one: AI as an _accelerant_ — tools that scale the practice itself, from internal GenAI assistants at JPMorgan and Booking.com to an entirely new commercial category of agentic design-review platforms. Direction two: AI as a _target_ — AI systems introduce threat classes (prompt injection, memory poisoning, tool misuse) that STRIDE has literally no letter for, requiring new frameworks layered on top of the classical ones.

On the accelerant side, the deepest thing to understand is the philosophical bet each tool category makes. Traditional platforms bet that teams _will_ produce structured threat models if the tool is good enough — twenty years of adoption data mostly disproves this at scale. The new agentic category bets that teams _never will_ at the pace engineering moves, so instead of demanding new behavior, the tools read the artifacts engineering already produces (tickets, design docs, PRs) and review those continuously. It's a coverage play against the arithmetic problem from Part 8, and the reported numbers (Neo4j: 49% → 100% ticket coverage) show why it's getting funded.

On the target side, the hardest problem isn't technical — it's organizational. Security teams often don't understand the ML stack; ML engineers don't think adversarially; and risk pools in the gap between them. When you model an AI system, closing that ownership gap is task zero, before any framework gets opened.

### 10a. AI accelerating threat modeling — three categories people constantly confuse

**Category 1: Internal enterprise builds.** Security teams building their own AI assistants on top of an existing process.

- JPMorgan Chase's AITMC: ~20% efficiency gain, average of 9 additional novel threats per model
- Booking.com: a self-service GenAI assistant (built by one engineer, open-sourced) letting dev teams self-initiate reviews, security sampling outputs for QA; the author notes it doesn't yet replace a security-led session
- Google: Gemini + computer vision reading architecture diagrams to auto-enumerate components and threats

**Category 2: Traditional platforms with AI features.** Diagram-driven, security-operated. You build/import a DFD, the platform enumerates threats against libraries. IriusRisk + ThreatModeler (merged Jan 2026) is the consolidated leader. Requires structured diagram input — which is exactly what most teams never produce.

**Category 3: Agentic design-stage platforms.** These don't wait for a diagram. They continuously read the engineering artifacts that _already exist_ — Jira tickets, Confluence design docs, PRs — and run automated security/privacy design reviews inside those tools.

- **Prime Security:** agentic reviews across Jira/Confluence/Linear/Azure DevOps in under 20 minutes; PayPal, Redis, Qualtrics; $20M Series A
- **Clover Security:** agents in Confluence/Jira/GitHub/Cursor/Slack; also detects **design-to-code drift** (what shipped vs. what was designed) and guardrails AI-generated code; Instacart, Plaid, Notion; $36M raised

**Reported Category 3 outcomes:** Neo4j went from 49% of tickets reviewed to 100%; Lemonade cut review time from ~2 hours to ~15 minutes; PROS reported a 400% lift in design review coverage.

**The tradeoff, stated plainly:** coverage vs. depth. Agentic review catches a wider surface earlier but doesn't produce the structured STRIDE/DFD output of a formal model. For the long tail, great trade. For payments, identity, and health data, formal human-led modeling still does something these tools don't. Both Prime and Clover say this themselves — neither accepts the "threat modeling tool" label.

**GIGO still applies:** these tools enrich engineering artifacts with threat context, but garbage tickets and empty design docs produce shallow risk output on any platform.

### 10b. AI as a new attack surface

STRIDE was built for deterministic software. LLMs are probabilistic, and several threat classes simply have no STRIDE category:

|Threat|What it is|Where it's covered|
|---|---|---|
|Prompt injection|Attacker steers model behavior through crafted input (seen in 73% of production AI audits in 2025)|OWASP LLM Top 10 #1|
|Training data poisoning|Corrupting behavior via the training set|MITRE ATLAS|
|Model inversion|Extracting training data from outputs|ATLAS (partial STRIDE fit: Info Disclosure)|
|Memory poisoning|Corrupting an agent's long-term memory over time|MAESTRO|
|Tool misuse / excessive agency|Agent takes unintended actions through its tool access|MAESTRO|
|System prompt extraction|Probing out role definitions and workflow logic|OWASP LLM Top 10|

**How the three AI frameworks fit together (complementary, not competing):**

- **OWASP LLM Top 10** → the threat taxonomy; start here
- **MAESTRO** → layer-by-layer decomposition for _agentic_ architectures specifically
- **MITRE ATLAS** → technique-level detail and mitigation mapping

**AI security vendors worth knowing** (distinct from design-stage platforms): Mindgard (automated AI red teaming / DAST for AI in CI/CD), Lakera → acquired by Check Point (runtime guardrails for prompt injection and leakage), ProtectAI and HiddenLayer (model supply chain security).

**Market signal:** ThreatModeler acquiring IriusRisk (Jan 2026), Check Point absorbing Lakera, Cato acquiring Aim Security, and $56M of fresh funding across Prime and Clover all point the same direction — the market is bifurcating into diagram-driven depth (consolidating incumbents) versus developer-native coverage (funded entrants).

---

## Part 11: The Tooling Landscape — Native vs. What People Actually Use

**Overview.** Here's the uncomfortable truth about threat modeling tools: the purpose-built ones are decades old, mostly free, and mostly unused, while the "tools" that actually carry the practice at most companies are whiteboards, Miro boards, Confluence templates, spreadsheets, and Jira. That's not a maturity failure to be fixed. It's a signal to be understood — and once you understand it, your own tooling decisions get much easier.

The signal is this: threat modeling's value is generated in a _conversation_, and its impact is delivered through a _backlog_. Neither of those lives in a dedicated modeling tool. Purpose-built tools optimize the middle artifact — the diagram, the generated threat list — which turns out to be the easy 20% of the job. The hard 80% (getting the right people in a room, scoping well, prioritizing honestly, and making findings actually get fixed) is process and integration, which is why the winning "stack" at most orgs is whatever collaboration and ticketing tools engineering already lives in.

Read the tables below with that lens, and note that adoption friction compounds: every extra login, license seat, and unfamiliar UI is another reason a busy team skips the review entirely — and a skipped review at depth zero is worth less than a shallow one that happened. This is the exact bet the agentic platform category is making commercially, and it's the same bet you should make personally when choosing your own stack.

### The "native" / purpose-built tools

|Tool|Cost|Style|Reality check|
|---|---|---|---|
|Microsoft Threat Modeling Tool|Free|Desktop DFD + auto-STRIDE|Windows-only, dated UX, still the most-taught tool|
|OWASP Threat Dragon|Free/OSS|Web DFD + threat tracking|Best free starting point; lighter threat libraries|
|AWS Threat Composer|Free/OSS|Four-question structured docs|Excellent for learning and JSON-portable models|
|pytm|Free/OSS|Threat-model-as-code (Python)|Loved by dev-led programs; needs coding comfort|
|Threagile|Free/OSS|Threat-model-as-code (YAML)|Agile/CI-friendly; same audience as pytm|
|IriusRisk / ThreatModeler|$$$|Enterprise platform|Deep libraries, compliance mapping, MAESTRO support; needs diagram discipline|
|SD Elements (Security Compass)|$$$|Survey-driven requirements|"Threat modeling without diagrams" — generates security requirements from questionnaires|
|Prime / Clover|$$$|Agentic design-stage|Not threat modeling tools by their own labeling; continuous design review|

### What people use _instead_ — and why

|Non-native tool|Used for|Why people choose it over native tools|
|---|---|---|
|**Whiteboard / Miro / Excalidraw / Lucidchart / draw.io**|The DFD + the session itself|Zero learning curve, real-time collaboration, engineers already have it open. The _conversation_ is the value; the tool just needs to not get in the way|
|**Mermaid / PlantUML in the repo**|Living DFDs|Diagrams version with the code, update in PRs, render in GitHub. Solves the #1 failure (stale diagrams) for free|
|**Confluence / Notion templates**|The threat model document|It's where design docs already live. A "Security Considerations" template section gets 10x the adoption of a separate tool|
|**Jira / GitHub Issues / Linear**|Findings tracking|Findings in the engineering backlog get fixed; findings in a security tool get forgotten. Non-negotiable in practice|
|**Spreadsheets**|STRIDE enumeration + risk ranking|Ugly, universal, sortable, filterable. Half the formal threat models in the world live in Excel|
|**Elevation of Privilege / OWASP Cornucopia card decks**|Teaching + running sessions|Gamified STRIDE lowers the intimidation barrier|
|**LLM chat (Claude, ChatGPT, Copilot)**|Enumeration brainstorming|"Here's my architecture, walk STRIDE against it" is genuinely useful for a first pass — with the same GIGO and no-institutional-memory caveats as any internal wrapper|

### Why native tools lose (the actual reasons)

1. **The tool isn't the value; the conversation is.** Purpose-built tools optimize the artifact. Practitioners optimize the discussion. A whiteboard beats a modeling tool for a 45-minute session every time.
2. **Adoption friction kills coverage.** Every login, license seat, and new UI is a reason a dev team skips the review. Tools living inside Jira/Confluence/GitHub win on coverage even when they lose on depth.
3. **Diagram discipline is rare.** IriusRisk-class platforms need structured diagram input maintained over time. Most orgs can't sustain that — which is exactly the bet Category 3 vendors are making.
4. **Findings must land in the dev backlog.** Any tool that traps findings in a security silo fails Question 4 by design. Jira integration is the single most important "feature" of any threat modeling stack.
5. **Native tools solve the easy 20%.** Drawing boxes and generating threat lists is the easy part. Scoping, facilitation, prioritization, and follow-through — the hard 80% — are process, not tooling.

### A pragmatic stack by maturity

|Maturity|Diagramming|Enumeration|Tracking|Formal high-risk|
|---|---|---|---|---|
|Starting out|Whiteboard/Excalidraw|STRIDE by hand + EoP cards|Jira|Threat Composer to learn structure|
|Scaling|Mermaid in-repo|pytm/Threagile in CI + LLM-assisted first pass|Jira (auto-created)|Threat Dragon or Threat Composer|
|Mature/enterprise|Both in-repo + platform|Agentic layer (Prime/Clover-style) on the long tail|Jira, bidirectional|IriusRisk-class platform for regulated systems|

---

## Part 12: Failure Modes — Why This Fails at Most Companies

**Overview.** Threat modeling has an odd reputation problem: nearly everyone in security agrees it's valuable, and nearly nobody does it consistently. That gap isn't hypocrisy — it's a predictable set of failure modes that repeat so reliably across companies that you should treat them as the default outcome to be engineered against, not as rare mistakes. Studying how it fails teaches you as much as studying how it works.

Notice the pattern across the table below: almost every failure is a failure of _process and incentives_, not of technique. Nobody fails at threat modeling because they applied STRIDE wrong. They fail because the session happened after architecture was locked (so nothing could change), or because findings had no owners (so nothing got fixed), or because the model was never updated (so it quietly became fiction). This is why the workflow in Part 7 obsesses over triggers, owners, and backlogs — each of those mechanisms exists to neutralize a specific failure mode listed here.

Keep the economics handy too, because the failure modes persist partly because the value is invisible until something goes wrong. The design-versus-production cost multiplier is your standing argument for doing this work early, and it lands with executives in a way that "best practice" never does.

|Failure|Smell|Fix|
|---|---|---|
|Too late|"Security review" scheduled the week before launch|Threat model at design doc stage; make it a design input, not a launch gate|
|One-time artifact|The model's last update predates two re-architectures|Written triggers: auth changes, new integrations, new data types, relevant threat intel|
|Wrong people|Session invite list is all security|Builders in the room, always. Security facilitates|
|Flat findings|40 threats, no severity, no owners|Every finding gets a rating, an owner, and a ticket before the session ends|
|No follow-through|Findings doc filed in SharePoint, never opened again|Findings → backlog → validation. AWS turns them into test cases; do a smaller version of the same|
|Theater|Beautiful diagrams produced for the auditor|Measure fixes shipped, not documents produced|

**The economics, for when you need to justify it:** IBM's System Science Institute found a design-stage defect costs roughly 6x less to fix than a production one — and for security flaws the multiplier is usually worse, because production means incident response, customer notification, and regulatory fallout, not just a patch.

---

## Part 13: The Expert Path — A Maturity Ladder

**Overview.** Expertise in threat modeling has two distinct tracks that people constantly conflate: personal skill (can _you_ look at a system and find what matters?) and program maturity (can your _organization_ do this consistently without heroics?). The maturity ladder below measures the second; the 30-day path measures the first. You need both — a brilliant individual modeler in a Level 1 org produces insights that go nowhere, and a Level 3 process staffed by checklist-runners produces volume without quality.

The most useful diagnostic on the ladder is Question 4. Organizations at Level 1 routinely believe they're at Level 2, and the tell is always the same: ask them to name their written update triggers and to show three findings that became shipped fixes. If they can't, the diagrams are decoration. The jump from Level 2 to Level 3 is the hard one, and it's cultural rather than technical — it happens when engineering owns the habit and security owns the quality bar, instead of security owning everything and engineering attending.

For personal skill, there's no substitute for the fourth week below: facilitating a real session with real engineers on a real feature. Reading builds vocabulary; drawing your own systems builds technique; but facilitation — asking questions, managing the room, converting findings to owned tickets, then watching what actually happens to them — is where all of it fuses into judgment.

```
 LEVEL 0          LEVEL 1           LEVEL 2            LEVEL 3           LEVEL 4
 ───────          ───────           ───────            ───────           ───────
 Nothing    ──▶   Ad-hoc      ──▶   Repeatable   ──▶   Embedded    ──▶   Continuous
                  sessions          process            in SDLC           + measured
 "What's a        Security runs     Templates,         Design docs       Agentic layer on
 threat           occasional        triggers,          require it,       long tail, formal
 model?"          reviews on        findings in        devs self-        on crown jewels,
                  big things        Jira, DFDs         serve, pipeline   metrics: coverage %,
                                    exist              is in scope       time-to-mitigate,
                                                                        model freshness
```

**Diagram breakdown:** Most organizations sit at Level 1 and _think_ they're at Level 2. The tell is Question 4: if you can't name your update triggers and can't show findings that became shipped fixes, you're at Level 1 no matter how good the diagrams look. The 2 → 3 jump is cultural, not technical. Level 4 adds measurement: coverage percentage (what fraction of changes got any review), time-to-mitigate, and model freshness (time since last update per system). Note this ladder maps onto the maturity stack table in Part 11: "Starting out" ≈ Levels 1–2, "Scaling" ≈ Level 3, "Mature/enterprise" ≈ Level 4.

### Your 30-day path to competence

1. **Week 1:** Read Shostack's _Threat Modeling: Designing for Security_ (or at minimum his free Four Question Framework whitepaper at shostack.org/whitepapers). Learn the STRIDE-per-element mapping cold. Run the Part 2 mindset drills daily — narrate one breach writeup per day.
2. **Week 2:** Pick a system you own. Draw the DFD in Excalidraw or Mermaid. Mark every trust boundary. Run STRIDE against each element yourself. Use AWS Threat Composer to structure the output — the gaps in your thinking become visible immediately.
3. **Week 3:** Threat model your own CI/CD pipeline using the same method, then walk it against the OWASP SPVS controls. This is the exercise almost nobody does and it will change how you see your infrastructure.
4. **Week 4:** Facilitate a 45-minute session with real engineers on a real upcoming feature. Whiteboard only. Your job is questions, not answers. Then convert findings into tickets with owners and watch what happens to them. That last part teaches more than everything above combined.

---

## Quick Reference Card

**The mindset in one line:** builders see what a system is supposed to do; you see what it permits.

**The four questions:** What are we working on? What can go wrong? What are we going to do about it? Did we do a good job?

**STRIDE:** Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege — applied per element, per flow.

**Element → STRIDE mapping:** External entity (S,R) · Process (all six) · Data store (T,R,I,D) · Data flow (T,I,D)

**Expert questions:** worst thing a legitimate user can do? · who do we trust and how do we know? · what if this input lies? · if this component falls, what does the attacker own? · would we even know?

**Update triggers:** auth flow changes · new external integration · significant new data handling · relevant threat intel

**Framework picker:** component design → STRIDE · business risk → PASTA/OCTAVE · privacy → LINDDUN · agentic AI → MAESTRO + LLM Top 10 + ATLAS · pipeline → four questions + SPVS

**The three-layer program:** dev habits on everything · automated/agentic review on the long tail · formal security-led sessions on crown jewels

**The one-sentence version:** Threat modeling is a habit of asking "what could go wrong" while it's still cheap to change the answer.