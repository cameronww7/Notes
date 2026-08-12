# Password Manager (3-Tier Architecture + Mobile App): System Design & Threat Model

---

## Section 0 — Concepts You Need First (Vocabulary Primer)

You can't threat model what you can't name. These are the concepts the rest of this document assumes. Each is explained once, here, in plain language.

**Three-tier architecture.** A way of splitting a system into three layers that only talk to their neighbors:

- **Tier 1 (Presentation):** what the user touches. Here: the mobile app, browser extension, and desktop app.
- **Tier 2 (Application):** the logic layer. API servers that authenticate you, sync your vault, handle sharing, and enforce rules like rate limits.
- **Tier 3 (Data):** where things are stored. Databases and blob storage. Tier 1 never talks to Tier 3 directly. That rule is itself a security control: even if an attacker fully controls a client, they can only ask Tier 2 for things Tier 2 is willing to do.

**The DOM (Document Object Model).** When a browser loads a web page, it turns the HTML into a live tree of objects called the DOM. Every login form, every input box, every button is a node in that tree. JavaScript on the page can read and change the DOM. This matters enormously for a password manager because **autofill works by writing your password into a DOM node**, and the moment your password is in the DOM, any JavaScript running on that page can read it. Autofill is the act of handing your secret to a web page and trusting that page not to be hostile.

**Content script vs. background script (browser extension anatomy).** A browser extension has two halves:

- The **background service worker**: runs in the extension's own isolated world. It holds the connection to the vault and never touches web pages directly.
- The **content script**: a small piece of the extension that gets injected _into_ web pages so it can see and fill forms. It lives in a partially shared world with the page's own JavaScript, and it can touch the page's DOM.

The security design of the whole extension comes down to: keep secrets in the background worker as long as possible, and give the content script the smallest job and the shortest exposure window you can.

**Origin.** A web origin is the combination of scheme + host + port: `https://bank.com:443`. Browsers isolate origins from each other (the "same-origin policy"). A password manager's most important client-side decision is: _does the origin of this page exactly match the origin stored with this credential?_ Get that comparison sloppy (matching on substrings, ignoring subdomains) and you will autofill `bank.com` credentials into `bank.com.evil-login.net`.

**KDF (Key Derivation Function).** A deliberately slow, expensive math function that turns a human password into a cryptographic key. Slowness is the feature: it makes each password guess cost the attacker real time and memory. Argon2id is the modern choice because it's _memory-hard_: guessing requires lots of RAM per attempt, which neutralizes GPU farms.

**AEAD (Authenticated Encryption with Associated Data).** Encryption that also detects tampering. AES-256-GCM and XChaCha20-Poly1305 are AEADs. If anyone flips a single bit of your encrypted vault, decryption fails loudly instead of silently producing corrupted-but-plausible data.

**Key wrapping / key hierarchy.** Instead of encrypting the vault directly with the key derived from your master password, you generate a random key (the vault key), encrypt the vault with that, and then encrypt ("wrap") the vault key with the password-derived key. Now changing your master password only means re-wrapping one small key, not re-encrypting megabytes of vault.

**Zero-knowledge architecture.** A design where the server stores your data but is mathematically incapable of reading it, because every key needed to decrypt it exists only on your devices. This is a _claim about architecture_, not a marketing term, and later sections spell out exactly where the claim holds and where it quietly doesn't.

**Secure Enclave / StrongBox / TEE.** Dedicated hardware on phones that stores keys and does crypto operations in a chip the main operating system can't read into, even with root access. Used here to bind biometric unlock to hardware instead of software.

**HSM (Hardware Security Module).** The server-side equivalent: tamper-resistant hardware that holds keys and enforces rules (like "only release this key after N verified factors") that even the server's own admins can't override. Relevant to recovery design in Section 13.

**STRIDE.** A checklist of threat categories: **S**poofing, **T**ampering, **R**epudiation, **I**nformation disclosure, **D**enial of service, **E**levation of privilege. Used in the threat table to make sure a whole category wasn't just forgotten.

---

## Section 1 — What This System Is and Why It Works This Way

A password manager stores all your credentials in one encrypted vault, syncs that vault across your phone, laptop, and browser, and fills passwords into apps and websites for you. The company running the service hosts your vault but cannot read it.

Why each major part exists, framed as the user need it answers:

- **Master password** — "I can only remember one secret." Everything in the system exists to protect this one secret and the keys derived from it.
- **Client-side encryption** — "What if the company gets hacked?" The vault is encrypted _on your device, before it ever leaves_. A server breach yields ciphertext.
- **Three-tier backend** — "What if one server bug exposes everything?" Splitting API logic (Tier 2) from storage (Tier 3) means a bug in the API layer doesn't hand out raw database access, and the database never accepts a connection from the internet.
- **Mobile app with OS autofill integration** — "I log into apps on my phone more than websites on my laptop." iOS and Android both have official autofill frameworks (Credential Provider on iOS, Autofill Framework on Android) that let the manager fill _native apps_, not just browsers. Without this, mobile users copy-paste passwords through the clipboard, which is one of the most attackable paths on a phone.
- **Browser extension** — "Stop making me type." Autofill is the retention feature. If it's clunky, users go back to reusing `Summer2026!` everywhere, and the product has failed at its actual mission even if its crypto is perfect.
- **Sync service** — "My vault must be identical on every device." The server moves encrypted blobs between devices and resolves version conflicts _without ever holding a decryption key_.
- **Vault sharing** — "My partner needs the Netflix login; my team needs the AWS root credentials." Sharing done wrong (email the password) destroys the whole model, so sharing has to be a first-class encrypted feature (Section 4 covers the crypto).
- **Recovery** — "What if I forget the master password?" The most dangerous feature in the product. Section 13 is entirely about this.

What breaks if a part is missing:

- No client-side encryption → the company, its employees, its cloud provider, and anyone who breaches any of them can read every password you have.
- No tier separation → one SQL injection or SSRF bug in an API endpoint becomes full database exfiltration.
- No OS-level mobile autofill → users route secrets through the clipboard, where any app polling the clipboard can grab them.
- No strict origin matching → the manager becomes a phishing _amplifier_: it fills real credentials into fake pages faster than a human ever would.
- No recovery path → forgotten master password permanently destroys access to every account the user has.

---

## Section 2 — Why Security Matters Here (Conversational)

Think about what's actually in the vault: your email password (which resets everything else), bank logins, work SSO, TOTP seeds, maybe your crypto seed phrase in a secure note. This isn't "an account gets compromised." This is _every account, simultaneously, plus the 2FA that was supposed to protect them_ if you stored TOTP seeds in the same vault. The vault is a single point of catastrophic failure by design, and the entire engineering job is making that acceptable.

What a real person worries about, and whether they're worried about the right things:

- "What if my phone gets stolen?" — Reasonable, well handled. Vault encrypted at rest, keys gated behind biometrics in hardware.
- "What if the company gets hacked?" — Reasonable, well handled _if_ zero-knowledge is real. The 2022 LastPass breach is the case study: attackers took the encrypted vaults, and every user's safety instantly reduced to the strength of their personal master password and the KDF iteration count the company had configured for them. Users with old, low iteration settings and weak master passwords got cracked.
- "What if I forget my master password?" — Reasonable, and this is where products quietly cheat. If the answer is "click a reset link in your email," the encryption was theater.
- What they _don't_ worry about but should: the browser extension. It runs inside the most hostile software environment on their computer (the browser, executing arbitrary attacker-controlled JavaScript all day) and its whole job is releasing secrets into that environment.

Why the security decisions are harder than they look: every single convenience feature is a hole punched in the core promise. Biometric unlock means a key exists somewhere that isn't derived from the master password. Autofill means secrets enter the DOM. Sharing means keys move between users. Recovery means a second door into the vault. Clipboard copy means the secret sits in a buffer any app can read. None of these are optional in a real product, so the job is never "don't punch holes," it's "make each hole small, short-lived, and monitored."

"Secure enough" for this system means three things concretely: (1) offline cracking of a stolen vault must cost more than the vault is worth, for a _median-strength_ master password, not just a great one; (2) the company must be architecturally unable to read vaults, so a subpoena, a rogue admin, or a full server breach all yield ciphertext; (3) the autofill path must never fill into an origin that wasn't explicitly saved, even when the page is actively trying to trick it.

---

## Section 3 — Architecture Components

### Tier 1 — Clients (Presentation)

|Component|What It Does|Security-Relevant Detail|
|---|---|---|
|Mobile app (iOS/Android)|Primary vault UI, unlock, autofill into native apps|Integrates iOS Credential Provider Extension / Android Autofill Framework; keys gated via Secure Enclave / StrongBox|
|Browser extension: background service worker|Holds unlocked session, talks to Tier 2, decides fill eligibility|Isolated extension origin; secrets live here, not in pages|
|Browser extension: content script|Detects login forms, injects credentials into DOM on approval|Runs inside hostile pages; smallest possible privilege and exposure window|
|Desktop app|Full vault management, offline access|Native messaging channel to extension must authenticate the peer binary|
|KDF module|Master password → stretched key (Argon2id)|Client-side only; parameters stored with vault metadata|
|Crypto engine|AEAD encrypt/decrypt of vault and items|Constant-time implementations, zeroize buffers after use|
|Local vault file|Encrypted blob on device disk|Excluded from OS cloud backups (iCloud/Google Backup) by policy decision|
|Session cache|Unlocked vault key + decrypted items, memory only|Never swapped/persisted; cleared on lock, timeout, OS lock event|

### Tier 2 — Application Services

|Component|What It Does|Security-Relevant Detail|
|---|---|---|
|API gateway / WAF|Single entry point, TLS termination, request shaping|Rate limiting, bot detection, credential-stuffing throttles live here|
|Auth service|Verifies auth secret (OPAQUE/SRP or hash-of-derived-secret), issues session tokens|Never receives the master password or vault key; enforces account 2FA|
|Sync service|Accepts/serves encrypted vault blobs, versioning, conflict handling|Treats blobs as opaque; validates size/format only|
|Sharing service|Routes wrapped item keys between users' public keys|Directory of user public keys; MITM risk if key verification is skipped|
|Device/session service|Device registration, new-device approval, session revocation|New-device alerts; kill switch for stolen devices|
|Recovery service|Whatever recovery model Section 13 selects|The most dangerous service in Tier 2|
|Notification service|Push triggers for sync, new-device and share alerts|Pushes contain no secret content, only "wake up and sync"|
|Breach-check proxy|k-anonymity range queries to HIBP-style services|Client sends hash prefix only; proxy prevents third party seeing user IPs + queries together|

### Tier 3 — Data Stores

|Store|Contents|Security-Relevant Detail|
|---|---|---|
|Encrypted vault blob store|Opaque client-encrypted vaults|Server-side encryption at rest _in addition_ (defense in depth, protects backups/snapshots)|
|Auth secret store|Verifiers (OPAQUE records / salted hashes of auth secret)|Compromise enables offline attack on auth secret, not on vault key (separate HKDF branch)|
|Metadata DB|Account records, device list, vault version, sharing relationships|This is what the server _can_ see; it leaks more than users expect (Section 15)|
|Public key directory|Per-user public keys for sharing|Integrity matters more than confidentiality; a swapped key = silent interception of shares|
|Recovery escrow (only if model 5 in §13)|HSM-guarded wrapped keys|Existence of this store is itself a design decision with residual risk|
|Audit/security logs|Auth events, device changes, admin actions|Must never contain secrets; retention and access tightly controlled|

### Third-Party Integrations

- **APNs / FCM** — push delivery. Treated as untrusted transport: payloads are content-free triggers.
- **HIBP-style breach data** — via k-anonymity proxy only.
- **OS secure hardware** — Secure Enclave, StrongBox/TEE, Windows Hello/TPM.
- **App stores / extension stores** — the software supply chain's last mile; a compromised publisher account is a full-system compromise (Section 11, attacker 4).

---

## Section 4 — Core Design Decisions

|Layer|Choice|Rationale|Plain-English Explanation|
|---|---|---|---|
|KDF|Argon2id, per-user random salt, parameters stored alongside vault and versioned|Memory-hardness resists GPU/ASIC cracking; versioning allows raising cost over time|Each password guess costs the attacker lots of RAM and time. Storing the parameters means old vaults can be upgraded to stronger settings at next login, which is exactly what LastPass failed to do for legacy users|
|Vault encryption|XChaCha20-Poly1305 (or AES-256-GCM where hardware-accelerated)|AEAD: confidentiality + tamper detection; XChaCha's large nonce removes nonce-reuse footguns|The vault can't be read _or_ silently modified. The nonce choice removes a classic implementation mistake (reusing a random nonce) from the table entirely|
|Key hierarchy|Master password → Argon2id → stretched key → HKDF splits → (a) key-wrapping key, (b) auth secret. Random vault key wrapped by (a)|Password rotation without vault re-encryption; server never holds vault-key material|Your password locks a small keybox; the keybox holds the real vault key. Change the password, re-lock the keybox, done in milliseconds|
|Client↔server auth|OPAQUE (preferred) or SRP|Server never sees a password-equivalent value, even at registration; resists precomputation on server breach|With plain "send a derived hash," a breached server database lets attackers start guessing immediately. OPAQUE means the stored record is useless without running the slow protocol per guess|
|Account 2FA|TOTP/WebAuthn required for new-device login, in addition to auth secret|Stolen master password alone shouldn't enroll a new device|Two different things must be stolen, from two different places, to add an attacker's device|
|New-device enrollment|Existing-device approval or 2FA + delay + notification|The enrollment flow is how attackers turn a stolen password into vault access|Adding a device is the real "front door"; it gets the most friction in the whole product|
|Vault sharing|Each shared item has its own random item key; item key wrapped with each recipient's public key (X25519); sender key verification encouraged|Sharing never exposes the sender's vault key; revoking a person = stop wrapping future keys for them + rotate item key|You never share your master key. You share one locked box per item, and each recipient gets their own copy of that box's key sealed to them personally|
|Per-item encryption|Yes, item keys under the vault key|Limits blast radius; enables sharing model above; enables partial sync|One leaked item key exposes one item, not the vault|
|Autofill origin matching|Exact origin match (scheme+host+port), no substring matching, subdomain fill requires explicit user opt-in per credential, no fill into cross-origin iframes by default|The manager must be harder to phish than the human|The single most common real-world extension vulnerability class is sloppy URL matching. Default-deny is the only defensible position|
|Autofill interaction|Fill only on explicit user gesture (click/keyboard confirm), never silent auto-submit|Prevents invisible-form harvesting and clickjacked fills|The page never gets a password without the human deciding, in that moment, to hand it over|
|Mobile key storage|Biometric unlock wraps the vault key with a hardware-backed key (Secure Enclave/StrongBox), invalidated on biometric enrollment change|Convenience without a software-readable key on disk|Your face/fingerprint releases a key from a chip the OS itself can't read. Add a new fingerprint, and the stored key is deliberately destroyed|
|Session policy|Auto-lock on timeout, on OS lock, on app background (configurable); memory zeroized on lock|Shrinks the window where plaintext exists|The unlocked state is the dangerous state; the product's job is to spend as little time in it as usability allows|
|Tier 2 → Tier 3 access|Private network only, per-service DB credentials, least privilege, no internet route to Tier 3|Contain API-layer bugs|A bug in the sync API can, at worst, read what the sync service can read: opaque blobs|
|Server-side encryption at rest|Yes, on top of client-side encryption|Protects backups, snapshots, decommissioned disks|Defense in depth. Client-side crypto is the real protection; this covers operational leak paths like a backup tape or misconfigured snapshot|
|Telemetry|No URLs, no item names, no usage-per-site in analytics|Metadata is the data the company _can_ leak|If the company can't read your vault but logs every domain you autofill on, it built a browsing tracker with extra steps|

---

## Section 5 — Architecture Diagram (ASCII)

```
                                   [ APNs / FCM ]          [ HIBP-style Breach Data ]
                                        ^                            ^
                                        | content-free push          | k-anon hash-prefix query
                                        |                            |
+=============================== TIER 1: CLIENTS ==============================+
|                                                                              |
|  MOBILE APP                          BROWSER                                 |
|  +------------------------+         +--------------------------------+      |
|  | [Master Password/Bio]  |         |  EXTENSION                     |      |
|  |        |               |         |  +--------------------------+  |      |
|  |        v               |         |  | Background Service Worker|  |      |
|  | [KDF: Argon2id]        |         |  | (session, fill decisions)|  |      |
|  |        |               |         |  +-------------------------+  |      |
|  |        v               |         |               | approved fill  |      |
|  | [Stretched Key]        |         |               v                |      |
|  |    |          \        |         |  +--------------------------+  |      |
|  |    v           v       |         |  | Content Script           |  |      |
|  | [Vault Key] [Auth Sec] |         |  | (form detect, DOM write) |  |      |
|  |    |                   |         |  +------------+-------------+  |      |
|  |    v                   |         +---------------|----------------+      |
|  | [Crypto Engine]        |                         v                       |
|  |    |                   |         [ Web Page DOM  (hostile territory) ]   |
|  |    v                   |                                                 |
|  | [Encrypted Vault File] |   DESKTOP APP <--native msg--> extension        |
|  | [Session Cache (RAM)]  |                                                 |
|  +-----------+------------+                                                 |
|              |                                                              |
|  [ Secure Enclave / StrongBox: hardware-wrapped keys ]                      |
+==============|===============================================================+
               |  TLS  (blobs already encrypted client-side)
               v
+=========================== TIER 2: APPLICATION ==============================+
|                                                                              |
|   [ API Gateway / WAF / Rate Limiter ]                                       |
|        |         |          |          |            |                        |
|        v         v          v          v            v                        |
|   [Auth Svc] [Sync Svc] [Sharing] [Device/Session] [Recovery Svc]           |
|                                        |                                     |
|                                   [Notification Svc] --> APNs/FCM            |
|                                   [Breach Proxy] -----> HIBP                 |
+==============|===============================================================+
               |  private network only, per-service least-privilege creds
               v
+============================= TIER 3: DATA ===================================+
|                                                                              |
|  [Encrypted Vault Blob Store]   [Auth Secret Store (OPAQUE records)]         |
|  [Metadata DB]                  [Public Key Directory]                       |
|  [Audit Logs]                   [Recovery Escrow (HSM) - only if chosen]     |
+==============================================================================+

LEGEND
  [ ]   component          + boxes  tier / trust zone
  -->   data flow          TLS      encrypted transport
  RAM   memory-only, never persisted
```

---

## Section 6 — Data Flow Diagram, Level 0 (ASCII)

```
                     +--------------------------+
      User --------> |                          | ------> Autofilled credentials
                     |     PASSWORD MANAGER     |          (into apps & pages)
 Breach data ------> |         SYSTEM           | ------> Breach & new-device alerts
 (3rd party)         |                          |
 Other user -------> |  (clients + backend as   | ------> Shared items
 (share recipient)   |   one bubble, for now)   |          (to recipient's vault)
                     +--------------------------+
                          ^               |
                          |               v
                   New device        Sync updates
                   enrollment        to all devices
```

This level teaches the big picture: one system, four outside actors (user, breach feed, share recipients, and the user's own new devices), and what crosses the outer edge. : sharing appears as a first-class external flow, because another human being is an external actor to _your_ trust zone even when they're a customer of the same product.

---

## Section 7 — Data Flow Diagram, Level 1 (ASCII)

```
User -> [1. Unlock Vault] -> [2. Decrypt Items] -> [3. Autofill] -> App/Website DOM
             |                       ^
             v                       |
       [Local Vault Store] <---------+
             ^                                      [7. Breach Check]
             |                                        |  (hash prefixes only)
       [4. Sync Process] <------TLS------> [T2 Sync Svc] <-> [T3 Blob Store]
             |
             v
       [5. Sharing Process] <---TLS------> [T2 Sharing Svc] <-> [T3 Key Directory]
             |                                                    |
             v                                                    v
       [6. Recovery Process] <--TLS------> [T2 Recovery Svc] <-> [T3 Escrow, if any]

       [8. New Device Enrollment] <--TLS--> [T2 Device Svc] <--> [T3 Metadata DB]
```

What this adds to Level 0: the single bubble splits into eight real processes, and the tier boundary becomes visible. Notice which processes touch plaintext (1, 2, 3 — all client-side) and which only ever touch ciphertext and wrapped keys (4, 5, 6, 7, 8). That split _is_ the zero-knowledge architecture, drawn as a diagram.

---

## Section 8 — Data Flow Diagram, Level 2: Unlock Vault (ASCII)

The unlock process, because this is where the master secret briefly exists in usable form:

```
Master Password entered            OR         Biometric gesture
        |                                          |
        v                                          v
+--------------------+                 +---------------------------+
| Argon2id KDF       |                 | Secure Enclave/StrongBox  |
| memory-hard,       |                 | releases hardware-wrapped |
| per-user salt      |                 | copy of vault key         |
+--------------------+                 +---------------------------+
        |                                          |
        v                                          |
 [Stretched Key]                                   |
        |                                          |
   +----+---------------------+                    |
   v                          v                    |
+----------------+   +--------------------+        |
| HKDF           |   | HKDF               |        |
| info="vault"   |   | info="auth"        |        |
+----------------+   +--------------------+        |
   |                          |                    |
   v                          v                    |
[Key-Wrapping Key]      [Auth Secret]              |
   |                    (OPAQUE exchange           |
   v                     with server, TLS)         |
Unwrap [Vault Key] <-------------------------------+
   |
   v
Decrypt vault blob (AEAD verify + decrypt)
   |
   v
Items loaded into memory-only session cache
(zeroized on lock / timeout / OS lock event)
```

What Level 2 reveals that higher levels hide: two things. First, the exact point where skipping the HKDF split would hand the server vault-key material. Second, that biometric unlock is a _parallel path around the KDF entirely_ — a hardware-wrapped key copy exists, which is a deliberate weakening of "only the master password unlocks the vault" traded for usability. Higher-level diagrams show "unlock" as one arrow and hide both facts.

---

## Section 8a — Data Flow Diagram, Level 2: Autofill and the DOM (ASCII)

, because autofill is the other place plaintext leaves the app's control — and unlike unlock, it leaves into _someone else's code_.

```
Page loads in browser tab
        |
        v
[Content Script scans DOM for login forms]
        |
        v
[Send page ORIGIN (only) to Background Worker]     <- content script never
        |                                              holds credentials yet
        v
+---------------------------------------------+
| Background Worker: eligibility decision      |
|   - exact origin match against saved entry?  |
|   - top-level frame? (cross-origin iframe    |
|     => deny by default)                      |
|   - page visible & focused?                  |
+---------------------------------------------+
        |                        |
     match                   no match
        |                        v
        v                  [No UI shown. Silence.]
[Show fill prompt UI, owned by extension,
 not injected into page DOM]
        |
        v
User gesture (explicit click/keypress)
        |
        v
[Background Worker releases credential
 to Content Script for THIS fill only]
        |
        v
[Content Script writes into input nodes]   <===  THE MOMENT OF EXPOSURE
        |                                        Any JS on the page can now
        v                                        read those DOM nodes. XSS on
[Credential erased from content script          the target site steals the
 memory immediately after write]                filled password. Nothing the
                                                 manager does can prevent it.
```

What this level reveals: the trust handoff. Everything above the "moment of exposure" line is the manager defending the secret; everything below it is the manager _having already given the secret away_ to whatever JavaScript the page runs. This is why origin matching, iframe policy, and explicit-gesture rules all sit _before_ the handoff — they are the last decisions the manager gets to make. It also shows why "the extension was never hacked" and "the password was stolen during autofill" can both be true: XSS on the destination site harvests filled credentials with zero flaws in the manager.

---

## Section 9 — User Journey Flow (ASCII)

```
[Install App / Extension]
      |
      v
[Create Account: email + master password] --(weak)--> [Strength meter, block below threshold]
      |
      v
[Client generates: vault key, keypair for sharing, recovery kit (per §13 choice)]
      |
      v
[Prompt: store recovery kit offline] --(user skips)--> [Re-prompt at day 7; badge until done]
      |
      v
[Enroll 2FA (TOTP or passkey)]
      |
      v
[Vault created, empty] -> [Import from browser/CSV offered]
      |
      v
[Add credential] -> [Encrypt item] -> [Save local] -> [Sync ciphertext up]
      |
      v
=== RETURNING USER, SAME DEVICE ===
[Open app] -> [Biometric] --(fail xN / enrollment changed)--> [Force master password]
      |
      v
[Vault unlocked] -> [Visit site] -> [Origin match?] --(no)--> [No prompt. Nothing fills.]
      |                                    |
      |                                 (yes)
      |                                    v
      |                        [User gesture confirms fill] -> [Filled] 
      |                                    |
      v                                    v
[Idle timeout / OS lock] ----------> [Vault locks, cache zeroized]

=== NEW DEVICE ===
[Login: email + master password] -> [2FA challenge] -> [Existing-device approval push]
      |                                                       |
      |                                             --(no other device)--> [2FA + cool-down delay
      |                                                                     + email/push alert]
      v
[Device registered] -> [Encrypted vault syncs down] -> [All other devices notified]

=== SHARING ===
[Select item -> Share -> pick recipient] -> [Fetch recipient public key]
      |
      v
[Optional but pushed: verify key fingerprint out-of-band]
      |
      v
[Wrap item key to recipient] -> [Recipient notified] -> [Item appears in their vault]
      |
      v
[Revoke later] -> [Stop wrapping + ROTATE item key + rotate the password itself]
                   (revocation can't un-see: they may have copied it while shared)
```

Every branch point is a security decision enforced in the user's actual path: weak-password block, biometric fallback, silent non-fill on origin mismatch (silence, not a warning dialog, so there's no "click OK to fill anyway" habit to train), new-device friction, and the honest revocation flow that admits crypto can't retract knowledge.

---

## Section 10 — Trust Boundaries

1. **Local device boundary** — separates the app's memory and encrypted storage from the rest of the OS, other apps, and malware. Crossing it: keystrokes, clipboard, screenshots, backups, memory pages. Matters because a compromised device defeats everything downstream.
2. **Client application boundary** — separates core vault logic from the autofill surfaces (extension content script, OS autofill service). Crossing it: fill requests and released credentials. Matters because the fill surfaces run in lower-trust environments by design.
3. **Extension ↔ page DOM boundary** __ — separates the content script from the web page's own JavaScript. Crossing it: form structure inward, filled credentials outward. Matters because everything past this line is attacker-controllable territory; this is the exact edge Section 8a diagrams.
4. **Network boundary** — separates client from Tier 2. Crossing it: ciphertext blobs, OPAQUE exchanges, wrapped keys, metadata. Matters least of all boundaries here (everything sensitive is already encrypted before TLS applies) — and saying so out loud is important, because teams over-invest here and under-invest at boundary 3.
5. **Tier 2 ↔ Tier 3 boundary** __ — separates internet-facing application logic from storage. Crossing it: parameterized queries, blob reads/writes under per-service credentials. Matters because it converts "bug in an API endpoint" from _database breach_ into _one service's narrow view_.
6. **Server operator boundary** — separates what the company can see (ciphertext, metadata, auth verifiers) from what only clients see (plaintext, keys, master password). Matters because this is where the zero-knowledge claim either holds or collapses.
7. **Inter-user boundary** __ — separates one user's trust zone from another's, crossed deliberately by sharing. Crossing it: wrapped item keys. Matters because the server sits in the middle of the key exchange and could substitute public keys unless fingerprints are verified.
8. **Human boundary** — separates the user's intent from everything that manipulates it: phishing pages, fake apps, MFA-fatigue push storms, support-desk social engineering. No cryptography operates here.

---

## Section 11 — Threat Model Table

|#|Attacker|STRIDE|Capability|What They Target|Mitigation|
|---|---|---|---|---|---|
|1|Network eavesdropper|I|Passive interception|Data in transit|TLS + client-side encryption underneath (double wrap)|
|2|Phishing attacker|S|Fake site / fake app clone|Master password typed into wrong place; autofill trigger|Exact-origin autofill (manager refuses where humans get fooled); passkey/2FA on account so password alone ≠ enrollment; store-signed app builds|
|3|Insider threat (vendor)|I, T, E|Production access; can push code|Server data; malicious client update|Zero-knowledge limits data value; signed reproducible builds, multi-party release approval limit update value; audit logs for repudiation|
|4|Supply chain attacker|T, E|Compromised dependency, build pipeline, or store publisher account|Vault key in memory at moment of use, before encryption|Dependency pinning, SBOM, reproducible builds, hardware-key-protected publisher accounts, pipeline attestation (SPVS control territory)|
|5|Physical device attacker|I|Stolen device, locked or unlocked|Vault file on disk; live session cache|At-rest encryption, hardware-gated keys, auto-lock on OS lock, remote device deauthorization|
|6|Malicious/compromised server operator|I, T|Full read of Tier 2/3; can serve poisoned data|Blobs, metadata, auth verifiers, public key directory|Zero-knowledge for blobs; OPAQUE so verifiers resist precomputation; key-fingerprint verification so a swapped public key is detectable|
|7|Device malware|I, E|Keylogging, memory scraping, screen capture|Master password as typed; session cache|Out of scope to fully prevent (Section 14); minimize plaintext residency, zeroize on lock, hardware-backed keys raise the bar from "read a file" to "read live process memory"|
|8|Malicious browser extension / local process|I|DOM scraping, event listeners on login forms|Credentials at the moment of fill|Extension-owned prompt UI (not page DOM), explicit gesture, immediate wipe after fill; residual risk is real and stated|
|9|Recovery-flow abuser|S, E|Triggers or intercepts recovery|The vault, bypassing all crypto|Recovery model choice + friction + alerts; Section 13|
|10|Credential stuffer _(new)_|S, D|Breached email:password lists replayed against auth API|Account takeover at the front door|OPAQUE (no offline check possible), gateway rate limits, per-account and per-IP throttles, 2FA on new devices, breach-list checks on the _account_ password|
|11|MFA-fatigue attacker _(new)_|S|Has master password; spams device-approval pushes|The human's patience|Number-matching approval (type code shown on new device), throttle approval requests, deny-and-report button|
|12|XSS on a destination website _(new)_|I|Runs JS on a site the user has credentials for|Credentials the moment they're autofilled|Cannot be prevented by the manager (Section 8a); explicit-gesture fill limits it to user-intended moments; per-site "require reprompt" option for high-value items|
|13|Mobile overlay / tapjacking attacker _(new)_|S|Draws fake UI over the real app (Android)|Master password typed into overlay; hijacked fill confirmation|Android `FLAG_SECURE`, obscured-touch rejection (`filterTouchesWhenObscured`), Play Integrity attestation; iOS analog largely closed by platform|
|14|Malicious third-party keyboard _(new)_|I|Logs everything typed on mobile|Master password|Force system keyboard on the master-password field where OS allows; biometric unlock reduces typing frequency|
|15|Backup/snapshot scavenger _(new)_|I|Reads cloud device backups, VM snapshots, decommissioned disks|Vault file copies outside the app's control|Exclude vault from OS cloud backup, or accept inclusion knowing file is encrypted; server-side: encrypted backups, snapshot access controls|
|16|Rogue share recipient _(new)_|I|Was legitimately granted access, then revoked|Continued use of a credential they saw|Honest model: revocation rotates the item key AND prompts rotating the underlying password; crypto cannot retract knowledge|
|17|Account enumerator _(new)_|I|Probes signup/login/recovery for "user exists" signals|Target list for stuffing and phishing|Uniform responses and timing on auth and recovery endpoints|
|18|Clipboard sniffer _(new)_|I|Any app polling clipboard (esp. mobile)|Copied passwords/TOTP codes|Auto-clear clipboard (30–60s), OS-level "sensitive" clipboard flags, prefer direct autofill over copy paths|

---

## Section 12 — Threat Diagram (ASCII)

```
BOUNDARY 8: HUMAN — surrounds everything. Attackers (2)(11) operate here,
against the person, not the software.

+======================= BOUNDARY 1: LOCAL DEVICE =============================+
|                                                                               |
|  (7) Malware ---> [Session Cache RAM] & keystrokes    (14) Malicious keyboard |
|  (5) Physical theft ---> [Encrypted Vault File]       (13) Overlay/tapjack    |
|  (15) Backup scavenger ---> OS cloud backup copy of vault file                |
|  (18) Clipboard sniffer ---> [Clipboard buffer]                               |
|                                                                               |
|   +----------- BOUNDARY 2: CLIENT APP ------------+                           |
|   |  [KDF]--[Crypto Engine]--[Vault Key]           |                           |
|   |  [Background Worker: fill decisions]           |                           |
|   +-----------------------|-----------------------+                           |
|                           | released credential (post-gesture)                |
|   ~~~~~~~~ BOUNDARY 3: EXTENSION <-> PAGE DOM ~~~~~~~~                        |
|                           v                                                   |
|   [ Web Page DOM ]  <--- (12) XSS on destination site                          |
|                     <--- (8) malicious extension scraping                     |
|                     <--- (2) phishing page hoping for a fill (gets silence)   |
+===========================|===================================================+
                            | BOUNDARY 4: NETWORK
                            |   (1) eavesdropper sees ciphertext only
                            v
+===================== BOUNDARY 6: SERVER OPERATOR =============================+
|  +----------------- TIER 2: APPLICATION --------------------+                 |
|  |  [API Gateway] <--- (10) credential stuffing              |                 |
|  |                <--- (17) account enumeration              |                 |
|  |  [Device Svc]  <--- (11) MFA-fatigue push storms          |                 |
|  |  [Recovery Svc] <-- (9) recovery-flow abuse  <== the bypass path            |
|  |  [Sharing Svc]  --- BOUNDARY 7: INTER-USER --- (16) rogue ex-recipient      |
|  +------------------------|----------------------------------+                 |
|          BOUNDARY 5: TIER 2 <-> TIER 3 (private network, least privilege)      |
|                           v                                                   |
|  +----------------- TIER 3: DATA ----------------------------+                |
|  |  [Blob Store] [Auth Secrets] [Metadata] [Key Directory]   |                |
|  |  (6) Malicious operator reads all of this; blobs stay      |                |
|  |      opaque; a swapped public key is the quiet attack      |                |
|  +------------------------------------------------------------+               |
|                                                                                |
|  (3) Insider sits INSIDE boundary 6 with legitimate access.                    |
|  (4) Supply chain attacker enters through build/deps/store and can            |
|      materialize inside ANY boundary above — that is what makes it worst.      |
+================================================================================+
```

### Section 12a — Threat Diagram Reference Key

|#|Plain-English Name|What It Represents|Specific Concern|Mitigation Category|
|---|---|---|---|---|
|1|Network eavesdropper|Anyone watching client↔server traffic|Reading data in transit|Transport crypto (redundant with client-side crypto)|
|2|Phishing attacker|Fake page or app|User types master password into a fake; or hopes autofill fires|Origin discipline, account 2FA, user-side silence on mismatch|
|3|Insider|Employee/contractor with prod access|Abusing legitimate access; pushing bad code|Least privilege, multi-party release, audit logging, zero-knowledge|
|4|Supply chain attacker|Poisoned dependency, build step, or store account|Trusted code that is attacker code|Pinning, SBOM, reproducible builds, publisher-account hardening|
|5|Physical device attacker|Person holding the device|Reading disk or live session|At-rest crypto, hardware key gating, auto-lock, remote deauth|
|6|Malicious server operator|The company, or whoever owns it after a breach|Reading server-side stores; swapping public keys|Zero-knowledge blobs, OPAQUE verifiers, key fingerprint verification|
|7|Device malware|Software already on the device|Keylogging, memory scraping|Exposure-window minimization; honestly out of scope to fully stop|
|8|Malicious extension/process|Other software in the browser|Scraping the DOM at fill time|Extension-owned UI, gesture requirement, immediate wipe|
|9|Recovery abuser|Anyone triggering "forgot password"|Walking through the second door|Recovery model choice (Section 13), friction, alerts|
|10|Credential stuffer|Bot replaying breached credential lists|Front-door account takeover|OPAQUE, rate limiting, 2FA, throttles|
|11|MFA-fatigue attacker|Push-approval spammer|Wearing the human down|Number-matching, request throttles, deny-and-report|
|12|XSS on destination site|Attacker script on a site you use|Harvesting the just-filled credential|Gesture-gated fill; per-item reprompt; otherwise the site's problem|
|13|Overlay/tapjacker|Fake UI drawn over the real app|Stealing typed master password; forging consent taps|FLAG_SECURE, obscured-touch rejection, platform attestation|
|14|Malicious keyboard|Third-party mobile keyboard|Logging the master password|System keyboard enforcement, biometric-first unlock|
|15|Backup scavenger|Reader of backups/snapshots|Vault copies outside app control|Backup exclusion policy, encrypted server backups|
|16|Rogue ex-recipient|Someone whose share was revoked|Using what they already saw|Item-key rotation + honest prompt to rotate the real password|
|17|Account enumerator|Prober of auth/recovery endpoints|Building a target list|Uniform responses and timing|
|18|Clipboard sniffer|Any app reading the clipboard|Copied secrets|Auto-clear, sensitive-clipboard flags, prefer autofill|

---

## Section 13 — The Hardest Unsolved Problem: Recovery

Recovery is still the one decision that can quietly undo everything else in this document, and the 3-tier redesign doesn't change that — it just gives the second door a nicer lobby.

Why it's hard, plainly: the product's core promise is _one way in, and only you hold it_. Recovery is, by definition, a second way in. The system's real security is `min(front door, second door)`. Every recovery design is an attempt to make the second door as strong as the first while still being usable by someone who has, by premise, already forgotten their secret.

Ranked options, strongest guarantee first:

1. **No recovery.** Forgotten master password = permanent loss. Cleanest security story, worst retention story, and it pushes users toward writing the master password on a sticky note, which relocates the risk rather than removing it.
2. **Offline recovery kit** (printed key generated at setup, server never sees it). Preserves zero-knowledge fully. Fails in practice because most users won't print it, will lose it, or will screenshot it into their camera roll — which silently converts it into "cloud-photo-library security."
3. **Social recovery / Shamir splitting** across trusted contacts or the user's own devices (e.g., 2-of-3 shares). Preserves zero-knowledge, survives single-point loss. Costs: real UX complexity, and it converts your threat model into your friends' threat models — each contact is now phishable on your behalf.
4. **Device-mesh recovery**: any signed-in device can approve a master password reset by re-wrapping the vault key. Great UX, preserves zero-knowledge, but it means device possession = vault ownership, which raises the stakes of attacker 5 and makes "I lost my only device" the unrecoverable case anyway.
5. **HSM-escrowed recovery** (iCloud Keychain model): a wrapped copy of the vault key is held server-side inside an HSM that will only release it after strong verification plus a user-held recovery PIN, with hardware-enforced guess limits and mandatory delays. This is the honest middle ground: it _weakens_ pure zero-knowledge (a key copy exists outside your devices) but weakens it inside auditable hardware with enforced limits, rather than pretending. Requires real HSM engineering and honest disclosure.
6. **Server-assisted email reset that restores vault access.** If this works, zero-knowledge was never true. Any vendor offering this while claiming zero-knowledge is lying in the whitepaper. This option exists in the ranking only so it can be named and rejected.

The pushback this section owes you (per standing instructions): option 2 _sounds_ like the responsible default and is what most of the industry ships, but its real-world failure rate is so high that shipping it alone is closer to option 1 with extra steps. A serious product ships 4 as the primary path, 2 as backup, and considers 5 with full disclosure. What it must never do is ship 6 and call it zero-knowledge.

---

## Section 14 — What Is Out of Scope and Why

- **Kernel-level device compromise (rootkits, malicious MDM with full control).** An attacker who owns the OS owns the keyboard, the screen, and process memory. No vault architecture survives it. This is device integrity's problem; the manager's only job here is to not make it _easier_ (no keys on disk, minimal plaintext residency).
- **Coerced disclosure (legal compulsion of the user, or violence).** A policy and physical-safety problem. A duress-PIN/decoy-vault feature is a product decision, not a core threat-model control, and it creates its own evidentiary complications.
- **Security of destination websites.** Attacker 12 (XSS on a site you log into) steals credentials the manager filled correctly into the right origin. The manager's responsibility ends at the DOM handoff, and pretending otherwise inflates the threat model with things it cannot control.
- **The user choosing a catastrophically weak master password despite the strength meter.** The KDF buys time per guess; it cannot make `password1` expensive to find. Blocking below a threshold is the control; beyond that threshold, residual risk is the user's.
- **Build-pipeline compromise depth.** Named as attacker 4 and mitigated at the control level (pinning, SBOM, attestation, reproducible builds), but the full pipeline threat model is its own document — this is SPVS's territory, not the vault architecture's.

---

## Section 15 — What the Core Security Claim Does NOT Cover

Stated plainly, no hedging:

- **Metadata is visible to the server.** Vault size, item count (if items sync individually), timestamps, device list, IP addresses, sync frequency, sharing graph (who shares with whom). Under a perfect zero-knowledge design, the company can still draw your social graph from sharing relationships.
- **Icon/favicon fetching can leak your site list.** If the client fetches site icons through the vendor's service, the vendor learns which sites you have credentials for, one request at a time. Either proxy-bundle it, fetch client-side direct, or accept the leak knowingly.
- **A malicious signed update ends everything.** The update channel is a master key to all client-side guarantees. Code signing moves the target; it does not remove it.
- **Autofilled credentials belong to the page the moment they're filled.** XSS on the destination site harvests them. Zero manager flaws required.
- **Biometric unlock means a non-password path to the vault key exists** in hardware on every enrolled device. That's a deliberate, bounded weakening — but it is a weakening, and marketing that says otherwise is wrong.
- **If recovery option 5 is chosen, a key copy exists outside user devices.** Bounded by HSM policy, but existing. If option 6 is chosen, zero-knowledge is simply false.
- **Revoking a share does not un-teach a human.** Rotation of the item key protects _future_ edits; the credential itself must be changed at the destination site.
- **RAM is not covered while the vault is unlocked.** Session cache, swap edge cases, crash dumps, and memory-scraping malware all sit inside the unlocked window. Zeroization shrinks the window; it does not close it.

---

## Section 16 — Common Implementation Mistakes in This Category

The standing instructions ask for the most common real-world failure modes. For password managers specifically, in rough order of observed frequency:

1. **Sloppy URL/origin matching in autofill** — substring matches, ignoring ports/schemes, filling into iframes. The single most repeated vulnerability class across shipped managers.
2. **Low or frozen KDF parameters for legacy users** — new users get strong settings, accounts from 2015 keep 5,000 PBKDF2 iterations forever. Parameter migration at next unlock must be designed in from day one.
3. **Auth secret not cryptographically separated from vault key** — one derived value doing both jobs, so a server breach yields crackable vault-key material. The HKDF split in Section 8 exists to kill this.
4. **Secrets in the page DOM longer than the fill instant** — prompt UI injected into the page, credentials parked in content-script state, values left in detached DOM nodes.
5. **Clipboard without auto-clear or sensitive flags** — copied passwords sitting in the clipboard through the next hour of app usage.
6. **Vault file included in cloud device backups unintentionally** — the vault is encrypted, so this is survivable, but it must be a documented decision, not an accident discovered later.
7. **Recovery flow added late, under retention pressure** — bolted-on email reset that silently voids the architecture. Recovery must be designed with the key hierarchy, not after it.
8. **Telemetry that rebuilds the vault's metadata in the analytics warehouse** — event logs with domains, item names, or per-site usage. The crypto team's guarantees die in the analytics pipeline.
9. **Trusting the server's public key directory blindly for sharing** — no fingerprint verification means the operator (or its intruder) can silently intercept every new share.
10. **Auto-submit after autofill** — removing the human gesture converts every origin-matching bug from "prompt shown" into "credential exfiltrated."

---

## Section 17 — Open Decisions Checklist

Cryptography and keys:

- [ ] Argon2id parameters (memory / iterations / parallelism) and minimum floor
- [ ] KDF parameter migration policy for existing users (upgrade at unlock)
- [ ] AEAD choice per platform (XChaCha20-Poly1305 vs AES-256-GCM w/ hardware accel)
- [ ] Per-item key derivation scheme and item-key rotation triggers
- [ ] Sharing keypair algorithm (X25519) and key-fingerprint verification UX

Authentication and sessions:

- [ ] OPAQUE vs SRP vs hash+HKDF verifier (and library maturity assessment)
- [ ] Account 2FA policy: required always vs required for new devices only
- [ ] New-device enrollment: existing-device approval flow + no-other-device fallback (delay length, alert channels)
- [ ] Session token lifetime, refresh, and remote revocation semantics
- [ ] Auto-lock triggers: timeout values, on-background, on-OS-lock

Autofill and clients:

- [ ] Origin matching: exact-only vs per-credential subdomain opt-in; iframe policy
- [ ] Fill gesture requirement and whether auto-submit is ever allowed (recommend: never)
- [ ] Per-item "reprompt master password before fill" for high-value credentials
- [ ] Content script memory-wipe verification approach
- [ ] Desktop app ↔ extension native messaging peer authentication
- [ ] Mobile: FLAG_SECURE, obscured-touch rejection, app-switcher snapshot masking
- [ ] Mobile: third-party keyboard blocking on master password field
- [ ] Biometric invalidation policy on enrollment change (recommend: invalidate)
- [ ] Root/jailbreak posture: block vs warn vs degrade
- [ ] Play Integrity / DeviceCheck attestation: enforce or log-only
- [ ] Clipboard: auto-clear timeout, sensitive flags, TOTP copy policy

Backend and operations:

- [ ] Recovery model selected (§13), residual risk written down and accepted
- [ ] HSM escrow design (only if option 5): guess limits, delays, PIN policy
- [ ] Rate limiting tiers: per-IP, per-account, per-endpoint; enumeration-resistant responses
- [ ] Tier 3 network isolation and per-service credential scoping
- [ ] Server-side encryption-at-rest and backup/snapshot access controls
- [ ] Audit log content policy (what is logged, what is forbidden, retention)
- [ ] Telemetry allow-list (recommend: deny-by-default, no domains/item names ever)
- [ ] Favicon strategy: client-direct vs proxied vs none
- [ ] OS cloud backup exclusion: exclude vault vs include-and-document

Supply chain:

- [ ] Dependency pinning + SBOM generation and review cadence
- [ ] Reproducible builds and multi-party release approval
- [ ] Extension/app store publisher account hardening (hardware keys, no shared creds)
- [ ] Update-channel signing key custody and rotation plan