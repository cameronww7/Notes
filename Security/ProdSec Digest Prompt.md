Build today's Product Security Vendor Digest for Cameron Walters (Director of AppSec and Security Engineering). Your final message is the digest itself: Cameron reads it by tapping the push notification for this run. Complete every step below. Do not stop early, do not skip or sample sources, and do not narrate your steps. If an extra message after this prompt sets a different lookback window, use that window instead of 26 hours.

STEP 0: LOAD THE LEDGER
The seen-topics ledger is a Claude Doc with container id 5b84c671-bf61-477f-959e-7e541ca62a0d (https://claude.ai/code/artifact/5b84c671-bf61-477f-959e-7e541ca62a0d). Call the Claude_Docs guide tool with items ["topic.index"] and ["topic.editing"], then read the ledger. Under "Seen topics" each line is: YYYY-MM-DD | topic key | primary URL. Keep this list for Step 3. If the ledger cannot be read, continue without it and add "⚠️ Ledger unavailable, repeats possible" to the footer.

STEP 1: COLLECT (all 14 sources, one at a time)
Make a separate WebFetch call for EACH of the 14 sources below, every run, in order. In each call ask for every post on the page with its exact title, full URL, and publish date. Prefer the RSS/Atom feed when listed; if the feed fails, fetch the index. If an index shows no dates, open the newest 3 posts individually to get their publish dates. Keep posts published within the last 26 hours (relative to now, America/Chicago).
Keep a coverage tally as you go: source name → number of posts in the window, or ✗ if both feed and index failed. You must have a tally entry for all 14 sources before moving to Step 2. Do not invent posts or dates.

Sources:
1. Clutch Security: https://www.clutch.security/blog
2. DryRun Security: https://www.dryrun.security/blog
3. depthfirst: https://depthfirst.com/blog
4. Wiz: https://www.wiz.io/feed/rss.xml (index https://www.wiz.io/blog)
5. XBOW: https://xbow.com/blog/feed.xml (index https://xbow.com/blog)
6. Snyk: https://snyk.io/blog/
7. OpenSourceMalware: https://opensourcemalware.com/blog
8. Socket: https://socket.dev/api/blog/feed.atom (index https://socket.dev/blog)
9. Aikido: https://www.aikido.dev/blog/rss.xml (index https://www.aikido.dev/blog)
10. Semgrep: https://semgrep.dev/blog/rss (index https://semgrep.dev/blog)
11. Endor Labs: https://www.endorlabs.com/learn/rss.xml
12. StepSecurity: https://www.stepsecurity.io/blog/rss.xml (index https://www.stepsecurity.io/blog)
13. Datadog Security Labs: https://securitylabs.datadoghq.com/articles/
14. Trail of Bits: https://blog.trailofbits.com/
15. ZeroPath: https://zeropath.com/blog/research
16. Corgea: https://corgea.com/blog (unverified)
17. Ghost Security: https://ghostsecurity.com/blog
18. Backslash Security: https://www.backslash.security/blog
19. Boost Security: https://boostsecurity.io/blog
20. Chainguard: https://www.chainguard.dev/unchained
21. Echo: https://www.echo.ai/blog
22. JFrog Security Research: https://research.jfrog.com/
23. ReversingLabs: https://www.reversinglabs.com/blog
24. Sonatype: https://www.sonatype.com/blog
25. Kusari: https://www.kusari.dev/blog
26. Seal Security: https://www.seal.security/blog
27. OpenSSF: https://openssf.org/blog/
28. GitGuardian: https://blog.gitguardian.com/
29. Truffle Security: https://trufflesecurity.com/blog
30. OX Security: https://www.ox.security/blog/
31. Legit Security: https://www.legitsecurity.com/blog
32. Apiiro: https://apiiro.com/blog/
33. Cycode: https://cycode.com/blog/
34. Arnica: https://www.arnica.io/blog
35. Oligo Security: https://www.oligo.security/blog
36. Miggo: https://www.miggo.io/blog (unverified)
37. Embrace The Red: https://embracethered.com/blog/
38. Zenity Labs: https://labs.zenity.io/
39. Pillar Security: https://www.pillar.security/blog
40. Straiker: https://www.straiker.ai/blog
41. GitHub Security Lab: https://securitylab.github.com/research/
42. Google Project Zero: https://projectzero.google/
43. watchTowr Labs: https://labs.watchtowr.com/
44. Doyensec: https://blog.doyensec.com/
45. NCC Group: https://www.nccgroup.com/research-blog/
46. Phrack: https://phrack.org/issues/73/introduction_md (index https://phrack.org/)
47. Phoenix Security: https://phoenix.security/cyber-risk-club-resources/blogs/
48. Anthropic Frontier Red Team: https://red.anthropic.com
49. Anthropic Engineering: https://www.anthropic.com/engineering
50. OpenAI Security: https://openai.com/news/security/
51. Cursor: https://cursor.com/blog (filter for security)
52. Google Security Blog: https://security.googleblog.com/
53. Google DeepMind: https://deepmind.google/blog/ (filter for security)
54. GitHub Blog Security: https://github.blog/security/

STEP 2: FILTER
Keep posts about: AppSec, software supply chain attacks, malicious packages, CI/CD and pipeline security, GitHub Actions, secrets and non-human identity, cloud security research, vulnerability research and CVEs, AI/agentic security, AI-assisted code review and pentesting, and substantive product capability launches with technical detail.
Drop: compliance certifications (SOC 2, FedRAMP), funding, hiring, culture posts, webinars, event recaps, customer stories, and pure marketing.

STEP 3: DEDUPLICATE
a) Within today: group posts that cover the same event, campaign, CVE, or announcement (for example, several vendors writing up the same npm compromise). Each topic appears once. Pick the single most technical, original-research write-up as the primary article and list the others on the "Also covered by" line.
b) Against the ledger: if a topic matches a ledger entry (same campaign, CVE, package compromise, or announcement, even under a different title), drop it, unless today's post adds materially new facts (a new wave, new affected packages or repos, a new CVE, a patch, attribution). In that case keep it as a short update: prefix the title with "🔄 Update:", and fill only The Hook, The Specifics (new facts only), What You Can Use, and the link lines. Skip The Case.

STEP 4: READ
Open (WebFetch) the full text of every primary article you are keeping, one call per article. Every number, CVE ID, package name, version, person, and link in the digest must come from the article itself. If a detail is not in the article, leave it out. Never fill gaps from memory.

STEP 5: WRITE THE DIGEST
Start with this header inside a code block so it renders monospaced, filling in the bracketed values:

```
 ___ ___  ___  ___  ___ ___ ___   ___ ___ ___ ___ ___ _____ 
| _ \ _ \/ _ \|   \/ __| __/ __| |   \_ _/ __| __/ __|_   _|
|  _/   / (_) | |) \__ \ _| (__  | |) | | (_ | _|\__ \ | |  
|_| |_|_\\___/|___/|___/___\___| |___/___\___|___|___/ |_|  
============================================================
  ☕ [Weekday, Month Day, Year]  ·  [N] topics  ·  [M] sources
============================================================
```

Order topics by impact to an AppSec or pipeline security program: active supply chain attacks and exploited vulns first, then research, then product launches. Put a line of 40 "─" characters between topics.

Each topic uses exactly this structure:

[one emoji matching the topic, e.g. 🐧 kernel, 📦 packages, ⚙️ CI/CD, 🔑 secrets/NHI, ☁️ cloud, 🤖 AI] [Article title]

✍️ [Author name(s)], [Vendor] · [Article | Research | Threat Intel | Advisory | Product Launch]   (vendor only if no author is listed)

💥 The Hook: 1 to 2 sentences that lead with the most striking concrete fact or number and say what the vendor found or shipped.

🧩 The Case: 2 to 3 sentences on the core argument or technical mechanism, and the broader claim the piece makes. If the piece frames or promotes the vendor's own product, add one sentence saying so plainly (e.g. "[Vendor] is a vendor and this frames their product, read accordingly.").

🧾 The Specifics:
- 3 to 5 bullets of hard facts: CVE IDs, package names and versions, counts, dates, affected platforms, who else found or reported it, linked repos or PoCs

📥 What You Can Use: 1 to 2 sentences naming the concrete action, control, or evaluation an AppSec, CI/CD, or ProdSec team should take, and where it applies.

🏷️ [2 to 4 comma-separated tags]
🔗 [Article URL]
🔁 Also covered by: [Vendor] ([URL]), [Vendor] ([URL])   (omit this line if none)

Example of one finished topic (for format only; do not reuse its content):

🐧 Containers Are Not a Security Boundary Anymore

✍️ Zhenpeng (Leo) Lin, depthfirst · Article

💥 The Hook: 5,976 Linux kernel CVEs in the first nine months of 2026, and AI is both finding them and generating working exploits for them. depthfirst used their own AI model to discover a kernel use-after-free in AF_UNIX, turned it into a working container escape, and published the code.

🧩 The Case: The piece walks through CVE-2026-80521, a race condition in the AF_UNIX garbage collector, then connects it to a larger claim: AI has reduced kernel exploit development from a nation-state-tier skill to a commodity task. Thirteen of the 36 CVEs disclosed through kernelCTF are reachable from unprivileged container interfaces. depthfirst is a vendor and this frames their product, read accordingly.

🧾 The Specifics:
- August 2026 alone produced 1,650 kernel CVEs, 27% of the year's total in a single month
- CVE-2026-80521 is still unpatched in Ubuntu 26.04 as of publication
- OpenAI's Kyle independently found the same bug; both reported it the same day
- 13 of 36 kernelCTF CVEs are reachable from default, unprivileged container syscall interfaces
- Working exploit published at github.com/Markakd/Container_escape

📥 What You Can Use: Any multi-tenant workload or untrusted code execution environment running on a shared kernel should be evaluated against microVM isolation (Firecracker, Kata Containers) rather than container namespaces. For AI eval sandboxes specifically, this is directly relevant to recent agentic escape incidents.

🏷️ Container Security, Kernel Exploitation, AI Security
🔗 https://depthfirst.com/research/containers-are-no-longer-safe

After the last topic, add a footer:
────────────────────────────────────────
🧮 Checked: [all 14 sources with their tally, e.g. Clutch 0 · DryRun 1 · depthfirst 0 · Wiz 2 · ... · Trail of Bits ✗]
♻️ Skipped as already covered: [topic keys dropped by the ledger check]   (omit if none)
🗑️ Filtered out: [count] posts (marketing, compliance, hiring, events)   (omit if 0)

If there are zero qualifying topics, output the header, one line saying no new product security posts in the window, and the footer.

Style rules: direct and specific. No em dashes anywhere (use commas, periods, or colons). No bold or markdown emphasis. No preamble or closing summary. Never use the words "signal," "highlights," "underscores," "worth noting," "game-changing," "robust," "leverage," or "seamless." Paraphrase vendor content in your own words; no direct quote longer than 15 words and at most one quote per article.

STEP 6: UPDATE THE LEDGER
Edit the ledger doc in one update: under "Seen topics", add one line per topic included today (including updates), formatted YYYY-MM-DD | short topic key (campaign, CVE, package, or announcement name) | primary URL. Delete every line dated more than 7 days before today, and delete the "ledger created" line. Change nothing else in the doc.

STEP 7: FINISH
End the run with the full digest from Step 5 as your final message and nothing else.