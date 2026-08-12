# Multi-Tenant SaaS: System Design + Threat Model Learning Guide

## Section 1 — What This System Is and Why It Works This Way

Think of this SaaS product like an apartment building. One building, one address, one set of pipes and wiring, but each tenant has their own locked unit and nobody can walk into someone else's apartment just because they live in the same building.

That's what "multi-tenant" means. Instead of building a separate copy of the whole application for every customer (every company that signs up), one shared system serves all of them. Each customer's data sits in the same building, so to speak, but the walls between units have to be real, not just polite suggestions.

Why build it this way instead of giving every customer their own dedicated copy?

- **Cost.** Running one shared building is far cheaper than running a thousand separate buildings, one per customer.
- **Speed of updates.** Fix a bug once, every tenant gets the fix. With separate copies, you'd fix it a thousand times.
- **Operational sanity.** One system to monitor, patch, and scale beats a thousand tiny ones.

Each major part exists because of a specific need:

- The **application tier** exists because someone has to actually run the logic: create a task, assign it, mark it done. This is the "staff" of the building who take requests and do the work.
- The **database** exists because all that data (tasks, users, companies) has to live somewhere durable. This is the building's storage room, shared by everyone but with locked cages per tenant.
- The **authentication service** exists because before staff do anything for you, they need to know who you are and which apartment you belong to.
- The **object storage** (for file uploads, attachments) exists for the same reason as the database: durable storage, but for files instead of structured records.
- The **admin/control plane** exists because someone has to manage the building itself: add new tenants, remove old ones, handle billing. This isn't a tenant function, it's a landlord function.

If any of these were missing, something obvious would break:

- No auth service: anyone could claim to be anyone. There'd be no way to know which apartment a visitor belongs in.
- No isolation logic in the database: every tenant would see every other tenant's data. The walls between apartments would be painted on, not built.
- No admin/control plane: nobody could onboard new customers or shut off access for one that stopped paying, without editing the database by hand.

The single hardest thing to get right in this kind of system isn't any one component. It's making sure the "wall between apartments" is enforced by the building itself (the database), not just by the staff remembering to knock before entering. People forget. Databases don't, if you set them up right.

---

## Section 2 — Why Security Matters Here (Conversational)

Here's the thing a real person using this system would actually worry about: "if my competitor also uses this same software, can they somehow see my company's data?" That's the whole ballgame for a multi-tenant product. Not "can a hacker on the internet break in," although that matters too, but "can the guy in the apartment next door pick my lock because we share a building."

If that wall fails, here's who gets hurt: every tenant whose data leaked, obviously, but also the company that built the product, because "we mixed up customer data" is the kind of headline that ends contracts and lawsuits follow it. This isn't hypothetical. It has happened to real companies, more than once, usually because of one forgotten line of code in one report-generation feature nobody thought was risky.

Why is this harder than it sounds? Because the wall between tenants isn't one wall, it's dozens of small walls, one in the database, one in the file storage, one in the caching layer, one in the background job system, one in the admin tools, one in the logging system. Miss any single one of these and you've got a leak. It's not enough to get 9 out of 10 right.

What does "secure enough" mean here, practically? It means: no code path, anywhere in the system including the boring internal ones like a nightly report job or a support engineer's debug tool, can return or expose data across the tenant boundary. It also means you can prove that's true, not just hope it's true, because "we're pretty sure the ORM always filters correctly" is not a security control, it's a guess.

---

## Section 3 — Architecture Components

**Client-side**

- Web app used by tenant employees (their "front door" into their own apartment)
- Mobile app, same idea
- A separate admin console for tenant admins (higher privilege inside their own tenant only)

**Server-side**

- Edge/API gateway: the front desk that checks ID before letting requests further in
- Auth service: verifies who someone is and which tenant they belong to
- Application/API tier: does the actual work (create task, assign task, etc.)
- Background job workers: does slower work behind the scenes (sending emails, generating reports)
- Control plane: the "landlord's office," handles onboarding, billing, offboarding tenants

**Third-party integrations**

- Identity providers (a tenant's own Okta/Azure AD, for logging in)
- Payment processor (Stripe or similar)
- Email/notification provider
- Tenant-configured outbound webhooks (a tenant tells the system "send updates to this URL of mine")

**Data stores**

- Shared relational database (Postgres), tenant data separated by row-level rules
- Object storage (S3-style) for file attachments, separated by tenant-prefixed paths and access policy
- Cache (Redis), keys namespaced per tenant so one tenant can't read another's cached data
- Logs/observability store, ideally with tenant-scoped access controls

---

## Section 4 — Core Design Decisions

|Layer|Choice|Rationale|Plain-English Explanation|
|---|---|---|---|
|Tenancy model|Shared database for most tenants, separate database for a few high-trust enterprise tenants|Cheap and simple for most customers, with an option for those who need stronger walls|Most apartments share a building. A few tenants pay for their own private building.|
|Isolation enforcement|Database-level row security (RLS), not just app code filtering|If the app code forgets to filter by tenant, the database itself still refuses to leak rows|This is like the locks being built into the apartment doors themselves, not just a rule the staff follows by memory.|
|Tenant identity check|Server figures out which tenant you belong to itself, never trusts a tenant ID sent by the client|If the client could just say "I'm tenant 5" and be believed, anyone could type in any tenant number|Nobody gets to just announce which apartment is theirs. Staff check your ID and look it up themselves.|
|Database connections|Tenant context reset between every use of a shared connection pool|If tenant context "sticks" to a reused connection, the next request could accidentally run as the wrong tenant|Like making sure a hotel key card is fully deactivated before being reprogrammed for the next guest.|
|File storage isolation|Tenant-prefixed storage paths plus access policy at the storage layer itself|If only app code decides who can fetch a file, one bug in that one code path exposes everything|The storage room itself has locked cages per tenant, not just a rule that staff always grab the right box.|
|Stored integration credentials|Encrypted per-tenant, not one shared key for everyone's secrets|One shared key compromised means every tenant's stored secrets are exposed at once|Each apartment gets its own safe, not one big shared safe for the whole building.|
|Background jobs|Re-check tenant authorization at the moment the job actually runs, not just trust what was true when it was scheduled|Things can change between when a job is queued and when it runs; trusting old context is risky|Don't assume the person who asked for a delivery yesterday still lives there today, check again.|
|Support/admin access|Time-limited, logged, tenant-specific access, not permanent all-access accounts for staff|Permanent broad access is both a breach risk and hard to explain to an auditor|Staff need a temporary master key, logged and returned, not a permanent one they keep in their pocket.|

**Pushback**: "we use an ORM that always adds the tenant filter" sounds secure. It isn't, by itself. The moment someone writes a raw SQL query, a reporting script, or connects a BI tool directly to the database, that ORM guarantee disappears. This is the single most common way real breaches like this happen.

---

## Section 5 — Architecture Diagram (ASCII)

```
Legend:
  [C] Client-facing component     [S] Server component
  [D] Data store                  [T] Third-party
  --> data flow direction

           [C] Web/Mobile App          [C] Tenant Admin Console
                  |                              |
                  v                              v
           +-------------------------------------------+
           |         [S] Edge / API Gateway             |
           +-------------------------------------------+
                          |
                          v
           +-------------------------------------------+
           |         [S] Auth Service     <----->  [T] Tenant IdP (Okta/Azure AD)
           +-------------------------------------------+
                          |
                          v
           +-------------------------------------------+
           |     [S] Application / API Tier              |
           +-------------------------------------------+
             |            |              |
             v            v              v
     [D] Shared DB   [D] Object      [D] Cache
     (RLS + tid)      Storage        (namespaced)
             ^
             |
     +-------------------+
     | [S] Background Job |
     |     Workers        |
     +-------------------+
             |
             v
     [T] Email / Notifications, [T] Payment Processor, [T] Tenant Webhooks

     [S] Control Plane (onboarding, billing, offboarding) -- reaches across
     all tenants, sits outside normal request path
```

---

## Section 6 — Data Flow Diagram, Level 0

```
                +------------------+
   Tenant  ---->|                  |----> Task/project data,
   User         |                  |      notifications
                |   THE SaaS       |
   Tenant  ---->|    SYSTEM        |----> Billing events
   Admin        |   (one bubble)   |
                |                  |----> Webhook payloads
   Payment <----|                  |
   Processor    +------------------+
                     ^        |
                     |        v
              Tenant IdP   Email Provider
```

This level teaches: the system as a black box. You don't yet see how it works internally, only who talks to it and what crosses its outer edge. This is useful for spotting every external relationship the system has, before worrying about internal design at all.

---

## Section 7 — Data Flow Diagram, Level 1

```
   Tenant User --> [Auth & Session] --> [Task/Project Process] --> [Shared DB]
                         |                      |
                         v                      v
                  [Tenant Resolution]    [Notification Process] --> Email Provider
                         |                      |
                         v                      v
                  [Authorization Check]   [Job Queue] --> [Background Workers]
                                                 |
                                                 v
                                          [Object Storage]

   Tenant Admin --> [Control Plane Process] --> [Shared DB] (cross-tenant reach)
```

This level adds: the internal processes that were hidden inside the single bubble in Level 0. You can now see that "Auth" and "Tenant Resolution" and "Authorization Check" are separate steps, and that the Control Plane process has a different, wider reach than the normal tenant request path.

---

## Section 8 — Data Flow Diagram, Level 2

Drilling into the most security-sensitive process from Level 1: **Tenant Resolution + Authorization Check**, because this is the exact point where isolation either holds or fails.

```
   Incoming Request (has session token)
           |
           v
   +-------------------+
   | Decode session,    |
   | get user identity  |
   +-------------------+
           |
           v
   +-------------------------+
   | Look up user's tenant   |----> [Shared DB: users table]
   | membership (server-side,|
   | NOT from client claim)  |
   +-------------------------+
           |
           v
   +-------------------------+
   | Set DB session variable |
   | tenant_id = X for THIS   |
   | connection only          |
   +-------------------------+
           |
           v
   +-------------------------+       Decision point:
   | Run query. RLS policy   | ----> if tenant_id mismatch --> 0 rows returned
   | on table filters rows   | ----> if match --> rows returned
   | using session variable  |
   +-------------------------+
           |
           v
   Connection released back to pool --> MUST reset tenant_id before reuse
```

This level reveals what the higher levels hide: the exact moment where a mistake becomes a breach. Level 0 and 1 make it look like "the app checks authorization" is one clean step. Level 2 shows it's actually several fragile steps in a row, and the last one, resetting the connection before it's reused by a different tenant's request, is the one most teams forget to test for.

---

## Section 9 — User Journey Flow

```
   [User opens app]
          |
          v
   [Enters credentials] ---- fail ----> [Error: invalid login] --> back to start
          |
        success
          |
          v
   [Server resolves tenant] ---- no tenant match ----> [Error: account not linked]
          |
        match found
          |
          v
   [Dashboard loads, scoped to tenant]
          |
          v
   [User opens a task] ---- task belongs to another tenant? ----> [403, not found]
          |
        belongs to user's tenant
          |
          v
   [Task detail shown, user can edit/comment]
          |
          v
   [User uploads attachment] --> [Object storage, tenant-prefixed path]
          |
          v
   [Session expires after inactivity] --> [Re-authentication required]
```

Each step is labeled with what the user actually sees: a login screen, an error message, a dashboard, a task page. The two decision points worth noticing: "no tenant match" and "task belongs to another tenant," because those are the two moments where isolation logic is actually doing its job in front of a real user, not just in the database internals.

---

## Section 10 — Trust Boundaries

1. **Internet to edge gateway.** Anyone on the internet can reach this line. Nothing here is trusted yet.
2. **Edge to application tier.** The request is now authenticated, but tenant context isn't fully resolved yet.
3. **Application tier to database.** This is the big one. This is where the "which apartment does this belong to" check either holds or fails.
4. **Application tier to object storage.** A separate wall from the database wall. Getting the database right doesn't automatically get file storage right.
5. **Application tier to third-party integrations.** Tenant-owned secrets (API keys, webhook URLs) cross into the outside world here.
6. **Control plane to data plane.** The landlord's office has reach across every apartment. This is intentionally wider access, which makes it intentionally more dangerous.
7. **CI/CD pipeline to production systems.** Whoever can deploy code can potentially weaken or remove the isolation rules themselves.
8. **Support/admin tooling to tenant data.** Human staff with the ability to look into any apartment, which needs its own separate set of rules.
9. **Background job execution boundary.** The moment a job runs is different from the moment it was scheduled. Trust has to be re-checked, not inherited.

---

## Section 11 — Threat Model Table

|Attacker|Capability|What They Target|Mitigation|
|---|---|---|---|
|External unauthenticated attacker|Credential stuffing, scanning for unauthenticated endpoints|Login flow, any endpoint that doesn't require auth|Rate limiting, MFA, WAF rules, no sensitive data returned pre-auth|
|Compromised/malicious tenant user|Parameter tampering, IDOR attempts against another tenant's records|Task IDs, file IDs, any object reference that might not be re-checked for ownership|Server-side ownership check on every object access, RLS as backstop|
|Malicious tenant admin|Attempts to escalate beyond own tenant's scope, abuses tenant-level integration config|Webhook config, SSO settings, tenant-level API keys|Admin privilege scoped at DB/IAM level to own tenant only, SSRF protection on outbound webhooks|
|Insider threat (employee)|Direct database or infrastructure access, bypassing app-layer rules entirely|Raw DB access, backups, log search across tenants|No standing superuser DB access for engineers, all cross-tenant reads logged and reviewed|
|Compromised dependency / supply chain|Malicious code introduced through a package or CI/CD pipeline|Isolation logic itself, e.g. a migration that quietly disables RLS|Required review on schema/RLS changes, automated drift detection, dependency pinning and review|
|Compromised third-party integration|Attacker gets whatever scope that tenant's stored credential has|That one tenant's data reachable through the compromised integration|Per-tenant scoped credentials, no shared service account across all tenants for one integration type|

**Pushback**: teams often stop at "external attacker" and "insider threat" and call the threat model done. The compromised dependency and compromised integration rows are usually the ones skipped, and they're realistically more likely than a sophisticated external attacker breaking in through the front door.

---

## Section 12 — Threat Diagram (ASCII)

```
   Trust Zone A: Internet (untrusted)
   +--------------------------------------------------+
   | (1) External attacker --> credential stuffing     |
   +--------------------------------------------------+
                     |
                     v  [boundary 1]
   Trust Zone B: Edge / Auth
   +--------------------------------------------------+
   |         Edge Gateway --> Auth Service             |
   +--------------------------------------------------+
                     |
                     v  [boundary 2]
   Trust Zone C: Application Tier
   +--------------------------------------------------+
   | (2) Compromised tenant user --> IDOR attempts      |
   | (3) Malicious tenant admin --> escalation attempts |
   |          Application / API logic                  |
   +--------------------------------------------------+
        |                |                  |
        v [boundary 3]   v [boundary 4]     v [boundary 5]
   Trust Zone D:     Trust Zone E:      Trust Zone F:
   Shared DB          Object Storage     Integrations
   (RLS + tid)         (tenant prefix)    +------------+
                                          | (6) Compromised
                                          | integration  |
                                          +------------+

   Trust Zone G: Internal Ops
   +--------------------------------------------------+
   | (4) Insider threat --> direct DB/infra access      |
   | (5) Compromised CI/CD --> weakens isolation code   |
   |     Support tooling, Control Plane                 |
   +--------------------------------------------------+
                     |  [boundary 6, 7, 8]
                     v
   Reaches across ALL tenant zones (D, E, F) at once
```

### Section 12a — Threat Diagram Reference Key

|Element|Plain-English Name|What It Represents|Specific Concern|Mitigation Category|
|---|---|---|---|---|
|Zone A|The open internet|Anyone, anywhere, unauthenticated|Brute force, scanning|Rate limiting, MFA, WAF|
|Zone B|Front desk|Where identity gets checked|Weak or bypassable auth|Strong auth, session handling|
|Zone C|Office floor|Where the actual app logic runs|Logic bugs that skip ownership checks|Server-side authorization on every request|
|Zone D|Shared storage room (data)|The database, walled per tenant|One missed filter leaking rows|RLS, connection reset discipline|
|Zone E|Shared storage room (files)|Object storage, walled per tenant|App-only gating with no storage-layer backstop|Tenant-prefixed paths plus IAM policy|
|Zone F|Outside contractors|Third-party integrations|Compromised or overly broad credentials|Per-tenant credential scoping|
|Zone G|Landlord's office|Control plane, admin tools, CI/CD, employees|Wide reach across every tenant at once|Logging, time-boxing, least privilege, drift detection|
|(1)-(6)|Numbered attackers|Match the threat model table in Section 11|See table|See table|

---

## Section 13 — The Hardest Unsolved Problem

The design decision where getting it wrong breaks the whole security promise: **making sure every single code path that touches tenant data, forever, without exception, actually applies the tenant isolation check.**

This is hard because it isn't a one-time decision. RLS handles the normal case well. But there will always be a superuser database role needed for migrations, a new engineer writing a raw query who doesn't know about the RLS convention, a reporting tool connected directly to the database, or a background job that was written before the isolation pattern existed. Every one of these is a way to accidentally bypass the wall, and none of them look like an attack when they're written, they look like ordinary engineering work.

Ranked options, strongest to weakest, with what each costs:

1. **Database-enforced isolation (RLS) plus automated drift detection that fails a build if any table loses its RLS policy or any role gains bypass rights.** Cost: real engineering investment upfront, ongoing discipline to never hand out bypass privileges casually, migration tooling has to be built to respect this from day one.
    
2. **Full physical separation, a separate database per tenant.** Cost: the strongest guarantee, but expensive and slow to operate once you have more than a few hundred tenants. Every migration has to run many times instead of once.
    
3. **Application-code-only enforcement (ORM always adds the filter).** Cost: cheapest to build, weakest guarantee. This is where most real breaches in this category actually come from, because it only takes one exception to the pattern.
    
4. **Hybrid: RLS as the default, physical separation reserved for a small number of high-trust tenants who require it, with drift detection covering the whole system.** Cost: the most complex to design and maintain, but it's where most companies that take this seriously actually end up.
    

Pushback: if the only evidence that isolation holds is "we tested it once when we built it," that's not good enough. It has to be checked continuously, automatically, as the system changes, not verified once and trusted forever.

---

## Section 14 — What Is Out of Scope and Why

- **Physical datacenter security.** Handled by the cloud provider's own security program, not something this application design controls.
- **Nation-state attacks on the cloud provider's own infrastructure.** Outside what any single SaaS company's architecture can defend against directly.
- **Tenant's own device security.** If a tenant employee's laptop is compromised, that's their endpoint security problem, not a flaw in this system's tenant wall.
- **Network-level DDoS.** A separate concern handled by edge/CDN capacity, not by tenant isolation logic.
- **Tenant's own identity provider being compromised.** If a tenant's Okta account gets breached, an attacker gets into that tenant the same way a legitimate employee would. This system's isolation design assumes the tenant's own IdP is trustworthy.

---

## Section 15 — What the Core Security Claim Does NOT Cover

This system's core promise is: **Tenant A cannot access Tenant B's data.** It does not promise:

- That Tenant A's own data is safe from Tenant A's own internal mistakes or a malicious insider within that same tenant.
- That a vulnerability affecting only one tenant's own view of the app (like an XSS bug that only fires for that tenant's users) is prevented. That's a real bug, but it isn't a cross-tenant isolation failure.
- That performance is isolated. One tenant hammering the system can slow down another tenant's experience, even if no data crosses the boundary.
- That an authorized employee's legitimate, logged, time-boxed access into a tenant's data is somehow prevented. That access is expected to exist for support purposes; the claim is only that it's controlled and audited, not that it doesn't exist.
- That a backup file, which typically contains every tenant's data mixed together, is automatically as well-isolated as the live system. Backups need their own explicit handling.

---

## Section 16 — Open Decisions Checklist

- [ ] Threshold for offering physical (silo) separation instead of shared database
- [ ] Whether RLS is the only enforcement layer or paired with app-layer double-checking
- [ ] How database connection pooling resets tenant context between uses
- [ ] Whether tenant identity comes from a validated server-side lookup only, or is ever trusted from a client-supplied value
- [ ] Encryption key strategy for tenant-stored third-party credentials (shared key vs per-tenant)
- [ ] Backup isolation and restore process for single-tenant data requests
- [ ] Whether logs are segmented per tenant or searchable across all tenants by staff
- [ ] Per-tenant rate limits and resource quotas
- [ ] Process for time-boxed, logged support/admin access into a specific tenant
- [ ] CI/CD pipeline's actual reach into production data, and whether that reach is separated from deploy access
- [ ] Defined and verified SLA for tenant offboarding and data deletion, including backups
- [ ] Automated check that fails a build if RLS is missing or bypassed on any tenant-scoped table
- [ ] SSRF protections for tenant-configured outbound webhook destinations