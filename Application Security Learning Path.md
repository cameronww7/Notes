# Application Security Learning Path

So you want to break into AppSec, DevSecOps, or Product Security? Here's a practical roadmap to get you there.

## How to Learn Effectively

Reading or listening is not enough. You need to actively engage with what you're learning. As you go through any material:

**Ask critical questions:**

- Why is this important? Why does it matter?
- What is the small picture (the specific detail or technique)?
- What is the bigger picture (how does this fit into the overall security landscape)?
- How would an attacker exploit this? What could go wrong?

**Apply actively:**

- Take notes in your own words, not verbatim
- Immediately test concepts in a lab or personal project
- Explain what you learned to someone else (or write a blog post)
- Connect new concepts to real-world breaches or vulnerabilities you've heard about
- Review your notes within 24 hours, then again after a week

Learning security is about building mental models, not memorizing facts. The goal is to develop intuition for how systems break and how to build them securely.

- Active Learning: 5 Reasons You're Doing Active Learning WRONG: https://www.youtube.com/watch?v=0A5Ji-QdFvg

## Foundational Knowledge

### Start Here, Core AppSec Concepts

- Read "Alice and Bob Learn Application Security" (As you complete chapters, watch Tanya's chapter summaries at the following YouTube playlist: https://youtube.com/playlist?list=PLI9RITMnVbygrVQaGvpojIzgHTpkRrIn8&si=xrUXxdTsikHFA0vF) and "Alice and Bob Learn Secure Coding"
- Watch YouTube videos from Jim Manico (particularly "The History of AppSec") and Tanya Janca (any of her talks on YouTube)

#### Essential Video Content

**Jim Manico Talks:**

- Keynote: Jim Manico - The Abridged History of Application Security: https://www.youtube.com/watch?v=QbEjNqKRhCg
- Jim Manico - Enhancing ReactJS Application Security: XSS Scripting and Client-Side Defense: https://www.youtube.com/watch?v=gOODVasKdsk&t=6s
- From the OWASP Top Ten(s) to the OWASP ASVS - Jim Manico: https://www.youtube.com/watch?v=CCIO3GOaFe0
- Jim Manico - Securing AI-Generated Code in the Development Lifecycle (React Focus): https://youtu.be/O2hUJ3LdulE?si=KOssISXZHosoh6iW

**Tanya Janca Talks:**

- Lightning Talk: Purple is the new black: Modern Approaches to Application Security - Tanya Janca: https://www.youtube.com/watch?v=iy1xcmsIx14
- Global AppSec Dublin: Shifting Security Everywhere - Tanya Janca: https://www.youtube.com/watch?v=ei3Subi59Kg

**Additional Essential Talks:**

- OWASP Top 10 2025 Announcement - Updated view of modern applications' most common security risks: https://youtu.be/yGOXewm3DsA?si=FPFDLho0-q_ihFUG
- Keynote by Daniel Miessler: The Future of AppSec Is Continuous Context: https://youtu.be/C4L5hPYl1Os?si=LYABTnE5tbCsDq08
- Keynote by Adam Shostack: Stop Trying to "Manage Risk": https://youtu.be/THSfJIlPGPk?si=yg-ufzX_Q8vBhylC
- James Wickett - Out of Control: Promise Theory and the Future of Security Agents: https://www.youtube.com/watch?v=0TyqE5ARJw4
- Dustin Lehr - The Cybersecurity-Minded Future, And How Security Champions Get Us There: https://www.youtube.com/watch?v=aUwe0KsUo7I&t=2350s
- Mauve Hed & Francesco Cipollone - Navigating Challenges of Risk-Based Vulnerability Management in a Cloud World: https://www.youtube.com/watch?v=bDtJA551vpI
- Jenn Gile - How to sell your soul, err, your security program: https://www.youtube.com/watch?v=SM36aCbxj6c&t
- Rafal Los - Why Your Security Metrics Suck, And How to Fix This: https://www.youtube.com/watch?v=aLibY2Azneo
- Chris Hughes - Securing AI Where it Acts: Why Agents Now Define AI Risk | AI Summit Q1 2026 - https://www.youtube.com/watch?v=A6eIooEm7Cs

### Learn the Standards, OWASP Frameworks

These aren't just reading material. They're the standards you'll reference daily in this field.

- OWASP Top 10 2025 (Most Critical Web Application Security Risks): https://owasp.org/Top10/2025/
- OWASP Top 10 for LLMs 2025: https://owasp.org/www-project-top-10-for-large-language-model-applications/assets/PDF/OWASP-Top-10-for-LLMs-v2025.pdf
- OWASP API Security Top 10: https://owasp.org/API-Security/
- OWASP Top 10 CI/CD Security Risks: https://owasp.org/www-project-top-10-ci-cd-security-risks/
- OWASP ASVS (Application Security Verification Standard): https://owasp.org/www-project-application-security-verification-standard/
- OWASP SPVS (Secure Pipeline Verification Standard): https://owasp.org/www-project-spvs/

### Understand the Cloud

- Take AWS Certified Cloud Practitioner (CCP) or general cloud understanding courses. 
    - Free course: https://youtu.be/NhDYbskXRgc?si=pHmsypJT3WfrzCxt
- Or for Azure (Microsoft Certified: Azure Fundamentals (Exam AZ-900))
    - Free course: https://www.youtube.com/watch?v=5abffC-K40c
- Follow up with cloud security-specific training (Youtube, Blogs, just search Cloud Security)
- Modern AppSec lives in the cloud. You need to understand how cloud environments work. 


## Technical Hands-On Skills

### Learn to Code (Yes, Really)

You can't secure what you don't understand. There are lots of free courses on YouTube from different channels such as FreeCodeCamp, and Udemy offers cheap courses if you want more structured learning. Learning how to code, understand code, and recognize what languages are used in different parts of software is absolutely key. Take full stack programming courses and learn Python or Go for automation/API-to-API scripts. You're going to talk to developers and work with developers every single day. You need to speak their language or you won't be effective.

### AI & Emerging Technologies

AI security is exploding right now. Get ahead of it:

- Start with TCM Security's AI Fundamentals course (free on YouTube and TCM Academy)
- Go to YouTube and search "AI Security" and you'll find tons of videos. Start digesting and learning, taking notes as you go.
- Dive into Model Context Protocol (MCP), MCP Security, AI Agents, and related topics (Just Google or search on LinkedIn, you'll find a ton of stuff)
- Reference the OWASP Top 10 for LLMs when working with AI/ML applications

**Key AI Security Terms to Research:** Prompt injection, Model poisoning, Jailbreaking, Hallucination, Context window attacks, Model extraction, Data leakage, Shadow AI, AI supply chain, Model Context Protocol (MCP), AI Agents, AI-assisted coding

### Build Your Own Secure Pipeline

This is your hands-on laboratory. Set up your own secure pipeline using open source tools. Use GitHub runners or Jenkins locally. Configure it to run against vulnerable code and scan using free open source scanners:

- **SAST**: Semgrep (free), Snyk (free tier), OpenGrep (free)
- **DAST**: ZAP (free)
- **SCA (Software Composition Analysis)**: OWASP Dependency-Check, Snyk (has free tier)
- **Container Scanning**: Trivy (free)
- **SBOM Generation**: Syft (https://github.com/anchore/syft)

#### Example Pipeline Build Guides

- Building a DevSecOps Pipeline with Open Source Tools: https://medium.com/cloud-native-daily/building-a-devsecops-pipeline-with-open-source-tools-ad4fd0e13515
- FPL Insights Dashboard - Real-World DevSecOps Lab Project: https://medium.com/@bll_78288/fpl-insights-dashboard-a-real-world-devops-devsecops-lab-project-85e7065ea7c9
- Complete DevSecOps Pipeline Tutorial (Video): https://m.youtube.com/watch?v=mZoOnWjv_QM

### Infrastructure as Code & Container Security

- Infrastructure as Code in a DevSecOps World: https://snyk.io/learn/infrastructure-as-code-iac/
- Infrastructure as Code Security - OWASP Cheat Sheet Series: https://cheatsheetseries.owasp.org/cheatsheets/Infrastructure_as_Code_Security_Cheat_Sheet.html
- Everything You Need to Know to Get Started With Container Security: https://snyk.io/learn/container-security/
- Building a Strong Container Security Foundation: https://snyk.io/learn/container-security/container-scanning/

The crucial next step is understanding and triaging those results. Learn how to interpret findings, prioritize based on risk and exploitability, and create actionable remediation plans. Being able to run a scanner is table stakes. Knowing how to triage, prioritize, and communicate findings to developers is what gets you hired.

Alternatively, build a similar home lab environment. Document what you build. This becomes portfolio material.

### Learn the Attacker Mindset

Learning the basics of hacking can really contextualize vulnerabilities and is critical for replicating them. Start with the free TCM Security Practical Ethical Hacking course on YouTube: https://youtube.com/playlist?list=PLLKT__MCUeixqHJ1TRqrHsEd6_EdEvo47&si=xxMBSn3Eae4BJH6C

Then consider taking the Certified Penetration Testing Specialist (CPTS) from HackTheBox or the TCM PJPT or PNPT. Understanding how attackers think and operate makes you exponentially better at defense.

### Certifications Worth Considering

While certifications aren't everything, some can provide structured learning paths and credibility:

**Foundation Certifications:**

- CompTIA Security+ (debatable, but gives you a well-rounded perspective)
- AWS Certified Cloud Practitioner (foundations of what AWS and clouds do in general)
- AWS Certified Solutions Architect - Associate (gets your brain thinking on how we build things in the cloud)

**Penetration Testing Certifications** (these teach you how to break things, which helps you learn to defend them):

In order of recommended progression:

- eLearnSecurity Junior Penetration Tester
- CPTS (HTB Certified Penetration Testing Specialist): https://academy.hackthebox.com/preview/certifications/htb-certified-penetration-testing-specialist
- Practical Junior Penetration Tester (PJPT)
- Practical Network Penetration Tester (PNPT)
- Web Pentest Associate (PWPA)

## Core Skills That Separate You From the Pack

### Threat Modeling

This is fundamental to AppSec. It's not optional. Threat modeling helps you identify security risks during the design phase, before code is even written. Focus on methodologies like STRIDE (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege) and PASTA. This skill separates good AppSec engineers from great ones. Hiring managers look for this.

## Local Networking

Getting into security isn't just about learning technical skills. It's also about building relationships with other practitioners in your area. Look up local cybersecurity meetups on Meetup.com, search for security events on Eventbrite, and find conferences near you. This will take some Googling, but search for things like "OWASP chapter [your city]", "cybersecurity meetups [your city]", and "security conferences [your state/region]". These events are invaluable for networking, learning what skills are in demand locally, and often hearing about job opportunities before they're posted. Show up, introduce yourself, and don't be afraid to tell people you're looking to break into the field. The security community is generally welcoming to newcomers who show genuine interest.

## Additional Resources

### Curated Lists & Collections

- Awesome AppSec: A comprehensive curated list of resources, tools, and learning materials: https://github.com/paragonie/awesome-appsec

### Newsletters & Blogs

Stay current with security news and practical advice:

- TL;DR sec (Newsletter & Blog): https://tldrsec.com/subscribe
- Boring AppSec (Newsletter & Blog): https://boringappsec.substack.com/subscribe
- Resilient Cyber: https://www.resilientcyber.io/
- Cybersecurity Pulse: https://www.cybersecuritypulse.net/

### YouTube Channels & Video Content

- LASCON: Real-world AppSec conference talks and presentations from practitioners: https://youtube.com/@lascon512
- OWASP DevSlop (Tanya Janca): Practical AppSec education, secure coding, and developer-focused security content: https://youtube.com/@DevSlop
- DevSecOps Course - Web Vulnerabilities, DevSecOps Tools, and Container Security: https://www.youtube.com/watch?v=F5KJVuii0Yw

### Interesting YouTube content creators with security education and training content

- https://youtube.com/@vulnerableu?si=I1OOhNfuZbGBwbDG
- https://youtu.be/lqZo4waMB3c?si=wdbwYNi14CSnP-6T

### Recommended Books

- "The Code Book" by Simon Singh: Understanding cryptography and its history
- "The DevOps Handbook": Essential reading for understanding how DevOps and security integrate
- "The DevSecOps Playbook: Deliver Continuous Security at Speed" by Sean D. Mack: Practical guide to implementing DevSecOps
- "Agile Application Security: Enabling Security in a Continuous Delivery Pipeline" by Laura Bell: Security in agile and CI/CD environments
- "Practical Vulnerability Management: A Strategic Approach to Managing Cyber Risk" by Andrew Magnusson: Strategic approach to vulnerability management

## Final Thought

Breaking into AppSec isn't about collecting certifications. It's about building practical skills, understanding how developers work, and being able to secure software at every stage of the SDLC. Build things, break things, document what you learn, and contribute to the community. That's what gets you in the door.