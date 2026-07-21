# Cloud File Storage Service: System Design and Threat Model

## 1. What This System Is and Why It Works This Way

Think about what you actually want when you save a file to Dropbox or Google Drive. You want it to show up on your other devices. You want it to still be there if your laptop dies. You want to share a folder with a coworker without emailing giant attachments. That's the whole product from your side.

Every piece of the system exists because of one of those wants.

- You want your file on every device: that's why there's a **sync engine** watching your folder for changes and a **server** that becomes the single source of truth everyone checks against.
- You want it to survive your laptop dying: that's why the file doesn't just live on your machine, it gets copied to **durable storage** somewhere else, usually in multiple data centers.
- You want to share without huge email attachments: that's why there's a **sharing and permissions system** that hands out access to specific files or folders to specific people.
- You want it to feel instant: that's why files get broken into small pieces (**chunks**) and only the changed pieces get uploaded, instead of re-uploading the whole file every time you save it.

If you removed any one of these, the product would feel broken in an obvious way. No sync engine, and your phone never sees the file you just edited on your laptop. No durable storage, and one bad hard drive wipes out your work. No sharing system, and it's just a personal backup tool, not the collaboration tool people actually pay for.

Here's the design tension that shapes everything else in this document: the company running this service wants to see enough about your files to make search, sharing, and support work well. But the more they can see, the more there is to steal, subpoena, or leak. That tension between "useful" and "private" is the real design problem, not the file syncing part, which is a solved engineering problem at this point.

## 2. Why Security Matters Here

If you used a service like this, here's what you'd actually worry about, in the order most people worry about it:

**"What if someone hacks my account?"** This is the number one fear, and it's the right one. Most real-world breaches of file storage happen because someone's password got reused from another site that got hacked, or someone got phished. The system itself getting broken into is rarer than your front door key getting copied.

**"What if the company can read my files?"** This is the one companies don't like to answer directly. For almost every mainstream file storage service, the honest answer is yes, they can, because they hold the encryption keys. They encrypt your files so a stolen hard drive is useless to a thief, but that's different from the company itself being unable to read them. We'll build a version here where that's not true, and you'll see what it costs to get there.

**"What if I share a folder with someone and then need to un-share it?"** This is where people get surprised. If someone already downloaded a file before you revoked their access, revoking access doesn't reach into their laptop and delete it. Revocation stops future access. It doesn't undo past access. That's not a bug, it's just what "access" means once a copy exists somewhere else.

**"What if I lose my password?"** Normally the company resets it for you. But if we build this so the company genuinely cannot read your files, then the company also genuinely cannot help you if you lose the key. There's no secret backdoor for good reasons that also works for you when you're locked out. This is the hardest problem in the whole design, and it's Section 13.

Who gets hurt when this goes wrong isn't abstract. It's the freelancer whose client contracts leak. It's the company whose product roadmap gets read by a competitor because an employee's account got phished. It's the person whose private photos get exposed because a share link they thought was private turned out to be guessable or indexed by a search engine.

"Secure enough" for this category of system means: a stolen server or a subpoena gets the attacker ciphertext, not your files. A phished password doesn't get past MFA. A shared file's access can be revoked going forward. And nobody markets "encrypted" without being specific about who holds the key, because that word alone is doing a lot of quiet, misleading work in this industry.

## 3. Architecture Components

**Client-side**

- Sync engine: watches the local folder, detects changes, talks to the server
- Local encrypted cache: stores chunk metadata and sync state on disk
- Key management module: derives and stores your encryption keys, never sends the unwrapped key to the server
- Chunker: splits files into small content-based pieces so only changed pieces re-upload
- Encryption module: encrypts each chunk before it leaves the device
- Local session cache: holds your login session so you're not typing a password every time

**Server-side**

- API gateway / auth service: handles login, session tokens, multi-factor authentication
- Metadata service: keeps track of your file tree, versions, folder structure, who has access to what
- Block storage service: stores the actual encrypted chunks, addressed by content hash
- Dedup service: checks if a chunk already exists somewhere so it doesn't get stored twice
- Sharing/ACL service: issues and revokes access grants, manages share links
- Sync/notification service: tells your other devices "something changed, go fetch it"
- Audit log service: records who did what, in a way that's hard to quietly edit after the fact
- Key escrow/wrapping service: holds only wrapped (encrypted) keys, never your actual content key
- Malware/DLP scanning service: scans files for malicious content, which only works on plaintext

**Data stores**

- Object storage: where the actual encrypted chunks physically live
- Metadata database: file trees, permissions, version history
- Audit log store: append-only, tamper-evident
- Session/token store: active login sessions

**Third-party integrations**

- Object storage backend (S3-class provider)
- KMS/HSM: hardware-backed key management, used for metadata encryption, not for your personal content key
- Identity provider / SSO (Okta, Azure AD, Google Workspace)
- CDN: delivers public share links quickly
- Payment processor
- Push notification providers for mobile

## 4. Core Design Decisions

|Layer|Choice|Rationale|Plain-English Explanation|
|---|---|---|---|
|Content confidentiality|Client-side encryption, one key per file|Server compromise or subpoena hands over ciphertext, not readable files|Your device encrypts the file before it ever leaves, so the server only ever sees scrambled data|
|Chunking method|Content-defined chunking (splits based on the data itself, not fixed sizes)|Editing the middle of a file doesn't force re-uploading the whole thing|The chunk boundaries shift with the content, so only the changed piece looks different after an edit|
|Dedup scope|Per-user or per-organization only, never across unrelated customers|Company-wide dedup lets one user check whether another user already has a specific file|Trades some storage savings for not leaking "does this exact file already exist somewhere"|
|Metadata visibility|Filenames, folder structure, and sharing graph are visible to the server in plain text by default|Fully encrypting this too breaks search, previews, and the sharing UI unless you build a lot more|This is the part of "your files are private" that usually isn't true, and it should be stated up front|
|Key storage|Master key comes from your passphrase (Argon2id, a slow hashing function meant to resist guessing), then gets copied to each of your devices in a wrapped form|No server anywhere holds your actual usable key|If you forget the passphrase, there is no "reset" button that also keeps this promise true|
|Sharing model|Each person you share with gets their own wrapped copy of the file's key, instead of everyone sharing one secret|Lets you revoke one person's future access without breaking it for everyone else|Public "anyone with the link" shares are a separate, weaker category, and should be treated as such|
|Transport security|TLS 1.3 with certificate pinning on official apps|Stops someone snooping or tampering with traffic in transit|Pinning means the app only trusts one specific certificate, which also breaks some corporate network monitoring tools on purpose|
|Malware/DLP scanning|Turning this on for a file or folder turns off the "we can't read it" claim for that content|You can't scan something you can't read|If a vendor claims both full encryption and full malware scanning at the same time, one of those claims is false|
|Client software integrity|Signed app builds, ideally with a public log proving what was built and shipped|An update channel that pushes a bad build to one target defeats every other protection at once|The update system is part of the trust boundary whether or not it gets diagrammed that way|
|Audit logging|Append-only, chained so a record can't be quietly edited|Lets you detect an insider or a break-in after the fact|This catches tampering, it doesn't prevent someone with access from looking in the first place|

## 5. Architecture Diagram

```
LEGEND: [C] Client component  [S] Server component  [D] Data store  [3P] Third party
        --> data flow direction

   +-------------------------+
   |  [C] Sync Engine         |
   |  [C] Chunker             |
   |  [C] Encryption Module   |
   |  [C] Key Mgmt Module     |
   |  [C] Local Encrypted Cache|
   +------------+-------------+
                | (1) encrypted chunks + metadata requests
                v
   +-------------------------+
   |  [S] API Gateway/Auth    |<---- [3P] Identity Provider (SSO)
   +------------+-------------+
                |
     +----------+-----------------------+
     v                                  v
+-------------+                 +----------------+
| [S] Metadata |<--------------->| [S] Dedup Svc   |
| Service       |                | (chunk lookup)   |
+------+-------+                 +--------+-------+
       |                                  |
       v                                  v
+-------------+                 +----------------+
| [D] Metadata |                 | [S] Block       |
| DB (tree,     |                | Storage Service  |
| ACLs, shares) |                +--------+-------+
+-------------+                          |
                                          v
                                 +----------------+
                                 | [3P] Object     |
                                 | Storage (S3)     |
                                 +----------------+

   [S] Sync/Notification Service --> pushes change events --> other devices
   [S] Audit Log Service --> [D] Audit Log Store (append-only)
   [S] Sharing/ACL Service --> issues wrapped keys to recipients
   [S] Malware/DLP Scanning --> only runs if E2EE is disabled for that file
```

## 6. Data Flow Diagram, Level 0

```
                +------------------+
   User -------->                   |
   (upload/      |  Cloud File       <-------- Identity Provider
    download/    |  Storage System   |          (login/SSO)
    share)        |                   |
   User <--------+                   +--------> Object Storage
                +------------------+           (durable backend)

                        ^
                        |
              Shared-with User
           (receives access grant,
            downloads/decrypts file)
```

This level teaches the boundary of the whole system as one box. It shows who talks to it (the owning user, someone they share with) and what outside services it depends on (identity provider, object storage). Nothing about internal structure is visible yet, on purpose. This is the "what does it connect to" view.

## 7. Data Flow Diagram, Level 1

```
User --> [Sync Engine] --> [Chunker] --> [Encryption Module] --> encrypted chunks
                                                                        |
                                                                        v
[API Gateway/Auth] <---- login/session ---- [Identity Provider]
        |
        v
[Metadata Service] <--file tree, ACLs--> [Metadata DB]
        |
        v
[Dedup Service] --checks hash--> [Block Storage Service] --> [Object Storage]
        |
        v
[Sharing/ACL Service] --wrapped key per recipient--> Recipient's Key Mgmt Module
        |
        v
[Sync/Notification Service] --change event--> other devices
```

This level breaks the single box from Level 0 into its major moving parts and shows where data actually goes between them. This is where you can first see the real design decisions: encryption happens before the chunker's output ever reaches the API gateway, and the dedup check happens after encryption, on ciphertext, not on the original file.

## 8. Data Flow Diagram, Level 2 (Sharing Service, the most security-sensitive process)

```
Owner requests share with Recipient
        |
        v
[Sharing/ACL Service]
        |
        +--> Step 1: Look up Owner's file content key (still wrapped)
        |
        +--> Step 2: Unwrap content key using Owner's device key
        |            (this unwrap happens ON THE OWNER'S DEVICE, never on the server)
        |
        +--> Step 3: Re-wrap content key using Recipient's public key
        |
        +--> Step 4: Store new wrapped-key record in Metadata DB,
        |            tagged to Recipient's identity
        |
        +--> Step 5: Notify Recipient's device of new grant
        |
        v
Recipient's device fetches wrapped key, unwraps locally with own private key
        |
        v
Decision point: is Recipient's device key still valid / not revoked?
   |-- Yes --> Recipient can decrypt file
   |-- No  --> Access denied, grant exists in DB but is inert
```

This level shows what the higher-level boxes hide: the unwrap-and-rewrap of the content key never happens on the server. If it did, the server would briefly hold a usable key, and the whole "server can't read your files" claim would quietly become false at exactly this step. This is the step where an implementation mistake would be invisible from Level 0 or Level 1 but would break the entire security model.

## 9. User Journey Flow

```
[User opens app]
      |
      v
[Enter passphrase] --> Correct? --No--> [Error: retry, limited attempts]
      |
     Yes
      |
      v
[Device key unwrapped locally]
      |
      v
[Sync engine checks for changes]
      |
      v
[User drags file into folder]
      |
      v
[File chunked + encrypted locally]
      |
      v
[Chunks uploaded] --> Upload fails (network)? --Yes--> [Retry queue, show "syncing" spinner]
      |
     No
      |
      v
[Other devices notified] --> [Other devices fetch new chunks] --> [File appears there too]
      |
      v
[User right-clicks file, selects "Share"]
      |
      v
[Enter recipient email] --> Recipient has an account? --No--> [Invite sent, share pending]
      |
     Yes
      |
      v
[Content key wrapped for recipient, grant recorded]
      |
      v
[Recipient sees file, downloads, decrypts locally]
```

Each box is something the user actually sees on screen: a spinner, an error message, a new file appearing, an invite email. This is the view that connects the abstract data flow diagrams back to what a real person experiences when something goes right or wrong.

## 10. Trust Boundaries

1. **Client device to local operating system.** The app trusts the OS to keep its memory and disk isolated from other processes. Malware with OS-level access sits inside this boundary already and the app can't defend against it.
2. **Client to network.** Everything before this point is plaintext-adjacent on the device. TLS protects it once it leaves.
3. **Network to API gateway.** This is where an anonymous internet connection becomes an authenticated one. Everything past this point assumes the login was legitimate.
4. **API gateway to internal services.** Internal services trust that the gateway correctly verified who's asking. If that trust is misplaced, every internal service inherits the mistake.
5. **Metadata service to block storage service.** The metadata service knows the file tree and structure. Block storage should only ever see meaningless encrypted blobs and hashes, never filenames or folder context.
6. **Provider infrastructure to third-party object storage.** The company doesn't fully control the physical security or staff of whoever runs the actual disks underneath their object storage.
7. **Owner to share recipient.** The moment a share grant is issued, the recipient becomes a new, independent point of trust for that content. The owner can revoke future access but can't reach into a device that already decrypted the file.
8. **Provider to legal and regulatory environment.** A subpoena or lawful order cuts across every boundary above at once, because it compels the company, not any single system.
9. **Client update channel to the running client app.** Code signing decides whether a compromised update system can silently rewrite what "the app" even does. This boundary is easy to forget because it doesn't show up in a normal architecture diagram.

## 11. Threat Model Table

|Attacker|Capability|What They Target|Mitigation|
|---|---|---|---|
|External passive network attacker|Can sniff traffic between client and server|Session tokens, file content in transit|TLS 1.3, certificate pinning|
|External active network attacker (MITM)|Can intercept and alter traffic, potentially with a rogue certificate authority|File integrity, login credentials|Certificate pinning, HSTS, certificate transparency monitoring|
|Malicious or compromised company insider|Direct access to production databases, object storage, key management systems|Metadata, ciphertext, and (if the design is done wrong) actual content keys|Content keys never touch the server unwrapped; sensitive internal actions require two-person approval and are audit logged|
|Company compelled by legal process|Can be legally ordered to hand over anything the company holds|Whatever data exists on company servers|Ciphertext and metadata only get handed over; content plaintext genuinely doesn't exist on the server if the design holds|
|Compromised client endpoint (malware on the user's device)|Full plaintext access, can log the passphrase as it's typed|Everything the user can see|Out of scope for the service itself; device attestation raises the cost but can't eliminate this attacker|
|Malicious share recipient|Legitimate access they were given, can re-share or copy out what they received|The specific files shared with them|Per-recipient key wrapping and revocation; can't stop someone from copying a file after they've already decrypted it|
|Holder of a leaked public share link|Anyone with the URL or token gets access, no login required|Whatever that link points to|Treat public links as bearer tokens: default expiry, no confidentiality guarantee against the company, optional watermarking|
|Attacker who compromises the client build/update pipeline|Can ship a backdoored app to some or all users|Every user's key material, silently, at scale|Signed builds, reproducible builds, a public transparency log so a targeted bad build can be independently detected|
|Attacker who achieves RCE on the metadata service|Full read/write of the file tree, permissions, and sharing graph|Metadata, not content, under a correctly built E2EE design|Doesn't yield file plaintext, but is still a serious breach on its own, since the sharing graph alone reveals who works with whom|
|Attacker who compromises the object storage backend|Read access to stored encrypted chunks|Ciphertext blobs|Ciphertext-only exposure; still a real incident if chunk-hash patterns allow correlation across files|
|Dedup confirmation attacker|Already has a copy of a target file, checks whether it exists in someone else's storage by observing a dedup hit|Confirms a specific file exists somewhere, without decrypting anything|Dedup scoped per-tenant, never global, so cross-account confirmation isn't possible|
|Attacker with a stolen or phished password|Full account access if multi-factor authentication is absent or weak|Login session, and everything reachable from it|MFA on login; but MFA alone doesn't protect a content key that's cached and derivable client-side once logged in|

## 12. Threat Diagram

```
                (8) Legal/Regulatory boundary spans everything below
+---------------------------------------------------------------------+
|                                                                       |
|   [5] Malware on device --> attacks -->                              |
|   +-----------------+                                                |
|   | [C] Client Device |<--[8] targeted bad update via update channel |
|   | Sync/Key Mgmt/Cache|    (boundary 9)                              |
|   +---------+---------+                                              |
|             | boundary (1) device / (2) network                      |
|   [1][2] Passive/Active MITM --> attacks TLS link -->                |
|             |                                                        |
|   +---------v---------+                                              |
|   | [S] API Gateway/Auth|<--[11] phished/stuffed credential attacker |
|   +---------+---------+                                              |
|             | boundary (3)(4)                                        |
|      +------+------------------------+                               |
|      v                               v                               |
| +-----------+                 +--------------+                      |
| | Metadata   |<--[9] RCE attacker on metadata service                |
| | Service     |                 | Dedup Svc     |<--[10] confirmation|
| +-----+------+                 +------+-------+     attacker         |
|       |    boundary (5)               |                              |
|       v                               v                              |
| +-----------+                 +--------------+                      |
| | Metadata DB|                 | Block Storage |                      |
| +-----------+                 +------+-------+                      |
|                                       | boundary (6)                  |
|                                       v                               |
|                              +----------------+                      |
|                              | Object Storage  |<--[8] object storage |
|                              | (3rd party)      |    compromise       |
|                              +----------------+                      |
|                                                                       |
|   [3] Company insider --> attacks --> Metadata DB, Object Storage,   |
|       Key Wrapping Service directly (not shown as a single arrow,    |
|       insider has broad internal reach by definition)                |
|                                                                       |
|   boundary (7) sharing boundary                                      |
|   +----------------------+                                           |
|   | Recipient's Client     |<--[6] malicious share recipient          |
|   +----------------------+<--[7] leaked public link holder           |
|                                                                       |
+---------------------------------------------------------------------+
```

### 12a. Threat Diagram Reference Key

|Element|Plain-English Name|What It Represents|Concern It Maps To|Mitigation Category|
|---|---|---|---|---|
|[1][2]|MITM on network link|Someone between the client and server|Traffic interception/tampering|Transport security (TLS, pinning)|
|[3]|Company insider|Employee or contractor with production access|Unauthorized internal data access|Access control, audit logging, key architecture|
|[4]|Legal compulsion|Government or court order to the company|Forced data disclosure|Architecture that limits what exists to disclose|
|[5]|Compromised endpoint|Malware on the user's own device|Plaintext exposure at the edge|Out of scope; device attestation as partial mitigation|
|[6]|Malicious recipient|Someone legitimately given access who misuses it|Post-decryption exfiltration|Per-recipient keys, revocation, DLP as a policy layer|
|[7]|Leaked public link|Anyone who obtains a shared URL|Unauthenticated access to a specific resource|Link expiry, no confidentiality promise for this tier|
|[8]|Update pipeline / storage backend compromise|Attacker controlling build distribution or the physical storage layer|Mass or physical-layer compromise|Signed/reproducible builds, transparency logs, vendor attestations|
|[9]|RCE on metadata service|Remote code execution on a specific internal service|Metadata and sharing graph exposure|Service isolation, least privilege, monitoring|
|[10]|Dedup confirmation attacker|Someone checking if a known file exists elsewhere|Existence-confirmation leak|Per-tenant dedup scoping|
|[11]|Credential attacker|Phishing or credential stuffing against login|Account takeover|MFA, anomaly detection on login|

## 13. The Hardest Unsolved Problem

**Key recovery versus the zero-knowledge promise, made harder by sharing.** This is the direct equivalent of the recovery problem in a password manager, and it's actually worse here, because content in this system also gets shared with other people, not just stored for one person.

Here's why it's genuinely hard, not just an engineering gap: if the company can help you recover a forgotten passphrase, that means the company has some way to get back to your usable key. If the company has any way to do that, the company can also be compelled, hacked, or socially engineered into using that same path against you. There is no version of "the company can help you" that doesn't also mean "the company technically can read your files." Those two things are the same capability described two different ways.

Ranked options and what each one costs:

1. **No recovery at all.** Forget your passphrase, lose your data. Strongest possible security claim, zero trust required in the company. Cost: guaranteed permanent data loss for a real percentage of users, and it's genuinely hostile to businesses, since an admin will demand an override for departing employees, which quietly reintroduces a trusted party anyway.
2. **Social recovery, split across trusted contacts.** No single point of trust; a threshold of people you chose have to cooperate to help you recover. Cost: complicated for normal people to set up, doesn't work if you don't have several people you trust, and a colluding threshold of those contacts can recover your key without you.
3. **Company-held recovery key, wrapped by a second company-held key.** Best user experience, matches what people expect from Dropbox or Drive today. Cost: this is the option that quietly ends the "we can't read your files" claim. If this is what you ship, say so plainly, don't call it end-to-end encryption.
4. **Enterprise admin recovery key via a hardware security module.** Makes sense when the organization, not the individual employee, is supposed to be the trust root. Cost: employees have zero confidentiality from their own employer, which is fine if disclosed at onboarding and dishonest if marketed as personal privacy.
5. **Recovery requires approval from a second registered device.** A middle-ground option. Cost: someone with only one device and no backup has no recovery path at all, which is functionally the same outcome as option 1 for that user.

There is no option on this list that gives you both "the company can never read your files" and "you can always get your files back." You have to pick one and say which one out loud, in plain language, to users. Most companies avoid saying it because the honest answer is less reassuring than the vague word "encrypted."

## 14. What Is Out of Scope and Why

|Item|Why It's Not Covered Here|
|---|---|
|Physical data center security|Delegated to the object storage provider's own compliance certifications (SOC 2, ISO 27001); re-verifying it is a vendor audit exercise, not an architecture decision|
|Device or OS-level endpoint security|A rooted phone or a compromised OS keychain is a fight the app itself can't win; it's a separate, mature discipline (endpoint security)|
|DDoS and general availability engineering|This document is about confidentiality and integrity, not uptime; availability threat modeling is its own separate exercise|
|Internals of the malware scanning engine|Treated as a black box dependency; its detection accuracy is a product risk, not something this architecture can fix|
|Billing and payment fraud|Falls under separate PCI-scoped systems with their own threat model entirely|
|The internal legal process for responding to a subpoena|The boundary itself is modeled (Section 10, #8); the internal workflow for handling a specific legal request is a legal/operations process, not an architecture one|

## 15. What the Core Security Claim Does NOT Cover

- **Metadata is not protected by default.** Filenames, folder structure, file sizes, the sharing graph, and access timing are all visible to the company even with full content encryption, unless encrypted metadata is specifically built, which most designs, including the default version of this one, don't include.
- **A compromised device defeats everything.** Encryption protects data sitting on company servers and moving across the network. It does nothing once the plaintext is sitting on a device that's already compromised.
- **Traffic patterns are still visible.** Someone watching network traffic, even without breaking the encryption, can see file sizes, how often you sync, and when you share things.
- **Nothing stops a legitimate recipient from copying data out after they've decrypted it.** Screenshotting or re-uploading a file elsewhere after receiving legitimate access is a policy problem, not something cryptography can prevent.
- **Any feature that needs plaintext to work (search, thumbnails, malware scanning, OCR) cannot also claim full encryption for the content it touches.** Either it's an explicit opt-in exception per file or folder, or the encryption claim is false. There's no middle option.
- **A company legally compelled to push a targeted bad update to one specific user isn't fully stopped by code signing alone.** That requires a public transparency log with independent verification, which almost no company actually runs in production today.

## 16. Open Decisions Checklist

- [ ] Which recovery model from Section 13 gets shipped, stated to users in plain language, not marketing language
- [ ] Metadata encryption level: none, filenames only, or full, each with a real usability cost
- [ ] Dedup scope: per-user, per-organization, or global, and whether the storage savings of global dedup are worth the confirmation-attack risk
- [ ] Public share link defaults: expiry period, revocation behavior, whether the owner sees a log of who accessed the link
- [ ] Server-side scanning default: on by default (which quietly disables full encryption by default) or off by default with explicit opt-in, and how that tradeoff gets disclosed
- [ ] Enterprise admin override: does an org admin get a master recovery key, and is that disclosed to employees before they onboard
- [ ] Build and release transparency: reproducible builds, signed releases, and whether a public transparency log actually gets maintained
- [ ] Corporate TLS inspection compatibility: certificate pinning breaks these tools on purpose, decide whether managed enterprise fleets get a pinning bypass and what that does to the MITM mitigation
- [ ] Revocation semantics for shared content: state clearly that "revoke" stops future access, it does not undo what a recipient already downloaded and decrypted

---

**Where this design pushes back on something that sounds secure but isn't:** "Files are encrypted" and "end-to-end encrypted" are not the same claim, and treating them as interchangeable is the most common misleading move in this entire product category. Encryption at rest protects against someone stealing a physical disk. It does nothing against a subpoena, an insider, or a server breach, because the company still holds the keys. If a product only says "your files are encrypted" without saying who holds the key, that's the exact gap to push on before it ships.

**The most common implementation mistake in this category:** convergent encryption for cross-tenant deduplication, meaning each chunk gets encrypted with a key derived from the chunk's own content hash, so identical files from different users produce identical ciphertext. It looks like a clever way to save storage while "still encrypting." It's broken by design: anyone who already has a copy of a target file can upload it and watch for a dedup hit to confirm that exact file exists somewhere in storage, and two users with the same file become linkable by comparing ciphertext, without either copy ever being decrypted. Scope dedup to the tenant boundary, or accept and disclose the confirmation-attack exposure. Don't ship it quietly.