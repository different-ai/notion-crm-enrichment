---
name: setup-crm
description: Bootstrap your AI-powered CRM. Verifies Notion connection, creates databases for leads and messages, creates a Feedback Insights page, and configures workspace. Run this first.
---

# Setup CRM - Bootstrap Your Outreach System

You are the setup wizard for the Notion CRM Enrichment system. You help users go from zero to a working AI-powered outreach system.

## Overview

This skill:
1. Verifies Notion MCP connection works
2. Creates an "Outreach CRM" page in Notion with two databases
3. Creates a "Feedback Insights" page for the learning loop
4. Writes a local config file so other skills can find the databases

---

## Step 1: Check Prerequisites

### Check Notion MCP

Call `mcp__notion__notion-get-users` to verify Notion is connected.

**If fails:** Tell user to:
1. Run `claude mcp add notion --url https://mcp.notion.com/mcp`
2. Authenticate with Notion when prompted
3. Run /setup-crm again

**If succeeds:** Report `[OK] Notion connected`

---

## Step 2: Create CRM Infrastructure

### Create the main page

Use `mcp__notion__notion-create-pages` to create a page titled "Outreach CRM" with this content:

```markdown
# Outreach CRM

This page was created by the Notion CRM Enrichment system. It contains your lead pipeline, message tracking, and feedback insights.

## How to Use

1. **Add leads:** `/add-lead [Name] from [Company]`
2. **Draft messages:** `/draft-message [Name]`
3. **Log sent messages:** `/update-crm [Name] sent: "[message]"`
4. **Rate outcomes:** `/rate-message [Name] replied/no-reply`

## The Learning Loop

Every message you send gets rated. Every outcome gets tracked. The system learns what works for YOU.
```

### Create Leads Database

Use `mcp__notion__notion-create-database` with parent = the page you just created:

```json
{
  "title": [{ "type": "text", "text": { "content": "Leads" } }],
  "properties": {
    "Name": { "type": "title", "title": {} },
    "Company": { "type": "rich_text", "rich_text": {} },
    "LinkedIn": { "type": "url", "url": {} },
    "Status": {
      "type": "select",
      "select": {
        "options": [
          { "name": "New", "color": "gray" },
          { "name": "Researched", "color": "blue" },
          { "name": "Contacted", "color": "purple" },
          { "name": "Replied", "color": "yellow" },
          { "name": "Meeting", "color": "orange" },
          { "name": "Won", "color": "green" },
          { "name": "Lost", "color": "red" }
        ]
      }
    },
    "ICP": {
      "type": "multi_select",
      "multi_select": {
        "options": [
          { "name": "Startup", "color": "blue" },
          { "name": "Enterprise", "color": "purple" },
          { "name": "SMB", "color": "green" }
        ]
      }
    },
    "Context": { "type": "rich_text", "rich_text": {} },
    "PULL Analysis": { "type": "rich_text", "rich_text": {} },
    "Last Contact": { "type": "date", "date": {} }
  }
}
```

### Create Messages Database

Use `mcp__notion__notion-create-database`:

```json
{
  "title": [{ "type": "text", "text": { "content": "Messages" } }],
  "properties": {
    "Title": { "type": "title", "title": {} },
    "Platform": {
      "type": "select",
      "select": {
        "options": [
          { "name": "LinkedIn", "color": "blue" },
          { "name": "Email", "color": "red" },
          { "name": "Twitter", "color": "default" }
        ]
      }
    },
    "Date": { "type": "date", "date": {} },
    "Result": {
      "type": "select",
      "select": {
        "options": [
          { "name": "Pending", "color": "gray" },
          { "name": "Reply", "color": "green" },
          { "name": "No Reply", "color": "red" },
          { "name": "Meeting", "color": "orange" }
        ]
      }
    },
    "Rating": { "type": "number", "number": { "format": "number" } }
  }
}
```

### Create Feedback Insights Page

Use `mcp__notion__notion-create-pages` with parent = the main "Outreach CRM" page:

```markdown
# Feedback Insights

This page tracks what's working and what's failing in your outreach. It's automatically updated by `/rate-message` and read by `/draft-message`.

---

## What's Working

_Patterns that get replies. Updated automatically._

- (No data yet - send some messages and track outcomes!)

---

## What's Failing

_Anti-patterns to avoid. Updated automatically._

- (No data yet)

---

## Proven Templates

_Message structures that have worked._

### Vision Templates
- "startups lose money sitting in checking accounts"

### Weakness Templates
- "honestly stuck on whether [X] or [Y]"

### Pedestal Templates
- "you [specific experience] so you've probably [relevant insight]"

### Ask Templates
- "is it [option A] or [option B]? even a one-liner helps"

---

## Stats

| Metric | Value |
|--------|-------|
| Total Messages | 0 |
| Reply Rate | 0% |
| Avg Rating | 0/10 |
| Best Element | - |
| Worst Element | - |

_Last updated: Never_
```

---

## Step 3: Save Configuration

After creating databases and pages, extract their IDs from the API responses.

Write to `.claude/config/workspace.json`:

```json
{
  "initialized_at": "[CURRENT_ISO_DATE]",
  "crm_page_id": "[OUTREACH_CRM_PAGE_ID]",
  "crm_page_url": "[OUTREACH_CRM_PAGE_URL]",
  "databases": {
    "leads": "collection://[LEADS_DB_ID]",
    "messages": "collection://[MESSAGES_DB_ID]"
  },
  "pages": {
    "feedback_insights": "[FEEDBACK_INSIGHTS_PAGE_ID]"
  }
}
```

---

## Step 4: Report Success

```
Setup complete!

[OK] Notion connected
[OK] Created "Outreach CRM" page
[OK] Created "Leads" database
[OK] Created "Messages" database
[OK] Created "Feedback Insights" page
[OK] Config saved to .claude/config/workspace.json

Your CRM: [Notion URL]

You can now use:
  /add-lead [Name] from [Company]
  /draft-message [Name]
  /update-crm [Name] sent: "[message]"
  /rate-message [Name] replied

Try it:
  /add-lead Sam Altman from OpenAI
```

---

## Error Handling

| Error | Response |
|-------|----------|
| Notion auth fails | Provide MCP configuration instructions |
| Database creation fails | Check parent page permissions |
| Page already exists | Offer to use existing or create new |
| Config write fails | Check file permissions |
