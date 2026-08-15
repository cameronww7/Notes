# The Cybersecurity Career Map

Real job titles, what they actually do, how hard they are to get, and how many of them exist.

---

## What cybersecurity actually is, in 2026

Start with what attackers are doing right now, because that's what defines the work.

The big shift of the last few years is that breaking in stopped being the hard part. Attackers mostly don't smash through anything anymore, they log in. Stolen credentials, session tokens lifted from a browser, a convincing phone call to a help desk that resets somebody's multi-factor. Identity became the way in, and once they're inside they move sideways through cloud accounts and management tools that were built for administrators, using the same features an administrator uses, which is exactly why it's hard to spot.

The second shift is that the attack surface stopped being your company. Your vendors are your attack surface. Your software's open source dependencies are your attack surface. The SaaS tool one team signed up for with a credit card is your attack surface. A majority of organizations now report a security incident that started somewhere in that chain rather than inside their own walls.

The third is money. Ransomware evolved into extortion, and extortion doesn't need to encrypt anything, it just needs to have your data and a deadline. Backups stopped being a complete answer the moment stealing became more profitable than locking.

And running underneath all of it, AI is compressing timelines on both sides. Phishing that used to be detectable by bad grammar now sounds like your CFO. The gap between a vulnerability going public and it being exploited at scale keeps shrinking. Meanwhile defenders are wiring AI into detection and triage, and companies are quietly discovering how many teams already connected an AI agent to production systems without telling security.

So cybersecurity is not one skill applied to one problem. Look at what that picture actually demands:

- Somebody has to decide who's allowed to be who, and prove it every time. That's identity.
- Somebody has to know what the company even owns, and what's broken in it. That's vulnerability management and asset work.
- Somebody has to watch, notice, and respond when it starts. That's operations.
- Somebody has to check the software before it ships, because it's assembled from thousands of parts nobody on your team wrote. That's application and product security.
- Somebody has to configure the cloud so a single mistake doesn't expose everything. That's cloud security.
- Somebody has to judge which vendors and which risks are acceptable, and write it down so the decision is deliberate. That's risk and governance.
- Somebody has to break it on purpose first. That's offensive security.
- Somebody has to pay for it, staff it, and explain it to a board that's now personally on the hook. That's leadership.

Those are eight completely different jobs. They share a vocabulary and a Slack workspace. They do not share the skills, the day, the personality, or the path in.

### What this document is for

If you're trying to get into this field, or trying to figure out where to go next inside it, the hardest part is that nobody hands you the map. You get told to learn networking, get a cert, do some labs, and then you find out the job you landed doesn't use most of that and the job you actually wanted needed something else entirely.

So this is the map. Every role that meaningfully exists, what the work is in plain language, how realistic it is as a first job, and how many of them are actually out there. Read it, find two or three that sound like something you'd want to do on a Tuesday, and point your studying at those instead of at "cybersecurity."

The goal is not to tell you what to pick. It's to make sure you know what you're picking from.

---

## Why saying "I want to get into cyber" tells nobody anything

Imagine walking up to someone and saying you want to work in medicine.

Cool. Surgeon? Pharmacist? Radiologist? Paramedic? Hospital billing? Medical device sales? Hospital administrator? Every one of those is medicine. They share some vocabulary and a building. They do not share the training, the hours, the temperament, the pay, or what your Tuesday looks like.

Cyber is exactly the same and nobody tells you that up front, so people spend two grand on a certification for a job they would hate.

Here's four real jobs, all of them cybersecurity, all of them hiring right now:

| Job | What Tuesday actually looks like |
|---|---|
| SOC Analyst | A queue of alerts, deciding which ones are real, possibly at 3am |
| Application Security Engineer | Reading code somebody else wrote, then convincing them to change it |
| GRC Analyst | Writing policy, collecting evidence, prepping for an audit |
| Red Team Operator | Breaking into a company on purpose and trying not to get caught |

The SOC analyst and the GRC analyst might go their whole career without writing code. The red teamer has probably never opened a compliance document. The AppSec engineer might not look at a single security alert all year.

There is no such thing as "the cyber skill set." There are about a dozen of them and they barely overlap.

### So before you spend money on anything, answer these

- Do you want to deal with things happening right now, or plan for things that might happen in eighteen months?
- Do you want to build stuff, break stuff, watch stuff, or write stuff down?
- Code, networks, cloud, or people and process?
- Can you do shift work and on-call, or do you need your evenings?
- Do you want to be the person who says no, or the person who finds a way to yes?

Those answers point at completely different jobs. That's what the rest of this is for.

---

## A warning about job titles

This industry has never standardized anything, and titles are the worst offender. Two companies will post the exact same job under different names, and the same title at two companies can mean genuinely different work. Every role below uses a title you'll actually see on a job board, with the common variants listed, but you still have to read the description instead of trusting the header.

A few patterns worth knowing:

- "Security Engineer" and "Security Analyst" are the two biggest catch-alls in the field. Between them they cover an enormous range of work. At a small company, "Security Engineer" might mean you do everything in this document.
- Fancy specialist titles you see in conference talks and vendor marketing, supply chain security, secrets management, detection validation, are usually work someone does inside an existing role, not a job posting you'll find.
- The smaller the company, the more jobs are crammed into one title. The bigger the company, the narrower the slice.

---

# Quick Reference

Every role in this document, in one place. Full details further down.

| Group | Title | In one line |
|---|---|---|
| Security Operations | Security Analyst (SOC Analyst) | Works the alert queue, decides what's real, escalates what matters |
| Security Operations | Incident Response Analyst | Takes over when something is confirmed real and runs the cleanup |
| Security Operations | Detection Engineer | Writes and tunes the rules that generate alerts in the first place |
| Security Operations | Threat Hunter | Goes looking for attackers manually, without an alert pointing the way |
| Security Operations | Malware Analyst | Takes apart suspicious files to figure out exactly what they do |
| Security Operations | Threat Intelligence Analyst | Researches who's attacking, how, and what they'll try next |
| Security Operations | SOC Manager | Runs the operations team, shifts, metrics, and escalation |
| AppSec / ProdSec | Application Security Engineer | Finds security bugs in the company's own code and gets developers to fix them |
| AppSec / ProdSec | Product Security Engineer | Owns security across the whole product, not just the code |
| AppSec / ProdSec | PSIRT Analyst | Handles security holes reported in your product by outsiders and customers |
| Security Engineering | Security Engineer | Builds and runs the defensive tooling, the broadest title in the field |
| Security Engineering | DevSecOps Engineer | Wires security checks into the build and deploy pipeline so they happen automatically |
| Cloud Security | Cloud Security Engineer | Keeps cloud environments configured so one mistake doesn't expose everything |
| Cloud Security | Cloud Security Architect | Designs how cloud environments should be built before anyone builds them |
| Vulnerability Management | Vulnerability Management Analyst | Scans everything, decides what matters, chases teams until it's fixed |
| Vulnerability Management | Vulnerability Management Engineer | Runs the scanning infrastructure and automates the findings pipeline |
| Offensive Security | Penetration Tester | Breaks into systems on purpose within a defined scope, then writes it up |
| Offensive Security | Red Team Operator | Imitates a real adversary for weeks while trying not to get caught |
| Offensive Security | Vulnerability Researcher | Finds flaws nobody knew existed and proves they can be used |
| Offensive Security | Purple Team Engineer | Runs attacks against your own environment to test whether detection actually works |
| Identity (IAM) | IAM Analyst | Handles accounts, access requests, and periodic access reviews |
| Identity (IAM) | IAM Engineer | Builds and runs SSO, MFA, and the directory everything connects to |
| Identity (IAM) | PAM Engineer | Controls and monitors the admin accounts that can do the most damage |
| Architecture / Network | Security Architect | Decides how systems should be secured before they get built |
| Architecture / Network | Network Security Engineer | Runs firewalls, segmentation, and the controls governing traffic |
| GRC | GRC Analyst | Tracks security requirements, collects evidence, survives audits |
| GRC | Risk Analyst | Sizes up what could go wrong and helps leaders decide what to do about it |
| GRC | Third-Party Risk Analyst | Assesses whether your vendors are a way into your company |
| GRC | IT Auditor | Independently tests whether the controls actually work as documented |
| GRC | Privacy Analyst | Governs what personal data is collected, kept, and shared |
| GRC | Security Awareness Manager | Makes the rest of the company harder to trick |
| Specialized | Data Security Engineer | Protects the data itself, classification, encryption, and where it moves |
| Specialized | OT / ICS Security Engineer | Secures factory, utility, and industrial equipment that can't be patched or stopped |
| Specialized | Embedded Security Engineer | Secures firmware and devices, from cars to medical equipment |
| Specialized | AI Security Engineer | Secures machine learning systems and how the company uses AI tools |
| Specialized | Fraud Analyst | Stops abuse from people who signed up legitimately and then misused the product |
| Specialized | Insider Threat Analyst | Looks for risk coming from employees and contractors |
| Leadership / Program | Security Program Manager | Keeps multi-team security initiatives on track and reported |
| Leadership / Program | Security Manager / Director | Leads a security team, owns hiring, budget, and priorities |
| Leadership / Program | CISO | The executive accountable for security across the whole company |
| Adjacent | Security Consultant | Does security work for many client companies at a consulting firm |
| Adjacent | Security Sales Engineer | The technical half of a security vendor's sales conversation |
| Adjacent | Technical Writer (Security) | Writes the documentation, standards, training, and research nobody else will |

---

## How to read the ratings

Every role below gets four things.

**Entry Rating**, 1 to 5, with a plain-language label. This is about how realistic the job is as your first security job, not how hard the work is once you're in it. People confuse those constantly.

| Rating | Label | What it means |
|---|---|---|
| 5 of 5 | True entry level | Companies hire with zero security experience and train you. |
| 4 of 5 | Reachable early | Wants roughly 1-3 years of something adjacent, helpdesk, sysadmin, project management, audit. Not security specifically. |
| 3 of 5 | Career pivot | Roughly 3-5 years in a technical role first, security or otherwise. |
| 2 of 5 | Senior specialist | Roughly 5-8 years including real hands-on security. Rarely hired from outside the field. |
| 1 of 5 | Expert or leadership | 10 plus years. Nobody gets this first, and almost nobody gets it second. |

**Job Availability**, which is how many of these postings actually exist:

| Level | Roughly what it means |
|---|---|
| Very high | Tens of thousands of US openings. Every industry, every city, constantly hiring. |
| High | Thousands of openings. Common at any company past a few hundred people. |
| Moderate | Hundreds to low thousands. Usually needs a company big enough to have a dedicated function. |
| Low | Large enterprises, security vendors, consulting firms, government. You'll wait for the right one to open. |
| Rare | Dozens nationally. Specific industries or a handful of companies. Often filled internally before it ever gets posted. |

**Why the rating**, which explains the number instead of making you guess.

**Key terms**, ten words you'd hear in that job so you can follow a conversation about it.

A 1 rating is not a closed door, it's a destination. Nearly everyone doing 1-rated work today started at a 4 or a 5 and moved.

---

# The Jobs

---

## Security Operations

The SOC is the emergency room. Alerts come in, humans look at them, real problems get escalated to somebody who can do something about it. A lot of SOCs run around the clock, which means they're always hiring, which is why this is the most common way into the industry by a mile.

### Security Analyst (SOC Analyst, Tier 1)
*Also posted as: SOC Analyst, Cybersecurity Analyst, Information Security Analyst, Cyber Defense Analyst*

**Entry Rating:** 5 of 5, true entry level

**Job Availability:** Very high
- This and "Security Engineer" are the two most-posted security titles in the country, and 24/7 coverage means a single SOC needs a lot of bodies.

Your tools generate thousands of alerts. Most of them are garbage, a few of them are somebody actually attacking the company, and your job is to tell the difference. You pick up an alert, gather a little context, and decide to close it, ignore it, or hand it up the chain. There's a written playbook telling you what to check, and you write down what you found so the next shift isn't starting from zero.

People call this the assembly line of security like it's an insult. It isn't. It's the fastest way to see a huge volume of real attacks and build an instinct for what normal looks like, and that instinct is what everything else in defense is built on.

**Why the rating:** Procedural work with training baked in, so companies hire without security experience. Nights and weekends thin out the competition. Tons of people come in from helpdesk, from the military, or straight out of school. Be aware that "Information Security Analyst" also gets used for GRC jobs and even management roles, so read the posting.

**Key terms:** alert, triage, SIEM, playbook, false positive, true positive, escalation, indicator of compromise (IOC), log source, ticket

---

### Incident Response Analyst
*Also posted as: Senior Security Analyst, SOC Analyst Tier 2, Incident Responder, DFIR Analyst*

**Entry Rating:** 3 of 5, career pivot

**Job Availability:** High
- Every SOC needs a senior tier, and consulting firms keep dedicated IR benches to fly out to clients mid-crisis.

When Tier 1 says "this one's real," it lands on your desk. Now you're figuring out what actually happened, how far they got, what they touched, and how to throw them out without tipping them off that you noticed. You're pulling evidence off laptops and servers, building a timeline, and coordinating cleanup with IT teams who have their own day jobs.

The playbook runs out fast here. You are the person answering "how bad is it" while a VP stands behind you waiting for a number.

**Why the rating:** Usually one to three years as a Tier 1, or a solid sysadmin background. The real requirement is being okay making calls with half the information you'd like to have.

**Key terms:** incident, containment, eradication, forensics, timeline, lateral movement, persistence, chain of custody, root cause, post-incident review

---

### Detection Engineer
*Also posted as: Detection & Response Engineer, Security Engineer (Detection), Threat Detection Engineer*

**Entry Rating:** 3 of 5, career pivot

**Job Availability:** Moderate
- Needs a company with a mature SOC, which cuts out most of the market, though every managed security provider hires them.

Instead of answering alerts, you write the things that create them. You study how an attack technique works, then build logic that catches it in your log data. And then, at any company that's been doing this a while, you spend most of your time deleting and tuning rules that fire constantly and mean nothing, because a SOC drowning in noise is worse off than a SOC with fewer rules.

It's a builder job wearing an operations badge.

**Why the rating:** You have to know what attacks look like in actual data, which basically means analyst time first. You also need to be comfortable with query languages and keeping rules in version control like code.

**Key terms:** detection rule, MITRE ATT&CK, telemetry, tuning, signal-to-noise, log normalization, use case, detection-as-code, coverage gap, alert fatigue

---

### Threat Hunter
*Also posted as: Threat Hunting Analyst, Senior Security Analyst (Hunt)*

**Entry Rating:** 2 of 5, senior specialist

**Job Availability:** Low
- At most companies this is a task somebody does when the queue is quiet, not a headcount. The standalone job lives at large enterprises and vendors.

Detection rules only catch the things somebody already thought of. Hunting is what you do about everything else. You assume something is already in the building, you come up with a theory about what it would be doing, and you go dig through the data looking for it without any alert pointing the way.

**Why the rating:** You cannot spot weird if you don't already know what normal looks like in your specific environment, and that takes time in that environment specifically.

**Key terms:** hypothesis, baseline, anomaly, dwell time, pivot, threat model, hunt cycle, living-off-the-land, beaconing, data enrichment

---

### Malware Analyst
*Also posted as: Reverse Engineer, Malware Reverse Engineer, Threat Research Engineer*

**Entry Rating:** 2 of 5, senior specialist

**Job Availability:** Rare
- Concentrated at security vendors, government, and a handful of large enterprises. Most companies send the sample to a vendor instead.

Somebody hands you a suspicious file and you figure out exactly what it does. Sometimes that's detonating it in a sealed environment and watching. Sometimes it's reading the raw machine instructions one at a time because whoever wrote it went to a lot of trouble to make that unpleasant.

**Why the rating:** Low-level programming, assembly, operating system internals. Takes years to get good, and there are a tiny fraction as many of these seats as there are SOC seats.

**Key terms:** sandbox, static analysis, dynamic analysis, disassembly, obfuscation, packer, payload, command and control (C2), YARA rule, sample

---

### Threat Intelligence Analyst
*Also posted as: Cyber Threat Intelligence Analyst, CTI Analyst, Threat Researcher*

**Entry Rating:** 3 of 5, career pivot

**Job Availability:** Moderate
- A good chunk of it sits at vendors selling intelligence rather than at the companies consuming it, so look at vendors too.

You figure out who's going after companies like yours, what they use, how they operate, and what they'll probably try next, then turn that into something your own team can actually do something with. Half this job is reading and correlating sources. The other half is writing for two very different audiences, the responders who want indicators and the executives who want to know if they should be worried.

**Why the rating:** One of the few security roles where an analytical non-technical background genuinely carries over. People land here from intelligence work, journalism, the military, academic research. You still need enough technical grounding that the SOC takes you seriously.

**Key terms:** threat actor, campaign, attribution, TTPs, IOC, intelligence requirement, open-source intelligence (OSINT), dark web, finished intelligence, confidence level

---

### SOC Manager
*Also posted as: Security Operations Manager, Manager of Incident Response*

**Entry Rating:** 1 of 5, leadership

**Job Availability:** Low
- One or two per SOC by definition, and they don't turn over often.

You run the team. Shift coverage, hiring, metrics, escalation paths, and the conversations with the rest of the business about why the SOC needs more people. You're accountable when something gets missed at 2am on a Sunday.

**Why the rating:** You cannot run a queue you've never worked. Everyone can tell.

**Key terms:** mean time to detect (MTTD), mean time to respond (MTTR), coverage, staffing model, runbook, service level agreement (SLA), escalation matrix, tabletop exercise, capacity, burnout

---

## Application and Product Security

AppSec is about the software your company writes, and about the far larger amount of software your company assembled from other people's code. Humans wrote all of it, humans make mistakes, and some of those mistakes let a stranger read your customers' data.

The thing nobody warns you about: you have no authority over the developers. None. You cannot make anyone fix anything. Persuasion is half the job and the half that decides whether you're any good at it.

### Application Security Engineer
*Also posted as: AppSec Engineer, Software Security Engineer, Security Engineer (Application)*

**Entry Rating:** 3 of 5, career pivot

**Job Availability:** Moderate
- Far fewer of these than SOC roles, because a company needs enough engineers writing enough software to justify one. A 200-person company has a SOC contract and zero AppSec people.

You review code and designs, you run and tune the tools that scan code automatically, and you sit with developers to explain what got found and what to do about it. You get pulled into design meetings so you can be the one who points out that the shiny new feature would let any logged-in user pull any other user's records.

This is also where a lot of specialty work actually lives. Securing the build pipeline, dependency and open source risk, secrets that got committed to a repo, container images full of known vulnerabilities. You'll see those written up as their own disciplines online. In a job posting they're bullet points under this role or under DevSecOps.

You have to be able to read code. Not necessarily ship production software, but read it fluently, in whatever languages your company uses, without needing an hour and a coffee to get through a pull request.

**Why the rating:** Software literacy is the gate. Most people get here from a developer background, or from security plus a lot of serious self-taught coding. Almost nobody walks into AppSec with zero technical history.

**Key terms:** SAST, DAST, SCA, secure code review, OWASP Top 10, threat modeling, remediation, false positive, software bill of materials (SBOM), security requirement

---

### Product Security Engineer
*Also posted as: ProdSec Engineer, Senior Security Engineer (Product)*

**Entry Rating:** 2 of 5, senior specialist

**Job Availability:** Low to moderate
- Concentrated at software companies, device manufacturers, and anywhere the product is the business.

Same neighborhood as AppSec, bigger yard. Where AppSec focuses on the code, product security covers the whole thing: the code, the infrastructure under it, how it authenticates people, what it does with data, and how it behaves once it's sitting in a customer's environment where you can't see it anymore.

Worth knowing that at some companies "Product Security Engineer" and "Application Security Engineer" are the same job with a different label, and at others product security is the umbrella and AppSec is one team inside it. Read the description.

**Why the rating:** The scope is the problem. You need to be credible on code, cloud, and architecture at the same time. Usually a step up from AppSec, or where a senior engineer lands when they move into security.

**Key terms:** attack surface, security design review, hardening, security requirement, customer-facing risk, vulnerability disclosure, security architecture, trust boundary, defense in depth, secure default

---

### PSIRT Analyst
*Also posted as: Product Security Incident Response, Vulnerability Response Analyst, Security Response Engineer*

**Entry Rating:** 3 of 5, career pivot

**Job Availability:** Low
- Only exists at companies that ship products to customers, so vendors and manufacturers, and usually only the ones big enough to get a steady stream of reports.

When a researcher, a customer, or a bug bounty hunter reports a hole in your company's product, you're the one who catches it. You verify whether it's real, decide how bad it is, get it in front of the right engineering team, track the fix, and handle the advisory that goes out to customers. You also keep the researcher from getting frustrated and going public while you wait on that fix.

**Why the rating:** Good bridge job. You need enough skill to validate a finding, but you don't have to invent the attack yourself. What you really need is patience, because you'll be the person in the middle when the researcher is annoyed and the engineering team is annoyed and both of them are emailing you.

**Key terms:** triage, severity, CVSS, CVE assignment, coordinated disclosure, advisory, reproduction steps, embargo, safe harbor, researcher relations

---

## Security Engineering and DevSecOps

This org exists because of one hard-won lesson: if security depends on a human remembering to do the right thing, security will not happen. So you build it into the machinery instead, into the pipelines and tools engineers already use, so the safe way is also the easy way.

### Security Engineer
*Also posted as: Information Security Engineer, Cybersecurity Engineer, Security Systems Engineer*

**Entry Rating:** 4 of 5, reachable early

**Job Availability:** Very high
- The other giant catch-all title alongside Security Analyst. Almost every company with a security team has at least one.

The generalist who builds and runs the defensive stack. Endpoint protection, email filtering, network controls, logging agents, the vulnerability scanner, whatever the company bought last quarter. You deploy it, you tune it, you fix it when it breaks, and you get the alerts about it at inconvenient hours.

At a big company this is a specific slice of work with a team around it. At a 300-person company this role is the entire security engineering function and also the person who resets the CEO's MFA. Read the posting, because the same title covers both.

**Why the rating:** The single most common transition from IT and sysadmin work. If you already know how to run servers, networks, and endpoints, you're most of the way there and mostly need the security vocabulary. It's also the title most likely to be posted as "senior" when the actual requirements are not senior, so apply anyway.

**Key terms:** EDR, endpoint agent, deployment, hardening, policy, allowlist, firewall rule, log forwarding, integration, change management

---

### DevSecOps Engineer
*Also posted as: Security Engineer (DevSecOps), Security Automation Engineer, Platform Security Engineer*

**Entry Rating:** 3 of 5, career pivot

**Job Availability:** Moderate
- Growing, but heavily concentrated at software companies. If your company doesn't build software, it doesn't have a pipeline to secure.

Software gets built and shipped through an automated pipeline, and you're the one wiring security into it. Scans that run on every change, checks that stop a bad build from going out, secrets that live in a vault instead of pasted into a config file where they'll sit for six years. You write a lot of scripts and YAML, and you spend a shocking amount of time making your checks faster, because if your scan adds nine minutes to every build the developers will find a way around it and they will be right to.

You'll also end up owning a decent chunk of the security team's own infrastructure, the logging pipelines, the tool integrations, the place all the findings land. Some companies call that platform engineering, most just call it your job.

**Why the rating:** Engineering work with a security specialty on top. Most people arrive from software engineering, DevOps, or platform teams rather than from a security background.

**Key terms:** CI/CD, pipeline, build artifact, secrets management, infrastructure as code, policy as code, shift left, gate, container image, supply chain

---

## Cloud Security

Companies moved their servers out of rooms they owned and into services they rent, and the security problem changed shape completely. The thing that gets most companies popped in cloud isn't a brilliant attack, it's a storage bucket somebody set to public for fifteen minutes during testing eighteen months ago, or an over-permissioned role an attacker found after logging in with someone else's credentials.

### Cloud Security Engineer
*Also posted as: Cloud Security Analyst, Security Engineer (Cloud), Cloud Infrastructure Security Engineer*

**Entry Rating:** 3 of 5, career pivot

**Job Availability:** High
- One of the fastest-growing categories in the field, because nearly every company has cloud now and most of them secured it after the fact.

You make sure the cloud environments are set up in a way that won't end up in a news article. Tools that continuously check for bad configurations, guardrails that stop somebody from creating the bad configuration in the first place, and a lot of conversations with infrastructure teams about the pile of stuff that's already broken.

Container and Kubernetes work usually lives here too. Images with known vulnerabilities, workloads running with far more power than they need, networking that by default lets everything talk to everything. You'll rarely see that posted as its own job, but you'll see it in the requirements.

You need to genuinely understand how at least one of the big cloud providers works under the hood.

**Why the rating:** One of the more efficient pivots available, because cloud knowledge is learnable on your own time and the certifications actually mean something here. Very popular route out of IT and sysadmin work.

**Key terms:** IAM policy, misconfiguration, public exposure, least privilege, CSPM, landing zone, guardrail, shared responsibility model, workload, region

---

### Cloud Security Architect
*Also posted as: Security Architect (Cloud), Principal Cloud Security Engineer*

**Entry Rating:** 1 of 5, expert

**Job Availability:** Low
- One or two per large enterprise, plus consulting firms who rent them out.

You decide how the cloud should be laid out before anyone builds in it. Account boundaries, network design, the identity model, what gets logged and where it goes, how encryption and keys work. You hand engineers the blueprint and then spend the next two years explaining it.

**Why the rating:** Architecture is an opinions job, and good opinions come from having watched designs fail. Expect five to ten years before anyone hands you this.

**Key terms:** reference architecture, segmentation, blast radius, multi-account strategy, encryption at rest, encryption in transit, key management, zero trust, design pattern, control objective

---

## Vulnerability Management

Every company on earth has thousands of known security holes in its systems right now, and none of them can fix all of them. VM is the discipline of knowing what you own, knowing what's broken, deciding what gets fixed first, and confirming it actually got fixed instead of getting marked done in a ticket.

It's unglamorous, it's completely necessary, and it's one of the most reliable ways into the field.

### Vulnerability Management Analyst
*Also posted as: Vulnerability Analyst, Security Analyst (Vulnerability Management), Threat and Vulnerability Analyst*

**Entry Rating:** 4 of 5, reachable early

**Job Availability:** High
- Every company past a certain size has this function, and it's frequently the second security hire after the SOC.

You run scans across everything the company owns, sort through results that come back in the tens of thousands, work out which ones actually matter here, and then chase the teams responsible until things get patched. A huge portion of this job is reporting and follow-up, and being pleasant enough that people answer your messages.

**Why the rating:** Companies hire into this with limited security experience because the tools are learnable and the core skill is organization plus persistence. It's also a sneaky-good place to start, because within six months you'll understand how the company's entire technology estate is put together, which almost nobody else can say.

**Key terms:** CVE, CVSS, scan, asset inventory, remediation, exception, risk acceptance, patch cycle, exploitability, service level agreement (SLA)

---

### Vulnerability Management Engineer
*Also posted as: Security Engineer (Vulnerability Management)*

**Entry Rating:** 3 of 5, career pivot

**Job Availability:** Low to moderate
- Only exists where the company is big enough to split the analyst and engineer halves of the function.

The technical half of the same function. You run and tune the scanning infrastructure, build the pipelines that merge findings from six tools that all name the same thing differently, and automate the reporting so the analysts stop living inside a spreadsheet.

**Why the rating:** Everything the analyst knows, plus scripting, data wrangling, and infrastructure.

**Key terms:** scanner, credentialed scan, asset discovery, deduplication, normalization, data enrichment, coverage, ticketing integration, false positive, dashboard

---

## Offensive Security

Authorized attacking. The point isn't to embarrass the defenders, it's to find the real weaknesses before someone who isn't sending you an invoice finds them, and to check whether anyone would actually notice.

### Penetration Tester
*Also posted as: Security Consultant (Offensive), Ethical Hacker, Offensive Security Engineer, Application Security Consultant*

**Entry Rating:** 3 of 5, career pivot

**Job Availability:** Moderate on paper, brutal in practice
- It's the third most-posted security title in the country, and it's also the job nearly everyone entering the field wants, so the ratio of applicants to openings is unlike anything else here.

You get a defined window and a defined scope, you break into networks, systems, or applications, and you write up everything that worked plus what somebody should do about it. Most of these jobs are at consulting firms testing for many clients rather than inside one company.

Social engineering work usually rides along with this, phishing campaigns and the occasional physical assessment where you walk into a building carrying two boxes so someone holds the door. It's almost never its own job title.

**Why the rating:** Consulting firms do hire and train junior testers, which makes this more reachable than its reputation suggests. But you're competing with a thousand people who watched the same YouTube channel. What separates people is a visible pile of hands-on work: labs, CTFs, real bug bounty findings, writeups you can point at. A degree does almost nothing here. Also worth saying out loud: the report is the deliverable, and a lot of technically strong testers stall out because they can't write.

**Key terms:** scope, reconnaissance, enumeration, exploitation, privilege escalation, pivoting, post-exploitation, finding, severity, rules of engagement

---

### Red Team Operator
*Also posted as: Adversary Simulation Engineer, Senior Offensive Security Engineer, Red Team Engineer*

**Entry Rating:** 1 of 5, expert

**Job Availability:** Rare
- Standing red teams exist at big banks, big tech, defense, and a handful of specialist firms. That's close to the whole list.

You pick a real-world adversary and pretend to be them for weeks or months, going after one specific objective, and a big part of whether you succeeded is whether the defenders ever noticed. Requires patience, stealth, custom tooling, and the discipline to move slowly when everything in you wants to grab the thing and go.

**Why the rating:** One of the hardest doors in the industry. When these teams hire, they hire experienced pentesters who've already proven they can do the work.

**Key terms:** adversary emulation, objective, tradecraft, operational security, command and control (C2), detection evasion, initial access, persistence, exfiltration, deconfliction

---

### Vulnerability Researcher
*Also posted as: Security Researcher, Exploit Developer, Offensive Security Researcher*

**Entry Rating:** 1 of 5, expert

**Job Availability:** Rare
- Vendors, government, and specialist boutiques, and that's about it.

You find flaws nobody knew about, in software or hardware, and prove they can actually be used for something. This is research work, closer to computer science than to IT, and it involves staring at crash dumps for longer than sounds healthy.

**Why the rating:** Memory management, compilers, OS internals, at a depth most engineers never reach.

**Key terms:** zero-day, fuzzing, memory corruption, buffer overflow, mitigation bypass, proof of concept, crash triage, root cause analysis, responsible disclosure, exploit chain

---

### Purple Team Engineer
*Also posted as: Adversary Simulation Engineer, Detection Engineer (Validation), Security Validation Engineer*

**Entry Rating:** 2 of 5, senior specialist

**Job Availability:** Rare as a title, common as an activity
- Most purple teaming is done by detection engineers and pentesters who get in a room together for a week, not by someone with the title on their badge.

You run a controlled attack technique against your own environment, then check whether anything caught it, and when nothing did, you sit down with detection engineering and fix the gap. Then you run it again. It's a loop, not a contest, and it's the fastest way to find out that the expensive tool everyone assumed was covering something is not covering it at all.

**Why the rating:** You have to be credible on both sides of the table, offensive technique plus detection engineering, and that combination takes years to assemble.

**Key terms:** adversary emulation, detection coverage, MITRE ATT&CK, technique, control validation, gap analysis, atomic test, telemetry, tuning, feedback loop

---

## Identity and Access Management

Identity is where the fight actually is right now. Most modern intrusions start with credentials that work, taken by phishing, pulled out of a repo, lifted from a browser session, or talked out of a help desk. Everything downstream depends on whether the account being used is really the person it claims to be.

### IAM Analyst
*Also posted as: Identity and Access Management Analyst, Access Management Analyst, IAM Administrator*

**Entry Rating:** 4 of 5, reachable early

**Job Availability:** High
- Frequently sits in IT rather than security, which is worth knowing when you search, because you'll miss half the postings otherwise.

The operational side of identity. Accounts get created when people join, access gets removed when they leave, requests come in and get approved or denied, and every quarter you run the reviews where managers confirm their people still need everything they have, which they will do at 4:58pm on the last day.

**Why the rating:** Process-driven with clear procedures, so companies hire from IT support and even from non-technical operations backgrounds. Good visibility into how the whole org works.

**Key terms:** provisioning, deprovisioning, access review, entitlement, role-based access control (RBAC), joiner-mover-leaver, separation of duties, privileged account, group membership, approval workflow

---

### IAM Engineer
*Also posted as: Identity Engineer, Security Engineer (Identity), Okta Engineer, Entra ID Engineer*

**Entry Rating:** 3 of 5, career pivot

**Job Availability:** High
- One of the most consistently in-demand skill sets in the field, and it got more so once attackers made identity the primary way in.

You build and run the identity plumbing. Single sign-on, multi-factor, the directory that holds every account in the company, and the integrations connecting a few hundred applications to all of it. When this breaks, nobody in the company can work, which tells you something about how it gets treated.

The fast-growing corner of this is non-human identity: the service accounts, API keys, tokens, and certificates that applications use instead of passwords. There are usually ten to fifty of those per human, nobody owns them, nobody rotates them, and a lot of them have more access than the CTO. Real problem, real specialty, but you'll see it as a requirement inside this role rather than as its own posting.

**Why the rating:** Needs authentication protocol knowledge and a tolerance for integration work. Very common pivot for anyone who's administered a directory service.

**Key terms:** SSO, SAML, OIDC, MFA, federation, directory service, conditional access, service account, token, identity provider

---

### PAM Engineer
*Also posted as: Privileged Access Management Engineer, CyberArk Engineer, IAM Engineer (Privileged Access)*

**Entry Rating:** 3 of 5, career pivot

**Job Availability:** Moderate
- Heavily concentrated in regulated industries like finance, healthcare, and energy, where an auditor is asking about it.

Admin accounts are the keys to everything, so they get special treatment: passwords in a vault, access granted only for a specific window, sessions recorded. You run that system, and you spend a fair amount of energy on the engineers who want their standing admin access back.

**Why the rating:** A specialty inside IAM, usually reached after general identity or systems work. The tooling is vendor-specific and learnable once you're in the seat, which is why you see vendor names in the job titles.

**Key terms:** vault, credential rotation, just-in-time access, session recording, break-glass account, privilege escalation, service account, checkout, least privilege, standing access

---

## Architecture and Network Security

### Security Architect
*Also posted as: Enterprise Security Architect, Cybersecurity Architect, Principal Security Engineer*

**Entry Rating:** 1 of 5, expert

**Job Availability:** Low
- A handful per large enterprise, plus consulting. Small companies don't have one, they have a senior engineer who also does this.

You review systems before they get built and say how they should be secured. You write the standards everyone designs against. And you make the call when the security requirement and the delivery date are in direct conflict, which is most Tuesdays.

**Why the rating:** Credibility is the whole job. You're advising senior engineers who've been building for fifteen years, and they will not take direction from someone who has never built or defended anything.

**Key terms:** reference architecture, design review, security control, trust boundary, threat model, standard, requirement, risk tradeoff, zero trust, defense in depth

---

### Network Security Engineer
*Also posted as: Security Engineer (Network), Firewall Engineer, Network Security Analyst*

**Entry Rating:** 3 of 5, career pivot

**Job Availability:** High
- Steady rather than trendy, which is its own kind of good. Every company with physical offices and data centers still needs this.

You design and run the controls that decide how traffic moves. Firewalls, segmentation, VPNs, monitoring. Segmentation especially, because it's what determines whether one compromised laptop stays one compromised laptop or becomes the entire company.

**Why the rating:** The cleanest pivot in the field for an existing network engineer, and genuinely difficult without networking fundamentals.

**Key terms:** firewall, segmentation, VLAN, VPN, east-west traffic, north-south traffic, network access control, packet inspection, proxy, microsegmentation

---

## Governance, Risk, and Compliance

GRC is the part of security dealing with rules, evidence, and decisions. Somebody has to define what "secure enough" means here, prove it to auditors and customers, and make sure risk decisions get made on purpose rather than by nobody noticing.

People love to dismiss this as paperwork. Those people are usually the ones who can't figure out why their project didn't get funded. GRC touches budget, priorities, and executive attention, and it's one of the faster routes to senior leadership in the whole field.

### GRC Analyst
*Also posted as: Compliance Analyst, Security Compliance Analyst, Information Security Analyst (GRC), IT Compliance Analyst*

**Entry Rating:** 4 of 5, reachable early

**Job Availability:** High
- Rising, because every company selling to another company now gets asked for a security attestation before the contract gets signed.

To sell to certain customers or operate in certain industries, a company has to meet a specific security standard. You track those requirements, gather the evidence proving each one is happening, and run the audits that verify it. A lot of chasing screenshots and a lot of asking engineers for things they consider a waste of their time.

Writing and maintaining the actual policies usually falls here too, along with the exceptions, the annual reviews, and the arguments about whether the rule applies to this one team's very special situation. Only very large companies have a dedicated policy person.

**Why the rating:** One of the genuinely open non-technical doors into security. People come in from accounting, audit, project management, paralegal work. The technical concepts get picked up as you go.

**Key terms:** control, framework, evidence, audit, SOC 2, ISO 27001, PCI DSS, gap assessment, policy exception, attestation

---

### Risk Analyst
*Also posted as: Cyber Risk Analyst, Information Risk Analyst, IT Risk Analyst*

**Entry Rating:** 3 of 5, career pivot

**Job Availability:** Moderate
- Concentrated in finance, insurance, healthcare, and anywhere else with a regulator watching.

You identify what could go wrong, estimate how likely it is and how much it would hurt, and help leaders decide whether to fix it, insure it, accept it, or walk away from it. Then you write the decision down somewhere it'll actually get revisited, which is the part everybody skips.

**Why the rating:** Needs enough technical grounding to tell a real risk from a theoretical one, plus the ability to explain it to someone who doesn't care about the technology. Common pivot from audit, finance, or operational risk.

**Key terms:** risk register, likelihood, impact, inherent risk, residual risk, risk appetite, mitigation, risk acceptance, control effectiveness, quantification

---

### Third-Party Risk Analyst
*Also posted as: TPRM Analyst, Vendor Risk Analyst, Supplier Security Analyst*

**Entry Rating:** 4 of 5, reachable early

**Job Availability:** Moderate to high
- One of the more overlooked doors in the field, and growing fast now that most organizations report a security incident that came through a vendor rather than through their own front door.

Your company depends on hundreds of outside vendors and every one of them is a potential way in. You assess vendors before they get onboarded, read their audit reports, and keep an eye on the relationship afterward instead of filing the questionnaire and forgetting it existed.

**Why the rating:** Structured, procedure-driven, and regularly staffed with people new to security, especially from procurement, contracts, and operations backgrounds.

**Key terms:** vendor assessment, questionnaire, SOC 2 report, contractual requirement, fourth party, onboarding, remediation commitment, concentration risk, right to audit, continuous monitoring

---

### IT Auditor
*Also posted as: Internal IT Auditor, IT Audit Analyst, Security Assessor, Information Systems Auditor*

**Entry Rating:** 3 of 5, career pivot

**Job Availability:** High
- The big accounting firms hire in volume every single year, which makes this one of the most predictable doors in the entire field.

You independently check whether the controls are doing what the documentation claims, and you write up where they aren't. Internal auditors work inside one company, external auditors work for a firm and assess a lot of them.

**Why the rating:** Audit methodology transfers straight over from financial and operational audit, so career changers with that background do well. Coming in with neither audit nor technology experience is a longer road.

**Key terms:** control testing, sample, evidence, finding, material weakness, scope, audit trail, compensating control, management response, attestation

---

### Privacy Analyst
*Also posted as: Data Privacy Analyst, Privacy Program Analyst, Privacy Engineer (technical variant)*

**Entry Rating:** 4 of 5 for the analyst role, 2 of 5 for privacy engineering

**Job Availability:** Moderate for analysts, low for engineers
- Concentrated at companies handling consumer data at scale, and at anyone operating in Europe or California.

Privacy is about what personal data you collect, why you have it, how long you keep it, and who you hand it to. Analysts run the legal and process side. Privacy engineers build the technical controls that make it real, deletion that actually deletes, anonymization that actually anonymizes, consent that actually gets honored downstream.

**Why the rating:** The analyst role is a common landing spot from legal, policy, and compliance work. The engineering version needs serious data infrastructure skill and is a much harder door.

**Key terms:** personal data, GDPR, data subject request, consent, data minimization, retention, anonymization, pseudonymization, lawful basis, data mapping

---

### Security Awareness Manager
*Also posted as: Security Awareness and Training Specialist, Security Culture Manager, Human Risk Manager*

**Entry Rating:** 4 of 5, reachable early

**Job Availability:** Low
- Usually one person for the whole company, and often it's a slice of somebody's GRC job rather than a headcount of its own.

You're responsible for making everyone else at the company harder to trick, which matters more every year as AI makes phishing and voice impersonation genuinely convincing. Training content, phishing simulations, internal comms, and measuring whether any of it changed behavior instead of just measuring who clicked through the module while on a call.

**Why the rating:** Teaching, writing, and communication are the actual requirements. People come in from marketing, internal comms, education, and HR and do very well. The catch is that there just aren't many of these seats.

**Key terms:** phishing simulation, click rate, reporting rate, awareness campaign, training completion, security culture, behavior change, targeted training, human risk, nudge

---

## Specialized Domains

These are real jobs with real postings, but the availability numbers are the point. Do not build a plan around landing one of these first.

### Data Security Engineer
*Also posted as: Data Protection Engineer, Security Engineer (Data), DLP Engineer*

**Entry Rating:** 2 of 5, senior specialist

**Job Availability:** Low
- Mostly large enterprises and heavily regulated industries.

You protect the data itself rather than the walls around it. Finding where the sensitive stuff actually lives, which is never only where the architecture diagram says, classifying it, controlling who can query it, encrypting it, and noticing when a lot of it moves somewhere it shouldn't. Extortion attacks made this matter more, because stealing the data is now the whole plan.

**Why the rating:** Databases, data platforms, and encryption, at depth. Usually a pivot from data engineering or infrastructure security.

**Key terms:** data classification, discovery, data loss prevention (DLP), encryption, tokenization, masking, key management, exfiltration, data lineage, access control

---

### OT / ICS Security Engineer
*Also posted as: Industrial Control Systems Security Engineer, SCADA Security Engineer, OT Security Analyst*

**Entry Rating:** 2 of 5, senior specialist

**Job Availability:** Low overall
- But the applicant pool is even smaller, so the odds are better than the raw number suggests. Critical infrastructure is now a standing target in nearly every threat report published, which is pushing budget here.

Factories, power plants, water treatment, pipelines, hospitals. All running specialized equipment that was designed decades ago to never touch a network and now touches a network. You can't patch it, you can't reboot it, it's going to run for another twenty years, and if it stops, something physical stops with it. Safety outranks security, and availability outranks basically everything.

**Why the rating:** You need to understand the industrial process as well as the security, and a lot of the best people here came from plant operations rather than IT.

**Key terms:** SCADA, PLC, HMI, Purdue model, air gap, safety instrumented system, legacy protocol, availability, physical process, network segmentation

---

### Embedded Security Engineer
*Also posted as: Firmware Security Engineer, Product Security Engineer (Hardware), Automotive Security Engineer, Medical Device Security Engineer*

**Entry Rating:** 2 of 5, senior specialist

**Job Availability:** Rare
- Tied to specific industries: automotive, medical devices, industrial equipment, consumer hardware.

Cars, pacemakers, sensors, smart home gear. All of it runs software, most of it connects to something, and a bad update can be a recall or worse. You secure the device, its firmware, and the mechanism that updates it, with regulators watching.

**Why the rating:** Embedded systems and hardware knowledge, which comes from engineering programs and not from IT.

**Key terms:** firmware, secure boot, over-the-air update, hardware root of trust, JTAG, side-channel, device provisioning, safety regulation, supply chain, attack surface

---

### AI Security Engineer
*Also posted as: AI/ML Security Engineer, Security Engineer (AI), AI Red Team Engineer*

**Entry Rating:** 2 of 5, senior specialist

**Job Availability:** Low but climbing fast
- Currently concentrated at AI companies, big tech, and security vendors building AI products. Expect this to spread outward as more companies put agents into production.

You're securing machine learning systems against a category of attacks that didn't exist a few years ago, and at the same time you're securing how your own company uses AI tools, which is largely discovering how many teams already connected an agent to production systems without telling anybody.

**Why the rating:** New enough that there's no established path, which cuts both ways. You need either strong ML understanding or strong security understanding plus the ability to learn fast. Titles and expectations vary wildly right now, so read the description carefully, because two roles with the same title can be completely different jobs.

**Key terms:** prompt injection, model poisoning, training data, guardrail, jailbreak, model inversion, agentic system, evaluation, data leakage, adversarial input

---

### Fraud Analyst
*Also posted as: Fraud Prevention Analyst, Trust and Safety Analyst, Risk Operations Analyst*

**Entry Rating:** 4 of 5, reachable early

**Job Availability:** High
- At consumer tech, banks, payments companies, and marketplaces, all of which hire these in volume.

Not every attacker breaks in. Plenty of them sign up legitimately and abuse the product from inside the front door: fake accounts by the thousand, stolen cards, scams pointed at your other users. You find the patterns and shut them down without breaking the experience for everyone real.

**Why the rating:** Common entry point, and pattern recognition matters more here than deep security knowledge. If you like puzzles and large datasets, this is a very underrated door. Fair warning that trust and safety work can involve reviewing genuinely upsetting content depending on the platform.

**Key terms:** account takeover, chargeback, velocity, device fingerprint, bot, fraud ring, false decline, rules engine, anomaly detection, abuse pattern

---

### Insider Threat Analyst
*Also posted as: Insider Risk Analyst, Insider Threat Program Analyst*

**Entry Rating:** 3 of 5, career pivot

**Job Availability:** Rare
- Mostly at defense contractors, government, big banks, and large tech companies, and many of these roles require a security clearance, which is its own gate.

You look for risk coming from inside the building. Someone copying the customer list two weeks before they resign, someone poking at systems they have no business in, someone who got approached by an outsider and said yes. Unusually sensitive work, because these are your coworkers, and you'll be joined at the hip with HR and Legal for good reason.

**Why the rating:** Discretion and judgment matter more than deep technical skill, but the sensitivity means companies are picky. Backgrounds in investigations, law enforcement, and HR show up a lot.

**Key terms:** user behavior analytics, baseline, data exfiltration, watchlist, case management, HR referral, privacy consideration, legal hold, indicator, intent

---

## Leadership and Program Roles

### Security Program Manager
*Also posted as: Technical Program Manager (Security), Security Project Manager, Cybersecurity Program Manager*

**Entry Rating:** 4 of 5, reachable early

**Job Availability:** Moderate
- Mostly at companies large enough to have several security teams that need coordinating.

Security initiatives span a dozen teams and run for months, and somebody has to keep them alive. Scope, timeline, dependencies, blockers, and the status update to leadership. You don't have to be the most technical person in the room, but you do need enough understanding to tell when "we're on track" isn't true.

**Why the rating:** One of the strongest doors for experienced program and project managers. The transferable skill is most of the job and the domain knowledge comes with time.

**Key terms:** roadmap, milestone, dependency, stakeholder, blocker, scope, status reporting, cross-functional, resourcing, delivery risk

---

### Security Manager / Director
*Also posted as: Information Security Manager, Director of Information Security, Manager of Security Engineering*

**Entry Rating:** 1 of 5, leadership

**Job Availability:** Moderate
- Every company with a security team has at least one, but they turn over slowly.

You lead a team. Hiring, budget, priorities, performance reviews, and representing your people to a business that mostly wants to know why this costs so much. Your hands-on depth stops growing right about here and your influence starts mattering a lot more than your skills did.

**Why the rating:** Needs technical credibility and management ability at the same time, which is rarer than either one on its own.

**Key terms:** headcount, budget, roadmap, metrics, stakeholder management, vendor management, prioritization, team development, escalation, business alignment

---

### CISO
*Also posted as: Chief Information Security Officer, VP of Information Security, Head of Security*

**Entry Rating:** 1 of 5, expert

**Job Availability:** Low by definition
- One per company, though "Head of Security" at a startup is a much more reachable version of the same idea.

The executive accountable for security across the company. The day is budget, board decks, hiring, regulatory exposure, and the knowledge that if something goes badly wrong your name is on it in a way that now carries real legal weight. Boards have started involving themselves directly in these decisions, which changed the job.

**Why the rating:** End of a long road. Paths run through engineering leadership, GRC leadership, and consulting, and the job is far less technical than people expect.

**Key terms:** board reporting, risk appetite, budget, regulatory exposure, incident accountability, security strategy, materiality, cyber insurance, program maturity, executive alignment

---

## The Ones People Forget Are Also Cyber Jobs

### Security Consultant
*Also posted as: Cybersecurity Consultant, Security Advisory Consultant, Associate Consultant*

**Entry Rating:** 3 of 5, career pivot, but the big firms have genuine entry-level programs that run closer to a 5

**Job Availability:** High
- The consulting firms are among the largest employers of security people in the world, and they hire on a schedule every year.

You work at a firm doing security work for a lot of client companies. Assessments, program builds, incident response, pentests, whatever got sold. You'll see more environments in two years than most internal people see in ten, and you'll pay for it in utilization targets, travel, and the occasional client who bought a report they never intended to read.

**Why the rating:** The structured graduate and associate programs at the large firms are one of the most reliable ways into this industry with no experience, full stop. The tradeoff is pace.

**Key terms:** engagement, statement of work, deliverable, utilization, client stakeholder, assessment, maturity model, recommendation, roadmap, scoping call

---

### Security Sales Engineer
*Also posted as: Solutions Engineer, Solutions Architect (Security), Pre-Sales Engineer, Technical Account Manager*

**Entry Rating:** 3 of 5, career pivot

**Job Availability:** Moderate
- Every security vendor is hiring them constantly, and there are a lot of security vendors.

You work for a vendor and you're the technical half of the sales conversation. Demos, proof-of-concept deployments, and answering the customer's hard questions honestly enough that they trust you, which is the entire job and also the thing that separates the good ones from the ones nobody calls back.

**Why the rating:** You need technical ability and the willingness to talk to strangers all day, and that combination is rarer than you'd think, which is why it pays well. Vendors will train someone technically capable who can present.

**Key terms:** proof of concept, demo, discovery call, requirements, competitive differentiation, deployment, evaluation criteria, technical win, integration, customer environment

---

### Technical Writer (Security)
*Also posted as: Technical Writer, Documentation Specialist, Content Developer*

**Entry Rating:** 4 of 5, reachable early

**Job Availability:** Low as a security-specific role
- But general technical writing openings at security companies are plentiful and get you to the same place.

Somebody has to write the documentation, the standards, the training, the threat reports, the product content. This industry is full of brilliant people who cannot write a clear paragraph, which makes anyone who can unusually valuable and weirdly hard to find.

**Why the rating:** The writing is the qualification. The vocabulary gets built while you're doing the job. Badly underrated as a way in, and it puts you next to practitioners all day.

**Key terms:** documentation, style guide, audience, technical accuracy, subject matter expert, review cycle, publication, glossary, information architecture, plain language

---

## The feeder jobs nobody lists as cybersecurity

These are not security titles. They are how a large share of security people actually got here, and they're worth taking seriously if you're starting cold, because they're plentiful, they hire without experience, and every one of them teaches you something the security job assumes you already know.

| Role | What it sets you up for |
|---|---|
| IT Help Desk / Support Technician | SOC Analyst, IAM Analyst, Security Engineer |
| Systems Administrator | Security Engineer, Incident Response, Cloud Security, Vulnerability Management |
| Network Engineer | Network Security Engineer, Security Architect |
| Software Engineer | Application Security, Product Security, DevSecOps |
| DevOps / Platform Engineer | DevSecOps, Cloud Security |
| Financial or Operational Auditor | IT Auditor, GRC Analyst, Risk Analyst |
| Project or Program Manager | Security Program Manager, GRC |

Patching and configuration management, which you'll sometimes see written up as its own security career, lives inside the sysadmin row. It's real work and it matters enormously, it just isn't usually its own job.

---

# Where to actually start

Sorted by how many of these jobs exist, because a 5-rated role with no openings is worse than a 4-rated role with thousands.

| If you are | Look at | Availability |
|---|---|---|
| Willing to work shifts and learn fast | Security Analyst / SOC Analyst | Very high |
| Coming from IT or sysadmin work | Security Engineer, IAM Analyst | Very high |
| Organized and relentless about follow-up | Vulnerability Management Analyst | High |
| Good with process and documentation | GRC Analyst, Third-Party Risk Analyst | High |
| From an audit, finance, or legal background | IT Auditor, GRC Analyst, Privacy Analyst | High |
| Good at spotting patterns in data | Fraud Analyst | High |
| Willing to trade lifestyle for pace | Security Consultant, entry program at a large firm | High |
| Experienced in program or project management | Security Program Manager | Moderate |
| A strong communicator or teacher | Technical Writer, Security Awareness | Low |

Anything rated 3 or lower is a second or third job, not a first one. That's not a wall, it's an order of operations. The people doing red team work and architecture right now almost all started somewhere in that table, and most of them will tell you the boring first job is where they learned the thing that made everything after it possible.

So stop asking how to get into cybersecurity. Start asking which one of these you'd actually want to do for three years, and then study for that.
