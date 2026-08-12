# Building a Password Manager the Way a Real Team Would

### A beginner-friendly walkthrough of modern system architecture

---

## Section 0 — Concepts and Terms You Need First (Vocabulary Primer)

Read this section once, then come back to it whenever a term feels fuzzy. Every term is defined in plain language, and nothing later in the document uses a term that isn't defined here.

### Compute & Orchestration

| Term               | Plain-language definition                                                                                                                                                                                                                           |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Container          | A packaged-up copy of your app plus everything it needs to run (code, libraries, settings). Like a lunchbox: whatever kitchen you open it in, the meal inside is the same. This solves "it works on my machine but not on the server."              |
| Docker             | The most popular tool for building and running containers. When people say "dockerize the app," they mean "put it in a container."                                                                                                                  |
| Kubernetes (K8s)   | Software that runs and babysits many containers across many machines. It restarts crashed containers, spreads them across servers, and adds copies when traffic spikes. Think of it as an autopilot for a fleet of containers.                      |
| Pod                | The smallest unit Kubernetes manages. Usually one container (sometimes a container plus a small helper container) that gets scheduled onto a machine as one unit.                                                                                   |
| Namespace          | A labeled section inside a Kubernetes cluster that keeps groups of pods separate, like folders on a computer. Teams use namespaces to separate apps, environments, or security zones.                                                               |
| Helm               | A package manager for Kubernetes. Instead of writing 15 configuration files by hand to deploy an app, you install a "chart" (a template bundle) and fill in a few values.                                                                           |
| ConfigMap          | A Kubernetes object holding plain, non-secret settings (like "log level = info") so you can change configuration without rebuilding the container.                                                                                                  |
| Secrets Management | The practice (and tools) for storing sensitive values (API keys, database passwords, encryption keys) somewhere locked down, instead of pasting them into code or config files. Examples: Kubernetes Secrets, HashiCorp Vault, AWS Secrets Manager. |
| Ingress            | The Kubernetes component that routes traffic coming from outside the cluster to the right service inside it. The front door of the cluster.                                                                                                         |
| EKS / GKE / AKS    | Managed Kubernetes from Amazon, Google, and Microsoft. The cloud provider runs the hard parts of Kubernetes for you; you just run your pods.                                                                                                        |
| ECS                | Amazon's simpler, non-Kubernetes container runner. Less flexible, less to learn.                                                                                                                                                                    |
| Fargate            | An AWS mode where you run containers without managing any servers at all. You say "run this container," AWS finds a machine.                                                                                                                        |
| Lambda / FaaS      | Functions as a Service. You upload a small piece of code, and the cloud runs it only when an event triggers it, billing per run. No servers, no containers to manage.                                                                               |
| Cold Start         | The delay when a FaaS function hasn't run recently and the cloud has to spin up a fresh copy before your code executes. Can add hundreds of milliseconds to a request.                                                                              |

### Networking & Traffic

| Term            | Plain-language definition                                                                                                                                                                               |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| API Gateway     | A single front door for all API traffic. It checks auth tokens, enforces rate limits, and routes each request to the right backend service. Clients talk to it instead of talking to services directly. |
| Load Balancer   | A traffic cop that spreads incoming requests across multiple copies of a service so no single copy gets overwhelmed and a dead copy stops receiving traffic.                                            |
| Reverse Proxy   | A server that sits in front of your app and forwards requests to it. Load balancers, API gateways, and CDNs are all specialized reverse proxies.                                                        |
| Service Mesh    | A layer that manages service-to-service traffic _inside_ your system: encrypting it, retrying failures, and measuring it, without app code changes.                                                     |
| Istio           | The best-known service mesh for Kubernetes. Powerful and famously complex.                                                                                                                              |
| Envoy           | The high-performance proxy that Istio (and many gateways) use under the hood to actually move the traffic.                                                                                              |
| Sidecar         | A helper container that runs next to your app container in the same pod and handles cross-cutting work like traffic encryption or log shipping. It "rides alongside," like a motorcycle sidecar.        |
| Circuit Breaker | A safety switch in code or a proxy: if calls to a dependency keep failing, stop calling it for a while and fail fast instead. Prevents one sick service from dragging down everything that calls it.    |
| Rate Limiting   | Capping how many requests a client can make per time window (say, 100/minute). Protects against abuse, password-guessing attacks, and buggy clients hammering the API.                                  |
| CDN             | Content Delivery Network. Servers around the world that cache your static files (JavaScript, images) close to users so pages load fast and your servers do less work.                                   |
| Edge Computing  | Running small bits of logic on those CDN servers near the user (bot checks, redirects) instead of at your main data center.                                                                             |

### API Paradigms

|Term|Plain-language definition|
|---|---|
|REST|The most common API style: URLs represent things (`/vaults/123/items`), and standard HTTP verbs act on them (GET reads, PUT updates). Simple, cache-friendly, works everywhere.|
|GraphQL|An API style where the client sends a query describing exactly which fields it wants and gets exactly that back. Great when many different screens need different slices of complex data.|
|gRPC|A fast, strictly-typed API style using binary messages instead of text. Popular for service-to-service calls inside a system; awkward directly from browsers.|
|WebSocket|A long-lived, two-way connection between client and server, so the server can push data to the client instantly instead of waiting to be asked.|

### Messaging & Events

|Term|Plain-language definition|
|---|---|
|Message Queue|A mailbox between services. Service A drops a message in; service B picks it up when ready. If B is down, messages wait safely.|
|Kafka|A heavy-duty event log. Messages are written to named streams ("topics"), kept for days, and many consumers can read them independently. Built for high volume and replay.|
|RabbitMQ|A classic message queue: great at routing individual messages to workers, simpler to run than Kafka, not built for replaying history.|
|Pub/Sub|Publish/subscribe. Senders publish events to a topic without knowing who's listening; any number of subscribers receive them.|
|Event-Driven Architecture|Designing services to react to events ("an item was updated") rather than calling each other directly. Loosens the coupling between services.|

### Data Layer

|Term|Plain-language definition|
|---|---|
|Cache Layer|Fast, temporary memory storage in front of a slower database. You check the cache first; on a miss, you hit the database and save the answer for next time.|
|Redis|The most popular cache server. Stores data in memory, answers in under a millisecond. Also used for rate-limit counters and sessions.|
|ORM|Object-Relational Mapper. A library that lets code work with database rows as normal objects instead of hand-written SQL. Speeds development; can hide slow queries.|
|Database Sharding|Splitting one huge database into several smaller ones by some key (like user ID), because a single machine can no longer hold or serve all the data.|
|ETL|Extract, Transform, Load. Copying data out of production systems, reshaping it, and loading it into an analytics warehouse for reporting.|

### CI/CD & Infrastructure as Code

|Term|Plain-language definition|
|---|---|
|CI/CD|Continuous Integration / Continuous Delivery. Automation that builds, tests, and ships every code change through a pipeline instead of humans copying files to servers.|
|Terraform|The most common IaC tool: you describe your cloud infrastructure (clusters, databases, networks) in files, and Terraform creates or updates the real thing to match.|
|IaC|Infrastructure as Code. Defining infrastructure in version-controlled files rather than clicking around cloud consoles, so it's reviewable, repeatable, and rebuildable.|
|Blue-Green Deployment|Run two identical environments. "Blue" serves traffic while you deploy the new version to "green," then flip all traffic at once. Instant rollback: flip back.|
|Canary Release|Send a small slice of traffic (say 5%) to the new version, watch the metrics, then ramp up gradually. Catches bad deploys before they hit everyone.|

### Observability

|Term|Plain-language definition|
|---|---|
|Observability|Being able to answer "what is my system doing and why?" from the outside, using metrics (numbers), logs (text records), and traces (request paths).|
|Distributed Tracing|Tagging each request with an ID and recording every service it touches, so you can see the full journey of one slow request across ten services.|
|Prometheus|The standard tool for collecting metrics (request counts, error rates, latency) from services in a Kubernetes world.|
|Grafana|Dashboards. Turns Prometheus (and other) data into graphs and alerts humans can read at 3 a.m.|
|ELK Stack|Elasticsearch + Logstash + Kibana: collect logs from everywhere, index them, and search them like Google for your own system.|

### Identity & Access

|Term|Plain-language definition|
|---|---|
|OAuth|A standard way to grant limited access without sharing passwords, like "Sign in with Google" or letting an app read your calendar but nothing else.|
|JWT|JSON Web Token. A signed slip of paper the server hands a client after login. The client shows it on every request; the server can verify the signature without a database lookup.|
|RBAC|Role-Based Access Control. Permissions attached to roles ("admin," "member"), and roles attached to users, instead of granting permissions one by one.|
|mTLS|Mutual TLS. Both sides of a connection prove their identity with certificates, not just the server. Standard for service-to-service traffic inside a mesh.|
|Zero Trust|The security stance that nothing on your network is trusted by default: every request is authenticated and authorized, even between your own internal services.|

### Scale & Multi-Tenancy

|Term|Plain-language definition|
|---|---|
|Horizontal Scaling|Handling more load by adding more copies of a service, rather than buying a bigger machine (that's vertical scaling).|
|Auto-scaling|Letting the platform add or remove those copies automatically based on load (CPU, request rate).|
|Multi-tenant|One deployment of the software serves many customers, with their data logically separated. The opposite is running a separate copy per customer.|
|Cloud Native|Software designed from the start for cloud platforms: containerized, horizontally scalable, config-driven, observable.|

### Mobile Delivery

|Term|Plain-language definition|
|---|---|
|iOS/Android SDK|The official toolkits (Swift/Kotlin) for building native apps for each platform. Best performance and OS integration, but two codebases.|
|React Native|Write your app once in JavaScript/TypeScript and it runs on both iOS and Android with mostly-native UI.|
|Flutter|Google's cross-platform toolkit (Dart language). Also one codebase for both platforms, drawing its own UI.|
|Push Notifications|Messages the server sends through Apple's (APNs) or Google's (FCM) delivery systems to reach a phone even when the app is closed.|
|Offline-first|Designing the app so it works fully from a local copy of the data and syncs with the server when a connection exists, instead of failing without internet.|
|Deep Linking|URLs that open a specific screen inside an app (`kestrel://vault/item/123`) rather than just launching the app's home screen.|
|App Store Optimization|Improving an app's listing (keywords, screenshots, ratings) so people actually find it in the App Store or Play Store.|
|MDM|Mobile Device Management. Software companies use to control employee phones: enforcing passcodes, pushing apps, wiping lost devices. Matters to us because businesses deploy password managers through MDM.|

---

## Section 0.5 — Product and Scope

Everything in this document is about one made-up product, and we never switch examples.

**The product: Kestrel**, a password manager with three clients:

- A **web app** (view and edit your vault in a browser)
- A **mobile app** (iOS + Android, offline-first, with autofill)
- A **browser extension** (autofills passwords on websites, the client people use most)

**The one rule that shapes everything:** Kestrel is **zero-knowledge**. Passwords are encrypted _on the user's device_ with a key derived from their master password. The server only ever stores and moves **ciphertext** (encrypted blobs it cannot read). If Kestrel's database leaked tomorrow, attackers would get scrambled data. This is not optional for a password manager; it's the product.

**Our user:** Maya, a nurse in Austin, `user_id: "usr_71fq"`. Her personal vault is `vault_id: "vlt_8f3a"`.

**The one action we trace all document long:** Maya signs up for a new bank account in her browser. The Kestrel extension offers to save the login. She clicks "Save." The extension encrypts the credentials locally and sends this:

```
PUT /v1/vaults/vlt_8f3a/items/it_29xk
Authorization: Bearer <JWT>

{
  "item_id": "it_29xk",
  "vault_id": "vlt_8f3a",
  "encrypted_blob": "pZk3…9aQ=",     // ciphertext of {site, username, password}
  "nonce": "Yt5w…1c==",              // random value used during encryption
  "key_version": 3,                   // which vault key encrypted this
  "client_updated_at": "2026-07-25T14:03:22Z"
}
```

Response:

```
200 OK
{
  "item_id": "it_29xk",
  "server_version": 47,               // sync counter for conflict detection
  "server_updated_at": "2026-07-25T14:03:22.310Z"
}
```

A moment later, Maya's phone gets a silent push notification and pulls down the new item, so the login is on her phone before she leaves her desk. That save-and-sync loop is the heart of the system.

**Assumed scale (this drives every "is it overkill?" judgment later):**

|Assumption|Value|
|---|---|
|Registered users|500,000 (mostly individuals, some small teams)|
|Active devices|~1.2 million (extension + phone + web per user)|
|Steady traffic|~300 requests/sec|
|Peak traffic|~2,000 requests/sec (morning sign-in wave, sync bursts)|
|Data size|Vault items are small; whole dataset fits comfortably in one Postgres instance (~200 GB with history)|
|Regions|Primary in us-east, second region eu-west for EU users|
|Team|~12 engineers|

**Target complexity level:** mid-size. Big enough that a single server and manual deploys would hurt; nowhere near Google scale. Whenever a pattern in this document only makes sense at 100x this size, I'll say so out loud.

---

## Section 1 — What This System Is and Why It Works This Way

**From Maya's point of view**, Kestrel is simple: every password she saves shows up everywhere, instantly, and autofill just works. Behind that simplicity are four hard promises: her secrets are unreadable to us, sync is fast, the service is basically always up (people are locked out of their lives if a password manager is down), and login is very hard to attack.

Now let's walk the layers and, for each one, ask the honest question: _what breaks without it?_

**Clients that do the cryptography.** The web app, mobile app, and extension all derive an encryption key from Maya's master password on-device and encrypt before sending. Without this, Kestrel would just be a database of plaintext passwords, one breach from catastrophe, and no security-conscious user would touch it. This choice ripples everywhere: the server can't search inside items, can't "reset" a forgotten master password, and clients must be fat (they hold real logic and local data, not just screens).

**Offline-first mobile storage.** The phone keeps an encrypted local copy of the vault. Without it, Maya couldn't open her hotel Wi-Fi password _while standing in the hotel lobby with no Wi-Fi_. Offline-first is why the API is built around _syncing changes_ (versions, conflict detection via `server_version`) rather than just reading live data.

**An edge layer (CDN).** The web app's files and the extension's update bundles are static; a CDN serves them from a server near Maya instead of from us-east. Without it, every page load hauls megabytes across the country, feels sluggish, and our origin servers burn capacity serving files that never change.

**An API gateway.** All three clients hit one front door that validates Maya's JWT, rate-limits her IP, and routes to the right service. Without it, every backend service would re-implement auth checks and rate limiting (guaranteed inconsistency, and password managers are a top target for credential-stuffing attacks, where attackers replay stolen passwords at scale). Rate limiting at the door is a security control here, not just politeness.

**A small set of services on Kubernetes.** Kestrel runs three services: `auth-svc` (login, tokens, 2FA), `vault-svc` (item storage and sync, the workhorse), and `notify-svc` (tells other devices "something changed"). Why not one program? Mostly one reason: `auth-svc` and `vault-svc` have different risk and change profiles: auth code changes rarely and must be reviewed hard; vault sync changes weekly. Why Kubernetes? Because at 2,000 req/s peaks with a 12-person team, you want crashed containers restarted, load spread, and scaling handled by the platform, not by a pager. Without orchestration: a memory leak at 2 a.m. means a human SSHing into a box.

**A message queue between "save" and "notify."** When Maya saves `it_29xk`, `vault-svc` writes to the database, then publishes an event; `notify-svc` reads the event and pushes to her phone. Without the queue, the save request would wait on Apple's push servers, and if push were slow or down, _saving passwords would fail_, which is insane. The queue makes the important thing (the save) fast and reliable and the nice thing (instant sync) eventually happen.

**Postgres plus Redis.** Postgres holds users, devices, and encrypted vault items: data that must never be lost or half-written. Redis holds hot, disposable data: session/rate-limit counters and sync cursors. Without Redis, every request would hit Postgres for token and limit checks, and Postgres would become the bottleneck long before real vault traffic did. Without Postgres's transactional guarantees, a crash mid-save could corrupt a vault, and "we lost your passwords" ends the company.

**CI/CD and IaC.** Every change ships through an automated pipeline; the infrastructure itself is Terraform files. Without this, deploys are scary rituals, environments drift apart, and the EU region can't be rebuilt from scratch, which matters when a compliance question or disaster hits.

**Observability.** Metrics, logs, and traces. Without them, "sync feels slow for Android users in Europe" is an unsolvable mystery instead of a Grafana graph.

**Key design considerations, plainly:** protect ciphertext integrity and availability above all; make the write path (saving) boring and synchronous; make fan-out (notifying devices) asynchronous; keep the service count low because 12 engineers can't feed a zoo; and treat auth as the crown-jewel attack surface.

**Check yourself:**

1. _Why can't Kestrel offer "forgot master password" email resets?_ Because the server never has the key. The key comes from the master password on Maya's device; without it, the ciphertext is unrecoverable, by design.
2. _Why does saving an item publish an event instead of directly calling the push notification code?_ So a slow or failing push provider can never make saving fail or feel slow. The save is the promise; the push is a bonus.
3. _What breaks with no CDN?_ Web and extension load times balloon (all assets ship from one region) and origin servers waste capacity on static files, hurting API headroom during peaks.

---

## Section 2 — Why Modern Architecture Looks This Way (Conversational)

**"Okay, how would a small team have built Kestrel in 2015?"**

Honestly? A Rails or Django monolith on a few rented servers behind one load balancer, Postgres, maybe Redis, deploys via a shell script, monitoring via a ping check and hope. And here's the uncomfortable truth: _that would have worked_ for the first couple of years. Some very successful password managers started exactly like that.

**"So why did the industry move?"**

Because of the specific ways that setup fails as you grow, and every one of these is a real scar somebody has:

- **Deploy risk.** In the monolith-on-servers world, deploying meant replacing the running code in place. One bad deploy at 5 p.m. Friday takes down _everything_: login, sync, all of it. Containers plus canary releases exist because people got tired of deploys being the number-one cause of outages. Now you ship the new version to 5% of traffic, watch the error graph, and roll back with a click.
- **Snowflake servers.** Server #3 has a hand-edited config from 2016 that nobody remembers. It dies; nobody can rebuild it. IaC (Terraform) exists because "the infrastructure is whatever we clicked over the years" eventually bites everyone.
- **Scaling walls.** With traffic spikes (Monday 9 a.m. sign-in wave), you either pay for peak capacity 24/7 or get paged. Auto-scaling on Kubernetes turns that into a dial the platform manages.
- **3 a.m. mysteries.** "The site is slow" with only text logs on five servers is misery. Metrics, tracing, and centralized logs turned debugging from archaeology into looking at a graph.

**"Where do teams overdo it?"**

Everywhere, and this is the part most architecture content won't tell you:

- **Microservices too early.** Teams split into 15 services with 8 engineers and spend their lives on inter-service plumbing instead of product. Kestrel runs _three_ services, and even that is debatable; a modular monolith with clean internal boundaries would be a defensible choice at this scale. We split mainly for the auth-vs-vault risk separation, not for scale.
- **Service mesh before it earns its keep.** Istio manages traffic between dozens of services. With three services, a full mesh is more moving parts than the problem it solves. Kestrel's honest posture: mTLS between services (yes, always, this is a password manager) via a _lightweight_ mesh (Linkerd or Istio's ambient mode), and we consciously skip the fancy traffic-shaping features. I'll draw the mesh in the diagrams because you asked to learn it, but flagging clearly: at 3 services, this is the most skippable box in the picture.
- **Kafka when a simple queue would do.** Kestrel's event volume (one event per vault change) is modest. We use a managed Kafka because we also want replayable history for the audit/event log, which is a real product feature for team vaults. If we didn't need replay, RabbitMQ or cloud Pub/Sub would be simpler and cheaper.
- **Sharding before it's needed.** 200 GB fits in one well-tuned Postgres with a read replica. Sharding now would be solving a 2030 problem with 2026 pain.

**"So what does 'right-sized' mean for Kestrel?"**

It means: containers and Kubernetes (real payoff at our team size), CI/CD and IaC (non-negotiable table stakes), a _small_ number of services, one boring relational database, a queue for the one place async genuinely helps, lightweight mTLS, and observability from day one. And a written list of the things we deliberately _aren't_ doing yet (sharding, multi-cluster mesh, GraphQL) with the triggers that would change our minds.

**Check yourself:**

1. _Was the 2015 monolith wrong?_ No. It was right for its moment. The modern stack exists because of how that setup fails at scale and team growth, not because monoliths can't work.
2. _What's the most common premature adoption here?_ Microservices and service mesh before the service count and team size justify them. The overhead is paid immediately; the benefits arrive only with scale.
3. _Why does Kestrel still choose Kafka over a simpler queue?_ Replayable event history doubles as the audit log for team vaults, a product need, not a scale need. Remove that need and the simpler queue wins.

---

## Section 3 — Architecture Components

Everything below serves Maya's save of `it_29xk` and the sync that follows.

### Client-side components

|Component|Tech choice|What it does for Maya|Why this choice|
|---|---|---|---|
|Browser extension|TypeScript, WebExtension API (Chrome/Firefox/Safari)|Detects login forms, encrypts `{site, username, password}` into `encrypted_blob` with `nonce`, sends the `PUT`, autofills later|Extensions must be per-browser-platform anyway; this is where most usage happens|
|Web app|React SPA|Full vault management UI; does the same client-side crypto in the browser (WebCrypto)|Static bundle → CDN-friendly; crypto in browser preserves zero-knowledge|
|Mobile app|React Native, with native modules (Swift/Kotlin) for crypto, keychain, and OS autofill|Offline-first vault copy in encrypted local storage; registers for push; provides iOS/Android autofill|One codebase for 12 engineers; the security-critical bits (key handling, biometrics, autofill hooks) must be native modules regardless. Pure-native would be better crypto ergonomics but double the client team; Flutter is equally viable, RN chosen for shared TS code with the extension|
|Push channel|APNs (Apple) / FCM (Google)|Wakes Maya's phone to fetch `it_29xk`|The only way to reach a closed app on phones|

Deep linking (`kestrel://item/it_29xk`) lets a "new login saved" notification open the exact item. MDM support matters for team customers deploying the app to staff phones, and App Store Optimization is a marketing concern we note and move past.

### Edge and traffic layer

|Component|Choice|Role|
|---|---|---|
|CDN|CloudFront (or Fastly)|Serves web app bundle, extension assets; absorbs static traffic; TLS termination near the user|
|Edge logic|Small CDN functions|Bot filtering, geo-routing EU users to eu-west|
|API Gateway|Managed gateway (or Kong)|JWT signature check, per-IP and per-user rate limits (credential-stuffing defense), routing `/v1/auth/*` → auth-svc, `/v1/vaults/*` → vault-svc|
|Load Balancer|Cloud L4/L7 LB in front of the cluster ingress|Spreads traffic across gateway/ingress instances, health checks|

### Compute and orchestration layer

|Component|Choice|Notes|
|---|---|---|
|Cluster|EKS, one per region (us-east, eu-west)|Managed control plane; 12 engineers should not operate raw Kubernetes|
|auth-svc|Container, its own namespace|Login (SRP-style password proof, so the master password never travels), 2FA, JWT issuance. Small, slow-changing, heavily reviewed|
|vault-svc|Container, app namespace|CRUD + sync for vault items; the workhorse; scales horizontally on request rate|
|notify-svc|Container, app namespace|Consumes change events, talks to APNs/FCM|
|icon-fn|Lambda (FaaS)|Fetches and caches website icons for vault items. Perfect FaaS fit: bursty, stateless, latency-tolerant. Cold starts don't matter here; they'd be unacceptable on the sync path, which is why the core services are _not_ serverless|
|Helm + ConfigMaps + Secrets|Deployment packaging and config|Secrets come from a real secrets manager (cloud KMS-backed), never plain YAML. For a password manager company, secrets hygiene is reputation|

### Service mesh and inter-service communication

Linkerd (lightweight mesh) provides automatic **mTLS** between the three services and basic retries/metrics via **sidecars**. Inter-service calls (gateway → services, vault-svc → auth-svc token introspection where needed) are REST/gRPC over the mesh. Stated plainly again: at three services this is near the minimum viable justification for a mesh; we adopt it for zero-config mTLS (Zero Trust posture: even inside the cluster, services prove identity), and we skip Istio's heavier machinery.

### Messaging and event-driven components

|Component|Choice|Role|
|---|---|---|
|Event bus|Managed Kafka, topic `vault.item.updated`|vault-svc publishes `{user_id: "usr_71fq", vault_id: "vlt_8f3a", item_id: "it_29xk", server_version: 47}` after each committed write|
|Consumers|notify-svc (push fan-out), audit-writer (append-only audit log for team vaults)|Independent consumers, independent progress; replay supported|

Note the payload: events carry IDs and versions, **never** `encrypted_blob`. Devices fetch ciphertext themselves over authenticated channels. Keeps secrets off the bus entirely.

### Data stores and caching layer

|Store|Choice|Holds|
|---|---|---|
|Primary DB|Managed Postgres (per region), one writer + read replica|`users`, `devices`, `vault_items` (item_id, vault_id, encrypted_blob, nonce, key_version, server_version, timestamps), team memberships (RBAC roles)|
|Cache|Managed Redis|Rate-limit counters, revoked-token list, per-device sync cursors ("device X has seen server_version 46")|
|Object storage|S3|Encrypted vault attachment blobs, backups|

Access via a thin ORM with review discipline on generated queries. No sharding: one region's data fits one Postgres comfortably; the EU region is a _separate_ database (data residency), which is regional partitioning for compliance, not sharding for scale. ETL: nightly jobs copy _metadata only_ (counts, activity, never blobs) to an analytics warehouse.

### CI/CD and infrastructure-as-code

GitHub Actions pipeline: build → test → scan → container image → registry; Terraform for all cloud resources; Argo CD (GitOps) syncs cluster state; **canary releases** for services (blue-green reserved for risky database-adjacent changes). Full pipeline drawn in Section 12.

### Observability stack

Prometheus (metrics) + Grafana (dashboards/alerts), ELK for centralized logs (scrubbed of any sensitive values), OpenTelemetry distributed tracing so one trace ID follows Maya's `PUT` from gateway → mesh → vault-svc → Postgres → Kafka.

**Check yourself:**

1. _Why is icon-fn serverless but vault-svc isn't?_ Icon fetching is bursty and latency-tolerant; cold starts are free to absorb. Vault sync is the core UX; a 500 ms cold start on saves would be felt by every user.
2. _Why don't Kafka events contain `encrypted_blob`?_ Blast-radius control. IDs on the bus mean a compromised consumer or misconfigured topic leaks activity metadata, not ciphertext; devices fetch blobs over the fully authenticated API path.
3. _Is the EU database "sharding"?_ No. It's regional partitioning for data residency. Sharding splits one logical dataset for scale; here each region is its own logical dataset.

---

## Section 4 — Core Design Decisions

|Layer|Choice|Rationale|Plain-English Explanation|Tradeoff / What It Costs You|
|---|---|---|---|---|
|Application structure|3 services (auth, vault, notify), not a monolith, not microservices|Auth needs a different change/review cadence and tighter blast radius than sync; notify needs independent scaling on fan-out|Split where the _risk profiles_ differ, nowhere else|Three deploy pipelines and service-to-service auth to maintain; a modular monolith would be simpler and is a legitimate alternative at this scale|
|Client-facing API|REST|Three very different clients, simple resource model (vaults, items), great caching/tooling, easy to rate-limit per route|URLs for things, verbs for actions; boring and reliable|Some over-fetching vs GraphQL; multiple round trips for complex screens. GraphQL rejected: our data shape is simple and a GraphQL layer can't see inside `encrypted_blob` anyway, killing its main benefit. gRPC rejected client-side: poor browser-extension ergonomics|
|Internal calls|gRPC over mesh mTLS|Typed contracts between services, fast, plays well with sidecars|Strict binary contracts inside, friendly REST outside|Two API styles to maintain; codegen tooling in the build|
|Save→notify path|Event-driven (Kafka)|Saves must never wait on Apple/Google push infrastructure|Do the important thing now, announce it async|Eventual consistency: a phone can lag seconds behind; consumers must handle duplicate events (idempotency)|
|Compute|Kubernetes (EKS), FaaS only at the edges|Steady baseline + spiky peaks fits autoscaled containers; core path can't eat cold starts|Autopilot for containers; functions only for odd jobs|Kubernetes' standing complexity and cost even when idle; genuine learning curve for the team|
|Primary data|SQL (Postgres), no sharding|Strong transactions for vault writes, relational fits users/devices/teams, dataset is small|One boring, trustworthy database per region|Vertical growth ceiling eventually; a future shard-by-user_id migration is real work, accepted knowingly. NoSQL rejected: we'd give up transactions and gain scale we don't need|
|Cache strategy|Redis for counters/sessions/sync-cursors; **no caching of vault ciphertext**|Hot metadata belongs in memory; ciphertext is small, per-user, and security-sensitive, cache adds risk surface for near-zero win|Cache the bookkeeping, not the secrets|Every item fetch hits Postgres; fine at our volumes, revisit only if read latency ever says otherwise|
|Delivery|Canary by default|Bad deploys caught at 5% blast radius|Try the new version on a small slice first|Slower rollouts; need good metrics to judge the canary (observability is a prerequisite, not an accessory)|

**Check yourself:**

1. _Why does zero-knowledge weaken the case for GraphQL?_ GraphQL shines when clients slice complex server-readable data. Kestrel's payload core is an opaque blob the server can't parse, so there's little to slice.
2. _What's the accepted cost of "no sharding now"?_ A known future migration if growth demands it. That's cheaper than paying sharding's complexity tax for years before it's needed.
3. _Why is canary "observability-dependent"?_ A canary only helps if you can tell it's misbehaving. Without error/latency metrics per version, 5% rollout is just a smaller mystery.

---

## Section 5 — Architecture Diagram (ASCII)

```
LEGEND (used in every diagram in this document)
  [ Box ]    = component / service          ( Store )  = data store
  { Topic }  = message queue / topic        --->       = synchronous request (caller waits)
  ~~~>       = asynchronous event/message   ===>       = bulk/streaming data (metrics, logs, backups)
  (n)        = numbered deployment boundary marker (defined in Sections 7–8)

                        MAYA'S DEVICES
  [Browser Extension]   [Web App (React)]   [Mobile App (RN)]
        |                     |                   |                ^
        |  PUT /v1/vaults/... |  static assets    | sync pulls     | push: "vlt_8f3a changed"
        v                     v                   v                |
   -----+---------------------+-------------------+----------  [APNs / FCM]
        |                [ CDN + Edge ]  <-- static bundle          ^
        |                     |                                     |
        v                     v                                     |
              [ Cloud Load Balancer ]                               |
                        |                                           |
                        v                                           |
              [ API Gateway ]   JWT check, rate limits              |
                |           |                                       |
      /v1/auth/*|           | /v1/vaults/*                          |
                v           v                                       |
   ..........................................................      |
   :  KUBERNETES CLUSTER (per region)                        :      |
   :                                                         :      |
   :  [ auth-svc ]<---gRPC--->[ vault-svc ]    [ notify-svc ]-------+
   :   (+ sidecar)             (+ sidecar)      (+ sidecar)  :
   :        |                    |       \           ^       :
   :        |                    |        \          |       :
   :        |                    |         ~~~~~~~~~ | ~~~>  {vault.item.updated}
   :        |                    |                   +~~~~~~~~~~~~~~/    |
   :        |                    |                                       ~~~> [audit-writer]
   :........|....................|.......................................:        |
            |                    |                                                v
            v                    v                                        ( Audit Log store )
     ( Redis )            ( Postgres primary ) ---> ( read replica )
   rate limits,            users, devices,
   sessions, sync          vault_items (ciphertext)
   cursors                       |
                                 ===> ( S3: backups, encrypted attachments )

   [ icon-fn (Lambda) ] ---> external websites (icon fetch)   <--- called by vault-svc, async

   OBSERVABILITY (everything above reports in)
   all services ===> [ Prometheus ] ---> [ Grafana ]
   all services ===> [ ELK ]         traces ===> [ OTel collector ]
```

**Walking the picture.** Maya's three clients all converge on one path: LB → API Gateway → cluster. Static content short-circuits at the CDN and never touches the cluster. Inside the cluster, the gateway routes by URL prefix: auth traffic to `auth-svc`, vault traffic to `vault-svc`; each pod carries a mesh sidecar so every internal arrow is mTLS without app code knowing. `vault-svc` owns the only write path to `vault_items` in Postgres; after a committed save it emits to `{vault.item.updated}` (wavy arrow: nobody waits on it). Two independent consumers read that topic: `notify-svc` (turns events into APNs/FCM pushes, closing the loop back to Maya's phone at top right) and `audit-writer` (append-only history for team vaults). Redis sits beside the request path holding counters and cursors, deliberately _not_ ciphertext. The Lambda handles icon fetching off the critical path. Every component streams metrics/logs/traces to the observability stack; those double-line arrows are high-volume and one-directional. The same picture exists twice, once per region, with EU clients steered to eu-west at the edge.

**Check yourself:**

1. _Which arrow style carries Maya's save, and which carries the "tell her phone" signal?_ The save is `--->` (synchronous, she waits ~100 ms for 200 OK). The phone signal rides `~~~>` through the topic (async, nobody waits).
2. _Why do sidecars appear on every service but the mesh never appears as its own big box?_ The mesh _is_ the sidecars plus a small control plane; it's a property of every connection, not a station traffic stops at.
3. _Where would a credential-stuffing attack hit first, and what stops it?_ The API Gateway: per-IP/per-user rate limits and JWT checks reject the flood before it reaches auth-svc or Postgres.

---

## Section 6 — Data Flow Diagram, Level 0 (ASCII)

Section 5 showed Kestrel's internal organs; this diagram zooms all the way out and treats the whole system as one bubble, so we can see only who talks to it and what crosses the line.

```
LEGEND: same as Section 5

 [ Maya's Extension ]        [ Maya's Web App ]         [ Maya's Mobile App ]
       |    ^                      |    ^                     |    ^
  PUT it_29xk|200 OK          static|vault reads       sync pull| items since v46
       |    | server_version=47    |                          |    |
       v    |                      v                          v    |
  +---------------------------------------------------------------------+
  |                                                                     |
  |                        KESTREL  (the system)                        |
  |                                                                     |
  +---------------------------------------------------------------------+
       |            |                     ^                    |
  push request  icon fetch           icon bytes           metadata-only
  "vlt_8f3a     (GET site            (responses)          analytics export
   changed"      favicon)                                      |
       v            v                                          v
  [ APNs / FCM ] [ External websites ]                [ Analytics warehouse ]
       |
       ~~~> Maya's phone (delivered notification)
```

**What Level 0 teaches that Section 5 didn't:** the _contract with the outside world_. It forces a complete inventory of external actors, exactly four: Maya's clients, Apple/Google push, third-party websites (icons), and the analytics warehouse, and of every data type crossing the boundary. That inventory is where security review starts: notice that ciphertext crosses only on the client edges, push messages carry only "something changed," and the analytics export is metadata-only. Section 5 couldn't make those guarantees visible because it was busy showing internals; Level 0's entire job is the perimeter.

**Check yourself:**

1. _Why is a context diagram useful when Section 5 already exists?_ It's a checklist of external dependencies and boundary-crossing data. Anything not drawn here doesn't get to talk to the system; that's a security statement, not just documentation.
2. _Does Maya's plaintext bank password ever appear on this diagram?_ No. It exists only inside her devices, above the boundary. Only `encrypted_blob` crosses.
3. _Which external actor could break sync-to-phone without breaking saves?_ APNs/FCM. The save path doesn't touch them; only notification delivery does.

---

## Section 7 — Data Flow Diagram, Level 1, With Deployment Boundaries (ASCII)

Now we pop open the Section 6 bubble and show the major processes and stores inside it, and we number every wall that data has to pass through.

```
LEGEND: same as Section 5; (n) = deployment boundary, explained in Section 8

 [Extension] [Web App] [Mobile App]                             [APNs/FCM]
      \         |         /                                          ^
       \        |        /                                           |
        v       v       v                                            |
  ~~(1)~~~~~~~~~~~~~~~~~~~ public internet -> edge ~~~~~~~~~~~~~~(1)~~~
        [ CDN + Edge ]                                               |
              |                                                      |
              v                                                      |
        [ Load Balancer ]                                            |
              |                                                      |
  ~~(2)~~~~~~ | ~~~~~~~ edge -> private network (VPC) ~~~~~~~~~~~~~~ | ~~~
              v                                                      |
        [ API Gateway ]                                              |
              |                                                      |
  ~~(3)~~~~~~ | ~~~~~~~ gateway -> Kubernetes cluster ~~~~~~~~~~~~~~ | ~~~
              |                                                      |
   +----------+-----------+                                          |
   | auth namespace       |  app namespace                           |
   |                 (4)  |                                          |
   |  [ auth-svc ] <--gRPC--> [ vault-svc ]         [ notify-svc ]---+
   |                      |      |      \                ^
   +----------------------+      |       \               |
              |                  |        ~~~(6)~~~> {vault.item.updated} ~~~(6)~~~+
              |                  |                       |                         |
  ~~(5)~~~~~~ | ~~~~~~~~~~~~~~~~ | ~~ cluster -> managed data stores ~~            |
              v                  v                                                 v
         ( Redis )        ( Postgres primary ) --> ( replica )              [audit-writer]
                                 |                                                 |
                                 ===> ( S3 backups )                       ( Audit Log store )

  all services ===(7)===> [ Prometheus / ELK / OTel ]  (observability VPC)
```

Flow of Maya's save: `PUT it_29xk` crosses (1) into the edge, (2) into the VPC to the gateway (JWT + rate limit), (3) into the cluster to `vault-svc`, which crosses (5) to write the row in Postgres, then crosses (6) to publish the event. `notify-svc` consumes across (6) and exits back through (1)'s outbound side to APNs/FCM. Auth flows earlier crossed (4) between namespaces when vault-svc needed token validation help.

**What Level 1 adds that Level 0 couldn't show:** _which internal wall each piece of data crosses_. Level 0 could only say "data enters the system"; Level 1 shows that Maya's request passes through four distinct security zones before touching a database, that ciphertext at rest lives behind (5), and that events and observability data leave the cluster through their own doors, (6) and (7), each of which can be locked down independently. This is the diagram threat modeling and network policy are written against.

**Check yourself:**

1. _How many boundaries does Maya's `PUT` cross before her ciphertext is durably stored?_ Four: (1) internet→edge, (2) edge→VPC, (3) gateway→cluster, (5) cluster→Postgres.
2. _Why is (4) drawn between auth and app namespaces inside one cluster?_ Blast radius. If vault-svc is compromised, namespace-level network policy limits what it can reach in auth's territory; one cluster does not mean one flat trust zone.
3. _Which boundary would data-residency rules care about most?_ (5): which region's managed Postgres the ciphertext lands in determines where EU user data physically lives.

---

## Section 8 — Deployment Boundaries Explained

|#|Boundary|What crosses it|Why it exists|
|---|---|---|---|
|(1)|Public internet → CDN/edge (and outbound to APNs/FCM)|Maya's HTTPS requests (`PUT it_29xk` with JWT), static asset fetches; outbound push requests|The hostile-to-controlled line. TLS terminates, DDoS absorption, bot filtering, geo-routing happen here. Everything outside is untrusted by definition|
|(2)|Edge → private network (VPC)|Only gateway-bound API traffic; nothing else from the internet reaches the VPC|The private network is unreachable except through this one door; shrinks the attack surface to a single, hardened, rate-limited entry point|
|(3)|API Gateway → Kubernetes cluster|Authenticated, rate-limited requests with validated JWTs; trace IDs attached|Cluster ingress only accepts gateway traffic. If the gateway didn't stop garbage, the cluster's own ingress is a second checkpoint (defense in depth)|
|(4)|auth namespace → app namespace|gRPC calls over mesh mTLS (token introspection, key-version lookups)|Kubernetes network policy per namespace: a compromised vault-svc cannot open arbitrary connections into auth's pods. Auth is the crown jewels; it gets its own room, not just its own shelf|
|(5)|Cluster → managed data stores (Postgres, Redis, S3)|SQL over TLS carrying `vault_items` rows (ciphertext), Redis commands for counters/cursors, backup streams|Databases live outside the cluster in the cloud provider's managed layer: separate credentials, separate network rules, encryption at rest, and cluster destruction can't destroy data. Also the line where data-residency (EU vs US) is enforced|
|(6)|Cluster → managed Kafka|`{user_id, vault_id, item_id, server_version}` events; never `encrypted_blob`|The bus is its own trust zone with per-topic ACLs. Keeping ciphertext off it means the messaging layer's compromise leaks activity metadata, not secrets|
|(7)|Cluster → observability VPC|Metrics, scrubbed logs, traces (one-way, high volume)|Telemetry aggregates in a separate zone with its own access control; log pipelines are a classic accidental-secret-leak path, so they're isolated and scrub-enforced|

**Check yourself:**

1. _Why not run Postgres inside the Kubernetes cluster and delete boundary (5)?_ Coupling failure domains: cluster upgrades/misconfigurations could take out the data layer, and stateful workloads in K8s are operationally expensive. The wall is the feature.
2. _Which two boundaries most directly implement Zero Trust?_ (3)/(4): even inside the private network, services authenticate to each other (mTLS) and namespaces restrict who may call whom. Being "inside" buys nothing by itself.
3. _An attacker fully controls the Kafka cluster. What do they get?_ Who changed what item, when (activity metadata). Not passwords: ciphertext never crosses (6).

---

## Section 9 — Data Flow Diagram, Level 2 (ASCII)

The most architecturally dense process from Level 1 is the one crossing the most boundaries: Maya's save traveling gateway → mesh → vault-svc → Postgres → Kafka, so we drill into `vault-svc`'s pod and its immediate neighbors.

```
LEGEND: same as Section 5; boundaries (3)(5)(6) carried over from Section 7

  from API Gateway: PUT /v1/vaults/vlt_8f3a/items/it_29xk  (JWT verified, trace_id attached)
        |
  ~~(3)~| ~~~ into cluster ~~~
        v
  +---------------- vault-svc POD --------------------------------------+
  |                                                                     |
  |  [ Envoy sidecar ]  terminates mesh mTLS, checks caller identity,   |
  |        |            adds retry/timeout policy, records metrics      |
  |        v                                                            |
  |  [ vault-svc app ]                                                  |
  |     1. parse + validate body (nonce present? key_version known?)    |
  |     2. authorize: does usr_71fq own vlt_8f3a? (RBAC check)          |
  |     3. concurrency check: incoming vs stored server_version         |
  |        (stale write -> 409 Conflict, client must re-sync)           |
  |     4. BEGIN TXN ------------------------------------------+        |
  |     5. UPSERT vault_items row (ciphertext, nonce, kv=3)    |        |
  |     6. server_version 46 -> 47; write outbox row           |        |
  |     7. COMMIT ---------------------------------------------+        |
  |     8. respond 200 {server_version: 47}                             |
  |     9. (outbox relay, async) publish event                          |
  |        |                        |                                   |
  +--------|------------------------|-----------------------------------+
           |                        |
   ~~(5)~~ | ~~~                ~~(6)~~ ~~~
           v                        v
   ( Postgres primary )      {vault.item.updated}
   row it_29xk: blob,         ~~~> [notify-svc sidecar]->[notify-svc]
   nonce, key_version=3,            builds push for Maya's other
   server_version=47                devices -> APNs/FCM
```

**Where data transforms:** the request body becomes a validated internal write model (steps 1–3); the write model becomes a database row plus an _outbox row_ in one transaction (steps 4–7); the outbox row becomes a Kafka event (step 9); the event becomes a push payload inside notify-svc. The ciphertext itself is never transformed, only moved: the server treats `encrypted_blob` as an opaque brick from arrival to storage.

**What Level 2 reveals that higher levels hid:** three things. First, the sidecar is doing real work on every hop: identity, retries, metrics happen _before_ app code runs. Second, the version check at step 3 is where sync conflicts are actually decided: two of Maya's devices racing to update `it_29xk` is resolved right here, not somewhere abstract. Third, the **outbox pattern** in steps 6/9: the event is written to Postgres _in the same transaction_ as the data, then relayed to Kafka afterward. Without it, a crash between "committed to DB" and "published to Kafka" would silently strand Maya's phone on version 46 forever. Levels 0 and 1 can't show this; it lives inside one process's transaction boundary, and it's exactly the kind of detail that separates a design that syncs reliably from one that mostly syncs.

**Check yourself:**

1. _Why write an outbox row instead of publishing to Kafka directly after commit?_ Because commit-then-publish can crash in between, losing the event. Putting the event in the same transaction as the data makes "saved but never announced" impossible.
2. _What does a 409 at step 3 mean for Maya?_ Another of her devices updated the item first. Her client pulls the newer version, merges or re-applies, and retries: sync-conflict handling made concrete.
3. _Name two jobs the sidecar did before vault-svc code executed._ Verified the caller's mesh identity (mTLS) and recorded metrics/applied timeout policy (also: retries).

---

## Section 10 — Request Journey Flow (ASCII)

One request, hop by hop: Maya clicks "Save" in the extension.

```
LEGEND: same as Section 5.  <?> = decision point

[Extension]                                            what's happening at this hop
  |  derive key from master password (cached in       client-side crypto: plaintext
  |  session), encrypt -> encrypted_blob + nonce      never leaves this box
  v
[DNS + CDN edge] ---> nearest edge node               TLS handshake near Maya; API
  |                                                    calls pass through, not cached
  v
[Load Balancer]                                       picks a healthy gateway instance
  v
[API Gateway]
  <?> JWT valid & unexpired? --no--> 401 -> extension silently refreshes token, retries
  <?> rate limit ok (usr_71fq / IP)? --no--> 429 -> client backs off (stuffing defense)
  |   yes: attach trace_id, route /v1/vaults/* -> vault-svc
  v
[Ingress -> Envoy sidecar]                            mesh mTLS; caller identity checked
  <?> circuit breaker to vault-svc closed? --open--> fail fast 503 -> client queues the
  |   closed (healthy): forward                       save locally, retries later
  v                                                   (offline-first saves the day)
[vault-svc]
  <?> usr_71fq authorized for vlt_8f3a? --no--> 403
  <?> Redis: session/revocation + sync cursor lookup
  |         (cache hit: ~1 ms; miss -> Postgres, then repopulate)
  <?> server_version fresh? --stale--> 409 -> client re-syncs, merges, retries
  |   fresh:
  v
( Postgres )  TXN: upsert it_29xk + outbox row, version 46->47, COMMIT
  v
[vault-svc] ---> 200 OK {server_version: 47} back up the same path to the extension
  |
  ~~~> {vault.item.updated}                            after response; Maya isn't waiting
         ~~~> [notify-svc] ---> APNs/FCM ---> Maya's phone ---> phone pulls it_29xk
                                                          (total: push typically < 2 s)
```

Elapsed time for Maya: roughly 80–150 ms to the 200 OK. Everything after the wavy arrow is bonus speed she perceives as "magic sync."

**Check yourself:**

1. _The circuit breaker to vault-svc is open. What does Maya experience?_ The save fails fast instead of hanging; her extension stores the change locally and retries when the breaker closes. Offline-first turns an outage into a delay.
2. _Which two decision points are primarily security controls rather than correctness checks?_ JWT validation and rate limiting at the gateway (auth + credential-stuffing defense). The 409 version check, by contrast, is correctness.
3. _Why is the Redis lookup on this path at all?_ Session/revocation and cursor checks happen on every request; serving them from memory keeps Postgres free to do the one job only it can do: the transactional write.

---

## Section 11 — Scaling, Reliability, and Delivery Model

|Concern|Mechanism|Tooling Example|Tradeoff|
|---|---|---|---|
|Handling the 9 a.m. peak (300 → 2,000 req/s)|Horizontal scaling of stateless services; auto-scaling on request rate/CPU|Kubernetes HPA on vault-svc; cluster autoscaler adds nodes|Scale-up lags spikes by a minute or two, so you keep headroom (paying for some idle); scaling is only free because services hold no local state|
|Credential stuffing / abusive clients|Rate limiting at the gateway (per-IP, per-user, stricter on `/v1/auth/*`)|Gateway limits backed by Redis counters|Too strict punishes shared-IP users (hospitals, campuses: Maya's workplace); too loose lets attacks through. Tuning is ongoing, not one-time|
|One sick dependency poisoning everything|Circuit breakers + timeouts on every internal call|Envoy/Linkerd policy|Fail-fast means deliberately rejecting some requests that might have succeeded; thresholds mis-set cause false trips|
|Read pressure on Postgres|Redis for counters/sessions/cursors; read replica for heavy reads; **no ciphertext caching**|Managed Redis, Postgres replica|Cache invalidation bugs, replica lag (a just-saved item read from the replica can be stale, so sync reads pin to primary); consciously accepted extra DB reads for blobs|
|Outgrowing one database|_Not sharding yet_; regional partitioning (US/EU) is compliance, not scale|Managed Postgres per region|The honest tradeoff runs the other way: deferring sharding is cheap now, expensive later if growth is sudden. Trigger point written down: sustained primary CPU/storage pressure that read replicas and tuning can't absorb|
|Cold starts|Core path never touches FaaS; Lambda only for icon-fn|Provisioned concurrency not even needed for icons|If we'd gone serverless for vault-svc, we'd own p99 latency spikes on the product's core promise; this is why we didn't|
|Team vaults on shared infrastructure|Multi-tenant by design: row-level tenancy (`vault_id` scoping), RBAC roles per vault, per-tenant rate limits|Postgres row-level checks in vault-svc; RBAC tables|Pure multi-tenancy is cheap to run but a tenancy-check bug is a cross-customer data leak; enterprise buyers may eventually demand dedicated instances, which we don't offer at this scale|
|Shipping without breaking Maya|Canary by default (5% → 25% → 100% watching error/latency per version); blue-green reserved for schema-risky releases|Argo Rollouts + Prometheus metrics gates|Canary needs solid metrics and slows rollouts; blue-green doubles infrastructure during the window and can bite on DB migrations shared by both colors|
|Losing a region|Per-region full stacks; DNS failover for US; EU data stays in EU by law, so EU failover is within-region redundancy only|Multi-AZ Postgres, cross-AZ cluster|True active-active multi-region is a large complexity step we have not taken; a full us-east regional outage means degraded service for US users, stated plainly in Section 15|

**Check yourself:**

1. _Why does replica lag specifically threaten a password manager's sync?_ A device could read the replica right after a save and miss version 47, concluding it's up to date. Pinning sync reads to the primary trades some scalability for correctness on the core promise.
2. _What's the multi-tenancy nightmare scenario and its first line of defense?_ A scoping bug returning another tenant's items. Defense: every vault-svc query is scoped by `vault_id` + membership check (RBAC), enforced in one shared data-access layer, not re-implemented per endpoint.
3. _When would blue-green beat canary here?_ Releases where two versions can't safely coexist, typically breaking schema changes: flip everything at once, flip back instantly if wrong.

---

## Section 12 — Deployment and Delivery Pipeline Diagram (ASCII)

```
LEGEND: same as Section 5

[Engineer] ---> [Git: commit + PR]
                     |
                     v
              [CI: build & unit tests]
                     |
                     v
              [CI: security scans]        deps (SCA), static analysis (SAST),
                     |                    secret detection, container scan
                     v
              [Build container image] ---> ( Image registry, signed )
                     |
                     v
              [Terraform plan/apply]      infra changes only; reviewed plan
                     |
                     v
              [Argo CD: GitOps sync]      cluster state pulled to match Git
                     |
                     v
              [Canary rollout: 5%]
                     |
        ===> [ Prometheus / Grafana ] <=== error rate, p99 latency, 409/5xx per version
                     |
              <?> canary healthy?
               |            |
              yes           no ---> [auto-rollback to previous version] ---> [alert on-call]
               |
               v
        [Ramp 25% -> 100%]
               |
               ===> [ ELK + OTel traces ]  post-deploy: logs/traces tagged by version
```

### Section 12a — Pipeline Reference Key

|Stage|Plain-English name|What it does|If skipped, in production…|Section 0 tool|
|---|---|---|---|---|
|Git commit + PR|Change proposal|Code review gate; every change has an author, reviewer, and history|Unreviewed auth changes ship; no audit trail when something breaks|CI/CD (the trigger)|
|Build & unit tests|"Does it even work?"|Compiles, runs fast tests on every change|Trivially broken code reaches users; bugs found by Maya instead of machines|CI/CD|
|Security scans|Automated security gate|Checks dependencies for known CVEs, code for dangerous patterns, images and repos for leaked secrets|A vulnerable library or a committed API key ships, in a _password manager_, existential, not embarrassing|CI/CD|
|Container image build|Packaging|Produces the immutable, signed artifact that will run everywhere|"Works in staging, differs in prod" returns; unsigned images open a supply-chain hole|Docker|
|Terraform apply|Infrastructure change|Makes cloud resources match the reviewed code|Click-ops drift: environments diverge, EU region can't be rebuilt, disaster recovery becomes fiction|Terraform / IaC|
|Argo CD sync|Cluster reconciliation|Cluster continuously pulls desired state from Git|Manual kubectl deploys drift from Git; "what's actually running?" has no trustworthy answer|Kubernetes (+ GitOps)|
|Canary 5%|Small-blast-radius trial|New version serves a slice of real traffic under real load|Bad deploys hit 100% of users instantly; deploy fear returns, so deploys get rarer and bigger and riskier|Canary Release|
|Metrics gate|The judge|Compares canary's error/latency against baseline, automatically|Canary is theater: nobody actually checks, bad versions ramp anyway|Prometheus + Grafana|
|Auto-rollback|Undo button|Reverts to last good version without human heroics|3 a.m. incidents wait for a human to wake, read dashboards, and act|Blue-Green/Canary tooling|
|Post-deploy logs/traces|Flight recorder|Version-tagged logs and traces for everything after ramp|Subtle regressions (slow sync for one client type) go undetected until users complain|ELK + Distributed Tracing|

**Check yourself:**

1. _Why is the metrics gate the pipeline's most load-bearing stage?_ Every safety property downstream (canary, rollback) depends on correctly judging health. Remove it and the pipeline still deploys, it just stops protecting anyone.
2. _Which stage most directly defends against supply-chain attacks?_ Security scans plus signed images: known-bad dependencies are caught, and only artifacts the pipeline produced can run in the cluster.
3. _What does GitOps (Argo CD) give that "CI pushes to the cluster" doesn't?_ The cluster converges on Git continuously, so drift self-heals and Git is always the truthful record of what's running.

---

## Section 13 — The Hardest Unsolved Design Problems

Ranked by how much production pain a wrong answer causes.

### 1. Sync conflicts and cross-device consistency (severity: highest)

**Why it's hard, plainly:** Maya edits `it_29xk` on her phone in airplane mode while her extension updates the same item at her desk. Both are "right." The server sees two writes claiming to follow version 46, and because the payloads are _encrypted blobs_, the server cannot merge them: it can't see fields, so "merge the username from A and the password from B" is impossible server-side. Every password manager lives with this; it's the zero-knowledge tax. **Options, ranked:**

1. **Last-write-wins with conflict copies** (recommended): version check rejects the stale write (409); the losing client keeps its version as a visible "conflicted copy" so nothing is silently destroyed. Simple, honest, occasionally annoying. This is what most real products ship.
2. **Client-side field merge:** losing client decrypts both versions locally and merges fields. Better UX, significantly more client complexity, and every client platform must implement it identically or you get divergence bugs.
3. **CRDT-based item format:** mathematically merge-friendly data structures inside the blob. Elegant, heavy engineering, hard to retrofit, overkill for items that change rarely. Revisit only if collaborative team editing becomes core.

### 2. Encryption key rotation at scale (severity: high)

**Why it's hard:** `key_version: 3` exists because keys must be rotatable (master password change, team member removal). Rotating means _clients_ re-encrypting every item, the server can't. For a team vault with 10,000 items and a just-removed employee, you need the remaining members' devices to re-encrypt promptly, while offline devices still hold old-key material. Get it wrong and either ex-employees retain access or users hit undecryptable items. **Options, ranked:**

1. **Lazy rotation with key wrapping** (recommended): items stay under item keys; item keys are wrapped by a vault key; rotation replaces the small wrapping layer immediately (fast, removes the ex-member's access) and re-encrypts item bodies opportunistically. Standard industry practice.
2. **Eager full re-encryption:** strongest posture, but a big vault on a phone on hotel Wi-Fi makes this slow and failure-prone.
3. **Server-side re-encryption:** impossible without breaking zero-knowledge. Listed to be explicit that the easy answer is off the table.

### 3. Mesh and platform complexity vs. a 12-person team (severity: medium, sneaks up on you)

**Why it's hard:** every platform layer we adopted (K8s, mesh, Kafka, canary tooling) is justified individually and _collectively_ consumes a fixed team's attention. The failure mode isn't an outage; it's a year where the platform ate the roadmap. **Options, ranked:**

1. **Aggressively managed everything + lightweight mesh** (recommended, and what this document chose): EKS, managed Kafka/Postgres/Redis, Linkerd not Istio. Pay the cloud provider to hold complexity.
2. **Drop the mesh, keep mTLS via app libraries:** fewer moving parts, but security-critical code re-implemented in three services; reasonable minority position.
3. **Full Istio for future flexibility:** the classic over-engineering trap at three services. Rejected.

---

## Section 14 — What Is Out of Scope and Why

|Not covered|What it is|Why not here|
|---|---|---|
|Cryptographic protocol design|Exact KDF parameters, SRP/OPAQUE choice, cipher suites|A specialist domain deserving its own document and external review; this document treats "client-side encryption" as a settled requirement and teaches the architecture around it|
|Detailed cost modeling / vendor pricing|Monthly bills for EKS, Kafka, egress|Prices change constantly and vary by negotiation; the architecture holds across vendors|
|Org-specific compliance programs|SOC 2 audit mechanics, GDPR paperwork, HIPAA questions from Maya's hospital|Compliance shapes some boundaries (we noted EU data residency) but the programs themselves are legal/organizational work, not architecture|
|Billing, subscriptions, admin consoles|Payment processing, plan management, support tooling|Real services in the real product; architecturally unremarkable and they'd clutter every diagram|
|Anti-phishing / client hardening details|Extension anti-injection defenses, secure enclave usage specifics|Client security engineering, a sibling discipline to this system architecture|
|Detailed incident response / DR runbooks|Backup restore drills, on-call rotations|Operational practice built _on_ this architecture, not part of its design|

---

## Section 15 — What This Architecture Does NOT Guarantee

- **Not zero downtime.** Canary and multi-AZ reduce risk; a full us-east regional failure degrades US service until failover completes. EU has no cross-region escape by design (residency).
- **Not immediate cross-device consistency.** Sync is eventual. A phone can lag seconds (or, offline, hours) behind. Conflicts happen and surface as 409s/conflict copies.
- **Not a cost ceiling.** Auto-scaling means bills scale with traffic, including abusive traffic that gets through before limits trip.
- **Not protection against a compromised client device.** Malware on Maya's laptop sees what Maya sees. Zero-knowledge protects against _server-side_ breach, full stop.
- **Not recovery from a forgotten master password without a recovery kit.** The same property that protects the vault makes it unrecoverable. This is a product promise, not a limitation to be fixed.
- **Not infinite scale.** One Postgres per region has a ceiling. The sharding trigger is defined; the work is not done.
- **Not immunity to tenancy bugs.** Multi-tenant row-level isolation is code, and code has bugs; it is defended, tested, and reviewed, not guaranteed.

---

## Section 16 — Open Decisions Checklist

- [ ] Auth protocol final selection: SRP vs OPAQUE for password-proof login, and 2FA methods at launch (TOTP only? passkeys?)
- [ ] Key hierarchy details: item-key wrapping structure, rotation triggers, recovery-kit format
- [ ] Conflict UX: exact client behavior on 409 (auto conflict-copy vs prompt), identical across all three clients
- [ ] Mesh decision confirmed: Linkerd vs mTLS-in-app-libraries (Section 13 #3, option 1 vs 2)
- [ ] Kafka vs simpler queue: confirm the audit/replay requirement is real enough to keep Kafka
- [ ] Managed vendor picks: gateway (cloud-native vs Kong), Postgres flavor (RDS vs Aurora), Redis provider
- [ ] EU region scope at launch: full stack day one, or US-only launch with EU fast-follow
- [ ] Sharding trigger metrics: the specific Postgres thresholds that start the sharding project
- [ ] Rate-limit numbers: per-route, per-IP, per-user values, and the shared-IP (NAT) exception policy
- [ ] Push strategy details: silent push + pull (assumed here) vs payload-bearing push; APNs/FCM failure fallback (polling interval)
- [ ] Mobile framework final call: React Native (assumed) vs Flutter vs full native, driven by autofill/keychain integration spikes
- [ ] Observability retention and scrubbing rules: log retention windows, PII/metadata scrub list, trace sampling rate
- [ ] Deployment gate thresholds: the exact error/latency deltas that fail a canary
- [ ] Backup/restore RPO and RTO targets, and restore drill cadence
- [ ] Team-vault RBAC model depth: roles at launch (owner/admin/member?) and sharing granularity (vault-level only vs per-item)

---

## Section 17 — Confidence Notes

- **Stated assumptions (Section 0.5), everything downstream inherits them:** the scale numbers (500K users, 2,000 req/s peak, 200 GB), team size (12), and two-region layout. Change these and the right-sizing verdicts (no sharding, lightweight mesh, three services) change with them.
- **Established industry practice, high confidence:** zero-knowledge client-side encryption for password managers; offline-first mobile with push-triggered sync; queue between write path and notification fan-out; the outbox pattern; canary with metrics gates; keeping ciphertext out of caches and event buses; rate limiting as credential-stuffing defense.
- **Reasonable choices with respectable alternatives, moderate confidence:** three services vs modular monolith; Kafka vs simpler queue (hinges on the audit/replay assumption); React Native vs Flutter/native; Linkerd vs library-level mTLS; REST externally + gRPC internally.
- **Deliberately simplified for teaching:** the auth protocol is hand-waved ("SRP-style") and the key hierarchy compressed; real designs here need cryptographic review. Diagrams omit billing/admin/support services entirely.
- **Lowest confidence:** latency figures (80–150 ms, <2 s push) are typical-case estimates, not measurements; canary thresholds and rate-limit values were intentionally left as open decisions because they can only be tuned against real traffic.