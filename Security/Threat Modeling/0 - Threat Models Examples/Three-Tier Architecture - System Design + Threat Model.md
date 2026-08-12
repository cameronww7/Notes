# Three-Tier Architecture: System Design + Threat Model

## 1. What This System Is and Why It Works This Way

A three-tier architecture splits an application into three layers, each with one job:

|Tier|Job|Why It Exists|What Breaks Without It|
|---|---|---|---|
|Presentation|Show pages/screens, collect input|Users need a friendly surface, kept separate from dangerous logic|Nobody can use the system|
|Application|Enforce business rules (pricing, auth, checkout)|Rules change often and must live where users can't edit them|Clients talk to DB directly; rules enforced only by attacker-controlled devices|
|Data|Persist users, products, orders|Data must outlive requests and reboots; safe storage is specialized work|System forgets everything on restart|

Users reach the system through two clients: a **browser** and a **native mobile app**. Both are presentation-tier surfaces running on hardware we don't control.

Why three layers instead of one program: change one tier without breaking others, scale tiers independently, defend tiers separately. Core rule: **tiers only talk to adjacent tiers.** No client ever touches the database.

## 1a. How It All Comes Together: Anatomy of a Modern Request

What actually happens when a user opens the store and the page shows their cart. Numbered, end to end:

1. **DNS.** Browser asks "what IP is store.example.com?" Gets back the load balancer (or CDN) address.
2. **TCP + TLS handshake.** Browser and edge agree on encryption keys. Everything after this is inside the HTTPS tunnel. This is where the padlock comes from, and it only proves _who you're talking to_, not that they're safe.
3. **CDN serves the static shell.** In a modern single-page app, the HTML that comes back is nearly empty: a skeleton plus a `<script>` tag pointing at a large JavaScript bundle. The CDN serves both from a server near the user.
4. **Browser parses HTML into the DOM.** The text becomes a live object tree in browser memory (Section 3a). The screen renders from this tree.
5. **The JS bundle boots.** The framework (React-style) takes over the DOM. From here on, the "presentation tier" is substantially running on the user's machine.
6. **JS calls the app-tier API.** `GET /api/cart` with the user's credential attached: a session cookie (web) or a bearer token (mobile). This crosses TB1.
7. **WAF/LB inspects and routes.** Known-bad patterns dropped, request forwarded to a web server, forwarded to an app server (TB2).
8. **App tier authenticates and authorizes.** Validates the session against the cache, then checks _this user may read this cart_. This is the check that clients can never be trusted to do.
9. **Cache check, then DB.** App server asks the cache for the cart; on a miss it queries the database (TB3), gets rows back, caches them.
10. **JSON response returns.** Not HTML. Raw data: `{"items":[{"title":"Book A","price":1499}]}`. It travels back through the same tunnel.
11. **JS writes the data into the DOM.** New nodes appear in the tree, the browser repaints, the user sees their cart. No page reload happened.
12. **Every subsequent interaction repeats steps 6–11.** Click, fetch, JSON, DOM update.

The mental model to keep: **the server ships behavior once (the JS bundle), then ships data forever (JSON).** After first load, your tiers are talking to a program executing on the user's device. That program is helpful, but it is not trustworthy, because its owner isn't necessarily friendly.

Login variant: steps 6–8 become POST credentials → app tier verifies hash → session created in cache → credential returned as an HttpOnly cookie (web, JS can't read it) or an access + refresh token pair (mobile, stored in Keychain/Keystore).

## 2. Why Security Matters Here

- Customer's worry: card data, reused passwords, order history and address exposure.
- Owner's worry: the database leaking hurts every customer at once, plus the business.
- Why it's harder than it looks: **tiers must trust each other to function, and each trust relationship is a place an attacker can stand.** The database cannot distinguish "app asking for data for a user" from "app asking because an attacker owns it."
- The front door must stay open to everyone on Earth, including every attacker.
- "Secure enough" = one-tier compromise doesn't cascade automatically, detection is fast enough to matter, and boring controls (patching, least privilege, input validation) are actually done.

## 3. Architecture Components

|Category|Components|
|---|---|
|Client-side|Browser (DOM + JS bundle; our code, their hardware), native mobile app (iOS/Android binary, token auth, possible embedded WebViews)|
|Server-side|Load balancer + WAF, web servers (TLS termination, static/SSR), app servers (business logic, session/token validation, JSON APIs), cache (Redis-style)|
|Third-party|Payment processor (tokenized; no raw cards on our servers), email service, push notification providers (APNs/FCM), CDN, app stores (distribution channel)|
|Data stores|Relational DB (users/products/orders), session cache, object storage, encrypted off-site backups|

## 3a. The DOM: What It Is and Why Client-Side Security Lives There

**What it is.** HTML arriving from the server is just text. The browser _parses_ that text into a live tree of objects in memory: the Document Object Model. The screen renders from the DOM, not the HTML file. The HTML is the recipe; the DOM is the cake, and the cake stays editable after baking.

```
  HTML text (from server)          DOM (in browser memory)
  ----------------------           -----------------------
  <html>                                 [document]
    <body>                                   |
      <h1>Cart</h1>          parse         [html]
      <ul id="cart">        ------->         |
        <li>Book A</li>                    [body]
      </ul>                               /      \
      <button>Buy</button>            [h1]      [ul#cart]   [button]
    </body>                             |           |           |
  </html>                            "Cart"    [li]:"Book A"  "Buy"
```

Every tag became a **node**: an object with properties and methods. JavaScript never edits HTML; it edits this tree (`appendChild`, `textContent`, etc.), and the browser repaints. Events (click, submit) fire on nodes; JS listens and reacts. That loop is the entire interactive web.

**Where it sits in the tiers.** The DOM lives on the user's side of TB0/TB1. Nothing on our side can see it, verify it, or protect it. "No business logic in the client" (Section 4) means, concretely: anyone can open DevTools and delete a `disabled` attribute, change a displayed price, un-hide "hidden" form fields, or bypass client-side validation. None of that is hacking; it's a user editing a data structure in their own RAM. Therefore: **the DOM is a rendering of state, never the authority on state.** The charged price comes from the database, not the screen.

**SPA consequence.** In server-rendered apps, every click rebuilds the DOM from fresh server HTML. In single-page apps, the server sends one shell + JS bundle, and JS rewrites the DOM continuously from API JSON. Part of the presentation tier has physically relocated into the browser, so the enforceable boundary is no longer "pages we render" but **"APIs we expose."** Every authz check must live on the API, because the JS deciding what to display runs on the attacker's machine.

**XSS in DOM terms.** XSS = an attacker getting the browser to parse their input into _script nodes_ instead of _text nodes_. A review containing `<script>steal()</script>` inserted as raw HTML becomes an executing script node with the page's full authority: read the whole DOM, fire requests with the victim's session, redraw the page as a fake login form.

|Variant|Path|Who Can See It|
|---|---|---|
|Stored/reflected XSS|Server embeds attacker input into HTML it sends; victim's browser builds the poisoned DOM|Server-side logs/WAF have a chance|
|DOM-based XSS|Page's own JS writes attacker-controlled input (e.g. `location.hash`) into the DOM via `innerHTML`|**Never touches the server.** WAF and logs are blind|

**The mitigation, precisely.** Output encoding is a DOM instruction: encoding `<` as `&lt;` tells the parser "build a text node, not an element node." In code: `element.textContent = input` always produces text nodes (safe); `element.innerHTML = input` invokes the parser (dangerous). Encoding isn't string cleanup; it's controlling **what kind of node input is allowed to become.** CSP (Content Security Policy) is the backstop: a response header telling the browser which script sources may execute even if a script node sneaks in.

One-sentence summary: _the DOM is the user's private, fully editable copy of your UI, so it can display security but never enforce it, and XSS is anything that lets an attacker write to someone else's copy._

## 3b. The Mobile App Client: Same Rules, Different Wrapper

A mobile app occupies **the same trust position as the browser: minus the DOM, plus a decompilable binary.** Everything Section 3a says about client-controlled state still applies. What changes:

|Dimension|Browser|Mobile App|Security Consequence|
|---|---|---|---|
|UI model|DOM tree, parsed from HTML|Native view hierarchy, compiled UI|No XSS in native views|
|WebViews|n/a|App may embed a browser engine for some screens|**Reintroduces the entire DOM/XSS surface inside the app.** Most-missed mobile finding|
|Session credential|HttpOnly cookie (JS can't read it)|Bearer access token + refresh token|Token storage (Keychain/Keystore), lifetime, and revocation become explicit design problems|
|Code visibility|JS is readable but at least served fresh|APK/IPA is downloadable and decompilable by anyone|Every "hidden" endpoint, hardcoded key, and client-side check is public. **No secrets in the binary, ever**|
|Transport|TLS + trusted CA store|TLS + optional certificate pinning|Pinning blunts MITM on hostile networks; costs operational pain on cert rotation|
|Distribution|We serve the code|App store serves the code|App store review and account are a supply chain hop; signing keys become crown jewels|
|Extra third party|—|Push notifications via APNs/FCM|New TB4 flow; never put sensitive data in push payloads|
|Device reality|Shared/borrowed machines|Lost/stolen devices, rooted/jailbroken devices|Biometric unlock, short-lived tokens, remote revocation; decide root-detection posture|

Architecturally, the mobile app calls **the same app-tier APIs** as the browser JS (steps 6–11 in Section 1a are identical from the app tier's perspective). That's the payoff of the SPA shift: once your enforceable boundary is the API, adding a client is cheap and doesn't add trust, because you never trusted clients to begin with.

## 4. Core Design Decisions

|Layer|Choice|Rationale|Plain-English Explanation|
|---|---|---|---|
|Presentation|Clients talk only to edge over HTTPS|Single controlled entry point|One front door is easier to guard than ten|
|Presentation|No business logic or secrets in any client|Clients are attacker-controlled (DOM editable, binary decompilable)|Anything on their device can be read and modified|
|Presentation|Output encoding + CSP for all user-influenced content|Control what node type input becomes|User text stays text; it never becomes script|
|Application|All rules and authz enforced at the API|Only tier we fully control; clients just render|The referee stands on our field|
|Application|Stateless app servers, sessions in cache|Scaling and failover|Any server handles any request|
|Application|Cookies (HttpOnly/Secure) for web, short-lived bearer tokens + refresh for mobile|Fit each client's storage model|Right credential shape for each pocket it lives in|
|Application|Payment tokenization|Shrinks breach and compliance scope|We hold a claim ticket, not the card|
|Data|DB accepts connections only from app subnet|Network containment|The vault opens for one hallway only|
|Data|Encryption at rest + TLS in transit|Stolen disks and sniffed traffic|Theft yields gibberish|
|All|Adjacent-tier communication only|Limits blast radius|No room has doors into every other room|

## 5. Architecture Diagram

```
 LEGEND:  [X] our component   (X) third party   --> data flow
          ==== trust boundary (numbered, Section 10)

              USER DEVICES (TB0: our code, their hardware)
   [Browser: DOM + JS bundle]        [Mobile App: binary + tokens]
        |            \                    |            ^
        |             +--> (CDN) <--------+            |
        | HTTPS                | static     (Push: APNs/FCM)
        |                      | assets           ^
========|======================|==================|====== TB1: Internet -> Edge
        v                      v                  |
      [Load Balancer + WAF] <---------------------|-- (mobile API calls too)
            |                                     |
            v                                     |
      [Web Servers]  <-- presentation tier        |
            |                                     |
============|===================================== TB2: Web -> App
            v                                     |
      [App Servers]  <------> [Session Cache]     |
        |    |    \-------------------------------+
        |    |     +--------> (Payment Processor)   } TB4
        |    |     +--------> (Email Service)       }
        |    |     +--------> (Push Providers)      }
========|====|===================================== TB3: App -> Data
        v    v
   [Database] --> [Encrypted Backups]
        |
   [Object Storage]

   [Admin/Ops] --(VPN + MFA)--> app & data tiers    } TB5
```

## 6. Data Flow Diagram, Level 0

```
 [Customer (web/mobile)] --creds, searches, orders--> +-------------+
 [Customer (web/mobile)] <--pages/JSON, confirmations-|             |
                                                      |   ONLINE    |
 [Admin] --catalog updates, config------------------> |  BOOKSTORE  |
 [Admin] <--reports, logs---------------------------- |   SYSTEM    |
                                                      |             |
 (Payment Processor) <--payment token---------------- |             |
 (Payment Processor) --approve/decline--------------> |             |
                                                      |             |
 (Email Service) <--outbound messages---------------- |             |
 (Push Providers) <--notification requests----------- +-------------+
        \--> delivers to [Customer mobile device]
```

Teaches the system's edges: who is outside and exactly what crosses in and out. Every arrow is a potential leak or lie before we know what's inside the box. Note push notifications exit our system and re-enter the customer's device through a third party we don't control.

## 7. Data Flow Diagram, Level 1

```
[Web/Mobile Client]--creds-->(2.0 Authenticate)----reads---->[D1 Users]
                        |  cookie or token pair
                        v
[Web/Mobile Client]-browse->(1.0 Handle Requests)--reads---->[D2 Products]
                        |                 --r/w------------->[D4 Session Cache]
                        | checkout
                        v
                    (3.0 Process Orders)---writes----------->[D3 Orders]
                        |        |
                        |        +--token--> (Payment Processor)
                        |        +--send---> (Email Service)
                        |        +--notify-> (Push Providers)
                        v
[Admin]---updates-->(4.0 Manage Catalog)---writes----------->[D2 Products]
```

Both clients converge on the same processes: one API surface, one set of authz checks, two renderers.

## 8. Data Flow Diagram, Level 2: Authenticate User

Most security-sensitive process: authentication, the gate everything else hides behind.

```
[Client]--email+password-->(2.1 Validate Input)
                                  | well-formed?
                          no ---> reject (generic error)
                                  | yes
                                  v
                             (2.2 Rate-Limit Check)<-->[D4 Cache: attempt counts]
                          over--> reject + lockout/delay
                                  | under
                                  v
                             (2.3 Look Up User)---read-->[D1 Users]
                                  |
                                  v
                             (2.4 Verify Password Hash)
                          fail--> record attempt, generic error
                                  | pass
                                  v
                             (2.5 Create Session)--write-->[D4 Cache]
                                  |
                                  v
                    web: HttpOnly Secure cookie
                    mobile: access token (short TTL) + refresh token
```

Reveals internal decision points hidden at higher levels: skipping 2.2 enables password spraying; 2.4 must compare bcrypt/argon2 hashes, not plaintext; errors at 2.1 and 2.4 must be identical so attackers can't enumerate accounts. The mobile branch adds a refresh flow: long-lived refresh token exchanges for short-lived access tokens, and revoking the refresh token is your kill switch for a stolen device.

## 9. User Journey Flow

```
[Open site or app] -> [Browse/search]  (no account needed)
                     |
                [Add to cart]    (session cart)
                     |
                [Checkout clicked]
                     |
              < Logged in? >
               no |      | yes
                  v      |
      [Login form or     |
       biometric (mobile)|
      < Valid? >         |
         no |     | yes  |
            v     +------+
   [Generic error,       |
    retry, lockout       v
    after N tries]  [Enter shipping]
                          |
                    [Payment form (processor-hosted fields/SDK)]
                          |
                    < Payment approved? >
                     no |         | yes
                        v         v
              [Decline msg,  [Order confirmed screen]
               retry/other        |
               method]      [Confirmation email + push notification]
```

## 10. Trust Boundaries

|#|Boundary|What Crosses|Why It Matters|
|---|---|---|---|
|TB0|Client runtime (our code on their hardware)|Our JS bundle / app binary, all rendered data, user input|The DOM is editable and the binary is decompilable; nothing here can enforce anything. Everything client-side is display, not control|
|TB1|Internet -> Edge|Anonymous global traffic, API calls from both clients|Everything before this line is hostile by default|
|TB2|Web -> App|Parsed requests into logic/secrets zone|Compromised web server shouldn't equal compromised logic|
|TB3|App -> Data|Queries to crown jewels|Last gate; DB inherently trusts whoever crosses|
|TB4|App -> Third parties|Payment tokens, email content, push payloads|Their breach becomes ours; we can't audit their insides. Push payloads transit infrastructure we don't own|
|TB5|Admin -> All tiers|Elevated human access|Bypasses the tier model on purpose; most valuable path to steal|

TB0 is the unifying concept for client security: DOM manipulation, DevTools edits, decompiled binaries, and WebView XSS are all the same fact viewed from different angles: our code executes where we have no authority.

## 11. Threat Model Table

|Attacker|Capability|What They Target|Mitigation|
|---|---|---|---|
|1. External attacker|Any traffic to public endpoints, tooling at scale|Login (credential stuffing), inputs (SQLi/XSS), session cookies/tokens|WAF, parameterized queries, output encoding + CSP, rate limiting, MFA, HttpOnly/Secure cookies, short token TTLs|
|2. Unauthenticated user|Legit-looking probing, ID guessing|Endpoints missing authz, direct object references|Deny-by-default authz, object-level checks, unguessable IDs, authz tests in CI|
|3. Insider threat|Valid credentials, internal knowledge|Bulk export, quiet privilege abuse, log tampering|Least privilege, separation of duties, immutable off-host audit logs, access reviews, bulk-read alerting|
|4. Compromised dependency|Malicious code inside app tier with app's permissions|Secrets, DB credentials, exfiltration|SCA + lockfiles, pinning, provenance checks, minimal runtime perms, egress filtering, pipeline controls|
|5. Attacker with the client|DevTools on the DOM, or decompiled APK/IPA; full read/write of everything client-side|Client-enforced logic, hidden/undocumented APIs, hardcoded secrets, tampered prices/params, WebView injection points|Server-side enforcement of everything, authz on every API, no secrets in clients, signed/verified requests where warranted, WebView hardening, cert pinning|

## 12. Threat Diagram

```
   (A5 owns a client: DevTools / decompiler)
        |
   [Browser DOM]  [Mobile binary]     <-- TB0 zone: fully attacker-readable
        \             /
         (A1 External)  (A2 Unauth user)
              \             /
               v           v
 +---------------- INTERNET ----------------+
 |TB1                                       |
 |  [WAF/LB] -> [Web Servers]               |
 |TB2 ======================================|
 |  [App Servers] <== (A4 lives here if a   |
 |     |               bad package ships)   |
 |     +----> (Payment)(Email)(Push) <-TB4  |
 |TB3 ======================================|
 |  [Database] [Backups]                    |
 |       ^                                  |
 +-------|----------------------------------+
         |
   (A3 Insider) --VPN/TB5--> can reach app + data zones
```

### 12a. Threat Diagram Reference Key

|Element|Plain-English Name|Represents|Specific Concern|Mitigation Category|
|---|---|---|---|---|
|A1|External attacker|Anyone on the internet|Breaking in through public surface|Edge hardening, input handling, authn|
|A2|Unauthenticated user|Visitor without account|Reaching data without logging in|Authorization, deny-by-default|
|A3|Insider|Employee/contractor|Abusing granted power quietly|Least privilege, audit, monitoring|
|A4|Compromised dependency|Malicious library in app tier|Attacker code already past TB1/TB2|Supply chain, egress filtering|
|A5|Attacker with the client|Anyone with DevTools or a decompiler|Client-side "controls," secrets in binaries, hidden APIs|Server-side enforcement, no client secrets|
|TB0|Client runtime boundary|Our code on user hardware|Display, never enforcement|API-side authz, encoding, no trust in clients|
|TB1-TB5|Trust boundaries|Trust-level change lines|Each checkpoint can be bypassed|Boundary-specific controls (Sec. 10)|
|WAF/LB|Front door filter|Traffic inspection/routing|Only blocks what it understands; blind to DOM-based XSS|Edge/network controls|
|Database|The vault|All persistent data|Trusts app tier completely|Data-layer controls (Sec. 13)|

### Standing-Instruction Callouts

- Relying on the WAF sounds secure but isn't: it blocks known patterns, doesn't understand business logic, and is completely blind to DOM-based XSS, which never touches the server.
- "Internal network = trusted" sounds secure but isn't: if TB2/TB3 are drawn but unenforced, three tiers are one tier with extra latency.
- "It's compiled, so it's hidden" sounds secure but isn't: decompilers make the mobile binary as readable as DevTools makes the DOM. Same lesson as hidden form fields, sharper teeth.
- Most common implementation mistake in this category: authorization gaps (IDOR). Endpoints check who you are but not whether the object is yours. The SPA/mobile shift makes this worse: more API surface, and client code that "hides" endpoints hides nothing.
- Most-missed mobile finding: WebViews. Teams threat-model the native app, forget the embedded browser, and ship XSS inside a "native" binary.

## 13. The Hardest Unsolved Problem

**The database trusts the application tier completely.** The app tier holds a broad credential because it needs broad access to function. Anyone who owns the app tier (dependency compromise, RCE, stolen deploy key) gets effectively all data using legitimate credentials, breaking the core claim at TB3. Hard because every fix fights the architecture: centralized data access is the app tier's purpose. Adding the mobile client changes nothing here, which is itself the lesson: clients were never trusted, so adding one doesn't move the hard problem.

Ranked options:

|Rank|Option|Tradeoff|
|---|---|---|
|1|Split DB accounts by function, least privilege each|Real containment, moderate effort; account sprawl, migration pain; accounts still plural-user-scoped|
|2|Row-level security tied to end-user identity|Strongest containment; performance and design cost, hard retrofit, pooling complications|
|3|Stored procedures / query allowlisting|Kills exploit classes; slows development, pushes logic into the DB|
|4|Decompose into services with separate stores|Best isolation ceiling; buys a distributed system's new failure and security modes|

Minimum bar: option 1. Otherwise the answer to attacker A4 is hope.

## 14. Out of Scope and Why

|Item|Why Excluded|
|---|---|
|Volumetric DDoS|Network-layer problem, solved upstream, not by app architecture|
|End-user device security|Can't control customer hardware or OS; limit blast radius only (short sessions/tokens, re-auth, revocation)|
|Mobile OS internals and app store review process|Apple/Google's domain; we inherit their sandboxing and their review, and control neither|
|CI/CD pipeline internals|Its own threat model; here we assume gated deploys and protected signing keys|
|Regulatory mapping (PCI, GDPR)|Compliance consumes the design; control mapping is separate work|
|Fraud / business-logic economics|Fought with fraud tooling and policy, not tiers|
|Physical / cloud-provider security|Inherited via shared responsibility model|

## 15. What the Core Security Claim Does NOT Cover

Claim: tier separation limits blast radius so one component's compromise doesn't hand over the system. It does not cover:

- App-tier compromise still exposes most customer data (Section 13)
- Stolen valid credentials or tokens crossing every boundary legitimately
- Malicious insiders using access granted on purpose
- Anything on the user's device: malware reading the DOM or app memory, stolen unlocked phones
- Breaches at the payment processor, email provider, or push provider
- Business-logic abuse via actions allowed by design
- Availability

## 16. Open Decisions Checklist

- [ ] Web session storage: server sessions vs JWT; lifetime; revocation story
- [ ] Mobile token strategy: access TTL, refresh TTL, rotation, revocation on device loss
- [ ] Token storage on device: Keychain/Keystore policy, biometric gating
- [ ] Certificate pinning: yes/no, and the cert rotation runbook if yes
- [ ] WebView policy: allowed at all, allowed origins, JS bridge exposure
- [ ] Root/jailbreak detection posture: block, degrade, or ignore
- [ ] MFA policy: required/optional/risk-based; factors
- [ ] CSP: enforced vs report-only, and script source allowlist ownership
- [ ] DB account model: per-service least privilege minimum (Sec. 13 opt. 1)
- [ ] Secrets management: storage, rotation, nothing in code, env files, or client bundles/binaries
- [ ] App signing key custody and release process
- [ ] Push payload policy: what may never appear in a notification
- [ ] Encryption: at-rest scope, field-level for sensitive columns, key management ownership
- [ ] Logging: per-tier capture, PII handling, retention, off-host shipping
- [ ] Dependency policy (server AND mobile SDKs): lockfiles, pinning, SCA gate, update SLA
- [ ] Egress filtering rules for app tier
- [ ] Admin access path: VPN vs bastion vs zero-trust proxy; session recording
- [ ] Backup keys separate from prod; restore testing cadence
- [ ] Rate-limit thresholds per endpoint class
- [ ] WAF mode (block vs monitor) and tuning ownership
- [ ] Data classification and differential handling
- [ ] Object storage access: signed URLs vs proxied downloads
- [ ] API versioning strategy for old mobile app versions in the wild
- [ ] Incident response: TB3 alert ownership and containment runbook