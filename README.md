An AI-powered backlog organizer that runs autonomously every night, categorizes your tasks with Amazon Bedrock, and emails you a clean priority summary before you wake up.

## What It Does

-   **Runs on schedule**: Triggered by EventBridge Scheduler at 11 PM IST daily (no manual clicks)
    
-   **Reads your backlog**: Fetches tasks from S3 JSON file
    
-   **Analyzes with AI**: Uses Amazon Bedrock (Nova Lite) to categorize, prioritize, and identify stale items
    
-   **Sends a summary**: Emails you a clean, scannable digest via Amazon SES
    
-   **Saves reports**: Archives each analysis to S3 for history tracking
    

## Architecture

```
EventBridge Scheduler (11 PM IST daily)
        ↓
AWS Lambda (Python 3.12)
    ├── S3 (read backlog)
    ├── Bedrock Nova Lite (AI analysis)
    ├── S3 (save report)
    └── SES (send email)
```

## AWS Services Used

<table style="min-width: 75px;"><colgroup><col style="min-width: 25px;"><col style="min-width: 25px;"><col style="min-width: 25px;"></colgroup><tbody><tr><th colspan="1" rowspan="1"><p>Service</p></th><th colspan="1" rowspan="1"><p>Purpose</p></th><th colspan="1" rowspan="1"><p>Free Tier?</p></th></tr><tr><td colspan="1" rowspan="1"><p>EventBridge Scheduler</p></td><td colspan="1" rowspan="1"><p>Daily trigger</p></td><td colspan="1" rowspan="1"><p>Yes (14M/month)</p></td></tr><tr><td colspan="1" rowspan="1"><p>AWS Lambda</p></td><td colspan="1" rowspan="1"><p>Agent code runner</p></td><td colspan="1" rowspan="1"><p>Yes (1M requests/month)</p></td></tr><tr><td colspan="1" rowspan="1"><p>Amazon S3</p></td><td colspan="1" rowspan="1"><p>Backlog + reports storage</p></td><td colspan="1" rowspan="1"><p>Yes (5 GB)</p></td></tr><tr><td colspan="1" rowspan="1"><p>Amazon Bedrock (Nova Lite)</p></td><td colspan="1" rowspan="1"><p>AI summarization</p></td><td colspan="1" rowspan="1"><p>Credits for new accounts</p></td></tr><tr><td colspan="1" rowspan="1"><p>Amazon SES</p></td><td colspan="1" rowspan="1"><p>Email delivery</p></td><td colspan="1" rowspan="1"><p>Yes (62K emails/month)</p></td></tr></tbody></table>

**Total cost: ~$0/month** (within Free Tier)

## Quick Start

### Prerequisites

-   AWS Account with Free Tier credits
    
-   Verified email in SES
    
-   Nova Lite model access in Bedrock
    

### Deployment (5 steps)

1.  **Create S3 bucket** → upload `sample_backlog.json` to `backlog/items.json`
    
2.  **Verify email** in SES (Identities → Create identity)
    
3.  **Enable Nova Lite** in Bedrock (Model access)
    
4.  **Create IAM role** with permissions from `iam_permissions_policy.json`
    
5.  **Deploy Lambda** with `lambda_function.py` + environment variables:
    
    -   `BUCKET_NAME`: your S3 bucket name
        
    -   `EMAIL_TO`: your verified email
        
    -   `EMAIL_FROM`: your verified email
        
    -   `MODEL_ID`: `amazon.nova-lite-v1:0`
        
6.  **Create EventBridge schedule**: `cron(30 17 * * ? *)` (11 PM IST)
    

**Full step-by-step guide**: See `DEPLOYMENT_GUIDE.md`

## Files in This Repo

-   `lambda_function.py` - Complete agent code (Python 3.12)
    
-   `sample_backlog.json` - Example backlog data
    
-   `iam_permissions_policy.json` - IAM policy (copy-paste into AWS)
    
-   `iam_trust_policy.json` - IAM trust policy
    
-   `DEPLOYMENT_GUIDE.md` - Detailed deployment walkthrough
    
-   `README.md` - This file
    

## How to Use

1.  Clone this repo
    
2.  Follow `DEPLOYMENT_GUIDE.md` to set up on AWS
    
3.  Update `sample_backlog.json` with your actual backlog items
    
4.  Upload to S3: `s3://YOUR-BUCKET/backlog/items.json`
    
5.  Agent runs every night at 11 PM IST automatically
    

## Example Output

The agent sends an HTML email with:

-   Categorized tasks (Urgent / Needs Attention / Low Priority / Done)
    
-   Stale item alerts
    
-   Top 3 priorities for the day
    
-   Suggested archival/delegation items
    

## Cost Estimate

-   Lambda invocations: ~30/month = ~$0
    
-   Bedrock Nova Lite: ~$0.01/day = ~$0.30/month
    
-   SES emails: ~30/month = ~$0
    
-   S3 storage: <1 MB = ~$0
    

**Total: ~$0.30/month** (well within Free Tier)

## Architecture Decisions

-   **Nova Lite over Nova Pro**: Excellent quality for backlog summaries at minimal cost
    
-   **EventBridge Scheduler**: Simple, reliable, serverless scheduling (no server to manage)
    
-   **S3 for data**: Easy to update backlog from anywhere, history tracking built-in
    
-   **SES for email**: Free Tier friendly, supports rich HTML formatting
    

## Next Steps / Enhancements

-   Connect to Jira/Notion API instead of S3 JSON
    
-   Add Slack notifications
    
-   Weekly digest in addition to nightly
    
-   Support multiple team backlogs
    
-   Add filtering by project/team
    

## Built For

AWS Builder Center Weekend Agent Challenge, July 2026

**Challenge**: Build an always-on agent using AWS Free Tier services, deploy it, and write about your experience.
