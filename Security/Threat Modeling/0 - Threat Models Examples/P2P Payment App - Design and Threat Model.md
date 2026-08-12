# Learning Peer-to-Peer Payments: System Design and Threat Model

## 1. What This System Is and Why It Works This Way

A peer-to-peer (P2P) payment app, like Venmo or Cash App, lets one person send money to another person using just a phone. No cash, no check, no trip to the bank.

Here's what the user actually experiences: they open the app, pick a friend, type an amount, hit send. A few seconds later, the friend has the money in their account and can spend it or move it to their bank.

That simple experience hides a lot of engineering, because money is not like a text message. If you send a text twice by accident, nothing bad happens. If you send money twice by accident, someone just got paid double, and now the company has to get that money back or eat the loss. Every design choice in this system exists because "just send data" isn't good enough. It has to be "send data that represents real money, exactly once, to the right person, and never lose track of it."

Why each major part exists:

- **A ledger (a record of every transaction)** exists because the app needs to know, at any moment, exactly how much money everyone has and where it came from. Without it, balances would just be a guess.
- **A risk engine** exists because scammers and thieves specifically target payment apps, since that's where the money is. A social app doesn't need this. A money app does.
- **Identity verification (KYC)** exists because the government requires companies that move money to know who their customers are, to prevent money laundering and fraud.
- **A connection to real banks** exists because the app itself isn't a bank. It's a layer on top of the actual banking system. Someone still has to move real dollars between real bank accounts.

What would break if any part were missing:

- No ledger: balances would drift and become wrong, and nobody could prove who really owned what money.
- No risk engine: fraud would be cheap and easy to commit at scale.
- No identity verification: the company would be breaking the law and would become a tool for laundering stolen money.
- No bank connection: the app would just be an IOU system with no way to turn "app money" into real spendable cash.

The key design consideration underneath everything: **the client app (the thing on your phone) can never be trusted.** It can be hacked, faked, or modified. Every important decision (how much money, who gets it, whether it's allowed) has to be double-checked by the server, not just accepted from the phone.

---

## 2. Why Security Matters Here (Conversational)

Let's talk like two people, not like a lecture.

If you used this app, what would actually worry you? Probably: "What if someone sends money from my account without my permission?" or "What if I send $50 to my friend and it accidentally goes out twice?" or "What if a scammer tricks me into sending money to them on purpose?"

That last one is the sneaky one. Most security problems involve a hacker breaking in. This one involves the real, legitimate owner of an account approving a payment themselves, because someone lied to them ("this is your landlord, pay the deposit here"). The system worked exactly as designed. The user still lost money. That's what makes payment app security harder than typical app security: you can build a perfect lock, and the attacker just convinces the owner to open the door.

Who gets hurt when things go wrong? Usually the least technical, most trusting users. Someone's grandmother sending money to a fake "grandkid in trouble." A small business owner accepting a payment from a scammer who then reverses it through their bank.

"Secure enough" for this type of system doesn't mean "unhackable." It means: money can never be created out of nothing, money can never be duplicated by accident, every transaction can be traced back to who approved it, and when something does go wrong, the company can prove exactly what happened and reverse it correctly. It also means accepting that some fraud will always get through, because you can't fully engineer away human trust. The job is to keep that fraud small and recoverable, not to promise zero.

---

## 3. Architecture Components

**Client-side (the phone app)**

- The app itself (iOS/Android), and a lighter web version
- A secure storage area built into the phone (keychain on iPhone, keystore on Android) that holds login tokens, never the actual money balance
- Fingerprint/Face ID as a way to unlock quick actions, not as the only proof that a payment is authorized
- Certificate pinning: a trick that makes it harder for an attacker to intercept the app's traffic even on a compromised network

**Server-side (the company's backend)**

- API gateway: the "front door" that all requests pass through, checks basic things like rate limits and valid login tokens
- Authentication/authorization service: confirms who you are and what you're allowed to do
- Ledger service: the permanent, append-only record of every transaction; this is the actual source of truth for money
- Payment orchestration service: manages the steps a payment goes through, like a checklist (started, risk-checked, posted, settled)
- Risk/fraud engine: watches for suspicious patterns
- Idempotency service: prevents the same request from being processed twice if the phone sends it twice (like from a bad network connection)
- Notification, profile, and social feed services (if the app has a public activity feed like Venmo)
- Reconciliation jobs: overnight processes that check the ledger against what actually happened at real banks
- Admin/support tooling: a separate system for employees, which never edits balances directly

**Third-party integrations**

- Card networks and instant-transfer rails (Visa Direct, Mastercard Send) for moving money to debit cards
- ACH/RTP/FedNow, routed through a partner bank, for moving money to bank accounts
- A bank-linking service (like Plaid) that lets users connect their real bank account
- Identity verification vendor (checks government ID, confirms you are who you say)
- Sanctions/OFAC screening vendor (checks you're not on a government watch list)
- Fraud-scoring vendor
- Push notification services (Apple/Google)
- A cloud key management service (HSM/KMS) that protects encryption keys

**Data stores**

- Ledger database: append-only, this one matters most
- User/profile database
- Device and session database
- Fraud/risk signal store
- Audit log store (for admin actions and support tickets)

---

## 4. Core Design Decisions

|Layer|Choice|Rationale|Plain-English Explanation|
|---|---|---|---|
|Balance storage|Append-only ledger, no editable balance number|Prevents lost or corrupted balances|Your balance isn't a number stored somewhere, it's calculated by adding up every transaction you've ever had. Nobody can just quietly change one number.|
|Payment authorization|Server decides the real amount and recipient, not the phone|The phone app can be hacked or faked|Even if someone tampers with the app, they can't trick the server into sending more money than was actually approved, because the server checks everything itself|
|Duplicate payment prevention|A unique ID for every payment request, enforced by the database itself|Phones send duplicate requests all the time due to bad signal|If your phone's network hiccups and it accidentally sends the "pay $20" request twice, the database itself refuses the second one, it's not just a polite check in the app code|
|Login sessions|Short-lived login tokens that expire and refresh often|Limits the damage if a token is stolen|A stolen token becomes useless quickly, instead of working forever|
|Sending money to a bank|Treated as "in progress" until confirmed the next day|Banks don't confirm transfers instantly|The app might show "sent" right away for a good user experience, but internally, the company doesn't fully trust that until it's double-checked against the bank the next day|
|Extra security checks (step-up auth)|Triggered by risk score, not a fixed dollar amount|A fixed rule ("always check above $500") is easy for scammers to work around|If scammers know the exact rule, they just stay under it. Randomizing based on risk makes it harder to predict|
|Identity verification|Ongoing and tied to how much you send, not just a one-time check at signup|People who pass a check once can behave differently later|Someone might sign up honestly and get verified, then later their account gets hacked or they start doing something risky. Limits should adjust to current behavior|
|Employee/support access|No employee can directly change a balance; every fix creates a new traceable ledger entry|Prevents insider abuse and mistakes|If a support agent needs to fix an error, the system doesn't erase what happened, it adds a new correction that's fully visible later|

---

## 5. Architecture Diagram (ASCII)

```
Legend: --> = data flow direction   [X] = component   (X) = external party

   [Phone App]
       |
       | 1. login/payment requests (HTTPS)
       v
   [API Gateway] ---> [Auth Service]
       |
       | 2. validated request
       v
   [Payment Orchestration]
       |
       |--3--> [Risk/Fraud Engine] --> (Fraud data vendor)
       |
       |--4--> [Idempotency Check] --> [Ledger Database]
       |
       |--5--> [Notification Service] --> (Push notification provider)
       |
       v
   [Ledger Service] ---6---> [Reconciliation Job] ---7---> (Bank rails / Card networks)
       |
       v
   [Admin/Support Tool] (writes only via new ledger entries, never direct edits)
```

This shows requests flowing one direction (in from the phone) and money-movement confirmations flowing back the other direction (from the banks, overnight).

---

## 6. Data Flow Diagram, Level 0 (Context Diagram)

```
                +-----------------------------+
   (User A) --> |                             | --> (User B)
                |     P2P PAYMENT SYSTEM       |
   (Bank) <---> |                             | <--> (Card Network)
                +-----------------------------+
                          ^        ^
                          |        |
                (KYC Vendor)   (Fraud Vendor)
```

This level treats the whole system as one black box. It's teaching you to first ask: **who is outside this system, and what crosses the boundary into or out of it?** Before you design anything internal, you need to know who the players are.

---

## 7. Data Flow Diagram, Level 1

```
(User A phone) --> [1. Auth/Login] --> [2. Create Payment Intent]
                                              |
                                              v
                                   [3. Risk Check] --uses--> (Fraud Vendor)
                                              |
                                        pass  |  fail
                                              v
                                   [4. Post to Ledger] <--> [Ledger Data Store]
                                              |
                                              v
                                   [5. Notify Both Users] --> (Push Provider)
                                              |
                                              v
                                   [6. Reconcile Nightly] --> (Bank Rails)
```

Level 0 told you the system exists and who it talks to. Level 1 breaks that single box into the actual steps a payment takes on the inside, and shows where a data store (the ledger) sits in the middle of the process. This is where you start seeing which step could go wrong.

---

## 8. Data Flow Diagram, Level 2 (Drilling into the Risk Check, the most sensitive step)

```
[Payment Intent Created]
        |
        v
[Pull device + account history] --> [Device/Session Store]
        |
        v
[Score transaction: amount, velocity, new payee?] --> (Fraud Vendor API)
        |
   +----+----+
   |         |
 low risk  high risk
   |         |
   v         v
[Auto-approve]  [Require step-up auth: biometric/2FA]
   |                     |
   |                pass | fail
   |                     v         v
   +-------------> [Post to Ledger]  [Block + flag for review]
```

The higher-level diagrams hide the fact that "risk check" isn't one step, it's a small decision tree with its own pass/fail paths. Level 2 reveals exactly where a scammer or a bug could slip through: at the scoring step, at the step-up auth step, or in how "high risk" gets defined in the first place.

---

## 9. User Journey Flow (ASCII)

```
[Open App] --> [Login] --?--> Success? 
                              |            |
                             Yes           No --> [Show error, allow retry, lock after N attempts]
                              v
                     [Choose recipient]
                              v
                     [Enter amount]
                              v
                     [Tap Send]
                              v
                [Server risk-checks in background]
                              v
                    Risk level? --high--> [Prompt: confirm with Face ID / 2FA]
                              |low                  |
                              v                     v pass         v fail
                     [Show "Payment Sent"]  [Show "Payment Sent"]  [Show "Payment blocked, contact support"]
                              v
                     [Recipient notified]
                              v
                     [Both balances updated in app]
```

Each step is labeled with what the person actually sees, because that's what a real user journey needs to show: not just backend logic, but the visible moments where a person decides to keep going or gives up.

---

## 10. Trust Boundaries

1. **Phone app ↔ API Gateway.** Crosses: login credentials, payment requests. Matters because the phone is outside company control and can be modified or hacked, so nothing coming from it can be blindly trusted.
2. **Phone's secure storage ↔ the app itself.** Crosses: login tokens. Matters because if the phone is stolen or the OS is compromised (jailbroken), this boundary is the last line of defense before someone impersonates the real user.
3. **API Gateway ↔ internal services.** Crosses: validated requests. Matters because this is where you assume the outer wall could be bypassed, so internal services still check permissions themselves rather than trusting the gateway blindly.
4. **Payment orchestration ↔ Ledger service.** Crosses: the actual instruction to move money. This is the most important boundary in the whole system, because this is the point where "a request happened" becomes "money actually moved."
5. **Company backend ↔ third parties (banks, card networks, KYC/fraud vendors).** Crosses: personal data, verification results, money transfer instructions. Matters because these are outside companies the platform doesn't fully control, and their mistakes or breaches become the platform's problem too.
6. **User ↔ user (inside the app).** Crosses: payment requests, public activity feed posts. Matters because this is a boundary between two customers, not between a customer and the company, and most scams happen here, not through hacking.
7. **Support/admin tools ↔ production data.** Crosses: account lookups, manual corrections. Matters because employees are insiders, and insider misuse or simple mistakes are a real, common risk, not just a theoretical one.

---

## 11. Threat Model Table

|Attacker|Capability|What They Target|Mitigation|
|---|---|---|---|
|External attacker (hacker)|Can attempt to log in, intercept traffic, or reverse-engineer the app|User accounts, payment requests|Rate limiting, MFA, certificate pinning, server-side validation of every request|
|Insider (employee/support)|Has legitimate access to internal tools|Account balances, personal data|No direct balance edits, every action logged with an identity attached, dual approval for large corrections|
|Compromised dependency (a third-party vendor gets breached)|Can inject false data or leak information the company shared with them|KYC results, fraud scores, personal data|Treat vendor responses as one input, not the final word; share only the minimum data each vendor needs|
|Unauthenticated user (someone with no account, or a stolen session)|Can probe public endpoints or attempt to reuse a stolen token|Login endpoints, payment endpoints|Strong session expiration, device binding, anomaly detection on new devices|
|Scammer targeting a legitimate user (social engineering)|Convinces a real, logged-in user to approve a real payment|The user's own trust and judgment|Friction for new/unusual payees, warnings, but no full technical fix (see Section 13)|
|Money mule network|Coordinates many real accounts to move stolen funds quickly|The reconciliation and detection systems|Cross-account graph analysis, not just single-transaction checks|

---

## 12. Threat Diagram (ASCII)

```
   (A) External attacker         (D) Unauthenticated user / stolen session
         |                                |
         v                                v
   +---------------------------------------------+
   |          TRUST ZONE 1: Public Internet        |
   |                                                |
   |     [Phone App] -----> [API Gateway]           |
   +------------------------|----------------------+
                             | TB1/TB3
   +------------------------v----------------------+
   |          TRUST ZONE 2: Internal Backend        |
   |                                                 |
   |   [Auth] --- [Risk Engine] --- [Payment Orch.]  |
   |                                    |            |
   |                                    | TB4         |
   |                              [Ledger Service]    |
   +------------------------|----------------------+
                             | TB5
   +------------------------v----------------------+
   |     TRUST ZONE 3: External Vendors & Rails      |
   |   (KYC vendor) (Fraud vendor) (Banks/Cards)     |
   +--------------------------|----------------------+
                               ^
                               | (C) Compromised dependency

   (B) Insider ---------> [Admin/Support Tools] --TB7--> Ledger (mediated only)

   (E) Scammer -----------> [Victim User] -----> Phone App (this attacker
                                                    never has to break in,
                                                    they convince the user
                                                    to do it themselves)

   (F) Mule network ------> Multiple [User] accounts ------> Payment Orch.
                             (only visible through cross-account analysis)
```

### 12a. Threat Diagram Reference Key

|Element|Plain-English Name|What It Represents|Specific Concern|Mitigation Category|
|---|---|---|---|---|
|Trust Zone 1|Public internet|Anything outside company control|Anyone can send traffic here|Input validation, rate limiting|
|Trust Zone 2|Internal backend|Company-controlled servers|Assume outer defenses can fail|Server-side re-validation everywhere|
|Trust Zone 3|External vendors/rails|Companies you depend on but don't control|Their failures become your failures|Minimum data sharing, treat as advisory input|
|TB4 (Ledger boundary)|The money-truth line|Where a request becomes an actual balance change|If this breaks, balances become wrong forever|Append-only design, database-level uniqueness constraints|
|TB7 (Admin boundary)|Insider access point|Employee tools touching real data|Insider abuse or mistakes|Logging, dual control, no direct writes|
|Attacker (E)|The scammer|Targets the user directly, not the system|No technical control catches this cleanly|Friction, warnings, limits (never fully solved)|

---

## 13. The Hardest Unsolved Problem

The single hardest problem here is **Authorized Push Payment (APP) fraud**: the real account owner, using their real login, on their real device, approves a real payment to a scammer because they were tricked.

Why it's hard: every technical defense in this system (encryption, authentication, the ledger) assumes the danger is someone pretending to be the user. This threat is the actual user, doing exactly what the system was built to let them do. There's no password to crack and no token to steal. The "attack" happens entirely in the human's head before the payment is even sent.

Ranked options and what each one costs:

1. **Delay new/large payments to first-time recipients, with a recall window.** Buys real fraud protection and a chance to catch scams. Costs the app its main selling point: instant transfers.
2. **Show warnings and confirmation prompts for unusual payments.** Cheap to build, no delay. Costs very little upfront, but people click through warnings out of habit, so it catches fewer scams over time.
3. **Cap how much money can go to a brand-new payee.** Limits the size of any single loss. Costs legitimate users who have a real reason to send a large first payment (like a security deposit).
4. **Have the company absorb some losses as a cost of doing business.** Keeps the product frictionless and builds trust. Costs the company real money and can attract abuse from people who fake being scammed.
5. **Educate users at the moment of payment.** Nearly free to implement. Costs almost nothing, but also barely works, since most people don't read warnings carefully in the moment.

There is no option on this list that solves the problem. Every real payment app accepts this as an ongoing cost, not a bug to be fixed once.

---

## 14. What Is Out of Scope and Why

- **Physical debit/credit card issuance and security.** That's its own product with its own manufacturing and PIN-security requirements, separate from the app itself.
- **Cryptocurrency or blockchain-based transfers.** These settle differently (no reversals, no central ledger owner), which changes the entire threat model.
- **Cross-border currency exchange and international banking rules.** Each country adds its own regulatory threat model; this design assumes a single country's rails.
- **The legal outcome of disputes and chargebacks.** That's a legal/compliance process, not a technical security control.
- **Tax reporting accuracy (like 1099 forms).** That's a compliance and accounting problem, separate from whether money moved securely.
- **The internal workings of the cloud key management service (HSM/KMS).** This design treats it as a trusted building block someone else already secured, not something built here.

---

## 15. What the Core Security Claim Does NOT Cover

The core claim of this design is: **the ledger stays internally consistent, and money only moves when a real, authenticated request approves it, exactly once.**

This claim does **not** mean the system is safe from fraud. Specifically, it does not cover:

- Whether the person behind the login is really who their ID says they are, beyond what the KYC vendor could confirm
- Whether that real, authenticated person was tricked into approving a payment on purpose (Section 13)
- Whether the bank on the other end actually completes the transfer (the ledger says "sent," the bank still has to agree)
- Card network chargebacks that happen after money has already been recorded as sent
- What happens if someone's phone itself is stolen and unlocked
- Whether funds stay available if a banking partner has an outage or ends the relationship

Anyone reading this design should understand: "the ledger is correct" is not the same thing as "nobody loses money."

---

## 16. Open Decisions Checklist

- [ ] Is the duplicate-payment protection enforced at the database level, or only in application code? (If only in code, this needs to be fixed before launch.)
- [ ] Where exactly does the risk-scoring threshold live, and can an attacker learn it by testing the system?
- [ ] Does identity verification ever get re-checked later, or is it one-and-done at signup?
- [ ] Is SMS ever used as the only method to prove identity for high-risk actions (like resetting a password)?
- [ ] What's the actual dollar threshold requiring a second employee to approve a manual account correction?
- [ ] Does fraud detection look across multiple accounts (to catch mule networks), or only at one transaction at a time?
- [ ] Has legal/compliance formally decided who eats the loss in a scam payment, or is that still assumed rather than documented?
- [ ] Are admin/support actions logged in enough detail to support an actual investigation, not just a basic activity log?
- [ ] Is there a hard cap on how much money can go to a brand-new, unverified payee?