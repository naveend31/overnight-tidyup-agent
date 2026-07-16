# Weekend Agent Challenge: The Overnight Tidy-Up Agent

*An AI-powered backlog organizer that runs every night, categorizes your tasks, identifies stale items, and delivers a clean priority summary to your inbox before you wake up.*

---

## Vision & What the Agent Does

As a Product Manager, I juggle dozens of backlog items across features, stakeholders, and deadlines. Items pile up. Some go stale. Others need a nudge. The mental overhead of triaging every morning eats into my most productive hours.

**The Overnight Tidy-Up Agent** solves this by doing the sorting while I sleep. Every night at 11 PM, it:

1. **Wakes up on schedule** — triggered by Amazon EventBridge Scheduler (no manual button clicks)
2. **Reads my backlog** — pulls the latest items from an S3 bucket
3. **Analyzes with AI** — uses Amazon Bedrock (Nova Lite) to categorize items by urgency, identify stale tasks that need follow-up, and suggest what can be archived
4. **Delivers a clean digest** — emails me a prioritized, scannable summary via Amazon SES
5. **Archives the report** — saves each nightly analysis to S3 for reference

The result? I open my inbox to a sorted, AI-prioritized view of my work. No morning triage needed. The best productivity tool is the one I never have to open.

---

## How I Built It

### Development Process

I started by defining the core loop: **fetch → analyze → report → notify**. Each step maps cleanly to an AWS service, keeping the architecture simple and serverless.

**Key decisions:**
- **Nova Lite over Nova Pro**: For a nightly summary of ~10-20 items, Nova Lite provides excellent quality at minimal cost. The temperature is set to 0.3 for consistent, focused outputs.
- **S3 as the data source**: Simple JSON file that I can update from anywhere (web, CLI, or another tool). Easy to extend later to pull from Jira/Notion APIs.
- **SES for delivery**: Reliable, free-tier-friendly, and supports rich HTML emails with proper formatting.
- **Reports saved to S3**: Every nightly run creates a dated report, giving me a history of how my backlog evolved over time.

### Challenges & Solutions

1. **Bedrock prompt engineering**: My first prompt returned generic lists. I iterated to request specific formatting (emoji headers, stale-item detection with time thresholds), which made the output immediately scannable.

2. **Timezone handling**: EventBridge uses UTC, but I need the agent to run at 11 PM IST. Solution: `cron(30 17 * * ? *)` — 5:30 PM UTC = 11:00 PM IST.

3. **Lambda timeout**: Initial 3-second default wasn't enough for Bedrock inference. Bumped to 60 seconds with 256 MB memory for comfortable headroom.

---

## AWS Services Used / Architecture Overview

### Architecture Diagram

```
┌──────────────────────────┐
│  EventBridge Scheduler   │
│  cron(30 17 * * ? *)     │
│  (11 PM IST / daily)     │
└───────────┬──────────────┘
            │ triggers
            v
┌──────────────────────────┐
│      AWS Lambda          │
│   (Python 3.12, 256MB)   │
│                          │
│  1. Fetch backlog (S3)   │
│  2. Analyze (Bedrock)    │
│  3. Save report (S3)     │
│  4. Send email (SES)     │
└──┬────────┬────────┬─────┘
   │        │        │
   v        v        v
┌──────┐ ┌───────┐ ┌─────┐
│  S3  │ │Bedrock│ │ SES │
│      │ │(Nova  │ │     │
│input │ │ Lite) │ │email│
│  +   │ │       │ │     │
│output│ │  AI   │ │notif│
└──────┘ └───────┘ └─────┘
```

### Services Breakdown

| Service | Role | Free Tier? |
|---------|------|------------|
| **EventBridge Scheduler** | Triggers Lambda at 11 PM IST daily | Yes (14M invocations/month) |
| **AWS Lambda** | Runs the agent code (Python 3.12) | Yes (1M requests/month) |
| **Amazon S3** | Stores backlog items + nightly reports | Yes (5 GB) |
| **Amazon Bedrock (Nova Lite)** | AI-powered categorization & summarization | Credits for new accounts |
| **Amazon SES** | Sends formatted HTML email digest | Yes (62K emails/month) |

### How It's Triggered

The agent is triggered **exclusively by EventBridge Scheduler** — there is no button, no API endpoint, no manual step. The cron expression `cron(30 17 * * ? *)` fires every day at 5:30 PM UTC (11:00 PM IST). EventBridge invokes the Lambda function with an empty event payload, and the agent takes it from there.

---

## What I Learned

1. **EventBridge Scheduler is incredibly powerful for "always-on" patterns.** Setting up a cron-triggered Lambda took minutes — no servers to manage, no daemons to keep alive. The agent truly runs itself.

2. **Amazon Bedrock's Nova Lite model is surprisingly capable for structured analysis.** With the right prompt, it consistently categorizes items, detects staleness by comparing dates, and produces well-formatted summaries. At $0.001 or less per run, it's essentially free.

3. **The "overnight agent" pattern is a game-changer for PMs.** Having a sorted inbox waiting for me completely eliminated my 15-minute morning triage. I'm extending this to pull from actual project management tools next.

4. **IAM permissions are the hardest part for beginners.** Getting the right combination of S3, Bedrock, SES, and CloudWatch permissions took trial and error. My tip: start with the AWS-managed `AmazonBedrockFullAccess` policy for testing, then scope it down for production.

5. **Prompt temperature matters.** At temperature 0.7, the summaries were creative but inconsistent. Dropping to 0.3 gave reliable, structured output every time — exactly what you want from an autonomous agent.

---

## Link to Repo

**GitHub Repository**: [INSERT YOUR GITHUB REPO URL HERE]

The repo includes:
- `lambda_function.py` — complete agent code
- `sample_backlog.json` — example backlog data
- `iam_permissions_policy.json` — IAM policy template
- `README.md` — deployment instructions

---

*Built for the AWS Builder Center Weekend Agent Challenge, July 2026.*
*Powered by Amazon Bedrock, EventBridge, Lambda, S3, and SES.*
