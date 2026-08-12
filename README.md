# Notes

Long-form write-ups on security, AI, and career growth, published here so they're easy to link, fork, and reuse.

[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg)](LICENSE)

## About

This repo is a public collection of the notes and guides I write for my own reference and share with the community. Most of it comes out of day-to-day work running AppSec/Product Security teams, plus some general AI-usage and career material that doesn't fit anywhere else. Everything here is free to reuse under the license below: take what's useful, adapt it, pass it on.

## Contents

### Security
- [AI Security Checklist](Security/AI%20Security%20Checklist.md): P0-to-P3 prioritized controls for standing up an AI security program, from governance to LLM/RAG/agent-specific safeguards.
- [Security Tooling Landscape](Security/Security%20Tooling%20Landscape.md): a categorized map of the InfoSec vendor/tool landscape (GRC, SIEM, SOAR, and more).
- **Threat Modeling**: system design + threat model write-ups, each covering architecture, trust boundaries, DFDs, and a threat table:
  - [Password Manager](Security/Threat%20Modeling/Password%20Manager.md)
  - [P2P Payment App](Security/Threat%20Modeling/P2P%20Payment%20App.md)
  - [Cloud File Storage Service](Security/Threat%20Modeling/Cloud%20File%20Storage%20Service.md)
  - [Multi-Tenant SaaS Isolation Model](Security/Threat%20Modeling/Multi-Tenant%20SaaS%20Isolation%20Model.md)
  - [Three-Tier Architecture](Security/Threat%20Modeling/Three-Tier%20Architecture.md)
  - [GitLab-Style SaaS (Multi-Tier)](Security/Threat%20Modeling/GitLab-Style%20SaaS%20%28Multi-Tier%29.md)
  - [Threat Modeling a Local AI Box (Mac Studio Edition)](Security/Threat%20Modeling/Threat%20Modeling%20a%20Local%20AI%20Box%20%28Mac%20Studio%20Edition%29.md)
  - [Building a Password Manager the Way a Real Team Would](Security/Threat%20Modeling/Building%20a%20Password%20Manager%20the%20Way%20a%20Real%20Team%20Would.md)
  - [Threat Modeling: The Complete Guide, From Zero to Expert](Security/Threat%20Modeling/Threat%20Modeling%20-%20The%20Complete%20Guide%2C%20From%20Zero%20to%20Expert.md)

### AI
- [The Complete Claude Workflow Guide](AI/The%20Complete%20Claude%20Workflow%20Guide.md): subscriptions, tools, agents, MCP, and automation, covering the Claude ecosystem from beginner to expert.
- [Token Efficiency Guide](AI/Token%20Efficiency%20Guide.md): a practitioner's guide to using Claude efficiently, covering what drives token/cost and how to cut waste.

### Terms & Definitions
- [AI Terminology Reference for Software Engineering and Security](Terms%20%26%20Definitions/AI%20Terminology%20Reference%20for%20Software%20Engineering%20and%20Security.md): AI/ML terms for engineers and security folks, each with what problem it solves and when it's the wrong choice.
- [Cloud & App Architecture Glossary](Terms%20%26%20Definitions/Cloud%20%26%20App%20Architecture%20Glossary%20-%20What%20It%20Is%2C%20Why%20It%20Exists%2C%20and%20When%20to%20Skip%20It.md): cloud and app architecture concepts (microservices, serverless, and more), what they are, why they exist, and when to skip them.
- [Product Security Definitions](Terms%20%26%20Definitions/Product%20Security%20Definitions.md): core product security vocabulary, scoped and framed for engineers new to the discipline.
- [Software Terms to Learn](Terms%20%26%20Definitions/Software%20Terms%20to%20learn.md): plain-language explainers for common software engineering terms, with everyday analogies.

### Career
- [AppSec-ProdSec-DevSecOps Learning Path](Career/AppSec-ProdSec-DevSecOps%20Learning%20Path.md): a practical roadmap for breaking into AppSec/ProdSec/DevSecOps, from someone who hires for it.
- [Product Security Competency Pathway](Career/Product%20Security%20Competency%20Pathway.md): what's expected at each career level, laid out dimension by dimension.
- [Managing Up](Career/Managing%20Up.md): what managing up actually means and why visibility matters more than people think.

### Misc
- [Spotting AI-Generated Text](Misc/Spotting%20AI-Generated%20Text.md): patterns and tells for identifying AI-written content, kept current as models evolve.
- [LinkedIn Algorithm 2026 Guide](Misc/LinkedIn%20Algorithm%202026%20Guide.md): a deep dive into how the LinkedIn algorithm works and what changed going into 2026.

## License

Everything in this repo is licensed under [Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)](LICENSE). In short: you're free to share and adapt this material for any purpose, including commercially, as long as you give appropriate credit and license any derivatives under the same terms.
