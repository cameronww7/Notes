# Comprehensive AI Security Controls Checklist

## Strategic Foundation & Governance

### AI Security Program Establishment
**Implementation Priority: P0 (Week 1-4)**

[ ] AI Security Working Group: Establish cross-functional team (Security, Legal, Privacy, Engineering, Business) meeting monthly to review AI risks, policies, and governance decisions.

[ ] AI-SPM Deployment: Deploy AI Security Posture Management platform with cloud account access to automatically discover AI assets (models, agents, MCPs, API usage) across environments.

[ ] Shadow AI Discovery: Deploy network monitoring for unauthorized LLM API calls; scan code repositories for API keys; use endpoint detection for local AI tools; integrate findings into AI inventory.

[ ] Shadow AI Amnesty Program: Establish reporting mechanism for teams to register previously untracked AI systems without penalty; communicate broadly to encourage compliance.

[ ] Implementation Roadmap: Define phased rollout plan with clear milestones: P0 (Weeks 1-4), P1 (Months 1-3), P2 (Months 3-6), P3 (Months 6+).

---

## Centralized AI Gateway Architecture
**Implementation Priority: P1 (Month 1-3)**

### Gateway Deployment & Configuration
[ ] AI Gateway Selection: Deploy centralized AI gateway solution (API gateway with AI plugins, standalone AI gateway, or proxy architecture) positioned between applications and AI services.

[ ] Multi-Model Support: Ensure gateway supports OpenAI, Anthropic, Google, AWS Bedrock, Azure OpenAI, and other organizational AI providers.

[ ] MCP Protocol Awareness: Configure gateway to inspect and govern Model Context Protocol traffic, not just HTTP/REST APIs.

[ ] Agent-Aware Security: Enable gateway to understand multi-step agent workflows and apply policies across action chains.

[ ] Real-Time Decisioning: Configure gateway to block risky requests in milliseconds without degrading user experience.

### Gateway Security Controls
[ ] Prompt Injection Detection: Deploy automated scanning of all inputs for known prompt injection patterns before requests reach models; block or flag suspicious patterns.

[ ] Data Loss Prevention (DLP): Implement real-time inspection of prompts and responses for sensitive data (PII, PHI, credentials, proprietary information); automatic redaction or blocking.

[ ] Content Filtering: Configure bidirectional content filters blocking inappropriate, policy-violating, or malicious content in prompts and model outputs.

[ ] Output Validation: Implement automated validation ensuring AI responses meet safety, quality, and policy standards before delivery to users.

### Gateway Governance & Observability
[ ] Policy Enforcement: Configure gateway to block unauthorized AI services and enforce approved use cases; maintain approved service catalog.

[ ] Usage Tracking: Implement comprehensive logging of which users/teams use which AI services, for what purposes, with what data, and at what cost.

[ ] Cost Management: Deploy monitoring and alerting for AI API spend by team/project/use case; set budgets and spending limits; prevent Denial-of-Wallet attacks.

[ ] Audit Logging: Configure comprehensive, tamper-evident logging of all AI interactions for compliance requirements; retention ≥ 1 year (or per policy).

[ ] Performance Metrics: Monitor and alert on latency, error rates, availability, token consumption, and quality metrics across AI services.

[ ] Anomaly Detection: Implement behavioral analytics identifying unusual usage patterns (volume spikes, off-hours access, anomalous data patterns) that might indicate compromise.

[ ] Rate Limiting: Configure per-user, per-team, and per-application rate limits to prevent abuse and manage costs; alert on threshold violations.

### Gateway Integration
[ ] SIEM Integration: Connect gateway logs and alerts to central SIEM for correlation with other security events; define detection rules for AI-specific threats.

[ ] DLP Integration: Integrate gateway with enterprise DLP solution for consistent policy enforcement and centralized sensitive data management.

[ ] IAM Integration: Connect gateway to identity providers (SSO/OIDC) for authentication and authorization; enforce consistent access policies.

[ ] Secret Manager Integration: Integrate gateway with approved secret management solutions for secure API key/token storage and rotation.

---

## Technology-Specific Controls

## Generative AI (LLMs, Chatbots, Copilots)
**Implementation Priority: P0 (Week 1-4) for foundational controls, P1 (Month 1-3) for advanced**

### Governance, Inventory & Risk
[ ] Inventory: The GenAI system (model/provider, app, integrations) is registered in the AI inventory with owner(s), environment(s), use case, training data sources, access permissions, and deployment model documented.

[ ] Security Risk Assessment: Use case is reviewed and approved by security team with documented threat model covering prompt injection, data leakage, model misuse, and supply chain risks.

[ ] Risk Tiering: Classify models into risk tiers (low/medium/high/critical) based on data sensitivity, decision impact, and access scope; apply controls proportionally.

[ ] Model Approval Workflow: Implement approval process requiring security sign-off before production deployment; higher-risk tiers require additional reviews (Legal, Privacy, Compliance).

[ ] Model Lifecycle Policy: Establish model sunset policy flagging models >18 months old for security review and potential retirement; document exceptions and risk acceptance.

### Data Security & Privacy
[ ] Encryption: TLS 1.2+ (Prefer TLS 1.3) for all traffic; AES-256 (or equivalent) at rest for logs, transcripts, embeddings, and indexes.

[ ] Data Minimization & Purpose Limitation: Only necessary data is sent to the model; prompts/context are scrubbed to remove sensitive data not essential for the task.

[ ] Sensitive Data Controls: Confidential/regulated data is not sent to public/third-party models without explicit authorization; enforce automatic redaction/DLP on prompts and outputs.

[ ] Retention & Deletion: Prompt/response retention is the minimum necessary; subject-rights and deletion/rectification mechanisms exist for stored transcripts/telemetry.

### Access Control, Authentication & Secrets
[ ] Least Privilege: RBAC/ABAC enforced for application, model gateways, vector databases, and tools; separate duties for development, approval, and release.

[ ] SSO + MFA: Required for administrative and sensitive operations (e.g., prompt library, safety policy config, model routing).

[ ] Modern Authentication: OAuth 2.1/OIDC (or mTLS) for APIs; short-lived tokens with audience/claim checks; no token passthrough or shared accounts.

[ ] Secrets Management: No hardcoded secrets; secrets stored in approved managers; pre-commit secret scanning; routine rotation (≤90 days).

### Prompt & Context Management
[ ] Secure Prompt Templates: Embed security guidance in prompts (including input validation, AuthN/Z requirements, and safe-output handling).

[ ] Context Hygiene: Remove or redact secrets and PII before processing or sharing.

### Retrieval-Augmented Generation (RAG)
*These controls apply when building a GenAI solution. These would be inherited controls when leveraging a 3rd party GenAI solution (e.g., Microsoft Copilot).*

[ ] Approved Sources: Retrieval indexes are built only from approved and licensed sources; content allowlists are enforced.

[ ] Grounding Quality: Evaluate grounding accuracy; require source citations in outputs where possible; block hallucinated citations.

[ ] Index Security: Embeddings and vector stores are encrypted; PII is tokenized or excluded; document retention/TTL for indexes; access is least-privilege.

[ ] Freshness & Drift: Re-index schedules and freshness checks are defined; detect source content changes that may invalidate answers.

### Tool/Function Calling & Integrations
[ ] Least-Privilege Tooling: Each tool/API has scoped credentials; no long-lived tokens; per-call authorization at the gateway/PDP.

[ ] Sandboxing: Execution environments are isolated with resource limits and read-only filesystems where feasible.

[ ] Human-in-the-Loop (HITL): High-risk actions (payments, admin configuration, data deletion) require explicit human review/approval.

### Output Safety & Execution
*These controls apply when building a GenAI solution. These would be inherited controls when leveraging a 3rd party GenAI solution (e.g., Microsoft Copilot).*

[ ] Sanitization: Never auto-execute model-generated commands or code; sanitize/escape outputs before rendering or using downstream.

[ ] Content Safety: Apply toxicity, malware, jailbreak, and policy filters (pre/post); block known exfiltration patterns and indirect injection markers.

[ ] Disclosure & Provenance: Clearly label AI-generated content and disclose limitations; enable provenance/watermarking where supported.

### Model/Provider Assurance & Supply Chain
[ ] Vendor Assurance: Third-party providers offer SOC 2 Type II or ISO 27001; document model safety practices and incident processes; right to audit where feasible.

### Change, Drift & Lifecycle
[ ] Controlled Releases: Canary/blue-green deployments with rollback criteria; re-run eval suites on every model/prompt/policy change.

[ ] CI/CD Security Gates: Implement automated gates blocking production deployment without security sign-off; require passing security tests.

[ ] Drift Monitoring: Monitor performance drift (accuracy, toxicity, hallucination) and trigger retraining/re-prompting or model swap when thresholds are breached.

### Testing & Adversarial Evaluation
**Implementation Priority: P2 (Month 3-6)**

[ ] Prompt Injection Testing: Integrate automated prompt injection test suites into CI/CD pipelines; use established attack patterns (jailbreaks, context poisoning, indirect injection); block deployments failing injection tests.

[ ] Bias Testing: Test models for demographic fairness using established frameworks (Fairlearn, AI Fairness 360); measure disparate impact across protected attributes; document bias mitigation strategies.

[ ] Adversarial Testing: Generate adversarial inputs designed to fool models; measure robustness against edge cases and manipulated inputs; establish acceptance thresholds.

[ ] Red Team Exercises: Conduct quarterly red team exercises simulating attacker tactics (prompt injection, data exfiltration, policy bypass) against AI systems; document findings and remediation.

### Cost & Usage Management
[ ] Cost Monitoring: Track AI API spend by team/project/use case; set budgets and alert on overages; prevent Denial-of-Wallet attacks.

[ ] Usage Analytics: Monitor API call volumes, token consumption, and inference latency; detect anomalous usage patterns indicating abuse or compromise.

### Incident Response
[ ] Runbooks: IR playbooks for data leaks, jailbreak outbreaks, indirect-injection incidents, tool abuse, and mass hallucination; include kill-switch steps and credential revocation.

[ ] SOC Notification: Report AI incidents within 24 hours; conduct RCA; implement corrective actions; re-test controls.

---

## AI-Assisted Coding
**Implementation Priority: P1 (Month 1-3)**

### Governance, Inventory & Risk
[ ] Inventory: AI-assisted coding tools/workflows are registered in the AI inventory with owner(s), environment(s), repos/projects, IDEs, and use cases documented.

[ ] Risk Assessment: A threat model and risk analysis are completed for the use case (adversarial risks, data exfiltration via prompts, training-data leakage, dependency risks, supply chain, model misuse).

[ ] Tool Approval: Only approved assistants/plugins (by security review) may be used; personal accounts prohibited; telemetry sharing scoped to minimum.

### Data Security & Privacy (Prompts, Context, Telemetry)
[ ] Encryption: TLS 1.2+ (Prefer TLS 1.3) in transit; AES-256 (or equivalent) at rest for logs, code indexes, and prompt/response archives.

[ ] Data Minimization & Purpose Limitation: Only necessary code/context is shared with assistants; disable sending large repository snapshots by default; sensitive files are excluded.

[ ] Sensitive Data Safeguards: Confidential/regulated data is not entered into public/third-party AI tools unless explicitly authorized, with DPA/SCCs in place and automated redaction/DLP on prompts.

[ ] Repository Access Audit: Document what data coding assistants can access; restrict access to highly sensitive repositories (security tooling, auth systems, cryptographic implementations).

### Access Control, Authentication & Secrets
[ ] Least Privilege: Enforce RBAC/ABAC for repos, CI/CD, and assistant backends; separate duties for commit, review, and release.

[ ] SSO + MFA: Required for developer tooling, code hosts, pipelines, and administrative consoles.

[ ] Modern Auth: Use OAuth 2.1/OIDC (or mTLS) for tool/service APIs; short-lived tokens with audience/claim validation; no token passthrough.

[ ] Secrets Management: No hardcoded secrets; store only in approved secret managers; enable pre-commit secret scanning; rotate tokens/keys routinely (≤90 days).

### Secure Development & Supply Chain
[ ] SSDLC: Threat modeling, security requirements, code review, and verification gates are required for agent frameworks/orchestrations.

[ ] Security Scanning: Application Security (SAST/SCA/IaC/Container, etc.) scans pass before release; DAST for exposed endpoints; container and base images scanned pre-deploy.

[ ] Hardcoded Secrets: Ensure all code generated does not have hardcoded secrets committed. (Use environment files and do not commit environment files)

[ ] Continuous Scanning: Weekly (or faster) vulnerability scans of generated source code, 3rd party dependencies, container images, and build images.

[ ] Vulnerability Remediation: Remediation within SLAs: Critical ≤ 30 days; High ≤ 30; Medium ≤ 90; Low ≤ 180; document exceptions.

[ ] Transitive Risk Control: Remove unused dependencies; review high-risk transitive packages; prefer curated, well-maintained libraries; monitor license obligations.

[ ] Secure Prompting: Prompts instruct assistants to meet security requirements (OWASP-aligned): input validation, output encoding, AuthN/AuthZ patterns, SSRF/CSRF mitigations, safe file/network I/O. Prompts are version-controlled.

[ ] Secure Prompt Templates: Embed security guidance in prompts (including input validation, AuthN/Z requirements, and safe-output handling). Additionally in AI-Assisted instruction files embed generic security guidance.

[ ] Security-Critical Code: Additional expert review mandatory for auth, crypto, deserialization, sandboxing, and secrets handling; no custom/homegrown crypto.

[ ] AI-Generated Code Review: Implement specific code review requirements for AI-generated code; reviewers verify security patterns and understand suggested code before approval.

[ ] Testing: Unit, integration, and security tests (including negative tests) required; fuzzing or property-based tests for parsers and input handlers; coverage thresholds defined.

### Repository & Build System Security
[ ] Repository Hygiene: Protected branches, required reviews, signed commits, mandatory status checks, immutable history on main; secure webhooks and app integrations.

### Use of AI Assistants (IDE, CLI, Chat)
[ ] Context Controls: Exclude secrets, API keys, and sensitive code paths from assistant context; use organization policy to mask/redact before sending.

[ ] Output Labeling & Traceability: Label AI-generated code fragments in PRs; retain prompt/context that materially influenced changes for audit and IP.

[ ] Safety Checks: Block code suggestions that introduce unsafe crypto, insecure random, insecure deserialization, command injection, path traversal, or disabling security checks.

### Testing & Adversarial Evaluation
**Implementation Priority: P2 (Month 3-6)**

[ ] Vulnerability Pattern Detection: Deploy SAST tools configured to detect common AI-suggested vulnerability patterns (SQL injection, XSS, insecure deserialization, path traversal).

[ ] Security Test Coverage: Require security tests (including negative/abuse cases) for all AI-generated code; establish minimum coverage thresholds.

[ ] AI Code Quality Baseline: Establish quality metrics for AI-generated code; track defect rates, security findings, and technical debt introduced by AI suggestions.

### Observability, Logging & Monitoring
[ ] Logging: Record prompts, assistant outputs, file diffs, security tool results, and merge decisions in tamper-evident storage; redact secrets/PII in logs.

[ ] Behavioral Baselining: Monitor anomaly signals (sudden dependency spikes, unfamiliar registries, unusual CI egress, mass code generation).

[ ] Cost/Usage Controls: Quotas and alerts for assistant API usage to detect abuse (DoW/DoS scenarios).

### Incident Response & Red Teaming
[ ] SOC Reporting: Report supply-chain compromise, data leak, or code injection within 24 hours; conduct RCA and track corrective actions.

[ ] Red Team Exercises: Conduct quarterly exercises attempting to inject malicious code via AI assistants; test detection and response capabilities.

---

## Agents/Agentic Systems
**Implementation Priority: P1 (Month 1-3) for foundational controls, P2 (Month 3-6) for advanced**

### Governance, Inventory & Risk
[ ] Inventory: Agentic systems are registered in the AI inventory with owner(s), environment(s), use cases, models, frameworks, and data sources documented.

[ ] Agent Registry: Each agent has a record of identity, purpose/scope, allowed tools/APIs, data access, environments, and risk tier.

[ ] Risk Assessment: Threat model and risk analysis completed (prompt/indirect-injection, tool abuse, lateral movement, data exfiltration, model/memory poisoning, cost abuse/DoW, collusion). Privacy Impact Assessment (PIA/DPIA) performed where applicable.

[ ] Risk Tiering & Approvals: High-risk actions (financial transfers, admin config, PII handling, prod changes) require elevated controls and pre-approval by Security/Privacy/Legal.

[ ] Compliance Mapping: Controls mapped to frameworks/regulations; evidence retained for audit.

[ ] Agent Dependency Mapping: Document agent-to-agent communication patterns and dependencies; identify potential cascade failure scenarios.

### Data Security & Privacy (Prompts, Memory, Tool I/O)
[ ] Encryption: TLS 1.2+ (Prefer TLS 1.3) for all agent/tool traffic; AES-256 (or equivalent) for data at rest (logs, memories, settings).

[ ] Minimization & Purpose Limitation: Only the minimum necessary context shared; sensitive files and secrets excluded from prompts/plans/memories.

[ ] Sensitive Data Controls: Confidential/regulatory data not sent to public/third-party services without explicit authorization, DPA/SCCs, and approved redaction/DLP.

[ ] Data Provenance & Quality: Data sources are documented (provenance, licenses, quality checks); untrusted inputs validated before use by agents.

### Access Control, Authentication & Authorization
[ ] Least Privilege: Agents receive only the permissions necessary for the task (RBAC/ABAC). Separate identities for each agent and tool.

[ ] Modern Auth: All agent-to-tool and agent-to-agent interactions use OAuth 2.1/OIDC or mTLS; no shared accounts; short-lived tokens; per-request audience/claim validation.

[ ] Dynamic Policy Enforcement: Central policy decision point (PDP) evaluates context (user, risk score, data sensitivity) for every tool call.

[ ] Privileged Operations: Require JIT elevation, approval workflow, session recording, and automatic expiry.

[ ] Service Identities: Agents must have a unique non-human identity (NHI) associated with them so their actions can be monitored and captured in an audit trail. These credentials must be stored securely according to secrets management best practices.

[ ] Inter-Agent Authentication: Implement authentication between agents to verify each other's identity before accepting commands or data.

[ ] Break-Glass Access: Implement emergency access procedures for critical agent systems; require multi-party approval; audit all break-glass usage.

### Runtime Guardrails & Safety
[ ] Behavioral Baselines: Define permissible actions, tool chains, and states; enforce maximum steps, recursion depth, and timeouts.

[ ] Tool Governance: Maintain allow/deny lists; wrap tools with input/output validation, schema checks, and content sanitizers; block open-ended shell/API access unless sandboxed and explicitly approved.

[ ] Budget & Rate Controls: Quotas/rate limits to prevent DoS/Denial-of-Wallet; per-tenant and per-task budgets with alerting.

[ ] Kill Switches: Immediate isolate/disable capability for agents, tools, and orchestrators; periodically tested (at least quarterly).

[ ] Message Validation: Deploy validation for inter-agent messages to detect and block malicious payloads; sanitize all cross-agent data transfers.

### Environment & Network Security
[ ] Isolation & Sandboxing: Dedicated, non-root execution sandboxes per agent/session; read-only filesystems; CPU/memory limits; syscall profiles (seccomp/AppArmor).

[ ] Agent Isolation Boundaries: Implement containment so compromise of one agent cannot automatically spread to others; segment agent environments.

[ ] Zero Trust Networking: Private endpoints only; no public ingress; mutual TLS for service-to-service; egress restricted to an allowlist (default-deny); DNS policies to prevent data exfiltration.

[ ] Hardening: CIS Level 2 baseline for hosts and container runtimes; STIG hardening and FIPS-validated crypto where required; timely patching of images and hosts.

### Secrets & Key Management
[ ] No Secrets in Prompts/Memories: Secrets never placed in prompts, plans, memory, or logs.

[ ] Approved Managers: All secrets stored in approved secret managers; automated rotation (≤90 days); short-lived scoped tokens; hardware-backed keys (HSM/TPM) where supported.

[ ] Just-in-Time Retrieval: Agents fetch credentials via brokered workflows (not stored long-term in memory or config).

### Development Lifecycle & Supply Chain
[ ] SSDLC: Threat modeling, security requirements, code review, and verification gates are required for agent frameworks/orchestrations.

[ ] Security Scanning: SAST/SCA/IaC scans pass before merge; DAST for exposed endpoints; container and base images scanned pre-deploy.

[ ] Prompt/Tool Versioning: Version and review prompt templates, policies, tool schemas, and action graphs; change control with rollback.

[ ] Dependency Hygiene: Allowlisted registries, lock files, and license checks; remove unused/unknown packages; monitor for typosquats.

[ ] Hardcoded Secrets: Ensure all code generated does not have hardcoded secrets committed. (Use environment files and do not commit environment files)

### Testing, Evaluation & Red Teaming
**Implementation Priority: P2 (Month 3-6)**

[ ] Safety & Robustness Evaluations: Pre-prod tests for accuracy, robustness, privacy leakage, harmful output, and safe-tool usage; acceptance thresholds defined.

[ ] Staging Verifications: Verify egress restrictions, policy enforcement, and HITL trigger points in staging; no real secrets/test data only.

[ ] Regression Suite: Automated tests rerun on each change to agents, tools, prompts, or policies.

[ ] Agent Red Team Exercises: Conduct quarterly exercises simulating agent compromise, tool abuse, and cascade failures; document findings and remediation.

[ ] Adversarial Agent Testing: Test agents with adversarial inputs designed to manipulate behavior, induce tool abuse, or cause data exfiltration.

### Observability, Telemetry & Logging
[ ] Comprehensive Logging: Prompts, plans, tool calls, outputs, memory changes, inter-agent messages, safety filter decisions logged to immutable/tamper-evident storage.

[ ] SIEM Integration: Centralized collection with detections for anomalous chains (excessive steps), unusual tool combos, large egress volumes, and policy denials.

[ ] Operational Metrics: Track task success, HITL interventions, denied actions, cost/budget use, and safety filter hit rates; alert on thresholds.

[ ] Behavioral Monitoring: Deploy ML-based anomaly detection on agent behavior; alert on deviations from established baselines.

### Incident Response & Recovery
[ ] AI-Specific Runbooks: For agent misuse, data exfiltration, runaway behaviors, and cost spikes; include steps to freeze/contain, snapshot context, and revoke credentials.

[ ] 24-Hour SOC Notification: All AI incidents reported within 24 hours; RCA documented; compensating controls tracked to closure.

[ ] Quarantine & Rollback: Ability to quarantine agents/tools and roll back to safe configurations/models.

[ ] Automated Suspension: Configure automated agent suspension when high-severity threats are detected; define clear trigger conditions.

### Third-Party & Vendor Controls
[ ] Security Assurance: Vendors provide SOC 2 Type II or ISO 27001; map to CSA AI Controls Matrix where applicable; disclose model/tool lineage and safety practices.

[ ] Contractual Safeguards: Breach/incident notification (≤ 24h), data handling and deletion terms, crypto requirements (FIPS where mandated), uptime/SLOs, and right to audit.

[ ] Block Shadow/Unapproved Tools: Discovery and blocking of unapproved agent tools and orchestrators; maintain approved/denied lists.

---

## Model Context Protocol (MCP)
**Implementation Priority: P1 (Month 1-3)**

### Governance, Inventory & Risk
[ ] Inventory: MCP servers, clients, tools, and orchestrators are registered in the AI inventory with owner(s), environment(s), purpose/use cases, data categories, and integrations documented.

[ ] Agent/Tool Registry: Each MCP server and exposed tool/skill has a record (identity, owner, purpose/scope, APIs, data access, tenant, risk tier).

[ ] Security Risk Assessment: Use case is reviewed and approved by security team; threat model covers MCP-specific risks (prompt injection via tools, tool abuse, context poisoning, lateral movement).

[ ] MCP Approval Workflow: Implement vetting process where security reviews each MCP server before deployment; risk-based classification with proportional controls.

### Architecture, Isolation & Network Security
[ ] No Public Ingress: MCP services are not internet-exposed; access only via private endpoints, VPN, or service mesh.

[ ] Zero Trust: Mutual TLS (mTLS) for service-to-service; default-deny egress with explicit allowlist; DNS egress controls and domain pinning where feasible.

[ ] Environment Separation: Dedicated environments for dev/test/prod; prohibit cross-environment connections; sanitize test data.

[ ] Network Segmentation: MCPs only accessible from authorized agent environments; implement micro-segmentation where feasible.

### Authentication, Authorization & Identity
[ ] Modern Authentication: OAuth 2.1 with PKCE (or mTLS) required for clients; per-request token validation (audience, issuer, expiration, nonce/claims); no token passthrough; short-lived tokens (≤ 60 minutes); no shared accounts.

[ ] Fine-Grained Authorization: RBAC/ABAC with policy decision point (PDP); per-tool scopes; per-tenant and per-user/task context isolation.

[ ] Privileged Operations: Just-In-Time elevation, approval workflows, session recording, and auto-expiry for administrative tasks.

[ ] Service Identities: Unique non-human identities for MCP servers/tools; key/cert lifecycle managed centrally.

[ ] MFA for Sensitive MCPs: Require multi-factor authentication for MCPs accessing sensitive data or executing privileged operations.

### Data Security & Privacy
[ ] Encryption: TLS 1.2+ (Prefer TLS 1.3) in transit; AES-256 (or equivalent) at rest for logs, memories, configs, caches.

[ ] Minimization: Limit prompts/context to minimum necessary; redact PII/PHI/secrets before transmission; disable unnecessary tool exposure.

[ ] No Local Persistence: No local persistence of prompts, outputs, or sensitive context beyond transient buffers; define TTLs and purging.

### Secrets & Key Management
[ ] Approved Secrets Management Solutions: All secrets (API keys, credentials, tokens) stored only in approved secret managers; no secrets hard coded in images, environment files, prompts, or memory.

[ ] Rotation & Hardware Protection: Rotate secrets/keys at least every 90 days (or on compromise); use HSM/TPM-backed keys where available; enforce short-lived, scoped credentials.

### Workload & Platform Hardening
[ ] Host/Container Baselines: CIS Level 2 on host OS and container runtime; enable FIPS-validated cryptography and STIG hardening where required; run MCP services as non-root. This applies when running an MCP server on shared infrastructure (i.e. an actual server).

[ ] Execution Sandboxing: Isolate MCP servers from the underlying host (e.g. run in Docker container) to limit MCP server's ability to take actions outside of its intended purpose.

[ ] EDR/Workload Protection: Endpoint and workload protection enabled on hosts and containers; telemetry integrated with SOC. (Microsoft Defender and Prisma Cloud Defender agents deployed to hosts/container clusters).

### Development Lifecycle, Supply Chain & Integrity
[ ] SSDLC: Threat modeling, secure code requirements outlined in the Secure SDLC (Secure TPL), code review, and verification gates for MCP code and infrastructure. (Where applicable)

[ ] Security Scanning: SAST/SCA/IaC scans for MCP code and dependencies; container image scanning pre-deployment.

[ ] Dependency Management: Track MCP dependencies; use approved registries; monitor for vulnerabilities and license compliance.

### Vulnerability Management & Patching
[ ] Routine Scans: Daily scans for hosts, containers, and images; SLAs: Critical ≤ 30 days; High ≤ 30; Medium ≤ 90; Low ≤ 180.

[ ] Patch Cadence: Apply security patches promptly; rebuild images regularly; retire vulnerable components quickly.

### Testing & Validation
[ ] AuthN/AuthZ Test Cases: Automated tests for authentication, scope/policy enforcement, and least-privilege tool access; negative tests for token replay/audience misuse.

[ ] Egress & Isolation Validation: Verify egress allowlists and tenant/task isolation in staging and prod; simulate data exfiltration and lateral movement.

[ ] MCP Security Testing: Test MCP servers for prompt injection via tool parameters, unauthorized tool access, and context manipulation.

### Gateway & Tool/Skill Governance
[ ] Gateway Allowlist: Security maintains and enforces an allowlist of MCP servers at gateways; block unregistered endpoints by default.

[ ] Tool/Skill Registries: Lock and allowlist tool/skill registries; version prompts and tool schemas; enable output filters/sanitizers for risky responses; enforce tenant/task isolation.

[ ] Least Privilege Tooling: Each MCP server exposes only minimum required tools; tools have scoped credentials; per-call authorization.

### Observability & Monitoring
[ ] MCP Traffic Monitoring: Monitor all MCP traffic in real-time; alert on unauthorized access attempts, permission escalation, and anomalous tool usage patterns.

[ ] Rate Limiting: Configure rate limits per agent, per user, per MCP server to prevent abuse; automated quarantine for suspicious MCPs.

[ ] Usage Analytics: Track which agents use which MCPs, tool invocation patterns, data access patterns, and costs.

### Incident Response & Security Operations
[ ] Logging to SIEM: Send authentication, authorization, configuration, tool/skill registry changes, and runtime event telemetry to central SIEM; retention ≥ 1 year (or per policy). This applies for MCP servers hosted on shared infrastructure (i.e. servers).

[ ] Runbooks: IR runbooks for MCP compromise, token leakage, unauthorized tool activation, exfiltration attempts, and cost spikes.

[ ] SOC Integration: Alerts on policy violations, anomalous tool chains, excessive steps, or unexpected destinations; 24-hour SOC notification and RCA for incidents.

[ ] Kill Switch: Ability to immediately isolate/disable MCP services and revoke credentials; periodic drills to validate effectiveness (at least quarterly).

---

## Compliance & Regulatory Management
**Implementation Priority: P2 (Month 3-6)**

### Regulatory Mapping & Documentation
[ ] Regulatory Inventory: Identify all applicable regulations for AI systems (EU AI Act, GDPR, CCPA, HIPAA, sector-specific regulations); map to organizational AI inventory.

[ ] System-to-Regulation Mapping: Map each AI system to applicable regulatory requirements; document compliance status and gaps for each requirement.

[ ] Risk Classification: Classify AI systems per regulatory frameworks (e.g., EU AI Act risk tiers: unacceptable, high, limited, minimal); apply controls accordingly.

[ ] Legal Review: Engage Legal/Privacy teams in approval workflow for high-risk AI systems; document legal sign-off.

### Automated Compliance Controls
[ ] Automated Evidence Collection: Implement automated collection of compliance evidence (logs, approvals, test results, security scans, training records) for auditors.

[ ] Compliance Dashboard: Deploy real-time compliance posture dashboard showing coverage, gaps, and risk status; accessible to stakeholders and auditors.

[ ] Policy Automation: Automate enforcement of regulatory requirements where possible (data minimization, retention limits, access controls, consent management).

[ ] Continuous Compliance Monitoring: Deploy continuous monitoring for compliance drift; alert when systems fall out of compliance.

### Audit & Assessment
[ ] Compliance Review Cadence: Conduct quarterly compliance reviews; assess new regulations and update controls accordingly; document changes and rationale.

[ ] Third-Party Audits: Engage external auditors for annual assessment of AI security and compliance posture; track findings to closure.

[ ] Documentation & Records: Maintain comprehensive documentation of AI systems, decisions, risk assessments, and approvals; retention per regulatory requirements.

### Data Subject Rights
[ ] Subject Rights Implementation: Implement mechanisms for data subject access, rectification, deletion, and objection requests related to AI processing.

[ ] Consent Management: Deploy consent management for AI processing where required; track consent status and honor withdrawals.

[ ] Impact Assessments: Conduct Data Protection Impact Assessments (DPIA) and Algorithmic Impact Assessments (AIA) for high-risk systems.

---

## Security Operations & Incident Response
**Implementation Priority: P1 (Month 1-3) for foundational capabilities, P2 (Month 3-6) for maturity**

### 24/7 Operations
[ ] AI Security Operations Coverage: Establish 24/7 security operations coverage for AI systems; integrate AI monitoring into SOC workflows.

[ ] AI-Specific Detection Rules: Deploy SIEM detection rules for AI-specific threats (prompt injection, tool abuse, data exfiltration, agent compromise, cost spikes).

[ ] Escalation Procedures: Define clear escalation paths for AI incidents; identify decision-makers for different incident types.

### Incident Response Planning
[ ] AI Incident Playbooks: Develop incident response playbooks for each AI technology (LLM data leak, agent compromise, MCP abuse, AI-assisted code injection).

[ ] 24-Hour Notification: Establish process for reporting AI incidents to SOC within 24 hours; define incident severity criteria.

[ ] Kill Switch Procedures: Document and test procedures for emergency shutdown of AI systems; define authorization requirements.

[ ] Credential Revocation: Establish rapid credential revocation procedures for compromised AI systems; include API keys, tokens, and service accounts.

### Incident Response Execution
[ ] Containment Procedures: Define containment strategies for AI incidents (isolate agents, revoke access, snapshot state, quarantine models).

[ ] Forensics & Analysis: Develop forensic procedures for AI incidents; preserve prompts, responses, tool calls, and decision logs.

[ ] Root Cause Analysis: Conduct RCA for all AI incidents; document findings, contributing factors, and lessons learned.

[ ] Corrective Actions: Track corrective actions to closure; re-test controls after remediation; update runbooks based on lessons learned.

### Testing & Exercises
[ ] Tabletop Exercises: Conduct quarterly tabletop exercises simulating AI incidents; test communication, decision-making, and coordination.

[ ] Kill Switch Testing: Test kill switch procedures at least quarterly; verify effectiveness and identify gaps.

[ ] Incident Response Drills: Conduct annual full-scale incident response drills including technical containment, communication, and recovery.

---

## Continuous Improvement & Metrics
**Implementation Priority: P3 (Month 6+)**

### Security Metrics & KPIs
[ ] Control Coverage Metrics: Track percentage of AI systems with required controls implemented; report coverage by technology and risk tier.

[ ] Incident Metrics: Monitor AI incident frequency, severity, mean time to detect (MTTD), mean time to respond (MTTR), and recurrence rates.

[ ] Testing Metrics: Track security test coverage, vulnerability detection rates, and remediation velocity for AI systems.

[ ] Compliance Metrics: Monitor compliance posture, gap closure rates, and audit findings trends.

### Program Maturity
[ ] Maturity Assessment: Conduct annual AI security maturity assessment; identify gaps and prioritize improvements.

[ ] Benchmark Analysis: Compare AI security posture against industry benchmarks and peer organizations.

[ ] Capability Roadmap: Maintain 12-18 month roadmap for AI security capability development; align with business AI adoption plans.

### Threat Intelligence
[ ] AI Threat Intelligence: Subscribe to AI security threat intelligence feeds; monitor emerging threats and attack techniques.

[ ] Vulnerability Tracking: Track vulnerabilities in AI frameworks, models, and tooling; prioritize patching based on exploitability.

[ ] Attack Pattern Library: Maintain library of known AI attack patterns (prompt injections, adversarial inputs, tool abuse techniques).

### Continuous Learning
[ ] Security Training: Provide AI security training for developers, security teams, and business stakeholders; update training as threats evolve.

[ ] Lessons Learned: Conduct lessons learned sessions after incidents and red team exercises; update controls and playbooks.

[ ] Community Engagement: Participate in AI security communities, working groups, and information sharing forums.

---

## Implementation Priority Summary

**P0 (Weeks 1-4) - Foundational Discovery:**
- AI-SPM deployment and shadow AI discovery
- Basic inventory and registration
- AI Security Working Group establishment
- Essential encryption and authentication controls

**P1 (Months 1-3) - Governance & Architecture:**
- AI Gateway deployment and configuration
- MCP governance framework
- Agent behavioral baselines
- Model approval workflows
- 24/7 security operations coverage

**P2 (Months 3-6) - Advanced Protection:**
- Comprehensive testing programs (injection, bias, adversarial, red team)
- Compliance automation and dashboards
- Behavioral monitoring and anomaly detection
- Advanced incident response capabilities

**P3 (Months 6+) - Optimization & Maturity:**
- Continuous improvement programs
- Maturity assessments and benchmarking
- Advanced metrics and analytics
- Community engagement and threat intelligence

---

*This checklist should be reviewed quarterly and updated based on evolving threats, new technologies, regulatory changes, and lessons learned from incidents and exercises.*