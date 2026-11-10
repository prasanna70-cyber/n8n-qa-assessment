# Task 2: n8n API Integration Workflow - Hacker News Morning Brief

**Author:** Deepthi Prasanna

## What it does
Every hour (Schedule Trigger) the workflow fetches the top Hacker News stories, keeps the top 5, enriches each with its details, builds a short digest and sends it by email (Gmail). A Manual Trigger is also connected so I can test it on demand.

## APIs used and why
- **HackerNews Firebase API** (`/v0/topstories.json`, then `/v0/item/{id}.json`). It is free, needs no API key, and the two endpoints fit the "list then enrich" pattern. The second call fetches the title, score, link and comment count for each story.
- **Gmail** (n8n Gmail node, OAuth) for the output. It is free and the digest arrives in my inbox within seconds.

## Flow
1. **Schedule Trigger** (every 1 hour) / **Manual Trigger** for testing.
2. **Get Top Story IDs** (HTTP Request).
3. **Take Top 5** (Code node): normalizes the response and keeps only the first 5 IDs.
4. **Get Story Details** (HTTP Request, runs once per ID): second API call that enriches each item.
5. **Build Digest** (Code node): removes invalid items, sorts by score (highest first), and builds one numbered text digest.
6. **Top Score > 100?** (IF node): threshold I chose because stories above 100 points are clearly popular.
   - True: **Send Hot Digest** (email subject starts with "HOT").
   - False: **Send Normal Digest** (email subject says "quiet day").

## Error handling
- Both HTTP Request nodes use **On Error: Continue (using error output)**, so a failed API call never stops the workflow silently.
- Failures are routed to **Alert on Failure**, which sends an email with the error text.
- **Build Digest** throws a clear error if no valid story comes back.

## Credentials
No secrets are in the JSON. The Gmail login is stored in n8n's **Credentials** store (Google OAuth) and referenced by the three Gmail nodes. Nothing secret is in the JSON.

## How to run
1. Import `Task2_Workflow_DeepthiPrasanna.json` in n8n.
2. In the 3 Gmail nodes, sign in with Google to create the credential and set the recipient email.
3. Click **Execute Workflow** (Manual Trigger) and check the email.
