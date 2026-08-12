## Core Concepts

**Microservices** — Breaking one big application into small, independent pieces that each do one job. Like a restaurant with separate stations for grilling, salads, and desserts instead of one cook doing everything. Companies use this so teams can update their piece without breaking everyone else's.

**Serverless** — You write code, the cloud provider handles the server. Like renting a food truck by the hour instead of owning a restaurant building. You pay only when your code actually runs. Companies use this to avoid managing infrastructure for code that runs occasionally.

**K8s (Kubernetes)** — A system that manages hundreds of containers for you: starts them, restarts them if they crash, spreads them across machines. Like an air traffic controller for your app's moving parts. Companies use it so nobody has to manually babysit servers.

**Backend** — The part of an app the user never sees: the database, the logic, the servers. Like a restaurant kitchen versus the dining room. Companies build this to handle data and business logic securely, away from the user's browser.

**Middleware** — Code that sits between two systems and does something to the traffic passing through, like checking a login before letting a request reach the app. Like a bouncer checking IDs before you get into the club. Companies use it to handle repetitive tasks (auth, logging) in one place instead of every service.

**Mobile** — Apps built specifically for phones and tablets, with their own rules for screen size, offline use, and app stores. Companies invest here because that's where most users actually are.

## Containers and Orchestration

**Container** — A packaged bundle of your app plus everything it needs to run, so it works the same everywhere. Like a shipping container: same box, works on any ship, truck, or train. Companies use this to stop the "it worked on my machine" problem.

**Orchestration** — The automated management of many containers: deciding where they run, restarting failed ones, scaling up under load. Like a conductor keeping an orchestra of 50 musicians in sync.

**Docker** — The most popular tool for building and running containers. Think of it as the shipping company that makes the containers.

**Helm** — A tool that packages Kubernetes configurations into reusable templates, like a recipe card for deploying an app instead of writing every step by hand. Companies use it to avoid copy-pasting config files everywhere.

**Pod** — The smallest deployable unit in Kubernetes, usually one container (or a few tightly related ones). Like a single food truck in a fleet.

**Namespace** — A way to divide one Kubernetes cluster into separate virtual sections, like folders on a shared drive. Companies use it so different teams or environments (dev, staging, prod) don't step on each other.

**ConfigMap** — A place to store non-secret settings (like a URL or a feature flag) separately from your code, so you can change settings without rebuilding the app.

**Secrets Management** — A secure vault for passwords, API keys, and certificates, so they're never hardcoded in your code. Like a safe instead of leaving cash on the counter. Critical for security, not just convenience.

**Sidecar** — A helper container that rides alongside your main app container, handling things like logging or security without touching the app's code. Like a motorcycle sidecar carrying the extra gear.

## Networking and Traffic

**API Gateway** — The single front door that all requests pass through before reaching your backend services. Like a hotel front desk routing guests to the right department. Companies use it to handle auth, rate limiting, and routing in one place.

**Load Balancer** — Spreads incoming traffic across multiple servers so no single one gets overwhelmed. Like a host at a restaurant seating people evenly across sections instead of overloading one waiter.

**Service Mesh** — A dedicated layer that manages how services talk to each other: security, retries, monitoring, all handled automatically. Like a private courier network between your microservices instead of each one figuring out delivery on its own.

**Istio / Envoy** — Specific tools that implement a service mesh. Envoy is the courier itself (moves the traffic); Istio is the dispatch system managing all the couriers.

**Ingress** — The Kubernetes rule set that controls how outside traffic gets routed into the cluster. Like the building's main entrance and directory sign.

**Reverse Proxy** — A server that sits in front of your backend and forwards requests to it, hiding the real server from the internet. Like a receptionist who takes messages instead of letting callers reach your desk directly.

**Circuit Breaker** — A safety switch that stops sending requests to a service that's failing, so one broken piece doesn't crash the whole system. Like a fuse box tripping before your house burns down.

**Rate Limiting** — Capping how many requests a user or system can make in a given time. Like a buffet limiting you to one plate per trip so nobody hogs the shrimp.

**mTLS (mutual TLS)** — Both sides of a connection verify each other's identity before talking, not just the client checking the server. Like two people showing ID to each other before a handoff, not just one.

**Zero Trust** — A security model where nothing inside or outside the network is trusted by default; everything has to prove who it is, every time. Companies adopt this because "inside the firewall" no longer means "safe."

## Communication Between Services

**gRPC** — A fast way for services to talk to each other using a strict, compact format. Like sending a pre-filled form instead of a rambling email.

**REST** — The standard way web services talk over HTTP using predictable URLs and verbs (GET, POST, etc). The common language most APIs speak.

**GraphQL** — Lets the client ask for exactly the data it needs in one request, instead of getting a fixed shape back. Like ordering a custom sandwich instead of picking from a fixed menu.

**WebSocket** — Keeps a connection open so both sides can send messages anytime, instead of asking-and-waiting. Used for live chat, stock tickers, multiplayer games.

**Message Queue** — A holding line for tasks or messages that get processed one at a time, even if the sender is faster than the receiver. Like a deli ticket system: you take a number, orders get handled in sequence without chaos.

**Kafka / RabbitMQ** — Popular message queue systems. Kafka handles huge, continuous streams of data (like a firehose); RabbitMQ handles discrete task messages (like a mail sorting room).

**Pub/Sub (Publish/Subscribe)** — One system announces an event, and any other system that's interested picks it up, without them needing to know about each other directly. Like a radio broadcast: the station doesn't know who's listening.

**Event-Driven** — An architecture where actions trigger events, and other parts of the system react to those events instead of being directly called. Like a smoke detector: it doesn't call the fire department itself, it just triggers an alarm and other systems respond.

## Auth and Access

**Auth (OAuth/JWT)** — OAuth is a standard way to let users log in via another service (like "Sign in with Google"). JWT is a signed token that proves who a user is without checking a database every time. Like a wristband at a festival: flash it once, get in everywhere.

**RBAC (Role-Based Access Control)** — Permissions are assigned based on a user's role, not the individual. Like a hospital: nurses, doctors, and admins each get access based on their job title, not their name.

## Scaling and Deployment

**Horizontal Scaling** — Adding more machines to handle more load, instead of making one machine bigger. Like adding more cash registers instead of building a faster one.

**Auto-scaling** — The system automatically adds or removes capacity based on demand. Like a restaurant calling in extra staff only on a busy Friday night, automatically.

**Cold Start** — The delay when a serverless function has to "wake up" before running, because it wasn't already active. Like a cold car engine taking a second to turn over.

**Multi-tenant** — One system serving many different customers, with their data kept separate. Like an apartment building: one structure, separate locked units.

**Blue-Green Deployment** — Running two identical environments (blue and green), sending traffic to one while updating the other, then switching over instantly. Reduces downtime and gives you an instant rollback.

**Canary Release** — Rolling out a new version to a small slice of users first, before going to everyone. Named after canaries in coal mines: if something's wrong, you find out small before it's a big problem.

**CI/CD (Continuous Integration/Continuous Deployment)** — Automated pipelines that test and ship code changes constantly instead of in big, risky batches. Like an assembly line with quality checks at every station instead of one inspection at the end.

**IaC (Infrastructure as Code) / Terraform** — Defining your servers, networks, and cloud resources in code files instead of clicking around a dashboard. Lets you version, review, and repeat infrastructure changes like software.

## Cloud and Infrastructure

**Cloud Native** — Software built specifically to take advantage of cloud features (scaling, containers, managed services) rather than just being lifted from a data center.

**EKS/GKE/AKS** — Managed Kubernetes services from Amazon, Google, and Microsoft. They run the Kubernetes control plane for you so you don't manage it yourself.

**ECS / Fargate** — Amazon's own container orchestration (ECS) and its serverless version (Fargate), where you don't manage the underlying servers at all.

**Edge Computing** — Running code physically closer to the user (at "edge" locations) instead of one central data center, to cut down latency. Like having local warehouses instead of shipping everything from one central factory.

**CDN (Content Delivery Network)** — A network of servers around the world that cache and serve content close to the user. Why a video loads fast no matter where you are.

## Data

**ETL (Extract, Transform, Load)** — The process of pulling data from one place, cleaning/reshaping it, and loading it somewhere else, usually for analytics. Like sorting and repackaging raw groceries before stocking the shelves.

**Cache Layer / Redis** — A fast, temporary storage layer that holds frequently used data so you don't have to hit the slow database every time. Redis is the most common tool for this. Like keeping snacks on the counter instead of walking to the pantry every time.

**ORM (Object-Relational Mapping)** — A tool that lets developers work with database data using their programming language's normal objects, instead of writing raw SQL constantly.

**Database Sharding** — Splitting a huge database into smaller pieces spread across multiple servers, so no single database gets overwhelmed. Like splitting one massive filing cabinet into several smaller ones by last name.

## Observability

**Distributed Tracing** — Tracking a single request as it moves across multiple services, so you can see exactly where it slowed down or broke. Like a tracking number that shows every stop a package makes.

**Observability** — The broader ability to understand what's happening inside your system from the outside, using logs, metrics, and traces. Not just "is it up," but "why is it behaving this way."

**Prometheus / Grafana** — Prometheus collects and stores metrics (numbers over time, like CPU usage). Grafana turns those metrics into dashboards you can actually look at.

**ELK Stack (Elasticsearch, Logstash, Kibana)** — A common toolset for collecting, searching, and visualizing logs at scale.

## Mobile-Specific

**iOS/Android SDK** — The official toolkits Apple and Google provide for building apps on their platforms.

**React Native / Flutter** — Frameworks that let you write one codebase and ship it to both iOS and Android, instead of building two separate native apps.

**Push Notifications** — Messages sent to a user's device even when the app isn't open. Why your phone buzzes with a sale alert.

**Offline-first** — Designing an app to work even without internet, syncing up once a connection returns. Like a notes app that still lets you type in airplane mode.

**Deep Linking** — A link that opens a specific screen inside an app, instead of just launching the app to its home screen. Like a link that opens directly to a product page instead of the store's front door.

**App Store Optimization (ASO)** — Improving how an app ranks and converts in app store search results, the mobile equivalent of SEO.

**MDM (Mobile Device Management)** — Tools companies use to secure and manage employee phones, like wiping a lost phone remotely or enforcing a passcode. Common in enterprise environments with BYOD policies.