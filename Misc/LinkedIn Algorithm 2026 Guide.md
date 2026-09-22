# LinkedIn Algorithm 2026: Changes and Strategy Guide
Updated September 2026. Version 2026.5. Built from 60+ sources, November 2025 to September 2026.

---

## Contents

Overview: Executive Summary, What Changed, 2026 Timeline, Quick Wins

Part 1: How LinkedIn Ranks Posts
1. How the Algorithm Works
2. Engagement Hierarchy
3. Attention and Engagement Quality
4. The AI Slop Crackdown
5. What Gets Suppressed

Part 2: Set Up Your Profile
6. Positioning and Profile-to-Content Match
7. Personal Profile vs Company Page

Part 3: Create Posts
8. Post Types and Content Mix
9. The Hook
10. Post Length and Structure
11. Formatting
12. Hashtags: Dead
13. Links
14. Format Guide
15. Video
16. Newsletters and Articles
17. Pre-Publish Checklist

Part 4: Publish and Engage
18. Posting Frequency and Timing
19. The Golden Hour (First 60 to 90 Minutes)
20. Comments
21. Tagging and Collaboration
22. Editing, Reposting, and Repurposing
23. Weekly Cadence Example

Part 5: Measure and Recover
24. Analytics
25. Diagnostic and 30-Day Recovery

Reference
26. Contested and Unverified Claims
27. Watch List

---

## Executive Summary

What changed in 2026:

- LinkedIn rebuilt the feed on LLMs. Retrieval uses LLM embeddings that match posts to professional interests; ranking uses a sequential transformer (Generative Recommender) that reads each member's last 1,000+ interactions in order.
- Generic AI content is capped at your immediate network. LinkedIn says it identifies generic content correctly 94% of the time.
- Since July 30, any member can flag a post as "Seems like AI slop." By August 20, slop-classified content was getting 40% fewer views.
- LinkedIn removed its own "Enhance your post" AI writer and replaced it with a grammar-only proofreader.
- Profile-to-content match determines who your post is tested against. Your headline and About must confirm expertise in what you post about.
- Reshares with commentary, saves, and substantive comments outweigh likes.
- Document posts (PDF carousels) lead engagement at about 7.0%.
- External links reduce reach, but datasets disagree on how much (see section 13).
- Hashtags are dead. Skip them.
- Company pages get about 5% of feed distribution; personal profiles about 65%.
- Reach reset lower for everyone: down about 60% over two years for active creators. Engagement per impression is up.
- LinkedIn's own posting guidance: 2 to 5 posts per week, 1 to 2 of them video.
- LinkedIn Live must be scheduled as an event since June 22.

Bottom line: match your profile to one niche, write from real experience, create document posts, optimize for saves and reshares, skip hashtags, never automate engagement.

---

## What Changed: 2025 vs 2026

| Before | Now |
|---|---|
| Relationship graph (who you know) | Interest graph (what you know and who cares) |
| Hashtag discovery | Semantic reading of full post text |
| Recency | Relevance + topic consistency |
| Any engagement counts | Saves, reshares, substantive comments count |
| 24 to 48 hour lifespan | Days to weeks for relevant posts |
| Company pages | Personal profiles |
| Volume and automation | Human voice and real experience |
| AI rewrites tolerated | Generic AI content held to your network |

The numbers:
- Reach down ~60% over two years for active creators
- Typical reach: 8 to 12% of followers, down from 15 to 20%
- Top creators' share of feed visibility rose from 15% to 31% since 2022; everyone else fell from 57% to 28%
- Posting volume up ~15% YoY
- Engagement per impression up
- Company page organic reach down 60 to 66%

What wins:
- Consistent expertise in 2 to 3 lanes
- Real experience with specifics
- Personal profile over company page
- Document posts and short video
- Substantive comments over likes
- Human voice and real experience

---

## 2026 Timeline

| Date | Change |
|---|---|
| Dec 2025 | LinkedIn says hashtags "play a much smaller role"; hashtag following and profile hashtag fields removed |
| Late 2025 to Mar 2026 | Feed retrieval moved to a single LLM embedding system; ranking moved to a sequential transformer |
| Mar 12 | LinkedIn engineering blog details the new feed architecture |
| Mar 2026 | Industry-named "Authenticity Update": pods, automation tools, engagement bait, link-in-comment workaround, and polls lose reach |
| Mar 2026 | Interest Picker: new members declare topic interests at signup |
| Mar 29 | LinkedIn announces Live must be scheduled starting June 22 |
| May 2026 | LinkedIn video guidance: 2 to 5 posts per week, 1 to 2 videos, grounded in real experience |
| May 20 | "Keeping conversations real on LinkedIn": generic AI posts and automated comments held to immediate network |
| Jun 2026 | Post analytics show in-network vs out-of-network reach split under Discovery |
| Jun 22 | Spontaneous LinkedIn Live ends; every broadcast needs a scheduled event |
| Jul 30 | "Seems like AI slop" report on posts and comments; "Enhance your post" removed; private dashboard warnings for creators |
| Aug 2026 | Collaborative posts ("Add Collaborators") in testing |
| Aug 20 | 1M+ members used the slop report in two weeks; slop-classified content down 40% in views |
| Sep 2026 | EU DSA filing: 46% more inauthentic activity detected in H1 2026 vs H2 2025 |

---

## Quick Wins

Do now:
- Align headline and About to your 2 to 3 topic lanes
- Complete LinkedIn verification
- Create one document post per week (6 to 12 slides)
- Post 2 to 5 times per week, 24+ hours apart
- Add 1 to 2 short videos per week
- 10 to 15 minutes of substantive commenting daily
- Be present 60 to 90 minutes after posting
- Track out-of-network % on every post
- Check mobile preview before publishing

Stop doing:
- Automated comments
- Hashtags
- Engagement pods or coordinated early comments
- Polls
- "Link in comments"
- Promotional links in the post body
- More than one post per 24 hours
- Reposting identical text
- Bait endings ("Agree?", "Comment YES")
- Stock images
- Generic posts outside your lanes
- Relying on company page reach

---

# Part 1: How LinkedIn Ranks Posts

---

## 1) How the Algorithm Works

### Stage 1: Retrieval
LLM embeddings read your post, your profile, and each viewer's interests, then pull candidate posts, including from creators the viewer doesn't follow if the topic fits. New members with little history get matched from profile data and Interest Picker choices.

### Stage 2: Quality filter
Posts are classified before distribution. Filtered or demoted:
- Generic AI content lacking perspective (slop classifier)
- Automated comments and comments that only restate the post
- Engagement bait ("Comment YES", "Agree?")
- Engagement pod and automation patterns
- Video/text mismatches
- External links (reduced, see section 13)

If the system can't clearly identify your topic, distribution suffers from the start.

### Stage 3: Test audience
The post goes to a small sample: your most engaged connections plus people interested in the topic. The first 60 to 90 minutes decide whether it expands.

What gets measured: dwell time, "see more" expansion, saves, substantive comments, reply threads, reshares, profile clicks.

### Stage 4: Ranking and expansion
The transformer predicts each viewer's likely engagement from the order of their past interactions. Posts that were shown and ignored are used as negative training examples, so posts people scroll past teach the model to show you less.

Strong posts expand to 2nd and 3rd degree and to cold interest-matched audiences, and can resurface days or weeks later. Relevance beats recency.

### Topic consistency
Posting on the same topics for 90+ days builds recognition of your expertise and stronger distribution for new posts in that lane. Topic jumps dilute it.

---

## 2) Engagement Hierarchy

Every action the ranker weighs, strongest to weakest. LinkedIn does not publish exact weights; the order combines LinkedIn statements with large independent datasets. Saves > comments > likes has dataset support; the rest of the order is practitioner consensus.

### Positive actions

| Rank | Action | What it tells the ranker |
|---|---|---|
| 1 | Dwell time | The reader stopped and spent time on the post |
| 2 | Saves | The post has reference value worth returning to |
| 3 | Reshare with added commentary | The resharer vouches for it in front of their own network; LinkedIn has named reshares as one of its clearest indicators of value |
| 4 | Substantive comment (15+ words, adds something) | The post started real discussion |
| 5 | Reply threads (back-and-forth, including your replies) | Conversation is continuing; threads were associated with up to 2.4x more reach |
| 6 | Sends (private share via message) | The reader thought a specific person needed to see it |
| 7 | "See more" expansion | The hook earned a full read |
| 8 | Document swipes and video watch time | How much of the content was actually consumed |
| 9 | Profile click from the post | The post created interest in you |
| 10 | Follow from the post | The post converted a reader into audience |
| 11 | Plain reshare (repost, no commentary) | Some endorsement, no added context |
| 12 | Short or generic comment ("Great post!", emoji-only, "Following") | Minimal; often filtered out entirely |
| 13 | Likes and all reaction types | Lowest weight of any positive action; no public evidence that "Insightful" or "Celebrate" counts more than "Like" |

Relative weights from AuthoredUp's analysis of 621,833 posts: a comment is worth roughly 2x a like, and a save roughly 2x a comment.

### Negative actions (reduce distribution)

| Action | Effect |
|---|---|
| Scrolling past without stopping | Shown-and-ignored posts are used as negative training examples, so the model learns to show your posts less |
| Clicking in and leaving immediately | Indicates the hook overpromised |
| "Not interested" or hide post | Removes you from that viewer's feed and lowers predicted interest for similar viewers |
| Unfollow or mute after seeing the post | Strongest per-viewer negative |
| "Seems like AI slop" report | Mainly changes the reporter's feed; broad distribution drops when many members report the same post |
| Automated, pod, or coordinated engagement on the post | Detected and discounted; can reduce distribution |

### Content built for likes vs reshares
- Likes: short, relatable, emotionally validating
- Reshares: something the reader wants their own audience to see; teaches something specific, makes the sharer look informed, states a clear point of view they couldn't have put as well

Build both deliberately. They are different strategies, and reshares are worth far more.

---

## 3) Attention and Engagement Quality

The ranker weighs how people spend time with your post and who engages, not reaction counts. The "Depth Score" name circulating in marketing content is not a LinkedIn term, but the components are real. Section 2 ranks the individual actions; this section covers what makes that engagement count.

### What makes engagement high quality
- Time spent: dwell time relative to the format. 30 seconds is long for an image post and short for a video.
- Consumption: "see more" expansion, slides swiped, video watched to the end
- Substance: comments that add a data point, experience, or counterpoint, and replies that build threads
- Relevance of who engages: engagement from people in your topic lanes and target audience moves the post further than engagement from unrelated accounts
- Authenticity: engagement from real, active, ideally verified members; automated or coordinated engagement is discounted
- Durability: saves and reshares keep a post circulating for days or weeks after the first hours

### Do
- Deliver value in the first two lines
- Write content worth saving: frameworks, checklists, specific numbers
- Give readers something to add to, not just agree with
- End with a specific question tied to the content
- Use document posts for multi-step material

### Don't
- Use clickbait that the post doesn't pay off
- Bury the point
- Chase likes over saves, reshares, and comments
- Pad posts to increase length; padding lowers dwell per line and gets skimmed

---

## 4) The AI Slop Crackdown

Timeline: May 20 policy announcement, July 30 report button, August 20 results.

LinkedIn defines AI slop as content that is polished in presentation but lacks substance: no particular experience, perspective, or insight.

LinkedIn targets:
- Posts that read generic or repetitive, even if polished
- Comments posted at scale by automation
- Replies that restate the post without adding anything

Effects:
- Distribution held to your immediate network
- Slop-classified content down 40% in views since the report button launched
- Individual reports mainly change the reporter's own feed; broad distribution drops when many members report the same content
- Heavily reported creators get a private warning in analytics

Context: Pangram scanned ~1M posts and found 40%+ of long-form LinkedIn posts fully AI-generated. Readers now scroll past model-sounding text faster, which cuts dwell time independent of any classifier.

---

## 5) What Gets Suppressed

1. Generic AI content (held to immediate network)
2. Automated or AI-generated comments
3. Engagement pods and coordinated engagement
4. Engagement bait ("Comment YES", "Tag someone")
5. Heavy slop reports on the same post
6. External links (magnitude disputed)
7. Multiple posts within 24 hours
8. Recycled identical content
9. Hashtags and emoji spam
10. Video/text mismatches

The full list of negative reader actions is in section 2.

---

# Part 2: Set Up Your Profile

---

## 6) Positioning and Profile-to-Content Match

LinkedIn checks whether your profile confirms you're a credible source on what you post about. A mismatch costs reach before anyone sees the post.

Do:
- Pick 2 to 3 topic lanes and stay in them
- Headline states your expertise in the same words your posts use
- About section: first 3 lines state your lanes in plain language
- Featured section: pin 2 to 3 posts or documents that represent your lanes
- Complete every profile section, banner, and headshot
- Complete LinkedIn verification (viewers can filter comments and conversations to verified members)
- Post from your personal profile, not a company page

Practical check: cover your name on your last 5 posts. Could someone else in your field have written them? If yes, that's the problem.

Interest Picker: new members pick topics at signup, which creates a cold-audience path to reach people who have never interacted with you. This makes topic consistency more valuable.

---

## 7) Personal Profile vs Company Page

- Company pages: ~5% of feed distribution; personal profiles ~65%
- Employee posts reach 561% further than company page posts
- Company page organic reach down 60 to 66% from 2024 to 2026

Do:
- Build the personal profile as the primary channel
- Use the company page for jobs, official news, and amplifying employee content

Don't:
- Rely on company page organic reach

---

# Part 3: Create Posts

---

## 8) Post Types and Content Mix

| Type | Example | Why it works | Format |
|---|---|---|---|
| Field report | "We rolled out X across N repos. Here's what broke." | Specific, hard to fake | Text or text + screenshot |
| Contrarian take with evidence | Disagree with common advice, show the case | Real debate in comments | Text |
| Framework / checklist | Reusable model people reference later | Highest save rate | Document post |
| Teardown | Incident, breach report, standard, or vendor claim | Timely + expertise | Text or document |
| Decision log | A call you made, the tradeoffs, what it cost | Rare, high trust | Text |
| News + your read | Practitioner view within 24 to 48 hours of news | Retrieval favors timely topical posts | Text or video |
| Mistake / lesson | What you got wrong and what changed | High comment rate | Text |
| Behind the scenes | How a program, team, or tool actually runs | Differentiated | Text + real photo |

Content mix:
- Educational frameworks and how-to: 30 to 40%
- Industry commentary and analysis: 25 to 30%
- Personal stories with professional lessons: 20 to 25%
- Updates (speaking, launches, milestones): 10 to 15%

What underperforms: generic career advice, motivational posts, "excited to announce," listicles anyone could write, polls, link drops, pure self-promotion, fear-mongering without solutions, jargon without context.

---

## 9) The Hook

LinkedIn shows roughly the first 210 characters before "see more" in most 2026 sources; the exact cutoff varies by device and screen. Check the mobile preview before posting.

Patterns that work:
- Specific number or result: "We cut 4,000 SAST findings to 212 without touching a rule."
- Concrete scene: "The deploy was blocked at 11pm by a scanner flagging a test fixture."
- Direct claim you can defend: "Most SBOM programs produce documents that are never read."
- Stat hooks outperformed direct hooks in one dataset (1.67x vs 1.45x); imperative hooks ("Stop doing X") underperformed

Patterns that fail:
- Throat clearing: "I've been thinking a lot about..."
- "Excited to share..."
- Vague curiosity bait the post doesn't deliver on
- Context-free opening question
- Anything that reads like a hook template

Test: from the first two lines alone, would a stranger know the topic and have a reason to expand?

---

## 10) Post Length and Structure

Length by post type:

| Post type | Characters |
|---|---|
| Single take, news reaction, contrarian point | 600 to 1,200 |
| Field report, decision log, story | 1,000 to 1,600 |
| Teardown or framework in text | 1,300 to 1,900 |
| Caption on a document post or video | 150 to 500 |

What the data says:
- No single length wins. ConnectSafely (10K posts, own users) favors 1,300 to 1,900; Final Layer favors 800 to 1,100 for engagement; others favor 400+ words only when content justifies it.
- Short posts tend to spike quick reactions; medium posts tend to drive deeper comments and longer reach.
- Hook, topic relevance, and a clear takeaway matter more than character count.
- Under ~400 characters generally underperforms for text-only posts.
- Rule: long enough for one complete idea with evidence, no longer. Padding kills dwell time.

Your analytics override these ranges. Test length against out-of-network reach % and saves, not likes alone. Personal observation: 600 to 1,200 character posts have outperformed longer ones.

Structure:
1. Hook
2. Context
3. Evidence (a number, system, or example in the first third)
4. The insight or framework
5. Specific question or your actual point

Do:
- One idea per post
- Short paragraphs (1 to 3 lines) for mobile
- Uneven paragraph lengths (reads human)
- Numbered lists for real steps

Don't:
- Walls of text
- One sentence per line (reads templated)
- Perfectly even paragraphs and parallel bullets
- Lesson-summary paragraph restating the post

Specificity requirement, at least two per post:
- A real number, tool, timeframe, or system
- A decision and what it cost
- An opinion a peer would argue with
- Something that failed
- A detail only someone doing the work would know

---

## 11) Formatting

Do:
- 1 to 3 emojis, for emphasis only
- Line breaks between paragraphs
- Mobile preview before posting

Don't:
- More than 5 emojis (spam pattern)
- Emoji bullet lists (template tell)
- Unicode bold/italic generator text (LinkedIn has no native bold; screen readers and classifiers handle it poorly)
- Excessive special characters

---

## 12) Hashtags: Dead

LinkedIn reads posts semantically. Hashtags provide no discoverability. LinkedIn removed hashtag following, hashtag fields on profiles, and Creator Mode hashtag topics.

Do:
- Skip hashtags entirely
- Use natural topic keywords in your copy

Don't:
- Use hashtags expecting any benefit
- Use generic tags (#Marketing, #Leadership)

---

## 13) Links

Datasets conflict:

| Source | Finding |
|---|---|
| van der Blom, 1.3M posts | One external link in the post body cuts median reach 18.8%; comments with external links see visibility cut up to 80% |
| Saywhat, ~400K posts | Posts with multiple useful resource links outperformed no-link posts |
| Various practitioner sources | 40 to 60% reduction; some say first-comment links still work if the body is fully native |

Working rule:
- Default to no link. Deliver the value in the post itself.
- If the links are the value (resource roundup), include them and accept the tradeoff.
- Promotional links to your own site or content are the case most likely to lose reach.
- Don't write "link in comments." The bridging pattern is detected.
- If you need a link, add it in a reply after conversation starts, and remove the link preview card.
- Make the post valuable enough that people DM you for the resource.

---

## 14) Format Guide

| Format | Data | Use for |
|---|---|---|
| Document (PDF carousel) | ~7.0% engagement (Socialinsider, 1.3M posts); 2.3x median reach (LinkPost, 438K posts) | Frameworks, checklists, processes, benchmarks |
| Multi-image | ~6.45% engagement | Before/after, screenshot series |
| Native video | ~6.0% engagement; short-form growing fastest | Short explainers, talk and podcast clips, takes on news |
| Text | Lower average rate, strong when specific | Takes, field reports, stories |
| Text + real image | Middle | Screenshots, whiteboards, event photos (no stock) |
| Newsletter | Subscriber email + notification, bypasses feed ranking | Weekly or biweekly long-form |
| Article | Low feed reach; indexed and cited in AI search | Evergreen reference pieces |
| LinkedIn Live | Must be scheduled as an event since June 22 | Planned sessions, 30 to 45 min |
| Poll | ~0.07% engagement in 2026 data | Don't |

### Document posts
- 6 to 12 slides; engagement drops past ~10
- 1080x1080 or 1080x1350, exported as PDF
- Slide 1 is the hook; one point per slide; text readable on a phone
- Real content, not a stretched text post; templated carousels read as low effort
- Caption of 150 to 500 characters explaining why you made it
- Final slide: your point or a specific question, not "Like / Save / Follow"
- Watch item: LinkedIn has been shrinking carousel display size without explanation

### Single images
Use real screenshots, diagrams, or photos. No stock images.

---

## 15) Video

LinkedIn's guidance: videos grounded in real experience and a clear point of view perform best; share your view on industry news, break down trends, or talk through career lessons.

Do:
- 30 to 90 seconds for feed video
- Vertical, captions on, native upload
- Strong hook in the first seconds; get to the point immediately
- Simple backdrop, no distracting visuals
- Clip existing recordings (talks, podcasts, panels): one hour of recording usually holds 4 to 8 usable clips
- Write the caption from the transcript so it keeps your actual phrasing

Don't:
- Long intros
- Skip captions
- Embed YouTube or Vimeo links instead of uploading natively
- Assume sound is on

Specific percentages circulating for vertical vs square vs horizontal, first-3-second retention, and muted viewing have no traceable source. Direction is consistent: vertical, short, captioned.

Live: schedule the event in advance (minutes is enough), promote it ahead of time, clip it afterward.

---

## 16) Newsletters and Articles

- Any member can start a newsletter; creator mode is no longer required
- Video covers and email metrics (sends, open rate) are available
- Subscribers get email and notifications, which bypasses feed ranking
- LinkedIn ranks #2 in citations across ChatGPT Search, Perplexity, and Google AI Mode
- Articles get low feed reach but build long-term discoverability and topic credibility

Use a newsletter on your core lane with a fixed structure you can sustain.

---

## 17) Pre-Publish Checklist

Content
- [ ] One main idea
- [ ] Inside one of my topic lanes
- [ ] At least two specifics only I could provide
- [ ] Contains an opinion or decision, not just information
- [ ] I would say every sentence out loud to a peer

Hook
- [ ] First ~210 characters state the topic and give a reason to expand
- [ ] No announcement, throat clearing, or template hook

Form
- [ ] Length fits the post type (600 to 1,200 for a single take; longer only when evidence needs it), or 6 to 12 slides for a document post
- [ ] No hashtags
- [ ] No external link unless the links are the value
- [ ] No bait ending
- [ ] Mobile preview checked

Timing
- [ ] Not a second post today
- [ ] I can be present for the next 60 to 90 minutes
- [ ] Commented on 3 to 5 posts in my lanes today

---

# Part 4: Publish and Engage

---

## 18) Posting Frequency and Timing

### How often
Do:
- 2 to 5 posts per week, 1 to 2 of them video
- 24 hours minimum between posts; 48 to 72 hours is safer
- Consistency over volume; three strong posts beat five rushed ones

Don't:
- Post more than once in 24 hours (you cannibalize your own reach)
- Post daily without guaranteed quality
- Post just to stay visible

Your experience validated: posting within 6 to 7 hours of a previous post kills the first post's engagement.

### When
- Post when you can be present for 60 to 90 minutes afterward; this matters more than clock time
- Tuesday to Thursday, 7 to 9 AM or 2 to 3 PM in your audience's time zone is the common default
- Avoid late evenings and weekends for B2B audiences
- Check your own analytics for follower time zones
- Scheduling tools do not reduce reach

---

## 19) The Golden Hour (First 60 to 90 Minutes)

Do:
- Post when you can be present for the next 60 to 90 minutes
- Reply to comments with substance and a follow-up question to build threads. Threaded replies were associated with up to 2.4x more reach.
- Spread replies across the window instead of answering everything in five minutes
- Keep replying after the window; the top 1% of creators reply about 134 times per week vs 38 for average creators, and reply activity correlated with follower growth (0.41)

Don't:
- Post and ghost
- Schedule posts for times you're unavailable
- Reply with "Thanks!" only
- Coordinate colleagues to comment at a set time on every post (pod detection targets this pattern)

---

## 20) Comments

Comments rank above likes; AuthoredUp's analysis of 621,833 posts puts a comment at roughly 2x a like, with a save roughly 2x a comment.

On your posts:
- Respond to all comments, spread over the first hours
- Ask follow-up questions that start threads

On others' posts:
- 10 to 15 minutes a day commenting in your topic lanes, on peers and larger creators your audience follows
- 15+ words, adding a data point, counterexample, or specific experience
- Comment on 2nd-degree connections to appear in their networks
- Comment before you post to warm up the audience your post will be tested against

Don't:
- "Great post!", "Following!", emoji-only, or one-word replies
- Comments that restate the post
- Automated comments or comment tools, ever

Comment quality hierarchy:
1. Multi-sentence, adds something new
2. Shares relevant experience or counterpoint
3. Asks a clarifying question
4. Generic positive reaction (minimal value)

---

## 21) Tagging and Collaboration

Do:
- Tag 1 to 5 people who are part of the story (co-hosts, collaborators, people you cite)
- Respond when tagged
- Use Add Collaborators (in testing) for co-authored launches and joint work

Don't:
- Mass-tag
- Tag people irrelevant to the post

---

## 22) Editing, Reposting, and Repurposing

Editing:
- Fix typos and add context early
- No major rewrites after engagement starts
- Don't delete and repost

Repurposing:
- Transform, don't recycle: text to document post, document to video, post to newsletter section
- Spread repurposed pieces 1 to 3 weeks apart with different angles
- Share others' posts with your own added perspective
- Never repost identical text; recycled content is devalued

---

## 23) Weekly Cadence Example

- Post 1: Document post with a framework or checklist
- Post 2: Text post with a contrarian take or teardown
- Post 3: Field report, decision log, or behind-the-scenes story
- Post 4 (optional): 30 to 90 second video clip

Every week:
- 10 to 15 minutes a day commenting in your lanes
- 60 to 90 minutes present after each post
- No two posts within 24 hours
- No hashtags, no bait, no link-in-comments

---

# Part 5: Measure and Recover

---

## 24) Analytics

Primary metrics:

| Metric | What it tells you | Where |
|---|---|---|
| Out-of-network reach % | Whether the post traveled beyond your network or was held inside it | Post analytics, Discovery section under impressions |
| Saves | Reference value | Post analytics |
| Substantive comments | Conversation quality | Manual count |
| Reshares with commentary | Whether readers vouched for it | Post analytics |
| Profile views and follows from post | Whether the post built authority | Post analytics |
| Slop / inauthentic warning | Direct penalty indicator | Analytics dashboard |

Secondary:
- Engagement rate (target >2%; document posts 6%+)
- Who engages (titles, seniority, industries vs your target audience)
- Format performance (document vs video vs text)

Business outcomes:
- Profile-to-connection conversion
- Inbound messages and opportunities
- Podcast listens and speaking/consulting inquiries from LinkedIn

Ignore as success metrics: total impressions (baseline reset), likes alone, follower count alone.

---

## 25) Diagnostic and 30-Day Recovery

### Warning signs
- Out-of-network reach % collapsing while in-network holds steady (classifier holding posts to your network)
- Slop or inauthentic warning in analytics
- Comments drying up or coming only from the same small group
- No profile visits from posts

### Common causes
- Posts that read polished but generic, without specifics
- Topic drift outside your lanes
- Link-heavy promotional posts
- Automation or pod patterns
- Posting too frequently
- Profile not matching post topics

### 30-day recovery experiment
1. Baseline: pull the last 15 posts; record out-of-network %, saves, substantive comments.
2. Weeks 1 to 4: 3 to 4 posts per week applying the specificity rules in section 10. At least one document post per week. No links. Daily commenting.
3. Log each post: type, format, hook type, length, out-of-network % at 48 hours, saves, substantive comments.
4. Day 30: compare against baseline by post type. Double down on the top two types; drop the bottom one.

Expect lag. The ranker needs new interaction history before it changes how it routes your posts.

---

# Reference

---

## 26) Contested and Unverified Claims

| Claim you'll see | Status |
|---|---|
| "360Brew is the live feed algorithm" | Research model published by LinkedIn in 2025, labeled pre-production. LinkedIn's engineering blog describes a different ranker. |
| "Depth Score" is a LinkedIn metric | Marketing term; no LinkedIn source |
| "AI content gets -30% / -47% reach" | No primary source. LinkedIn's figure: 40% fewer views for slop-classified content. |
| "LinkedIn cannot detect AI" | Contradicted by LinkedIn. Detection targets generic content, not AI use itself. |
| "Comments are worth 15x likes" | No primary source. AuthoredUp: ~2x. |
| "Links always cut reach 60%" | Datasets range from 18.8% to 60%, and one finds resource links help |
| "Pod detection is 97% accurate" | No LinkedIn source |
| "Only 5% of posts recover after a weak first hour" | No source |
| "Golden hour decides 70% of reach" | No source |
| "15-minute replies = 90% boost" | No source |
| "SSI score affects feed visibility" | No evidence the feed ranker uses SSI |
| "50-70 connection requests per day hard cap" | Vendor figure, not LinkedIn documentation |
| "Recycled content gets 84% less reach" | No source |
| "Dramatic topic shifts = 43 days reduced reach" | No source |
| "A single slop report tanks a post" | False per LinkedIn; broad impact requires many reports |
| "Posts need 31 to 60 seconds dwell" | Single source; thresholds are relative to format |
| "1,200 to 1,900 characters is the optimal length" | Vendor datasets conflict (800 to 1,100 vs 1,300 to 1,900); length depends on post type and audience |
| Video format percentages (vertical +80%, etc.) | No traceable source |
| "Followers see 25-30%, connections 10-15%" | Conflicts with van der Blom's 8 to 12% of followers overall |
| "Creator Mode boosts distribution" | Creator Mode hashtag topics retired; newsletters no longer require it |

---

## 27) Watch List

- Carousel display size shrinking
- Slop warning rollout in creator analytics
- Collaborative posts moving from test to general availability
- Personalized suggested feed test
- Further changes to link treatment
- Expansion of the immersive video tab and carousel

---

## Sources

### Official (LinkedIn and executive statements)
1. LinkedIn Pressroom, "Keeping conversations real on LinkedIn" (Jun 4, 2026) https://news.linkedin.com/2026/keeping-conversations-real-on-linkedin
2. TechCrunch, LinkedIn slop button (Jul 30, 2026) https://techcrunch.com/2026/07/30/linkedin-adds-a-button-to-report-ai-generated-slop/
3. Social Media Today, "LinkedIn offers the option to report AI slop" https://www.socialmediatoday.com/news/linkedin-offers-the-option-to-report-ai-slop/826781/
4. Social Media Today, "LinkedIn says 1M people have reported AI slop" (Aug 20, 2026) https://www.socialmediatoday.com/news/linkedin-says-1m-people-have-reported-ai-slop/828465/
5. Social Media Today, "LinkedIn increases push against inauthentic activity" https://www.socialmediatoday.com/news/linkedin-increases-push-against-inauthentic-activity/829385/
6. Social Media Today, "LinkedIn adds more post performance insights" (Jun 2026) https://www.socialmediatoday.com/news/linkedin-adds-more-post-performance-insights/822193/
7. Social Media Today, "LinkedIn shares video creation tips based on platform trends" (May 2026) https://www.socialmediatoday.com/news/linkedin-shares-video-creation-tips-based-on-platform-trends/821050/
8. Social Media Today, "LinkedIn will no longer allow real-time livestreams" (Mar 29, 2026) https://www.socialmediatoday.com/news/linkedin-will-no-longer-allow-real-time-livestreams/816050/
9. PPC Land, spontaneous Live ending June 22 https://ppc.land/linkedin-kills-spontaneous-live-streaming-from-june-22/
10. Gizmodo, slop button https://gizmodo.com/linkedin-adds-new-seems-like-ai-slop-button-to-report-all-the-ai-slop-2000793107
11. Inc., slop button https://www.inc.com/chris-morris/linkedin-just-added-a-button-to-report-ai-slop-theres-just-1-problem/91383248
12. GCN, slop flag metrics https://gcn.com/linkedin-slop-flag-used-million-members/21090/
13. Global Dating Insights, DSA enforcement report https://www.globaldatinginsights.com/featured/linkedin-steps-up-enforcement-against-inauthentic-activity/
14. The Gain Blog, collaborative posts test (Aug 2026) https://blog.gainapp.com/social-media-updates/
15. SocialPilot, new LinkedIn features 2026 (newsletters, slop report) https://www.socialpilot.co/blog/new-linkedin-features-and-updates
16. HeyOrca, monthly LinkedIn news tracker https://www.heyorca.com/blog/linkedin-social-news
17. Forbes / Oscar Rodriguez, VP Trust, profile-to-content match and reshares (Mar 22, 2026)
18. Svenja Maltzahn / Tim Jurka, LinkedIn March 2026 algorithm update (Mar 15, 2026)

### LinkedIn engineering and research
19. PPC Land, feed rebuild coverage of the Mar 12 engineering blog https://ppc.land/linkedin-rebuilds-its-feed-from-scratch-with-llms-and-gpu-powered-ranking/
20. Net Influencer, LLM-based feed ranking https://www.netinfluencer.com/linkedin-deploys-llm-based-feed-ranking-system-to-surface-content-beyond-members-networks/
21. ByteByteGo, "How LinkedIn Feed Uses LLMs" https://blog.bytebytego.com/p/how-linkedin-feed-uses-llms-to-serve
22. arXiv, "An Industrial-Scale Sequential Recommender for LinkedIn Feed Ranking" https://arxiv.org/pdf/2602.12354
23. Fast Growth Advisors, proven vs invented algorithm claims https://fast-growth.fr/linkedin-algorithm-2026-proven-invented/
24. Publora, 360Brew naming https://publora.com/blog/linkedin-algorithm-2026
25. ALM Corp, LLM feed update https://almcorp.com/blog/linkedin-feed-algorithm-update-llm-2026/

### Independent data
26. Creator Science #307, Richard van der Blom https://podcast.creatorscience.com/richard-van-der-blom-2/
27. Melanie Goodman, van der Blom 2026 and Saywhat link data https://melaniegoodmanlinkedinconsultant.substack.com/p/linkedin-algorithm-2026-reach-topic-authority
28. Vulse, Socialinsider 2026 benchmarks https://vulse.co/blog/how-linkedin-2026-algorithm-works-and-what-it-means-for-your-content-strategy
29. Carousels Generator, Socialinsider and van der Blom carousel data https://carousels-generator.com/blog/linkedin-algorithm-2026-carousels
30. LinkPost, 438,413-post study https://www.linkpost.gg/en/playbooks/linkedin-algorithm-playbook-2026/study
31. Meet Lea, dwell time, AuthoredUp weights, unverified-claim notes https://meet-lea.com/en/blog/linkedin-algorithm-explained
32. Saywhat, State of the Algorithm Q1 2026 https://saywhat.ai/algorithm-webinar/
33. Sarah Evans, Pangram data https://prsarahevans.substack.com/p/linkedin-gave-everyone-a-button-to
34. MagicPost, AI detector scan of 45,965 top posts https://magicpost.in/blog/does-linkedin-penalize-ai-content
35. DowSocial, reach as % of followers https://www.dowsocial.com/linkedin-algorithm-2026/
36. Sales and Marketing Engineers, reply and comment data https://www.salesandmarketingengineers.co.uk/the-ultimate-linkedin-posting-guide-for-2026
37. Dataslayer, formats and March update https://www.dataslayer.ai/blog/linkedin-algorithm-february-2026-whats-working-now
38. ALM Corp, in/out-of-network analytics and carousel sizing https://almcorp.com/news/linkedin-post-performance-insights-in-network-out-network/
39. Vulse, in/out-of-network reach https://vulse.co/blog/linkedin-in-network-vs-out-of-network-reach-what-the-new-metric-means-and-how-to-use-it

### Practitioner
40. ConnectSafely, post length https://connectsafely.ai/articles/ideal-linkedin-post-length-engagement-guide-2026
41. Final Layer, post length https://finallayer.com/blog/ideal-linkedin-post-length
42. ViralBrain, hooks and updates https://www.viralbrain.ai/blog/linkedin-algorithm-2026-what-changed
43. SocialPilot, algorithm August 2026 https://www.socialpilot.co/blog/linkedin-algorithm
44. SocialBee, algorithm guide https://socialbee.com/blog/linkedin-algorithm/
45. Sourcegeek, ranking stages https://www.sourcegeek.com/en/news/how-the-linkedin-algorithm-works-2026-update
46. Stackmatix, commenting and topic consistency https://www.stackmatix.com/blog/linkedin-algorithm-how-it-works
47. Hootsuite, LinkedIn algorithm 2026 https://blog.hootsuite.com/linkedin-algorithm/
48. SMARTe, algorithm changes https://www.smarte.pro/blog/linkedin-algorithm-changes
49. ClipoAI, video clipping from recordings https://clipo.pro/blog/linkedin-video-strategy
50. Wheels Up Collective, slop reporting https://www.wheelsupcollective.com/post/linkedin-ai-slop-reporting-feature
51. SocialNexis, originality scoring https://socialnexis.com/guides/linkedin-ai-originality-reach-penalty
52. ZoomSphere, generic AI content and reach https://www.zoomsphere.com/blog/linkedin-algorithm-2026-why-generic-ai-content-kills-your-organic-reach

### Carried forward from v2026.3
Kanbox, TryOrdinal, Clicknara, TechCrunch (Dec 2025), River, Speedwork Social, Agorapulse, GrowLeads, MeetEdgar, Exxar Digital, Vertebrae Social, Chad Wyatt, Closely, Growth Terminal, Adobe Express, AuthoredUp (621K posts), Buffer (2M posts)

---

Last updated: September 21, 2026. Version 2026.5 (changes and general strategy edition).
