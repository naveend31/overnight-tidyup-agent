# Overnight Tidy-Up Agent - Complete Deployment Guide

> **Your agent**: Runs every night at 11 PM, reads your backlog from S3, uses Amazon Bedrock (Nova Lite) to categorize/prioritize items, and emails you a clean summary so you start fresh every morning.

---

## Architecture

```
EventBridge Scheduler (cron: 11 PM daily)
        |
        v
   AWS Lambda (Python 3.12)
        |
        ├── Reads backlog from S3
        ├── Calls Amazon Bedrock (Nova Lite) to analyze & categorize
        ├── Saves report back to S3
        └── Sends email digest via Amazon SES

```

**AWS Services Used:**

- **Amazon EventBridge Scheduler** — triggers Lambda on schedule (Free Tier)
- **AWS Lambda** — runs agent code (Free Tier: 1M requests/month)
- **Amazon S3** — stores backlog items + generated reports (Free Tier: 5GB)
- **Amazon Bedrock (Nova Lite)** — AI categorization & summarization
- **Amazon SES** — sends email digest (Free Tier: 62,000 emails/month)

---

## Prerequisites

1. An **AWS Account** (new accounts get Free Tier credits)
2. Your **email address** (for receiving reports)
3. ~30 minutes of setup time

---

## Step-by-Step Setup (All via AWS Console)

### Step 1: Create an S3 Bucket

1. Go to **S3 Console**: [https://console.aws.amazon.com/s3/](https://console.aws.amazon.com/s3/)
2. Click **Create bucket**
3. **Bucket name**: `my-backlog-tidyup-agent` (must be globally unique — add your initials)
4. **Region**: `us-east-1` (N. Virginia) — keep all services in the same region
5. Leave all other settings as default
6. Click **Create bucket**

**Upload the sample backlog:**

1. Click into your new bucket
2. Click **Create folder** → name it `backlog` → click Create
3. Click into the `backlog` folder
4. Click **Upload** → upload the `sample_backlog.json` file (rename it to `items.json`)
5. Also create a `reports` folder (the agent saves summaries here)

---

### Step 2: Verify Your Email in SES

1. Go to **SES Console**: [https://console.aws.amazon.com/ses/](https://console.aws.amazon.com/ses/)
2. In the left menu, click **Identities**
3. Click **Create identity**
4. Choose **Email address**
5. Enter your email (e.g., `naveenhari123456@gmail.com`)
6. Click **Create identity**
7. **Check your inbox** — click the verification link in the email from AWS

> **Note**: SES starts in "sandbox mode" — you can only send to verified emails. This is fine for the challenge!

---

### Step 3: Enable Amazon Bedrock Model Access

1. Go to **Bedrock Console**: [https://console.aws.amazon.com/bedrock/](https://console.aws.amazon.com/bedrock/)
2. In the left menu, click **Model access**
3. Click **Manage model access** (or "Modify model access")
4. Check the box next to **Amazon Nova Lite**
5. Click **Save changes**
6. Wait ~1 minute for status to show "Access granted"

---

### Step 4: Create the IAM Role for Lambda

1. Go to **IAM Console**: [https://console.aws.amazon.com/iam/](https://console.aws.amazon.com/iam/)
2. Click **Roles** → **Create role**
3. **Trusted entity**: AWS service → **Lambda** → Next
4. **Permissions**: Click **Create policy** (opens new tab):- Click **JSON** tab

- Paste the contents of `iam_permissions_policy.json`
- **IMPORTANT**: Replace `YOUR-BUCKET-NAME` with your actual bucket name
- Click **Next** → Name it `TidyUpAgentPolicy` → **Create policy**

1. Back in the role creation tab, refresh and search for `TidyUpAgentPolicy`
2. Check the box → Next
3. **Role name**: `TidyUpAgentRole`
4. Click **Create role**

---

### Step 5: Create the Lambda Function

1. Go to **Lambda Console**: [https://console.aws.amazon.com/lambda/](https://console.aws.amazon.com/lambda/)
2. Click **Create function**
3. Choose **Author from scratch**
4. **Function name**: `overnight-tidyup-agent`
5. **Runtime**: Python 3.12
6. **Architecture**: x86_64
7. Expand **Change default execution role** → **Use an existing role** → select `TidyUpAgentRole`
8. Click **Create function**

**Configure the function:**

1. In the **Code** tab, delete the default code
2. Paste the entire contents of `lambda_function.py`
3. Click **Deploy**

**Set environment variables:**

1. Click **Configuration** tab → **Environment variables** → **Edit**
2. Add these variables:| Key | Value | | --- | --- | | `BUCKET_NAME` | `my-backlog-tidyup-agent` (your bucket name) | | `EMAIL_TO` | `naveenhari123456@gmail.com` (your email) | | `EMAIL_FROM` | `naveenhari123456@gmail.com` (same verified email) | | `MODEL_ID` | `amazon.nova-lite-v1:0` |
3. Click **Save**

**Increase timeout:**

1. **Configuration** → **General configuration** → **Edit**
2. Set **Timeout** to `1 min 0 sec` (Bedrock calls can take 10-20s)
3. Set **Memory** to `256 MB`
4. Click **Save**

---

### Step 6: Test the Lambda Function

1. Click the **Test** tab
2. **Event name**: `TestRun`
3. **Event JSON**: `{}`
4. Click **Test**
5. You should see a green "Execution result: succeeded" and receive an email!

> If it fails, check CloudWatch Logs for error details (click the "logs" link in the error output).

---

### Step 7: Create EventBridge Schedule (The "Always-On" Part!)

1. Go to **EventBridge Console**: [https://console.aws.amazon.com/events/](https://console.aws.amazon.com/events/)
2. In the left menu, click **Schedules** (under Scheduler)
3. Click **Create schedule**
4. **Schedule name**: `nightly-tidyup-schedule`
5. **Schedule type**: **Recurring schedule**
6. **Schedule expression**: Choose **Cron-based schedule**
7. Enter: `cron(30 17 * * ? *)` — this is 11 PM IST (5:30 PM UTC)
8. **Flexible time window**: Off
9. Click **Next**

**Target:**

1. Select **AWS Lambda - Invoke**
2. Choose your function: `overnight-tidyup-agent`
3. Click **Next**

**Settings:**

1. Leave retry policy as default (2 retries)
2. Click **Next** → **Create schedule**

**Done! Your agent will now run every night at 11 PM IST.**

---

### Step 8: Verify the Schedule Fired (For Your Article Screenshots)

After the first scheduled run:

1. Go to **CloudWatch Console** → **Log groups** → `/aws/lambda/overnight-tidyup-agent`
2. Click the latest log stream — you'll see the execution logs
3. Check your **email inbox** for the digest
4. Check your **S3 bucket** → `reports/` folder for the saved report

**Take screenshots of:**

- The EventBridge schedule showing "Enabled"
- The CloudWatch log showing execution
- The email you received
- The S3 report file

These prove your agent runs autonomously — great for the article!

---

## Cost Estimate (Free Tier)

| Service | Free Tier Allowance | Your Usage |
| --- | --- | --- |
| Lambda | 1M requests + 400K GB-sec/month | ~30 requests/month |
| S3 | 5 GB storage | < 1 MB |
| Bedrock Nova Lite | $200 credits (new accounts) | ~$0.01/day |
| SES | 62,000 emails/month | ~30/month |
| EventBridge | 14M invocations/month | ~30/month |

**Total monthly cost: Effectively $0** (within Free Tier)

---

## Troubleshooting

| Issue | Fix |
| --- | --- |
| "Access denied" on S3 | Check IAM policy has correct bucket name |
| Bedrock "model not found" | Ensure model access is granted in Bedrock console |
| SES "Email not verified" | Re-verify your email in SES → Identities |
| Timeout error | Increase Lambda timeout to 2 minutes |
| Schedule not firing | Check timezone — `cron(30 17 * * ? *)` = 11 PM IST |

---

## Next Steps (Optional Enhancements)

- **Real data source**: Connect to Jira, Notion, or Trello API instead of S3 JSON
- **Slack notifications**: Add SNS + Slack webhook for instant messages
- **Weekly summary**: Add a second schedule for a weekly rollup report
- **Multiple backlogs**: Support different project backlogs in separate S3 prefixes

