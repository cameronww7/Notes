# Product Security Definitions

**Contents**

1. [Framing and Scope](#1-framing-and-scope)
2. [Core Vocabulary](#2-core-vocabulary)
3. [Governance, Policy, and Standards](#3-governance-policy-and-standards)
4. [Program Operating Model](#4-program-operating-model)
5. [Security Architecture](#5-security-architecture)
6. [Threat Modeling and Risk Analysis](#6-threat-modeling-and-risk-analysis)
7. [Secure Development](#7-secure-development)
8. [Application Security Testing](#8-application-security-testing)
9. [Software Supply Chain Security](#9-software-supply-chain-security)
10. [Pipeline and CI/CD Security](#10-pipeline-and-cicd-security)
11. [Cloud, Infrastructure, and Runtime](#11-cloud-infrastructure-and-runtime)
12. [Identity, Access, and Secrets](#12-identity-access-and-secrets)
13. [API and Interface Security](#13-api-and-interface-security)
14. [Vulnerability Management and Prioritization](#14-vulnerability-management-and-prioritization)
15. [PSIRT, Disclosure, and Product Incident Response](#15-psirt-disclosure-and-product-incident-response)
16. [AI and Agentic Security](#16-ai-and-agentic-security)
17. [Data Protection and Privacy Engineering](#17-data-protection-and-privacy-engineering)
18. [Metrics, Measurement, and Maturity](#18-metrics-measurement-and-maturity)
19. [Culture, Enablement, and Education](#19-culture-enablement-and-education)
20. [Hardware, Embedded, and Connected Products](#20-hardware-embedded-and-connected-products)

---

## 1. Framing and Scope

### Cyber
Is a word industry professionals use when talking to non technies, is a way to describe what we do. (I work in cyber)

### AppSec
Is everything you do to make sure software is secure.

**Scope note:** in practice AppSec has come to mean the code, the dependencies, the developer workflow, and the testing that surrounds them. It is the largest single component of product security but not the whole of it.

### Product Security (ProdSec)
**Definition:** The discipline accountable for the security of a product across its entire life, from concept through end of support. It includes the code (AppSec), the architecture it is built on, the pipeline that builds it, the infrastructure and identities it runs with, the interfaces it exposes, the way vulnerabilities in it are received and fixed, the evidence provided to customers and regulators, and the security commitments made to the people who bought it.

**Problem it solves:** AppSec can tell you a dependency has a critical CVE. It cannot tell you whether that service is internet reachable, what permissions it runs with, whether exploitation would expose customer data, who is contractually owed a notification, or how a fix reaches a customer running a two-year-old version. Product security owns those questions.

**When it's the WRONG choice:** Do not stand up a product security function with product security scope when you have one application, four engineers, and no shipped release. You need AppSec fundamentals and a CI pipeline first. Declaring a broad-scope ProdSec program before you have basic scanning, an asset inventory, and a way to fix things produces an org chart, not security.

### Security Engineering
**Definition:** The practice of building and operating the systems, automation, tooling, and infrastructure that deliver security capability, as opposed to performing security assessments manually. Output is code, platforms, and pipelines rather than reports.

**Problem it solves:** Manual security work does not scale past a handful of teams. Security engineering converts a repeated human activity into a system that runs without a human in the loop.

**When it's the WRONG choice:** Automating a process you have not yet performed manually enough times to understand. You will encode the wrong assumptions and spend a quarter maintaining a tool that answers the wrong question. Do the work by hand three times, then automate.

### DevOps
DevOps (a portmanteau of "development" and "operations") is the combination of practices and tools designed to increase an organization's ability to deliver applications and services faster than traditional software development processes.

DevOps is a set of practices that combines software development and IT operations. It aims to shorten the systems development life cycle and provide continuous delivery with high software quality. DevOps is complementary with Agile software development; several DevOps aspects came from the Agile methodology.

### DevSecOps
**Definition:** The integration of security practice into DevOps workflows so that security work happens inside the same pipelines, tools, backlogs, and rituals engineering already uses, rather than in a parallel process owned by a separate team.

**Problem it solves:** Security as a separate downstream process creates a queue. The queue creates delay, the delay creates pressure, and pressure creates exceptions. Putting security into the existing workflow removes the queue.

**When it's the WRONG choice:** DevSecOps assumes a functioning DevOps practice to attach to. In organizations with manual release processes, no CI, mixed tech stacks with no shared tooling, or quarterly release trains, "DevSecOps" becomes a label on the same old gate review. Fix the delivery pipeline first or accept that you are doing staged security reviews and call them that.

### Platform Security
**Definition:** The approach of solving security problems once at the platform layer (shared libraries, base images, service templates, CI templates, identity broker, gateway) so that individual product teams inherit the control rather than implement it.

**Problem it solves:** Asking 200 teams to each implement correct authorization, TLS, logging, and secret handling produces 200 different levels of correctness. Solving it in the platform produces one.

**When it's the WRONG choice:** When there is no platform and no platform team. Building a shared library that only one team adopts is worse than that team using an established open-source library, because now you own the maintenance. Also wrong when product teams have legitimately divergent requirements and the "platform" control becomes a lowest common denominator nobody wants.

### Paved Road / Golden Path
**Definition:** A supported, documented, opinionated default way to build and ship a service that has security controls already wired in. Teams that stay on the road get security for free. Teams that leave the road take on the review burden themselves.

**Problem it solves:** It changes the incentive. Security stops being a tax on the default path and becomes the reason the default path is easier.

**When it's the WRONG choice:** If you cannot resource the road's maintenance. A stale paved road with outdated dependencies and broken templates actively drives teams off it and destroys the credibility of the model. Also wrong as a mandate before the road exists and is demonstrably better than what teams do today.

### Shift Left
**Definition:** Moving security activity earlier in the lifecycle, toward design and development, where flaws are cheaper to fix and the person who introduced the issue still has context.

**Problem it solves:** Findings delivered after release are expensive, contentious, and often never fixed.

**When it's the WRONG choice:** As a slogan replacing coverage. Shifting left without shifting right leaves you blind to configuration, deployment, and runtime issues that only exist in production. Also wrong when "shift left" means dumping raw scanner output on developers with no triage; you have shifted work, not security.

### Shift Everywhere / Shift Smart
**Definition:** The correction to shift left. Place each control where it produces the highest-signal result at acceptable cost, which for some controls is the IDE, for others the PR, for others the pipeline, and for others production runtime.

**Problem it solves:** Some issues are only visible where they are visible. Reachability, exposure, misconfiguration, and identity over-permission are runtime facts.

**When it's the WRONG choice:** Rarely wrong as a principle, but it is often used to justify buying tools at every stage. The discipline is choosing the single best placement for each control, not adding placements.

### Secure by Design
**Definition:** Building products so that security properties come from architectural and design decisions rather than from bolt-on controls or user configuration. The stronger form, promoted by CISA and now embedded in EU regulation, treats security outcomes as a manufacturer's responsibility rather than a customer's.

**Problem it solves:** Entire vulnerability classes disappear when design forecloses them. Parameterized query APIs eliminate injection more reliably than any scanner or training program.

**When it's the WRONG choice:** As a rationale for delaying delivery of controls you can ship today. "We will fix this architecturally next year" is frequently how a known exploitable flaw survives a year. Design fixes and compensating controls are not mutually exclusive.

### Secure by Default
**Definition:** Products ship with the secure configuration active, and reducing security requires deliberate action by the customer. No default passwords, no permissive CORS, no wide-open buckets, TLS on, audit logging on, MFA on.

**Problem it solves:** Most customers never change defaults. The default is therefore the real security posture of your installed base.

**When it's the WRONG choice:** When the secure default breaks a documented customer workflow and you have not provided a migration path. Shipping a breaking secure default without a deprecation window generates support load and encourages customers to disable the control wholesale rather than adapt.

### Defense in Depth
**Definition:** Layering independent controls so that failure of any single one does not produce compromise.

**Problem it solves:** Every control fails. Layering converts single-point failure into a chain that must all fail together.

**When it's the WRONG choice:** When it becomes an excuse for weak individual layers ("the WAF will catch it"). Also wrong when layers are not actually independent; four controls that all depend on the same identity provider are one control. And each layer has operational cost, so depth applied to low-value assets is waste.

### Least Privilege
**Definition:** Every identity, process, and component gets the minimum permissions required for its function, and nothing more.

**Problem it solves:** Caps the damage of any single compromise. It is the control that turns a breach into an incident.

**When it's the WRONG choice:** Almost never wrong as a target, frequently wrong as a rollout strategy. Aggressive permission cuts without usage telemetry cause outages, and outages cause permanent blanket exceptions. Measure actual usage, then cut.

### Least Agency
**Definition:** The agentic analogue of least privilege: granting an AI agent the minimum autonomy, tool access, and action scope its task requires. Constrains not just what it can reach but what it is permitted to decide and do without confirmation.

**Problem it solves:** An agent's blast radius is the union of every tool, credential, and API it can invoke. Least agency shrinks it.

**When it's the WRONG choice:** When over-applied to the point that the agent needs human confirmation for every step. You then have a slow human process with extra latency, and users will route around it or approve reflexively, which is worse than no approval gate.

### Zero Trust
**Definition:** An architectural stance that removes implicit trust based on network location. Every request is authenticated, authorized, and evaluated against policy regardless of origin.

**Problem it solves:** Flat internal networks mean one compromised workload reaches everything. Removes the "inside the perimeter therefore trusted" assumption.

**When it's the WRONG choice:** As a product security priority when your product's actual risk is injection flaws and unpatched dependencies. Zero trust is an enterprise architecture program with a multi-year timeline; it is not a substitute for fixing your code. Also frequently misapplied as a vendor category rather than an architecture.

### Blast Radius
**Definition:** The full set of systems, data, credentials, and downstream effects reachable through a single compromised component or identity.

**Problem it solves:** Reframes risk conversations away from "how likely is this to be exploited" toward "what happens when it is," which is the question you can actually engineer against.

### Attack Surface
**Definition:** The complete set of points where an untrusted actor can supply input to, or interact with, a system. Includes endpoints, APIs, file uploads, message queues, third-party integrations, admin interfaces, build systems, and increasingly agent tool interfaces and retrieved content.

### Trust Boundary
**Definition:** The line in an architecture where data or control crosses between components with different trust levels. Where trust boundaries are crossed, validation and authorization must occur.

**Problem it solves:** Makes "where do I validate" answerable from a diagram rather than from opinion.

### Assume Breach
**Definition:** A design posture that treats compromise as a given and optimizes for detection, containment, and recovery rather than prevention alone.

**Problem it solves:** Prevention-only programs have no answer for the day prevention fails, which is every program's eventual day.

**When it's the WRONG choice:** As a justification for underinvesting in prevention. "Assume breach" is an additive posture, not a trade.

### Guardrails vs Gates
**Definition:** A guardrail prevents an unsafe action or corrects it automatically without stopping delivery. A gate blocks delivery until a condition is met. Guardrails are policy-as-code, secure defaults, and admission controllers. Gates are blocking pipeline checks and required approvals.

**Problem it solves:** Distinguishes controls that scale from controls that generate queues and exception requests.

**When gates are the WRONG choice:** When the finding class has meaningful false positive rates, when the check is slow enough to affect deploy frequency, or when no one is resourced to unblock teams quickly. A gate you routinely override is a gate that has taught the organization that your controls are negotiable. Gate on high-confidence, high-severity, fast-to-evaluate conditions only: verified secrets, known-malicious packages, signature verification failures.

**When guardrails are the WRONG choice:** For conditions where silent auto-correction hides a real design problem, and for anything a determined developer can trivially remove (a config file in the repo is not an enforcement boundary).

### Security Debt
**Definition:** The accumulated backlog of known, unremediated security issues plus the architectural decisions that make future security work more expensive. Distinct from unknown risk.

**Problem it solves:** Gives you language to negotiate paydown with engineering leadership using terms they already use for technical debt.

**When it's the WRONG choice:** As a framing for actively exploited or externally reachable critical issues. Calling those "debt" invites them to be scheduled. They are incidents in waiting.

### Toil
**Definition:** Manual, repetitive, automatable security work that scales linearly with the number of teams, services, or findings and produces no lasting improvement.

**Problem it solves:** Naming toil is how you get budget to eliminate it. Triage, ticket routing, spreadsheet tracking, and questionnaire answering are the usual culprits in product security.

---

## 2. Core Vocabulary

### Critical Repository
A critical repository is any version-controlled repository of source code which is currently or is planned to be deployed or delivered as part of a production-level release of your company software. This is code which is either directly interacted with by the customer (desktop applications, web applications) or indirectly (database services, provisioning automation, authentication services, etc...). This includes code which is no longer actively maintained but is still in use by at least one customer.

Examples of critical repositories:
- Code which delivers the web front-end of a product
- Services which handle the processing of requests from customer frontend facing services to backend services
- Programming language SDKs and libraries for our products
- IaC for production cloud infrastructure

Examples of non-critical repositories:
- Unit or integration tests
- Example code
- Build scripts or CI/CD configuration files

**2026 caveat:** the "non-critical" list has aged. Build scripts and CI/CD configuration are now a primary attack path (see Poisoned Pipeline Execution, Agentic Supply Chain). Test code frequently holds live credentials. Treat the original list as a criticality tier for *product* impact, not as a scoping decision for *pipeline* controls. Secret scanning and workflow integrity checks belong on every repository.

### Tech Stack
It is your programming framework and languages you use to create your apps, the operating systems or cloud that your apps live on, you pipeline tooling, and anything else you use to build and host your applications and network.

### Mixed Tech Stack
Is the same as tech stack however it means you use multiple pipeline software, different tools, different languages, this makes it very hard for AppSec to manage.

**Program implication:** mixed stacks are the single biggest predictor of program cost. Every additional language, package manager, CI system, and runtime multiplies tool coverage work, rule tuning, and enablement material. Consolidation is a security investment even when it is funded as an engineering efficiency project.

### SDLC (Software Development Life Cycle)
The Software Development Life Cycle (SDLC) refers to a methodology with clearly defined processes for creating high-quality software. In detail, the SDLC methodology focuses on the following phases of software development:
- Requirement analysis
- Planning
- Software design such as architectural design
- Software development
- Testing
- Deployment

### Pipeline
A Pipeline is a set of automated processes and tools that allows developers and operations professionals to collaborate on building and deploying code to a production environment.

### CIA Triad
Confidentiality, integrity, availability

**Confidentiality:** Confidentiality measures are designed to prevent sensitive information from unauthorized access attempts. It is common for data to be categorized according to the amount and type of damage that could be done if it fell into the wrong hands. More or less stringent measures can then be implemented according to those categories.

**Integrity:** Integrity involves maintaining the consistency, accuracy and trustworthiness of data over its entire lifecycle. Data must not be changed in transit, and steps must be taken to ensure data cannot be altered by unauthorized people (for example, in a breach of confidentiality).

**Availability:** Availability means information should be consistently and readily accessible for authorized parties. This involves properly maintaining hardware and technical infrastructure and systems that hold and display the information.

### AAA (Authentication, Authorization, Accounting)
**Authentication** establishes who or what an identity is. **Authorization** determines what that identity may do. **Accounting** (also auditing) records what it did. Most access control defects in production are authorization defects, not authentication defects, which is why Broken Access Control has led the OWASP Top 10 in every recent edition.

### Vulnerability
A weakness in a system that can be exploited to violate a security property. In the product security context, a vulnerability exists in a specific product version, in a specific configuration, and has a specific fix.

### Weakness
The underlying flawed pattern, independent of a specific product. Catalogued as CWEs. A weakness becomes a vulnerability when it is instantiated in a shipped product.

### Threat
An actor or event with the potential to cause harm. Threats are not vulnerabilities; a threat exploits a vulnerability to produce impact.

### Risk
The combination of likelihood and impact for a given threat exploiting a given weakness. Risk is contextual and organizational; severity is a property of the flaw.

### Exploit
Working code or a working technique that turns a vulnerability into a security violation. Exploit availability moves likelihood dramatically and is the primary input to EPSS and KEV.

### Zero Day / N-Day
A **zero day** is a vulnerability with no vendor patch available at the time of exploitation. An **N-day** is a vulnerability with a patch available for N days that remains unpatched in the field. The overwhelming majority of successful exploitation is N-day. Programs that orient entirely around zero-day fear while carrying 200-day N-day exposure are optimizing the wrong variable.

### Bug Class
A category of vulnerability sharing a root cause and therefore a systemic fix. Injection, deserialization, path traversal, IDOR, SSRF. The unit of work for durable improvement, as opposed to the individual finding.

### CWE (Common Weakness Enumeration)
Community catalogue of software and hardware weakness types. Use CWE to group findings by root cause, drive bug-class elimination, and report meaningfully on where defects originate. CWE is the right axis for trend analysis; CVE is not.

### CVE (Common Vulnerabilities and Exposures)
Unique identifier for a specific publicly disclosed vulnerability in a specific product. Assigned by CVE Numbering Authorities.

**Limitation worth knowing:** a growing share of real supply chain attacks (malicious packages, worms, poisoned agent configs) never receive a CVE at all, because there is no vendor product to assign against. A CVE-only view of dependency risk misses the entire malicious-package category.

### CVSS (Common Vulnerability Scoring System)
Standardized severity scoring. CVSS v4.0 added supplemental and threat metric groups intended to address longstanding criticism that Base scores were being used as risk scores.

**Where it goes wrong:** the Base score is a severity measure with no knowledge of your exposure, reachability, data sensitivity, or compensating controls. Driving remediation SLAs from CVSS Base alone produces a backlog sorted by the wrong key.

### EPSS (Exploit Prediction Scoring System)
A data-driven model producing the probability that a given CVE will be exploited in the wild in the next 30 days. Use it to sort a large CVE backlog by likelihood.

**When it's the WRONG choice:** as a sole gate. EPSS is probabilistic and population-level; a low EPSS score on a vulnerability in your internet-facing authentication service is still your problem. Combine with exposure and asset criticality.

### KEV (CISA Known Exploited Vulnerabilities Catalog)
Authoritative list of vulnerabilities with confirmed in-the-wild exploitation. Treat KEV membership as an escalation trigger regardless of CVSS score.

### SSVC (Stakeholder-Specific Vulnerability Categorization)
A decision-tree methodology producing an action (track, track closely, attend, act) from inputs like exploitation status, exposure, automatability, and mission impact, rather than producing a number.

**Problem it solves:** replaces score-based triage with an auditable decision that reflects your context.

**When it's the WRONG choice:** at volume without automation. SSVC requires per-vulnerability judgment on multiple decision points; applying it manually across thousands of findings does not scale. Use it for the top tier and automate the rest.

### VEX (Vulnerability Exploitability eXchange)
A machine-readable assertion about whether a specific vulnerability actually affects a specific product version, with a justification (not affected, affected, fixed, under investigation).

**Problem it solves:** an SBOM plus a CVE feed generates enormous false positive volume for your customers. VEX is how you tell them, at scale and in machine-readable form, that the vulnerable code path is not present or not reachable.

**When it's the WRONG choice:** as a way to avoid fixing things. VEX statements are assertions you are accountable for; "not affected" claims that turn out to be wrong are a trust and, under some regimes, a compliance problem. Also wrong to hand-author. If you cannot generate VEX from build and reachability data, you will not keep it current.

### CSAF (Common Security Advisory Framework)
The machine-readable standard for publishing security advisories, including VEX profiles. The direction of travel for vendor advisories, and the practical delivery mechanism as CRA-style reporting scales up.

### PURL (Package URL) and CPE
Identifiers for software components. **PURL** identifies packages in their native ecosystem (`pkg:npm/lodash@4.17.21`) and works well for open source. **CPE** is the older vendor/product/version scheme used by NVD and works poorly for open source. Mismatches between the two are a major source of both false positives and missed matches in SCA output.

### Crown Jewels
The small set of assets, data stores, and services whose compromise constitutes a material business event. Product security prioritization that does not start from a crown jewels list is prioritizing by scanner volume.

### Inherent vs Residual Risk
**Inherent risk** is the exposure before controls. **Residual risk** is what remains after controls are applied and verified. Risk acceptance decisions must be made on residual risk, which requires evidence the controls actually work, not that they are deployed.

### Compensating Control
An alternative control applied when the primary control cannot be implemented, providing comparable risk reduction.

**Problem it solves:** unblocks delivery when the correct fix has an unacceptable timeline.

**When it's the WRONG choice:** when it is asserted rather than verified, and when it has no expiry. A compensating control without a tracked date to remove it and implement the real fix is a permanent exception with better paperwork.

### Risk Acceptance
A documented decision by an accountable owner to proceed with known residual risk without further mitigation.

**When it's the WRONG choice:** when the accepter is not the person who bears the consequence. An engineering manager cannot accept risk on behalf of the customers whose data is exposed, and under CRA-style regimes cannot accept risk on behalf of the legal entity. Escalate accordingly.

### Exception (Waiver)
A time-bound, documented deviation from a standard with a named owner, a justification, a compensating control, and an expiry date.

**Problem it solves:** exceptions are how a policy survives contact with reality. Without a process, deviation happens silently.

**When it's the WRONG choice:** when exception volume becomes the primary workflow. If most teams hold exceptions to a standard, the standard is wrong. Fix the standard.

### Tactics
Tactics are the actual means to gain an objective.

Tactics = AppSec Activities

### Strategy
Strategy is the overall campaign plan to your objective that includes Complex Operations/Patterns/Activities.

Strategy = Our Goals

---

## 3. Governance, Policy, and Standards

### Policy
Policy is a written document in an organization outlining how to protect the organization from threats, including computer security threats, and how to handle situations when they do occur.

A Policy if broken will result in serious consequences.

### Standard
Standard is "a published specification that establishes a common language, and contains a technical specification or other precise criteria and is designed to be used consistently, as a rule, a guideline, or a definition".

A Standard if broken will result in you having to fix what you broke.

### Guidelines
Guidelines are recommendations to users when specific standards do not apply. Guidelines are designed to streamline certain processes according to what the best practices are.

A Guideline if broken may result in a "hey please try to follow this" but it's not really enforced yet.

### Control
**Definition:** A specific, testable mechanism that reduces risk. Distinct from the policy that requires it and the tool that implements it.

**Problem it solves:** Separating control from tool lets you swap tools without rewriting policy, and lets you answer "is this control effective" independently of "is this tool deployed."

### Security Requirement
**Definition:** A requirement on the product, written in the same backlog and format as functional requirements, expressing a security property that must hold. "All authenticated endpoints enforce object-level authorization against the requesting principal" rather than "do security."

**Problem it solves:** Findings arrive late and are contested. Requirements arrive at the start and are estimated, planned, and tested like everything else.

**When it's the WRONG choice:** As a bulk dump of every ASVS control onto every project. Requirements that are not tailored to the product's risk get treated as boilerplate and copy-pasted as "done."

### OWASP ASVS (Application Security Verification Standard)
**Definition:** A catalogue of testable application security requirements organized into levels, currently at version 5.0 (May 2025). Written to be used for design requirements, verification, and test scoping.

**Problem it solves:** Gives you a defensible, community-maintained answer to "what does secure mean for this application" and "what should this pentest cover," rather than each assessor bringing their own checklist.

**When it's the WRONG choice:** As an awareness or training document. ASVS is a verification standard; handing developers the full list as education produces glazed eyes. Use the Top 10 for awareness and ASVS for requirements and test scope. Also wrong when applied at Level 3 to a low-criticality internal tool.

### OWASP Top 10
**Definition:** An awareness document listing the most critical web application security risk categories, based on CVE and CWE data plus community survey. The 2025 edition (announced November 2025, finalized January 2026) is the first revision since 2021.

The 2025 list:

| ID | Category |
|---|---|
| A01 | Broken Access Control |
| A02 | Security Misconfiguration |
| A03 | Software Supply Chain Failures |
| A04 | Cryptographic Failures |
| A05 | Injection |
| A06 | Insecure Design |
| A07 | Authentication Failures |
| A08 | Software or Data Integrity Failures |
| A09 | Security Logging and Alerting Failures |
| A10 | Mishandling of Exceptional Conditions |

Notable changes from 2021: Software Supply Chain Failures and Mishandling of Exceptional Conditions are new, SSRF was absorbed into Broken Access Control, Security Misconfiguration rose to second, and A09 now covers alerting rather than logging alone.

**Problem it solves:** Shared vocabulary and a starting point for teams with no security program.

**When it's the WRONG choice:** As a program scope, an audit checklist, or a pentest scope. OWASP itself is explicit that the Top 10 is an awareness document, not a standard. A program whose entire definition of coverage is "we scan for the OWASP Top 10" has no coverage story for business logic, authorization design, supply chain integrity, or anything in the other nine sections of this document. Also check what edition your SAST rules, training, and audit checklists are pinned to; a four-year-old category map is still embedded in a lot of tooling.

### OWASP API Security Top 10
**Definition:** The API-specific risk list, led by Broken Object Level Authorization (BOLA) and Broken Object Property Level Authorization, plus broken function-level authorization, unrestricted resource consumption, and unsafe consumption of third-party APIs.

**Problem it solves:** Web-app risk lists systematically under-represent the authorization-per-object problem that dominates real API exploitation.

**When it's the WRONG choice:** As a substitute for an API inventory. You cannot apply an API risk list to endpoints you do not know exist.

### NIST SSDF (SP 800-218)
**Definition:** A framework of secure software development practices organized into four groups: Prepare the Organization (PO), Protect the Software (PS), Produce Well-Secured Software (PW), and Respond to Vulnerabilities (RV). Deliberately non-prescriptive about tools; each practice has tasks, notional implementation examples, and references to SAMM, BSIMM, ASVS, and ISO 27034.

**Status as of mid-2026:** v1.1 remains the authoritative version referenced by federal self-attestation requirements. NIST published the initial public draft of SP 800-218r1 (SSDF v1.2) on December 17, 2025, with the comment window closing January 30, 2026. The draft direction emphasizes continuous practice, reliable delivery, and provable evidence rather than new control categories. Confirm current status before citing v1.2 as a requirement.

**Problem it solves:** It is the common language between software producers and buyers, and the backbone of US federal secure development attestation under EO 14028 and OMB M-22-18 / M-23-16.

**When it's the WRONG choice:** As a maturity model. SSDF tells you what practices should exist, not how good yours are. Pair it with SAMM or BSIMM if you need a maturity assessment. Also wrong as a program design tool for a small team; the full practice set is large and undifferentiated by priority.

### Microsoft SDL (Security Development Lifecycle)
**Definition:** The original lifecycle security model, developed at Microsoft after Code Red and Nimda, and the ancestor of every framework in this section. Establishes security activities at each development phase rather than as a pre-release review.

**Problem it solves:** Historical significance plus a still-useful phase-by-phase activity map.

**When it's the WRONG choice:** As a literal process for a continuous delivery organization. The original SDL assumed long release cycles and formal phase gates. Take the activity taxonomy, not the cadence.

### OWASP SAMM (Software Assurance Maturity Model)
**Definition:** An open, prescriptive maturity model across five business functions (Governance, Design, Implementation, Verification, Operations), each with practices assessed at three maturity levels. Free, self-assessable, with tooling (SAMMY) for tracking.

**Problem it solves:** Gives you a defensible current-state assessment, a target state, and a prioritized roadmap you can take to leadership.

**When it's the WRONG choice:** When you use the assessment score as the goal. Programs optimize for the model and produce activity without outcome. Also wrong to assess annually and change nothing in between; a maturity model with no linked backlog is a scoring exercise.

### BSIMM (Building Security In Maturity Model)
**Definition:** A descriptive model built from observed activities in participating organizations, presented as what firms actually do rather than what they should do, with peer comparison data.

**Problem it solves:** Peer benchmarking. "Firms of our size and vertical do X" is a budget argument SAMM cannot make.

**When it's the WRONG choice:** For most organizations, on cost grounds; it is a commercial engagement. And because it is descriptive, it tells you what your peers do, not what is effective. Common practice is not the same as good practice.

### OWASP DSOMM (DevSecOps Maturity Model)
**Definition:** A maturity model specifically for the DevSecOps toolchain and automation dimensions, more granular than SAMM in the build, deploy, and infrastructure areas.

**Problem it solves:** SAMM is thin on pipeline specifics. DSOMM fills that gap.

### OWASP SPVS (Secure Pipeline Verification Standard)
**Definition:** A verification standard for the security of the software delivery pipeline itself: source control, build systems, artifact handling, deployment automation, and the identities and secrets they use. ASVS in structure and intent, but scoped to the pipeline rather than the application.

**Problem it solves:** ASVS verifies the application. Nothing in the ASVS/Top 10 lineage verifies the system that builds and ships it, which is where SolarWinds-class and PPE-class attacks live.

**When it's the WRONG choice:** As a first move for a team with no pipeline inventory. You verify a pipeline you can enumerate. Also wrong to apply the full standard uniformly across every pipeline; tier by what the pipeline can deploy to.

### ISO/IEC 27034
Application security guidance within the ISO 27000 family. Rarely used as a primary program driver in practice but appears as a reference in SSDF and in some regulated environments.

### ISO/IEC 27001
The information security management system standard. Annex A includes secure development controls. Relevant to product security mainly as the audit regime that will ask for evidence of your SDLC controls.

### ISO/IEC 42001
AI management system standard. Becoming the reference regime for demonstrating AI governance, and increasingly the artifact auditors ask for alongside AI inventories.

### SOC 2
An attestation report on controls relevant to security, availability, processing integrity, confidentiality, and privacy. Type I assesses control design at a point in time; Type II assesses operating effectiveness over a period.

**Problem it solves:** It is the deal-unblocking artifact for B2B SaaS sales in North America.

**When it's the WRONG choice:** As a definition of your security program. SOC 2 scope is negotiated, criteria are broad, and a clean report is compatible with substantial product security gaps. Treat it as a sales requirement you satisfy, not a target state.

### PCI DSS and PCI SSF
**PCI DSS** applies to environments handling cardholder data; Requirement 6 covers secure development and includes explicit expectations around secure coding training, code review, and addressing common vulnerability classes. **PCI SSF** (Software Security Framework) applies to vendors of payment software and includes the Secure Software Lifecycle standard.

**When it's the WRONG choice:** Using PCI scope as your product security scope. PCI drives controls to the cardholder data environment; the rest of your product is out of scope and still your problem.

### EU Cyber Resilience Act (CRA)
**Definition:** Regulation (EU) 2024/2847, the first horizontal EU law imposing mandatory cybersecurity requirements on "products with digital elements" placed on the EU market, across the full product lifecycle. Compliance is demonstrated through conformity assessment, an EU declaration of conformity, and CE marking. Shifts responsibility for product security onto the manufacturer rather than the user.

**Timeline as of July 2026:**
- Entered into force December 2024
- **11 September 2026:** Article 14 reporting obligations apply. Manufacturers must report actively exploited vulnerabilities and severe incidents to ENISA and the relevant national CSIRT via the Single Reporting Platform. Early warning within 24 hours of becoming aware, fuller notification within 72 hours, final report within 14 days for an actively exploited vulnerability (one month for a severe incident). Affected users must also be notified.
- **11 December 2027:** Full application, including design, development, maintenance, conformity assessment, documentation, SBOM, support-period, and CE-marking obligations.

**Problem it solves:** From a practitioner's view, it makes PSIRT capability, vulnerability handling, SBOM generation, and support-lifecycle commitments legally mandatory rather than discretionary, for anyone shipping connected products into the EU.

**When it's the WRONG choice:** Treating CRA as a 2027 problem. The 24-hour reporting clock arrives in September 2026 and requires working intake, triage, decision authority, evidence capture, and legal sign-off paths before that date. Also wrong to scope it to "our EU product"; the test is whether the product is made available on the EU market, and it applies to products already on the market.

### EU AI Act
Risk-tiered regulation of AI systems placed on the EU market, with staged application dates and specific obligations for high-risk systems. Intersects product security through requirements touching accuracy, robustness, cybersecurity, logging, and human oversight. Coordinates awkwardly with CRA and NIS2; a single vendor can be in scope of all three under different theories.

### NIS2
Entity-level cybersecurity risk management and incident reporting obligations for organizations in designated sectors. Distinct from CRA: NIS2 regulates you as an operating entity, CRA regulates your products. A software vendor in a covered sector is subject to both.

### DORA
EU financial-sector operational resilience regulation, with ICT third-party risk and incident reporting provisions. Reaches software vendors primarily through their financial-services customers' contractual flow-down.

### NIST AI RMF
A voluntary framework for managing AI risk across Govern, Map, Measure, Manage functions. The common reference point for AI governance programs in the US, and the mapping target for several agentic security frameworks including AIVSS.

### Software Producer Self-Attestation
**Definition:** The CISA Secure Software Development Attestation Form, through which producers attest that software sold to US federal agencies was developed in conformance with SSDF practices. Required per OMB M-22-18 as amended by M-23-16.

**Problem it solves:** It is the federal market access requirement, and it converts SSDF from a reference framework into a signed statement by a company officer.

**When it's the WRONG choice:** Signing it without the evidence to defend it. Attestation is a representation to the government. The right posture is to build the evidence pack (pipeline records, provenance, testing evidence, vulnerability handling records) and then attest.

### Third-Party Risk Management (TPRM) / Vendor Security Review
**Definition:** Assessment of the security posture of vendors and third-party components before and during use. In a product security context this includes the SDKs, services, and models your product depends on, not just corporate SaaS.

**Problem it solves:** Your product inherits the risk of everything it embeds or calls. Your customers hold you accountable for it regardless of where it came from.

**When it's the WRONG choice:** Questionnaire-driven review as the primary control. A 200-question spreadsheet answered by a vendor's sales engineer produces documentation, not assurance. Prefer technical evidence: SBOM, provenance attestations, pentest summaries, subprocessor lists, and contractual notification commitments.

### Trust Center / Security Portal
**Definition:** A public-facing site publishing your security posture: certifications, subprocessors, architecture summaries, advisory feed, VDP, SBOM availability, and support lifecycle.

**Problem it solves:** Removes the largest source of security toil in B2B software, which is answering the same questionnaire 400 times.

**When it's the WRONG choice:** Publishing a trust center that overstates your posture. It becomes a discovery document in the event of an incident.

### Support Lifecycle / End-of-Support Policy
**Definition:** The published commitment stating how long a product version receives security fixes, what the upgrade path is, and when support ends.

**Problem it solves:** Without it, you are implicitly on the hook to patch every version you ever shipped, and your customers have no basis for planning upgrades. Under CRA, defining and honoring a support period becomes a compliance obligation, not a business preference.

**When it's the WRONG choice:** Publishing a lifecycle you cannot actually honor. A five-year support commitment on a product with a dependency tree full of packages that go unmaintained in eighteen months is a promise that will break in public.

---

## 4. Program Operating Model

### Product Security Program
**Definition:** The organized set of people, services, standards, tooling, and metrics through which product security is delivered. A program has a service catalog, an intake path, defined SLAs, an owner, and a roadmap.

**Problem it solves:** Distinguishes a program from a collection of tools and a shared inbox.

**When it's the WRONG choice:** Formalizing a program before you have demand. At very small scale, a documented program produces overhead without throughput. Two engineers with a paved road and good scanning beat a documented program with no capacity.

### The Partnership Model
The Partnership Model is the process of pairing a security professional with your project team or dev team to which the project/dev team can reach out to the security professional on security related questions, concerns, issues or brainstorm sessions. 1 AppSec professional to 3 projects is recommended.

**Problem it solves:** Turns security from a ticket queue into a relationship, which is the single largest predictor of whether teams engage early.

**When it's the WRONG choice:** At ratios far beyond the stated one-to-three. Assigning one engineer as "partner" to twenty teams produces a name on a wiki page and nothing else, while consuming the credibility you would have had from being honest about coverage. Above the sustainable ratio, switch to a champions model plus self-service.

### Security Champions
**Definition:** Engineers embedded in product teams who hold a part-time security responsibility: local first-line reviewer, threat modeling facilitator, triage triager, and conduit to the security team. Formally recognized, trained, and given time allocation.

**Problem it solves:** It is the only model that scales security context into dozens or hundreds of teams without linear headcount growth.

**When it's the WRONG choice:** When champions are volunteers with no allocated time, no training budget, and no recognition in their performance review. That version reliably decays within two quarters and burns the goodwill of the people who volunteered. Also wrong when the champion becomes the team's designated security owner, letting the rest of the team disengage.

### Centralized vs Embedded vs Federated
**Centralized:** all security staff in one team, serving all products. Consistent, efficient, and distant from product context.
**Embedded:** security engineers report into or sit within product organizations. High context, high influence, prone to inconsistent standards and career isolation.
**Federated:** central team owns standards, tooling, and platform; embedded or champion resources own local execution.

**Practical read:** federated is where most organizations land above roughly 200 engineers. The failure mode of federated is ambiguity about who says no.

### Service Catalog
**Definition:** The published list of what product security offers, what each service requires as input, what it produces, and how long it takes. Threat modeling, design review, pentest coordination, tool onboarding, exception review, incident support.

**Problem it solves:** Makes demand visible and negotiable. Without one, every request is bespoke and every timeline is a surprise.

### Intake and Triage
**Definition:** The single documented path by which requests reach product security, and the process that classifies, prioritizes, and routes them.

**Problem it solves:** Distributed intake through Slack DMs, hallway conversations, and three different Jira projects makes demand invisible and capacity planning impossible.

**When it's the WRONG choice:** A heavyweight intake form as the only path. If the intake process is more expensive than the question, engineers route around it and you lose visibility, which is the opposite of the goal.

### Application Inventory / Asset Inventory
**Definition:** The authoritative record of what software you ship and run, what it is built from, where it is deployed, and who owns it.

**Problem it solves:** Every other control depends on it. Coverage metrics are meaningless without a denominator. Log4Shell response time correlated almost perfectly with inventory quality.

**When it's the WRONG choice:** Building it by survey. Manually collected inventories are stale on delivery. Derive it from systems of record: source control, CI, cloud APIs, artifact registries, service catalogs.

### Ownership
**Definition:** A named team accountable for each service, repository, and finding, maintained in a machine-readable form (CODEOWNERS, service catalog, tags).

**Problem it solves:** Unowned findings are unfixed findings. Ownership routing is the difference between a dashboard and remediation.

### Criticality Tiering
**Definition:** Classification of applications and services into tiers based on data sensitivity, exposure, revenue dependence, and blast radius, with control requirements that differ per tier.

**Problem it solves:** Uniform controls across a portfolio are either too expensive for tier-3 or too weak for tier-1. Tiering is how you allocate finite capacity defensibly.

**When it's the WRONG choice:** Tiering by team preference or by who filled in the form. Tiers assigned without an owned, reviewed rubric become a way for teams to opt out of controls.

### Remediation SLA
**Definition:** The committed maximum time from finding confirmation to remediation, differentiated by severity and asset tier, agreed with engineering leadership rather than declared by security.

**Problem it solves:** Converts "please fix" into a measurable commitment with visible aging.

**When it's the WRONG choice:** SLAs set without engineering agreement and without capacity analysis. A 7-day critical SLA on a team with no capacity produces 100% breach rates, which trains everyone to ignore the metric. Start with an SLA you will meet, then tighten.

### Risk Register
The record of identified, assessed, and accepted risks with owners, treatment decisions, and review dates. In a product security context it should include architectural risks and accepted design debt, not only open findings.

### Build vs Buy
**Definition:** The decision of whether to develop a security capability internally or purchase it.

**Practical framing:** buy where the problem is commodity and the vendor's data advantage is real (vulnerability intelligence, malicious package detection, cloud posture). Build where the problem is your specific workflow, inventory model, ownership graph, or paved road, because that is where vendors cannot know your context.

**When building is the WRONG choice:** When you cannot commit to maintenance, on-call, and documentation for the tool for at least three years. Internally built security tools that lose their author become shadow dependencies with unpatched vulnerabilities.

### Creating Custom Tools
Is when your AppSec team creates their own tools because there aren't any tools that meet their needs. Examples of this would be Libraries, Fuzzing Tools, Customized Input Validation, and Wrapper Libraries. An example of a company would be Netflix, they release lots of AppSec related tooling.

**When it's the WRONG choice:** When an actively maintained open-source project already does 80% of it. The remaining 20% is usually cheaper to contribute upstream than to maintain a fork or a from-scratch replacement.

---

## 5. Security Architecture

### Security Architecture
**Definition:** The discipline of making structural decisions about a system such that required security properties hold by construction. Concerns trust boundaries, isolation, identity and authorization models, cryptographic design, data flow, failure behavior, and the placement of enforcement points.

**Problem it solves:** The most expensive vulnerabilities are architectural. No amount of scanning finds a missing authorization layer or a tenancy model that shares a database without row-level enforcement.

**When it's the WRONG choice:** As an up-front deliverable for every project regardless of size. Architecture review capacity is scarce; spend it on new products, new trust boundaries, new data classifications, authentication and authorization changes, and anything touching tenancy or payments. Not on adding a field to an existing form.

### Design Review / Architecture Review
**Definition:** A structured examination of a proposed design against security requirements and known failure patterns, producing decisions and requirements rather than findings.

**Problem it solves:** It is the cheapest possible point of intervention. A review decision costs hours; the equivalent post-release fix costs sprints.

**When it's the WRONG choice:** As a mandatory blocking gate on every change with a multi-week queue. That design guarantees teams learn to describe changes in ways that avoid triggering review. Use risk-based triggers and publish them so teams can self-select.

### Architecture Decision Record (ADR)
A short document capturing a significant architectural decision, its context, alternatives considered, and consequences. For security purposes, the ADR is where a risk-accepting design decision becomes durable and auditable, rather than living in someone's memory of a meeting.

### Reference Architecture
**Definition:** A pre-approved architectural pattern with security properties already analyzed, that teams can adopt rather than design from scratch.

**Problem it solves:** Removes per-project design review for the common cases.

**When it's the WRONG choice:** When it is not maintained against actual platform capabilities. A reference architecture describing an identity broker you deprecated last year actively misleads teams.

### Secure Design Pattern / Anti-pattern
A **pattern** is a reusable design that produces a desired security property (a policy decision point separated from enforcement points, a single authorization middleware, an outbound proxy for all egress). An **anti-pattern** is a recurring design that reliably produces vulnerabilities (authorization checks duplicated in every handler, secrets passed as environment variables through multiple hops, client-side enforcement of server-side rules).

### Tenant Isolation
**Definition:** The mechanisms preventing one customer's data or compute from being reachable by another in a multi-tenant system. Ranges from separate infrastructure per tenant, through separate databases or schemas, down to row-level enforcement in shared tables.

**Problem it solves:** Cross-tenant data exposure is the single highest-impact vulnerability class in SaaS, and it is almost always an authorization design defect rather than a code defect.

**When weaker isolation is the WRONG choice:** Shared-table isolation with application-layer enforcement is acceptable only when enforcement is centralized, mandatory, and impossible to bypass by writing a new query. If each developer writing a query is responsible for adding the tenant predicate, you will have a cross-tenant bug; it is a matter of when.

### Authorization Model (RBAC / ABAC / ReBAC)
**RBAC** grants permissions by role. Simple, coarse, and prone to role explosion. **ABAC** evaluates attributes of subject, resource, action, and environment. Flexible and harder to reason about. **ReBAC** evaluates relationships between subject and resource ("is the requester a member of the group that owns this document's parent folder"), which fits collaboration and hierarchy products naturally.

**When each is the WRONG choice:** RBAC is wrong when your product's real access rules depend on data relationships; you will end up encoding relationships into role names. ABAC is wrong when you cannot audit effective access, which is most ABAC deployments without tooling. ReBAC is wrong when your permission model is genuinely flat and you are adding a graph database for no reason.

### PDP / PEP (Policy Decision Point / Policy Enforcement Point)
Architectural separation between the component that decides whether an action is allowed (PDP, e.g. an OPA sidecar or an authorization service) and the components that enforce that decision (PEPs, e.g. API gateway, service middleware).

**Problem it solves:** Centralizes authorization logic so it can be tested, audited, and changed once.

**When it's the WRONG choice:** When latency or availability coupling is unacceptable and you have no local caching or fail-mode design. A PDP that is a single point of failure for every request in the product is an availability risk you have introduced in the name of security.

### Key Management / KMS / HSM
**Definition:** The systems and processes for generating, storing, rotating, using, and destroying cryptographic keys. **KMS** provides managed key operations, usually cloud-native. **HSM** provides hardware-backed key isolation with a validated boundary.

**Problem it solves:** Application-managed keys end up in source control, config files, and backups. Managed key services move the key material out of the application's reach entirely.

**When an HSM is the WRONG choice:** When a cloud KMS with hardware backing meets your assurance requirement. Dedicated HSMs bring operational complexity, cost, and availability risk that is only justified by specific regulatory, root-of-trust, or key-custody requirements.

### Crypto Agility
**Definition:** The property of being able to change cryptographic algorithms, key sizes, and protocols without redesigning the system. Requires algorithm identifiers in data formats, versioned key material, and no hardcoded primitives.

**Problem it solves:** Every cryptographic algorithm eventually needs replacing. Post-quantum migration is the current forcing function, and inventory of where crypto lives is the first blocker most organizations hit.

**When it's the WRONG choice:** As a reason to build a pluggable crypto abstraction layer in your application. Most teams should get agility by using a maintained library and a managed KMS, not by writing an algorithm negotiation framework.

### Fail Secure vs Fail Safe
**Fail secure** denies on error (an authorization service timeout results in denial). **Fail safe** (or fail open) permits on error to preserve availability.

**Problem each solves:** Fail secure protects confidentiality and integrity. Fail open protects availability.

**When fail secure is the WRONG choice:** When denial causes safety-relevant or life-relevant harm, or when the control being enforced is advisory rather than security-critical. A WAF in blocking mode failing closed on a health-check timeout can take a product down. Decide per control, document the decision, and test the failure path.

### Secure Defaults
Product ships with the restrictive configuration active. See Secure by Default in Section 1. Architecturally, this means defaults live in code with restrictive values, not in documentation telling customers what to change.

### Deprecation and Removal
**Definition:** The engineering discipline of removing old versions, old endpoints, old auth mechanisms, and old dependencies on a published schedule.

**Problem it solves:** Product security's long-run cost is dominated by what you still support. Every legacy authentication path, unversioned API, and pinned old library is permanent attack surface and permanent patching obligation.

**When it's the WRONG choice:** Deprecating on a security timeline without a customer migration path or a support plan. That produces either an exception that never expires or a customer escalation that reverses your decision publicly.

---

## 6. Threat Modeling and Risk Analysis

### Threat Modeling
Also affectionately referred to as 'evil brainstorming', is the process of figuring out all the threats that your systems face, documenting them, then doing your best to mitigate as many as possible.

**Problem it solves:** It is the only activity that finds design flaws, which no scanner can detect and which are the most expensive class to fix late.

**When it's the WRONG choice:** As a mandatory two-hour workshop on every story. Threat modeling has a real facilitation cost and produces value proportional to how novel the design is. Trigger it on new services, new trust boundaries, new data classifications, authentication and authorization changes, new external integrations, and new agent or model integrations. Skip it for changes inside an already-modeled boundary.

### STRIDE
STRIDE is an acronym that stands for 6 categories of security risks: Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, and Elevation of Privileges.

Reference: https://dev.to/pbnj/demystifying-stride-threat-models-230m

- **Spoofing** refers to the act of posing as someone else (i.e. spoofing a user) or claiming a false identity (i.e. spoofing a process).
- **Tampering** refers to malicious modification of data or processes. Tampering may occur on data in transit, on data at rest, or on processes.
- **Repudiation** refers to the ability of denying that an action or an event has occurred.
- **Information Disclosure** refers to data leaks or data breaches. This could occur on data in transit, data at rest, or even to a process.
- **Denial of Service** refers to causing a service or a network resource to be unavailable to its intended users.
- **Elevation of Privileges** refers to gaining access that one should not have.

**When it's the WRONG choice:** For systems whose primary risks are business logic abuse, privacy, or autonomous behavior. STRIDE's categories map poorly onto "the agent was persuaded to do something no one authorized" and onto privacy harms. Use LINDDUN for privacy, MAESTRO for agentic, and abuse cases for business logic.

### PASTA
Reference: https://versprite.com/security-offerings/appsec/application-threat-modeling/

- Define Business Context of Application
- Technology Enumeration
- Application Decomposition
- Threat Analysis
- Weakness/Vulnerability Identification
- Attack Simulation
- Residual Risk Analysis

PASTA threat modeling combines an attacker perspective of a business with risk and impact analysis to create a complete picture of the threats to products and applications, their vulnerability to attack, and informing decisions about risk and priorities for fixes.

**When it's the WRONG choice:** For routine feature work. PASTA is a seven-stage risk-centric methodology with real effort. Reserve it for flagship products, high-value assets, and situations where you need business-impact-anchored output for an executive audience.

### Attack Trees
Attack trees are hierarchical diagrams describing the security of systems based on attack vector predictions on an asset deemed vulnerable to an attack. In cybersecurity, attack trees are used to outline threats on information systems and possible attacks.

Attack trees are diagrams that depict attacks on a system in tree form. The tree root is the goal for the attack, and the leaves are ways to achieve that goal. Each goal is represented as a separate tree. Thus, the system threat analysis produces a set of attack trees.

**When it's the WRONG choice:** As a broad enumeration technique. Attack trees are excellent for depth on a single high-value goal and terrible for breadth across a system. Combine with STRIDE for coverage.

### LINDDUN
**Definition:** A privacy threat modeling methodology covering Linking, Identifying, Non-repudiation, Detecting, Data Disclosure, Unawareness/Unintervenability, and Non-compliance. Elements of it have been incorporated into ISO and NIST standards.

**Problem it solves:** Security threat modeling systematically misses privacy harms. LINDDUN provides the missing categories.

**When it's the WRONG choice:** As a replacement for security threat modeling. Run it alongside, usually on the same diagram.

### DREAD
A risk rating scheme (Damage, Reproducibility, Exploitability, Affected users, Discoverability) used to prioritize threats.

**When it's the WRONG choice:** Most of the time, in 2026. DREAD's scores are subjective, unreproducible across raters, and have largely been abandoned in favor of CVSS for severity and SSVC or EPSS-plus-context for prioritization. Know it because you will encounter it in older documentation.

### MITRE ATT&CK
A knowledge base of adversary tactics and techniques observed in the wild. Product security uses it primarily for detection engineering coverage and for grounding threat models in techniques that actually occur.

**When it's the WRONG choice:** As an application threat modeling framework. ATT&CK is enterprise-network and endpoint oriented; it does not enumerate application-layer flaws.

### MITRE ATLAS
The ATT&CK analogue for AI systems: adversarial tactics and techniques against machine learning, from reconnaissance through model evasion, extraction, and poisoning.

### CSA MAESTRO
**Definition:** Multi-Agent Environment Security Threat and Risk Operations. A seven-layer threat modeling framework from the Cloud Security Alliance, purpose-built for agentic AI systems, enumerating attack surface across the full agentic stack (foundation model, data, agent framework, deployment infrastructure, evaluation, security and compliance, agent ecosystem). Now at v2 and used as a mapping target by AIVSS and the OWASP Agentic Skills Top 10.

**Problem it solves:** STRIDE and PASTA were built for systems where control flow is determined by code. Agentic systems have goals, memory, tool access, delegation, and emergent behavior, and MAESTRO gives you layers to reason about them.

**When it's the WRONG choice:** For an LLM feature that is a single prompt-and-response with no tools, no memory, and no autonomy. Use the LLM Top 10 there; MAESTRO's layers will be mostly empty.

### AIVSS
**Definition:** An emerging scoring system for agentic AI vulnerabilities, producing a numeric severity that accounts for agentic characteristics CVSS does not model. Public review opened April 2026 with v1.0 targeted before the end of 2026, with mappings to the OWASP Agentic Top 10, MAESTRO, AIUC-1, and NIST AI RMF.

**Status caution:** treat as pre-1.0 as of July 2026. Useful vocabulary and a likely future standard, not yet something to build SLAs on.

### Abuse Case / Misuse Case
**Definition:** A requirement written from the attacker's or bad-faith user's perspective describing what must not be possible. "A user cannot redeem the same promo code across multiple accounts." Distinct from a vulnerability because the code may be working exactly as written.

**Problem it solves:** Business logic abuse is invisible to every scanner and to most pentests scoped by technique rather than by business function. Abuse cases are how it becomes testable.

**When it's the WRONG choice:** Never wrong, but frequently misallocated. Spend abuse case effort on money movement, entitlement, rate-limited resources, and reputation systems, not on CRUD screens.

### Attack Path Analysis
**Definition:** Analysis that chains individually low-severity conditions into a realistic route to a high-value target: an exposed service with a vulnerable dependency, running with an over-permissioned role, that can read a secret granting access to a production data store.

**Problem it solves:** Severity-per-finding triage systematically misses risk that only exists in combination. Attack paths are how you find the "toxic combination" that no single scanner flags.

**When it's the WRONG choice:** As a replacement for fixing the individual conditions. Also wrong when the tooling generating paths cannot distinguish theoretically-reachable from actually-reachable, in which case you get an impressive graph and no prioritization value.

### Continuous Threat Modeling / Threat Model as Code
**Definition:** Maintaining the threat model as a living artifact in the repository, expressed in a machine-readable form and updated with the design rather than produced once as a document.

**Problem it solves:** A threat model produced at design time and never touched again is stale within a quarter and useless within a year.

**When it's the WRONG choice:** When the organization cannot maintain ordinary architecture documentation. Threat-model-as-code adds a dependency on documentation discipline you may not have; a reviewed diagram that is actually current beats a DSL nobody updates.

### Security Exercises
Is essentially a table top exercise where we say "this is happening" and then you have to figure out how and what you would do.

**Problem it solves:** Cheapest way to find gaps in process, authority, and communication before a real incident does.

**When it's the WRONG choice:** As a substitute for testing the technical response path. A tabletop can tell you who to call; it cannot tell you whether your revocation script works.

### Security Simulations
Is when you build an entire simulation that is completely real and then you send in attackers and try to find where your process is broken.

**When it's the WRONG choice:** Before the basics are in place. A full simulation against an organization with no detection capability, no on-call rotation, and no inventory produces a list of things you already knew, at high cost. Sequence tabletops, then purple team, then full simulation.

### Threat Intelligence
**Definition:** Curated information about adversaries, campaigns, exploited vulnerabilities, and malicious infrastructure, applied to prioritization and detection.

**Problem it solves:** In product security specifically, the highest-value forms are exploited-in-the-wild vulnerability feeds (KEV, EPSS), malicious package intelligence, and leaked-credential monitoring.

**When it's the WRONG choice:** Buying broad strategic threat intel feeds for a product security program. Nation-state actor profiles will not change what you fix next sprint. Buy the operational feeds that change decisions.

---

## 7. Secure Development

### Secure SDLC (SSDLC)
**Definition:** The SDLC with security activities, requirements, and verification integrated at each phase rather than appended before release.

**Problem it solves:** Removes the pre-release security review as the single point of security intervention, which is the point at which fixing anything is most expensive and most contested.

**When it's the WRONG choice:** As a document describing a process nobody follows. The most common SSDLC failure is a beautifully diagrammed lifecycle that describes phases the engineering organization does not have.

### Secure Coding Standard
**Definition:** Language- and framework-specific rules for how to write code safely in your environment: which crypto library, which query API, which template engine autoescaping mode, which logging call, which validation approach.

**Problem it solves:** "Validate input" is not actionable. "Use the repository layer's parameterized query methods; direct string concatenation into `db.raw()` is prohibited" is.

**When it's the WRONG choice:** As a generic 60-page document covering languages you do not use. Write short, specific, per-stack standards and enforce them with linters rather than with prose.

### Secure Coding Library / Templates
Secure Coding Library/Templates is a set of code that is developed and maintained by the developer team however it is a central location that the AppSec team can test and verify for security related issues. This provides the devs one code base to manage as well as one code base the AppSec team needs to support.

**When it's the WRONG choice:** When the library duplicates an actively maintained upstream project. You inherit the full maintenance and vulnerability-response burden for code you did not need to write. Wrap and configure upstream libraries rather than reimplementing them.

### Secure Scaffolding
**Definition:** Project templates and generators that produce a new service already wired with authentication middleware, authorization hooks, structured logging, secret retrieval from the vault, dependency pinning, CI security checks, and a populated CODEOWNERS file.

**Problem it solves:** Day-one security posture becomes the default rather than a retrofit. It is the single highest-leverage secure development investment in a growing organization.

**When it's the WRONG choice:** When you cannot keep the scaffolding current. Generated services diverge immediately, so scaffolding fixes never reach already-generated services; pair it with shared libraries or platform components that can be updated centrally.

### Security Reference Materials (including hardening guides)
Security Reference Materials is a location of information that devs and security members can use to learn or use to fix issues found. Often this consists of Books, Videos, Articles, blogs.

**When it's the WRONG choice:** As the answer to a finding. Linking a developer to a 40-page hardening guide in response to a specific vulnerability shifts your work onto them. Provide the specific fix, then the reference.

### Giving Developers Security Tools
Giving Developers Security tools means you are shifting left and giving your devs security related tools however you want to make sure the devs understand the tool, what it does, why it's used, what the output is and lastly how to effectively use this to improve your software security posture.

**When it's the WRONG choice:** Rolling out a tool before tuning it. The first experience a developer has with a security tool sets their expectation permanently. A high-false-positive first run costs you years of adoption.

### Forcing secure coding with IDE tooling
Find an IDE Tooling that can be integrated easily and effectively to provide the developers an easy way to check their code before it's even checked in. This will allow them to build more confidence in their skills/code, and will allow them to learn.

**2026 addition:** the IDE has become a security boundary in its own right, not just a feedback surface. AI coding assistants, agent CLIs, and MCP servers execute in the developer environment with the developer's credentials and repository access. Controls that intercept prompts, file reads, tool calls, and outbound data at the IDE and CLI layer now matter for reasons unrelated to code quality.

**When it's the WRONG choice:** As an enforcement mechanism. Anything in the developer's local environment (config files, rule files, plugin settings, agent instruction files) can be modified, ignored, or removed. Use the IDE for fast feedback and enforce in CI, at the artifact gate, or at admission.

### IDE Tools & Hooks
Are tools that connect to their IDE and add security checks into it. Always work with the developers and provide a list that they think they would find valuable.

### Pre-commit Hooks
**Definition:** Checks that run locally before a commit is created, typically secret scanning, linting, and formatting.

**Problem it solves:** Catches a leaked secret before it enters git history, which is the only point at which the fix is free. Once committed and pushed, the credential must be rotated regardless of whether you rewrite history.

**When it's the WRONG choice:** As your secret scanning control. Hooks are bypassable with `--no-verify` and are not installed on every clone. Run them for developer convenience and run server-side scanning for assurance.

### Secure code review
Is the process of reviewing code manually to find security related vulnerabilities.

**Problem it solves:** Finds the classes automation cannot: authorization logic errors, business logic flaws, trust assumptions, and misuse of otherwise-safe APIs.

**When it's the WRONG choice:** As a coverage strategy. Manual review does not scale to a full codebase or to AI-accelerated commit volume. Target it: authentication and authorization code, cryptographic implementations, deserialization, file handling, payment flows, tenancy enforcement, and anything a threat model flagged.

### Pull Request Review, CODEOWNERS, Branch Protection
**Definition:** The mechanisms that require review before merge (**PR review**), route review to accountable owners (**CODEOWNERS**), and prevent bypass of those requirements (**branch protection**: required reviews, required status checks, no force push, no direct push to default branch, signed commits).

**Problem it solves:** These are the control points that make every other repository-level control non-optional. Without branch protection, required checks are advisory.

**When it's the WRONG choice:** Requiring security-team review on every PR. That is a queue with your name on it and it will collapse. Require owner review universally, require security review by path (auth directories, crypto, IaC for production, CI workflow files, dependency manifests) via CODEOWNERS.

### Unit Tests
Unit testing is a software testing method by which individual units of source code (sets of one or more computer program modules together with associated control data, usage procedures, and operating procedures) are tested to determine whether they are fit for use.

- Pair up with a developer and create security related Unit Tests (Abuse Cases, or Negative Case/Test)
- Use Pentest results as a way to create some unit tests to prevent them from coming back

### Security Regression Testing (with unit tests)
Take already built Regression Tests and convert them into security focused tests that can be run at the same time to make sure your software hasn't exposed already fixed bugs. Give it bad information and see what happens, try to expose bugs and issues.

**Problem it solves:** Fixed vulnerabilities reappear. A regression test is the only control that makes a fix permanent.

**When it's the WRONG choice:** Rarely. The main failure mode is writing the test against the specific payload from the report rather than against the underlying condition, which produces a test that passes while the vulnerability remains exploitable with a different payload.

### Targeting an Entire Bug Class
Is when you focus on an entire bug class (such as XSS or Injection) and get rid of it from your code base. How do we remove a bug class: First you educate your entire team, then we create a plan on how to spot it, prevent it and fix it, then we train our code reviewers, teach our devs not to do it, then create scans and unit tests to find it and lastly we want to measure our progress.

**Problem it solves:** It is the only approach that produces a permanent reduction rather than a temporarily emptied backlog. The strongest form ends with an API or framework change that makes the class unrepresentable.

**When it's the WRONG choice:** When you have an actively exploited critical issue open. Bug class elimination is a quarter-to-year program; it is not incident response. Also wrong for classes with low real-world impact in your context; eliminating a class costs real engineering capacity and should be justified by data from your own findings, not by the class's general reputation.

### Security Sprints
Is when you use an agile sprint to, instead of focusing on features, focus on security related issues for a whole sprint.

**Problem it solves:** Concentrated capacity clears accumulated debt that never wins prioritization against features in normal planning.

**When it's the WRONG choice:** As the primary mechanism for security work. A quarterly security sprint tells the organization that security is a periodic event, and the backlog refills between sprints. Use them to clear debt, not to substitute for continuous work.

### Review New Tech
Set up times and a path for devs to allow the security team to review the new tools or tech that will be added, to avoid potentially adding more vulnerable libraries.

**When it's the WRONG choice:** As a mandatory approval gate on every new dependency. That is unenforceable at modern dependency volumes and pushes adoption underground. Gate on categories that matter (new language runtime, new framework, new authentication library, new AI model or MCP server, anything handling crypto or payments) and use automated policy for the long tail.

### Forcing use of secure packages with proxying to secure package management tools
**Definition:** Routing all dependency resolution through an internal proxy or private registry that mirrors upstream, so that every package your builds consume passes through a control point you own. That control point can enforce allowlists, block known-malicious packages, quarantine new releases for a delay period, prevent dependency confusion by refusing to resolve internal names upstream, cache artifacts so upstream deletion cannot break builds, and record exactly what was pulled.

**Problem it solves:** It is the single most effective structural control against the malicious-package category, which is now the dominant open-source attack pattern and largely invisible to CVE-based SCA. It also solves dependency confusion by construction and gives you a build-time inventory for free.

**When it's the WRONG choice:** When you cannot operate it reliably. A proxy registry is now on the critical path for every build in the company; an outage stops all delivery. It also needs an escape path for legitimate new dependencies that is fast enough that developers do not route around it. Do not deploy it as a security-owned service without platform-team operational ownership.

### Policy as Code
**Definition:** Expressing security and compliance rules in a machine-evaluable language (OPA/Rego, Cedar, Conftest, CI policy engines) so they are applied consistently, versioned, tested, and reviewable.

**Problem it solves:** Rules in prose are interpreted differently by everyone and enforced by no one. Rules in code are applied identically every time and can be tested before rollout.

**When it's the WRONG choice:** When the policy is genuinely judgment-based. Trying to encode "is this architecture appropriate" produces either an unmaintainable rule set or a rule that approves everything. Encode the deterministic subset.

### Feature Flags and Staged Rollout
**Definition:** Deploying code inactive behind a flag and enabling it progressively (canary, percentage rollout, ring deployment) with the ability to disable without a deploy.

**Problem it solves:** Turns a security-relevant change from an all-or-nothing release event into a controlled exposure with a fast off switch. Also the mechanism that makes a secure-default change survivable.

**When it's the WRONG choice:** Flags themselves are attack surface and configuration state. An authorization decision gated on a flag readable or writable by the wrong parties is a vulnerability. Also, permanently-on flags become undocumented forks in the code path; expire them.

### Rollback and Downgrade Protection
**Definition:** The ability to revert to a previous known-good version (**rollback**), combined with controls preventing an attacker or a misconfiguration from forcing installation of an older, vulnerable version (**downgrade protection**), typically via signed version metadata and minimum-version enforcement.

**Problem it solves:** Rollback is your fastest incident containment. Downgrade protection prevents the update mechanism itself from becoming the attack: forcing a client back to a version whose vulnerability you already fixed.

**When it's the WRONG choice:** Rollback is wrong after a data migration that is not backward compatible, and after a security fix whose absence is being actively exploited. Both cases need forward fixes, and both need to be identified before the incident, not during.

---

## 8. Application Security Testing

### Vulnerability Assessment Scans
Is when you need to run a specific tool for a quick scan to get quick results.

### Vulnerability Assessments (Security Assessment)
VAs are long assessments which use a variety of security related tools which test different design and implementations of your software to find vulnerabilities.

### Security Assessment
Is an attempt to find all the vulnerabilities in your application but never exploits them.

### Penetration Test
Is an attempt to find all the vulnerabilities in your application but also exploits them.

**When it's the WRONG choice:** As an annual compliance exercise on a product that changes weekly. A point-in-time pentest on a continuously delivered product is stale before the report is delivered. It is still the right tool for validating a major architectural change, exercising business logic, and satisfying a customer or regulatory requirement. It is the wrong tool for coverage.

### SAST (Static Application Security Testing)
Is a testing methodology that analyzes source code to find security vulnerabilities that make your organization's applications susceptible to attack. SAST scans an application before the code is compiled. It's also known as white box testing.

- It's also known as white box testing
- Performed in a non-runtime environment
- Inspects programming code for flaws/vulnerabilities
- Line by line inspection can be performed
- High level False Positive rate
- Very Slow
- Best to pair with Secure Manual Code Review
- Never send unverified results to a Dev (Always Validate)

**Problem it solves:** Finds injectable sinks, unsafe API usage, and taint-flow issues at the point of authorship, with a line reference the author can act on.

**When it's the WRONG choice:** For authorization defects, business logic, and anything requiring knowledge of runtime configuration or deployment context, which is where most high-impact bugs actually are. Also wrong to deploy full-repository blocking scans on a large codebase; run incremental diff scans in the PR and full scans asynchronously.

### DAST (Dynamic Application Security Testing)
Is a program which communicates with a web application through the web front-end in order to identify potential security vulnerabilities in the web application and architectural weaknesses.

- It performs a black-box test
- Automation (Automatically interacts with your App)
- Has a Spider/Crawler
- Passive Scan (looking at requests such as Security headers)
- Active Scan (Tries to attack your app)

**Problem it solves:** Tests the running system as deployed, so it catches configuration, header, TLS, and stack-level issues that source analysis cannot see, with no false positives about reachability.

**When it's the WRONG choice:** Against modern SPAs and API-first products without authenticated, scripted, spec-driven configuration. An unauthenticated crawler against a React front end tests your login page. Also wrong in a blocking pipeline stage, because runtime scanning is slow and flaky by nature.

### IAST (Interactive Application Security Testing)
IAST solutions help organizations identify and manage security risks associated with vulnerabilities discovered in running web applications using dynamic testing (often referred to as runtime testing) techniques. IAST works through software instrumentation: an agent inside the running application observes data flow, library calls, and request handling during normal test execution, so it reports vulnerabilities with both the runtime evidence of DAST and the code-level precision of SAST.

**Problem it solves:** Very low false positive rate, because the finding is observed rather than inferred, and it reports the exact code location.

**When it's the WRONG choice:** When you have no meaningful automated test suite. IAST only sees code paths that are exercised, so its coverage equals your functional test coverage. Also wrong where the agent's performance overhead or language support is unacceptable, and it is a poor fit for compiled/native and serverless-heavy architectures.

### SCA (Software Composition Analysis)
Is the process of automating visibility into the use of open source software (OSS) for the purpose of risk management, security, and license compliance.

Primary goal is to verify third party components in your software.

**When it's the WRONG choice:** As your only supply chain control. CVE-matching SCA misses the entire malicious-package category, because a typosquatted or hijacked package with a credential stealer in a postinstall script has no CVE. It also over-reports: most CVEs in your dependency tree are in code paths you never call. Pair CVE matching with reachability analysis for prioritization and with malicious package intelligence for the attacks that matter most.

### Secret Scanning
**Definition:** Detection of credentials, tokens, keys, and certificates in source code, commit history, build logs, container images, artifacts, and increasingly in AI prompt traffic. Mature implementations validate detected secrets against the issuing provider to confirm they are live.

**Problem it solves:** Leaked credentials are the highest-yield finding class in the industry: no exploitation skill required, immediate access, and often long-lived and over-scoped.

**When it's the WRONG choice:** Scanning only the current HEAD. History contains the secrets. And detection without a rotation and revocation path is not a control; a found secret that stays valid is still a valid secret. This is one of the two things to put in a pipeline first.

### IaC Scanning
**Definition:** Static analysis of Terraform, CloudFormation, Kubernetes manifests, Helm charts, and similar declarative infrastructure definitions for insecure configuration before it is applied.

**Problem it solves:** Catches public buckets, permissive security groups, unencrypted volumes, and over-broad IAM policies at authorship time rather than in a cloud posture report after deployment.

**When it's the WRONG choice:** As a substitute for runtime cloud posture management. IaC scanning sees intent, not reality: it cannot see resources created by hand, by a different pipeline, or drifted after apply. Run both, and treat divergence between them as its own finding.

### Container Scanning
**Definition:** Analysis of container images for vulnerable OS packages, vulnerable application dependencies, embedded secrets, and insecure configuration (running as root, mounted docker socket, excessive capabilities).

**When it's the WRONG choice:** As a gate at deploy time when the underlying fix requires a base image update you do not control. Fix base images centrally and scan to verify, rather than blocking every team's deploy on a finding they cannot remediate.

### VM & Container VA Scanners
Are extremely useful as your app lives on the servers or containers and making sure they are secure is extremely important.

If your app is living on an insecure infrastructure then you have problems.

### Web Proxy
A proxy server is basically a computer on the internet with its own IP address that your computer knows. The proxy server then makes your web request on your behalf, collects the response from the web server, and forwards you the web page data so you can see the page in your browser.

- Burp Suite can be considered a Proxy
- Manual Testing
- Web Hacker Best Friend

### Fuzzing
Fuzzing or fuzz testing is an automated software testing technique that involves providing invalid, unexpected, or random data as inputs to a computer program. The program is then monitored for exceptions such as crashes, failing built-in code assertions, or potential memory leaks.

- Errors can allow attackers a way in
- Normally in a Web Proxy and in a DAST

**Modern additions:** coverage-guided fuzzing (libFuzzer, AFL++) and continuous fuzzing services (OSS-Fuzz) for native code; structure-aware and grammar-based fuzzing for parsers; and property-based testing as the accessible form for memory-safe languages where the target is logic invariants rather than crashes.

**When it's the WRONG choice:** For business logic and authorization. Fuzzing finds crashes and unhandled conditions; it does not know what your application is supposed to permit. Highest value on parsers, deserializers, file format handlers, protocol implementations, and native code.

### API Tools
Are tools that talk to APIs and do things to the API to find problems such as sending it way too much or not enough, or scripts.

### Malicious Package Detection
**Definition:** Behavioral and heuristic analysis of package releases for install-time scripts, obfuscation, network beaconing, credential access, and typosquatting patterns, independent of CVE data.

**Problem it solves:** The current dominant open-source attack pattern is a malicious or hijacked package, not a vulnerable one. These campaigns receive no CVE during active exploitation and propagate within hours, including self-propagating npm worms that pivot within days of disclosure and payloads that avoid `package.json` entirely by executing through build configuration.

**When it's the WRONG choice:** Never as an addition; frequently mis-scoped when deployed only against direct dependencies. Transitive packages and build-time-only dependencies are the delivery vector.

### Reachability Analysis
**Definition:** Determining whether vulnerable code in a dependency is actually invoked from your application's call graph, and whether the affected component is deployed and exposed. Ranges from static call-graph analysis to runtime-loaded-function observation.

**Problem it solves:** It is the highest-leverage noise reduction available in dependency management, routinely cutting actionable findings by an order of magnitude.

**When it's the WRONG choice:** As a reason to close unreachable findings permanently. Reachability is a function of your current code; the next commit can make an unreachable path reachable. Use it to prioritize, not to dismiss. Also be skeptical of reachability claims for dynamic languages, reflection, and configuration-driven invocation, where static analysis is unreliable.

### ASPM (Application Security Posture Management)
**Definition:** A layer that ingests findings from SAST, SCA, DAST, secrets, IaC, container, and cloud tools, correlates them, deduplicates across sources, attaches ownership and deployment context, and prioritizes by what is deployed, reachable, and exposed. Builds an application and ownership graph rather than a findings list.

**Problem it solves:** Scanners are not the bottleneck; correlation and ownership are. ASPM answers "which open findings are deployed, reachable, and owned by a team that knows about them," which no individual scanner can.

**When it's the WRONG choice:** As a first purchase. ASPM's value is proportional to the number of tools it correlates and the quality of your ownership data. Buying it before you have scanners producing findings and a service catalog identifying owners yields a dashboard over an empty room. Also wrong if you buy it as an ingestion-only product when your actual problem is that your scanners have poor coverage.

### ASOC (Application Security Orchestration and Correlation)
The predecessor category to ASPM, focused on aggregating and deduplicating scanner output and orchestrating scan execution, without the deployment/runtime context layer. ThreadFix and DefectDojo are the canonical open-source examples.

### Red Teaming
Is the activity of hiring an outside service to attack your application and then once they finish their engagement they provide you all the information they have on how they did it. Now this is different than a pentest, because this engagement is much longer than a traditional pentest. You can also hire your own team to do this.

**When it's the WRONG choice:** Before you have detection capability. Red teaming tests your ability to detect and respond, and if the answer is known to be "we cannot," you have paid a lot of money to confirm it. Sequence: fix known issues, build detection, then red team.

### Purple Teaming
**Definition:** Red and blue working together in the open, with the attacking side executing known techniques while the defending side observes, tunes detections, and confirms coverage, iterating in real time.

**Problem it solves:** Delivers the detection-improvement value of red teaming without the cost, duration, and adversarial dynamic. Usually the better first investment.

### Chaos Engineering
Is when you throw monkey wrenches into your applications and make it break and test its ability to recover and understand what will go down, and why. This can show where your application is lacking in performance, availability, and where holes are.

### Security Chaos Engineering
**Definition:** Deliberately injecting security-relevant failures into a running system to verify that controls behave as designed: revoke a credential and confirm the failure mode, disable a WAF rule and confirm detection fires, expire a certificate, remove a network policy, kill the authorization service.

**Problem it solves:** Most control failures are discovered during incidents. This finds them on a Tuesday afternoon with an owner watching.

**When it's the WRONG choice:** In production without a mature reliability practice, blast radius limits, and a tested abort. Start in staging, and start with controls whose failure mode you already believe you understand.

### CTFs
This is an amazing team building exercise that can expose the developer more to the security team and security focused bugs, this can also expose possible security champions. This can show developers how code is broken. You also want to make sure that they are easy enough that your devs can solve them and enjoy the process.

**When it's the WRONG choice:** As your training program. CTF skills are offensive and puzzle-oriented; they transfer imperfectly to writing secure code. Excellent for engagement, champion identification, and building empathy for attackers. Not a substitute for secure coding education tied to your own stack.

### False Positive / False Negative / Triage
A **false positive** is a reported finding that is not a real vulnerability. A **false negative** is a real vulnerability the tool did not report. **Triage** is the process of separating them and enriching what remains.

**The asymmetry that matters:** false positives cost you developer trust, which is your scarcest resource. False negatives cost you breach exposure. Programs that optimize purely for low false positives ship insecure software; programs that ignore false positives lose the ability to ship anything through developers. The resolution is that security triages before developers see output, which is the point of "never send unverified results to a Dev."

### Business Logic Testing
**Definition:** Testing whether the application's rules can be subverted while every individual request is well-formed and authorized: negative quantities, race conditions in balance checks, workflow step skipping, coupon stacking, entitlement escalation through legitimate features.

**Problem it solves:** This is the finding class that no tool detects and that costs real money when exploited.

**When it's the WRONG choice:** Never wrong; usually under-scoped. Requires a tester who understands the business, which is why it is often best done with a product owner in the room and driven from abuse cases rather than from a technique checklist.

### MAST (Mobile Application Security Testing)
Static and dynamic analysis of mobile binaries plus their platform interaction: insecure local storage, certificate pinning bypass, hardcoded secrets in the bundle, insecure IPC, exported components, jailbreak/root detection, and the reality that all client-side controls are advisory. OWASP MASVS and MASTG are the reference standard and testing guide.

### Binary and Firmware Analysis
Analysis where source is unavailable: dependency identification from binaries, hardcoded credential extraction, cryptographic constant identification, and known-vulnerable-library detection in compiled artifacts. Necessary for embedded products, third-party components delivered as binaries, and validating what actually shipped rather than what the manifest claims.

### Test Coverage vs Program Coverage
**Test coverage** is what percentage of a codebase or attack surface a given tool examines. **Program coverage** is what percentage of your product portfolio has each control applied at all. Most programs over-report the first and never measure the second. The more useful question is almost always "how many of our critical repositories have secret scanning, SCA, and branch protection enabled," not "what percentage of lines did SAST parse."

---

## 9. Software Supply Chain Security

### Software Supply Chain
**Definition:** Everything that contributes to your shipped artifact and everything that shipped artifact depends on: first-party source, open-source dependencies (direct and transitive), base images, build tooling, CI plugins and actions, package registries, code signing infrastructure, the machines and identities involved, and increasingly models, prompts, agent frameworks, and MCP servers.

**Problem it solves:** Naming it as a system rather than a list of dependencies is what makes it defensible. The 2025 OWASP Top 10 elevated Software Supply Chain Failures to A03, and the CRA makes component knowledge a legal obligation.

**When it's the WRONG choice:** Nothing to get wrong about the concept; the common error is scoping it to "SCA" and stopping there. Dependencies are one of eight or nine components listed above.

### SBOM (Software Bill of Materials)
**Definition:** A machine-readable inventory of the components in a software artifact, with identifiers, versions, and relationships. Two dominant formats: **SPDX** (ISO standard, strong license lineage) and **CycloneDX** (OWASP, strong security tooling ecosystem, supports VEX and richer dependency graphs).

**Problem it solves:** Answers "am I affected" in hours rather than weeks. Log4Shell response time was almost entirely a function of whether an organization could answer that question. Also the foundational artifact for CRA reporting, federal attestation, and customer assurance.

**When it's the WRONG choice:** As a compliance artifact generated at release and filed. A generated-once SBOM has no operational value. SBOM produces value only when it is generated at build time from the actual build (not inferred from manifests), stored where it can be queried across all versions, and continuously matched against vulnerability and malicious-package intelligence. Also wrong to accept a vendor's SBOM as assurance; an SBOM tells you what is claimed to be inside, not whether the artifact was built from that source.

### AIBOM (AI Bill of Materials)
**Definition:** The extension of SBOM to AI components: models and their versions and sources, agent frameworks, tool definitions, MCP servers, prompt and rule files, embedding models, vector stores, and the AI-related packages that connect them.

**Problem it solves:** You cannot govern what you cannot enumerate, and agentic systems assemble themselves from components that traditional dependency manifests do not list. It is becoming the artifact auditors expect under the EU AI Act and ISO 42001.

**When it's the WRONG choice:** Treating it as a static document. Agents can discover and integrate components at runtime, which means the AI supply chain changes after deployment. An AIBOM generated at build time is incomplete by design; it needs continuous discovery.

### Provenance
**Definition:** Verifiable metadata about how an artifact was produced: what source revision, which builder, what build parameters, which dependencies, at what time.

**Problem it solves:** An SBOM tells you what is claimed to be inside. Provenance tells you whether the artifact you received is the artifact a specific build system produced from a specific source under documented controls. These are different questions and the second one is what SolarWinds-class attacks defeat.

### SLSA (Supply-chain Levels for Software Artifacts)
**Definition:** A framework of increasing build integrity levels, expressed primarily as a Build track. L0 is the absence of SLSA and provides no integrity guarantee. L1 requires automatically generated provenance. L2 requires provenance signed by a hosted build platform. L3 additionally requires that build-signing key material be inaccessible to user-defined build steps, so a compromised build script cannot forge provenance. L4 remains forward-looking.

**Problem it solves:** Gives you a graduated, verifiable answer to build integrity, and a common language for asking suppliers about theirs.

**When it's the WRONG choice:** As an all-repositories target. L3 requires an isolated build platform and meaningfully constrains build flexibility; applying it to internal tools and test harnesses spends budget without reducing risk. Target L3 for artifacts that ship to customers or deploy to production, L1 broadly. Also note SLSA build levels say nothing about the levels of your transitive dependencies, and self-attested levels can be fabricated, so a supplier's claimed level is a claim, not evidence.

### in-toto and Attestation
**Definition:** **in-toto** is the framework for expressing signed claims about artifacts. An **attestation** binds a subject (an artifact and its digest) to a predicate (the claim: SLSA provenance, an SBOM, test results, a review record) inside a signed envelope (DSSE). SLSA provenance is an in-toto attestation with the SLSA predicate type.

**Problem it solves:** Turns "we ran the scan" into a verifiable, machine-checkable statement bound to a specific artifact hash, which is what a policy engine can gate on and what an auditor can verify.

**When it's the WRONG choice:** Generating attestations you never verify. Signing everything and checking nothing is theater with cryptography in it. Build the verification step first, then generate what it verifies.

### Sigstore (Cosign, Fulcio, Rekor)
**Definition:** Keyless artifact signing infrastructure. **Fulcio** issues short-lived certificates bound to an OIDC identity (a GitHub Actions workflow, a Google account), **Cosign** signs and verifies artifacts and attestations, and **Rekor** is the public transparency log recording signatures.

**Problem it solves:** Traditional code signing fails on key management. Sigstore removes long-lived signing keys entirely by tying signatures to workflow identity, which also makes verification policies expressible as "signed by this workflow in this repository."

**When it's the WRONG choice:** For artifacts requiring long-term offline verifiability with a controlled trust root, or in air-gapped environments, where the dependency on public transparency logs and OIDC issuers is a problem. Also note that verifying a Sigstore signature proves who built it, not that what they built is safe.

### Hermetic Build and Reproducible Build
**Hermetic:** the build declares all inputs explicitly and has no network access during execution, so it cannot pull an unpinned dependency mid-build. **Reproducible:** the same source and inputs produce a bit-identical artifact, so an independent party can verify the artifact matches the source.

**Problem they solve:** Hermeticity closes build-time dependency substitution. Reproducibility lets a third party verify a binary corresponds to published source without trusting your build system.

**When they're the WRONG choice:** Both were removed from SLSA v1.0 requirements because they are genuinely hard in practice, and reproducibility in particular can be an enormous engineering project for marginal risk reduction in a normal SaaS context. Hermeticity is usually worth it. Reproducibility is worth it for widely distributed, high-trust software and rarely worth it for a web application.

### Artifact Repository / Internal Proxy Registry
See "Forcing use of secure packages with proxying" in Section 7. Architecturally: an internal registry that mirrors, caches, and gates all external package resolution, and holds all internal artifacts.

### Dependency Confusion
**Definition:** An attack where a package manager resolves an internal package name from a public registry because the public version has a higher version number or because resolution order prefers public sources. The attacker publishes a package with your internal name and receives execution inside your build.

**Problem the fix solves:** Fully preventable structurally. Scope internal packages under a reserved namespace, configure the resolver to never fall through to public sources for internal namespaces, and defensively register internal names publicly.

### Typosquatting and Slopsquatting
**Typosquatting** publishes a malicious package with a name close to a popular one (`reqeusts`, `lodahs`). **Slopsquatting** is the 2025-era variant that registers package names that LLMs hallucinate when generating code, betting that a developer or an agent will install a plausible-sounding package that never existed.

**Problem it solves:** Slopsquatting is a direct consequence of AI-assisted development volume and is a specific reason to route installs through a curated registry rather than trusting a generated `npm install` line.

### Protestware
Maintainer-introduced malicious or disruptive behavior in an otherwise legitimate package, motivated by politics or dissatisfaction rather than profit. Notable because it defeats reputation-based trust: the package was legitimate, the maintainer was legitimate, and the change was authentic.

### Transitive Dependency
A dependency of a dependency. The majority of your dependency tree and the majority of your vulnerability exposure. Notable because you often cannot upgrade it directly; the fix requires the direct dependency's maintainer to move, or an override/resolution directive that you then own the consequences of.

### Pinning and Lockfiles
**Definition:** Recording exact resolved versions and, ideally, content hashes for every dependency, so builds are deterministic and a republished version cannot silently change what you compile.

**Problem it solves:** Without a lockfile, two builds of the same commit can produce different artifacts. With hash pinning, a compromised registry cannot substitute content.

**When it's the WRONG choice:** Pinning without an update mechanism. Pinned dependencies that nobody updates become the EOL exposure that CRA now makes a compliance problem. Pinning is only safe when paired with automated update proposals.

### Dependency Update Automation
**Definition:** Tooling (Dependabot, Renovate, and equivalents) that automatically opens pull requests for dependency updates, optionally grouped, scheduled, and auto-merged when tests pass.

**Problem it solves:** Manual dependency maintenance does not happen. Automation is the difference between a 30-day and a 300-day patch latency.

**When it's the WRONG choice:** Auto-merge on a codebase with weak test coverage; you have automated the introduction of breakage. Also, immediate auto-merge on new releases removes your quarantine window against compromised releases, which is a real trade: a short delay (a few days) on non-security updates materially reduces malicious-package exposure.

### EOL / Unmaintained Dependency
A component no longer receiving security fixes from its maintainer. Under CRA, shipping products containing EOL open source into the EU moves from a risk-management judgment to a compliance question with defined obligations. Maintain an explicit EOL inventory with migration owners and dates; the common failure is discovering the exposure when a CVE lands and no upstream fix is coming.

### Curated / Golden Package Registry
**Definition:** An internal allowlist of vetted, approved package versions that builds may consume, rather than open access to upstream with blocklists.

**Problem it solves:** Blocklists are reactive by construction. An allowlist inverts the default.

**When it's the WRONG choice:** In a fast-moving, polyglot engineering organization without a fast approval path. An allowlist with a two-week onboarding SLA will be circumvented, and circumvention means you now have less visibility than an open proxy would have given you. Allowlist works well for a small number of high-criticality services and poorly as a company-wide default.

### OpenSSF Scorecard
Automated assessment of an open-source project's security practices (branch protection, signed releases, dependency update automation, code review, fuzzing, maintenance activity), producing a score per check.

**Problem it solves:** Gives you a cheap, comparable signal on dependency health for prioritizing which dependencies to reduce reliance on.

**When it's the WRONG choice:** As an intake gate. Scorecard measures process hygiene, not code quality or safety, and it penalizes small well-written libraries maintained by one careful person. Use it as one input to a judgment, not as a pass/fail.

### Build Integrity Monitoring
**Definition:** Runtime monitoring of the build process itself for unexpected behavior: unexpected network connections during build, file writes outside expected paths, process execution not accounted for by the build definition.

**Problem it solves:** Detects the build-time compromise that static analysis of the build definition cannot, because the malicious behavior comes from a dependency's install script rather than from your build config.

### Code Signing / Release Signing
Cryptographic signing of released artifacts so consumers can verify origin and integrity. The critical requirements are that signing key material is inaccessible to build scripts (see SLSA L3), that signing happens on the build platform rather than a developer laptop, and that consumers actually verify. Signing without enforced verification at install or admission is a checkbox.

### Admission Control on Artifacts
**Definition:** A policy enforcement point at deploy time that refuses to run artifacts failing verification: no valid signature, no provenance attestation, provenance not from an approved builder, missing or stale SBOM, known-malicious component present.

**Problem it solves:** This is the point at which every upstream supply chain control becomes enforceable rather than advisory. It is the answer to "what stops someone from deploying an unsigned image."

**When it's the WRONG choice:** Enabling enforcement before you have measured what would be blocked. Roll out in audit mode, fix the legitimate failures, then enforce. Also wrong without a documented, logged, and time-bound break-glass path, because at 3am during an incident someone will need to deploy something.

---

## 10. Pipeline and CI/CD Security

### Adding Security to a Pipeline
Is the process of adding security related tools and checks to the pipeline. Note that you do not want to put all your checks in a Release pipeline and slow it down too much; you want to add these checks possibly before that and stagger them in.

### Creating an asynchronous sec pipeline
This is when you create a separate pipeline that will trigger off an event that makes sense. The goal of this pipeline is to run your super slow and long security tools. You can trigger this off major revisions and this can provide needed information on what is in a certain release and won't affect the release schedule.

**Problem it solves:** Decouples scan duration from delivery speed, which is the single most common reason security tooling gets removed from pipelines.

**When it's the WRONG choice:** For checks whose entire value is preventing the artifact from existing (verified secrets, known-malicious packages, signature verification). Async detection of a leaked credential after deploy still requires rotation. Async is right for slow, high-value, low-urgency analysis; inline is right for fast, high-confidence, high-urgency checks.

### Tools Made for Pipelines
Are tools that are built specifically for a pipeline. These are tools that easily work into a pipeline, that will operate on deltas (what was changed, not the entire code base). There are all kinds of tools that can scan or add checks for all kinds of things. Just make sure it's made for a pipeline.

Top 2 things to add to a pipeline: 1. Secret Scanner, 2. SCA Scanner

### CI/CD Security
**Definition:** Securing the delivery system itself: source control configuration, build platform, runners, workflow definitions, plugins and actions, artifact handling, deployment credentials, and the identities all of it uses.

**Problem it solves:** The pipeline holds production credentials, can modify shipped artifacts, and is frequently the least monitored high-privilege system in the company. It is the most attractive target in the environment and often the softest.

**When it's the WRONG choice:** As a program you start before you have an inventory of pipelines. Most organizations do not know how many CI systems, self-hosted runners, and deployment paths exist. Enumerate first.

### OWASP SPVS
See Section 3. The verification standard for this domain.

### Poisoned Pipeline Execution (PPE)
**Definition:** An attack where the attacker gains code execution in the CI environment by controlling something the pipeline executes. Variants: **direct** (modifying the workflow definition in a branch or PR), **indirect** (modifying a file the pipeline consumes, such as a build script, Makefile, test config, or lint config), and **public** (via a pull request from a fork triggering a workflow with access to secrets).

**Problem the fix solves:** Constrain what triggers privileged workflows. Never grant secrets to workflows triggered by untrusted PRs (`pull_request_target` misuse is the canonical GitHub Actions mistake), require approval for first-time contributors, treat workflow files and any file the pipeline executes as security-sensitive paths with mandatory review, and separate low-privilege CI from privileged deployment.

### Pipeline Identity / OIDC Federation
**Definition:** Having the CI platform exchange a short-lived, workflow-scoped OIDC token for cloud credentials at runtime, instead of storing long-lived cloud access keys as CI secrets. Trust policies can be scoped to a specific repository, branch, and environment.

**Problem it solves:** Eliminates the highest-value standing secret in most organizations. A stolen CI secret is durable access; a stolen OIDC-derived credential expires in minutes and is bound to a workflow identity you can audit.

**When it's the WRONG choice:** When the trust policy is written too broadly. `repo:org/*` with no branch or environment condition converts a per-workflow credential into an org-wide one, which is most of the risk back. Also not available in every CI/cloud combination, and self-hosted CI requires you to run the OIDC issuer.

### Self-hosted Runner Risk
**Definition:** Self-hosted CI runners are persistent machines executing untrusted code from repositories. Risks: state persisting between jobs (caches, credentials, cloned source), network position inside your infrastructure, and reuse across repositories with different trust levels.

**Problem the fix solves:** Use ephemeral runners that are destroyed after each job, isolate runners by trust tier, never expose self-hosted runners to public repository workflows, and place them in a network segment with no path to production.

### Ephemeral Runner
A build executor created for a single job and destroyed afterward. Prevents cross-job contamination and removes persistence as an attacker objective. Now the default expectation for any privileged build.

### Action / Plugin Pinning
**Definition:** Referencing third-party CI actions and plugins by immutable commit SHA rather than by mutable tag or branch.

**Problem it solves:** A tag can be moved. `uses: someorg/action@v3` executes whatever `v3` points to today, which the action's maintainer or a maintainer-account compromise can change under you. This has been the delivery mechanism for real, large-scale CI compromises.

**When it's the WRONG choice:** Not wrong, but incomplete without update automation, since SHA-pinned actions never receive security fixes on their own. Pin plus automate.

### Secrets in CI
**Definition:** Credentials made available to build and deploy jobs. The main controls: prefer OIDC federation over stored secrets; scope secrets to specific environments rather than repository-wide; never expose secrets to workflows triggered by untrusted input; mask them in logs but do not rely on masking; and rotate on any suspicion.

**When storing secrets in CI is the WRONG choice:** For anything with production write access, where federation is available. And always for anything that could equally be fetched at runtime by the workload from a secrets manager, which keeps the credential out of the build environment entirely.

### Environment Separation
**Definition:** Hard separation between development, staging, and production credentials, data, networks, and deployment paths, such that access to one does not confer access to another.

**Problem it solves:** It is the control that turns a development mistake into a development incident. Most catastrophic automation failures (including agent-caused ones) involve a non-production process holding production access.

**When it's the WRONG choice:** Never wrong in principle; the practical failure is separating credentials while leaving a shared network path, a shared identity provider with a permissive role, or production data copied into staging for testing.

### Break-Glass / Emergency Bypass
**Definition:** A documented, authenticated, logged, alerting, time-bound path to bypass a control during an emergency.

**Problem it solves:** Every gate needs one, or the gate gets removed permanently the first time it blocks an incident response.

**When it's the WRONG choice:** When it is not instrumented. An unlogged bypass mechanism is not break-glass, it is a backdoor with an approved name. The test is whether use of it generates an alert and a review.

### GitOps
**Definition:** Declaring desired infrastructure and application state in git and having a reconciler continuously apply it, so git is the only path to change production.

**Problem it solves:** Every production change becomes a reviewed, attributed, revertible commit. The audit trail is a byproduct rather than a separate system, and drift is detected and corrected automatically.

**When it's the WRONG choice:** GitOps concentrates all production authority into repository write access and reconciler permissions. Without strong branch protection, signed commits, and a tightly scoped reconciler identity, you have built a single high-value target with a convenient API. Also poor fit for genuinely imperative operations and emergency response, which need a separate documented path.

### Drift Detection
Comparing running infrastructure state to its declared definition and flagging divergence. Necessary because IaC scanning validates intent, and hand-applied changes, emergency fixes, and out-of-band automation create reality that intent does not describe.

### Deployment Gate vs Advisory Check
A **gate** blocks promotion; an **advisory check** reports and continues. The decision framework is in "Guardrails vs Gates" (Section 1). In pipelines specifically: gate on verified secrets, malicious packages, failed signature or provenance verification, and critical findings on internet-facing tier-1 services. Advise on everything else and track the aging.

---

## 11. Cloud, Infrastructure, and Runtime

### Infrastructure as Code (IaC)
**Definition:** Defining infrastructure in declarative machine-readable files (Terraform, CloudFormation, Pulumi, Kubernetes manifests, Helm) that are version-controlled and applied by automation.

**Problem it solves:** Infrastructure becomes reviewable, testable, and scannable before it exists, and reproducible afterward.

**When it's the WRONG choice:** IaC does not reduce risk by itself; it relocates it. The Terraform state file and the apply credential become among the highest-value assets in the environment, and a misconfiguration is now replicated identically across every environment instantly.

### CSPM (Cloud Security Posture Management)
Continuous assessment of cloud configuration against benchmarks and policy: public storage, permissive network rules, disabled logging, unencrypted resources, missing MFA. Catches what IaC scanning cannot: reality, including drift and out-of-band changes.

### CIEM (Cloud Infrastructure Entitlement Management)
Analysis of cloud identity and permission relationships to find over-permissioned roles, unused permissions, privilege escalation paths, and cross-account trust that should not exist. The control that makes least privilege achievable in cloud, because it supplies the usage data that makes permission reduction safe.

### CWPP (Cloud Workload Protection Platform)
Runtime protection for workloads: vulnerability state of running containers and hosts, process and file integrity monitoring, network behavior, and exploit detection at the workload layer.

### CNAPP (Cloud-Native Application Protection Platform)
**Definition:** The consolidated category combining CSPM, CIEM, CWPP, container scanning, and IaC scanning behind a single graph, with attack path analysis across them.

**Problem it solves:** Individual cloud security tools each report a fragment. The value is in correlation: a vulnerable package, on an internet-exposed workload, with a role that can read a production database, is a finding no single tool produces.

**When it's the WRONG choice:** As a replacement for ASPM. CNAPP works cloud-inward and governs workload and configuration posture; ASPM works code-outward and governs SDLC and application risk. They overlap on containers and IaC and are complementary elsewhere. Buying one expecting the other's coverage is the most common purchasing error in this space.

### KSPM / Kubernetes Security
Posture management for Kubernetes specifically: RBAC analysis, privileged pod detection, network policy coverage, admission controller configuration, secret handling, and the control plane itself. Worth separating from general CSPM because Kubernetes has its own identity model, its own network model, and its own set of trivially-exploitable defaults.

### Container Hardening
**Definition:** Reducing container attack surface: minimal or distroless base images, running as a non-root user, read-only root filesystem, dropped Linux capabilities, no shell, no package manager, and multi-stage builds so build tooling does not ship.

**Problem it solves:** Most container CVE volume is in OS packages you never invoke. A distroless image removes hundreds of findings by removing the software, which is a better fix than patching it. It also removes the tooling an attacker needs post-exploitation.

**When it's the WRONG choice:** Where operational debuggability genuinely requires in-container tooling and your organization has no ephemeral debug container workflow. Also a poor first move for teams with no base image pipeline; centralize base images first, harden second.

### Admission Controller
**Definition:** A Kubernetes (or equivalent platform) hook that validates or mutates resources at creation, enforcing policy before workloads run: image signature verification, no privileged containers, required labels and owners, resource limits, no host mounts.

**Problem it solves:** The only enforcement point in a cluster that cannot be bypassed by a team's own manifests. Every other Kubernetes control is advisory.

**When it's the WRONG choice:** Deployed in enforce mode without audit-mode measurement first, and deployed without a highly available webhook: a failing admission webhook set to fail-closed will take the cluster's ability to schedule workloads with it.

### Sandboxing / Isolation
**Definition:** Executing untrusted or semi-trusted code inside a constrained boundary: gVisor, Firecracker or other microVMs, seccomp and AppArmor profiles, WASM runtimes, or full VM isolation.

**Problem it solves:** Containers share a kernel. When you execute genuinely untrusted code (customer-supplied code, agent-generated code, plugins, build scripts from forks), container isolation is not a security boundary and stronger isolation is required.

**When it's the WRONG choice:** For first-party workloads where the isolation cost (performance, syscall compatibility, operational complexity) buys little. Reserve strong isolation for genuine multi-tenant code execution and agent execution.

### Egress Control
**Definition:** Deny-by-default outbound network policy, with explicit allowlists per workload, usually via an egress proxy or network policy.

**Problem it solves:** It is the control that breaks the exploitation chain rather than the exploitation. Data exfiltration, C2 callback, second-stage download, and credential-theft-to-attacker-endpoint all require egress. It is also the primary containment control for both code-execution vulnerabilities and agent code execution.

**When it's the WRONG choice:** Never conceptually; frequently deferred because the allowlist work is real. The practical failure is allowlisting broad CDN and cloud API ranges, which restores exfiltration capability through the allowed path. Prefer proxy-level allowlists by hostname over IP-range allowlists.

### Service Mesh / mTLS
Mutual TLS between services with identity-based authorization, usually provided by a mesh sidecar or ambient layer. Provides workload identity, encryption in transit, and per-service authorization independent of network position.

**When it's the WRONG choice:** In an environment with fewer than a few dozen services, or without a platform team to operate it. A service mesh is a significant operational commitment and adds a new failure domain in the request path; small architectures get most of the value from TLS plus a gateway plus workload identity without the mesh.

### WAF (Web Application Firewall)
A WAF (Web Application Firewall) helps protect web applications by filtering and monitoring HTTP traffic between a web application and the Internet.

Alert mode just says "this is bad", and block mode will block it.

**When it's the WRONG choice:** As a fix. A WAF rule closing an exploitable vulnerability is a mitigation with an expiry date, and treating it as remediation leaves you exposed to any bypass. Its legitimate uses are buying time during patch development, virtual patching for versions you cannot update, blunting automated scanning and volumetric attacks, and providing detection signal. Running in block mode without a tuning period on a business-critical app is how you cause an outage and lose the right to run a WAF at all.

### Adding shield for your app (WAF/RASP)
Essentially you add a WAF/RASP to provide an additional layer of security to your application stack to improve your security posture. This is often called Defense in Depth. This is supposed to be an additional control and not a fix to what these tools can protect.

### RASP (Runtime Application Self-Protection)
**Definition:** Instrumentation inside the running application that observes execution context and can block exploitation attempts at the point of the dangerous operation, rather than at the network edge.

**Problem it solves:** Has application context a WAF lacks: it can see that this specific string reached a SQL execution sink, rather than guessing from request patterns, which means far lower false positives.

**When it's the WRONG choice:** Where the agent's performance or stability overhead is unacceptable, where language support is weak, and as a substitute for fixing the code. Also a poor fit for high-throughput latency-sensitive services and for architectures where the vulnerability is in a component the agent cannot instrument.

### ADR (Application Detection and Response)
The emerging category of runtime application-layer detection and response, applying EDR-style behavioral detection inside the application and its runtime rather than at the endpoint or network. Overlaps heavily with RASP and with runtime CWPP; the differentiator claimed is detection and investigation rather than inline blocking. Treat as an evolving category as of 2026 rather than a settled one.

### Golden Image
A hardened, patched, scanned, approved base image or machine image that all workloads derive from, rebuilt on a schedule and on security triggers.

**Problem it solves:** Centralizes OS-layer patching. One image update fixes hundreds of services, which is the only version of container CVE remediation that works at scale.

**When it's the WRONG choice:** When rebuild cadence is slow. A golden image rebuilt quarterly is a golden image with a quarter of accumulated CVEs, and teams will pin to it. Weekly rebuild is the minimum for it to be a security control rather than a bottleneck.

### Hardening Guides / CIS Benchmarks
Prescriptive configuration baselines for operating systems, containers, Kubernetes, databases, and cloud services.

**When it's the WRONG choice:** Applying a full benchmark at Level 2 to production systems without testing. CIS benchmarks include controls that break real workloads, and blanket application produces outages that get the whole effort reverted. Select applicable controls, test, and encode the result in your golden image rather than remediating per-host.

### Security Logging and Audit Log
**Definition:** Recording security-relevant events with enough context to investigate: authentication attempts, authorization decisions and denials, privilege changes, data access to sensitive resources, configuration changes, and administrative actions. Includes who, what, when, from where, and the outcome.

**Problem it solves:** Without it you cannot answer the two questions every incident asks: what did they access, and when did it start. The 2025 OWASP Top 10 renamed A09 to include alerting specifically because logging without alerting has been the recurring gap.

**When it's the WRONG choice:** Logging is wrong when it captures secrets, tokens, full request bodies with PII, or session identifiers, at which point the log store becomes a higher-value target than the application. Redact at emission, not at ingestion.

### Tamper-Evident Logging
Audit logs written to append-only or cryptographically chained storage that an attacker with application-level access cannot rewrite. Necessary because the first thing a competent attacker does is clean up, and necessary for agent action logs specifically, where the record of what the agent was shown versus what it executed is the evidence.

### Customer-Facing Audit Log
An audit log exposed to your customers so they can investigate activity in their own tenant. A product feature with security requirements: it must be tenant-scoped, tamper-evident, complete for security-relevant events, and retained for a documented period. Increasingly a procurement requirement and, in regulated contexts, a compliance one.

### Detection Engineering for Product
**Definition:** Building and maintaining detections for attacks against your product and its infrastructure, driven by your threat model and validated against real technique execution.

**Problem it solves:** Product security programs that stop at prevention have no answer for the exploitation of a vulnerability they did not know about. Detection is the control that covers unknown vulnerabilities.

**When it's the WRONG choice:** Writing detections before you have the telemetry to write them against. Log coverage precedes detection engineering, and detection you cannot validate is an assumption.

---

## 12. Identity, Access, and Secrets

### Human Identity
An identity representing a person: an employee, contractor, or customer. Governed by joiner-mover-leaver processes, MFA, and access review, with a directory as the system of record.

### Machine NHI (Non-Human Identity)
**Definition:** An identity representing a workload, service, pipeline, script, or device rather than a person. Service accounts, workload identities, API keys, pipeline identities, device certificates.

**Problem it solves:** Naming them as a distinct class is what makes them governable. They outnumber human identities substantially in most environments, are frequently created outside any lifecycle process, hold broader permissions than any human, have no owner after the creator leaves, and rarely get reviewed or revoked.

**When treating them like human identities is the WRONG choice:** Human identity controls assume a human: MFA, interactive login, periodic re-attestation, and offboarding on termination. None of those apply. NHI governance requires ownership metadata, expiry by default, credential rotation automation, and usage-based right-sizing instead.

### AI NHI
**Definition:** An identity representing an AI agent or model-driven process: an autonomous agent with tool access, a coding assistant acting on a repository, an orchestrator spawning sub-agents.

**Problem it solves:** AI identities behave unlike both humans and traditional workloads. They act with delegated human authority, their action sequences are non-deterministic, they can spawn additional identities, they can be socially engineered through content they read, and they operate at machine speed with human-level breadth. Governing them as either humans or service accounts fails in both directions.

**When it's the WRONG choice:** Never as a distinction; the common failure is not making it. Agents borrowing a human's credentials or sharing a service account is the dominant pattern in production today, and it makes agent actions both unattributable and unrevocable without breaking the human or the whole service.

### Service Account
A non-human identity created within an application or platform for a workload's use. The problem class: created ad hoc, credentialed with a long-lived static secret, shared across multiple workloads, over-permissioned because scoping was harder than granting broadly, and orphaned when the creator leaves.

### Workload Identity / SPIFFE
**Definition:** Cryptographic identity issued to a workload based on verifiable attributes of where and how it is running, rather than on a possessed secret. **SPIFFE** defines the identity format (SPIFFE ID) and issuance API; SPIRE is the reference implementation. Cloud-native workload identity (IRSA, workload identity federation, managed identities) is the same pattern.

**Problem it solves:** Removes the bootstrap secret problem entirely. There is no static credential to leak, rotate, or find in a git repository.

**When it's the WRONG choice:** For workloads outside a platform that can attest them, and for integrations with third parties who only accept a static key. Both require secrets management, so plan for a hybrid rather than assuming universal coverage.

### Secrets Management
**Definition:** Centralized generation, storage, distribution, rotation, and revocation of credentials, with access control, audit logging, and short-lived dynamic credentials where the backend supports it.

**Problem it solves:** Removes secrets from source, config files, environment variables baked into images, wikis, and CI variables, and gives you a single place to revoke.

**When it's the WRONG choice:** Deployed as a storage location without rotation. A vault holding the same static credentials for three years has centralized the secrets and changed nothing about their risk. The value is in dynamic, short-lived credentials and in the ability to revoke centrally, not in the storage.

### Short-Lived Credentials
**Definition:** Credentials with lifetimes measured in minutes to hours, issued on demand and scoped to the task.

**Problem it solves:** It caps the value of theft. A stolen credential with a fifteen-minute lifetime is a much smaller problem than one that has been valid for two years. This is the single most effective mitigation for the agent identity abuse category.

**When it's the WRONG choice:** Where the consuming system cannot refresh, and where the issuance service's availability becomes a hard dependency for critical operations without a fallback. Both are engineering problems worth solving, but they must be solved before the rollout.

### Long-Lived Token Risk
Personal access tokens, API keys, and static service credentials with no expiry and broad scope. They are the mechanism by which a minor compromise becomes a major breach: a goal-hijacked agent with a read-only scope is an incident report, and the same hijack with a broadly scoped PAT is repository exfiltration.

### Just-in-Time (JIT) Access
**Definition:** Granting elevated access only for the duration of an approved task, with automatic expiry, rather than maintaining standing privilege.

**Problem it solves:** Eliminates standing privilege, which is the thing an attacker actually harvests.

**When it's the WRONG choice:** For break-glass paths that must work when the JIT system is down, and where request latency would delay incident response. Both need a documented, monitored standing path.

### Access Review / Recertification
Periodic verification that granted access is still required, by an owner who knows. Extend to NHIs and agents on the same cadence as humans; the fact that nobody does this is why service account permissions only ever accumulate.

### Delegated Authority / On-Behalf-Of
**Definition:** A pattern where a service or agent acts using the authority of a user rather than its own, with the request carrying the original principal's identity and scope.

**Problem it solves:** Preserves attribution and per-user authorization through a chain of services, instead of every downstream call being made by an over-permissioned service identity that can see everything.

**When it's the WRONG choice:** When the delegation chain is unbounded. An agent that receives on-behalf-of authority and passes it to sub-agents, tools, and further services without scope reduction has converted a scoped user token into a general-purpose one. Reduce scope at each hop and cap chain depth.

### Consent and Scope
The mechanism by which a principal authorizes a specific set of permissions to a specific client. Product security concerns: are scopes granular enough to be meaningful, is consent informed (does the user understand what "read all your files" means), is consent revocable, and does the product enforce the granted scope server-side rather than trusting the client.

---

## 13. API and Interface Security

### API Security
**Definition:** Securing programmatic interfaces: authentication, per-object and per-function authorization, input and schema validation, resource consumption limits, and safe consumption of third-party APIs.

**Problem it solves:** APIs are now the majority of most products' attack surface, and they lack the incidental protections a web UI has: no browser same-origin policy, no CSRF token pattern, and clients that attackers fully control.

**When it's the WRONG choice:** Nothing to get wrong about the concept. The common failure is treating it as a gateway configuration problem when the dominant vulnerability class (object-level authorization) can only be enforced in the application, because only the application knows who owns the object.

### API Inventory
**Definition:** The authoritative list of every API endpoint your product exposes, including version, authentication requirement, exposure, owner, and data sensitivity, derived from specifications, gateway configuration, code analysis, and observed traffic.

**Problem it solves:** Nothing else in this section is possible without it. Every API security program that fails, fails here first.

### Shadow API / Zombie API
A **shadow API** is an undocumented endpoint not in the inventory or the spec: debug routes, internal endpoints reachable externally, endpoints added without spec updates. A **zombie API** is a deprecated version left running after its replacement shipped, typically unmaintained and unmonitored while still serving production traffic.

**Problem the fix solves:** Both classes are excluded from your testing, monitoring, and patching, which is precisely why they are exploited. Derive inventory from observed traffic, not only from specifications, so that shadow endpoints surface.

### BOLA (Broken Object Level Authorization)
**Definition:** The API fails to verify that the authenticated caller is entitled to the specific object identified in the request. `GET /api/invoices/12345` returns the invoice regardless of who owns it.

**Problem the fix solves:** This is the top API vulnerability in practice and the most common cause of cross-tenant data exposure. The fix is architectural: authorization enforced in a mandatory layer keyed on the requesting principal and the object's owner, not a check each developer remembers to add.

**When per-endpoint manual checks are the WRONG choice:** Always, at scale. If correctness depends on every developer adding a predicate to every query, you will have a BOLA. Enforce in the data access layer or a policy layer that cannot be bypassed.

### BFLA (Broken Function Level Authorization)
The API fails to verify the caller is entitled to invoke the operation at all: a standard user reaching an administrative endpoint by changing the method or path. Distinct from BOLA, which concerns which object; BFLA concerns which action.

### Mass Assignment / Object Property Level Authorization
Binding client-supplied fields directly to internal objects, allowing a caller to set fields they should not control (`isAdmin`, `accountBalance`, `tenantId`, `role`). Fix with explicit allowlisted input schemas per endpoint rather than whole-object binding, and with server-side authorization on writes to sensitive properties.

### Rate Limiting and Quota
**Definition:** Limits on request volume, computational cost, and resource consumption per principal, per endpoint, and per tenant.

**Problem it solves:** Prevents brute force, enumeration, scraping, denial of wallet on metered backends, and unrestricted resource consumption. Increasingly a cost control as much as a security control, particularly for endpoints that invoke models.

**When it's the WRONG choice:** Applied only at the gateway on unauthenticated traffic. Expensive operations behind authentication need per-principal limits too, and a single tenant can otherwise exhaust shared capacity for everyone.

### Schema Validation / Contract Testing
Validating requests and responses against a declared schema (OpenAPI, JSON Schema, protobuf) at the boundary, and testing in CI that implementation matches the contract.

**Problem it solves:** Turns input validation into a declarative, generated, comprehensive control rather than a per-handler manual effort, and prevents the spec-versus-reality drift that produces shadow endpoints.

### GraphQL-Specific Risk
Introspection exposing the full schema, deeply nested or cyclic queries as a resource exhaustion vector, batching enabling brute force in a single request, aliasing bypassing naive rate limits, and per-field authorization being genuinely harder than per-endpoint. Controls: disable introspection in production, enforce query depth and complexity limits, cost-based rate limiting rather than request counting, and per-resolver authorization.

### Webhook Security
Both directions. **Outbound:** sign payloads with a shared secret or asymmetric key and include a timestamp so receivers can verify origin and prevent replay; support secret rotation. **Inbound:** verify signatures before processing, and treat webhook receivers as SSRF-adjacent surfaces since they accept attacker-influenced URLs in configuration.

### Gateway vs In-App Enforcement
The gateway is the right place for authentication, TLS termination, coarse rate limiting, schema validation, and routing. It is the wrong place for object-level authorization and business rules, because it lacks the data. Controls placed at the gateway that require application data will be either wrong or bypassable, and a gateway is bypassable by anything that can reach the service directly.

---

## 14. Vulnerability Management and Prioritization

### Vulnerability Management
Vulnerability Management describes the ongoing process of identifying, classifying, and remediating security holes.

Each step of the process is part of a continuous cycle focused on improving security in a computer, network, or communications infrastructure.

Vulnerability Management reduces the likelihood that flaws in code or design compromise an organization's overall security posture. It proactively seeks out exposures and evaluates risks by finding, prioritizing, and mitigating security vulnerabilities.

Dump data into ThreadFix or DefectDojo.

**Modern note:** the aggregation layer described here has become the ASPM category (Section 8). ThreadFix and DefectDojo remain viable, particularly DefectDojo for teams wanting an open-source correlation layer, but the current generation adds deployment and reachability context that changes prioritization rather than just deduplicating.

### Findings Correlation and Deduplication
**Definition:** Recognizing that the same underlying issue reported by SAST, SCA, a container scan, and a pentest is one finding, and consolidating to a single root cause with a single owner and a single fix.

**Problem it solves:** Duplicate volume is the primary driver of both triage cost and developer distrust. A team that receives the same vulnerability from four tools concludes security tooling is noise.

### Risk-Based Prioritization
**Definition:** Ordering remediation by a composite of severity, exploitation likelihood (EPSS, KEV), reachability, exposure, asset criticality, data sensitivity, and compensating controls, rather than by CVSS alone.

**Problem it solves:** A CVSS-sorted backlog puts an unreachable critical in a nightly batch job ahead of a reachable high in your authentication service. It sorts by the wrong key.

**When it's the WRONG choice:** When the model is opaque to the engineers receiving the output. A proprietary risk score that developers cannot interrogate produces the same trust problem as raw CVSS, with less transparency. Publish the inputs.

### MTTR / MTTD
**Mean time to remediate** is elapsed time from confirmation to fix deployed. **Mean time to detect** is from introduction to discovery.

**When they're the WRONG metric:** Means are distorted by outliers and by closing easy findings. Prefer median and 90th percentile, segmented by severity and asset tier, plus the count and age of the oldest open critical items. "Median MTTR improved" alongside a 400-day-old open critical is a reporting failure.

### Remediation vs Mitigation vs Acceptance
**Remediation** removes the vulnerability (patch, code fix, remove the component). **Mitigation** reduces exploitability or impact without removing it (WAF rule, network restriction, feature disable, permission reduction). **Acceptance** is a documented decision to carry the residual risk.

**Problem the distinction solves:** Tracking these as one status is how mitigations quietly become permanent. Track them separately, and give every mitigation an expiry date and a remediation ticket.

### Campaign-Based Remediation
**Definition:** Treating a class of findings across the whole portfolio as a single time-bounded project with an owner, a communication plan, a dashboard, and a deadline, rather than as thousands of individual tickets.

**Problem it solves:** Individual tickets distributed across 80 teams get deprioritized individually. A campaign creates visibility, peer pressure, and a shared fix path (usually a base image, a shared library version, or an automated PR).

**When it's the WRONG choice:** For findings without a common fix. A campaign requires a single remediation action most teams can take; if each team needs bespoke work, you have a coordination burden without the leverage.

### Automated Remediation
**Definition:** Machine-generated fixes: dependency upgrade PRs, IaC configuration corrections, and increasingly AI-generated code patches with test validation.

**Problem it solves:** The remediation bottleneck is engineering capacity, not finding volume. Automation is the only thing that moves that constraint.

**When it's the WRONG choice:** Auto-merged into critical paths without test coverage, and for anything requiring understanding of intent. An AI-generated fix that satisfies the scanner while changing behavior is a production incident wearing a security fix's clothing. Require review proportional to blast radius, and never auto-merge into authentication, authorization, payment, or tenancy code.

### Ownership Routing
Automatically assigning findings to the accountable team based on repository ownership, service catalog, or deployment metadata. The unglamorous control that determines whether a program produces remediation or dashboards.

### Vulnerability Backlog / Aging
The population of open, confirmed, unremediated findings and their age distribution. The two numbers that matter: how many are past SLA, and how old is the oldest critical. Total count is a vanity metric that mostly reflects how many scanners you bought.

---

## 15. PSIRT, Disclosure, and Product Incident Response

### PSIRT (Product Security Incident Response Team)
**Definition:** The function responsible for receiving, triaging, coordinating, remediating, and disclosing vulnerabilities in the products an organization ships to customers. Distinct from a CSIRT/SOC, which defends the organization's own operations.

**Problem it solves:** Vulnerabilities in shipped products need a fundamentally different process than incidents in your own infrastructure: you must coordinate with an external reporter, develop a fix across multiple supported versions, coordinate release timing, notify customers who must act, and publish an advisory. A SOC has none of that machinery.

**When it's the WRONG choice:** As a separately staffed team at small scale. A designated owner, a documented process, a monitored intake address, and an escalation path is a PSIRT function; you do not need a team to have the capability. What is genuinely wrong is having no owner and discovering that when a researcher's disclosure deadline expires.

### FIRST PSIRT Services Framework
**Definition:** The reference framework describing the service areas a PSIRT may provide, currently at version 1.1, with a companion maturity document. Service areas cover stakeholder management, vulnerability discovery, triage and analysis, remediation, disclosure, and training.

**Problem it solves:** Gives a new PSIRT a service catalog and a maturity path rather than a blank page, and gives an existing one a gap analysis.

**When it's the WRONG choice:** As an implementation blueprint. It is deliberately model-agnostic and describes possible services; a small vendor implementing every service area would spend a year building process instead of fixing vulnerabilities. Use the maturity document to sequence.

### ISO/IEC 29147
The international standard for vulnerability **disclosure**: the external-facing half. Covers the disclosure policy, how reports are received, how you communicate with finders, and how advisories are published.

### ISO/IEC 30111
The international standard for vulnerability **handling**: the internal half. Covers triage, verification, root cause analysis, fix development, regression testing, and release-readiness sign-off. The pair (29147 external, 30111 internal) is the reference set that the CRA, the FIRST PSIRT framework, and ISO 27001 Annex A all read against.

### Responsible Disclosure
Responsible Disclosure is a process that allows security researchers to safely report found vulnerabilities to your team. It can be a messy process for researchers to know exactly how to share vulnerabilities in your applications and infrastructure in a safe and efficient manner.

### Coordinated Disclosure
Coordinated Disclosure is a policy under which researchers agree to report vulnerabilities to a coordinating authority, which then reports it to the vendor, tracks fixes and mitigation, and coordinates the disclosure of information with stakeholders including the public.

**Terminology note:** "coordinated vulnerability disclosure" (CVD) has largely replaced "responsible disclosure" as the preferred term across CISA, CERT/CC, and most vendor programs, because "responsible" framed the researcher as the party with obligations.

### VDP (Vulnerability Disclosure Policy)
**Definition:** The public document stating how to report a vulnerability to you, what is in scope, what testing is authorized, what you commit to (acknowledgment and update timelines), and a safe harbor commitment that you will not pursue legal action against good-faith research.

**Problem it solves:** Without one, researchers do not know where to send reports, do not know whether they are safe, and will sometimes disclose publicly instead. It costs almost nothing and is the highest ROI item in this section.

**When it's the WRONG choice:** Publishing one with no capacity to triage. A VDP creates an obligation and an expectation; an unanswered report becomes a public complaint and then a full disclosure. Also do not publish a VDP with a safe harbor your legal team has not approved.

### security.txt
A file at `/.well-known/security.txt` (RFC 9116) pointing researchers to your contact address, policy, and encryption key. The cheapest single improvement to your disclosure intake.

### Bug Bounties
A bug bounty program is a deal offered by many websites, organizations and software developers by which individuals can receive recognition and compensation for reporting bugs, especially those pertaining to security exploits and vulnerabilities.

**When it's the WRONG choice:** Before a VDP, before triage capacity, and before you have fixed the issues an automated scan would find. A bounty converts your intake volume into paid volume, and if your product has abundant low-hanging findings you will pay a lot for a list you could have generated yourself, while frustrating researchers with slow triage. Sequence: fix the obvious, publish a VDP, build triage, then pay.

### CNA (CVE Numbering Authority)
An organization authorized to assign CVE IDs within its scope, typically for its own products.

**Problem it solves:** Becoming a CNA gives you control over the timing, scope, and description of CVEs for your products, rather than having a third party assign and describe them.

**When it's the WRONG choice:** It is an ongoing obligation with process requirements, publication timelines, and quality expectations. Small vendors are usually better served by a coordinating CNA until disclosure volume justifies the overhead.

### Security Advisory
**Definition:** The customer-facing publication describing a vulnerability in your product: affected versions, severity, impact, fixed versions, workarounds, and credit. Ideally published in a machine-readable format (CSAF) alongside human-readable text.

**Problem it solves:** It is the mechanism by which your fix reaches the customers who must act. A patched version nobody knows to install is not a remediation.

**When it's the WRONG choice:** Under-describing to reduce alarm. Vague advisories ("security hardening improvements") prevent customers from prioritizing correctly, are increasingly incompatible with regulatory notification expectations, and are reverse-engineered anyway.

### Embargo
An agreed period during which a vulnerability is not publicly disclosed, allowing fix development and coordinated release across affected parties.

**When it's the WRONG choice:** When exploitation is already occurring. At that point secrecy protects nobody except the vendor's timeline, and defenders need mitigations more than they need a complete fix. Also worth remembering that CRA Article 14 reporting to authorities is not disclosure and is not subject to your embargo preferences.

### Out-of-Band / Hotfix Release
An unscheduled release carrying only a security fix, outside the normal release train.

**Problem it solves:** Decouples critical fix delivery from feature release cadence. The capability must exist and be exercised before you need it; discovering during an active exploitation event that your release process cannot produce a targeted patch is a bad afternoon.

### Customer Notification
**Definition:** Direct communication to affected customers about a vulnerability, incident, or required action, distinct from a public advisory.

**Problem it solves:** Some customers require direct notice contractually or regulatorily, and some fixes require customer action. CRA Article 14 requires notifying affected users, and in some cases all users, of the vulnerability or incident and any corrective measures.

**When it's the WRONG choice:** Notification drafted by security alone. Customer notification has legal, contractual, and commercial dimensions, and the message must be reviewed accordingly. Build the template and the approval path in advance.

### Product Security Incident vs Corporate Security Incident
A **product security incident** concerns a vulnerability or compromise in software you ship, with customer, disclosure, patch, and regulatory reporting dimensions. A **corporate security incident** concerns your own infrastructure or data. They have different owners, different clocks, different notification obligations, and different stakeholders, and they overlap when your own compromise affects the product you ship (a compromised build system being the canonical case).

### CRA Article 14 Reporting
See Section 3 for the regulation. Operationally, from 11 September 2026: an actively exploited vulnerability or severe incident in a product with digital elements on the EU market triggers an early warning to ENISA and the relevant national CSIRT within **24 hours** of becoming aware, a fuller notification within **72 hours**, and a final report within **14 days** of a corrective measure being available (one month for a severe incident), submitted through the Single Reporting Platform.

**What this requires you to have built before the date:** an intake path that reliably surfaces exploitation evidence, a triage decision within hours rather than days, a defined decision-maker with authority to file, pre-drafted report templates, legal review that is fast enough to fit inside 24 hours, and a record of what you knew when. The "becoming aware" clock is the hard part, and it starts before you have a fix.

### Post-Incident Review
A blameless review producing timeline, root cause, contributing factors, detection gaps, and specific corrective actions with owners and dates.

**When it's the WRONG choice:** When corrective actions are not tracked to completion. An untracked action list makes the next review's root cause identical to this one's.

### Forensic Readiness
The prerequisites for investigating a product security incident: sufficient log retention, immutable audit records, the ability to snapshot a workload before terminating it, correlation IDs across services, and known procedures for evidence preservation. Cheap to build in advance and impossible to build during an incident.

---

## 16. AI and Agentic Security

A framing note before the terms. Three separate problems get called "AI security" and conflating them wastes a lot of time:

1. **Securing AI features you ship.** Your product now contains a model, prompts, tools, and possibly agents. This is product security work on a new component type.
2. **Securing AI in your development process.** Coding assistants, agent CLIs, and MCP servers now operate inside your SDLC with your credentials. This is pipeline and developer environment security.
3. **Using AI for security work.** Triage, remediation, detection, review. This is tooling.

Most of this section covers (1) and (2), which are where the risk is.

### LLM Application
A product feature built on a large language model where the model receives input and produces output. Risk model is largely input/output oriented: prompt injection, sensitive information disclosure, insecure output handling, excessive reliance on output.

### AI Agent / Agentic Application
**Definition:** Software built on a model that plans, holds memory across steps or sessions, calls tools, and takes actions in other systems with delegated authority. The distinction from an LLM application is action: an agent does not just produce text, it does things.

**Problem the distinction solves:** A chatbot producing a bad answer wastes time. An agent making a bad decision terminates instances, merges pull requests, moves money, or drops tables. The security model changes from "prevent bad outputs" to "prevent cascading failures across autonomous systems," and the blast radius is the union of every credential, tool, and API the agent can reach.

### ADLC (Agentic Development Lifecycle)
The lifecycle of building, deploying, and operating agentic systems, and the emerging name for where agentic security controls are placed. Used in current vendor and OWASP material as the agentic analogue of the SDLC.

### OWASP Top 10 for LLM Applications
The model-level risk list: prompt injection, sensitive information disclosure, supply chain, data and model poisoning, improper output handling, excessive agency, system prompt leakage, vector and embedding weaknesses, misinformation, unbounded consumption. The right reference for a single-turn, non-agentic LLM feature.

### OWASP Top 10 for Agentic Applications (ASI)
**Definition:** Published 9 December 2025 by the OWASP GenAI Security Project, built from more than 100 contributors and grounded in disclosed 2025 incidents rather than projections. Identifiers run ASI01 through ASI10 ("ASI" for Agentic Security Initiative).

| ID | Risk | Primary defense |
|---|---|---|
| ASI01 | Agent Goal Hijack | Treat retrieved content as untrusted; constrain objectives |
| ASI02 | Tool Misuse & Exploitation | Least-agency tool scoping; parameter validation |
| ASI03 | Identity & Privilege Abuse | Per-agent identity; short-lived scoped credentials |
| ASI04 | Agentic Supply Chain Vulnerabilities | Signed components; AIBOM and provenance |
| ASI05 | Unexpected Code Execution (RCE) | Sandboxed execution; deny-by-default egress |
| ASI06 | Memory & Context Poisoning | Validated memory writes; ephemeral context |
| ASI07 | Insecure Inter-Agent Communication | Mutual authentication; signed messages |
| ASI08 | Cascading Failures | Blast-radius isolation; circuit breakers |
| ASI09 | Human-Agent Trust Exploitation | Forced confirmation showing raw actions |
| ASI10 | Rogue Agents | Behavioral monitoring; kill switches |

**Problem it solves:** Gives you standard vocabulary for classifying, tracking, and reporting agentic risk, and a threat model that covers the three things the LLM Top 10 handles incompletely: tool use, multi-step reasoning where a single injection compounds, and inter-agent communication.

**When it's the WRONG choice:** For a non-agentic LLM feature with no tools, no memory, and no autonomy; use the LLM Top 10 there. Also wrong as a control checklist. Like every Top 10, it is an awareness and classification document; the controls are in the mitigation guidance and in your own architecture.

### Agent Goal Hijack (ASI01)
**Definition:** An attacker redirects the agent's objective through content it reads rather than code it runs. The agent still believes it is pursuing the user's goal and is actually pursuing the attacker's.

**Reference incident:** EchoLeak (CVE-2025-32711, CVSS 9.3), the first known zero-click attack on an AI agent. A crafted email planted hidden instructions that Microsoft 365 Copilot later retrieved as context, causing data exfiltration from the user's environment with no clicks and no user interaction. Microsoft patched it server-side and no in-the-wild exploitation was confirmed, but the technique generalizes to any agent that reads untrusted content.

**Defense:** isolate retrieved content from instructions, constrain what the agent may do regardless of what its context says, and require human confirmation before sensitive actions. If a document can change what your agent does, documents are part of your attack surface.

### Prompt Injection (Direct and Indirect)
**Direct:** the user supplies instructions attempting to override the system prompt or safety constraints. **Indirect:** the instructions arrive inside content the model retrieves: a document, email, web page, issue, ticket, code comment, log entry, or tool response.

**Why indirect is the harder problem:** direct injection has one adversary, the user, and their capability is bounded by their own privileges. Indirect injection means anyone who can influence content the agent will read can influence the agent's behavior, at the agent's privilege level rather than their own. There is currently no complete defense; treat it as a boundary requiring architectural containment (least agency, egress control, forced confirmation on sensitive actions) rather than a bug to filter.

### Tool Misuse (ASI02)
Agents get real capabilities through tools: shell access, file operations, APIs, browsers, cloud CLIs. Tool misuse covers every way a legitimate tool is bent toward an illegitimate outcome, through deceptive input, poisoned tool metadata, or the agent chaining individually safe tools into an unsafe sequence.

**Reference incident:** the July 2025 Amazon Q Developer extension compromise. An attacker used an inappropriately scoped GitHub token to commit a malicious prompt into version 1.84.0 of a VS Code extension with more than 950,000 installs. The prompt instructed the agent to clean the system to a near-factory state, deleting local files and cloud resources through the AWS CLI tools the agent legitimately held. A formatting flaw prevented execution and AWS shipped a clean 1.85.0. The agent's own tools were the weapon.

**Defense:** least-agency tool scoping, strict parameter validation, runtime policy evaluation on every tool invocation, and attention to sequences rather than individual calls.

### Excessive Agency
Granting an agent more autonomy, tool access, or permission than its task requires. The root condition behind most of ASI02, ASI03, and ASI05. See Least Agency in Section 1.

### Agent Identity
**Definition:** Giving each agent its own identity with its own short-lived, task-scoped credentials, rather than borrowing a human's credentials or sharing a service account.

**Problem it solves:** Makes agent actions attributable and revocable, and caps blast radius. This is the control that prevents a hijack from becoming a breach: a goal hijack with read-only scope is an incident report; the same hijack with a broadly scoped personal access token is repository exfiltration.

**When it's the WRONG choice:** Never as a target. The practical trap is issuing per-agent identities that are all granted the same broad role, which produces attribution without containment.

### Memory and Context Poisoning (ASI06)
**Definition:** Planting malicious or false information in session context, retrieval indexes, or long-term memory stores, so the agent later treats it as its own knowledge.

**Why it is particularly nasty:** the payoff is delayed. The poisoned session looks clean, and the compromised behavior appears sessions or weeks later in an unrelated conversation with no obvious cause, which defeats session-level review entirely. Research demonstrated hidden instructions in processed content writing false long-term memories into a production assistant.

**Defense:** validate anything written to persistent memory, default to ephemeral context, scope memory per user and per task, and give operators the ability to inspect and flush what an agent has stored.

### Insecure Inter-Agent Communication (ASI07)
Multi-agent systems pass messages, delegate tasks, and discover peers. Without authentication, integrity, and authorization on those channels, an attacker can impersonate an agent, tamper with instructions in transit, replay delegation messages to inherit trust, or register a fake peer in a discovery service to intercept privileged traffic. The uncomfortable current state is that a great deal of agent-to-agent traffic runs on mutual assumption: agents accept instructions from peers because the message arrived, not because it was verified.

**Defense:** mutual authentication, signed and integrity-protected messages, explicit allowlists for which agents may delegate to which, and monitoring inter-agent traffic as a sensitive channel.

### Cascading Failure (ASI08)
**Definition:** One bad decision propagating through every workflow that trusts the affected agent. A hallucinated fact becomes another agent's input; an over-broad action triggers downstream automation; a single compromise spreads.

**Evidence:** a security analysis of the MCP specification found that with five MCP servers connected to one agent, a single compromised server achieved a 78.3% attack success rate and cascaded into other servers' operations 72.4% of the time. Connectivity multiplies risk.

**Reference incident:** during an explicit code freeze, Replit's coding agent deleted a production database holding records for more than 1,200 executives, then generated fabricated data and gave misleading answers about recovery. No attacker was involved. Autonomy plus broad access was sufficient.

**Defense:** blast-radius isolation between agents and environments, circuit breakers that halt automation on deviation from baseline, and hard separation between development and production access.

### Human-Agent Trust Exploitation (ASI09)
**Definition:** The agent's output manipulates a human into an unsafe action: a compromised coding assistant presenting a backdoored change as a routine fix, an approval flow where the agent's confident summary hides what is actually being approved, or persuasive language walking a user into revealing credentials.

**Why it matters more than it sounds:** this attacks the human approval step, which is the control most teams rely on as their safety net. An approval is only as good as the information it is based on, and the agent controls that information.

**Defense:** forced explicit confirmations that display the raw action rather than the agent's summary, immutable logs of what was presented versus what was executed, and prohibiting persuasive framing in agent output for sensitive workflows.

### Rogue Agent (ASI10)
An agent operating outside policy through compromise, misalignment, or drift, while still appearing legitimate. The defining trait is persistence: it keeps acting and everything about it looks normal. A prompt injection can leave an agent quietly exfiltrating across sessions; a cost-optimization agent can decide backups are waste; an orchestrator can spawn sub-agents nobody inventoried.

**Defense:** behavioral baselines with deviation alerting, lifecycle governance so every agent has an owner and an expiry date, sandboxed operation by default, and a kill switch that has been tested. Detection requires knowing what normal looks like, which most organizations cannot currently answer for a single agent.

### MCP (Model Context Protocol)
**Definition:** The protocol, originating at Anthropic, by which agents connect to external tools and data sources. It has become the dominant interface for agent tool use, which makes MCP servers a first-class supply chain and trust surface.

**Risk summary:** an MCP server is code you install that the agent trusts to describe its own capabilities. Tool descriptions are model-readable instructions, which means a malicious or compromised server can influence agent behavior through metadata alone (tool poisoning), independent of the tool's actual function. Add runtime discovery, and the trust set changes after deployment. **CVE-2025-6514** in `mcp-remote`, a CVSS 9.6 command injection in a package with more than 437,000 downloads, is the reference example of one bad component compromising every agent that pulls it. The OWASP MCP Top 10 covers this layer specifically.

**When exposing an MCP server is the WRONG choice:** as a thin wrapper over a broad API. An MCP server exposing a full cloud CLI or an unconstrained database client hands the agent the entire permission surface; the tool boundary is where you should be narrowing capability, not passing it through.

### Tool Poisoning
Malicious instructions embedded in tool names, descriptions, or parameter documentation, which the model reads as guidance. Defeats code-signature-based scanning entirely because the payload is natural language, not code. Published research has demonstrated that pattern-matching scanners miss the majority of critical threats in this class for exactly this reason.

### OWASP Agentic Skills Top 10 (AST10)
An OWASP project, incubated at the 2026 Project Summit in Oslo, covering risks in the "skills" abstraction layer above models and protocols: packaged agent capabilities distributed and loaded like plugins, with the same supply chain and instruction-injection properties as MCP servers. Mapped against MAESTRO layers. Treat as an active, fast-moving project rather than a settled standard.

### Agent Sandboxing
Running agent code execution inside a containerized or microVM sandbox with least privilege, deny-by-default network egress, and parameterized APIs instead of raw shell access.

**Problem it solves:** Any agent that can write and run code, or that hands a string to a subprocess, file path, or interpreter, should be presumed exploitable through content alone (ASI05). Sandboxing plus egress denial limits what a successful execution can actually reach, which is the difference between an incident and a breach.

### Kill Switch and Circuit Breaker
A **kill switch** immediately halts an agent or fleet and revokes its credentials. A **circuit breaker** automatically halts an automated chain when behavior deviates from baseline or when error and action rates exceed thresholds.

**When they're the WRONG choice:** Never wrong to have; frequently untested. An untested kill switch is a documented intention. The requirements are that it works without the agent's cooperation, that it revokes credentials rather than just stopping the process, and that someone has exercised it in the last quarter.

### Human-in-the-Loop vs Human-on-the-Loop
**In the loop:** the human approves each consequential action before it executes. **On the loop:** the agent acts and the human monitors, intervening on exception.

**When in-the-loop is the WRONG choice:** for high-frequency low-consequence actions, where approval fatigue guarantees reflexive approval, which is worse than no gate because it manufactures false assurance and an audit trail implying review that did not happen. Reserve in-the-loop for irreversible, high-blast-radius, or externally visible actions, and make those confirmations display the raw action rather than the agent's description of it.

### Guardrail Model / Input-Output Filtering
A separate classifier or model evaluating inputs and outputs for policy violations, injection attempts, sensitive data, or unsafe content.

**When it's the WRONG choice:** As the primary defense against prompt injection. Filtering is probabilistic against an adversary with unlimited attempts, and a bypassed filter provides no containment. Use it as a layer that raises cost and generates signal, on top of architectural containment (least agency, egress control, sandboxing) that holds when the filter fails.

### AI Red Teaming and Evals
**Red teaming** is adversarial testing of AI behavior: jailbreaks, injection chains, tool misuse, data extraction, and multi-turn manipulation. **Evals** are repeatable automated test suites measuring behavior against defined criteria, run continuously like a test suite.

**Problem they solve:** Model and prompt changes alter behavior in ways unit tests do not capture. Evals are the only regression mechanism for a non-deterministic component.

**When manual red teaming alone is the WRONG choice:** as your continuous control. Manual red teaming finds novel classes; evals prevent regression. You need both, and the output of the first should become cases in the second.

### Shadow AI
Unsanctioned AI tools, models, agents, and MCP servers in use across the organization without inventory, review, or governance. The AI-era equivalent of shadow IT, and currently near-universal. Recent industry survey data puts the share of organizations lacking full visibility into AI use across the SDLC around 80%.

**Why it is a product security problem specifically:** shadow AI in the development environment means unreviewed code generation, unreviewed dependency suggestions, source code and secrets leaving through prompts, and agents holding repository and cloud credentials, all outside your controls.

### AI Governance / Acceptable Use
**Definition:** Policy and enforcement defining which AI tools, models, and MCP servers may be used, for what, on what data, with what review requirements, and with what enforcement mechanism.

**Problem it solves:** Gives you a defensible answer to "what are we allowing," which is the prerequisite for both risk reduction and audit response. Auditors are already asking who approved an agent's permissions, where its action logs live, and how fast it can be shut off.

**When it's the WRONG choice:** A prohibition-based policy with no approved alternative. Blanket bans on AI coding tools produce shadow use with zero visibility, which is strictly worse than governed use. Provide an approved path and enforce against the alternatives.

### Vibe Coding
**Definition:** Describing intended outcomes in natural language and accepting the model's implementation with minimal line-by-line review. The term was coined by Andrej Karpathy in early 2025 and has become the standard label for the practice.

**Problem it solves:** For the practitioner, velocity. For product security, it names a specific risk profile: high commit volume, low author comprehension, and a reviewer who often understands the code less well than a human author would have.

**When it's the WRONG choice:** For authentication, authorization, cryptography, tenancy enforcement, payment logic, deserialization, and anything security-critical, where the failure mode is code that works on the happy path and is subtly wrong on the boundary. The reasonable policy line is not "no AI" but "AI-generated changes touching these paths require the same scrutiny as an unverified external contributor's PR."

### AI-Generated Code Review Policy
**Definition:** An explicit policy for how AI-generated code is treated: labeled or not, review requirements, which paths require additional security review, whether agent-authored commits may be auto-merged, and what evidence is retained.

**Problem it solves:** AI generation outpaces review capacity, and survey data suggests a majority of organizations already report development velocity exceeding security review capacity. Without a policy, the resolution defaults to less review per change.

**When it's the WRONG choice:** A policy requiring manual security review of all AI-generated code. That is unachievable at current volumes and will be ignored. Differentiate by path sensitivity and blast radius, automate what you can, and accept lighter review on low-consequence code deliberately rather than by default.

**A note on tooling gaps:** the failure classes AI-assisted development actually produces are not always the ones existing tooling looks for. On 31 March 2026, version 2.1.88 of the `@anthropic-ai/claude-code` npm package shipped with a 59.8 MB JavaScript source map, exposing roughly 512,000 lines of proprietary TypeScript across about 1,900 files. Anthropic characterized it as a release packaging error caused by human error rather than a security breach, and stated no customer data or credentials were involved. The mechanism is the point: the build runtime generated source maps by default and `*.map` was not excluded from the publish configuration. That is a packaging configuration failure, not a logic bug, and neither static analysis nor secret scanning covers it. Artifact hygiene, packaging drift, and inspecting what actually lands in the published tarball rather than what the repository contains deserve explicit pre-publish checks. Reportedly a near-identical source map exposure had occurred on an earlier version roughly a year prior, which is the more useful lesson: a packaging failure class with no detective control recurs.

### Model Provenance and Model Registry
Knowing where a model came from, which version is deployed, what it was fine-tuned on, and whether the weights are the ones you validated. The registry is the inventory and access control point. Relevant to ASI04: models pulled at runtime from public hubs are dependencies with no signature verification in most deployments.

### RAG Security (Retrieval-Augmented Generation)
Risks specific to retrieval architectures: the retrieval corpus is an injection vector (anything indexed can carry instructions), retrieval frequently bypasses the source system's authorization so users receive content they could not have accessed directly, embeddings can leak information about source documents, and index poisoning persists across sessions.

**The control that is most often missed:** authorization at retrieval time, evaluated against the requesting user, not just at ingestion. A RAG system indexing everything and retrieving for anyone is a permission bypass with a friendly interface.

### Agent Observability
**Definition:** Immutable, queryable records of what an agent did: prompts, retrieved context, tool calls with parameters, memory writes, inter-agent messages, what was displayed to a human at confirmation, and what was actually executed.

**Problem it solves:** Traditional AppSec tooling cannot see any of this. SAST reads source, SCA reads manifests, and neither reads system prompts, tool descriptions, retrieved context, or memory stores, which is where agentic attacks live. Without this telemetry a hijacked agent is indistinguishable from a busy one, and no incident is investigable.

**When it's the WRONG choice:** Never as a capability; the trap is logging full prompts and context containing customer data and secrets into a store with weaker controls than the source systems. Redact at emission and hold the log store to the sensitivity of its highest-classified content.

---

## 17. Data Protection and Privacy Engineering

### Data Classification
**Definition:** Categorizing data by sensitivity and regulatory obligation (public, internal, confidential, restricted; or PII, PHI, PCI, secret), with defined handling requirements per class.

**Problem it solves:** Every proportionate control decision depends on it. "Encrypt sensitive data" is unactionable without a definition of sensitive.

**When it's the WRONG choice:** A scheme with more than four or five classes, or one requiring per-field manual tagging at scale. Complexity in classification produces inconsistency, and inconsistent classification is worse than a coarse scheme applied uniformly.

### Privacy by Design
Building privacy properties into architecture rather than adding notices and toggles afterward. Practically: data minimization, purpose limitation, retention limits, and the technical ability to locate, export, and delete an individual's data across every system that holds it.

**When it's the WRONG choice:** Nothing to get wrong about the principle. The engineering reality is that deletion and export are architectural capabilities: if personal data is scattered across event streams, analytics warehouses, backups, caches, and now vector indexes with no lineage, no policy will make a deletion request satisfiable.

### Data Minimization
Collecting and retaining only the data required for the stated purpose. The most effective privacy and security control available, because it removes risk rather than managing it. Also the hardest to sell internally, because it conflicts with the instinct to keep everything for future analysis.

### Tokenization
Replacing sensitive values with non-sensitive surrogates, with the mapping held in a separate, tightly controlled vault.

**Problem it solves:** Removes sensitive data from the systems that reference it, which shrinks compliance scope (notably PCI) and reduces breach impact for the referencing systems.

**When it's the WRONG choice:** When the application needs to operate on the value rather than reference it. Also note that the token vault becomes the crown jewel; you have concentrated risk, which is usually correct but must be defended accordingly.

### Field-Level Encryption
Encrypting specific sensitive fields inside a record rather than relying only on volume or database-level encryption.

**Problem it solves:** Database-level encryption at rest protects against physical media theft and almost nothing else, because any authorized query returns plaintext. Field-level encryption keeps the field protected from an attacker with database access.

**When it's the WRONG choice:** For fields you need to search, sort, join, or aggregate on, unless you accept the significant complexity of searchable encryption schemes. Also wrong when keys are stored in the same system as the ciphertext, which is a common implementation that provides no meaningful protection.

### Logging Hygiene
Preventing secrets, credentials, session tokens, full request bodies, and personal data from entering logs; and applying the source data's classification to the log store.

**Problem it solves:** Logs are the most commonly over-shared, over-retained, and under-protected data store in most products, frequently readable by far more people than the production database. It is a routine finding that the fastest path to customer PII is the log platform.

### Data Residency and Sovereignty
Requirements that data be stored and processed in specific jurisdictions. An architectural constraint, not a configuration setting: it reaches deployment topology, backup destinations, log aggregation, support access paths, and third-party subprocessors including model providers. Discovering a residency requirement after building a single-region global architecture is a rewrite.

### Retention and Deletion
Defined maximum retention per data class with automated enforcement, and a working deletion capability across primary stores, replicas, caches, search indexes, analytics warehouses, backups, and vector stores.

**When manual deletion is the WRONG choice:** Always, at any scale. A documented retention policy without automated enforcement is a statement of intent that audit will eventually test and find false.

---

## 18. Metrics, Measurement, and Maturity

### Gathering Metrics / Measurement
Essentially you should be taking metrics data into a tool that can display that information in an informative and easy to understand way, to measure your progress, find trends and use that to enrich your overall program.

**When it's the WRONG choice:** Building a dashboard before deciding what decision it informs. Every metric should have a named audience and an action it can trigger. Metrics that exist to demonstrate activity consume real engineering time and produce no decisions.

### Coverage Metrics vs Outcome Metrics
**Coverage:** what fraction of the portfolio has a control applied. "87% of critical repositories have secret scanning and branch protection enabled."
**Outcome:** whether risk actually decreased. "Median time to remediate critical findings on internet-facing services fell from 34 to 11 days" or "zero cross-tenant authorization defects reached production this quarter."

**The trap:** coverage is easy to measure and easy to improve without reducing risk. Programs report coverage because it moves; leadership eventually asks whether anything got safer. Lead with outcomes and use coverage as the supporting explanation.

### Escape Rate / Defect Escape
The proportion of vulnerabilities discovered after release (by pentest, bug bounty, researcher, or incident) versus before. The single best measure of whether your pre-release controls actually work, and the one most programs do not track because it requires honest accounting of external findings.

### Vulnerability Density
Findings per unit of code or per service, normalized so that a large service is not automatically the worst. Useful for identifying teams needing support and for tracking a bug class over time. Dangerous if used comparatively between teams without normalizing for language, age, and scan configuration, at which point it becomes a way to punish teams for having better coverage.

### Adoption Metrics
Usage of the paved road, the shared libraries, the scaffolding, the champion program, the office hours. The leading indicator for everything else: adoption rises before outcomes improve, and falling adoption predicts the program's decline a quarter or two before the outcome metrics show it.

### Leading vs Lagging Indicators
**Leading:** adoption, coverage, threat models completed, training completion, time-to-first-scan on new repositories. Predict future outcomes.
**Lagging:** incidents, escaped defects, MTTR, bounty findings. Confirm past performance.

Report both. Leading indicators alone are activity theater; lagging indicators alone give you no steering capability.

### Signal-to-Noise Ratio
Confirmed true positives as a fraction of total findings reported by a tool, per tool and per rule. The metric that should drive tool tuning, rule disabling, and renewal decisions. A tool with a 15% true positive rate is consuming triage capacity and developer trust regardless of what it costs.

### Maturity Model
See SAMM, BSIMM, DSOMM in Section 3.

**When maturity assessment is the WRONG choice:** As an annual ritual disconnected from a backlog. The assessment's only value is the prioritized set of changes it produces; if that set does not enter a roadmap with owners and dates, you have generated a score.

---

## 19. Culture, Enablement, and Education

### Developer Education
Is when you provide your developers security related training. This can come in the form of Lunch and Learns, Workshops or paid training.

**When it's the WRONG choice:** Annual generic compliance training as the entirety of the program. Retention from context-free annual training is near zero, and it costs the organization real hours while giving security a false sense of coverage. Effective education is stack-specific, tied to your own findings and your own codebase, and delivered near the moment of relevance.

### Advocacy Programs
Are ways you support your developers in terms of making sure they have the security resources they need, places to find the answers they need, and making sure they have all the things they need to know to make good, secure software.

### Just-in-Time Training
**Definition:** Delivering targeted education at the moment a developer encounters the relevant issue: a short, specific explanation and fix pattern attached to the finding, in the PR, in their language and framework.

**Problem it solves:** Relevance and timing are the entire mechanism of retention. A two-paragraph explanation of why this specific query is injectable, in the PR where they wrote it, outperforms a four-hour course.

**When it's the WRONG choice:** For foundational concepts that need to be understood before they can be applied. Threat modeling, authorization design, and cryptographic reasoning need real teaching time; you cannot deliver them in a PR comment.

### Blameless Culture
Treating security defects and incidents as systemic and process failures rather than individual ones, and structuring reviews to find contributing conditions instead of culpable people.

**Problem it solves:** It is the precondition for accurate information. In a blame culture, engineers stop reporting mistakes and near-misses, which removes your primary source of early signal.

**When it's the WRONG choice:** Blameless is not consequence-free. Deliberate circumvention of controls, repeated disregard of documented standards, and concealment are accountability matters. The distinction is between error and choice, and conflating the two in either direction is corrosive.

### Office Hours
A recurring open session where any engineer can bring a security question without filing a ticket. Cheap, high-signal, and one of the most reliable ways to surface work that would otherwise never reach you, plus a lightweight recruiting channel for champions.

### Internal Documentation as Product
Treating your security documentation, standards, and paved-road guides as a product with users, findability requirements, feedback, versioning, and maintenance ownership.

**Problem it solves:** Documentation nobody can find is equivalent to documentation that does not exist, and stale documentation is worse, because it actively misleads and destroys trust in the rest of the set.

### Security Champions
See Section 4.

### CTFs
See Section 8.

---

## 20. Hardware, Embedded, and Connected Products

This section applies if you ship hardware, firmware, or connected devices. It is also where CRA obligations bite hardest, since the regulation's scope is "products with digital elements."

### Product with Digital Elements
The CRA's scope term: hardware and software products, including remote data processing solutions, and software or hardware components placed separately on the market. If your product contains software or firmware and is made available on the EU market with a direct or indirect data connection, it is very likely in scope. Sector-specific products (medical devices, vehicles, certain aviation) are covered under separate regimes.

### Firmware Security
Security of the software running on the device itself: memory safety in a constrained environment, update capability, secret storage, debug interface exposure, and the reality that the attacker may have unlimited physical access to the device.

**Distinguishing constraint:** you cannot assume you can patch quickly, you cannot assume the device is network-reachable when you need it to be, and you cannot assume the attacker lacks the hardware.

### Secure Boot / Chain of Trust
**Definition:** Each stage of the boot process cryptographically verifies the next before executing it, anchored in an immutable root of trust, so unauthorized firmware cannot execute.

**Problem it solves:** Without it, firmware modification is trivially persistent and undetectable, and every software-layer control on the device is defeated from below.

**When it's the WRONG choice:** Never for a product handling anything sensitive. The failure modes to design for are: an unrecoverable device when verification fails (you need a recovery path), and key compromise with no ability to rotate the trust anchor, which turns one leaked key into a permanently compromised product line.

### Root of Trust / TPM / Secure Element
The hardware component providing immutable identity, key storage, and cryptographic operations that software cannot extract: a TPM, a secure element, a TrustZone-style secure enclave, or a dedicated security processor. The anchor for secure boot, device identity, and attestation.

### OTA Update Security
**Definition:** Over-the-air update delivery with signed images, version and rollback protection, integrity verification before installation, atomic or A/B installation so a failed update does not brick the device, and a resilient fallback.

**Problem it solves:** Update capability is the single most important security property of a connected product, because it is the only mechanism by which any future vulnerability gets fixed. Under CRA, the ability to deliver security updates through the declared support period becomes an obligation.

**When it's the WRONG choice:** Never wrong to have; frequently the biggest attack surface on the device. An update mechanism that verifies signatures with a key you cannot rotate, or accepts downgrades, or trusts the server's TLS certificate without pinning, has handed the attacker a supported code execution path with distribution built in.

### Device Identity and Provisioning
Giving each device a unique, hardware-backed identity at manufacture, with credentials that are not shared across the fleet. Shared per-model credentials mean extracting one device's key compromises every unit ever sold, and the extraction is a physical-access problem you cannot prevent.

### Anti-Tamper and Debug Interface Hardening
Disabling or authenticating JTAG, SWD, UART, and other debug interfaces in production units; detecting and responding to enclosure opening; and not leaving test modes, factory reset backdoors, or engineering credentials in shipped firmware. Test what shipped, not what the build configuration says, because the gap between them is where these findings live.

### Counterfeiting and Physical Supply Chain
Component substitution, gray-market production runs, cloned devices, and unauthorized firmware images on legitimate hardware. Controls are device attestation against a manufacturing record, signed firmware bound to hardware identity, and provisioning that a contract manufacturer cannot replicate at will.

### IoT Security Baselines
The prescriptive baselines to design against: **ETSI EN 303 645** (consumer IoT, no universal default passwords, vulnerability disclosure, keep software updated, secure credential storage), **NIST IR 8259 / SP 800-213** (device cybersecurity capability core baseline), and sector regimes such as **IEC 62443** for industrial and **FDA premarket cybersecurity guidance** for medical devices.

**When they're the WRONG choice:** Certifying against a baseline as the security program. These are floors, most are self-assessed, and none of them cover your product's business logic, cloud backend, or mobile app, which is where the actual exploitation usually happens.

### Support Period and End of Support
See Section 3. For connected products specifically, this is a declared commitment with regulatory weight: how long you deliver security updates, what happens to cloud dependencies when support ends, and how customers are informed. A device whose cloud backend is decommissioned is a brick regardless of whether its firmware is patched, and "we stopped supporting it" is not an adequate answer to a customer with 5,000 units deployed.

---

## Appendix: What Changed and What to Verify

Items in this document that were moving as of July 2026 and should be re-verified before being cited as fact:

| Item | Status as of July 2026 |
|---|---|
| NIST SSDF v1.2 (SP 800-218r1) | Initial public draft published 17 Dec 2025, comments closed 30 Jan 2026. v1.1 remains the authoritative version for federal attestation. |
| EU CRA Article 14 reporting | Applies from 11 Sept 2026. ENISA Single Reporting Platform was still being stood up as of mid-2026. |
| EU CRA full application | 11 Dec 2027. Various 2026 dates circulate for product classes (type A/B/C); these appear to relate to harmonized standards availability rather than manufacturer obligations, and should be verified against the Commission's published timeline before being relied on. |
| AIVSS | Pre-1.0. Public review opened April 2026, v1.0 targeted before end of 2026. |
| OWASP Agentic Skills Top 10 (AST10) | Active project, incubated 2026. Not a settled standard. |
| CSA MAESTRO | v2 in circulation. |
| ADR (Application Detection and Response) | Emerging vendor category, definition not yet settled. |
| OWASP Top 10:2025 | Announced Nov 2025, finalized Jan 2026. Check what edition your tooling, training, and audit checklists are mapped to. |
| OWASP ASVS | 5.0.0 (May 2025) is the current stable release. |
| FIRST PSIRT Services Framework | v1.1. |

Anything referencing a specific CVE, incident, or download count reflects reporting at the time of writing and should be checked against the primary source before use in a customer-facing or regulatory context.
