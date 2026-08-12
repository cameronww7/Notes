# The Complete Claude Workflow Guide

### From beginner to expert: subscription, tools, agents, MCP, automation, and the wider ecosystem

**Written:** July 26, 2026 **Verified against:** Anthropic's live docs at `code.claude.com/docs`, `claude.com/docs`, and `support.claude.com` on the date above.

---

## How to read this

Think of this as a course with 14 units (Parts 0 through 13). Each unit stands on its own, so you can jump around.

|Part|What it covers|Read it if you want to...|
|:--|:--|:--|
|0|The big map|See how every piece fits together|
|1|Vocabulary|Learn the words people throw around|
|2|What Max actually buys|Know your plan's real limits and gaps|
|3|Every surface (tool)|Pick the right front door for a task|
|4|The extension layer|Get to expert level: skills, agents, hooks, MCP, plugins|
|5|The `.claude` directory|Know where every config file lives|
|6|Workflows|Copy working recipes|
|7|How developers use it|Learn the real patterns and the traps|
|8|How security people use it|Apply this to AppSec, pipelines, IR, detection|
|9|Non-native tooling|Understand what people use instead, and why|
|10|Governance|Control this across an org|
|11|Cost and context economics|Stop burning your weekly limit|
|12|A 30/60/90 learning path|Actually build the muscle|
|13|Troubleshooting|Fix the common breakages|

Every diagram has a written breakdown underneath it. If you skim, read the diagrams and their breakdowns.

**One habit to build first.** Claude Code can look up its own documentation. When you want ground truth instead of a blog post, open a session and ask:

```
Fetch https://code.claude.com/docs/en/claude_code_docs_map.md and tell me
which page covers <thing>, then fetch that page and answer my question.
```

The docs move fast. That trick keeps you current when this guide goes stale.

---

# Part 0: The big map

Everything Anthropic ships is built on the same core idea: a model in a loop with tools. What changes between products is the interface, where the loop runs, and what it is allowed to touch.

```
                     ┌──────────────────────────────────────────┐
                     │        YOUR CLAUDE SUBSCRIPTION          │
                     │   (Free / Pro / Max 5x / Max 20x /       │
                     │       Team / Enterprise)                 │
                     │   = one identity, one usage pool         │
                     └───────────────────┬──────────────────────┘
                                         │
        ┌────────────────────────────────┼────────────────────────────────┐
        │                                │                                │
   ┌────▼─────┐                    ┌─────▼──────┐                  ┌─────▼──────┐
   │  CHAT    │                    │   COWORK   │                  │ CLAUDE CODE│
   │ surfaces │                    │ (Desktop)  │                  │  surfaces  │
   ├──────────┤                    ├────────────┤                  ├────────────┤
   │ web      │                    │ agentic    │                  │ CLI        │
   │ desktop  │                    │ workspace  │                  │ Desktop app│
   │ mobile   │                    │ for non-   │                  │ VS Code    │
   │ Chrome   │                    │ coders     │                  │ JetBrains  │
   │ Excel    │                    │ files,     │                  │ web/cloud  │
   │ PowerPnt │                    │ docs,      │                  │ Slack      │
   │ Design   │                    │ research,  │                  │ mobile     │
   │ Tag(Slack)                    │ decks      │                  │ Agent SDK  │
   └────┬─────┘                    └─────┬──────┘                  └─────┬──────┘
        │                                │                               │
        └────────────────────────────────┼───────────────────────────────┘
                                         │
                     ┌───────────────────▼──────────────────────┐
                     │          THE EXTENSION LAYER             │
                     │  (this is where expertise actually lives)│
                     ├──────────────────────────────────────────┤
                     │  CLAUDE.md + rules   → always-on context │
                     │  Skills (SKILL.md)   → on-demand know-how│
                     │  Subagents           → isolated workers  │
                     │  Agent teams         → peer sessions     │
                     │  Hooks               → guaranteed actions│
                     │  MCP servers         → outside systems   │
                     │  Plugins             → packaging + share │
                     └───────────────────┬──────────────────────┘
                                         │
                     ┌───────────────────▼──────────────────────┐
                     │      WHAT IT TOUCHES (blast radius)      │
                     │  your files · your shell · your git      │
                     │  your browser · your SaaS · your cloud   │
                     └──────────────────────────────────────────┘
```

### Breakdown of the big map

**Top layer: one subscription, one pool.** This is the single most misunderstood thing about Claude. Your plan is not "a chat plan plus a coding plan." It is one allowance. Anthropic's pricing page states that activity across web, desktop, mobile, and Claude Code all draws from the same pool. A heavy morning of chatting shrinks your afternoon of coding.

**Middle layer: surfaces.** A "surface" is just a front door. Chat surfaces are conversational. Cowork is an agentic workspace inside Claude Desktop for people who do not want a terminal. Claude Code surfaces are all the same agent wearing different clothes: terminal, desktop app, IDE plugin, cloud VM, Slack bot, SDK.

**Third layer: the extension layer.** This is the answer to "how do I become an expert." Anyone can type prompts. Expertise is configuring the loop: what context loads, what knowledge is available, what runs automatically, what outside systems are reachable, and what is flatly forbidden. Part 4 is the longest part of this guide for that reason.

**Bottom layer: blast radius.** Every capability you add expands what a mistake or a prompt injection can reach. As a security person you will read the extension layer twice: once as "what can I build" and once as "what did I just expose."

---

# Part 1: Vocabulary

Learn these 25 terms and most Claude conversations stop sounding like jargon.

### Core mechanics

**Model.** The brain. Current public generations include Opus (deepest reasoning), Sonnet (balanced workhorse), Haiku (fast and cheap), and Fable (frontier tier with extra safeguards). In Claude Code you refer to them by alias: `opus`, `sonnet`, `haiku`, `fable`.

**Token.** A chunk of text, roughly three quarters of a word. Everything is metered in tokens. Reading a file costs tokens. Tool output costs tokens. Thinking costs tokens.

**Context window.** The model's working memory for one session. Everything it can "see" right now: your prompt, your files, tool results, and its own earlier replies. When it fills up, something has to give.

**Compaction.** When the context window fills, Claude summarizes the older part of the conversation to free space. Useful, but lossy. Details can fall out. You can trigger it with `/compact` or let it happen automatically.

**Agentic loop.** The engine. Claude receives a goal, picks a tool, runs it, reads the result, decides the next step, and repeats until done. Chat is one turn of thinking. An agent is many turns of thinking plus doing.

```
   YOU: "fix the failing auth test"
     │
     ▼
 ┌────────────────────────────────────────────────┐
 │              THE AGENTIC LOOP                  │
 │                                                │
 │   ┌─────────┐   pick    ┌──────────┐           │
 │   │  MODEL  │──────────▶│   TOOL   │           │
 │   │ reasons │           │ Read     │           │
 │   │  about  │           │ Grep     │           │
 │   │  goal   │◀──────────│ Bash     │           │
 │   └─────────┘  result   │ Edit     │           │
 │        │                │ WebFetch │           │
 │        │                │ MCP tool │           │
 │        │                └──────────┘           │
 │        │  "am I done?"                         │
 │        │      no ──────────┐                   │
 │        │                   │ loop again        │
 │        └───────────────────┘                   │
 │        │      yes                              │
 └────────┼───────────────────────────────────────┘
          ▼
   YOU: read the diff, approve or steer
```

**Breakdown.** The loop is why an agent can do real work and also why it can burn your usage limit fast. One instruction from you can become forty tool calls. Two things matter more than prompt wording: (1) give it a way to check its own work, like a test suite, so the loop has a stopping condition it can verify; (2) control which tools are in that tool box, because the loop can only reach what you gave it.

**Harness.** The wrapper around the model that gives it tools, memory, permissions, and rules. Claude Code is a harness. Cursor is a harness. Two harnesses running the same model produce different quality, which is why benchmark scores are often about the harness, not the brain.

**Turn.** One round trip. You send a message, Claude works and replies, the turn ends. Hooks fire on turn boundaries, so this word matters later.

**Effort level.** How hard the model thinks before acting. Options are `low`, `medium`, `high`, `xhigh`, and `max`. Higher effort costs more tokens and finds harder bugs. `ultrathink` in a prompt asks for deep reasoning once.

### Extension mechanics

**Tool.** A capability with a name and a schema. Built-ins include `Read`, `Write`, `Edit`, `Bash`, `Grep`, `Glob`, `WebFetch`, `WebSearch`, `Agent`, `Skill`, and more.

**CLAUDE.md.** A plain markdown file of project rules that loads at the start of every session, every time. "Use pnpm, not npm. Run tests before committing."

**Rules.** Files in `.claude/rules/` that can be scoped to file paths, so a Terraform rule only loads when Claude touches `.tf` files. A way to keep CLAUDE.md small.

**Skill.** A folder with a `SKILL.md` inside it. Reusable knowledge or a workflow. Only the short description sits in context; the body loads when it is needed. You invoke it as `/skill-name`, or Claude loads it when your request matches the description.

**Subagent.** A worker with its own separate context window, its own system prompt, and its own tool restrictions. It does a noisy job and hands back only the summary. Defined by an "agent file," which is markdown with YAML frontmatter in `.claude/agents/`.

**Agent team.** Multiple independent Claude Code sessions that can message each other and share a task list. Heavier and more expensive than subagents. Experimental and off by default.

**Fork.** A subagent that inherits your whole conversation instead of starting fresh. Cheap, because it reuses the same prompt cache. Started with `/subtask`.

**Hook.** Your own code that runs automatically at a specific moment in the loop, like "before any Bash command" or "after every file edit." Hooks are guaranteed. Prompts are requests. This distinction is the heart of Part 8.

**MCP (Model Context Protocol).** An open standard for plugging outside systems into an AI agent. An MCP server exposes tools ("create a Jira ticket," "query this database"). Write the server once and it works across Claude, and across most competing agents too.

**Connector.** The consumer-friendly name for an MCP integration in the Claude apps. Google Drive, Gmail, Slack, Linear, and so on. You authorize it with OAuth, and Claude acts as you.

**Plugin.** A bundle. One installable package that can contain skills, subagents, hooks, MCP servers, and settings. Distributed through a marketplace, which is usually just a Git repo.

**Marketplace.** A catalog of plugins. Anthropic runs an official one called `claude-plugins-official`. You can host your own from a private repo, which matters for enterprise.

### Control mechanics

**Permission mode.** How much Claude has to ask before acting. Modes: `default` (ask), `acceptEdits` (auto-approve file edits), `plan` (read-only, produce a plan first), `auto` (a classifier reviews actions instead of you), `dontAsk` (auto-deny anything not pre-approved), `bypassPermissions` (skip prompts; treat as dangerous).

**Permission rule.** A pattern that allows, asks, or denies specific tool use. Examples: `Bash(git *)`, `Edit(*.ts)`, `Read` deny rules for secret files. Deny rules always win, including over hooks.

**Sandboxing.** OS-level filesystem and network isolation for what Claude runs. Different from permissions. Permissions say "may I," sandboxing says "you physically cannot."

**Checkpoint.** An automatic snapshot of your files as Claude works. `/rewind` restores. Not a replacement for git, and it does not capture everything (changes made through shell commands are not tracked).

**Worktree.** A second working directory on the same git repo. The standard trick for running agents in parallel without them stepping on each other's files.

**Managed settings.** Admin-controlled config pushed to machines that users cannot override. The main enterprise governance lever.

---

# Part 2: What Max actually buys you

Two things to separate in your head: **capacity** (how much you can do) and **capability** (which features exist for you at all). Max is mostly a capacity purchase. Some capability is reserved for Team and Enterprise, which surprises people who assume the most expensive individual plan gets everything.

## 2.1 Capacity: how the limits really work

There are two clocks running at once.

```
   ┌───────────────────────────────────────────────────────────────┐
   │  CLOCK 1: the 5-hour rolling session window                   │
   │                                                               │
   │  first message ──────────────────────────▶ resets 5h later    │
   │  10:00am                                    3:00pm            │
   │  ████████████████████░░░░░░░░  (burned / left)                │
   │                                                               │
   │  Starts on YOUR first message, not on a wall clock.           │
   └───────────────────────────────────────────────────────────────┘

   ┌───────────────────────────────────────────────────────────────┐
   │  CLOCK 2: weekly limits                                       │
   │                                                               │
   │  Max plans get TWO weekly buckets:                            │
   │    (a) all models   ████████████░░░░░░░░                      │
   │    (b) Sonnet only  ████░░░░░░░░░░░░░░░░                      │
   │                                                               │
   │  Resets at a fixed day+time assigned to your account.         │
   │  Not Monday. Not your signup date. Check Settings > Usage.    │
   └───────────────────────────────────────────────────────────────┘

   ┌───────────────────────────────────────────────────────────────┐
   │  ONE SHARED POOL feeds both clocks                            │
   │                                                               │
   │   chat  ──┐                                                   │
   │   Cowork ─┼──▶ [ your plan allowance ] ──▶ throttle when empty│
   │   Code  ──┘                                                   │
   │   IDE, Slack, web sessions, routines: same pool               │
   └───────────────────────────────────────────────────────────────┘
```

### Breakdown of the limits diagram

**Clock 1 is a sprint limit.** It stops a single marathon session from consuming everything. Hitting it costs you a coffee break, never more than five hours.

**Clock 2 is the one that actually hurts.** A weekly cap resets at a fixed time tied to your account. If you drain it Wednesday, you are throttled until that fixed reset. The second Sonnet-only bucket exists so heavy Sonnet use does not starve your access to everything else.

**Anthropic does not publish token counts.** The official material gives multipliers only. Max 5x means five times Pro's per-session usage. Max 20x means twenty times. The baseline Pro is measured against is not disclosed, and Anthropic reserves the right to adjust caps. Treat any blog post quoting an exact number like "44,000 tokens per window" as invented.

**Practical consequence:** the only trustworthy gauge is in-product. Use `/usage` and `/status` in Claude Code, and Settings > Usage on claude.ai.

**Usage credits are the escape hatch.** On paid plans you can turn on usage credits to keep working at standard API rates once your included allowance is gone. That converts a hard stop into a variable bill. Decide before a crunch week whether you want that switch on.

## 2.2 Capability: the Max feature table

This is the part people get wrong. Straight from Anthropic's feature availability page:

| Feature                                                 | Pro | **Max** | Team  |        Enterprise        |
| :------------------------------------------------------ | :-: | :-----: | :---: | :----------------------: |
| Claude Code CLI, subagents, hooks, skills, MCP, plugins | yes | **yes** |  yes  |           yes            |
| Claude Code on the web (cloud sessions)                 | yes | **yes** |  yes  |    yes (premium seat)    |
| Routines (`/schedule`, cron + GitHub triggers)          | yes | **yes** |  yes  |           yes            |
| Remote Control (drive a local session from phone)       | yes | **yes** | admin |          admin           |
| Channels (agent talks to outside systems)               | yes | **yes** | admin |          admin           |
| Computer use (Claude drives your screen)                | yes | **yes** |  no   |            no            |
| Dispatch (Desktop)                                      | yes | **yes** |  no   |            no            |
| Ultraplan / Ultrareview                                 | yes | **yes** |  yes  |           yes            |
| Chrome extension, voice dictation                       | yes | **yes** |  yes  |           yes            |
| Cowork                                                  | yes | **yes** |  yes  |           yes            |
| **Code Review (PR-time multi-agent review)**            | no  | **NO**  |  yes  |           yes            |
| **Artifacts (publish session output as a page)**        | no  | **NO**  |  yes  |          admin           |
| **Analytics dashboard + API**                           | no  | **NO**  |  yes  |           yes            |
| **Server-managed settings**                             | no  | **NO**  |  yes  |           yes            |
| **SSO**                                                 | no  | **NO**  |  yes  |           yes            |
| SCIM, Compliance API, Zero Data Retention               | no  | **NO**  |  no   | yes (ZDR needs enabling) |

**Read that middle column twice.** Max is the best individual plan for doing work. It is not the plan for governing an organization's work. If you want PR-time Code Review, usage analytics across your team, or settings you can push to other people's machines, that is a Team or Enterprise conversation, not a Max upgrade.

Interesting quirk in the other direction: **computer use and Dispatch are Pro and Max only, not Team or Enterprise.** Individual plans get some toys the corporate plans deliberately do not.

## 2.3 The rule that trips people up: OAuth is for Anthropic's own products

From Anthropic's legal and compliance page for Claude Code, paraphrased:

- **OAuth login** (your claude.ai account) is intended for ordinary use of Claude Code and other native Anthropic applications.
- **Developers building products or services**, including anything on the Agent SDK, should use **API key** auth through Claude Console or a supported cloud provider.
- Anthropic does not permit third-party developers to offer Claude.ai login or route requests through Free, Pro, or Max credentials on users' behalf, and reserves the right to enforce that without notice.

In April 2026 this got real. Anthropic cut off subscription OAuth tokens being used by third-party agent harnesses, citing the load those usage patterns put on capacity. Tools that had been piggybacking on subscription quotas broke.

```
   ALLOWED                                 NOT ALLOWED
   ───────────────────────────────         ─────────────────────────────
   claude.ai account (OAuth)               claude.ai account (OAuth)
        │                                       │
        ├─▶ Claude chat / Desktop / mobile      ├─▶ third-party harness  ✗
        ├─▶ Cowork                              ├─▶ your own product     ✗
        ├─▶ Claude Code CLI (any machine,       └─▶ someone else's tool  ✗
        │   local or over SSH)
        ├─▶ Claude Code Desktop / VS Code
        │   / JetBrains / web / Slack
        └─▶ Remote Control

   For anything in the right column: use an ANTHROPIC_API_KEY
   from Claude Console, or Bedrock / Vertex / Foundry. You pay per token.
```

### Breakdown

**Running the official binary anywhere is fine.** SSH into a server and run `claude` there on your subscription: supported.

**Wrapping the model in your own thing is an API-key activity.** Building an internal AppSec bot on the Agent SDK is a Console API key job, billed per token, separate from your Max plan.

**Watch this footgun.** If `ANTHROPIC_API_KEY` is set in your environment, Claude Code uses that key instead of your subscription. You will get an API bill while believing you are on Max. Check with `/status`.

## 2.4 What "priority access" means for you

Max gets early access to new models and features. Practically that means research previews land in your lap before they are documented well. Two habits help:

1. Skim `code.claude.com/docs/en/whats-new/` weekly. Anthropic publishes a weekly changelog page.
2. Expect features labeled "research preview" to change shape. Do not build a team process on one until it stabilizes.

---

# Part 3: The tools, surface by surface

## 3.1 Quick chooser

|Surface|Best for|Runs where|You need|
|:--|:--|:--|:--|
|**Claude chat** (web/desktop/mobile)|thinking, writing, research, analysis|Anthropic cloud|nothing|
|**Cowork** (in Desktop)|multi-step knowledge work on local files, decks, spreadsheets|local VM on your machine|Desktop app|
|**Claude Code CLI**|all serious agentic work, full control|your terminal|install + login|
|**Claude Code Desktop**|same agent with a GUI: diffs, previews, parallel sessions|your machine|Desktop app|
|**VS Code / JetBrains**|staying in your editor|your IDE|extension|
|**Claude Code on the web**|fire-and-forget cloud tasks against a GitHub repo|Anthropic cloud VM|GitHub connect|
|**Routines**|scheduled and event-triggered agent runs|Anthropic cloud VM|subscription|
|**Claude Code in Slack**|delegating from where your team talks|Anthropic cloud|Slack install|
|**Remote Control**|driving your local session from your phone|your machine + phone|subscription|
|**Claude in Chrome**|browser automation, testing real web apps|your browser|extension|
|**Claude in Excel / PowerPoint**|working inside Office documents|Office|subscription|
|**Claude Design**|design work on a canvas with chat|Anthropic cloud|subscription|
|**Claude Tag**|tag `@Claude` into Slack threads for anyone to delegate|Slack|setup|
|**Agent SDK** (TypeScript / Python)|building your own products and internal tools|wherever you host it|**API key**|

## 3.2 Claude chat: the surface people underuse

Most people treat chat as a smarter search box. The features that change its value:

- **Projects.** A container with its own knowledge base and instructions. Upload your standards, your architecture docs, your review checklist once, and every chat in that project starts with them. Caching makes repeated reference cheap.
- **Memory.** Claude can carry durable facts about you and your work between conversations, and you can inspect and edit what it kept.
- **Connectors.** OAuth links to Drive, Gmail, Calendar, and many more. Claude acts as you, with your permissions. No shared service accounts.
- **Skills.** The same `SKILL.md` concept as Claude Code, managed for your account. This matters because Cowork and cloud sessions load account-level skills, not the ones sitting in `~/.claude/skills/` on your laptop.
- **Research.** Multi-source investigation that runs for several minutes and returns a sourced report.
- **Artifacts and Code Execution.** Chat can write and run code and produce real files (xlsx, pptx, docx, pdf).

**Expert move:** keep one project per recurring responsibility. A "pipeline reviews" project with your standard loaded beats re-explaining your standard forty times.

## 3.3 Cowork: agentic work without a terminal

Cowork uses the same architecture as Claude Code, inside Claude Desktop, without the command line. Anthropic's docs describe four properties:

- works directly on your computer, reading and writing local files with no upload dance
- pairs with Claude in Chrome to automate websites
- coordinates subagents, splitting work into parallel streams
- produces real deliverables: spreadsheets with working formulas, slide decks, formatted docs

It extends with the same three things as everything else: **connectors** (access), **skills** (instructions), **plugins** (bundles). You manage them from **Customize** in the Desktop sidebar.

**The gotcha worth memorizing:** Cowork loads the skills and plugins enabled for your **claude.ai account**, synced at session start. It does **not** read `~/.claude/` on your machine. So a skill you wrote for your CLI will not show up in Cowork until you add it in Customize. Cloud sessions behave similarly, but they also pick up project skills committed to the repo's `.claude/skills/`.

```
   WHERE SKILLS COME FROM, BY SURFACE

   Claude Code CLI (local)
     ├── ~/.claude/skills/            ← your personal skills
     ├── ./.claude/skills/            ← project skills (committed)
     ├── parent + nested dirs         ← monorepo package skills
     ├── plugins                      ← installed bundles
     └── managed settings             ← pushed by your admin

   Cowork + cloud sessions + routines
     ├── skills enabled for your claude.ai ACCOUNT   ← "Customize"
     └── (cloud only) repo's ./.claude/skills/       ← committed

   ✗ neither Cowork nor cloud sessions read ~/.claude/skills/
```

**Breakdown.** If a routine fails with "skill not found," this diagram is your answer. Each routine run is a fresh remote session. Either enable the skill for your account, commit it to the repo, or ship it in a plugin declared in the repo's `.claude/settings.json`.

## 3.4 Claude Code CLI: the center of gravity

If you learn one tool deeply, learn this one. It is where the whole extension layer is available with no caveats, on every provider.

**Install and check.**

```bash
# after install, verify and diagnose
claude --version
claude                     # start an interactive session
/doctor                    # setup checkup: config problems, duplicates, context cost
/status                    # model, auth method, versions
/context                   # what is eating your context window
/usage                     # how much of your limits you have burned
```

**The commands that carry the most weight.**

|Command|What it does|
|:--|:--|
|`/init`|writes a first CLAUDE.md by reading your repo|
|`/memory`|view and edit CLAUDE.md and auto memory|
|`/skills`|list skills, toggle visibility per skill|
|`/agents`|manage subagent definitions|
|`/hooks`|browse every registered hook, grouped by event (read-only)|
|`/mcp`|server connection status and per-server token cost|
|`/plugin`|install, enable, disable plugins and marketplaces|
|`/permissions`|see and edit allow / ask / deny rules|
|`/rewind`|restore files to an earlier checkpoint|
|`/compact`, `/clear`|manage context deliberately|
|`/subtask`|fork the conversation into a background worker|
|`/tasks`|see running and finished background work|
|`/btw`|ask a side question with full context and no tool access; answer is discarded|
|`/loop`|run a prompt repeatedly on an interval|
|`/schedule`|create a Routine (cron, API, or GitHub trigger)|
|`/goal`|keep the session running until a condition is met|
|`/security-review`|one-pass security review of the current branch|
|`/code-review`|bundled review skill, runs as a forked subagent|
|`/run`, `/verify`|launch your app and confirm a change actually works|
|`/run-skill-generator`|record how to build and launch this project, once|
|`/statusline`|build a custom status line (context %, cost, git state)|
|`/workflows`|watch a running workflow|
|`/advisor`|pair a second model in as a consultant|
|`/fast`|toggle fast mode|
|`/doctor`|setup checkup|

**Flags worth knowing.**

```bash
claude -p "summarize the diff"          # headless / non-interactive, prints and exits
claude --agent code-reviewer            # run the WHOLE session as a subagent definition
claude --agents '{...json...}'          # define throwaway subagents for one session
claude --model opus                     # pick a model
claude --permission-mode plan           # start read-only
claude --add-dir ../shared-lib          # grant access to another directory
claude --worktree                        # start in an isolated git worktree
claude --strict-mcp-config              # ignore ambient MCP config
claude --bare                            # minimal startup
claude --debug-file /tmp/claude.log     # log everything for troubleshooting
```

## 3.5 Claude Code Desktop: the GUI for the same agent

Same agent, more eyes. What the desktop app adds over the terminal:

- a diff view for reviewing changes, plus PR status monitoring
- app preview, so it launches your app and you watch it work
- a built-in browser for external sites, with per-site approval
- parallel sessions in tabs, plus background tasks you can watch
- **local, cloud, or SSH** execution environments from one window
- **computer use** (Pro and Max), where Claude drives your screen for things no API can reach
- **scheduled tasks** that run locally on your machine, which unlike Routines can see your local skills

It shares configuration with the CLI, so your CLAUDE.md, skills, agents, and hooks carry over. A few CLI features are not there yet; the docs keep a "what's not available in Desktop" list.

## 3.6 Cloud, scheduled, and event-driven surfaces

This trio is how you stop babysitting.

```
   THE AUTOMATION LADDER (least to most autonomous)

   rung 1  INTERACTIVE          you type, you approve each step
           claude

   rung 2  HEADLESS             one shot, scriptable, no UI
           claude -p "..." --output-format json
           └─▶ pipe into anything: cron, Makefile, CI step

   rung 3  BACKGROUND / PARALLEL  many agents, you supervise
           claude agents          (agent view: dispatch + monitor)
           /subtask, worktrees, agent teams

   rung 4  CLOUD SESSIONS        runs on Anthropic's VM, not your laptop
           Claude Code on the web  ──▶ opens a PR when done
           /web-setup, --cloud, --teleport (pull it back local)

   rung 5  ROUTINES              triggered without you present
           /schedule  ──▶ cron schedule
                      ──▶ GitHub event (PR opened, issue labeled)
                      ──▶ API call (your own system fires it)

   rung 6  CI/CD                 part of the pipeline itself
           GitHub Actions (@claude in a PR), GitLab CI/CD
           Code Review (Team/Enterprise) on every PR
```

### Breakdown of the automation ladder

**Rungs 1 to 3 are your machine, your permissions, your eyeballs.** Start here always.

**Rung 4 changes the trust model.** Cloud sessions run on Anthropic infrastructure in an isolated environment. Two consequences: your local `~/.claude` config does not come along, and network access is controlled by an allowlist you configure. Setup scripts and `SessionStart` hooks prepare the environment.

**Rung 5 is where automation gets real and risky at the same time.** A Routine triggered by "PR opened" is an unattended agent reacting to input from outside your organization. That is a prompt injection surface. Treat routine prompts and permissions with the same care as a webhook handler.

**Rung 6 is the pipeline.** Claude Code Action for GitHub, and a GitLab equivalent. Both docs pages have explicit "security considerations" sections. Read them before you wire an agent to your default branch.

**Teleporting.** You can move work between web and terminal in both directions: send a local task to the cloud with `--cloud`, and pull a cloud session down to your terminal with `--teleport`.

## 3.7 The Agent SDK: when you are building, not using

TypeScript and Python. Same agent loop, exposed as a library, so you can put it inside your own product or internal tool.

What you get: the agentic loop, built-in tools, custom tools you define, MCP support, subagents defined in code, hooks as callbacks, structured outputs with Zod or Pydantic schemas, session persistence with a pluggable store, permissions, cost tracking, and OpenTelemetry observability.

```python
# shape of it, Python
from claude_agent_sdk import query, ClaudeAgentOptions

options = ClaudeAgentOptions(
    system_prompt="You are an AppSec triage assistant.",
    allowed_tools=["Read", "Grep", "Glob"],
    permission_mode="dontAsk",
    max_turns=12,
)

async for message in query(prompt="Triage findings in ./report.sarif", options=options):
    print(message)
```

**Three things to know before you build on it:**

1. **API key, not your subscription.** See Part 2.3.
2. **It is a subprocess model.** The SDK spawns the Claude Code binary and keeps state on local disk. The hosting docs cover container sizing, session persistence, and multi-tenant isolation.
3. **There is a secure deployment guide.** It covers the threat model, isolation options (sandbox runtime, containers, gVisor, VMs), and the credential proxy pattern so your agent never holds long-lived secrets. Read this one twice.

---

# Part 4: The extension layer (where expertise lives)

Seven mechanisms. Each solves a different problem. Most people learn two and stop, then wonder why their setup feels fragile.

## 4.1 The chooser

```
                        WHICH MECHANISM DO I REACH FOR?

   Claude got a convention wrong twice
        └──▶ CLAUDE.md                      (always-on rule)

   The rule only matters for certain files (*.tf, /admin/**)
        └──▶ .claude/rules/ with `paths:`   (scoped rule)

   I keep pasting the same checklist or procedure
        └──▶ Skill                          (on-demand knowledge / workflow)

   I keep typing the same multi-step prompt
        └──▶ Skill with a /name             (invocable workflow)

   A side task floods my window with output I will never re-read
        └──▶ Subagent                       (isolated worker)

   I want the SAME worker every time, with restricted tools
        └──▶ Custom subagent file           (.claude/agents/x.md)

   Workers need to talk to each other and share a task list
        └──▶ Agent team                     (experimental, expensive)

   The side task needs my whole conversation as background
        └──▶ /subtask (fork)                (cheap, shares prompt cache)

   This MUST happen every single time, no exceptions
        └──▶ Hook                           (deterministic, guaranteed)

   Claude needs data or actions from an outside system
        └──▶ MCP server                     (tools + auth)

   A second repo or teammate needs the same setup
        └──▶ Plugin + marketplace           (packaging)
```

### Breakdown of the chooser

**The single most important distinction on this page:** a rule written in CLAUDE.md or a skill is a **request**. A hook is **enforcement**. "Never edit `.env`" in CLAUDE.md is a strong hint that usually works. A `PreToolUse` hook that blocks the edit is a control. If you come from security, map that to "policy document" versus "policy engine." You need both, and you must not confuse them.

**Second most important:** skills and subagents look similar and are not. A skill is content that loads into whatever context is running. A subagent is a separate context window. Reach for a skill to share knowledge. Reach for a subagent to protect your context window.

## 4.2 Context economics (read this before configuring anything)

Every mechanism you add has a context cost. Here is Anthropic's own table, which is the closest thing to a physics equation for this work:

|Mechanism|When it loads|What loads|Cost|
|:--|:--|:--|:--|
|CLAUDE.md|session start|full content|**every request, forever**|
|Skills|start + on use|descriptions at start, body on use|low|
|MCP servers|session start|tool names only, schemas deferred|low until used|
|Code intelligence|after edits, on lookup|diagnostics, symbol info|low, often net negative|
|Subagents|when spawned|fresh isolated context|isolated from you|
|Hooks|on trigger|nothing|**zero**, unless it returns output|

```
   CONTEXT WINDOW, ONE SESSION

   ┌─────────────────────────────────────────────────────────────┐
   │ system prompt + tool definitions          [fixed overhead]  │
   ├─────────────────────────────────────────────────────────────┤
   │ CLAUDE.md (all levels, concatenated)      [PAID EVERY TURN] │
   ├─────────────────────────────────────────────────────────────┤
   │ skill NAMES + DESCRIPTIONS (a listing)    [paid every turn]  │
   ├─────────────────────────────────────────────────────────────┤
   │ MCP tool names (schemas deferred)         [paid every turn]  │
   ├─────────────────────────────────────────────────────────────┤
   │ your conversation + files read + tool output                 │
   │ ████████████████████████░░░░░░░░░░░░░  ← this is what grows │
   ├─────────────────────────────────────────────────────────────┤
   │ invoked skill BODIES (stay for the session)                  │
   └─────────────────────────────────────────────────────────────┘
                    │
                    ▼  window fills
            ┌────────────────┐
            │   COMPACTION   │  older turns become a summary
            │  (lossy!)      │  invoked skills get re-attached
            └────────────────┘  within a token budget

   SUBAGENT: a completely separate box. Only the summary comes back.
   ┌────────────┐   task    ┌──────────────────┐
   │ main       │──────────▶│ subagent window  │ reads 40 files,
   │ session    │◀──────────│ (its own)        │ runs the test suite,
   └────────────┘  summary  └──────────────────┘ returns 12 lines
```

### Breakdown

**"Paid every turn" is the phrase to internalize.** A 900-line CLAUDE.md is not a one-time cost. It is rent on every single request for the whole session, and it also adds noise that can make Claude miss the rules that matter. Anthropic's guidance: keep CLAUDE.md **under 200 lines** and move reference material to skills.

**Skill listings have a budget too.** The name-and-description listing gets truncated if you have many skills. The budget defaults to about 1% of the model's context window. When it overflows, descriptions are dropped starting with skills you invoke least, which silently breaks auto-triggering. `/doctor` estimates the cost and names the worst offenders.

**Hooks are free until they speak.** A hook that lints and exits quietly costs zero context. A hook that dumps 500 lines of lint output into the conversation costs 500 lines of context. Design hook output like you would design a log line.

**Subagents are the pressure valve.** Anything noisy (test runs, log parsing, doc fetching, big searches) belongs in one.

## 4.3 CLAUDE.md, rules, and memory

Three layers of persistent context.

**CLAUDE.md** loads every session. Files load from your working directory up to the repo root, and nested ones load as Claude touches subdirectories. All levels are **additive**. Put "always" and "never" rules here.

```markdown
# CLAUDE.md

## Commands
- Install: `pnpm install` (never npm)
- Test: `pnpm test` (must pass before any commit)
- Lint: `pnpm lint --fix`

## Conventions
- All API handlers validate input with the shared zod schemas in src/schemas
- Never write to files under infra/prod/ without asking
- Errors return the shared ApiError shape, never raw exceptions

## Structure
- src/api    HTTP handlers
- src/domain business logic, no I/O
- src/adapters everything that talks to the outside world

@docs/architecture-decisions.md
```

That last line is an **import**. You can pull other files in with `@path`.

**Rules** live in `.claude/rules/*.md` and can be path-scoped:

```markdown
---
paths: ["**/*.tf", "infra/**"]
---
- Every S3 bucket needs `block_public_access` set
- No inline IAM policies; reference the modules in infra/modules/iam
- Tag everything with `owner` and `data_classification`
```

That rule costs nothing until Claude touches Terraform. This is the main trick for keeping a large repo's guidance manageable.

**AGENTS.md** is also read, which matters because it is the cross-vendor convention other agents use. If you keep an AGENTS.md for portability, Claude Code honors it.

**Auto memory** is separate. Claude can save durable facts on its own between sessions. Inspect and edit it with `/memory`, and you can turn it off if you want fully deterministic context.

**Subagent memory** is a third thing: a subagent with `memory: project` gets a persistent directory at `.claude/agent-memory/<name>/` and builds up knowledge over time. A code-reviewer subagent that remembers your recurring issues gets better across weeks. That is one of the highest-value and least-used features in the product.

## 4.4 Skills: the most flexible mechanism

A skill is a folder. Inside it, `SKILL.md`. That is the whole idea.

```
   ~/.claude/skills/pipeline-review/
   ├── SKILL.md            ← required: frontmatter + instructions
   ├── reference.md        ← loaded only when Claude needs it
   ├── checklist.md
   └── scripts/
       └── extract-workflows.py   ← executed, not read into context
```

### Anatomy

```markdown
---
name: pipeline-review
description: Reviews CI/CD pipeline configs against our pipeline security
  standard. Use when the user asks to review a workflow file, a Jenkinsfile,
  a GitLab CI config, or asks about pipeline security posture.
argument-hint: [path-to-workflow-file]
allowed-tools: Read Grep Glob Bash(python3 ${CLAUDE_SKILL_DIR}/scripts/*)
disable-model-invocation: false
paths: [".github/workflows/**", ".gitlab-ci.yml", "Jenkinsfile"]
---

## Context
Workflow files present: !`ls -1 .github/workflows/ 2>/dev/null`

## Instructions
Review $ARGUMENTS against these controls, in this order:

1. Pinned actions. Every third-party action pinned to a full commit SHA,
   not a tag. Flag any `@v1` or `@main`.
2. Permissions. Top-level `permissions:` present and least-privilege.
   Flag any missing block or `write-all`.
3. Secrets. No secrets in `run:` steps. No `pull_request_target` with a
   checkout of untrusted code.
4. Triggers. Flag `workflow_run` and `pull_request_target` for review.

For each finding: control, severity, the exact line, and the fix as a diff.
See [reference.md](reference.md) for the full control list.
```

### The frontmatter fields that matter

|Field|Use it to|
|:--|:--|
|`description`|tell Claude when to load this. **The most important field.** Put the key use case first|
|`when_to_use`|add trigger phrases and example requests|
|`disable-model-invocation: true`|make it **you-only**. Use for anything with side effects (`/deploy`, `/commit`). Also drops its context cost to zero|
|`user-invocable: false`|make it **Claude-only**. For background knowledge that is not a command|
|`allowed-tools`|pre-approve tools for the turn that invokes it, so it does not prompt|
|`disallowed-tools`|remove tools while it is active|
|`context: fork`|run it in an isolated subagent instead of your conversation|
|`agent`|which subagent type runs it when forked (`Explore`, `Plan`, custom)|
|`paths`|only auto-load when working on matching files|
|`model`, `effort`|change model or thinking depth while it runs|
|`arguments`|name positional args so you can use `$issue` instead of `$0`|
|`hooks`|attach hooks that live only while this skill is active|

### Two features that make skills powerful

**1. Dynamic context injection.** A line like `` !`git diff HEAD` `` runs _before_ Claude sees the skill. The output replaces the placeholder. So the skill arrives with real data already inside it, not with an instruction to go get data.

```markdown
## Current state
- Diff: !`git diff HEAD`
- Failing tests: !`pnpm test 2>&1 | tail -40`
- Open PR: !`gh pr view --json title,body 2>/dev/null`
```

For multi-line, use a fenced block opened with ` ```! `.

**2. Running in a subagent.** Add `context: fork` and the skill body becomes the prompt for an isolated worker. Your main window stays clean. Combine with `agent: Explore` for read-only research.

### Skill hygiene

- Keep `SKILL.md` under about 500 lines. Push detail into sibling files and reference them.
- The body **stays in context for the rest of the session** once loaded. Every line is recurring rent. Write instructions, not narration.
- Write descriptions with the words a user would actually say. A skill that never triggers usually has a description problem, not an instructions problem.
- **Measure it.** Install `skill-creator` from the official marketplace. It runs each test prompt with and without the skill in isolated subagents, grades the output, and reports pass rate, tokens, and time. It also does blind A/B between two versions and tunes descriptions by measuring should-trigger and should-not-trigger hit rates. "It worked when I tried it" is not evidence.

### Bundled skills you already have

`/doctor`, `/code-review`, `/security-review`, `/batch`, `/debug`, `/loop`, `/run`, `/verify`, `/run-skill-generator`, `/claude-api`, and more. Custom commands in `.claude/commands/*.md` still work and are the same mechanism; skills just add a folder, frontmatter, and auto-loading.

## 4.5 Subagents and agent files

An agent file is markdown with YAML frontmatter. Location decides scope.

|Location|Scope|Priority|
|:--|:--|:-:|
|managed settings|whole organization|1 (wins)|
|`--agents` CLI flag|this session only|2|
|`.claude/agents/`|this project (commit it)|3|
|`~/.claude/agents/`|all your projects|4|
|plugin `agents/`|wherever the plugin is on|5|

### A real agent file

```markdown
---
name: appsec-triage
description: Triages security scanner findings against the actual codebase to
  decide which are real. Use proactively when the user mentions SAST results,
  a SARIF file, Dependabot alerts, or asks whether a finding is exploitable.
tools: Read, Grep, Glob, Bash(git *), Bash(rg *)
disallowedTools: Write, Edit
model: sonnet
effort: high
maxTurns: 30
memory: project
skills:
  - our-threat-model
  - taint-analysis-patterns
color: orange
---

You are an application security engineer triaging scanner output.

For every finding, answer three questions in order and stop early when you can:

1. REACHABILITY. Is the vulnerable code actually reachable from an entry
   point? Trace it. If it is dead code or test-only, mark NOT EXPLOITABLE
   and stop.
2. CONTROLS. Is there a sanitizer, framework protection, or authorization
   check between the entry point and the sink? Name the file and line.
3. IMPACT. If reachable and unprotected, what does an attacker get?

Output a table: ID | verdict | confidence | evidence (file:line) | reasoning.
Verdicts: EXPLOITABLE, NOT EXPLOITABLE, NEEDS HUMAN.

Never guess. If you cannot trace the path with evidence, say NEEDS HUMAN.

Before you start, read your agent memory for patterns you have already
confirmed in this codebase. When you finish, write new patterns you learned
back to memory: framework protections, common false-positive shapes, and
where the real entry points are.
```

### The frontmatter fields

Only `name` and `description` are required. The rest:

|Field|What it does|
|:--|:--|
|`tools`|allowlist. Omit to inherit everything available to subagents|
|`disallowedTools`|denylist. Applied first, then `tools` resolves against what is left|
|`model`|`sonnet`, `opus`, `haiku`, `fable`, a full ID, or `inherit` (the default)|
|`permissionMode`|`default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, `plan`|
|`maxTurns`|hard stop on agentic turns|
|`skills`|**preload** full skill content at startup, not just descriptions|
|`mcpServers`|give this agent MCP servers, inline or by name. Keeps them out of your main context|
|`hooks`|lifecycle hooks scoped to this agent only|
|`memory`|`user`, `project`, or `local`. Persistent learning directory|
|`background`|force it to run in the background|
|`effort`|thinking depth while active|
|`isolation: worktree`|give it its own copy of the repo in a temp git worktree|
|`color`|display color in the task list|
|`initialPrompt`|auto-submitted first turn when run as the main session via `--agent`|

### Built-in subagents you get for free

- **Explore.** Fast, read-only search and analysis. Skips CLAUDE.md and git status to stay cheap. Write and Edit denied.
- **Plan.** Read-only research used by plan mode.
- **general-purpose.** Full toolset, multi-step work.
- Plus helpers like `statusline-setup` and `claude-code-guide`.

### The isolation picture

```
   THREE WAYS TO PARALLELIZE, COMPARED

   (A) SUBAGENT                       fresh context, own tools, own model
   ┌───────────┐  delegation  ┌──────────────┐
   │   main    │─────────────▶│   subagent   │  sees: its prompt,
   │  session  │◀─────────────│              │         CLAUDE.md,
   └───────────┘   summary    └──────────────┘         preloaded skills
   cost: low. isolation: high. knows nothing about your conversation.

   (B) FORK  (/subtask)               inherits EVERYTHING
   ┌───────────┐    copy      ┌──────────────┐
   │   main    │═════════════▶│     fork     │  same history, prompt,
   │  session  │◀─────────────│              │  tools, model
   └───────────┘   summary    └──────────────┘  shares prompt cache
   cost: lowest per-token. isolation: output only. no re-explaining.

   (C) AGENT TEAM                     independent peers
   ┌───────────┐              ┌──────────────┐
   │   lead    │◀────────────▶│  teammate 1  │  each is a FULL
   │  session  │◀────────────▶│  teammate 2  │  Claude Code session
   └───────────┘   messages   └──────────────┘  shared task list
                   ▲                  ▲          peer-to-peer messages
                   └──────────────────┘
   cost: highest (N sessions). isolation: total. best for debate.

   (D) WORKTREE ISOLATION             orthogonal to all of the above
   repo/                 .claude/worktrees/task-a/    ← agent A edits here
     └── .git (shared)   .claude/worktrees/task-b/    ← agent B edits here
   Use `isolation: worktree` or `--worktree` so two agents never
   fight over the same file on disk.
```

### Breakdown of the isolation picture

**Use a subagent by default.** It is the cheapest way to keep your window clean, and restricting its tools is real security value, not just tidiness.

**Use a fork when the background story is long.** If explaining the situation to a fresh subagent would take three paragraphs, fork instead. It also reuses the parent's prompt cache, which makes it cheaper than spawning fresh.

**Use an agent team only when workers need to argue.** Competing hypotheses during a hard debugging session. Parallel review where reviewers should challenge each other. It is experimental, off by default, and each teammate is a full session's worth of tokens.

**Use worktrees whenever two agents write files at once.** Without it, two agents in one directory will corrupt each other's work. Claude Code can do this for you.

### Limits to keep in mind

- 200 subagents per session by default, and 20 running concurrently. Both tunable by environment variable.
- Nesting is off by default. A subagent cannot spawn its own subagents unless you raise `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`.
- **Subagent output gets scanned** before Claude reads it. If a subagent read a web page or file that contained something shaped like an instruction (a fake `<system-reminder>`, a line starting with `Human:`, a mention of `bypassPermissions`), Claude Code flags or neutralizes the imitation. That is a prompt-injection mitigation, not a guarantee. It does not stop the underlying instruction from influencing the model, and any resulting tool call still goes through permissions.

## 4.6 Hooks: the enforcement layer

A hook is your own code, wired to a moment in the loop. This is the mechanism a security engineer should learn first.

### The lifecycle

```
   SESSION LIFECYCLE AND WHERE HOOKS FIRE

   claude starts
     │
     ├─ SessionStart ................ bootstrap, inject context, set env
     ├─ Setup ....................... one-time prep (--init / CI)
     ├─ InstructionsLoaded .......... CLAUDE.md / rules loaded
     │
     ▼   ┌──────────────── ONE TURN ────────────────────────────┐
         │                                                      │
   you type a prompt                                            │
     ├─ UserPromptSubmit ........ inspect / block / ADD CONTEXT  │
     ├─ UserPromptExpansion ..... a /command expanded            │
     │                                                          │
     │   ┌─────────── TOOL CALL (repeats N times) ────────┐      │
     │   ├─ PreToolUse .......... ALLOW / DENY / ASK      │      │
     │   ├─ PermissionRequest ... answer the dialog for me │      │
     │   ├─ PermissionDenied .... auto mode said no        │      │
     │   │        [tool runs]                              │      │
     │   ├─ PostToolUse ........ lint, scan, log           │      │
     │   ├─ PostToolUseFailure . it errored                │      │
     │   └─ PostToolBatch ...... a parallel batch resolved │      │
     │                                                          │
     ├─ SubagentStart / SubagentStop ... per worker              │
     ├─ TaskCreated / TaskCompleted .... todo lifecycle          │
     ├─ MessageDisplay ................. text being shown        │
     ├─ Notification ................... Claude needs you        │
     ├─ Stop ........................... turn ending: GATE IT    │
     └─ StopFailure .................... turn died on API error  │
         └──────────────────────────────────────────────────────┘
     │
     ├─ PreCompact / PostCompact ..... context being summarized
     ├─ ConfigChange ................. someone edited settings/skills
     ├─ CwdChanged / FileChanged ..... environment drift
     ├─ WorktreeCreate / WorktreeRemove
     ├─ Elicitation / ElicitationResult .... MCP asked the user something
     └─ SessionEnd ................... cleanup
```

### Breakdown of the lifecycle

**Four events do 90% of real work.**

- `PreToolUse` is your policy engine. It sees the tool and its arguments before anything happens, and it can deny.
- `PostToolUse` is your reaction: format, scan, log, notify.
- `UserPromptSubmit` is your context injector. Anything it prints to stdout gets added to Claude's context.
- `Stop` is your quality gate: "tests must pass before you are allowed to finish."

**`PreToolUse` fires in every permission mode, including `bypassPermissions`.** That is the single most important sentence in the hooks documentation for a security person. A hook that denies cannot be bypassed by a developer switching modes or passing `--dangerously-skip-permissions`. Hooks can tighten, never loosen: a hook returning `allow` still cannot override a deny rule.

**`ConfigChange` is your audit trail.** It fires when someone edits settings or skills mid-session. Log it.

### How a hook talks to Claude

Input arrives as JSON on stdin. The exit code decides what happens.

```bash
#!/bin/bash
# .claude/hooks/block-secret-reads.sh
# Registered on PreToolUse with matcher "Read|Edit|Write"

INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

BLOCKED=(".env" "id_rsa" ".aws/credentials" "secrets.yaml" ".kube/config")

for pattern in "${BLOCKED[@]}"; do
  if [[ "$FILE" == *"$pattern"* ]]; then
    echo "Blocked: $FILE matches protected pattern '$pattern'." >&2
    echo "Ask a human if you need this value." >&2
    exit 2      # 2 = BLOCK. stderr goes back to Claude as feedback.
  fi
done

exit 0          # 0 = no objection. normal permission flow still applies.
```

**Exit codes.**

- `0` = no objection. For `PreToolUse` this is _not_ an approval; normal permissions still apply. For `UserPromptSubmit`, `UserPromptExpansion`, and `SessionStart`, stdout is added to Claude's context.
- `2` = block. stderr becomes Claude's feedback so it can adjust.
- anything else = proceed, and the transcript shows a hook error.

**For finer control, exit 0 and print JSON instead:**

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Production infra changes require a ticket. Open one first."
  }
}
```

### The five hook types

|Type|What it runs|Good for|
|:--|:--|:--|
|`command`|a shell command or script|almost everything|
|`http`|POSTs the event to a URL|a shared team audit service|
|`mcp_tool`|calls a tool on a connected MCP server|reuse an existing integration|
|`prompt`|a single LLM call (Haiku by default) returning yes/no|judgment calls, not rules|
|`agent`|a subagent that can read files and run commands|verification that needs investigation|

The `prompt` and `agent` types are the interesting new ground. A `Stop` hook of type `prompt` can ask "did you actually finish everything the user asked?" and, if the answer is no, feed the reason back so Claude keeps working. An `agent` hook can actually run the test suite and check.

```json
{
  "hooks": {
    "Stop": [
      { "hooks": [ {
        "type": "agent",
        "prompt": "Verify the unit tests pass. Run the suite and check results. $ARGUMENTS",
        "timeout": 180
      } ] }
    ]
  }
}
```

**Watch out:** a `Stop` hook that blocks eight times in a row gets overridden. Parse `stop_hook_active` from the input and exit early when it is true, or you build an infinite loop.

### Filtering precisely

`matcher` filters by tool name. The `if` field goes further, using permission-rule syntax to match tool **and arguments**, so your hook process only spawns when it is relevant:

```json
{
  "hooks": { "PreToolUse": [ {
    "matcher": "Bash",
    "hooks": [ {
      "type": "command",
      "if": "Bash(terraform apply *)",
      "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/require-change-ticket.sh"
    } ]
  } ] }
}
```

For MCP tools, match on the naming convention `mcp__<server>__<tool>`. So `mcp__github__.*` catches every GitHub MCP tool, and `mcp__.*__(write|create|delete).*` catches mutating calls across all servers. That regex is a good starting point for an "all MCP writes get logged" rule.

### Where hooks live

|Location|Scope|
|:--|:--|
|`~/.claude/settings.json`|all your projects|
|`.claude/settings.json`|this project, committed|
|`.claude/settings.local.json`|this project, gitignored|
|managed policy settings|organization-wide, admin only|
|plugin `hooks/hooks.json`|when the plugin is enabled|
|skill or agent frontmatter|while that component is active|

All matching hooks **merge and fire**, from every source. For `PreToolUse` decisions the most restrictive answer wins, in the order deny, defer, ask, allow.

## 4.7 MCP: connecting the outside world

MCP is the USB-C of agent tooling. One protocol, four transports, three scopes.

```
   MCP WIRING AND THE TRUST BOUNDARY

   ┌──────────────────── your machine ────────────────────┐
   │                                                      │
   │   Claude Code                                        │
   │     │                                                │
   │     ├─ stdio ──▶ local server process   (npx, uvx,   │
   │     │             runs as YOU, sees your fs,  docker)│
   │     │             your env vars, your network        │
   │     │                                                │
   └─────┼────────────────────────────────────────────────┘
         │
    ═════╪═══════════════ TRUST BOUNDARY ══════════════════
         │
         ├─ http / sse ──▶ remote server (OAuth sign-in)
         │                   vendor-hosted, e.g. Sentry, GitHub
         └─ ws ─────────▶ websocket server

   WHAT THE SERVER SENDS BACK IS UNTRUSTED INPUT:
     tool descriptions  ──┐
     tool schemas       ──┼──▶ all of it enters Claude's context
     tool results       ──┘     and can contain instructions

   SCOPES (which config file it lives in)
     local    ~/.claude.json for this project only   (just you)
     project  ./.mcp.json                            (committed, whole team)
     user     ~/.claude.json                          (all your projects)
   precedence: local > project > user
```

### Breakdown of the MCP diagram

**A stdio server is a local process running as you.** It inherits your environment, your filesystem access, and your network position. Installing an MCP server off a random GitHub repo is equivalent to running that repo's code on your laptop. It is a supply-chain decision, not a settings change.

**Everything a server sends back is untrusted input.** Tool descriptions and results land in the model's context. That is the mechanism behind "tool poisoning," where a description contains hidden instructions the model reads and the user never sees. Part 8 covers what to do about it.

**Project scope is how teams share, and how risk spreads.** A committed `.mcp.json` means everyone who clones the repo gets those servers. Great for onboarding. Also a route for one bad PR to add a server to everyone's setup. Review `.mcp.json` diffs like you review dependency changes.

### Setup and management

```bash
# add servers
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
claude mcp add --transport stdio postgres -- npx -y @some/postgres-mcp
claude mcp add --scope project github ...     # writes ./.mcp.json

claude mcp list                 # what is configured
/mcp                            # status + per-server token cost, in session
/mcp                            # also where you authenticate remote servers
```

In-session, `/mcp` shows connection status and token cost per server. That number is the one to watch: a chatty server with 60 tools is a permanent tax on every request.

### Features that matter at scale

- **MCP tool search** is on by default. Tool names load at startup; full schemas are fetched only when needed. That is why "I connected eight servers" is no longer instantly fatal to your context.
- **Environment variable expansion** in `.mcp.json` lets you commit a config without committing secrets.
- **Automatic reconnection** for remote servers that drop.
- **Long tool calls get backgrounded** automatically.
- **Resources and prompts.** Servers can expose data (`@server:resource`) and prompts that become commands.
- **Import from Claude Desktop**, so you do not configure twice.
- **Claude Code can itself be an MCP server**, which is how you plug it into other tools.
- **Managed MCP.** Admins can define the exclusive server set in `managed-mcp.json`, or run allowlists and denylists matching on URL, command, or name. This is your enterprise control point.
- **Scoping a server to one subagent** with the `mcpServers` frontmatter field. Inline definitions connect when the subagent starts and disconnect when it finishes, so the tools never enter your main context. This is both a context optimization and a least-privilege pattern.

## 4.8 Plugins and marketplaces: packaging

A plugin bundles skills, subagents, hooks, MCP servers, LSP servers, background monitors, themes, and default settings into one installable unit. A marketplace is a catalog, usually a Git repo with a manifest.

```bash
/plugin marketplace add anthropics/claude-plugins-official
/plugin install security-guidance@claude-plugins-official
/reload-plugins                    # apply without restarting

/plugin marketplace add github.com/your-org/internal-claude-plugins
/plugin list
```

**Why you care as a leader:** a plugin is the unit of standardization. Instead of a wiki page telling forty engineers to configure hooks, you ship one plugin from a private marketplace and require it through managed settings. Plugins support version constraints, dependencies, release channels per user group, and pinning.

**Security note built into the design:** plugin subagents cannot use the `hooks`, `mcpServers`, or `permissionMode` frontmatter fields. Those are ignored when loading from a plugin. Someone else's plugin cannot silently attach hooks or change permission modes. If you need those, the file has to live in your own `.claude/agents/`.

**Official plugins worth installing today:**

|Plugin|What it does|
|:--|:--|
|`security-guidance`|reviews Claude's own code for vulnerabilities as it writes, in-session|
|`claude-security`|multi-agent vulnerability scan of a repo or diff, produces reviewed patches|
|`skill-creator`|write, evaluate, and A/B test your own skills|
|code intelligence plugins|language server per language: symbol navigation, live type errors|

---

# Part 5: The `.claude` directory, mapped

Once you can read this tree, nothing about configuration is mysterious.

```
   USER LEVEL  (all your projects)
   ~/.claude/
   ├── settings.json              permissions, hooks, env, model, plugins
   ├── CLAUDE.md                  your personal always-on rules
   ├── skills/<name>/SKILL.md     your personal skills
   ├── agents/<name>.md           your personal subagents
   ├── rules/<name>.md            your personal path-scoped rules
   ├── commands/<name>.md         legacy custom commands (still work)
   ├── hooks/                     your hook scripts
   ├── agent-memory/<agent>/      persistent subagent memory (user scope)
   ├── plugins/                   installed plugins
   └── projects/<project>/<sess>/ transcripts
       └── subagents/agent-<id>.jsonl

   PROJECT LEVEL  (this repo)
   <repo>/
   ├── CLAUDE.md                  team rules, COMMITTED
   ├── CLAUDE.local.md            your personal overrides, gitignored
   ├── AGENTS.md                  cross-vendor convention, also read
   ├── .mcp.json                  MCP servers for the team, COMMITTED
   └── .claude/
       ├── settings.json          team settings + hooks, COMMITTED
       ├── settings.local.json    your overrides, gitignored
       ├── skills/<name>/SKILL.md project skills, COMMITTED
       ├── agents/<name>.md       project subagents, COMMITTED
       ├── rules/<name>.md        path-scoped rules, COMMITTED
       ├── hooks/*.sh             hook scripts, COMMITTED
       ├── agent-memory/          subagent memory (project scope), COMMITTED
       ├── agent-memory-local/    subagent memory (local scope), gitignored
       ├── claude-security-guidance.md   threat model for the review plugin
       ├── security-patterns.yaml        custom per-edit regex rules
       └── worktrees/             agent isolation workspaces (transient)

   MANAGED LEVEL  (pushed by your admin, users cannot override)
   platform-specific managed settings directory
   ├── managed-settings.json
   ├── managed-mcp.json
   ├── CLAUDE.md
   ├── skills/
   └── agents/
```

### Breakdown of the directory map

**The committed / gitignored split is the whole game.** Anything committed is team policy and gets code review. Anything `.local.` is personal and invisible to teammates. When you standardize a team, you are deciding what belongs in the committed column.

**Precedence differs by mechanism, and this trips everyone up.**

- CLAUDE.md files are **additive**. Every level contributes at once.
- Skills **override by name**: managed beats user beats project.
- Subagents **override by name**: managed beats CLI flag beats project beats user beats plugin.
- MCP servers **override by name**: local beats project beats user.
- Hooks **merge**. Every registered hook fires.

Read that list again. Skills and subagents resolve in _different orders_, and hooks never override at all. When behavior surprises you, `/doctor` and `/debug-your-config` show what actually loaded.

**Transcripts are on disk in plaintext.** `~/.claude/projects/...` holds your conversation history, including whatever code and data passed through. Cleanup follows the `cleanupPeriodDays` setting, default 30 days. For a security team, that is a data-at-rest question worth answering before you roll out widely.

---

# Part 6: Workflows you can copy

## 6.1 The core developer loop

This is the loop that separates people who get good results from people who get slop.

```
   THE EXPLORE → PLAN → CODE → VERIFY LOOP

   ┌─────────────────────────────────────────────────────────────────┐
   │ 1. EXPLORE          (read-only, cheap, isolated)                │
   │    "Use Explore subagents to find every place we validate JWTs. │
   │     Do not write anything yet."                                 │
   │    → noise stays in the subagent. You get a map.                │
   └───────────────────────────┬─────────────────────────────────────┘
                               ▼
   ┌─────────────────────────────────────────────────────────────────┐
   │ 2. PLAN             (plan mode, still read-only)                │
   │    Shift+Tab to plan mode, or --permission-mode plan            │
   │    "Plan the change. List files you will touch and why."        │
   │    → you review a plan BEFORE any edit exists                   │
   │    → for big work: /ultraplan (review the plan in a browser)    │
   └───────────────────────────┬─────────────────────────────────────┘
                               ▼
   ┌─────────────────────────────────────────────────────────────────┐
   │ 3. CODE             (acceptEdits or auto mode)                  │
   │    Small steps. Interrupt early when it drifts.                 │
   │    Hooks auto-format and auto-scan behind it.                   │
   └───────────────────────────┬─────────────────────────────────────┘
                               ▼
   ┌─────────────────────────────────────────────────────────────────┐
   │ 4. VERIFY           (the step everyone skips)                   │
   │    /verify   → builds and RUNS the app, not just the tests      │
   │    Stop hook → tests must pass before the turn can end          │
   │    /code-review or /security-review → adversarial second pass   │
   └───────────────────────────┬─────────────────────────────────────┘
                               ▼
              good?  ──yes──▶ commit / PR
                │
                no
                ▼
        /rewind  (restore files)  or steer and loop
```

### Breakdown of the developer loop

**Step 1 exists to protect your context window.** Codebase exploration is the single noisiest activity in agentic coding. Pushing it into an Explore subagent means you get the answer without the 40 file reads.

**Step 2 exists because reviewing a plan is 10x cheaper than reviewing a diff.** Plan mode is read-only, so the worst outcome is a bad plan you reject. `/ultraplan` sends the plan to a browser where you can revise it, then execute in the cloud or send it back to your terminal.

**Step 3's rule is "interrupt early."** Anthropic's own best-practices page says to course-correct early and often. The instinct to let it finish and then fix everything is expensive and produces worse code. Hit Escape the moment it goes sideways.

**Step 4 is the difference between a demo and engineering.** Give Claude something it can check itself against: a test suite, a type checker, a running app, a linter. An agent with a verification loop converges. An agent without one confidently declares victory. `/verify` specifically builds and runs the app rather than falling back to tests.

**Two mistakes worth naming.** First, letting one session run for hours: context degrades, compaction loses detail, quality drops. Clear and restart with a fresh, specific prompt instead. Second, being vague. "Fix the auth bug" produces guessing. "The `/refresh` endpoint returns 200 with an expired token; see `auth/refresh.ts:84`; here is the failing test" produces a fix.

## 6.2 Workflow: reviewing a pipeline against a standard

A recipe you can adapt to any standard you own.

```
   step 1  Write the standard as a SKILL, not a prompt.
           ~/.claude/skills/pipeline-review/SKILL.md
           + reference.md holding the full control list

   step 2  Make it a subagent so it gets a clean context and
           read-only tools.
           .claude/agents/pipeline-reviewer.md
             tools: Read, Grep, Glob
             skills: [pipeline-review]
             memory: project

   step 3  Give it live input via dynamic injection in the skill:
             !`ls -1 .github/workflows/`
             !`git log --oneline -5 -- .github/workflows/`

   step 4  Run it, in the foreground, on one repo:
             @"pipeline-reviewer (agent)" review .github/workflows/

   step 5  Measure it with skill-creator. Build 10 test cases:
           5 configs that SHOULD fail, 5 that should PASS.
           Check pass rate with and without the skill.

   step 6  Scale it. Pick one:
           a) headless across many repos:
              for r in $(cat repos.txt); do
                (cd "$r" && claude -p "/pipeline-review" \
                   --output-format json >> ../findings.jsonl)
              done
           b) a Routine on a GitHub trigger, so every workflow-file
              PR gets reviewed automatically
           c) a plugin published to your internal marketplace so
              every team gets the same reviewer
```

### Breakdown

**Steps 1 and 2 are the pattern to internalize:** knowledge goes in a skill, execution shape goes in an agent file. Keeping them separate means you can reuse the same standard in a subagent, in a hook, in a routine, and in a plugin without rewriting it.

**Step 5 is the step almost nobody does.** Without a with/without comparison you have no idea whether your skill is helping or whether the model would have found those issues anyway. That is the difference between having a tool and having a measured control.

**Step 6a is the underrated one.** `claude -p` with `--output-format json` turns the agent into a Unix citizen. You can fan out across a hundred repos with a bash loop and get structured findings out the other end.

## 6.3 Workflow: safe autonomy for a long unattended run

When you want it to work for an hour without you.

```
   LAYER THE CONTROLS (not just "trust the prompt")

   1. ISOLATE      run in a worktree or a container
                   --worktree  or  isolation: worktree
                   or sandbox-runtime / devcontainer

   2. RESTRICT     permissions.deny is your hard floor
                   deny: Read(./.env), Bash(curl *), WebFetch,
                         Bash(git push *), Agent(some-agent)

   3. GATE         PreToolUse hooks for the rules that must hold
                   (deny works even in bypassPermissions mode)

   4. SANDBOX      OS-level filesystem + network isolation
                   so "cannot" replaces "should not"

   5. BOUND        maxTurns on subagents, --max-turns headless,
                   a Stop hook that checks completion

   6. OBSERVE      OTel metrics + events to your SIEM
                   PostToolUse hook logging every Bash command
                   statusline showing cost and context live

   7. RECOVER      checkpoints (/rewind) + git + patches never
                   auto-applied
```

### Breakdown

**Read this as defense in depth, because that is exactly what it is.** Layer 2 (permissions) is your policy. Layer 3 (hooks) is your policy engine that survives a mode change. Layer 4 (sandboxing) is the OS saying no regardless of what the model decided. Layer 6 is your detection. Layer 7 is your recovery.

**The ordering matters.** People start at layer 5 ("I set maxTurns") and skip layers 1 through 4. Isolation and restriction do far more work than bounding turns.

**Auto mode deserves a specific note.** `auto` mode replaces you with a background classifier that reviews commands and protected-directory writes. It is genuinely useful for long runs, and its behavior is configurable: you define trusted infrastructure, override block and allow rules, and review denials. You can also route _all_ shell commands through the classifier. Learn `auto-mode-config` before you turn it loose on anything that matters.

## 6.4 Workflow: onboarding onto an unfamiliar codebase

Fifteen minutes to orientation.

```bash
claude --permission-mode plan          # read-only, so nothing can break

# then, in order:
"Give me a tour of this repo: entry points, main modules, how a request
 flows end to end. Use Explore subagents. Cite file:line."

"Where does this app talk to the outside world? Every HTTP client,
 every database call, every shell exec, every deserialization point.
 Table it."

"What are the trust boundaries? Where does user input enter and where
 does it reach a sink without validation?"

"Now write a CLAUDE.md for this repo based on what you found: build
 commands, test commands, structure, and the conventions you observed."
```

Then `/init` if you want Claude's own version, and diff the two.

**Why plan mode:** you are exploring code you do not know, possibly cloned from somewhere. Read-only is the right default for that. If the repo is genuinely untrusted, sandbox the whole session first, because a repo's committed `.claude/` settings, hooks, and CLAUDE.md apply in your session just like any other.

---

# Part 7: How developers actually use it

## 7.1 The patterns that show up in every good setup

**1. A short, sharp CLAUDE.md.** Under 200 lines. Commands, conventions, structure, hard "nevers." Everything else moved to skills and rules.

**2. Verification wired in.** A test command Claude knows about, a linter on a `PostToolUse` hook, and often a `Stop` hook gate.

**3. Explore-first as a reflex.** Research through subagents, implementation in the main session.

**4. Plan mode for anything non-trivial.** Read-only, review, then execute.

**5. Skills instead of prompt libraries.** The third time you paste the same instructions, it becomes a skill. That is the trigger.

**6. Parallelism through worktrees.** Multiple sessions on isolated copies of the repo, plus agent view (`claude agents`) to dispatch and monitor them from one place.

**7. Aggressive context management.** `/context` to see what is eating the window. `/clear` between unrelated tasks. Subagents for anything verbose. Compaction treated as a cost, not a feature.

**8. Code intelligence plugins on typed languages.** A language server means symbol lookup instead of reading whole files, plus live type errors after each edit. It usually _reduces_ total context.

**9. Headless mode for anything repeated.** `claude -p` in scripts, Makefiles, git hooks, and CI.

**10. An adversarial second pass.** A separate review step with a fresh context, because the model that wrote the code is the worst reviewer of it. `/code-review`, `/security-review`, or a custom reviewer subagent.

## 7.2 Anti-patterns

|Anti-pattern|Why it hurts|Do this instead|
|:--|:--|:--|
|One giant CLAUDE.md with everything|rent every turn, and the important rules get diluted|200 lines max, push the rest to skills and `paths`-scoped rules|
|Marathon sessions|context degrades, compaction drops details|`/clear` and restart with a specific prompt|
|Vague prompts|it guesses, you review guesses|name files, paste errors, state the expected behavior|
|Trusting instructions for safety-critical rules|a prompt is a request|hooks and deny rules|
|Connecting every MCP server you find|permanent context tax plus supply-chain risk|connect what you use, scope servers to subagents, check `/mcp` cost|
|Skipping verification|confident wrong answers|tests, `/verify`, a Stop hook|
|Letting it write and review its own work in one context|no independence|separate reviewer with fresh context|
|Auto-approving everything to "go faster"|one bad tool call and you are restoring from git|`plan` then `acceptEdits`, or `auto` mode with configured boundaries|
|Never measuring skills|you cannot tell if your setup helps|`skill-creator` evals, with and without|

## 7.3 The "which model" question

- **Haiku** for mechanical work: renames, formatting, simple extraction, cheap Explore agents.
- **Sonnet** as the default workhorse. Long context, fast, good at most coding.
- **Opus** for architecture, gnarly debugging, security reasoning, anything where being right matters more than being fast.
- **Fable** for the hardest frontier work, where available on your plan.
- `opusplan` as a model setting is a nice hybrid: plan with the heavier model, execute with the cheaper one.
- **Per-subagent model control** is the real trick. Run your session on Opus and your Explore agent on Haiku with `model: haiku` in its file. You pay premium prices only for the thinking that needs it.
- **Effort level** is orthogonal to model choice. `low` on a Sonnet session for mechanical work, `xhigh` when you need it to actually think. The `advisor` feature can pair a second model in as a consultant on hard calls.
- **Fast mode** trades cost for latency. Worth knowing, not worth defaulting to.

---

# Part 8: How security professionals use Claude

This part assumes you are the person who has to answer for the blast radius, not just use the tool.

## 8.1 The mental model shift

```
   ┌──────────────────────────────────────────────────────────────┐
   │  PROMPTS AND SKILLS = GUIDANCE                               │
   │  "Do not touch production." "Never log PII."                 │
   │  Probabilistic. Usually works. Not a control.                │
   ├──────────────────────────────────────────────────────────────┤
   │  PERMISSION RULES = POLICY                                   │
   │  deny: Read(./.env), Bash(kubectl * --context=prod *)        │
   │  Deterministic. Deny always wins, even over hooks.           │
   ├──────────────────────────────────────────────────────────────┤
   │  HOOKS = POLICY ENGINE                                       │
   │  PreToolUse fires in EVERY permission mode, including        │
   │  bypassPermissions. Users cannot mode-switch around it.      │
   ├──────────────────────────────────────────────────────────────┤
   │  SANDBOXING = ENFORCEMENT                                    │
   │  OS-level filesystem and network isolation.                  │
   │  "Cannot" instead of "should not."                           │
   ├──────────────────────────────────────────────────────────────┤
   │  MANAGED SETTINGS = GOVERNANCE                               │
   │  Admin-pushed config users cannot override.                  │
   └──────────────────────────────────────────────────────────────┘
              stronger as you go down. use all five.
```

### Breakdown

You already know this ladder from other domains. It is the same as "secure coding guidelines" versus "a linter rule" versus "a CI gate" versus "network policy." The mistake specific to AI tooling is that the top layer feels so convincing that people stop there. A model that follows your instruction 98% of the time is a great assistant and a terrible control.

**The one fact to carry out of this section:** `PreToolUse` hooks fire before any permission-mode check, in every mode, and a hook that returns deny blocks the tool even under `--dangerously-skip-permissions`. Hooks can tighten but never loosen. That makes hooks your enforcement point for anything that must always hold.

## 8.2 The native security stack, layered

Anthropic ships four review layers plus your existing tools.

```
   WHEN CODE GETS CHECKED, EARLIEST TO LATEST

   as Claude types
   ├─ security-guidance plugin  ── per-edit regex match (no model call, free)
   │                            ── end-of-turn diff review (separate model,
   │                               fresh context, background)
   │                            ── commit/push review (agentic, reads
   │                               surrounding code to cut false positives)
   │
   on demand, one pass
   ├─ /security-review          ── single security pass on the branch
   │
   on demand, deep
   ├─ claude-security plugin    ── multi-agent scan: maps architecture,
   │                               threat models, hunts, then INDEPENDENT
   │                               verifier agents confirm each finding.
   │                               Writes reviewed patches. Never auto-applies.
   │
   at PR time
   ├─ Code Review               ── multi-agent PR review  [TEAM/ENTERPRISE]
   │
   managed
   ├─ Claude Security product   ── hosted, monitors repos  [ENTERPRISE]
   │
   in CI
   └─ your existing SAST/SCA/secrets/IaC scanners
```

### Breakdown of the security stack

**`security-guidance` is the highest-value install for a hobbyist and a director alike.** It is three layers in one plugin, and its design answers the obvious objection. The per-edit layer is a deterministic string match with no model involved. The turn and commit reviews run as a **separate Claude call with fresh context and a security-focused prompt**, so the reviewer has no investment in the original approach. It does not ask the model that wrote the code to grade itself.

You extend it two ways, both additive:

```markdown
<!-- .claude/claude-security-guidance.md : guidance for the model reviews -->
# Security guidance for this repo
- Do not log `customer_id` or `account_number` at INFO or above.
- All routes under /admin must call require_role("admin") before any DB read.
- Use crypto.timingSafeEqual for token comparison, never ===.
```

```yaml
# .claude/security-patterns.yaml : deterministic per-edit rules
patterns:
  - rule_name: internal_api_key
    substrings: ["sk_live_", "AKIA"]
    reminder: "Hardcoded credential prefix. Load from the secret manager."
  - rule_name: tenant_unfiltered_query
    regex: "\\.objects\\.all\\(\\)"
    paths: ["src/tenants/**"]
    reminder: "Multi-tenant code must filter by org_id."
```

**Important limits, stated in the docs and worth repeating to your stakeholders:** none of these layers block writes or commits. Findings reach the writing Claude as instructions. The commit-review layer only fires on commits _Claude_ makes through its Bash tool, not commits you make in your own shell. The review model can miss things. Treat all of it as one layer of defense in depth.

**`claude-security` is the one that will surprise you.** A team of agents maps your architecture, builds a threat model, hunts, and then independent verifier agents confirm each finding before it reaches the report. Findings land as `CLAUDE-SECURITY-<timestamp>/` with a markdown report, a JSONL machine-readable version, and a **revision stamp** recording which commit was scanned, at what effort, and how thoroughly it was verified. That stamp is an audit artifact. Two honest caveats from the docs: scans are nondeterministic, so two runs on the same code can surface different findings, and it needs a paid plan plus dynamic workflows.

**Where the plan gap bites.** PR-time Code Review is Team and Enterprise only. On Max you get the in-session and on-demand layers, and you cover the PR gate with GitHub Actions or GitLab CI running `claude -p`. That is a real workaround, not a downgrade, but it is your work to build.

## 8.3 Ten things security people actually build with this

**1. Finding triage that checks reachability.** See the `appsec-triage` agent in Part 4.5. The value is not "AI reads SARIF." It is an agent that traces from entry point to sink in the real code, cites file and line, and says NEEDS HUMAN when it cannot prove it. Add `memory: project` and it stops re-learning your framework's protections every week.

**2. Threat modeling on real code instead of a whiteboard.**

```
"Build a threat model for this service from the code, not from docs.
 1. Map every trust boundary: network ingress, auth boundaries,
    tenant boundaries, deserialization, shell exec, file paths from input.
 2. For each, list what an attacker controls and what they reach.
 3. Apply STRIDE per boundary, with file:line evidence.
 4. Rank by exploitability given the controls you can actually see.
 5. Output a mermaid diagram of data flow plus a findings table.
 Use Explore subagents in parallel per module. Cite everything."
```

**3. Pipeline and IaC config review as a repeatable control.** Part 6.2. This is the single best fit for a skill, because a standard is exactly "reference material plus a checklist."

**4. Guardrail hooks as compensating controls.** A `PreToolUse` hook that blocks reads of credential paths, blocks `curl` to non-allowlisted hosts, blocks writes under `infra/prod/`, and requires a change ticket for `terraform apply`. Deployed as a plugin from your internal marketplace and required through managed settings, that is a control you can point an auditor at.

**5. Agent activity logging into your SIEM.** Two routes, use both. Claude Code emits OpenTelemetry metrics and events covering sessions, tool decisions, permission mode changes, MCP server connections, skill activations, hook executions, API errors, and refusals. And a `PostToolUse` hook of type `http` can POST every tool call to your own collector. The docs include a "map security questions to events" section built for exactly this.

**6. Detection engineering and rule work.** Give it your log schema as a skill and have it write and test detections. Pair with an MCP server for your SIEM so it can validate against real data instead of guessing at field names.

**7. Incident response assistance.** A read-only subagent with tools limited to `Read`, `Grep`, `Glob`, and specific MCP tools, pointed at logs and code, answering "when was this introduced, what else calls it, what is the blast radius." `Bash(git log *)` and `Bash(git blame *)` earn their place here.

**8. Security review of AI-generated code at volume.** This is the new job. Your developers are shipping more code than before, generated by tools with confident tone. `security-guidance` plus a PR gate plus sampling human review is the shape most teams land on.

**9. Reviewing MCP servers before you allow them.** Treat each one as a dependency intake: who publishes it, what transport, what credentials it wants, what its tool descriptions actually say, whether it runs locally as you. Scan tool metadata for injection patterns before approving.

**10. Standardizing all of the above as a plugin.** One internal plugin containing your guardrail hooks, your review skills, your triage subagent, and your approved MCP allowlist. Version it, channel it, require it.

## 8.4 The threat model of the tool itself

You will be asked about this, so have the answer ready.

```
   ATTACK SURFACE OF AN AGENTIC CODING SETUP

   [1] PROMPT INJECTION VIA CONTENT
       source: web pages, issue text, PR comments, dependency
               READMEs, log files, an MCP tool's result
       effect: instructions the model reads and you never see
       native mitigations: subagent output scanning, WebFetch
               domain safety checks, permission prompts on
               consequential actions
       your job: least privilege, deny rules, sandboxing,
               human gates on anything irreversible

   [2] TOOL POISONING (MCP-specific)
       source: a malicious or compromised MCP server's TOOL
               DESCRIPTION or SCHEMA, which enters context
               invisibly to the user
       effect: exfiltration, unauthorized actions
       your job: pin and review descriptions, allowlist servers
               via managed-mcp.json, scan tool metadata, alert
               on description changes between sessions

   [3] SUPPLY CHAIN
       source: MCP servers, plugins, marketplaces, skills from
               the internet, a repo's committed .claude/
       effect: arbitrary code as you, on your machine
       your job: private marketplace, pinned versions, review
               .mcp.json and .claude/ diffs like dependency PRs

   [4] EXCESSIVE AGENCY
       source: broad tool grants, bypassPermissions habits,
               unattended routines with write access
       effect: the agent does something irreversible, correctly
               following a bad instruction
       your job: deny rules, worktrees, no auto-apply of patches,
               PR-only writes for automation

   [5] SECRET EXPOSURE
       source: env vars visible to stdio MCP servers and hooks,
               secrets in transcripts on disk, secrets pasted
               into context
       effect: credential leak through a channel nobody logged
       your job: masked env vars in sandbox config, deny reads of
               credential paths, the credential proxy pattern,
               transcript retention policy

   [6] TRUST BOUNDARY CONFUSION
       source: a repo you cloned brings its own CLAUDE.md, hooks,
               and .mcp.json into YOUR session
       effect: someone else's config running with your permissions
       your job: workspace trust prompts are not optional theater;
               sandbox untrusted repos before scanning them
```

### Breakdown of the threat model

**Row 1 is the one with no complete fix.** Prompt injection is unsolved industry-wide. Anthropic's own security docs treat it as a layered problem: core protections plus privacy safeguards plus your restrictions. Design as if the model will eventually be tricked, and make sure the tricked model still cannot do the thing you care about.

**Row 2 is where the MCP ecosystem is genuinely immature.** Public tracking through 2026 found a steady stream of CVEs across MCP servers, clients, and tooling, weighted heavily toward command injection and path traversal, including in reference implementations. OWASP now maintains an **MCP Top 10** alongside its LLM and Agentic AI lists, the Cloud Security Alliance has published agentic MCP guidance, and NSA published MCP security design considerations in May 2026. If you are writing internal guidance, those are your citations.

**Row 6 is the one people miss.** Cloning a repo and opening Claude Code in it means that repo's committed configuration is now part of your session. The `claude-security` docs say this outright: the plugin adds no isolation of its own, and a repo's committed `.claude/` settings, hooks, and `CLAUDE.md` apply exactly as they would in any other session. To scan something you do not trust, sandbox the session first. Anthropic points at `sandbox-runtime` for OS-level filesystem and network restrictions.

## 8.5 Sandboxing, concretely

Options from lightest to heaviest:

|Option|What it isolates|When|
|:--|:--|:--|
|Sandboxed Bash tool|shell commands|default hardening, low friction|
|Sandbox runtime|filesystem + network at the OS level, with env var masking|untrusted repos, long autonomous runs|
|Dev container|whole environment|team standardization, reproducibility|
|Custom container / gVisor|whole environment, stronger boundary|hostile input, multi-tenant|
|Virtual machine|everything|highest assurance|
|Claude Code on the web|Anthropic-managed isolated VM, network allowlist|you want isolation without operating it|

Sandboxing and permissions are **different mechanisms**. Permissions ask. Sandboxing prevents. Configure both, and use managed settings to keep developers from widening the policy.

---

# Part 9: The non-native ecosystem, and why people use it

You asked the right question: not just "what exists" but "why do people reach past the native tools." Here is the honest map.

## 9.1 Where the ecosystem sits

```
                       THE AGENTIC TOOLING LANDSCAPE

   ┌── LAYER 1: MODELS ───────────────────────────────────────────────┐
   │  Anthropic (Claude) · OpenAI · Google · Meta · Mistral ·         │
   │  Chinese open-weight labs (Qwen, GLM, Kimi, DeepSeek) ·          │
   │  self-hosted open weights                                       │
   └──────────────────────────────┬──────────────────────────────────┘
                                  │
   ┌── LAYER 2: HARNESSES (the agent wrapper) ────────────────────────┐
   │  NATIVE: Claude Code (CLI/Desktop/IDE/web/Slack), Cowork         │
   │  RIVALS: OpenAI Codex CLI + app, Google's Gemini/Antigravity     │
   │          line, Cursor, GitHub Copilot agents, Devin              │
   │  OPEN:   OpenCode, Cline, Aider, Goose, Continue, Kilo Code,     │
   │          Crush, Plandex, Roo Code, Forge, Qwen Code              │
   └──────────────────────────────┬──────────────────────────────────┘
                                  │
   ┌── LAYER 3: ORCHESTRATORS (run many agents at once) ──────────────┐
   │  NATIVE: agent view (`claude agents`), agent teams, worktrees,   │
   │          workflows, Routines                                     │
   │  THIRD:  Conductor, Vibe Kanban, Claude Squad, Crystal,          │
   │          Emdash, Superset, Sculptor (containers), Nimbalyst      │
   └──────────────────────────────┬──────────────────────────────────┘
                                  │
   ┌── LAYER 4: CONTEXT + TOOL PLUMBING ─────────────────────────────┐
   │  MCP servers (thousands) · MCP registries and catalogs ·         │
   │  MCP gateways and proxies · MCP inspectors and scanners          │
   └──────────────────────────────┬──────────────────────────────────┘
                                  │
   ┌── LAYER 5: BUILD-YOUR-OWN FRAMEWORKS ───────────────────────────┐
   │  NATIVE: Claude Agent SDK (TS/Python)                            │
   │  THIRD:  LangGraph, CrewAI, AutoGen, OpenAI Agents SDK,          │
   │          Pydantic AI, Mastra, Temporal (durable execution)       │
   └──────────────────────────────┬──────────────────────────────────┘
                                  │
   ┌── LAYER 6: EVALS + OBSERVABILITY + GUARDRAILS ──────────────────┐
   │  NATIVE: OTel export, /usage, analytics (Team/Ent), skill evals  │
   │  THIRD:  Langfuse, LangSmith, Braintrust, Helicone, OpenLLMetry, │
   │          promptfoo, DeepEval, garak, Inspect                     │
   └─────────────────────────────────────────────────────────────────┘
```

### Breakdown of the landscape

**Layer 2 is where the loudest competition is,** and where the model-versus-harness confusion lives. When you read "tool X scores 83% on Terminal-Bench," that number belongs to a model _inside that harness_. Swap the harness and the number moves several points. Never compare a benchmark number across tools without checking which model produced it.

**Layer 3 barely existed in 2025 and exploded in 2026,** then got partially absorbed. Claude Code now ships native agent view, agent teams, workflows, and worktree isolation. Several third-party orchestrators were solving a problem the platform has since solved. Check the native feature before installing a wrapper.

**Layer 4 is the layer your security program has the least visibility into,** which is precisely why it matters most to you.

**Layer 5 is a different job entirely.** The Agent SDK gives you Claude's loop. LangGraph and friends give you a graph engine where you control every node and can mix providers. Choose by whether you want a great agent or a controllable pipeline.

## 9.2 Why people reach past the native tools

Nine honest reasons, roughly in order of how often they come up:

**1. Model choice and vendor independence.** OpenCode, Cline, Aider, and Continue are provider-agnostic by design. Point them at Claude, GPT, Gemini, or a self-hosted open-weight model. Teams with data-residency requirements or a policy against single-vendor dependence pick this on purpose.

**2. Running the same task on two models and diffing the results.** No native tool does cross-vendor A/B. Orchestrators like Conductor and Superset run Claude Code and Codex side by side on the same task so you can compare diffs. For a genuinely hard bug, that is a real technique.

**3. Cost shape.** Subscriptions are flat with a hard ceiling. API keys are variable with no ceiling. Open harnesses on a cheap open-weight model can be dramatically cheaper for mechanical work. Some teams route trivial tasks to a cheap model and reserve the expensive plan for the hard stuff.

**4. Container isolation, not just worktrees.** Native isolation is worktree-based plus sandboxing options. Some third-party tools (Sculptor, for example) put each agent in a container by default. If your threat model wants a stronger boundary out of the box, that is the pitch.

**5. A project-management surface.** Kanban boards, cards, assignment, review columns. Vibe Kanban and similar exist because engineers wanted agent work to look like a board rather than a list of terminal sessions. Note that Vibe Kanban's commercial backing wound down in April 2026 and it continues as community-maintained open source, which is exactly the kind of fact to check before you standardize on any of these.

**6. Editor preference.** Cursor's pitch is that the agent and the editor share one loop. If someone works best in an AI-native IDE, forcing them into a terminal costs more than it saves. Note the billing consequence: using Claude models inside Cursor runs on Cursor's plan or your API key, not your Claude subscription.

**7. Filling native gaps by plan tier.** If you are on Max and want PR-time review, you either build it in CI or buy a third-party review product. Same for cross-team analytics.

**8. Specialist depth in one domain.** A dedicated SAST vendor's MCP server knows its own findings model better than a general agent will. Same for SIEM, CSPM, and API security vendors.

**9. Not being locked to one roadmap.** Open harnesses trade polish for the freedom to move. That is a strategic bet, not a feature comparison.

## 9.3 Why you might not need to

Counter-arguments, stated fairly:

- **The extension layer is unusually deep.** Hooks with five types and roughly thirty events, agent files with twenty frontmatter fields, path-scoped rules, per-subagent MCP scoping, plugin marketplaces with release channels. Most third-party harnesses do not have an equivalent.
- **The native orchestration caught up.** Agent view, agent teams, workflows, worktree isolation, background sessions, Routines.
- **Every surface shares one config.** CLAUDE.md, skills, agents, hooks, plugins work across CLI, Desktop, IDE, web, and Slack. Multi-tool setups mean multiple config formats to maintain.
- **The OAuth rule changes the math.** Your Max plan legitimately covers Claude Code on any machine, local or over SSH. It does not cover third-party harnesses. Going non-native means an API bill on top of your subscription, or an entirely different provider.
- **Governance exists here.** Managed settings, managed MCP allowlists, sandbox enforcement, OTel telemetry, audit logging. Most third-party tools have thinner enterprise stories.

**The pragmatic answer most heavy users land on:** run Claude Code as the primary harness, keep one alternative installed for model comparison and for the rare task where a different provider is better, and be deliberate about which bill each one hits.

## 9.4 MCP infrastructure and security tooling

This is the table to actually act on.

|Category|What it is|Why it exists|
|:--|:--|:--|
|**Registries / catalogs**|searchable directories of MCP servers|discovery, and a first look at what a server claims to do|
|**Gateways / proxies**|one endpoint fronting many servers, with auth, logging, and policy|the enterprise control plane MCP does not have natively (Microsoft published on this exact pattern in April 2026)|
|**Inspectors**|interactive debugging of a server's tools|development, and reading tool descriptions before you trust them. Note the reference inspector itself had a serious CVE, so patch it|
|**Scanners**|scan a configured MCP setup for poisoning patterns and known issues|`mcp-scan` from Invariant Labs is the well-known one. It analyzes tool metadata, not your files or credentials|
|**Guardrail layers**|policy enforcement around agent tool calls|because prompt-only safety instructions measurably fail a meaningful share of the time|

**Security vendor MCP servers, as of mid-2026.** Official servers exist from a long list of vendors across SAST, SCA, IaC, secrets, containers, SBOM, threat intel, SIEM, and cloud posture. Public reviews of the category note that vendor investment has been strong, that CLI-integrated servers (embedding MCP into an existing CLI) are winning over standalone API-wrapper servers, and that coverage is uneven: DAST is thin, runtime protection is largely absent, and a few major enterprise AppSec vendors still have no MCP presence.

**How I would evaluate one, as a checklist:**

```
   MCP SERVER INTAKE CHECKLIST

   [ ] Publisher: official vendor, or a community repo? Pinned version?
   [ ] Transport: stdio (runs as you, locally) or remote (OAuth)?
   [ ] Credentials: what does it want, and what is the minimum scope?
   [ ] Tool descriptions: read every one. Anything instruction-shaped?
       Any <IMPORTANT> blocks, any "before using this tool, read..."?
   [ ] Tool count: how much context does it cost? Check /mcp.
   [ ] Write capability: which tools mutate? Should those require approval?
   [ ] Network egress: where does it call out to?
   [ ] Where does it belong: main session, or scoped to ONE subagent?
   [ ] Rollout: local scope for you, or project scope for everyone?
   [ ] Governance: does it belong on the managed allowlist or denylist?
   [ ] Monitoring: is its usage visible in your OTel events?
```

Two native controls to pair with that checklist: mark a specific tool as requiring approval, and use `managed-mcp.json` or `allowedMcpServers` / `deniedMcpServers` policies so the allowlist is not a suggestion.

## 9.5 Evals and observability

Native gives you `/usage`, OTel metrics and events, analytics on Team and Enterprise, and skill-level evals through `skill-creator`. What people add on top:

- **Tracing platforms** (Langfuse, LangSmith, Braintrust, Helicone, OpenLLMetry) for per-run traces, cost attribution, and datasets. The Agent SDK's observability docs cover exporting traces and tagging them, so this integrates rather than replaces.
- **Eval harnesses** (promptfoo, DeepEval, Inspect) for scoring prompts and agents against test sets, including red-team style cases. promptfoo in particular is used for LLM security testing, not just quality.
- **LLM red-teaming tools** (garak and similar) for probing a model or agent with adversarial inputs. Relevant if you ship anything customer-facing.

**If you build the AI threat-modeling and remediation tools you have been planning, this layer is not optional.** An agent that "fixes vulnerabilities" needs a measured pass rate on a fixed test set, or you have built a demo. Start with `skill-creator` for skill-level evals and add promptfoo or DeepEval when you need a real regression suite.

---

# Part 10: Governance, for the person who owns the risk

The levers, in the order you would deploy them.

|Lever|What it controls|Where|
|:--|:--|:--|
|**Managed settings**|any setting, unoverridable by users|platform managed settings directory|
|**Server-managed settings**|settings delivered from the server|Team and Enterprise only|
|**Managed MCP**|exclusive server set, or allow/deny by URL, command, or name|`managed-mcp.json`|
|**Permission rules**|allow / ask / deny per tool and argument pattern|settings at any scope|
|**Managed CLAUDE.md**|org-wide always-on rules|managed settings dir|
|**Managed skills and agents**|org-wide capabilities that win over user versions|managed settings dir|
|**Required plugins**|force your internal plugin on|`enabledPlugins` in managed settings|
|**Marketplace restrictions**|only your approved marketplaces|managed settings|
|**Model restrictions**|`availableModels` allowlist, org default, effort caps|settings|
|**Sandbox enforcement**|require sandboxing, prevent widening|managed settings|
|**`disableAllHooks`, `disableSkillShellExecution`**|turn off the code-execution paths in skills and hooks|settings, best in managed|
|**Excluded files**|keep specific paths out of Claude's reach|settings|
|**OTel export**|metrics, events, traces to your stack|env vars or settings|
|**Analytics + Compliance API**|usage, PR attribution, conversation export|Team / Enterprise|
|**ZDR**|no retention|Enterprise, needs enabling, disables some features|

### Three governance decisions to make early

**1. Where do developers get their configuration?** If the answer is "everyone writes their own," you have forty different setups and no baseline. Ship an internal plugin with your hooks, review skills, and approved MCP list, and require it. That single decision is worth more than any policy document.

**2. What is on the hard deny list, organization-wide?** Start small and defensible: credential file reads, production infrastructure writes, network egress to non-allowlisted hosts, and `git push` to protected branches. Deny rules beat hooks beat prompts, so put the non-negotiables in deny.

**3. What does your audit trail look like?** The monitoring docs include a section specifically on auditing security events, attributing actions to users, auditing MCP activity, and shipping events to a SIEM. Wire that up before you scale usage, not after your first incident.

### The data questions you will be asked

- **Where do transcripts live?** Locally in `~/.claude/projects/`, in plaintext, cleaned up per `cleanupPeriodDays` (default 30).
- **What is the training policy?** Covered in the data usage docs, including the Development Partner Program and what `/feedback` sends.
- **What runs in Anthropic's cloud versus locally?** The data usage page has separate data-flow sections for local Claude Code and cloud execution.
- **What does ZDR actually cover?** It has explicit scope, and it disables some features and limits model availability. Read the page before promising it.
- **Healthcare?** A BAA extends to Claude Code automatically if you have one executed and ZDR active, per-organization.

---

# Part 11: Cost and context economics

Two budgets: your **context window** (per session) and your **usage limit** (per week). They interact, because wasted context means more turns, and more turns means more usage.

### Reduce token usage, in order of impact

1. **Delegate verbose work to subagents.** Test runs, log parsing, doc fetching, big searches. The biggest single win.
2. **Move instructions out of CLAUDE.md into skills.** CLAUDE.md is rent every turn. Skills are on demand.
3. **Manage context deliberately.** `/context` to see what is expensive. `/clear` between unrelated tasks. Do not let one session sprawl.
4. **Right-size the model per job.** Haiku for mechanical work, Explore agents on a cheaper model, Opus only where it earns it.
5. **Install code intelligence plugins for typed languages.** Symbol lookup replaces whole-file reads. Often net negative on context.
6. **Trim MCP overhead.** Disconnect servers you are not using. Scope servers to subagents. Watch `/mcp` cost.
7. **Offload work to hooks.** A hook that lints costs zero context. Asking Claude to lint costs a tool call and its output.
8. **Tune effort down for mechanical work.** Not everything needs `xhigh`.
9. **Write specific prompts.** Vague prompts cause exploratory loops.
10. **Understand prompt caching.** Editing files keeps the cache. Switching models, changing effort, toggling fast mode, connecting or disconnecting an MCP server, enabling a plugin, and compacting all **invalidate** it. Model-hopping mid-session is more expensive than it looks. Forks share the parent's cache, which is why they are cheaper than fresh subagents.

### Watch it live

Build a status line showing context percentage, cost, and rate-limit usage:

```
/statusline show context window percentage, session cost, current git
branch, and how much of my rate limit is left
```

Claude writes the script for you. Having the number in front of you changes behavior more than any advice in this section.

---

# Part 12: A 30/60/90 path

No exit gates. Just an order of operations, since you asked for a top-level view.

### Days 1 to 30: fluency on one surface

```
   [ ] Install the CLI. Run /doctor, /status, /context, /usage.
   [ ] Write CLAUDE.md for one real repo. Keep it under 200 lines.
   [ ] Do the explore → plan → code → verify loop five times.
       Use plan mode every time. Notice what changes.
   [ ] Write your first skill. Something you have pasted three times.
   [ ] Write your first subagent. Read-only, restricted tools.
   [ ] Write your first hook. Start with a Notification hook so you
       get a desktop alert when Claude needs you.
   [ ] Connect one MCP server. Look at its token cost in /mcp.
   [ ] Install security-guidance. Watch it catch something.
   [ ] Read code.claude.com/docs/en/best-practices once, properly.
```

### Days 31 to 60: depth and control

```
   [ ] A PreToolUse hook that blocks something real (credential reads).
       Verify it still blocks under --dangerously-skip-permissions.
   [ ] Path-scoped rules in .claude/rules/ for one file type.
   [ ] A subagent with memory: project. Use it weekly. Read what it
       learned after two weeks.
   [ ] Parallel work: two worktrees, or `claude agents` to dispatch
       and monitor background sessions.
   [ ] Headless: claude -p in a shell loop across several repos,
       with --output-format json.
   [ ] Evaluate a skill with skill-creator. Build 10 test cases.
       Compare with and without. Write down the numbers.
   [ ] Try Cowork on a real non-code deliverable. Notice where the
       account-level skill sync matters.
   [ ] Turn on OTel export to something, even a local collector.
   [ ] Install one non-native harness (OpenCode or Cline) with an API
       key. Run the same task in both. Write down what differed.
```

### Days 61 to 90: build and govern

```
   [ ] Package your hooks + skills + agents into a plugin.
       Publish it to a private marketplace repo. Install it clean
       on a second machine.
   [ ] A Routine: GitHub-triggered review on pipeline-config PRs.
   [ ] A sandboxed session against a repo you do not trust.
   [ ] Write your MCP intake checklist as an internal standard.
       Run three servers through it.
   [ ] Build one Agent SDK tool with an API key. Read the secure
       deployment guide first. Add cost tracking and tracing.
   [ ] Draft the governance answer: managed settings baseline, hard
       deny list, audit trail, transcript retention.
   [ ] Write the "how we use AI agents safely" one-pager for your org.
       You now have the material.
```

---

# Part 13: Troubleshooting

|Symptom|Most likely cause|Fix|
|:--|:--|:--|
|Skill never triggers|description does not match how you phrase requests|rewrite the description with real user words; check `/skills`; run `--debug` for YAML parse errors|
|Skill triggers too often|description too broad|narrow it, or set `disable-model-invocation: true`|
|Skill descriptions look truncated|too many skills, listing budget overflowed|`/doctor`; set low-value skills to `name-only` in `skillOverrides`; raise `skillListingBudgetFraction`|
|Claude ignores CLAUDE.md|file too long, rules diluted, or lost after compaction|shorten it; move detail to skills; re-inject with a `SessionStart` hook matching `compact`|
|New subagent not found|the `agents` directory did not exist when the session started|restart the session|
|Two agents with the same name, wrong one runs|duplicate `name` in the same directory|`/doctor` reports duplicates; rename|
|Hook never fires|wrong event, case-sensitive matcher, invalid JSON|`/hooks`; test by piping sample JSON into the script; `chmod +x`|
|Hook JSON validation fails|your shell profile prints something|wrap profile echoes in an interactive-shell check|
|Stop hook loops forever|not checking `stop_hook_active`|exit 0 when it is true|
|Context fills constantly|verbose work in the main session|subagents; `/clear`; check `/context`|
|Skill not found in a Routine|routines run remotely and do not read `~/.claude/skills/`|enable it for your account, commit it to the repo, or ship it in a plugin|
|API charges while on Max|`ANTHROPIC_API_KEY` set in your environment|unset it; confirm with `/status`|
|MCP server shows failed|transport, auth, or a Windows path issue|`/mcp` for status; `claude mcp list`; check the troubleshooting section|
|Everything feels worse than last week|version change, model change, or config drift|`/doctor`; `--debug`; check the weekly changelog; test against a clean config|

**The two commands to reach for first:** `/doctor` (setup checkup) and `/debug-your-config` (what actually loaded into context, resolved settings, MCP servers, hooks). Most "Claude is being weird" reports are configuration surprises, and those two show you the truth.

---

# Sources and how to keep this current

Primary sources, all checked on July 26, 2026:

|Topic|Where|
|:--|:--|
|Claude Code docs index|`code.claude.com/docs/en/claude_code_docs_map.md`|
|Extension layer comparison and context costs|`code.claude.com/docs/en/features-overview`|
|Subagents and agent files|`code.claude.com/docs/en/sub-agents`|
|Skills|`code.claude.com/docs/en/skills`|
|Hooks guide and reference|`code.claude.com/docs/en/hooks-guide`, `/hooks`|
|MCP|`code.claude.com/docs/en/mcp`, `/managed-mcp`|
|Plan and provider availability|`code.claude.com/docs/en/feature-availability`|
|Security plugins|`code.claude.com/docs/en/security-guidance`, `/claude-security`|
|Sandboxing|`code.claude.com/docs/en/sandboxing`, `/sandbox-environments`|
|Telemetry and audit|`code.claude.com/docs/en/monitoring-usage`|
|Auth policy|`code.claude.com/docs/en/legal-and-compliance`|
|Agent SDK|`code.claude.com/docs/en/agent-sdk/overview`, `/secure-deployment`|
|Cowork|`claude.com/docs/cowork/overview`|
|Max plan and usage|`support.claude.com` (Max plan, usage limit best practices)|
|Weekly changelog|`code.claude.com/docs/en/whats-new/`|

**Keep it current with one habit.** Once a week, in a Claude Code session:

```
Fetch this week's page from https://code.claude.com/docs/en/whats-new/
and tell me only what changed in hooks, subagents, skills, MCP,
permissions, or sandboxing. Skip everything else.
```

**Two honest caveats about this document.**

1. Anything about usage numbers, plan contents, prices, or model names is a snapshot. Anthropic changes caps and features at its discretion, and this space moved several times in the first half of 2026 alone.
2. The third-party tooling in Part 9 moves faster than the native platform. Names, funding, licenses, and maintenance status all shift. Verify anything before you standardize a team on it.

---

# Appendix: structural review of this document

You asked for a review pass looking for gaps, inconsistencies, and improvements. Here is that review, honestly.

## What holds up

- **The layering is consistent.** Every part uses the same spine: surfaces sit on top of an extension layer that sits on top of a blast radius. Parts 0, 4, 8, and 10 all reinforce that instead of contradicting it.
- **One idea carries the whole document.** "Guidance versus enforcement" appears in Part 1 (vocabulary), Part 4.1 (the chooser), Part 4.6 (hooks), Part 6.3 (safe autonomy), and Part 8.1 (the ladder). That repetition is deliberate, because it is the concept that separates a user from an expert.
- **Every diagram has a breakdown**, as requested, and the breakdowns add the "why" rather than restating the picture.
- **Plan reality is stated plainly.** The Max feature table names what Max does _not_ get, which most write-ups skip.
- **Sources are named** so anything here can be checked or refuted.

## Gaps I know are in here

1. **Cowork gets one section and probably deserves three.** It is a genuine second product with its own workflow patterns for non-coders, and this document treats it as a surface. If your team has non-engineers, that is a follow-up document, not a section.
2. **Claude Tag, Claude in Excel, Claude in PowerPoint, and Claude Design appear only in the chooser table.** They are real surfaces with real workflows. They were out of scope for a workflow-and-agents guide, but "complete" is a stretch while they are one-liners.
3. **No worked end-to-end plugin example.** Part 4.8 explains plugins and Part 12 tells you to build one, but there is no full walkthrough with a manifest, a marketplace file, and an install. That is the single most useful thing to add next.
4. **No `workflows` deep dive.** Dynamic workflows are named in Part 9 and used by `claude-security`, but the feature (bundled workflows, having Claude write one, saving and distributing it) never gets its own treatment. It is a meaningful gap for automation specifically.
5. **Agent teams are described but not demonstrated.** No concrete example of a team configuration or a use case walked end to end. Defensible, since the feature is experimental and off by default, but it is a gap.
6. **Cost is qualitative, not quantitative.** Because Anthropic does not publish token quotas, Part 11 gives levers rather than numbers. That is accurate and also less actionable than a budget model would be.
7. **Detection engineering and IR get one bullet each** in Part 8.3, while AppSec gets most of the depth. Reasonable given your role, but a SOC reader would find Part 8 lopsided.
8. **Non-native tooling names are unverified individually.** Part 9 draws on third-party roundups that disagree with each other on version numbers and benchmark scores. I deliberately kept it qualitative and named the categories rather than ranking products, but treat every product name as "check before you commit."

## Inconsistencies I found and fixed while writing

- Early drafts used "agent" for three different things (subagent, agent file, agent team). Part 1 now defines all three separately, and later parts use the specific term.
- The precedence rules genuinely differ by mechanism, and an early draft implied one universal order. Part 5 now states each mechanism's order explicitly and flags that they differ, because that mismatch is a real source of confusion.
- "Skill" and "custom command" were used loosely. Part 4.4 now states that commands merged into skills and that `.claude/commands/` still works.
- The automation ladder in Part 3.6 and the layered-controls list in Part 6.3 were nearly duplicates. They now serve different purposes: one is "how autonomous," the other is "how controlled."

## Inconsistencies that remain by choice

- **Deliberate repetition.** The guidance-versus-enforcement point, the context-cost point, and the OAuth point each appear two or three times. In a reference document people jump into, repeating the load-bearing facts at each entry point beats sending readers backwards.
- **Two different "workflow" meanings.** Part 6 uses "workflow" in the plain-English sense. Claude Code also has a feature literally named Workflows. Part 9 mentions the feature. That collision is Anthropic's, and pretending otherwise would be more confusing than naming it. If this document gets a v2, the feature should get its own section titled "Workflows (the feature)."

## What I would add in a v2, ranked

1. A complete plugin walkthrough: manifest, marketplace, private repo, managed-settings enforcement, install verification.
2. A "Workflows (the feature)" section, since it is the missing rung between hooks and Routines.
3. A copy-paste starter kit: a real CLAUDE.md, three skills, two agent files, and a `settings.json` with a deny list and three hooks, all in one place.
4. A one-page printable of the decision tree from Part 4.1 plus the security ladder from Part 8.1.
5. An eval appendix: how to build a 20-case test set for a security skill and what a good pass rate looks like.
6. A Cowork companion document for non-engineers.
7. A cost model with placeholders you fill from your own `/usage` data over two weeks, since absolute numbers are unpublished but _your own_ burn rate is measurable.

## Reading-level check

Aimed at roughly ninth grade. Sentences are short, jargon is defined at first use, and the tone stays conversational. Two places run more technical than the rest by necessity: Part 4.6 (hook JSON and exit codes) and Part 9.4 (MCP intake). Both are reference material where precision matters more than smoothness, and both are preceded by a plain-language explanation of what the mechanism is for.