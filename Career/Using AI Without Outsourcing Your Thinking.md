# Using AI Without Outsourcing Your Thinking

AI is a great tool once you know what you're doing, but if you lean on it while you're still learning, you skip the part that actually makes you good at this job. Learning is supposed to be frustrating, you don't get it, you feel dumb, you try five things that don't work, you bang your head on the wall for a few days and then it finally clicks. That stretch is where the learning happens. If AI hands you the answer every time, you walk away with the answer and none of the understanding.

## Why the struggle matters

- You build understanding by pulling ideas out of your own head, connecting them, and testing them. Reading a correct answer feels like learning, but most of it is gone by next week.
- The aha moment sticks because of the hours you spent stuck before it. You remember the problem, the dead ends, and why the fix works.
- Feeling insecure and stuck is normal. You're at the edge of what you know, and that's exactly where you grow.
- If you never sit in that discomfort, you never build tolerance for it, and a production incident at 2am doesn't come with a chat window that already knows the answer.

## What senior engineers actually have

Senior engineers aren't faster because they memorized more. They carry years of context, how systems fail, what broke last time, which teams depend on what, and why the obvious fix is usually wrong.

New engineers get into trouble when they make decisions in isolation. The fix looks right for the problem in front of them, but they don't understand the system around it, so a senior ends up cleaning it up.

| Situation | Isolated decision (AI-assisted) | Decision with context |
|---|---|---|
| SCA flags a critical CVE | Bump the package to the fixed version | Checks reachability, notices the bump is a major version that changes auth middleware behavior, plans the upgrade with the owning team |
| Leaked secret in a repo | Rotate the key immediately | Identifies every service consuming the key first so rotation doesn't take down production |
| SAST finding looks noisy | Suppress it | Traces the data flow, confirms the input gets sanitized upstream, documents why |
| SSRF fix | Add a URL blocklist | Knows attackers bypass blocklists with redirects, DNS rebinding, and alternate IP encodings, so uses an allowlist and network controls |

The AI answer in the left column is sometimes fine. The problem is you can't tell when it isn't, and that's the gap your senior ends up fixing for you.

## Curiosity is the engine

A lot of the context seniors carry didn't come from tickets, it came from curiosity. They wonder how something works, dig in, and keep going after the ticket is closed, and after enough years that habit turns into the deep knowledge that shows up when a nasty problem lands.

Most seniors don't follow a study plan for this. They just operate this way:

- A dependency acts strange, so they read its source instead of just pinning a version.
- A new CVE drops, so they pull up the patch diff to see what actually changed.
- An auth flow works and they want to know why, so they trace the redirects and tokens end to end.
- They hit a weird error and stay on it after the workaround, because now they want the root cause.
- They read postmortems, RFCs, and changelogs for systems they don't even own.

This kind of passive learning compounds. Every rabbit hole adds a little more context, and eventually that context turns into the instinct people call experience.

AI can kill this habit fast. When every question gets an instant answer, the itch that makes you explore gets scratched before you learn anything, so use AI to dig deeper:

- When AI answers something, figure out what you still don't understand and go find it in the primary source.
- Follow the why one level further than the task needs.
- When something looks weird, dig into it.

Curiosity is a habit and you build it with reps. Force it at first, pick one thing a week you use every day without understanding it and figure out how it actually works. Keep that up long enough and you stop noticing the effort, it becomes how you work, and it's what separates the engineers who keep growing from the ones who plateau.

## How to tell you've offloaded your thinking

Be honest with yourself on these:

- You can't explain the solution once you close the chat window.
- You paste the error into AI before you read it yourself.
- You take the first answer and can't name an alternative or why it lost.
- Your notes are copied AI output instead of your own words.
- When AI is wrong you don't catch it, someone in code review does.
- You can't predict what a change will do before you run it.
- Your first question is "what should I do?" instead of "here's what I think, what am I missing?"
- Being stuck for 10 minutes feels unbearable.
- Someone asks "why did you do it this way?" and the honest answer is "the AI said so."

If a few of these hit home, AI is doing your thinking for you.

## Attempt first, then ask

For anything you're trying to learn:

1. Write the problem down in your own words. If you can't, you don't understand the problem yet.
2. Go to primary sources, official docs, RFCs, the vendor advisory, the source code, logs, the stack trace.
3. Write down what you think is happening and what you'd do about it. Have an opinion before you open a chat window.
4. Time-box the struggle. 30 to 60 minutes of real effort on a learning problem before you ask for help is reasonable, and real effort means you tried specific things and can say why they failed.
5. Bring in AI as a reviewer. Hand it your understanding and ask it to poke holes in it.
6. Check what it tells you against the primary source. AI is confidently wrong often enough that trusting it blind is a habit ProdSec can't afford.
7. Explain it back without AI. Write it up, teach it to someone, or whiteboard it, and if you can't, go back to step 2.

## Asking well vs asking lazy

| Lazy | Useful |
|---|---|
| "How do I fix this SSRF?" | "User-controlled URL reaches `requests.get` with no validation. I'm planning a hostname allowlist. What bypasses am I missing?" |
| "Explain OAuth" | "I think the auth code flow exists so tokens never pass through the browser. Is that right, and where does PKCE fit?" |
| "Is this vulnerable?" | "I traced this input to a SQL query built with string formatting. I think it's injectable because X. Am I wrong about the sanitization in Y?" |
| "Write me a Semgrep rule for this" | "Here's my rule. It misses case Z. Why?" |

The useful version makes you do the thinking first, and you get better answers out of it too.

## When offloading is fine

Not every task is a learning task, so here's where I draw the line:

- Offload what you already understand, boilerplate, syntax you've written a hundred times, reformatting, summarizing a doc you'd skim anyway.
- Keep what you're trying to learn for yourself. If the goal is understanding, AI doing the work defeats the purpose.
- Never ship what you can't explain. If it goes into production, a ticket, or a security decision, you own it, and "the AI wrote it" won't hold up in a postmortem.

## Your notes won't save you

Notes point you to things you understand, they can't do the understanding for you. Quick test:

- Can you explain the concept without looking anything up?
- Can you describe how it breaks or gets attacked?
- Can you apply it to a variant you haven't seen?
- Can you explain why the alternatives are worse?

If you can't say yes to all four, you have a reference and you still have some learning to do.

## Habits that build real understanding

- Predict before you run. Write down what you expect a command, exploit, or fix to do, then check, because being wrong is where you learn.
- Rebuild things to learn them. Write your own JWT validation, a toy SAST rule engine, or a minimal OAuth client, purely for the learning and never for prod.
- Break things in a lab. Exploit the vulnerable app before you read the writeup.
- Do CTFs and labs without AI hints, and only touch the hint system after you've honestly stalled.
- Read postmortems and incident writeups. It's borrowed experience and the closest shortcut to context I know of.
- Keep a struggle log of what you got stuck on, what you tried, and what finally worked. Reread it, it turns into your pattern library.
- Close the chat and rewrite. After AI explains something, close it and write the explanation from memory, and the gaps show you what you didn't actually learn.

AI makes experienced engineers faster because they can judge what it hands them, and it makes new engineers look productive while they stay new. So do the hard reps now and stay curious. I'd much rather you bash your head into a wall in a lab than into my production environment.
