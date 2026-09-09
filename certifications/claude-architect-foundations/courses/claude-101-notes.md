# Claude 101 — Notes

Part of: Claude Certified Architect — Foundations
Owner: agregorio1@kpmg.com.au · Started: 2026-07-20

---

## Lesson overview

_What this lesson covers, at a glance._

- The **4D Framework for AI Fluency** — four core competencies for effective AI collaboration.
- **Evaluating Claude for your workflows** — using lightweight evals to build discernment about where Claude adds value.
- **Claude's three main modes** — Chat, Cowork, and Code, and when to reach for each.
- **Projects vs Skills** — the split between stored knowledge Claude *references* and defined processes Claude *executes*.
- **Researching with Claude** — agentic multi-search research with extended thinking and citations.
- **Sample use cases** — canonical examples across general professional use, Sales, Marketing, Finance, HR, Legal, and Research.
- **Claude product surfaces** — the full lineup (Claude.ai, Claude Code, Cowork, @Claude, Design, Microsoft 365, Chrome) and where each runs.

## Key concepts

_Core ideas, definitions, and mental models._

### The 4D Framework for AI Fluency

Developed through research collaboration between Professor Rick Dakan (Ringling College of Art and Design) and Professor Joseph Feller (University College Cork). Four core competencies that, when combined, help you make the most of AI interactions:

- **Delegation** — Deciding what work should be done by humans, what by AI, and how to distribute tasks between them. Includes understanding your goals, AI capabilities, and making strategic choices about collaboration.
- **Description** — Effectively communicating with AI systems. Includes clearly defining outputs, guiding AI processes, and specifying desired AI behaviors and interactions.
- **Discernment** — Thoughtfully and critically evaluating AI outputs, processes, behaviors and interactions. Includes assessing quality, accuracy, appropriateness, and determining areas for improvement.
- **Diligence** — Using AI responsibly and ethically. Includes making thoughtful choices about AI systems and interactions, maintaining transparency, and taking accountability for AI-assisted work.

### Evaluating Claude for your workflows

Discernment in practice. **Evals** are systematic ways to test how well Claude performs on the specific tasks that matter to you, and to develop intuition for assessing its outputs.

**Why evals matter** — your work is unique. Claude might excel at drafting marketing copy but need more guidance for technical documentation in your domain. Simple evals help you:

- Understand where Claude adds the most value in your workflow.
- Identify tasks where you'll need to provide more context or examples.
- Build confidence in Claude's outputs for recurring tasks.

**A simple eval approach** (no complex infrastructure required):

1. **Gather examples** — collect 5–10 examples of a task you do regularly (emails, reports, analyses).
2. **Create test prompts** — write prompts that would generate similar outputs, including the context you'd naturally have.
3. **Compare outputs** — run your prompts and compare Claude's responses to your examples. Ask:
   - Does Claude capture the key information?
   - Is the tone and style appropriate?
   - What's missing or could be improved?
4. **Refine your approach** — adjust prompts, add examples to show Claude what "good" looks like, or identify where human review is essential.

### Claude's three main modes

|  | **Chat** | **Cowork** | **Code** |
|---|---|---|---|
| **Optimized for** | Quicker exchanges: exploring ideas, iterative drafting, quick answers, learning through dialogue | Complex or sustained work: research, analysis, file organization, producing finished documents and deliverables | Building software: writing, testing, running and deploying code |
| **Key features** | Quick entry, dictation | Work from local folders, plugins, subagents, scheduled tasks | Ask/Code/Plan modes, visual diffs, git integration, local and remote environments |
| **Tools and extensions** | Connectors, Skills, Claude in Chrome | Connectors (local and remote), Skills, Claude in Chrome, Plugins, Computer Use | Connectors, Skills, Claude in Chrome, Plugins, Hooks |

### Projects vs Skills

Two different mechanisms for extending Claude — and they solve different problems. Projects store the **knowledge** Claude draws on; Skills define the **processes** Claude runs.

|  | **Projects** | **Skills** |
|---|---|---|
| **Purpose** | Store knowledge Claude references | Define processes Claude executes |
| **Best for** | Long-term context, reference materials, team collaboration | Repeatable workflows, multi-step tasks, consistent methodology |
| **Example** | Customer hub, research buddy, feedback generator | Process guidelines (like brand or legal), blog drafting, PDF creation |
| **Persistence** | Knowledge available across all chats in the project | Instructions applied when the skill is invoked |

Mental model: **Projects = library. Skills = playbooks.** A project is where a team drops the docs, transcripts, and reference material that any chat inside it can draw on. A skill is a set of instructions the model consults *when it recognises the task*.

### Researching with Claude

Research is Claude's **agentic** investigation mode — not a single search but a chain of searches that build on each other, with Claude deciding what to look at next.

- **Agentic, not one-shot.** Instead of one query, Claude conducts multiple searches that build on each other, exploring different angles of the question automatically and working through open questions systematically.
- **Timeframe.** Most reports finish in **5–15 minutes**; complex investigations can take up to **45 minutes** — work that would normally take hours manually.
- **Extended thinking is automatic.** Research always runs with extended thinking on, so Claude both *plans its approach* and *gathers comprehensive information*, breaking complex requests into manageable pieces.
- **Citations built in.** Every finding is delivered with easy-to-check citations, so you can verify sources yourself rather than trusting the model blindly.

Why it matters: research is the first feature where Claude visibly *iterates on its own investigation*. It's a canonical example of **Discernment + Delegation** from the 4D Framework — you delegate the search process, but the citations let you exercise discernment on the output.

### Claude product surfaces

Broader than the three *modes* (Chat/Cowork/Code) — these are the actual **products / surfaces** where Claude shows up. The modes describe *how you interact*; these describe *where you interact*.

| Tool | Best for | Where it runs |
|---|---|---|
| **Claude.ai** | General tasks, research, writing, analysis, file creation | Web, desktop, and mobile apps |
| **Claude Code** | Software development, codebase navigation, git workflows | Terminal/command line, IDE, or your browser |
| **Claude Cowork** | Complex, multi-step tasks: research briefs, document creation, file organization, data analysis | Desktop (and mobile apps via Dispatch) |
| **@Claude** | Team collaboration, meeting prep, quick answers in context | Slack workspace |
| **Claude Design** | UI prototypes, design exploration, design-system-aware mockups | Web |
| **Claude for Microsoft 365** | Editing in place and carrying context across documents | Excel, PowerPoint, Word, and Outlook sidebars |
| **Claude in Chrome** | Web research, email management, browser automation | Chrome browser sidebar |

Mental model: **modes** (Chat/Cowork/Code) are the *interaction styles*; **surfaces** are the *deployment targets*. Multiple surfaces can host the same mode — e.g. Cowork mode runs on desktop and (via Dispatch) mobile; Chat mode is available in Claude.ai, @Claude, and the Chrome sidebar.

## Terminology

| Term | Meaning |
|---|---|
| 4D Framework | Dakan/Feller model of four AI fluency competencies: Delegation, Description, Discernment, Diligence. |
| Delegation | Deciding what work is done by humans vs. AI, and how to split tasks. |
| Description | Communicating clearly with AI — defining outputs, guiding process, specifying desired behaviour. |
| Discernment | Critically evaluating AI outputs, processes, behaviours, and interactions for quality and appropriateness. |
| Diligence | Using AI responsibly and ethically — transparency, thoughtful choices, accountability for AI-assisted work. |
| Eval | A systematic test of Claude's performance on a specific task, used to build intuition and confidence. |
| Chat mode | Claude mode optimized for quick exchanges, exploration, and iterative dialogue. |
| Cowork mode | Claude mode optimized for sustained, complex work — research, analysis, file-based deliverables. Supports local folders, plugins, subagents, scheduled tasks. |
| Code mode | Claude mode for building software — writing, testing, running, deploying. Includes Ask/Code/Plan modes, visual diffs, git integration. |
| Connectors | Extensions that link Claude to external tools/data. Available across Chat, Cowork, Code. |
| Skills | Reusable capabilities available across all three modes. |
| Plugins | Extensions available in Cowork and Code. |
| Hooks | Extension mechanism specific to Code mode. |
| Computer Use | Capability available in Cowork mode for operating computer interfaces. |
| Project | A container that stores knowledge (files, context, reference material) available across all chats inside it. Best for long-term context and team collaboration. |
| Skill | A defined, invocable process — a repeatable workflow or methodology Claude executes when the task matches. Best for multi-step, consistent work. |
| Research | Agentic investigation feature — Claude runs multiple chained searches, plans with extended thinking, and returns cited reports in 5–45 minutes. |
| Extended thinking | Mode where Claude plans and reasons at length before/while acting. Automatically on during Research. |
| Citations | Source references attached to research findings so the user can verify claims directly. |
| Claude.ai | Primary web/desktop/mobile Claude product for general tasks, research, writing, analysis, file creation. |
| Claude Code | Developer-focused surface for software work — runs in terminal, IDE, or browser. |
| Claude Cowork | Desktop (and mobile via Dispatch) surface for complex multi-step work: research briefs, document creation, file organization, data analysis. |
| @Claude | Slack integration for team collaboration, meeting prep, quick in-context answers. |
| Claude Design | Web surface for UI prototypes, design exploration, design-system-aware mockups. |
| Claude for Microsoft 365 | Sidebar integration in Excel, PowerPoint, Word, Outlook — edit in place, carry context across documents. |
| Claude in Chrome | Chrome browser sidebar for web research, email management, browser automation. |
| Dispatch | Delivery mechanism that brings Cowork to mobile apps. |

## Examples / demos

_Anything worked through in the lesson._

### Sample use cases by function

**General professional use** — applies across many roles and industries:

- **Generate project status reports** — keep stakeholders informed with clear, consistent updates.
- **Analyze patterns in user feedback** — extract insights from customer comments and survey responses.
- **Package your brand guidelines in a skill** — create a reusable Claude skill that applies your brand standards.

**Sales** — accelerate deal prep, create compelling materials, stay on top of competitive intelligence:

- **Build a battle card library** — competitive intelligence resources that help the team win deals.
- **Prepare for sales deals** — research prospects and organize talking points before important meetings.
- **Create sales reports** — turn pipeline data into clear, actionable reports.

**Marketing** — analyze performance data, efficiently repurpose content across channels:

- **Analyze campaign performance** — extract insights from campaign metrics to inform strategy.
- **Adapt content across platforms** — efficiently repurpose content for different channels and audiences.

**Finance** — build models, draft documents, make sense of complex spreadsheets:

- **Build financial models** — create and refine financial projections with Claude's help.
- **Draft investment memos** — structure and write investment analyses more efficiently.
- **Understand and extend an inherited spreadsheet** — decode complex spreadsheets and add new functionality.

**HR** — better onboarding experiences and documentation:

- **Create new hire onboarding guides** — comprehensive materials tailored to different roles.

**Legal** — track complex timelines, manage discovery:

- **Track discovery timelines and analyze patterns** — organize case timelines and identify key patterns in legal documents.

**Research** — plan literature reviews, verify data analysis:

- **Plan your literature review** — organize your approach to reviewing academic sources.
- **Verify statistics from raw data** — double-check calculations and statistical analyses.

## Questions / to revisit

_Things unclear, or worth drilling before the exam._

-

## Exam-relevant takeaways

_Points flagged as likely to appear on the exam → move to `exam-prep/` later._

- Memorise the four D's and be able to give a one-line definition of each.
- Discernment ↔ Evals: evals are the practical mechanism through which discernment is exercised.
- The simple eval loop: **Gather → Prompt → Compare → Refine**.
- Know the three modes (Chat / Cowork / Code) and what each is optimized for — likely a "which mode would you use for X?" style question.
- Note which tools/extensions are available in which mode (e.g. Hooks → Code only; Computer Use → Cowork only; Connectors/Skills/Claude in Chrome → all three).
- **Projects vs Skills** — know the split: Projects = knowledge Claude *references* (persistent across chats), Skills = processes Claude *executes* (invoked when the task matches). Likely a "which would you use for X?" question.
- **Research feature** — remember it's *agentic* (multi-step, self-directed searches), runs **5–15 min typical / up to 45 min complex**, has **extended thinking automatically on**, and always returns **citations**. Likely a "which of these is true about Research?" style question.
- **Modes vs surfaces** — don't confuse them. Modes = Chat/Cowork/Code (interaction style). Surfaces = Claude.ai, Claude Code, Cowork, @Claude, Design, M365, Chrome (where it runs). Expect "which surface for X task?" questions — e.g. Slack → @Claude, Excel → M365, git workflows → Claude Code, UI prototypes → Claude Design.
