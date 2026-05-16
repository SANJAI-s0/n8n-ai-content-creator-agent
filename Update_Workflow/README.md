# AI-Powered Content Creator Agent — n8n Workflow

## Overview

An end-to-end no-code automation that reads pending topics from Google Sheets, researches them in real-time using Tavily, generates platform-specific content via OpenAI GPT-4o (LinkedIn, X/Twitter, Blog), and writes all outputs back to the sheet — fully automated, zero manual effort.

\---

## n8n Workflow Screenshot

!\[n8n Workflow Canvas](workflow\_screenshot.png)

The workflow follows this pipeline across 12 nodes:

```
\[Schedule Trigger] → \[Google Sheets: Get Pending] → \[IF: Topic Found?]
                                                           │ YES
                                                   \[HTTP Request: Tavily]
                                                   \[Split Out]
                                                   \[Aggregate]
                                                   \[Code: Prepare Context]
                                               ↙        ↓         ↘
                               \[OpenAI: LinkedIn] \[OpenAI: X] \[OpenAI: Blog]
                                               ↘        ↓         ↙
                                             \[Code: Combine Outputs]
                                             \[Google Sheets: Update ✓]
```

\---

## API Keys Used

|Service|Purpose|Variable Name|
|-|-|-|
|**OpenAI API**|GPT-4o — generate LinkedIn, X, and Blog content|`OPENAI\_API\_KEY`|
|**Tavily Search API**|Real-time web research per topic|`TAVILY\_API\_KEY`|
|**Google Sheets OAuth2**|Read pending topics + write generated content|`GOOGLE\_SHEETS\_OAUTH2`|

> \*\*Note:\*\* Actual API keys are never stored in the workflow JSON. All keys are configured via n8n's built-in Credentials manager.

\---

## Google Sheet Structure

Create a sheet named **Content Topics** with these exact column headers:

|Topic|Status|LinkedIn\_Post|X\_Post|Blog\_Summary|Published\_Date|
|-|-|-|-|-|-|
|AI in Healthcare 2025|Pending|*(auto-filled)*|*(auto-filled)*|*(auto-filled)*|*(auto-filled)*|
|Remote Work Trends|Pending|||||

* Set **Status = "Pending"** for new topics
* The workflow auto-sets **Status = "Completed"** after generating all content

\---

## Sample Input \& Generated Outputs

### Input Topic

**"The Rise of Agentic AI in 2025"**

\---

### Generated LinkedIn Post

Most people think AI means chatbots. They're missing the real shift.

Agentic AI — systems that don't just answer questions but autonomously plan, decide, and execute multi-step tasks — is quietly transforming how work gets done in 2025.

Here's what changed:

In 2023, AI responded to prompts. In 2025, AI completes projects.

Companies like Salesforce and Microsoft are deploying agents that handle entire workflows: research → draft → review → send — with minimal human input. Early adopters report 40–60% reduction in time spent on knowledge work.

But here's the question no one is asking: As agents handle more execution, what does "skilled work" actually mean anymore?

Are you experimenting with AI agents in your workflow? What's working — and what isn't?

\#AgenticAI #FutureOfWork #AIProductivity #ArtificialIntelligence #WorkplaceInnovation

\---

### Generated X (Twitter) Post

Agentic AI doesn't answer your questions. It completes your projects.

2023: "Write me an email"
2025: "Handle my entire outreach campaign"

The shift is bigger than most realise. Thoughts? #AgenticAI #AI2025

*(247 characters)*

\---

### Generated Blog Summary

The artificial intelligence landscape underwent a defining transformation in 2025, moving decisively beyond conversational chatbots toward agentic systems capable of autonomous, multi-step reasoning and real-world action. Unlike earlier AI tools that responded to individual prompts, agentic frameworks enable models to decompose complex goals, sequence actions, use external tools, and self-correct — all with minimal human oversight.

Major technology platforms have accelerated enterprise adoption, with AI agents now handling workflows ranging from customer support escalation to software debugging and financial analysis. Research indicates organisations implementing agentic AI in knowledge-work pipelines have achieved efficiency gains of 35–60%, reshaping team structures and role definitions. The convergence of stronger reasoning models, reliable tool-use APIs, and lower inference costs has made production-grade agentic systems viable for mid-market companies for the first time.

As agentic AI matures, the strategic question for businesses is no longer whether to adopt these systems, but how to redesign workflows, governance, and talent strategies to maximise their transformative potential.

*(196 words)*

\---

## Prompt Design for Each Platform

### LinkedIn Post Prompt

**Goal:** Professional thought leadership that drives engagement (comments, shares).

**System prompt strategy:**

* Opens with a bold hook — no generic openers like "Excited to share"
* Short paragraphs (1–3 sentences) for mobile readability
* Ends with a question to trigger LinkedIn's comment-rewarding algorithm
* 3–5 relevant hashtags only
* Temperature: **0.75** | Max tokens: **600**

### X (Twitter) Post Prompt

**Goal:** Viral-ready, scroll-stopping content within the strict 260-character limit.

**System prompt strategy:**

* Strict 260-character ceiling stated explicitly
* Prohibits filler phrases ("Thread:", "Hot take:")
* Tone: sharp, direct, slightly provocative
* Temperature: **0.8** | Max tokens: **150**

### Blog Summary Prompt

**Goal:** SEO-friendly, flowing editorial prose serving as a compelling article preview.

**System prompt strategy:**

* Exact 150–200 word target with 3-paragraph structure
* Prohibits bullet points/headers — maintains editorial quality
* Natural keyword placement for SEO without stuffing
* Temperature: **0.65** | Max tokens: **400**

\---

## Workflow Node Details

|#|Node|Type|Purpose|
|-|-|-|-|
|1|Schedule Trigger|Trigger|Runs workflow every 6 hours automatically|
|2|Google Sheets|Read|Fetches first row where Status = Pending|
|3|IF|Conditional|Guards against empty runs|
|4|HTTP Request|HTTP|POST to Tavily API — 8 search results + answer|
|5|Split Out|Data|Separates result array into individual items|
|6|Aggregate|Data|Re-combines into single structured research object|
|7|Code|JS|Formats research into clean prompt context string|
|8|OpenAI GPT-4o|AI|Generates LinkedIn post|
|9|OpenAI GPT-4o|AI|Generates X post|
|10|OpenAI GPT-4o|AI|Generates blog summary|
|11|Code|JS|Merges 3 outputs, adds Published\_Date timestamp|
|12|Google Sheets|Write|Updates row, sets Status = Completed|

\---

## Setup Instructions

### Step 1 — Add Credentials in n8n

1. Go to **Settings → Credentials → New Credential**
2. Add **OpenAI API**: enter your API key
3. Add **HTTP Header Auth** (name: "Tavily API Key"):

   * Header Name: `api-key`
   * Header Value: your Tavily key
4. Add **Google Sheets OAuth2**: complete Google OAuth flow

### Step 2 — Import Workflow

1. In n8n: **+** → **Import from file**
2. Upload `AI\_Content\_Creator\_Agent-GeminiAI.json`
3. Open each node and assign the correct credential

### Step 3 — Set Google Sheet ID

1. Copy your Sheet ID from the URL:
`https://docs.google.com/spreadsheets/d/YOUR\_SHEET\_ID/edit`
2. Replace `YOUR\_GOOGLE\_SHEET\_ID` in both Google Sheets nodes

### Step 4 — Test

1. Add a row: Topic = "AI in Healthcare", Status = "Pending"
2. Click **Test Workflow** in n8n
3. Verify all 5 columns are filled and Status = "Completed"

### Step 5 — Activate

Toggle the workflow to **Active** — runs every 6 hours automatically.

\---

## Files in This Submission

```
submission/
├── AI\_Content\_Creator\_Agent-GeminiAI.json   ← n8n workflow export (import directly)
├── workflow\_screenshot.png      ← n8n canvas screenshot (12-node workflow)
└── README.md                    ← This file
```

\---

*Built with n8n · OpenAI GPT-4o · Tavily Search API · Google Sheets OAuth2*

