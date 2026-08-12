# How to Spot AI-Written Content

*Last updated: April 2026. Patterns shift with every model release. Review periodically.*

---

## Why This Matters: The Dead Internet Problem

You've probably seen it already. Facebook posts getting 20,000 likes that feel hollow. LinkedIn content that's technically correct and completely forgettable. Comment sections that feel like an echo chamber with no actual people in it.

This is what some call the "dead internet." Not literally dead, real people are still here, but increasingly crowded with AI-generated content designed to look human. Bots create posts, generate engagement, build follower counts, then get sold to whoever wants to push a narrative. A 2022 study found that nearly half of all internet traffic was bots, not humans. Social media is now the primary news source for almost half of young adults. When synthetic content can manufacture the *appearance* of consensus, it shapes what millions of people believe about real events.

That's the actual stakes. This guide is about keeping your own judgment intact.

---

## How AI Detection Tools Actually Work (And Why They're Unreliable)

Most detection tools rely on two metrics:

**Perplexity** measures how predictable the text is. AI tends to pick the most statistically likely next word — lower surprise, lower perplexity. Human writing is less predictable. AI text has a median perplexity score around 21.2 vs. 35.9 for human-written abstracts.

**Burstiness** measures variation in sentence length and structure. Humans naturally mix short punchy sentences with long winding ones. AI output stays more uniform.

The problem: these metrics fail in real conditions. Formal writing (academic papers, legal documents, corporate communications) naturally scores low on both. The US Constitution and the Bible have been flagged as AI-generated. Non-native English speakers get hit hardest because they write with simpler, more predictable sentence patterns, producing false positives at disproportionate rates. Well-edited corporate writing can also trip detectors, because polishing text toward clarity reduces the "messiness" that reads as human.

Tool accuracy claims range from 80-99% depending on who you ask. The 99% figures come from vendors testing their own tools. Independent benchmarks put most tools closer to 80%, with some misclassifying up to 27% of human-written content. The perplexity stats you'll see cited (AI around 21.2 vs. human around 35.9) are from specific research contexts, not universal benchmarks. Treat them as approximate. Detection tools are a signal, not proof. They improve and degrade with every new model release. Use them as one input, not a verdict.

**The more reliable method is pattern clustering**, looking for multiple behavioral signals in the same piece of text. No single tell proves AI authorship. Eight tells in the same document is a different story.

---

## Sentence Structure Patterns

- AI keeps sentence length suspiciously uniform. Humans are all over the place. Short. Then three long ones in a row. Then fragments. AI smooths this out.
- AI uses the "comma + present participle" construction far more than humans: *"The team analyzed the data, revealing new insights."* Research shows AI uses this pattern 2-5x more than human writers.
- AI replaces simple copulatives ("is/are") with more elaborate alternatives. "Serves as," "stands as," "marks," "represents" instead of just "is." One study documented a 10%+ drop in "is/are" usage in academic writing after 2023. The swap feels like polish but reads as a tell.
- **Elegant variation:** AI has a built-in repetition penalty to avoid reusing words, which creates unnatural synonym chains. A character gets called "the protagonist," then "the central figure," then "the eponymous hero" in consecutive sentences instead of just their name. Watch for nouns that keep shape-shifting across a paragraph.
- **Latinate bias:** AI consistently prefers longer, Latin-derived words over shorter Anglo-Saxon ones. "Utilize" instead of "use." "Commence" instead of "start." "Demonstrate" instead of "show." "Facilitate" instead of "help." Casual human writing naturally reaches for the short form. AI reaches for the formal one, even when the context doesn't call for it.
- Paragraph structure follows a rigid formula: topic sentence, supporting details, wrap-up sentence. Humans break this constantly. AI almost never does.
- All paragraphs end up roughly the same length. Humans give more space to what fascinates them and one sentence to what doesn't.

---

## Word Choice and Vocabulary

### The Significance Problem

AI cannot resist stating why things matter. A train station gets called a "vital transportation hub." A population figure becomes a "testament to the region's growth." Real writers let significance speak for itself or argue for it with evidence. AI just asserts it, repeatedly, for everything.

Watch for: "marks a pivotal moment," "reflects broader trends," "highlights the enduring legacy," "contributing to the rich tapestry," "underscores the importance of," "serves as a reminder."

### Overused AI Words (as of early 2026)

These words appear statistically more often in AI output than human writing, based on analysis of millions of texts. GPTZero's vocabulary research shows consistent clustering of these terms in AI-generated documents.

**Core list:** delve, leverage, utilize, moreover, furthermore, robust, comprehensive, multifaceted, nuanced, pivotal, crucial, underscores, highlights, tapestry, ecosystem, landscape (abstract noun), paradigm, seamless, streamlined, holistic, foster, cultivate, showcase, garner, bolstered, enduring, intricate

**Phrases:** "it's worth noting," "it's important to remember," "it is crucial to," "no discussion would be complete without," "in today's fast-paced world," "in an increasingly X world," "plays a vital role," "serves as a testament," "leaves a lasting impact," "moving the needle"

**Note on "delve":** peaked with GPT-4 in 2023, dropped sharply by 2025. Vocabulary fingerprints age. The underlying structural patterns are more durable than specific words.

### Per-Model Vocabulary Fingerprints

Different models leave different traces:

- **ChatGPT:** "such as," "certainly," "overall," "sure," "utilize," "various," "typically." Em dashes everywhere. GPT-5.1 made a deliberate attempt to reduce em dash use.
- **Claude:** "here," "according to," "based on," "the text," "while," "appears to," "both," "when." Longer sentences with more hedging. "I'd be happy to" as an opener.
- **Gemini:** "might," "but also," "not only," "helps in." Shorter paragraphs. Occasional emoji. "Great question!" opener.
- **Grok:** "which," "where," "not," "here is."

These are useful when combined with other signals, not on their own.

### Letter and Chatbot Residue

When people copy-paste AI output without cleaning it up, the conversational scaffolding stays:

- Letter openers: "I hope this message finds you well," "Thank you for your time and consideration"
- Chatbot closers: "I hope this helps!" "Of course!" "Let me know if you need anything else!" "Is there anything else I can assist you with?"
- These are major giveaways when they appear in places where no one would naturally write a letter.

---

## Rhetorical Patterns

### Rule of Three
AI groups things in threes constantly. "Speed, efficiency, and reliability." "Identify, evaluate, and recommend." "Clear, concise, and actionable." Real writers use two items, four items, seven, or one. Check any paragraph — if nearly every enumeration is a triplet, that's a pattern.

### Negative Parallelism
"It's not just X, it's Y." "Not X, but Y." AI loves binary oppositions. They structure thought cleanly and sound decisive. The problem is the relentlessness, paragraph after paragraph of balanced contrasts, each perfectly calibrated. Real writers make messy arguments. They contradict themselves. They qualify. AI keeps the antithesis machine running.

### Rhetorical Questions That Aren't Questions
"How do we solve this problem?" "What does this mean for leaders?" AI loves these, especially in clusters of three. But these aren't genuine questions, they're declarative statements wearing question marks. Real writers ask questions they genuinely don't know how to answer. AI's questions are staging.

### The "This Means" Tick
AI labels every logical transition explicitly. "This means several things." "This suggests three approaches." "This underscores the importance of..." It can't trust readers to follow without signposting. Confident human writers make jumps.

---

## Emotional and Tonal Patterns

- AI avoids strong emotions and controversial opinions. Everything is balanced, measured, positive.
- High levels of expressed hope and trust, almost no anger, disgust, or genuine frustration.
- Uniform enthusiasm throughout. If the writer seems equally invested in every section, they're probably not a writer.
- Super optimistic, encouraging, relentlessly helpful tone even when the subject doesn't warrant it.
- No genuine personality. AI can mimic voice, but it optimizes for broad palatability, not for actually sounding like a specific person.
- Humans slip registers. They go formal, then suddenly colloquial, occasionally throw in something that doesn't quite land. AI picks a lane and stays there.

**Risk aversion:** Real writers take risks that might fail. They make metaphors that stretch too far. They assert something a reader might dispute. They write sentences so long you forget how they started. AI never does this. It never writes a sentence that might confuse someone, never makes a joke that might not land, never asserts something without hedging it first. This safety-first approach produces competent, bloodless prose. If nothing in a piece could possibly offend or confuse anyone, anywhere, that's a tell.

---

## Content and Substance Patterns

### Generic Specificity
This is one of the strongest tells. AI provides examples that feel specific without being specific. *"A procurement policy that worked for a manufacturing business might not work for a software company."* True, but which policy? Which company? Human experts cite actual cases with names, dates, what failed, what got fixed. AI constructs plausible-sounding illustrations from genre conventions, not lived experience.

When content discusses concrete topics but never names a single real instance, you're reading synthetic prose.

### Missing Mess
Human first drafts contain false starts, tangents, slight redundancies, moments where the writer changes direction. Even polished human writing shows traces of thinking. AI produces clean prose on the first try. Nothing backtracks. No sudden enthusiasms derail the outline. This perfection is unnatural.

### Real-World Complications Are Absent
AI text sounds textbook-perfect. No "yeah but actually..." moments, no caveats from real experience, no cases where the theory broke down. Expertise comes with knowing where the theory breaks. AI just reports the theory.

### Conclusions That Repeat the Opening
AI conclusions start with "Overall," "In conclusion," or "In summary" and then restate everything already said. Humans either don't summarize at all or use the conclusion to say something new.

### Vague Authority
AI attributes claims to unnamed sources constantly. "Industry experts say..." "Some critics argue..." "Studies show..." "Observers have noted..." without naming anyone, citing anything, or providing a date. When specific sources are named, they may not actually say what's attributed to them.

---

## Structural Patterns

### Relentless Balance
Every section gets equal treatment. If there are pros and cons, they're symmetric. If there are four factors, each gets roughly the same paragraph. Real writers show bias — they spend three paragraphs on what fascinates them and one sentence on the rest. AI allocates attention democratically.

### Formatting Tells
- Heavy use of bullet points and numbered lists where prose would work fine
- Title Case In Every Single Heading Subheading and Section Label
- Skipping heading levels (going straight to H3 without H2) — almost never found in manually formatted documents
- Inline bold headers in bullet lists: `**Key Term:** Description text` — inherited from README files and slide decks
- All paragraphs indented the same way, no variation
- Long sentences packed with commas (20+ words on average)

### The Significance Bookend
Many AI-generated pieces include a "Challenges" section formatted as: *"Despite its [positive description], [subject] faces challenges..."* ending with vague optimism about future potential. The rigid formula is the tell, not the mention of challenges itself.

---

## Facts and Sources

- Hallucinated citations that look real: DOIs leading to unrelated papers, ISBNs failing checksum verification, book citations with no page numbers, multiple broken external links in a new article
- When AI tools with web access generate sources, ChatGPT may append `utm_source=chatgpt.com` to URLs — this is near-definitive proof of ChatGPT involvement in sourcing
- Sources clustered in the same narrow time window, or all from the same type of outlet
- Specific claims cited to sources that don't actually contain them

---

## Feelings About Being Checked

When AI-generated content is challenged, the response often follows a recognizable pattern:

- Formal first-person paragraphs with no abbreviations, explicitly echoing policy language
- "I can assure you that my contributions align with the standards of..."
- Itemized lists of things they "ensured" or "avoided"
- Offers to receive feedback and "make adjustments" — phrased in ways humans almost never actually write
- When asked to rewrite suspected AI content, AI swaps synonyms instead of restructuring sentences

---

## What Doesn't Work as a Detection Signal

False accusations cause real harm. These are commonly cited but ineffective as standalone indicators:

- **Perfect grammar** — many humans are skilled writers
- **Formal or academic tone** — context-dependent, not inherently suspicious
- **"Bland" or "robotic" prose** — AI skews positive and verbose, not robotic; people unfamiliar with AI output often don't recognize it
- **Transition words in isolation** — humans use "additionally," "furthermore" too
- **Unsourced content** — most of Wikipedia predates LLMs; modern AI often includes citations (just sometimes fake ones)
- **Non-native English speaker patterns** — simpler, more predictable sentence structure can trigger false positives without any AI involvement

---

## Pattern Clusters Are the Point

No single signal proves AI authorship. One em dash, one "robust," one tricolon proves nothing. The tell is when you find eight or ten of these signals in the same piece of text. That's the method: look for clustering, not individual instances.

Pattern signatures also shift with every model release. "Delve" peaked in 2023 and faded by 2025. Em dash use dropped in GPT-5.1 after it became notorious. New patterns replace old ones. The vocabulary fingerprint ages faster than the structural patterns, which are more durable.

By early 2026, 15% of Reddit posts are AI-generated, and 21% of ICLR 2026 academic reviews were written entirely by AI. Detection skills matter more now than they did two years ago, and will matter more still in two more.

---

## How to Use AI Without Leaving These Signals

AI should sharpen your ideas, not replace them. Write your actual thoughts first, your opinions, your specific examples, your rough sentences. Then use AI to clean grammar, restructure confusing sections, or find gaps in your argument.

When AI creates everything from scratch, you lose what makes content worth reading: your real voice, your specific experience, your original framing. The goal is AI-assisted human writing, not human-edited AI output.

### Quick Editing Checklist Before Publishing

Remove:
- AI vocabulary words from the list above
- Letter and chatbot opener/closer phrases
- All instances of grouping in threes
- "It's important to note that" and similar preambles
- Conclusions that just restate the opening
- Any example that could apply to any company, anywhere, at any time

Add:
- At least one specific real name, date, or case
- At least one sentence where you contradict yourself or qualify something
- Varied sentence lengths, including some very short ones
- One "yeah but actually" that comes from real experience, not the textbook version

### Prompt Template for More Human Output

```
Write about [TOPIC] in [casual/professional/conversational] tone.

AVOID:
- These words: delve, leverage, utilize, moreover, furthermore, comprehensive, robust,
  pivotal, showcase, holistic, multifaceted, tapestry, ecosystem, underscores, seamless,
  facilitate, commence, demonstrate (prefer: help, start, show)
- Letter phrases: "I hope this message finds you well," "Thank you for your time"
- Chatbot closers: "I hope this helps!" "Let me know if you need anything else"
- Importance language: "plays a vital role," "serves as a testament," "watershed moment"
- Preambles: "It's important to note," "It's worth remembering"
- Em dashes and semicolons
- Grouping everything in threes
- Conclusions that start with "Overall," "In conclusion," or "In summary"
- Perfectly symmetric pros/cons structure
- Generic examples with no names or specifics
- Latinate word choices when a shorter Anglo-Saxon word exists

INCLUDE:
- Mixed sentence lengths, including some very short ones
- Contractions
- At least one specific example with a real name or number
- The "yeah but actually" version, where the simple answer breaks down
- An opinion, stated directly
- Incomplete thoughts when that's what the moment calls for

Rewrite in my voice. No framing sentences. No summaries. Keep it concrete and slightly uneven.
```

---

*Sources: Wikipedia Signs of AI Writing (April 2026), Cherryleaf AI writing indicators (Feb 2026), CNET AI detection guide (April 2026), GPTZero AI Vocabulary research (3.3M text dataset), Carnegie Mellon AI writing pattern research (2025), SEOengine.ai AI writing signals analysis (Feb 2026), Pangram Labs detection research, UCLA HumTech AI detection review, Gonzaga University AI detector guide, peer-reviewed studies on LLM vocabulary patterns in biomedical and academic writing (2023-2026), Wikipedia AI Idiosyncrasies study (Sun et al., 2025).*

---

| | |
|---|---|
| **Last Updated** | April 20, 2026 |
| **Based On** | 20+ sources, November 2025 - April 2026 |
| **Version** | 2026.3 (April Update) |
| **Next Review** | July 2026 |
| **Note** | Vocabulary patterns age faster than structural patterns. Structural section is more durable. Vocabulary list should be reviewed each quarter. |