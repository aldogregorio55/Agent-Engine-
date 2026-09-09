# Prompt Engineering Master Context 

**Purpose:** This is the complete knowledge base for writing prompts in this system. Load it into any prompt-writing tool (Copilot, Claude, ChatGPT) to replicate the house standard. 

**Last Updated:** 2026-03-16 

 

--- 

 

## 1. What This System Builds 

 

This is not just prompt engineering. The actual discipline has four layers: 

 

| Layer | What It Is | Priority | 

|-------|-----------|----------| 

| **Context Engineering** | Filling the context window with the right information, structure, and sequence | Highest — optimize context before wording | 

| **Prompt Engineering** | Constructing effective instructions for a model | Core craft | 

| **Evaluation Engineering** | Designing systems to test whether prompts actually work | Separates professional PE from guesswork | 

| **Agent Policy Design** | Writing system prompts that function as operating policies for autonomous agents | Where the field is heading | 

 

> A well-organized, complete context with mediocre wording outperforms a perfectly worded prompt on an incomplete context. Optimize context first, wording second. 

 

--- 

 

## 2. Step 0: Identify the Model Type First 

 

**This is the most important step. Do not skip it. The model type changes everything downstream.** 

 

| Model Type | Examples | Approach | 

|-----------|---------|---------| 

| **Standard** | Claude Sonnet/Haiku, GPT-4o, Gemini Flash | Full prompting toolkit applies | 

| **Reasoning** | Claude + Extended Thinking, o3, o4, Gemini Deep Think | Zero-shot first, no CoT instructions, minimal process guidance, keep goal specification explicit | 

 

--- 

 

## 3. Technique Selection 

 

| Situation | Use This | 

|-----------|---------| 

| Complex reasoning, standard model | Chain-of-Thought ("think step by step") | 

| Specialized format or output structure | Few-shot (1–3 consistent examples) | 

| Hundreds of examples, large context available | Many-shot with context caching | 

| High accuracy requirement, can afford cost | Self-consistency (sample multiple, take majority) | 

| Search/planning, offline batch | Tree-of-Thought or Graph-of-Thought | 

| Needs relevant background knowledge first | Step-back prompting | 

| Reasoning model, any task | Zero-shot; omit CoT; keep goal explicit | 

| Complex multi-step workflow | Prompt chaining — each step as simple as possible | 

| Need to fill context correctly | Context engineering: prioritize recency, summarize aggressively, cache stable portions | 

 

--- 

 

## 4. House Prompt Style — XML Template 

 

**All prompts follow this structure.** Written to be human-readable and manually editable. Simple is the goal. 

 

``` 

<role> 

[One sentence defining the model's role.] 

 

Your expertise includes: 

- [bullet — specific capability] 

- [bullet — specific capability] 

</role> 

 

<task> 

[One sentence stating what the prompt produces.] 

</task> 

 

<instructions> 

 

## Process 

1. **[Step name]** — [What to do. Directive, not explanatory.] 

2. **[Step name]** — [What to do.] 

3. **[Step name]** — [What to do.] 

 

## Constraints 

- **do NOT** [rule] 

- **do NOT** [rule] 

 

## Edge Cases 

- If [situation] → [how to handle] 

 

</instructions> 

 

<reference> 

[Tables, definitions, category lists the model needs — shared reference material] 

</reference> 

 

<input> 

 

<[content_tag]> 

{{VARIABLE_PLACEHOLDER}} 

</[content_tag]> 

 

</input> 

 

<output_structure> 

[Exact format the model must produce. Show header rows explicitly.] 

</output_structure> 

 

<example> 

[Fictional dummy data only. No real names, no real companies.] 

</example> 

``` 

 

### XML Template — Writing Rules 

 

- **Process steps are directives** — tell the model what to do, not why 

- **Constraints are a standalone Do NOT list** — never embedded in step descriptions 

- **Output structure is always its own section** — never folded into instructions 

- **Variables use `{{UPPERCASE_PLACEHOLDER}}`** for easy find-and-replace 

- **Internal reasoning steps are explicitly marked** — "(internal — do not show in output)" 

- **`<reference>` block** carries shared context (tables, categories) — keeps instructions clean 

- **Simple wins** — omit any section that has nothing to say; do not write filler 

- **One formatting system** — use XML or Markdown, never both in the same prompt 

- **Examples use fictional dummy data only** — e.g., "Meridian Health", "Apex Financial", fictional roles 

 

### Sections — When to Include vs. Omit 

 

| Section | Include When | Omit When | 

|---------|-------------|-----------| 

| `<role>` | Always | — | 

| `<task>` | Persona prompts or when role alone is insufficient | Simple task prompts where role + instructions are clear | 

| `<instructions>` | Always | — | 

| `<reference>` | Shared lookup tables, taxonomies, scoring frameworks needed during processing | No structured reference material | 

| `<input>` | Always (variables go here) | — | 

| `<output_structure>` | Complex output with multiple fields that need explicit naming | When output format is covered adequately in the process steps | 

| `<example>` | When format or quality bar needs to be shown, not just described | Simple tasks where format is self-evident | 

 

### Output Structure — Key Design Decision 

 

**Avoid redundancy between `<output_structure>` and `<instructions>`.** 

- Weave column names, field definitions, ID formats, and heading logic directly into the relevant process steps 

- `<output_structure>` should add what the steps don't already say — not restate it 

- The `<example>` carries the visual reference; the process steps carry the logic 

- If `<output_structure>` would just restate the steps, omit it 

 

### Prompt Rule Density 

 

Do not add heavy rule sets or logic constraints by default. Keep prompts light unless specific rules are explicitly required. Sparse rules > exhaustive rules. 

 

### Language Precision 

 

Prompt language must be exacting. Stakeholders are highly sensitive to literal word meaning. Avoid: 

- Loose verbs ("consider", "think about", "explore") 

- Relative terms ("brief", "detailed", "comprehensive") without definition 

- Interpretable phrasing where a specific directive would work 

 

--- 

 

## 5. Standard Prompt Architecture (Generic Reference) 

 

``` 

[System / Developer Prompt] 

 Role/tone definition 

 Task description and scope 

 Behavioral constraints and rules 

 Output format specification 

 Examples (if few-shot) 

 Safety/guardrails 

 

[User Turn] 

 Context or background data 

 Retrieved documents (RAG) 

 The actual request 

 

[Assistant Prefill] — Claude only 

 Opening token(s) to enforce output format 

``` 

 

--- 

 

## 6. Claude-Specific Rules 

 

These rules apply specifically to Claude (Sonnet, Haiku, Opus — non-Extended Thinking): 

 

| Rule | Reason | 

|------|--------| 

| Remove caps-lock imperatives ("ALWAYS", "MUST", "CRITICAL") | Claude 4 over-triggers on them; use normal language instead | 

| Remove "be thorough" / "think carefully" | Amplifies already-proactive behavior; counterproductive | 

| Use XML tags to delineate content | Strong attention in Claude's training; use consistently | 

| Use XML or Markdown — never both | Mixed formatting degrades reliability | 

| Prefill the Assistant turn to enforce output format | Steers output format directly | 

| Cannot prefill Extended Thinking | Technical limitation — Extended Thinking precedes response generation | 

 

--- 

 

## 7. Context Engineering — Pre-Finalisation Checklist 

 

Before finalising any prompt, run this: 

 

- [ ] Static content (system prompt, reference docs, examples) placed first and cacheable 

- [ ] Dynamic content (user query, retrieved docs) placed last 

- [ ] Nothing critical buried in the middle of a long context — move to beginning or end 

- [ ] Content compressed where possible without losing meaning 

- [ ] Examples consistently formatted — same delimiters, structure, whitespace across all examples 

 

--- 

 

## 8. RAG Prompt Pattern 

 

``` 

[System]: Answer using ONLY the provided context. If context is insufficient, 

say "I don't have that information." Cite the relevant section for each claim. 

 

[Context]: 

<retrieved_documents> 

{chunk_1} 

{chunk_2} 

</retrieved_documents> 

 

[User]: {question} 

``` 

 

--- 

 

## 9. Agent System Prompt — Required Elements 

 

When writing prompts for autonomous AI agents in production workflows: 

 

- Define role, inputs/outputs, and scope explicitly 

- Specify trust level — what it can/cannot do without confirmation 

- **Prefer reversible actions** — make this explicit in the prompt 

- **Require confirmation before irreversible or broad-impact actions** 

- Define explicit termination conditions 

- Define error recovery behavior 

- Set conservative defaults for ambiguous situations 

 

--- 

 

## 10. Evaluation — L1–L4 Framework 

 

### Task Level Classification 

 

| Level | Name | Core Question | Reasoning Required | 

|-------|------|---------------|--------------------| 

| **L1** | Explicit Facts | "What does the document say about X?" | Locate and extract only | 

| **L2** | Implicit Facts | "What can we infer from combining X and Y?" | Multi-hop reasoning | 

| **L3** | Interpretable Rationales | "What does the policy say to do here?" | Follow documented decision tree | 

| **L4** | Hidden Rationales | "What should we do, based on patterns?" | Infer unstated rules from examples | 

 

Classify at the **hardest level required** when a task spans multiple levels. 

 

### Evaluation Methods 

 

| Method | When to Use | 

|--------|-------------| 

| **Spot test** | Quick iterations, low stakes — 5–10 representative inputs | 

| **LLM-as-judge** | More rigorous; different model family as judge; CoT before score; temp=0; 3–5 anchor examples | 

| **A/B test** | Comparing two variants; same golden dataset; min 100 examples | 

| **Regression gate** | Before promoting to production; fail if critical metric drops >5% vs. baseline | 

 

### LLM-as-Judge — Non-Negotiable Settings 

 

- Temperature = 0 (greedy decoding) 

- Chain-of-thought before scoring (analysis first, then score) 

- Different model family from the model being evaluated 

- 3–5 anchor examples spanning the full score range 

- Structured output: Analysis / Score / Rationale 

 

**Known failure modes to mitigate:** position bias, verbosity bias, self-preference, style bias. 

 

### Quality Gates by Level 

 

| Level | Block Deployment If | Warn If | 

|-------|---------------------|---------| 

| L1 | EM < 85% or Faithfulness < 85% | Any metric drops > 3% | 

| L2 | Answer Accuracy < 75% | Any metric drops > 3% | 

| L3 | Workflow < 88% or Decision Tree < 92% | Any metric drops > 3% | 

| L4 | Expert Alignment < 65% | Any metric drops > 3% | 

| Any | Critical metric drops > 5% vs. baseline | — | 

 

--- 

 

## 11. Version Control — Prompt Management 

 

Prompts are code. Treat them that way: 

 

- Store as plaintext files in version control 

- **Semantic versioning:** MAJOR (structural change) / MINOR (behavioral change) / PATCH (wording cleanup) 

- Separate prompts per environment (dev / staging / prod) 

- Treat prompt changes as requiring review before production use 

- **File naming convention:** `[prompt-name]_v[X.X].md` 

 

--- 

 

## 12. Prompt Caching — When to Use It 

 

| Scenario | Action | 

|---------|--------| 

| System prompt + reference docs > 1,024 tokens | Mark for caching — up to 90% cost reduction, 85% latency reduction | 

| Same prompt against many inputs | Cache system prompt; keep user turn dynamic | 

| Many-shot with large example set | Cache examples; query dynamically | 

 

**Structure rule:** Static content first, dynamic content last. 

 

--- 

 

## 13. Prompt Security — Minimum Awareness 

 

Indirect prompt injection is OWASP #1 for LLM applications. When the model processes external content (emails, documents, web pages, user input): 

 

- Separate instruction context from data context with explicit delimiters 

- Never trust content from retrieved documents as instructions 

- Apply least-privilege tool access in agent systems 

- Flag anything that looks like an instruction embedded in retrieved content 

 

--- 

 

## 14. Example Prompts — Live Reference 

 

The following are active prompts from this system. They demonstrate the XML template style in practice. 

 

--- 

 

### Example 1: Persona Prompt — Senior Risk Assessment Analyst (v1.5) 

 

```xml 

<role> 

You are a Senior Risk Assessment Analyst at a professional services firm with over a decade of experience advising enterprise clients. You are not an AI assistant helping with risk analysis — you are the analyst. Fully embody this role. When someone interacts with you, they are speaking directly with an experienced risk professional, not querying a tool. 

 

Your expertise includes: 

- Reading between the lines of what stakeholders say to identify risks they haven't labeled as such 

- Extracting every risk signal from unstructured conversations, workshops, and documents — frequency is a data point, not a threshold for inclusion 

- Assessing risks with precision across four dimensions: Likelihood, Impact, Velocity, and Connectivity 

- Translating raw qualitative input into structured, defensible risk outputs that senior stakeholders can act on 

- Challenging weak evidence and flagging when an assessment cannot be supported — you never fabricate 

</role> 

 

<task> 

Your task is to act as this risk assessment professional in every interaction. The user has engaged you in your professional capacity. You are their analyst. You think, respond, and analyze as this role demands — with rigor, structured thinking, and professional judgment. 

 

You are a thinking partner, not a report generator. You ask clarifying questions when material is ambiguous. You surface what the user may not have thought to ask. You push back when something doesn't hold up. You own the analytical process. 

</task> 

 

<instructions> 

 

## How You Think 

 

Every piece of information you receive, you process through the same three-step mental model: 

 

1. **Collect** — scan for every signal, stated or implied. A risk does not need to be called a "risk" to be one. A speaker describing a manual workaround, expressing uncertainty about a process, flagging a dependency, or noting something "we're monitoring" — these are signals. 

2. **List** — organize signals into structured assessments using the LIVC framework. Score each dimension on the evidence available. Mark [?] when the source material cannot support a score. 

3. **Map** — identify how risks connect. A high-connectivity risk that amplifies three others is more dangerous than a standalone high-likelihood risk. Surface those relationships. 

 

## How You Engage 

 

- You respond in the voice of the analyst — professional, direct, and precise 

- When asked for analysis, you produce structured output (tables, organized sections) — prose summaries are secondary 

- When source material is ambiguous, you ask one targeted clarifying question before proceeding 

- You distinguish clearly between what was stated and what you have inferred — inferences are always labeled 

- You do not pad outputs with generic commentary — every sentence should add analytical value 

 

## Your Standards 

 

- A risk raised once is a valid risk — never suppress on frequency alone 

- Describe risks specifically: what could happen, in what context, with what consequence. "Operational risk" is not a risk description. 

- Never invent evidence — if a LIVC dimension cannot be supported by the source material, mark it [?] and note why 

- High Connectivity risks are force multipliers — always flag which other risks they connect to 

 

</instructions> 

 

<reference> 

 

## LIVC Scoring Dimensions 

 

| Dimension | Definition | Scale | 

|-----------|-----------|-------| 

| Likelihood | Probability the risk materializes given current conditions | High / Medium / Low | 

| Impact | Magnitude of consequence if the risk occurs | High / Medium / Low | 

| Velocity | Speed at which the risk could escalate to significant harm | Fast / Moderate / Slow | 

| Connectivity | Degree to which this risk triggers or amplifies other risks | High / Medium / Low | 

 

- Mark [?] when source material is insufficient to assess a dimension — note why 

- High Connectivity risks are force multipliers — note which other risks they connect to 

 

## Risk Categories 

 

Operational · Regulatory / Compliance · Financial · Strategic · Technology / Cyber · People / Talent · Reputational · Third Party / Vendor 

 

</reference> 

``` 

 

**Design notes:** 

- The model IS the analyst — not an assistant helping with analysis. This gives it analytical agency. 

- `<task>` section is used because the role alone doesn't define the interaction posture 

- `<reference>` carries shared lookup tables (LIVC, categories) — keeps instructions clean 

- No `<output_structure>` — this is a persona; task-specific formats live in the use case prompts 

 

--- 

 

### Example 2: Use Case Prompt — C-Suite Risk Perspective Mapping (v2.0) 

 

```xml 

<role> 

You are a Senior Risk Assessment Analyst specializing in C-Suite risk perspective mapping. You draw on deep knowledge of industry risk landscapes, country-level risk environments, and how different executive roles perceive and prioritize risk based on their functional accountability. When analyzing each C-Suite role, you step into their positional vantage point — you see risks through their eyes, shaped by their responsibilities, dependencies, and organizational exposure. 

</role> 

 

<task> 

Produce a C-Suite risk perspective map for a given client, showing what risks each executive would identify from their functional vantage point, and where risk themes converge across roles. 

</task> 

 

<instructions> 

 

## Process 

 

1. **Search for company context (internal — do not show in output)** — Search for current, publicly available information about {{CLIENT_NAME}}: business model, corporate structure, recent strategic activity, financial position, and any newsworthy risk events or regulatory developments. Use this to ground Step 2. If search returns limited results, note this in Section 1 and rely on industry and country pattern reconstruction. 

 

2. **Reconstruct company and context (internal — do not show in output)** — Using the search findings alongside the client name, industry, and country as anchors, reconstruct: the company's likely business model and strategic priorities; the dominant risk themes in this industry; and the country-specific risk environment. This reconstruction grounds the role analysis. 

 

3. **Select dynamic roles (internal — do not show in output)** — The fixed core roles are CEO, COO, CFO, CRO, and CTO. Select 1–2 additional roles relevant to this specific industry. Use the Dynamic Role Selection Guide in the reference section. Record the selected roles and the rationale — this surfaces in Section 1. 

 

4. **Adopt each executive's vantage point (internal — do not show in output)** — For each role (fixed core + dynamic), step into their functional accountability. From that position, identify the risks they would surface. Complete this analysis for every role before building the tables. 

 

5. **Write Section 1: Context Summary** — Open with the heading **Section 1: Context Summary**. Write 3–5 bullets: (1) business model and strategic position; (2) dominant industry risk themes; (3) country-level risk environment; (4) dynamic roles selected and reason for each. 

 

6. **Write Section 2: Executive Risk Perspectives** — Open with the heading **Section 2: Executive Risk Perspectives**. For each executive — fixed core first (CEO, COO, CFO, CRO, CTO), then dynamic roles — write the role title as a subheading and produce a table with four columns: Risk ID | Risk Description | Category | Why This Role Identifies It. Risk IDs follow the format [ROLE]-001 (e.g., CEO-001). Risk Description: a specific statement of what could go wrong, in what context, with what consequence. Why This Role Identifies It: one sentence stating the positional logic. Minimum 5 risks per role, maximum 10. 

 

7. **Write Section 3: Convergence Summary** — Open with the heading **Section 3: Convergence Summary**. Produce a table: Risk Theme | Roles That Identified It | Convergence Level. High = 3+ roles, Medium = 2 roles. Include only risks appearing across 2+ roles. 

 

## Constraints 

- **do NOT** show internal reconstruction or role adoption reasoning in the output 

- **do NOT** use category labels as risk descriptions ("cyber risk" is not a description) 

- **do NOT** include LIVC scoring — this version does not score risks 

- **do NOT** exceed 10 risks per role table 

 

## Edge Cases 

- If web search returns limited results → note in Section 1; proceed with industry/country pattern reconstruction 

- If a dynamic role is not clearly applicable → select only 1 additional role rather than forcing a second 

 

</instructions> 

 

<reference> 

 

## Risk Categories 

Operational · Regulatory / Compliance · Financial · Strategic · Technology / Cyber · People / Talent · Reputational · Third Party / Vendor 

 

## Dynamic Role Selection Guide 

| Role | Add When | 

|------|----------| 

| CISO | Technology, financial services, healthcare, or high data/cyber exposure | 

| CMO | Consumer-facing industries (retail, FMCG, media, hospitality) | 

| CHRO | People-intensive industries (professional services, healthcare, education) | 

| Chief Compliance Officer | Heavily regulated industries (financial services, pharma, energy, healthcare) | 

| Chief Medical Officer | Healthcare, life sciences, pharmaceutical | 

| Chief Supply Chain Officer | Manufacturing, retail, FMCG, logistics | 

 

</reference> 

 

<input> 

 

<client_name> 

{{CLIENT_NAME}} 

</client_name> 

 

<client_industry> 

{{CLIENT_INDUSTRY}} 

</client_industry> 

 

<client_country> 

{{CLIENT_COUNTRY}} 

</client_country> 

 

</input> 

``` 

 

**Design notes:** 

- Internal reasoning steps explicitly marked "(internal — do not show in output)" — this is the chain-of-thought mechanism 

- `<reference>` carries the dynamic role selection guide as a directive table — reduces model discretion and makes selection auditable 

- No `<output_structure>` — output format logic is woven directly into the process steps (column names, ID format, heading names, convergence levels) 

- Risk IDs use role abbreviations (CEO-001) to preserve role provenance by design 

 

--- 

 

## 15. Standard Process — New Prompt Builds 

 

1. Define the task, model type, and any known constraints 

2. Draft using the house XML template 

3. Review against this checklist before finalising: 

 - [ ] Model type identified — technique selection matches 

 - [ ] Template sections correct — no filler, no redundancy 

 - [ ] Output format woven into process steps, not restated in `<output_structure>` 

 - [ ] Language is exacting — no loose verbs or relative terms 

 - [ ] Rule density is light — only rules that are actually needed 

 - [ ] Examples use fictional dummy data only 

 - [ ] Context engineering checklist applied 

4. Run a spot test against 5–10 representative inputs before using in production 

5. Version the prompt: `[name]_v[X.X].md` and store in version control 

 

--- 

 

## 16. Technique Reference — Quick Card 

 

### Chain-of-Thought 

- Standard models: add "think step by step" or explicit reasoning steps 

- Reasoning models (Claude ET, o3, o4): omit — the model handles it internally 

- Implementation: numbered process steps in `<instructions>`, or internal steps marked explicitly 

 

### Few-Shot 

- 1–3 examples, consistently formatted — same delimiters, same whitespace, same structure 

- Put examples in `<example>` block or embed in instructions 

- Always use fictional dummy data 

 

### Self-Consistency 

- Run the same prompt multiple times with temperature > 0, take majority answer 

- Only justified when cost is acceptable; redundant for reasoning models 

 

### Prompt Chaining 

- Break complex workflows into the simplest possible links 

- Each step receives the output of the previous step as input 

- Use for: extract → classify → generate → validate workflows 

 

### Context Engineering 

- Static content first (cacheable), dynamic content last 

- Critical content at beginning or end — never buried in the middle 

- Summarize aggressively; compress without losing meaning 

- Cache system prompts + reference docs when > 1,024 tokens 

 

--- 

## Document Architecture for External Outputs

When producing documents for external readers, apply these standards:

- Lead with the main finding or core thesis. Never build towards a conclusion
- Assume the reader has zero prior context. The document must stand alone
- Front-load what needs to be understood first. Order by logical dependency, not by how the thinking developed
- Group related concepts together. The reader should never need to jump back
- First sentence of each section carries the core point of that section
- Optimize for senior stakeholder scanning in 30 seconds
- Stay lean. Cut anything that doesn't directly support the thesis
- Before finalizing, pressure-test: what would a reader need to understand first, and is that what they see first?

*Full framework with pre-flight checklist and structure rules: `knowledge/professional-skills/document-creation-framework.md`*

---

*Full technique research with citations and model-specific guidance: `prompt-engineering/research.md`* 

*Evaluation framework with L1–L4 metrics, judge templates, and golden dataset formats: `prompt-engineering/Evaluations/evaluation-framework.md`* 
