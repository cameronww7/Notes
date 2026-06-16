# Claude Token Efficiency: The Practitioner's Guide

---

## Quick Reference

Everything on this page in one place. Use this first, go deeper in the sections below when you need the detail.

### Session Habits

- **Start a new chat for every new task.** The same question costs far fewer tokens in a fresh chat than as message #20 in a long thread.
- **Edit your original prompt instead of sending a follow-up.** Follow-ups re-send the entire history. Editing doesn't.
- **Reset proactively every 15–20 messages.** Don't wait until the model degrades. Use the Compact prompt (see Core Strategies) to carry context forward.
- **Don't burn your session window before you're ready to work.** Your usage window starts with your first message. Open the chat when you're ready to start, not to warm up.
- **Batch your heaviest work into the first half of a session.** Rate limits and resets mid-task are more disruptive than starting fresh intentionally.
- **One task per session.** Mixing unrelated tasks fills your context with irrelevant history that Claude pays attention to on every subsequent message.

### Prompting

- **State exactly what you need, upfront.** Vague prompts trigger clarifying questions, which create extra exchanges, which each re-send the entire history.
- **Specify output format and length in the prompt.** "In 3 bullet points" or "under 200 words" prevents Claude from generating a five-paragraph answer when you needed a sentence.
- **Don't repeat context you've already given.** Claude retains everything in the session. Re-pasting the same background info doesn't help it remember it just burns tokens.
- **Ask for structured output over prose.** Tables, bullet points, and JSON carry the same information in 30–50% fewer tokens than narrative paragraphs.
- **Paste excerpts, not full files.** A 50-page document is 10,000–15,000 tokens. The relevant section is 1,500–2,000. Claude also performs better on focused content.
- **Pre-compress documents before uploading.** Run large files through a cheaper model first to strip filler and extract only the facts and recommendations you actually need. (See Core Strategies for the prompt.)

### Model Selection

- **Default to Sonnet.** It handles 90% of tasks well. Opus is not a better Sonnet it's a different tool for a narrower set of problems.
- **Use Haiku for triage, preprocessing, and simple lookups.** Cheap enough that mistakes don't matter. Fast enough to not slow you down.
- **Reserve Opus for high-judgment tasks where reasoning depth changes the answer.** If Sonnet high-effort gives you the same result, use that.
- **Use Sonnet effort levels to tune cost vs quality.** Low for quick execution, Medium for most work, High when the output needs to be right the first time.

### Features and Tools

- **Turn off tools you're not actively using.** Enabled tools (search, code execution) add tokens to the context on every message, even when idle.
- **Set Memory and Preferences once.** Stop re-explaining your context, voice, or constraints at the start of every session.
- **Keep Extended Thinking OFF by default.** It burns tokens on internal reasoning before you see any output. Most tasks don't benefit. See the Extended Thinking section below for when to turn it on.
- **Use /compact and /clear in Claude Code.** Don't let a session exceed ~50,000 tokens without compacting. Guide the summary tell it what to keep.
- **Keep your CLAUDE.md or instruction file short.** A file that's too long means Claude can't prioritize what matters, and it burns tokens on every message. 5–10 specific, actionable rules outperform 50 verbose ones.

### Extended Thinking

Extended Thinking makes Claude reason through a problem internally before responding. That reasoning process burns tokens before you see any output. For most tasks the answer doesn't improve it just costs more.

**Turn it OFF for:**
- Writing tasks (posts, docs, summaries, agendas) output quality doesn't improve with more internal deliberation
- Factual lookups and research web search does the work, not reasoning depth
- Iterative work where you'll give feedback anyway faster to get a draft and correct it than wait for deep reasoning upfront
- Reformatting, cleanup, classification straightforward execution, no reasoning chain needed
- Anything conversational back-and-forth doesn't benefit from it

**Turn it ON for:**
- Problems with multiple valid approaches where the trade-offs are genuinely non-obvious e.g. "how should we structure ownership across these five domains and what breaks under each model?"
- Math, logic, or multi-step problems where getting the sequence wrong invalidates the answer
- Analyzing something with hidden dependencies e.g. "does this architecture decision create a conflict with these three compliance requirements?"
- Debugging where the root cause isn't clear and multiple components could be interacting
- Any task where you already tried without it and the answer came back wrong or too shallow

---

## Global Account Instructions (Copy-Paste)

Paste this into **Settings → Account → Instructions for Claude** (or equivalent in your interface). This runs as background context on every chat without you having to repeat it.

Customize the bracketed sections for your role, output preferences, and working style. Delete anything that doesn't apply.

```
Be concise. Lead with the answer. No preamble, no restating the question, no filler phrases like "Great question" or "Certainly" or "I'd be happy to."

Default to short responses unless the task requires length. If I need more detail I'll ask.

When I ask for a list, give me a list. When I ask for prose, give me prose. Don't switch formats unless I ask.

Prefer structured output over narrative when conveying information tables, bullet points, and code blocks over paragraphs where the content allows it.

Never repeat back context I already gave you. I know what I said. Use it, don't echo it.

If my prompt is ambiguous, make a reasonable assumption and state it, then proceed. Don't ask clarifying questions unless the ambiguity would cause the entire output to be wrong.

When writing, match the tone I use. If I'm direct and informal, be direct and informal. If I write formally, respond formally.

Skip the conclusion paragraph. I don't need a summary of what you just told me.

If you're not sure about something, say so plainly. Don't hedge everything or pad answers with caveats.
```

**What this does:**

- Cuts response verbosity, which directly reduces output token consumption
- Eliminates the back-and-forth that comes from ambiguous prompts
- Prevents Claude from padding answers with summaries and caveats you didn't ask for
- Sets format expectations globally so you don't repeat them per session

**What this does not do:**

- Replace per-session context for specific tasks you still need to give Claude the relevant details for each job
- Override explicit instructions you give mid-chat in-session instructions always take precedence
- Substitute for a well-scoped prompt global instructions set defaults, they don't write your prompts for you

---

## Fundamentals

| Concept | Rule of Thumb |
|---|---|
| Token size | 100 tokens ≈ 75 words. 1 page ≈ 300–400 tokens. |
| Output vs input cost | Output tokens cost ~5× more than input tokens. |
| Context window | 200,000 token max. Entire history re-sent with every message. |
| Accumulation math | A 20-message chat where each exchange averages 1,500 tokens ≈ 315,000 total tokens. Cost is exponential, not linear. |

---

## Match Model to Task

Don't default to Opus for everything. Same answer, much higher bill.

---

### Haiku $ (Cheapest)

Fast, cheap, good enough for tasks where quality ceiling doesn't matter. Use it for triage and preprocessing, not primary work.

| Task | Example |
|---|---|
| Quick factual lookups | "What port does LDAP run on?" / "What's the syntax for a bash for loop?" |
| Reformatting / cleanup | Fixing spacing, stripping extra whitespace, normalizing a copied table |
| Classification | "Is this finding critical, high, medium, or low based on these details?" |
| Pre-compressing a document | Stripping a long report down to facts and recommendations before sending to Sonnet |
| Simple summarization | "Give me 5 bullet points from this article" |
| Short-form copy drafting | A brief announcement, a short description, a one-paragraph summary |
| Checking for conflicts or duplicates in a list | "Do any of these entries overlap?" / "Flag duplicate values in this config" |
| Syntax validation | "Is this rule structured correctly?" / "Does this regex match IPv4?" |
| Extracting specific items from a paste | Pull all URLs / all action items / all version numbers from this block of text |
| Generating a checklist | "List 10 things to verify before deploying a new pipeline" |

---

### Sonnet $$ (Default Workhorse)

Where 90% of your work should happen. Sonnet has effort levels that let you tune cost vs quality for the task at hand.

**Sonnet Effort Levels**

- **Low effort** Lightweight tasks where speed matters more than depth. Equivalent to "just get it done."
- **Medium effort** Default. Good for most writing, analysis, and iterative work.
- **High effort** Engages more reasoning. Use when the output has to be right the first time or requires synthesizing across a lot of information.

| Task | Effort Level | Example |
|---|---|---|
| Promotional post or announcement | Medium | Writing the same announcement in multiple distinct voices for different audiences or authors |
| Analytical post or practitioner take | Medium–High | A breakdown of a recent incident, trend, or industry development aimed at practitioners |
| Meeting or episode agenda drafting | Medium | Structured agenda with timing, talking points, and discussion questions |
| Short-form writing in someone else's voice | Low–Medium | A brief reply, comment, or reaction written to match a specific person's established tone |
| Building a markdown reference document | Medium | Reference guides, framework docs, knowledge bases, onboarding docs |
| Iterating on a config or rules file | Medium | Revising a ruleset, policy file, or structured config across multiple correction cycles |
| Researching a topic with web search | Medium–High | Pulling and synthesizing information from multiple sources on a technical or industry topic |
| Competitive landscape analysis | High | Comparing tools or vendors across a category with capability and positioning breakdown |
| Writing with enforced voice or format rules | High | Content with a strict tone, forbidden phrases, required structure, and specific audience targeting all applied simultaneously |
| Building a structured interview or assessment guide | High | Multi-level question set with expected answers and evaluator notes across multiple topic areas |
| Cross-tool or cross-platform planning | Medium | Mapping unified settings or configurations across multiple tools with conflict resolution |
| Summarizing updates from interviews, releases, or announcements | Low–Medium | Pulling what changed, what's confirmed, and what's still missing from a set of sources |

---

### Opus $$$$$ (High-Judgment Tasks Only)

Reserve for problems where nuance, synthesis across complexity, or multi-layered reasoning genuinely changes the output. If Sonnet high-effort can handle it, use that instead.

| Task | Example |
|---|---|
| Hard architecture trade-off decisions | Should two domains merge or stay separate? What's the right ownership model for a given org structure? |
| Deep multi-file code debugging | Tracing a pipeline failure across 8 interdependent workflows |
| Critical analysis requiring synthesis across multiple domains | How does a new industry report map to your existing controls, content gaps, and program priorities simultaneously? |
| Nuanced long-form technical writing | A practitioner-grade analysis of a major framework, report, or standard not a summary, but a take |
| Evaluating a product category with real tradeoffs | Comparing vendors in a space where the differences are subtle and the marketing all sounds the same |
| Designing a new program or org structure from scratch | Building a multi-domain framework with ownership model, sub-domain scoping, and gap analysis |
| Writing with strict constraints across multiple dimensions | Content with enforced voice, exact terminology, forbidden patterns, specific format, and audience precision all active at once |
| Multi-source research synthesis into a single output | Taking 5+ sources reports, interviews, standards, incidents and producing a single coherent analysis |
| Threat modeling for complex or novel attack surfaces | Mapping attack surface across agentic systems, multi-step delegation chains, or non-standard pipeline architectures |
| Decision analysis with financial or logistical tradeoffs | Evaluating a major decision with competing variables across time, cost, and risk |

---

## Core Strategies

**A1. New conversation for new topics.** Single biggest impact. A new conversation starts at 0 tokens of history. Eliminating 50% of irrelevant context can reduce overall token usage 30–40%.

**A2. Specific and direct prompts.** Vague prompts trigger follow-up questions, which create extra exchanges, which each re-send the entire history. State exactly what you need, specify format and length, include all context in one message.

| Wasteful | Efficient |
|---|---|
| "Can you help me understand what this code does? I'm not sure what's happening here, it seems to be doing something with authentication maybe?" | "Explain what this authentication code does in 2–3 sentences." |
| "Write something about this topic for my blog" | "Write a 300-word blog intro with a headline and 3 body paragraphs about [topic]." |

**A3. Upload excerpts, not full files.** A 50-page document ≈ 10,000–15,000 tokens. The relevant chapter alone ≈ 1,500–2,000 tokens. That's 7–8× fewer tokens and Claude performs better on focused content.

**A4. Use /compact and /clear in Claude Code.** After completing a major task or phase. Guide the summary, don't just run /compact blindly:

```
Compact this conversation but keep the database schema discussion and the API endpoint decisions.
```

Don't let a single session exceed ~50,000 tokens without considering compaction.

**A5. Stop re-injecting context.** Claude retains context across the entire conversation. Repeating background info in multiple messages doesn't help it remember, it just burns input tokens.

**A6. Request structured output.** Structured formats carry the same information in fewer tokens than narrative prose. 30–50% reduction for equivalent information. Ask for JSON, tables, or bullet points instead of paragraphs.

**A7. Keep instruction files concise.** A CLAUDE.md that's too long means Claude can't prioritize what matters, and it burns tokens on every message. 5–10 specific, actionable rules outperform 50 verbose ones.

**A8. Pre-compress documents before uploading.** Run large or image-heavy files through a cheaper model first using this prompt:

```
Read this document end to end. Output a condensed plain-text version that preserves: (1) all factual claims, numbers, dates, and names; (2) every actionable instruction or recommendation; (3) the document's structure as short headings. Drop filler phrases, repeated context, marketing language, formatting artifacts, and page headers/footers. Target 20–30% of the original length. Return only the condensed text, no commentary.
```

---

## Context Degradation

The longer a session runs, the more the model's attention gets split across everything it has seen. It doesn't fail all at once, it degrades gradually. Knowing the symptoms lets you catch it before it costs you work.

### What's Happening

Every message re-sends the full conversation history. As that history grows, the model has to spread its attention across more content. Early instructions get pushed further back and carry less weight. The model doesn't tell you this is happening, it just quietly starts doing worse.

There are two failure modes depending on how full the context window is:

- **Under 50% full:** The model starts losing content from the middle of the conversation. Early instructions and mid-session decisions are the first to fade.
- **Over 50% full:** The model starts losing the earliest content. Your original framing, constraints, and context disappear while recent messages stay sharp.

### Warning Signs

**The model stops following earlier instructions.** You set a format, tone, or constraint at the start of the session and it stops being applied. This is the clearest indicator that early context has been pushed out or down-weighted.

**Responses drift in quality or coherence.** Answers that were sharp early in the session start feeling generic or less precise. The model is no longer working from your full framing.

**The model asks for information you already gave it.** If it asks you to re-explain something covered earlier in the same chat, that content is effectively gone.

**It contradicts earlier decisions.** You agreed on an approach, a format, or a specific output, and now the model is doing something different without being told to change.

**Responses get longer and more hedged.** When context degrades, the model compensates by being more generic and covering more bases instead of giving direct, scoped answers.

**It reads more files or takes more steps than the task needs.** In agentic or Claude Code sessions, degraded context leads to broader, less targeted behavior, more exploration, more tool calls, more output you didn't ask for.

**Token costs climb faster than the volume of work justifies.** If a session is burning more than expected for what you're producing, context bloat is usually the reason.

### What to Do

- **Compact before you hit the wall, not after.** Use the Compact prompt to distill decisions, outputs, and next steps into a portable summary. Paste it into a fresh chat.
- **Restate your key constraints when starting a new session.** A compact summary is not a full reconstruction, so explicitly re-anchor the model on the constraints that matter most.
- **Don't try to correct a degraded session by adding more messages.** Adding more context to a full context window makes the problem worse. Start fresh.
- **In Claude Code, run /context to check how full you are.** Don't guess, look at the number before you decide whether to compact or continue.

---

## Where to Monitor Usage

| Tool | How |
|---|---|
| claude.ai | Settings → Usage |
| Claude Code | `/cost`, current session costs |
| Claude Code | `/context`, full breakdown of what's consuming your context window |
| Cursor | Token % shown in the bottom status bar |

---

## Biggest Token Drains

- Long conversation history (exponential accumulation per message)
- Full file or PDF uploads when only a section is needed
- Verbose AI responses on simple queries ask for less, get less
- Repeated context re-injection across messages
- Dense or poorly structured content pasted without cleanup
- Unused tools left enabled in the session
- Extended Thinking enabled on tasks that don't require it
- CLAUDE.md or instruction files that are too long and unfocused
