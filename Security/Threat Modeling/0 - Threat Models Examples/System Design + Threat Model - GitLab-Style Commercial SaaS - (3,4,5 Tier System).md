# Commercial Code Platform SaaS (GitLab-Style): System Design & Threat Model
---

## Section 0 — Concepts You Need First (Vocabulary Primer)

You can't threat model what you can't name. Everything downstream assumes these. Each is explained once, here, in plain language.

**Three-, four-, and five-tier architecture.** A "tier" is a layer that runs separately, usually with a network boundary between it and its neighbors, so a break-in on one side doesn't automatically reach the other.

- **3-tier:** Client → Application → Data. One application layer does everything: serves pages, speaks Git, runs CI jobs, queries the database.
- **4-tier:** Client → Edge/Web → Application → Data. A front tier (load balancer, WAF, static content) is split off so the raw internet never touches business logic directly.
- **5-tier:** adds an Integration/middleware tier (job queues, background workers, the CI orchestrator and its runner fleet) between Application and Data. Work that must not run near the database gets pushed into its own zone.

The rule of thumb, repeated because it's the whole lesson: each tier buys you a boundary you can enforce and costs you complexity you must operate. A tier only counts if there's a real enforced boundary between it and its neighbor, not just a diagram box.

**Repository, commit, ref, push.** A repository ("repo") is a project's code plus its full history. A commit is one saved change. A ref (like a branch name) is a movable pointer to a commit. `git push` sends your commits to the server. The server speaks the Git wire protocol; the platform must implement it faithfully or `git push` breaks.

**Merge request / pull request (MR/PR).** A proposal to take changes from one branch and fold them into another, with review and discussion attached. This is the review gate: the place a human (or a required check) says "yes, this can go in."

**Protected branch.** A branch (often `main`) with extra rules: can't push directly, requires review, requires passing checks. It's an authorization control disguised as a workflow feature.

**Pipeline, job, runner.** A pipeline is the automated sequence that runs when code changes: build, test, scan, deploy. Each step is a job. A runner is the machine that actually executes a job. The critical fact: jobs run customer-written commands, so a runner is a machine executing untrusted code by design.

**Ephemeral runner / sandbox.** A runner that is created fresh for one job and destroyed after, so nothing from one job survives into the next. The alternative, a long-lived reused runner, is how one tenant's job contaminates another's.

**CI job token.** A short-lived credential the platform injects into a running job so the job can talk back to the platform (fetch dependencies, push artifacts). Its scope is the whole security question: a job token that can reach other projects is a lateral-movement tool.

**OIDC token / workload identity.** Instead of storing long-lived cloud credentials in CI, the job requests a short-lived signed token proving "I am pipeline X of project Y," and the cloud provider trades it for temporary access. Removes standing secrets from the pipeline. The trust rule (which project/branch the token is minted for) is exactly where this gets dangerous if it's loose.

**Personal access token (PAT).** A long-lived string a user creates to let scripts and tools act as them. Convenient, and the single most leaked credential type on code platforms, because they end up in configs, logs, and other repos.

**CI variables / secrets.** Values (cloud keys, signing keys, API tokens) the platform hands to jobs at runtime. The core secret-handling question is which jobs are entitled to which secrets, and whether an outsider's pipeline can ever receive them.

**Fork and fork pipeline.** A fork is a copy of someone's repo under your own account. You then open an MR back to the original. The trap: when the original project's CI runs against your proposed change, does it hand _your_ code the _original project's_ secrets? If yes, anyone on the internet can steal a project's CI secrets by opening one MR. This is the defining vulnerability of the category.

**Artifact and container registry.** An artifact is a file a pipeline produces (a compiled binary, a report). A container image is a packaged application. The registry stores images. Both are consumed downstream, by the customer and their users, so poisoning one is a supply-chain attack that flows outward.

**SBOM, provenance, SLSA.** An SBOM (Software Bill of Materials) lists what's inside a build. Provenance is a signed record of how and where a build was produced. SLSA is a framework of levels for build-integrity guarantees. Together they answer "can you prove this artifact came from the source you think it did?"

**Webhook and SSRF.** A webhook is an outbound HTTP call the platform makes to a URL the user configured (to notify Slack, Jira, etc.). SSRF (Server-Side Request Forgery) is tricking a server into making requests on the attacker's behalf, often to internal addresses the attacker can't reach directly. User-controlled webhook URLs are a classic SSRF on-ramp into the platform's own network.

**Multi-tenancy.** Many customers ("tenants") share the same infrastructure. The isolation between them is mostly logical (permission checks on every query), not physical. One missing `WHERE project_id =` is a cross-tenant breach.

**SSO / SAML / OIDC (as login).** Single sign-on lets a company's users log in via their own identity provider (Okta, Google). The platform trusts assertions from that provider. A misconfigured or spoofable federation means attackers arrive holding "valid" logins.

**STRIDE.** A threat-category checklist: **S**poofing, **T**ampering, **R**epudiation, **I**nformation disclosure, **D**enial of service, **E**levation of privilege. Used in the threat table so a whole category doesn't get silently skipped.

---

## Section 1 — What This System Is and Why It Works This Way

From a user's point of view, this platform does four things: it **stores code**, it lets teams **review changes** before they're accepted, it **automatically tests and builds** that code (CI), and it **deploys** it (CD). You push from your laptop, teammates comment, robots test, and it ships.

Why each major part exists, framed as the user need it answers:

- **Web interface** — "I need to browse code, review, and click merge without memorizing commands."
- **Git service** — "`git push` has to work." The platform must speak Git's protocol natively.
- **CI runners** — "Test and build my code automatically." This is also the feature that forces the entire security architecture, because it executes user-written commands.
- **Database** — "Remember my accounts, permissions, comments, pipeline history." Git stores code; the database stores everything _about_ the code.
- **Background workers and queues** — "Don't make me wait 30 seconds while you email 500 people or scan a huge repo." Slow work runs off the request path.
- **Vault/secrets store** — "Let my pipeline deploy without me pasting cloud keys into a script." Secrets are injected at runtime, which is a controlled hole punched in the system's isolation.
- **Sharing and permissions** — "My team needs write access; the public needs read-only; contractors need one project." Access control is a first-class feature, not an afterthought.

Why 5 tiers instead of 3, stated as the user's implicit fear:

- **"What if one server bug exposes everything?"** In 3-tier, a customer's build script runs on the same box that holds database credentials and every tenant's data. Splitting tiers means a bug in the API layer reaches only what the API layer can reach, and CI runs in a zone with no route back to the data tier.

What breaks if a part is missing:

- No tier separation → one SSRF or SQL-injection bug in an API endpoint becomes full database exfiltration, and a hostile build script sits next to your secrets.
- No ephemeral runners → one tenant's job leaves credentials, caches, or malware behind for the next tenant's job on the same machine.
- No fork-pipeline restriction → the platform becomes a secret-vending machine for anyone who can open a merge request.
- No strict CI-token scoping → a compromised job in one project becomes a foothold in every project it can reach.
- No review gate on protected branches → malicious or broken code ships straight to production.

---

## Section 2 — Why Security Matters Here (Conversational)

Think about what's actually on this platform: your company's entire source code, the CI secrets that deploy it (cloud root keys, signing keys, package-registry tokens), and the build system that produces the software your own customers run. This isn't "an account gets compromised." Compromise the build system and you compromise everything built there, and everything those builds are shipped into. That's the SolarWinds shape, and the Codecov shape: poison the factory, not the product.

What a real person worries about, and whether they're worried about the right things:

- **"Can anyone else read our code?"** — Reasonable, and mostly handled by tenant isolation and permission checks. The failure mode is subtle: not a dramatic breach but a missing authorization check that lets one project read another's.
- **"What about the robots running our builds?"** — This is the one they _under_-worry about. CI hands real secrets to a machine running arbitrary code. The danger isn't your own code; it's whether a stranger's pipeline, or a leftover from someone else's job on shared hardware, can reach those secrets.
- **"What if an employee at the vendor goes bad, or gets phished?"** — Reasonable. An insider with production access or the ability to push a platform update is the highest-leverage attacker in the model.
- **What they don't worry about but should:** fork pipelines and leaked tokens. The most common real-world compromise of a project on these platforms isn't a crypto break; it's a personal access token committed to a public repo, or a fork-MR pipeline handed secrets it should never have seen.

Why the decisions are harder than they look: the product must be simultaneously **open** (anyone can sign up, push any code, run any build) and **contained** (none of that code touches anyone else's stuff). Every convenience feature punches a hole in containment. Caching between jobs means state survives. Fork pipelines mean outsiders' code runs in your context. OIDC to the cloud means a token can be traded for real access. Long-lived PATs mean a leaked string is standing access. None are optional in a real product, so the job is never "don't punch holes," it's "make each hole small, scoped, short-lived, and monitored."

What "secure enough" means here, stated plainly: one tenant's code and CI secrets cannot be read or altered by anyone their permissions don't allow, even when another tenant on the same hardware is actively hostile, and even when that tenant is running arbitrary code inside a runner right now. Notice it says nothing about the customer's own habits, that's Section 15.

---

## Section 3 — Architecture Components

### Tier 1 — Clients (Presentation)

|Component|What It Does|Security-Relevant Detail|
|---|---|---|
|Browser web app|Browse code, review, merge, admin|Session tokens; CSRF and XSS surface; renders untrusted repo content (READMEs, filenames)|
|Git CLI|`git push`/`pull` over SSH or HTTPS|Authenticates via SSH key or PAT; the PAT is the leak-prone one|
|IDE / editor integrations|In-editor review, CI status|Hold long-lived tokens on developer laptops|
|API clients / bots / CI of other systems|Automation, terraform, deploy scripts|Machine tokens; often over-scoped and long-lived|

### Tier 2 — Application Services

|Component|What It Does|Security-Relevant Detail|
|---|---|---|
|API gateway / WAF|Single entry, TLS termination, rate limiting|Credential-stuffing throttles, bot detection, request shaping live here|
|Auth service|Sessions, tokens, SSO/SAML/OIDC handling, 2FA|Central identity enforcement; SSO assertion validation is a spoofing surface|
|API / application services|Business logic, authorization on every object|Where a missing permission check becomes a cross-tenant read|
|Git service|Speaks Git protocol, reads/writes repos|Path handling, access checks per ref; renders/serves untrusted content|
|Webhook dispatcher|Outbound HTTP to user-configured URLs|Prime SSRF surface; must not resolve internal addresses|
|Notification service|Emails, in-app alerts, push|Content-free where possible; no secrets in payloads|

### Tier 3 — Integration / Orchestration (the tier 3-tier designs lack)

|Component|What It Does|Security-Relevant Detail|
|---|---|---|
|Job queue|Buffers work between app and workers|Message content is a job spec, not secrets|
|Background workers|Slow jobs (scans, emails, imports)|Run with scoped service credentials, no broad DB access|
|CI orchestrator|Schedules pipelines, authorizes secret release|Makes the fork-vs-trusted decision (Section 8a)|
|CI runner fleet|Ephemeral sandboxes that execute jobs|Untrusted code by design; isolated network, destroyed per job|
|Secrets broker|Mints scoped CI tokens / OIDC assertions per job|Enforces which job gets which secret; the blast-radius chokepoint|

### Tier 4 — Data Stores

|Store|Contents|Security-Relevant Detail|
|---|---|---|
|PostgreSQL|Users, permissions, MRs, pipeline metadata, sharing graph|The tenant-isolation battleground; every query needs a scope check|
|Redis|Cache and queue|Not a source of truth; poisoning risk if trusted blindly|
|Git repository storage|The code itself|Dedicated nodes; access mediated only through the Git service|
|Object storage|CI artifacts, logs, container images|Downstream-consumed; poisoning here flows to customers|
|Secrets vault|CI variables, deploy keys|Encrypted separately from the main DB; never in DB columns|
|Audit / security logs|Auth events, permission changes, admin actions|Must never contain secrets or tokens; tight access and retention|

### Third-Party Integrations

- **Identity providers (Okta/Google/SAML)** — trusted for login; a breach there arrives as valid sessions.
- **Cloud provider (OIDC federation)** — trades CI tokens for temporary cloud access; trust config is security-critical.
- **Email / push** — untrusted transport; content-free triggers.
- **Package/dependency registries** — the platform's own supply chain (Section 11, attacker 6).
- **App/marketplace integrations** — third-party apps granted API scopes; over-granting is a standing risk.

---

## Section 4 — Core Design Decisions

|Layer|Choice|Rationale|Plain-English Explanation|
|---|---|---|---|
|Overall|5-tier over 3-tier|Untrusted code execution demands an isolation zone|Customer build scripts must never run near the database or other tenants' data|
|Edge|LB + WAF tier separate from app|Absorb floods and probes before logic|A bouncer at the door, not inside the vault|
|App|Stateless API services, authz on every object|Scale freely; contain the blast radius of any one bug|No server is special; every request re-checks "are you allowed this exact thing?"|
|Integration|Queue + orchestrator between app and runners|Slow and dangerous work leaves the request path and the app zone|Drop a ticket in a bin; something in a different room does the risky work|
|CI isolation|Ephemeral hardware-virtualized sandboxes, one per job, destroyed after|Contain hostile build code and prevent cross-job contamination|Every build gets a fresh disposable room, burned after use, with no door to the next room|
|CI credentials|Short-lived scoped job tokens + OIDC to cloud; no standing secrets in pipelines|Remove long-lived secrets from the most-attacked surface|The job gets a day-pass minted for exactly this run, not a permanent key ring|
|Fork pipelines|No secrets to fork pipelines by default; maintainer approval required to run with secrets|An outsider's MR must not be a secret-vending machine|Strangers' code runs, but it runs blind: no access to your project's secrets unless a maintainer explicitly allows it|
|Secret scoping|Per-project, per-environment secrets; job token cannot reach unrelated projects|Limit lateral movement|A compromised job in project A can't walk into project B|
|Data access|Tier 3/4 on private network, per-service DB credentials, no internet route to data|Contain API-layer bugs|An SSRF or injection in one service reaches only what that service can see|
|Webhook egress|Deny internal address ranges, DNS-rebinding protection, no redirects to private IPs|Kill the SSRF on-ramp|User-set URLs can't be used to knock on the platform's own internal doors|
|Untrusted content|Render repo content (READMEs, filenames, MR descriptions) in a sandboxed context|Repos are attacker-controlled input|Someone's malicious README can't run script in a reviewer's browser|
|Artifact integrity|Signed builds, provenance/attestation, image scanning|Prevent the platform from shipping poison downstream|You can prove an artifact came from the pipeline you think, not an attacker's swap|
|Insider access|Just-in-time elevation with approval, immutable audit logs, no standing prod admin|The highest-leverage attacker is internal|Admin power is checked out for a reason and a window, and every use is recorded where it can't be edited|
|Tenant isolation|Row-level scope checks enforced centrally, tested as a security control|One missing clause is a breach|The invisible wall between customers is code that must be verified, not assumed|
|Platform supply chain|Dependency pinning, SBOM, reproducible builds, multi-party release approval|The platform's own build channel is a master key|Poisoning what we ship poisons every customer at once; that channel gets the most locks|

Pushback, per standing instructions: **adding tiers is not adding security.** A 5-tier system on one flat network with one shared admin credential is a 3-tier system in costume. And "we run jobs in containers, so they're isolated" is the sentence that should end a vendor meeting early: containers are a packaging format that grew some walls, not a security boundary on their own. Cross-tenant CI needs a hardware-virtualization boundary, not just a namespace.

---

## Section 5 — Architecture Diagram (ASCII)

```
LEGEND: [X] component   -->  data flow   ==== tier boundary   (n) tier

              [IdP/SSO]        [Cloud (OIDC)]      [Pkg Registries]
                  ^                  ^                    ^
                  | assertions       | token trade        | deps
==================|==================|====================|=============
(1) CLIENTS       |                  |                    |
  [Browser]   [Git CLI]   [IDE]   [API/Bot]
      \___________|_________|_________/
                  |
========== TB1: INTERNET ============================================
(2) EDGE / WEB
  [CDN] --> [Load Balancer + WAF] --> [Web Servers]
                                          |
========== TB2 ======================================================
(2) APPLICATION
  [Auth/SSO] <-> [API Services] <-> [Git Service]   [Webhook Dispatcher]
                     |                                     |
                     |                                     v (egress filtered)
========== TB3 =======|=============================  [external URLs]
(3) INTEGRATION       v
  [Job Queue] --> [Background Workers]
  [CI Orchestrator] --> [Secrets Broker] --> [Runner Sandboxes]*
        (* TB4: isolated net, no route to tiers 2/4, destroyed per job)
                     |
========== TB5 =======|==============================================
(4) DATA
  [PostgreSQL]  [Redis]  [Git Storage]  [Object Storage]  [Secrets Vault]  [Audit Logs]

  [TB6 = tenant/tenant, logical, spans all data access]
  [TB7 = insider/operator, spans tiers 2-4]
```

---

## Section 6 — Data Flow Diagram, Level 0 (ASCII)

```
 [Developer] --code, clicks, tokens--> +------------------+
 [Developer] <--pages, results-------- |                  |
                                       |   Code Platform  |
 [IdP / SSO] --identity assertion----> |       SaaS       |
 [IdP / SSO] <--auth redirect--------- |                  |
                                       |                  |
 [Cloud/OIDC] <--short-lived token---- |                  |
 [Cloud/OIDC] --temp cloud access----> |                  |
                                       |                  |
 [Customer   ] <--webhook events------ |                  |
 [Systems    ]                         |                  |
 [Pkg Registry] --dependencies-------> +------------------+
```

Level 0 treats the whole platform as one bubble and asks only: who talks to it, and what crosses the line? This is where you inventory every external party you must trust or defend against, the SSO provider, the cloud you mint tokens for, the registries you pull from, before any internal detail distracts you.

---

## Section 7 — Data Flow Diagram, Level 1 (ASCII)

```
 [Developer]
   | creds        | git push        | UI actions       | webhook config
   v              v                 v                  v
 (1.0 Auth) --> (2.0 API/Web) <--> (3.0 Git Svc)   (6.0 Webhook Dispatch)
   |    ^          |    |               |                 | (egress filtered)
   |    |          |    | pipeline      v                 v
   v    |          |    | trigger    [D2 Git Store]   [external URL]
 [D1 Users/       |    v
  Perms]          |  (4.0 CI Orchestrator)
                  |       |    ^
                  |       |    | mint scoped token/OIDC
                  |       v    |
                  |  (5.0 Runner Job) <--> [D5 Secrets Vault]
                  |       |    |
                  |       |    +--> [Cloud via OIDC]
                  |       v
                  |  [D3 Artifacts/Images]
                  v
             [D4 Cache/Queue]
```

Level 1 shows which process touches which data. Auth never touches Git storage. The webhook dispatcher never touches secrets. The runner reaches the vault only through the orchestrator's scoped mint, not directly. Every line not drawn is a rule you can enforce with network policy, and an alarm you can raise when it's violated.

---

## Section 8 — Data Flow Diagram, Level 2: CI Pipeline Execution (ASCII)

```
 (git push / MR event)
    |
    v
 (4.1 Parse CI config) --invalid--> reject, notify user
    | valid
    v
 (4.2 Trust classification) --> is this a TRUSTED context
    |    (branch in the base repo by a member)          |
    |    or an UNTRUSTED context (fork MR, outsider)?   |
    |                                                    |
    | trusted                              untrusted --> go to Section 8a
    v
 (4.3 Authorize secret scope) <--> [D1 Permissions]
    |    (which project/env secrets is THIS job entitled to?)
    v
 (4.4 Mint short-lived job token + OIDC) <--> [D5 Vault]
    |    (scoped to this project/env, expires with the job)
    v
 (4.5 Provision ephemeral sandbox) <-- fresh microVM,
    |     no lateral network, no route to tiers 2/4
    v
 (4.6 Execute job) --> logs stream out (one-way, secret-masked)
    |
    v
 (4.7 Collect artifacts) --> scan --> [D3 Object Storage]
    |
    v
 (4.8 Destroy sandbox + revoke token) <-- always, even on failure
```

Level 2 reveals the decision points the higher levels hide: trust is classified _before_ any secret is touched (4.2), secret scope is authorized _before_ the mint (4.3), tokens are short-lived and revoked at teardown (4.4, 4.8), and logs are secret-masked on the way out (4.6). The most common real mistake in this whole category lives between 4.2 and 4.4: skipping the trust classification and handing every job the project's full secret set.

---

## Section 8a — Data Flow Diagram, Level 2: Fork-MR Pipeline Authorization (ASCII)

This is the second security-critical flow, and the one that most defines the category. An outsider forks your repo, changes the CI config, and opens an MR. What happens when your project's pipeline runs _their_ code?

```
 (MR opened from a FORK by a non-member)
    |
    v
 (A. Classify author) --> member of base project?
    |                          |
    | no (outsider)            | yes --> treat as trusted (Section 8)
    v
 (B. Run WITHOUT secrets by default)
    |    - no CI variables injected
    |    - job token scoped to the fork MR only, read-limited
    |    - no OIDC to cloud
    v
 (C. Maintainer review gate) --> maintainer clicks
    |    "run pipeline with secrets"?  (explicit, per-run)
    |         |                    |
    |         | no                 | yes
    v         v                    v
 (D. Pipeline runs blind)   (E. WARN: you are about to run
    - safe default              untrusted code with secrets)
                                    |
                                    v
                            (F. Run in isolated sandbox,
                                minimal secret scope, full audit)
```

What Level 2 reveals here that Section 8 doesn't: the _default must be deny_. The dangerous path (E/F) exists but is gated behind an explicit human action, a warning, minimal scope, and an audit record. The classic breach is a platform that runs fork pipelines with full secrets automatically "so contributors get green checkmarks." That convenience hands your deploy keys to anyone on the internet willing to open a pull request. Pushback per standing instructions: "we only expose secrets to trusted contributors" is worth nothing if "trusted" is computed from the fork's own metadata, which the attacker controls. Trust must be decided by the _base_ project, about the _author_, at _run time_.

---

## Section 9 — User Journey Flow (ASCII)

```
 [Open site] --> [Login page]
      |
      v
 <Credentials + MFA / SSO valid?>
      | no --> [Error, retry] --(N fails)--> [Lockout / alert]
      | yes
      v
 [Dashboard: sees only permitted projects]
      |
      v
 [git push from laptop] --> <Token valid + write access to ref?>
      | no --> [Push rejected]
      | yes
      v
 [Open Merge Request] (reviewer sees diff, comments)
      |
      v
 <Author trusted?  (member vs fork outsider)>
      | outsider --> [Pipeline runs WITHOUT secrets]
      | member
      v
 [Pipeline auto-runs with scoped secrets] --> <Checks pass?>
      | no --> [Red X, fix, push again]
      | yes
      v
 <Required reviewers approve + protected-branch rules met?>
      | no --> [Changes requested / merge blocked]
      | yes
      v
 [Merge enabled] --> [Merge] --> [Deploy pipeline via OIDC]
      |
      v
 [Notification: deployed] --> [Audit log entry written]
```

Each `< >` is a decision point, and every one is also a security control: login, MFA, ref-level write check, trust classification, required approval, protected-branch enforcement. The journey diagram is secretly the authorization map.

---

## Section 10 — Trust Boundaries

Ranked with a note on where teams actually misallocate effort.

1. **Internet ↔ Edge (TB1).** Credentials, code, API calls cross. All external attacks begin here. Well-invested by most teams (WAF, rate limits).
2. **Edge ↔ Application (TB2).** Filtered traffic crosses; raw internet doesn't. Lets app logic assume some hygiene while still validating everything.
3. **Application ↔ Integration (TB3).** Job specs cross into the zone where untrusted code will run. Matters because it keeps runner workloads out of the app's network.
4. **Platform ↔ Runner sandbox (TB4).** Job definitions and _scoped_ secrets in; logs and artifacts out. **The most important boundary in the system, and the most under-invested.** Hostile code runs inside by design. Teams over-invest at TB1 (the internet edge, where attacks are loud and expected) and under-invest at TB4 (where the attacker is already inside, running code you invited). If effort is misallocated anywhere in this system, it is here.
5. **Integration ↔ Data (TB5).** Scoped queries and blob reads cross. Converts "bug in an API endpoint" from _database breach_ into _one service's narrow view_.
6. **Tenant ↔ Tenant (TB6).** Purely logical, spans every data access. Crossed by a permission check on every query. One missing scope clause is a breach. Under-tested relative to its importance because it has no network line to point a scanner at.
7. **Platform ↔ Operator (TB7).** Separates what employees can reach from what they should, at any given moment. Matters because the insider is the highest-leverage attacker.
8. **Platform ↔ Third parties (TB8).** SSO assertions, OIDC trades, registry pulls, webhook targets. You inherit their compromises; a breached IdP arrives as valid logins.

---

## Section 11 — Threat Model Table

|#|Attacker|STRIDE|Capability|What They Target|Mitigation|
|---|---|---|---|---|---|
|1|External attacker|S|Phishing, credential stuffing|Accounts, then repos and secrets|MFA/SSO, rate limits, WAF, token expiry, anomaly detection|
|2|Unauthenticated user|I, E|Public endpoints, free signup|Public-repo abuse, IDOR probing|Authz on every object, uniform error/timing, abuse detection|
|3|Malicious tenant|E, I|Full account, runs arbitrary CI code|Sandbox escape to other tenants|Hardware-virtualized ephemeral runners, no lateral network, per-job scoped secrets|
|4|Fork-MR abuser|I, E|Opens MR from a fork against a target repo|The target project's CI secrets|Secretless fork pipelines by default; maintainer-approved, minimally-scoped runs (Section 8a)|
|5|Insider threat|I, T, E|Production/admin access; can push platform code|Any tenant's code/data; the release channel|Least privilege, JIT elevation with approval, immutable audit logs, multi-party release|
|6|Compromised dependency|T, E|Malicious code in the platform's own supply chain|Runs with platform privileges|Pinning, SBOM, reproducible builds, prod egress filtering, provenance|
|7|Runner-reuse contaminator|I|Leaves creds/malware for the next job on a reused runner|The next tenant's job on shared hardware|One-job ephemeral runners, full teardown, no shared cache across tenants|
|8|Sandbox escaper|E|Kernel/CPU exploit from inside a job|The host and neighboring jobs|Hardware virtualization (microVM), patched hosts, egress deny, escape detection|
|9|Token/PAT thief|S|Finds a leaked PAT in a repo/log/config|Standing access as the victim user|Short-lived tokens over PATs, secret scanning + auto-revoke, scoped tokens, expiry|
|10|CI-token over-scope abuser|E|Uses an over-broad job token to reach other projects|Lateral movement across projects|Minimal per-job token scope, project-boundary enforcement, token audience checks|
|11|Artifact/image poisoner|T|Injects into a build's outputs|Downstream consumers (the customer and theirs)|Signed builds, provenance/SLSA, image scanning, immutable registries|
|12|Webhook SSRF attacker|E, I|Sets a webhook URL to an internal address|The platform's internal network/metadata|Egress allowlist, block internal ranges, DNS-rebinding protection, no private redirects|
|13|Cryptomining abuser|D|Free-tier signups running mining jobs|Compute cost, availability for real users|CI quotas, job time limits, abuse detection, payment/verification gating|
|14|CI-cache poisoner|T|Writes malicious content to a shared build cache|Later pipelines that trust the cache|Per-scope cache keys, integrity checks, no cross-tenant cache, treat cache as untrusted|
|15|Log/artifact secret-leaker|I|Reads secrets accidentally printed to logs/artifacts|Credentials exposed in output|Secret masking in logs, artifact scanning, forbid secret echo, short-lived creds limit damage|
|16|SSO/OIDC federation attacker|S, E|Exploits loose federation trust config|Login as arbitrary users; cloud access via mis-scoped OIDC|Strict assertion validation, audience/subject pinning, per-project OIDC trust, signed metadata|
|17|Malicious platform update|T, E|Compromises the platform's build/release channel|Every client and tenant at once|Reproducible builds, multi-party approval, signing key custody, provenance|
|18|Repo-content XSS attacker|S, I|Puts hostile script in a README/filename/MR description|Reviewers' browser sessions|Sandboxed rendering, CSP, output encoding, no script execution in rendered repo content|

---

## Section 12 — Threat Diagram (ASCII)

```
 (1) External   (16) SSO/OIDC   (17) Update-channel
      \              |              /
       v             v             v
 +======== TB1: INTERNET ==============================================+
 |  EDGE ZONE:   [CDN]-[LB/WAF]-[Web]     (2) unauth probes here       |
 +======== TB2 ========================================================+
 |  APP ZONE:  [Auth]  [API]  [Git Svc]  [Webhook Disp]                |
 |                ^                            ^                        |
 |     (6) dep ---+                            +--- (12) SSRF           |
 |     (18) repo-content XSS -> reviewer browsers                      |
 +======== TB3 ========================================================+
 |  INTEGRATION:  [Queue] [Orchestrator] [Secrets Broker]              |
 +======== TB4: RUNNER (the blast chamber) ============================+
 |   [Sandbox A]        [Sandbox B]        [Sandbox C] ...             |
 |      ^                   ^                  ^                        |
 | (3) malicious tenant  (4) fork-MR       (7) runner reuse            |
 | (8) sandbox escape    (13) cryptomine   (14) cache poison          |
 |                       (10) token over-scope  (15) log leak         |
 +======== TB5 ========================================================+
 |  DATA ZONE: [Postgres][Redis][GitStore][ObjStore][Vault][Audit]    |
 |                              ^                                      |
 |             (5) insider -----+ (admin path)                        |
 |             (11) artifact poisoning -> [ObjStore] -> downstream     |
 +====================================================================+
   [TB6 tenant/tenant: logical, spans all data access]
   [TB7 operator: spans app->data]   (9) PAT theft: any client token
```

### Section 12a — Threat Diagram Reference Key

|Element|Plain-English Name|What It Represents|Concern It Maps To|Mitigation Category|
|---|---|---|---|---|
|TB1|Front door|Internet-to-platform line|Attackers 1, 2, 16, 17 arrive here|Edge controls, authn|
|TB2|Lobby-to-office door|Edge-to-app line|Bypass of edge filtering; content XSS (18); SSRF (12)|Network policy, CSP, egress filter|
|TB3|Back-office door|App-to-integration line|Keeps runner workloads out of app net|Zone separation|
|TB4|The blast chamber|Platform-to-sandbox line|Attackers 3,4,7,8,10,13,14,15 are _inside_ it|Hardware isolation, scoping, teardown|
|TB5|Vault door|Integration-to-data line|Bulk theft after a service compromise|Scoped credentials, private network|
|TB6|Invisible wall|Tenant separation|Cross-tenant reads (missing scope check)|Row-level authz, security testing|
|TB7|The keyholder line|Operator access path|Insider (5), highest leverage|JIT access, audit, dual control|
|TB8|Trusted neighbors|Third-party links|Inherited compromise (IdP, deps, OIDC)|Assertion validation, vendor review|
|Attacker 9|Loose key|Leaked PAT anywhere|Standing access as a user|Short-lived tokens, secret scanning|
|Attacker 11|Poisoned crate|Tainted build output|Supply chain to customers|Signing, provenance, scanning|

---

## Section 13 — The Hardest Unsolved Problem: Runner Isolation

The one decision that can quietly undo everything else is: **safely executing millions of hostile programs per day on shared hardware.** The 5-tier redesign gives this problem a nicer room; it does not solve it.

Why it's hard, plainly: every other threat is about keeping attackers _out_. This one requires letting them _in_, on purpose, as a paid feature, and betting the company that the walls hold. The walls are CPU and kernel features, which have real, recurring bugs (container escapes, Spectre-class side channels). The system's real isolation is `min(virtualization boundary, host patch level, network containment, secret scoping)`. Get any one wrong once and Tenant A reads Tenant B's deploy keys, which is the exact core promise of the product.

Ranked options, strongest guarantee first:

1. **Ephemeral microVMs (Firecracker-style), one per job, destroyed after.** A hardware-virtualization boundary plus fresh state every job. Cost: you're operating a small internal cloud, with startup latency and real expense. This is the industry answer, and it is the primary path a serious product ships.
2. **Customer-hosted runners for sensitive workloads.** The customer's code runs on the customer's own machines; you never hold that risk for those jobs. Cost: pushes operational and security burden onto customers, and their runner hygiene becomes your reputation problem when it fails. A good _option_, not a good _only_ answer.
3. **Hardened containers with a userspace kernel (gVisor-style).** Cheaper and faster than full VMs. Cost: a weaker boundary than hardware virtualization, historically escapable. Sound as defense-in-depth _inside_ option 1; dangerous as the sole wall.
4. **Dedicated hardware per tenant.** Strongest isolation. Cost: economically impossible for a free tier and defeats the point of multi-tenant SaaS. Sell it as a premium tier, don't build the whole product on it.

The pushback this section owes you: option 3 alone is what a lot of teams _want_ to ship because it's cheap and fast, and "it's containerized" sounds like isolation to a non-specialist. Shipping 3 as the only boundary for cross-tenant untrusted code is the runner-isolation equivalent of shipping email-reset recovery and calling it zero-knowledge. A serious product ships 1 as the foundation, 3 layered inside it, and offers 2 and 4 for customers who need more.

---

## Section 14 — What Is Out of Scope and Why

- **DDoS resilience engineering.** An availability discipline with its own tooling; assumed handled at the CDN/edge. Naming it as a control here would inflate the model with something the vault/CI architecture doesn't own.
- **Physical datacenter security.** Inherited from the cloud provider's compliance boundary.
- **Billing fraud and payment security.** Delegated to the payment processor; PCI scope stays with them.
- **Developer laptop endpoint security.** A stolen laptop with a valid session or SSH key defeats most server-side controls. The platform's only job here is to not make it worse (short-lived tokens, revocation, device alerts).
- **The customers' own deployed applications.** The platform builds and ships their code; whether their code is secure, and what their production does after an OIDC-driven deploy, is theirs.
- **Full depth of the platform's build-pipeline threat model.** Named as attackers 6 and 17 and mitigated at the control level, but the complete pipeline threat model is its own document. This is SPVS territory, not the platform-architecture doc's.

---

## Section 15 — What the Core Security Claim Does NOT Cover

Stated plainly, no hedging. The claim is: _one tenant's code and CI secrets cannot be read or altered by anyone their permissions don't allow, even against a hostile co-tenant running arbitrary code._ It does not cover:

- **Secrets a customer commits into their own repo in plaintext.** The platform stored the file faithfully; the customer leaked the secret. Secret scanning warns, it doesn't un-leak.
- **A leaked personal access token.** To the platform, a valid token _is_ the user. Short lifetimes and scanning shrink the window; they don't eliminate it.
- **Malicious code merged by an authorized reviewer.** Authorization worked; human judgment failed. The review gate is a control on process, not a guarantee of intent.
- **A compromised developer laptop with a valid session or SSH key.** That attacker is indistinguishable from the developer.
- **A breached identity provider issuing valid assertions.** Federation means you inherit the IdP's security.
- **Whatever the customer's own pipeline does to the customer's own production** once the OIDC-driven deploy hands off.
- **Metadata visible to the platform operator.** Repo names, commit timing, the collaboration graph, pipeline frequency, who-shares-with-whom. Zero cross-tenant reads still leaves a rich metadata picture the operator can see.
- **Runner memory while a job is executing.** The live job window contains secrets in use; ephemerality and masking shrink it, they don't close it.

---

## Section 16 — Common Implementation Mistakes in This Category

In rough order of observed real-world frequency:

1. **Fork pipelines run with full secrets automatically.** The single most damaging category-defining mistake: an outsider's MR receives the base project's CI variables. Default-deny plus maintainer approval is the only defensible position.
2. **Leaked long-lived PATs.** Tokens committed to repos, printed in logs, embedded in configs. Prefer short-lived scoped tokens; run secret scanning with auto-revoke from day one.
3. **Over-scoped CI job tokens.** A token that can reach projects unrelated to the job turns one compromised pipeline into lateral movement. Scope to the single project/environment, check the audience.
4. **Reused (non-ephemeral) runners.** State, credentials, or malware from one job survive into the next tenant's job. One job per runner, full teardown, no shared cache across tenants.
5. **Containers treated as a security boundary for cross-tenant code.** Namespaces are not a hardware boundary. Cross-tenant untrusted execution needs virtualization.
6. **Secrets printed to logs or baked into artifacts.** Mask secrets in log streams, scan artifacts, forbid echoing credentials, and keep credentials short-lived so a leak expires fast.
7. **Webhook URLs not egress-filtered.** User-controlled outbound requests reach internal addresses and cloud metadata endpoints. Block internal ranges and defeat DNS rebinding.
8. **Loose OIDC trust to the cloud.** An OIDC trust policy that doesn't pin the exact project and branch lets one repo's pipeline mint another's cloud access. Pin subject and audience tightly.
9. **Missing tenant-scope checks on queries.** The classic missing `WHERE project_id =`. It has no network signature, so it survives scanners and dies only under deliberate authorization testing.
10. **Untrusted repo content rendered without sandboxing.** A hostile README, filename, or MR description runs script in a reviewer's browser. Sandbox rendering, apply CSP, encode output.

---

## Section 17 — Open Decisions Checklist

CI isolation and execution:

- [ ] Runner isolation: microVM vs hardened container vs both layered (recommend: both, VM outer)
- [ ] One-job ephemeral runners vs pooled/reused (recommend: ephemeral, no cross-tenant reuse)
- [ ] Host patch cadence and sandbox-escape detection strategy
- [ ] Runner egress policy: deny-all with allowlist vs open (open is a cryptomining magnet)
- [ ] CI cache: per-scope keying, integrity checks, cross-tenant prohibition

Secrets and tokens:

- [ ] Fork-MR policy: secretless default + maintainer approval; who counts as trusted, decided by base repo
- [ ] CI job token scope model and project-boundary enforcement
- [ ] OIDC-to-cloud trust: subject/branch/audience pinning per project
- [ ] PAT policy: lifetimes, allowed scopes, whether to deprecate in favor of short-lived tokens
- [ ] Secret masking in logs and artifact scanning coverage
- [ ] Secret scanning + auto-revoke on push (including historical commits)

Auth and access:

- [ ] SSO/SAML/OIDC assertion validation and metadata signing requirements
- [ ] MFA policy: always vs new-device; step-up for sensitive actions
- [ ] Protected-branch and required-review enforcement defaults
- [ ] Insider access: standing prod admin vs JIT elevation with approval
- [ ] Session/token lifetime, refresh, and revocation semantics

Tenant isolation and data:

- [ ] Isolation model: shared tables + row-level checks vs schema-per-tenant
- [ ] Authorization test suite as a gating security control (recommend: yes)
- [ ] Tier 3/4 network isolation and per-service DB credential scoping
- [ ] Server-side encryption at rest, backup and snapshot access controls
- [ ] Audit log content policy (forbid secrets/tokens), immutability, retention

Platform surface and supply chain:

- [ ] Webhook egress filtering: internal-range block, DNS-rebinding protection, redirect policy
- [ ] Untrusted repo-content rendering: sandbox + CSP + output encoding
- [ ] Artifact/image integrity: signing, provenance/SLSA level, blocking vs advisory scanning
- [ ] Immutable container registry policy
- [ ] Platform's own build: dependency pinning, SBOM, reproducible builds, multi-party release approval
- [ ] Update-channel signing key custody and rotation
- [ ] Free-tier abuse: CI quotas, job time limits, verification/payment gating