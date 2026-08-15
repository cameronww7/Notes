# Application Security/Product Security/DevSecOps Learning Path

## Why Read This?
- Caveat: This is my opinion, and like opinions, everyone has one. What I've outlined could 100% conflict with someone else's. Keep that in mind, the path is yours and it's unique, this is a guide, nothing is set in stone. I just hope it helps you on your path to success.

So you want to break into AppSec, DevSecOps, or Product Security. This isn't a generic list of certifications to collect or courses to finish. It's a practical roadmap built from real experience.

I was a developer first. I made the transition into security and now run a global AppSec and Product Security team. This is what I hire for, what I mentor people toward, and what I expect from the engineers on my team.

Whether you're coming from software engineering, IT, sysadmin, or starting fresh, this path is designed to get you job-ready with skills that translate directly into the work -- not just the interview.

## How AI Is Changing Everything

We use to be required to code. We use to focus on implementing tools, picking a scanner, tuning it, wiring it into the pipeline, calling it a program. That world is closing out. Now the Build vs Buy decision is getting harder every quarter, not easier, because buying a vendor tool means betting that the vendor's roadmap keeps up with a landscape that's rewriting itself every few months, and half the time it doesn't, so you end up building anyway just to fill the gap they left.

From the ProdSec side we are shifting up the stack. We can now run multiple agents at once, one building automation for a specific workflow, one analyzing code for vulns across a whole repo, one reviewing a design doc from product engineering before a line of code gets written, and none of that required a human staring at a screen doing it manually. A few examples of what that actually looks like in practice:

- An agent that pulls every new PR, runs a vuln scan, triages the findings against known false-positive patterns from your codebase, and only surfaces the ones that actually need a human
- An agent that reads a design doc dropped in a shared drive, flags missing threat model sections, and drafts the questions a security architect would normally ask in review
- An agent orchestrating three other agents: one doing recon on a new service, one checking it against your SPVS-style pipeline controls, one writing the remediation ticket with context already filled in

That's the shift. It's not "AI helps you scan faster." It's AI doing the work that used to define the job.

Here's what doesn't change though, and this is the part people get wrong when they think AI means the fundamentals stop mattering: the core skills we've needed for the last 15 years are still highly relevant, arguably more relevant, because someone has to validate what the AI produces, and you can't validate what you don't understand. An agent tells you a finding is a false positive, you need to know enough about how that class of vuln actually works to know if the agent is right or if it's pattern matching its way to a wrong answer. An agent writes a remediation, you need the foundational knowledge to know if that remediation actually closes the gap or just makes the finding disappear from the dashboard. Knowing this core, foundational information is what lets you trust or distrust the output in front of you, instead of just rubber stamping whatever the agent hands back because it sounds confident.

So as ProdSec folk we have to rethink our role, and rethink it fast, because the org chart hasn't caught up to what the work actually looks like now. We have to shift up the stack. We have to understand orchestration, how to build and run fleets of agents, how to chain them, how to know when one agent's output should trigger another agent instead of a ticket. We have to start thinking and learning architecture, real architecture, because when everything downstream is agentic, the thing that actually matters is whether you designed the system correctly upstream. Get the architecture wrong and no amount of agent output fixes it.

The highly technical hands-on skills, pen testing, POC'ing a single vuln by hand, walking through an exploit chain manually, that's no longer the differentiator it used to be as a day-to-day task. It can be shifted off to an agent that does it faster and more consistently than a person grinding through it at 6pm before a release. But you still need to know how to do it yourself, because that's the only way you know when the agent did it wrong. The skill doesn't disappear, it moves from something you execute every day to something you hold as ground truth so you can check the machine's work.

This is exactly why it's going to be hard for new folks coming into ProdSec right now. You need the core and the basics, how injection actually works, how auth actually breaks, how a pipeline actually gets compromised, the same foundation people have needed for 15 years. But now you also have to layer AI fluency on top of that, orchestration, prompt design, knowing where models fail, and you have to build both at once instead of getting years to master the fundamentals before AI became part of the job. It's not AI instead of the basics. It's AI on top of the basics, and if the basics aren't there, the AI layer has nothing solid to stand on.

A few other threads worth pulling into this section as you build it out:

- **Vendor tooling is becoming a moving target.** A tool you evaluated six months ago may already be behind what an internally built agent can do, because the vendor's release cycle can't match the pace of model improvement. That changes how Build vs Buy evaluations should even be scored.
- **The job is becoming orchestration and validation, not execution.** You're not the one running the scan, you're the one deciding what the scan should be, what "good" looks like, and catching it when an agent's output is confidently wrong, which brings it right back to needing the fundamentals.
- **New skill floor for ProdSec: prompt design, agent chaining, and knowing model failure modes.** Just like the old floor was knowing how a SAST tool produces false positives, the new floor is knowing how an LLM hallucinates a vuln that isn't there or misses one that is, on top of the old floor, not instead of it.
- **Headcount math is changing.** One ProdSec engineer running four agents can cover ground that used to take a team of four. That's good for scale, it's uncomfortable for career paths that assumed you'd manage humans to grow.
---
---
# Application Security/Product Security/DevSecOps Learning Path - Starts

## How to Learn Effectively

Reading or listening is not enough. You need to actively engage with what you're learning. As you go through any material:

**Ask critical questions:**

- Why is this important? Why does it matter?
- What is the small picture (the specific detail or technique)?
- What is the bigger picture (how does this fit into the overall security landscape)?
- How would an attacker exploit this? What could go wrong?

**Apply actively:**

- Take notes in your own words, not verbatim
- Immediately test concepts in a lab or personal project
- Explain what you learned to someone else (or write a blog post)
- Connect new concepts to real-world breaches or vulnerabilities you've heard about
- Review your notes within 24 hours, then again after a week

Learning security is about building mental models, not memorizing facts. The goal is to develop intuition for how systems break and how to build them securely.

- Active Learning: 5 Reasons You're Doing Active Learning WRONG: https://www.youtube.com/watch?v=0A5Ji-QdFvg

Here's the list with names and context added:

## People to Follow on LinkedIn to Stay Up To Date
- **Tanya Janca** -- Founder of We Hack Purple, author of "Alice and Bob Learn Application Security", one of the most active AppSec educators online
  https://www.linkedin.com/in/tanya-janca/
- **Jim Manico** -- OWASP contributor, AppSec educator, frequent conference speaker on secure coding and developer security training
  https://www.linkedin.com/in/jmanico/
- **Ken Johnson** (cktricky) -- Co-founder & CTO of DryRun Security, co-host of the Absolute AppSec podcast, 18+ years breaking and building web apps
  https://www.linkedin.com/in/cktricky/
- **Cameron Walters** -- Director of Application Security & Security Engineering, co-founder of the OWASP Secure Pipeline Verification Standard (SPVS), co-host of Coffee, Chaos & ProdSec
  https://www.linkedin.com/in/cameronww7/
- **Clint Gibler** -- Founder of tl;dr sec, one of the best curated security newsletters available, strong signal-to-noise ratio on AppSec trends
  https://www.linkedin.com/in/clintgibler/
- **Chris Hughes** -- Resilient Cyber, author and analyst focused on software supply chain security, vulnerability management, and cyber risk
  https://www.linkedin.com/in/resilientcyber/
- **Daniel Miessler** -- Security researcher and writer, runs the Unsupervised Learning newsletter, strong voice on AI and security convergence
  https://www.linkedin.com/in/danielmiessler/
- **Matt Johansen** -- Practitioner-focused security voice, runs the Thoughtful Security newsletter, writes about real-world AppSec and vulnerability management
  https://www.linkedin.com/in/matthewjohansen/
- **Anshuman Bhartiya** -- Staff Security Engineer at Lyft, co-host of The Boring AppSec Podcast, strong practitioner voice on AI and AppSec
  https://www.linkedin.com/in/anshumanbhartiya/
- **Derek Fisher** -- Author of "The Product Security Handbook", AppSec and ProdSec leader focused on building security programs at scale
  https://www.linkedin.com/in/derek-fisher-sec-arch/
- **Adam Shostack** -- Threat modeling expert, author of "Threat Modeling: Designing for Security", one of the most referenced voices in the field
  https://www.linkedin.com/in/shostack/
- **Patrick Garrity** -- Security researcher focused on vulnerability data and exploitation trends, consistently useful signal on what's actually getting exploited
  https://www.linkedin.com/in/patrickmgarrity/
- **Katie Moussouris** -- Founder of Luta Security, pioneered bug bounty and vulnerability disclosure policy, key voice on responsible disclosure and CVD
  https://www.linkedin.com/in/kmoussouris/
- **Dan Lorenc** -- Co-founder & CEO of Chainguard, software supply chain security and open source security, deep expertise in SLSA and sigstore
  https://www.linkedin.com/in/danlorenc/
- **Caleb Sima** -- AI security leader and advisor, former CSO, one of the more credible voices on AI security in production environments
  https://www.linkedin.com/in/calebsima/
- **Codrut Andrei** -- Director of Product Security, AppSec and Secure SDLC practitioner, active mentor helping people break into the field
  https://www.linkedin.com/in/codrut-andrei/

## Local Networking
- Personal Take: Essentially every job I have ever gotten was from networking, meeting folks building relationships and trust. In a world of AI, I personally have hired over 30+ folks over the years, and in the last year hiring is awful from both sides. I open a job req and in minutes have 50 AI resumes or applications... Network Network Network.

Getting into security isn't just about learning technical skills. It's also about building relationships with other practitioners in your area. Look up local cybersecurity meetups on Meetup.com, search for security events on Eventbrite, and find conferences near you. This will take some Googling, but search for things like "OWASP chapter [your city]", "cybersecurity meetups [your city]", and "security conferences [your state/region]". These events are invaluable for networking, learning what skills are in demand locally, and often hearing about job opportunities before they're posted. Show up, introduce yourself, and don't be afraid to tell people you're looking to break into the field. The security community is generally welcoming to newcomers who show genuine interest.



## Podcasts to Tune into
- Coffee, Chaos and ProdSec: https://linktr.ee/coffeechaosprodsec
- Absolute AppSec: https://absoluteappsec.com/
- The OpenSourceMalware Show: https://podcast.opensourcemalware.com/
- Boring AppSec Podcast: https://www.boringappsec.com/s/podcast
- Application Security Weekly: https://www.scworld.com/podcast-show/application-security-weekly
- The Application Security Podcast: https://appsec.buzzsprout.com/
- The Security Champions Podcast: https://www.securityjourney.com/resources/security-champions-podcast
- What's in the SOSS? An OpenSSF Podcast: https://openssf.org/podcast/

## Newsletters & Blogs
Stay current with security news and practical advice:

- TL;DR sec (Newsletter & Blog): https://tldrsec.com/subscribe
- Boring AppSec (Newsletter & Blog): https://boringappsec.substack.com/subscribe
- Resilient Cyber: https://www.resilientcyber.io/
- Cybersecurity Pulse: https://www.cybersecuritypulse.net/

## YouTube Channels & Video Content
- OWASP Global: Watch recordings from OWASP AppSec conferences and expand your knowledge on application security.
  - https://www.youtube.com/@OWASPGLOBAL
- LASCON: Real-world AppSec conference talks and presentations from practitioners.
  - https://youtube.com/@lascon512
- BSidesSF: Security BSides San Francisco is a two-day information security conference. It is a conference by the community for the community.
  - https://www.youtube.com/@BSidesSF
- DEFCON Conference: DEF CON is one of the oldest continuously running hacker conventions around, and also one of the largest.
  - https://www.youtube.com/@DEFCONConference
- Black Hat: Black Hat Briefings and Trainings are driven by the needs of the global security community, striving to bring together the best minds in the industry.
  - https://www.youtube.com/@BlackHatOfficialYT/featured
- OWASP DevSlop (Tanya Janca): Practical AppSec education, secure coding, and developer-focused security content.
  - https://youtube.com/@DevSlop
- DevSecOps Course - Web Vulnerabilities, DevSecOps Tools, and Container Security.
  - https://www.youtube.com/watch?v=F5KJVuii0Yw
- IBM Technology: Whether it’s AI, automation, cybersecurity, data science, DevOps, quantum computing or anything in between, we provide educational content on the biggest topics in tech.
  - https://www.youtube.com/@IBMTechnology
- unprompted: The AI security practitioners' conference
  - https://www.youtube.com/@un_prompted
- NDC Conferences: NDC Conferences aims to find the greatest minds and leaders in the field of software development, striving to make the best and most updated knowledge available to developers everywhere. 
  - https://www.youtube.com/@NDC
- Bsides Seattle: Bsides Seattle is a community information security conference in Seattle Washington.
  - https://www.youtube.com/@bsidesseattle3784


## Interesting YouTube content creators with security education and training content
- vulnerableu: Cybersecurity veteran and ex-CISO Matt Johansen shares front-line stories, proven frameworks, and global security news to help you build a stronger, more resilient security practice.
  - https://youtube.com/@vulnerableu
- LowLevelTV: Videos about cyber security + software security
  - https://www.youtube.com/@LowLevelTV
- Unsupervised Learning: Building AI that upgrades humans and companies.
  - https://www.youtube.com/@unsupervised-learning

## Paid Courses
- Application Security - The Complete Guide
  - https://www.udemy.com/course/application-security-the-complete-guide/

## Foundational Knowledge

### Start Here, Core AppSec Concepts

- Read "Alice and Bob Learn Application Security" (As you complete chapters, watch Tanya's chapter summaries at the following YouTube playlist: https://youtube.com/playlist?list=PLI9RITMnVbygrVQaGvpojIzgHTpkRrIn8&si=xrUXxdTsikHFA0vF) and "Alice and Bob Learn Secure Coding"
- Watch YouTube videos from Jim Manico (particularly "The History of AppSec") and Tanya Janca (any of her talks on YouTube)

#### Essential Video Content

**Jim Manico Talks:**

- Keynote: Jim Manico - The Abridged History of Application Security: https://www.youtube.com/watch?v=QbEjNqKRhCg
- Jim Manico - Enhancing ReactJS Application Security: XSS Scripting and Client-Side Defense: https://www.youtube.com/watch?v=gOODVasKdsk&t=6s
- From the OWASP Top Ten(s) to the OWASP ASVS - Jim Manico: https://www.youtube.com/watch?v=CCIO3GOaFe0
- Jim Manico - Securing AI-Generated Code in the Development Lifecycle (React Focus): https://youtu.be/O2hUJ3LdulE?si=KOssISXZHosoh6iW

**Tanya Janca Talks:**

- Lightning Talk: Purple is the new black: Modern Approaches to Application Security - Tanya Janca: https://www.youtube.com/watch?v=iy1xcmsIx14
- Global AppSec Dublin: Shifting Security Everywhere - Tanya Janca: https://www.youtube.com/watch?v=ei3Subi59Kg

**Additional Essential Talks:**

- OWASP Top 10 2025 Announcement - Updated view of modern applications' most common security risks: https://youtu.be/yGOXewm3DsA?si=FPFDLho0-q_ihFUG
- Keynote by Daniel Miessler: The Future of AppSec Is Continuous Context: https://youtu.be/C4L5hPYl1Os?si=LYABTnE5tbCsDq08
- Keynote by Adam Shostack: Stop Trying to "Manage Risk": https://youtu.be/THSfJIlPGPk?si=yg-ufzX_Q8vBhylC
- James Wickett - Out of Control: Promise Theory and the Future of Security Agents: https://www.youtube.com/watch?v=0TyqE5ARJw4
- Dustin Lehr - The Cybersecurity-Minded Future, And How Security Champions Get Us There: https://www.youtube.com/watch?v=aUwe0KsUo7I&t=2350s
- Mauve Hed & Francesco Cipollone - Navigating Challenges of Risk-Based Vulnerability Management in a Cloud World: https://www.youtube.com/watch?v=bDtJA551vpI
- Jenn Gile - How to sell your soul, err, your security program: https://www.youtube.com/watch?v=SM36aCbxj6c&t
- Rafal Los - Why Your Security Metrics Suck, And How to Fix This: https://www.youtube.com/watch?v=aLibY2Azneo
- Chris Hughes - Securing AI Where it Acts: Why Agents Now Define AI Risk | AI Summit Q1 2026 - https://www.youtube.com/watch?v=A6eIooEm7Cs

### Learn the Standards, OWASP Frameworks

These aren't just reading material. They're the standards you'll reference daily in this field.

- OWASP Top 10 2025 (Most Critical Web Application Security Risks): https://owasp.org/Top10/2025/
- OWASP Top 10 for LLMs 2025: https://owasp.org/www-project-top-10-for-large-language-model-applications/assets/PDF/OWASP-Top-10-for-LLMs-v2025.pdf
- OWASP API Security Top 10: https://owasp.org/API-Security/
- OWASP Top 10 CI/CD Security Risks: https://owasp.org/www-project-top-10-ci-cd-security-risks/
- OWASP ASVS (Application Security Verification Standard): https://owasp.org/www-project-application-security-verification-standard/
- OWASP SPVS (Secure Pipeline Verification Standard): https://owasp.org/www-project-spvs/

### Understand the Cloud
- Take AWS Certified Cloud Practitioner (CCP) or general cloud understanding courses. 
    - Free course: https://youtu.be/NhDYbskXRgc?si=pHmsypJT3WfrzCxt
- Or for Azure (Microsoft Certified: Azure Fundamentals (Exam AZ-900))
    - Free course: https://www.youtube.com/watch?v=5abffC-K40c
- Follow up with cloud security-specific training (Youtube, Blogs, just search Cloud Security)
- Modern AppSec lives in the cloud. You need to understand how cloud environments work. 

## Learn to Code (Yes, Really) (Or Maybe not...)

- Caveat: I wanted to add a caveat. In the last 9 months things changed with AI. In 2025, AI was useful, but I'd say it was way more debatable. In 2026, AI, especially around coding, has really exploded in a positive way. AppSec automation is where you'll end up coding, and with AI, that skill could be lifted and shifted to AI. Now that doesn't mean you shouldn't know how to code, but the skill is being devalued in favor of AI coding and AI usability skills.

You can't secure what you don't understand. There are lots of free courses on YouTube from different channels such as FreeCodeCamp, and Udemy offers cheap courses if you want more structured learning. Learning how to code, understand code, and recognize what languages are used in different parts of software is absolutely key. Take full stack programming courses and learn Python or Go for automation/API-to-API scripts. You're going to talk to developers and work with developers every single day. You need to speak their language or you won't be effective.
- You will need to understand how code works -- not just syntax, but control flow, how data moves through a function, how inputs become outputs, and where things can go wrong. You're looking for vulnerabilities in code every day; you can't spot them if you can't read the code.
- Understand how 3rd party libraries (OSS) are packaged and pulled into code -- how package managers like npm, pip, Maven, and Go modules work, what a dependency tree looks like, and why transitive dependencies matter. Most vulnerabilities in modern apps aren't in custom code, they're in the libraries it uses.
- How software is built, common design patterns -- understand MVC, microservices, monoliths, and how data flows between layers. Know what an API gateway does, what a service mesh is, and why a frontend talking directly to a database is a problem. Security decisions map directly to architecture decisions.
- How to read someone else's code -- writing code is one skill, reading unfamiliar codebases is another. In AppSec you will rarely write the code you're reviewing. Practice navigating large repos, tracing data from entry points to sinks, and identifying where user input is handled. This is the core of manual code review.
- How web frameworks handle requests -- understand the request/response lifecycle in frameworks like Express, Django, Spring, or Rails. Know what middleware does, how routing works, and where user input enters the application. Most injection vulnerabilities live in the gap between input handling and data processing.
- How authentication and sessions are implemented -- not just how OAuth or JWTs work conceptually, but how developers actually wire them into an app. Where tokens are stored, how session state is managed, and where developers commonly cut corners under deadline pressure.
- How secrets and environment variables are used -- understand how apps consume API keys, database credentials, and config values at runtime. Know the difference between how secrets should be managed versus how they often are in practice, which is hardcoded in source or committed in .env files.
- How errors are handled -- stack traces, exception handling, logging. Poorly handled errors leak internal details. Over-verbose logging captures sensitive data. This shows up constantly in code review and is easy to miss if you don't know what you're looking for.
- How serialization and deserialization works -- data gets converted between formats constantly: JSON, XML, binary, protocol buffers. Understand how an app parses external input and why deserialization of untrusted data is one of the more dangerous things code can do.
- How version control and branching models work -- understand Git beyond basic commits. Know how feature branches, PRs, and merge strategies work because code review in AppSec happens inside that workflow. If you can't navigate a PR diff or trace a change back through history, you're slower than you need to be.

### Read Code Like an Attacker

Most security tools find the obvious stuff. The real vulnerabilities like business logic flaws, broken authorization, subtle trust boundary violations which require a human who can read code and reason about how an attacker would abuse it.

Security code review is not the same as a functional code review. You are not checking if the code works. You are asking: what happens when someone tries to
break this? Where does user input go? What happens if this assumption is wrong? Who is allowed to call this function, and is that actually enforced?

This skill takes time to develop but it compounds. Every codebase you review makes you faster at the next one. You start recognizing patterns: where injection flaws live, how authentication gets bolted on incorrectly, what insecure deserialization looks like across different languages. The goal is to build intuition that scanners don't have.

What to focus on during a security code review:
- How user input enters the application and whether it is validated, sanitized, and encoded correctly before use
- Authentication and authorization: not just that it exists, but that it cannot be bypassed by manipulating parameters, headers, or request order
- How secrets, credentials, and sensitive configuration are handled at runtime
- Error handling and logging: what gets exposed in stack traces, what gets written to logs that should not be there
- Third party library usage: how OSS dependencies are called and whether they are being used safely
- Business logic: flows that are technically functional but exploitable by someone who understands how the system is supposed to work

#### Know Your Codebase Before You Audit It: CLOC

Before you start reviewing code, understand what you are actually looking at. CLOC (Count Lines of Code) is a free, open source command line tool that scans a codebase and tells you every language present, how many files, and how many lines of code, comments, and blanks exist per language.

GitHub: https://github.com/AlDanial/cloc

This matters more than it sounds. To an experienced AppSec engineer, a CLOC output is a threat modeling shortcut:
- **Languages present** tell you what package managers to look for (npm, pip, Maven, Go modules, Gemfile), which scanners to run, and what vulnerability classes are most relevant to that stack
- **Legacy languages** buried in the output (COBOL, Perl, PHP, old JSP) signal tech debt and code that may not have been touched or reviewed in years, often the most vulnerable parts of the application
- **Disproportionate file counts** in unexpected languages can reveal forgotten scripts, internal tooling, or old services still running in production that nobody is accounting for in the security program
- **Comment density** gives a rough signal of code quality and documentation practices, sparse comments in complex code makes review harder and riskier

Running CLOC takes seconds. It gives you a map before you start the audit, so you are not discovering mid-review that half the backend is written in a language your SAST tool does not support.

## AI & Emerging Technologies
AI security is exploding right now. Get ahead of it:

- Start with TCM Security's AI Fundamentals course (free on YouTube and TCM Academy)
- Go to YouTube and search "AI Security" and you'll find tons of videos. Start digesting and learning, taking notes as you go.
- Dive into Model Context Protocol (MCP), MCP Security, AI Agents, and related topics (Just Google or search on LinkedIn, you'll find a ton of stuff)
- Reference the OWASP Top 10 for LLMs when working with AI/ML applications

### Follow Paolo, as he keeps posting free courses to take and learn on AI, look over his posts
- **Paolo Perrone** -- AI/ML content and audience building, useful follow for LinkedIn content strategy and staying current on AI trends
  https://www.linkedin.com/in/paoloperrone/

### AI Coding/Agents++
Using AI in a technical manner is becoming extremely needed and required in the job space. I highly recommend subscribing to Claude Code Pro (Currently $20 plan) and taking (Claude Code Essentials)[https://www.youtube.com/watch?v=brLhhkUqcn4]. This course will walk you through Claude Code and at least become a sudo beginner. From thre you should build some AI Agent or skill projects
- Dependency risk auditor: paste package.json/requirements.txt/pom.xml, get flagged deprecated/abandoned/CVE-prone deps and suggested replacements.
- Threat model generator from code paste: paste core service files, get a STRIDE-based threat model plus a Mermaid diagram artifact.
- AI agent permission auditor: describe an agent's tool access/scopes, get a report on blast radius if the agent is compromised or hallucinates a bad action.
- IAM policy reviewer: paste AWS/GCP/Azure policy JSON, get least-privilege violations and overly permissive wildcard findings.
- Crypto misuse scanner: scans a codebase for weak algorithms, hardcoded keys, and improper IV/nonce reuse across files, not just a single snippet.


### AI SAST
- **AI Vulnerability Discovery (AIxCC CRS, open source)**:
  Atlantis (Team Atlanta, 1st place), Buttercup (Trail of Bits, 2nd place), Theori CRS (Theori, 3rd place)
  - Atlantis: https://github.com/Team-Atlanta/aixcc-afc-atlantis
  - Buttercup (standalone, maintained): https://github.com/trailofbits/buttercup
  - Buttercup (competition submission): https://github.com/trailofbits/afc-buttercup
  - Theori CRS: https://github.com/theori-io/aixcc-afc-archive

These are not drop-in SAST tools; they are autonomous systems that combine fuzzing, static analysis, and multi-agent LLMs to find and patch vulnerabilities without human input. They competed across 54 million lines of code at DEF CON 33 (August 2025) and collectively found 18 real, non-synthetic vulnerabilities in production open-source projects.  
- Buttercup is the most accessible starting point: 
  - Trail of Bits rebuilt it post-competition as a standalone version designed to run on a laptop. 
  - Theori's repo is archived and unsupported. Atlantis is actively maintained.
- These require LLM API keys (OpenAI, Anthropic, or Google) and can burn through budget fast. Buttercup has a tuned-down mode for individual use.
- Worth studying even if you don't run them: the architecture of how these systems triage, analyze, and patch code is directly relevant to where AI-assisted AppSec tooling is heading.

**Key AI Security Terms to Research:** Prompt injection, Model poisoning, Jailbreaking, Hallucination, Context window attacks, Model extraction, Data leakage, Shadow AI, AI supply chain, Model Context Protocol (MCP), AI Agents, AI-assisted coding

## Build Your Own Secure Pipeline

This is your hands-on laboratory. Set up your own secure pipeline using open source tools. Use GitHub runners or Jenkins locally. Configure it to run against vulnerable code and scan using free open source scanners:

- **SAST**: Semgrep (free), OpenGrep (free), Snyk (free tier), Bandit (Python), Brakeman (Rails), Gosec (Go), SpotBugs (Java), ESLint security plugins (JavaScript) 
    - use language-specific tools where you can, they produce fewer false positives than generic scanners
- **DAST**: OWASP ZAP (free), Nikto (free), Nuclei (free, ProjectDiscovery), Burp Suite Community (free) 
    - Nuclei is worth learning early; the template library is massive and it's widely used in real pipelines
- **SCA (Software Composition Analysis)**: OWASP Dependency-Check, Snyk (free tier), Grype (free, Anchore), OSV-Scanner (free, Google), Dependabot (free, GitHub native) 
    - Grype pairs well with Syft; scan the SBOM you generate rather than the source directly
- **Container Scanning**: Trivy (free), Grype (free), Docker Scout (free tier), Clair (free, open source) 
    - Trivy does double duty here; it also handles IaC and filesystem scanning so it's worth learning well
- **SBOM Generation**: Syft (free, Anchore), CycloneDX CLI (free), Microsoft SBOM Tool (free)
    - understand both CycloneDX and SPDX formats; different tools and consumers expect different formats
- **Secrets Detection**: Gitleaks (free), TruffleHog (free, Trufflesecurity), detect-secrets (free, Yelp) 
    - this category is missing from most beginner lists but secrets in code and git history are one of the most common real-world findings; learn it early
- **IaC Scanning**: Checkov (free, Bridgecrew), Trivy (IaC mode), tfsec (free), KICS (free, Checkmarx) 
    - if you're working in cloud-native environments you will touch Terraform or CloudFormation constantly; knowing how to scan it is expected
- **API Security Testing**: Nuclei (API templates), ZAP (API scan mode), Postman (free tier with basic security testing) 
    - most modern apps are APIs; pure web DAST misses a lot without API-specific tooling

### Example Pipeline Build Guides

- Building a DevSecOps Pipeline with Open Source Tools: https://medium.com/cloud-native-daily/building-a-devsecops-pipeline-with-open-source-tools-ad4fd0e13515
- FPL Insights Dashboard - Real-World DevSecOps Lab Project: https://medium.com/@bll_78288/fpl-insights-dashboard-a-real-world-devops-devsecops-lab-project-85e7065ea7c9
- Complete DevSecOps Pipeline Tutorial (Video): https://m.youtube.com/watch?v=mZoOnWjv_QM

### Infrastructure as Code & Container Security

- Infrastructure as Code in a DevSecOps World: https://snyk.io/learn/infrastructure-as-code-iac/
- Infrastructure as Code Security - OWASP Cheat Sheet Series: https://cheatsheetseries.owasp.org/cheatsheets/Infrastructure_as_Code_Security_Cheat_Sheet.html
- Everything You Need to Know to Get Started With Container Security: https://snyk.io/learn/container-security/
- Building a Strong Container Security Foundation: https://snyk.io/learn/container-security/container-scanning/

The crucial next step is understanding and triaging those results. Learn how to interpret findings, prioritize based on risk and exploitability, and create actionable remediation plans. Being able to run a scanner is table stakes. Knowing how to triage, prioritize, and communicate findings to developers is what gets you hired.

Alternatively, build a similar home lab environment. Document what you build. This becomes portfolio material.

## Learn the Attacker Mindset

Learning the basics of hacking can really contextualize vulnerabilities and is critical for replicating them. Start with the free TCM Security Practical Ethical Hacking course on YouTube: https://youtube.com/playlist?list=PLLKT__MCUeixqHJ1TRqrHsEd6_EdEvo47&si=xxMBSn3Eae4BJH6C

For web application security specifically, work through PortSwigger Web Security Academy (free): https://portswigger.net/web-security -- it covers every major web vulnerability class hands-on in a browser-based lab environment. If you're going into AppSec this is one of the most directly applicable resources available. Most of the vulnerabilities you will find in code reviews and DAST results map directly to topics covered here.

Then consider taking the Certified Penetration Testing Specialist (CPTS) from HackTheBox or the TCM PJPT or PNPT. Understanding how attackers think and operate makes you exponentially better at defense.

## Certifications Worth Considering

While certifications aren't everything, some can provide structured learning paths and credibility:
- Caveat 1: Keep in mind there aren't really AppSec, DevSecOps, or ProdSec certs that just fit all. Looking at Security+, it leans more toward corporate risk rather than ProdSec. It's got useful info, but that's about it.
- Caveat 2: Keep in mind, certs will help you get your foot in the door or pass HR. They're not great for actually learning the day-to-day work. Certs are built to be generic, so the vendor makes them broad and easy enough that folks from all walks can pass.

**Foundation Certifications:**

- CompTIA Security+ (debatable, but gives you a well-rounded perspective)
- AWS Certified Cloud Practitioner (foundations of what AWS and clouds do in general)
- AWS Certified Solutions Architect - Associate (gets your brain thinking on how we build things in the cloud)

**Penetration Testing Certifications** (these teach you how to break things, which helps you learn to defend them):

In order of recommended progression:

- eLearnSecurity Junior Penetration Tester
- CPTS (HTB Certified Penetration Testing Specialist): https://academy.hackthebox.com/preview/certifications/htb-certified-penetration-testing-specialist
- Practical Junior Penetration Tester (PJPT)
- Practical Network Penetration Tester (PNPT)
- Web Pentest Associate (PWPA)

## Core Skills That Separate You From the Pack

**Threat Modeling:**
This is fundamental to AppSec. It's not optional. Threat modeling helps you identify security risks during the design phase, before code is even written. Threat Modeling or Secure Design Review, is evaluating architecture before code is written. Most high-impact vulnerabilities are baked in at the design phase. Catching them there is exponentially cheaper than finding them post-deployment. Focus on methodologies like STRIDE (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege) and PASTA. This skill separates good AppSec engineers from great ones. Hiring managers look for this.
- Adam Shostack Threat Modeling Playlists: https://www.youtube.com/@Shostack/playlists
- Threat Modeling with AI: Turning Every Developer into a Threat Modeler: https://www.youtube.com/watch?v=GW0zQGs8FCY
- Threat Modeling Agentic AI Systems: Proactive Strategies for Security and Resilience: https://www.youtube.com/watch?v=R49Cv7pJ2KA

**Risk Communication:**
A CVSS score means nothing to a VP of Engineering or a CFO. Learn to frame findings in terms of business outcomes: what data is at risk, what does exploitation look like, and what does fixing it cost versus not fixing it. If you can brief a CISO and a developer on the same finding two different ways without losing accuracy in either, 
you will stand out.

**Developer Empathy:**
Developers are under constant pressure to ship and security is rarely their primary job function. If you show up as the person who slows them down and blocks releases, you will be ignored. If you show up as the person who helps them write secure code faster and integrates into their workflow, you become an asset. The best AppSec engineers have either written production code or spent enough time with engineering teams to deeply understand the tradeoffs developers face daily.

**Metrics and Measurement:**
You cannot defend your program without data. Learn what metrics actually matter: mean time to remediate by severity, SLA compliance rates, scanner coverage, and reduction in repeat vulnerability classes over time. Know what metrics are vanity: total vulnerabilities found means almost nothing without context. When budget cycles come around and leadership asks whether the program is working, you need to answer 
with data, not anecdotes.

**Incident Response Fundamentals:**
AppSec engineers get pulled into incidents more than people expect. You do not need to be a full incident responder, but you need to understand the basics: how to scope an incident, what forensic preservation looks like, how to write a timeline, and how to communicate status to leadership without downplaying or escalating prematurely. Being calm and structured during an incident is one of the fastest ways to build 
credibility with senior leadership.

## Recommended Books
- "The Code Book" by Simon Singh: Understanding cryptography and its history
- "The DevOps Handbook": Essential reading for understanding how DevOps and security integrate
- "The DevSecOps Playbook: Deliver Continuous Security at Speed" by Sean D. Mack: Practical guide to implementing DevSecOps
- "Agile Application Security: Enabling Security in a Continuous Delivery Pipeline" by Laura Bell: Security in agile and CI/CD environments
- "Practical Vulnerability Management: A Strategic Approach to Managing Cyber Risk" by Andrew Magnusson: Strategic approach to vulnerability management

## Curated Lists & Collections
- Awesome AppSec: A comprehensive curated list of resources, tools, and learning materials: https://github.com/paragonie/awesome-appsec

# Final Thought

Breaking into AppSec isn't about collecting certifications. It's about building practical skills, understanding how developers work, and being able to secure software at every stage of the SDLC.

The people who thrive in this field share a few traits: they're curious, they don't stop at "it's vulnerable" -- they understand why, and they can explain it to a developer in a way that actually gets it fixed. They know how to prioritize when everything feels urgent. They build relationships with engineering teams instead of policing them. And they stay humble, because this field moves fast and yesterday's knowledge goes stale quickly.

You don't need to know everything to get started. You need to know enough to be useful, be honest about your gaps, and show that you're actively closing them. Hiring managers aren't looking for perfect candidates -- they're looking for people who can learn, contribute, and grow without constant hand-holding.

Build things. Break things. Document what you learn. Write about it, even if the audience is small. Contribute to open source projects. Show up to local meetups. Ask questions publicly. That body of work -- your GitHub, your blog, your community presence -- is often more compelling than any certification on your resume.

The security community is more accessible than it looks from the outside. Most practitioners remember what it felt like to be starting out and are willing to help people who show genuine effort. Put in the work, be consistent, and the door will open.