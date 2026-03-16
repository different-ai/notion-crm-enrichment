---
name: update-crm
description: Log sent messages to your CRM, update lead status, and rate messages against the 5-element framework. Use after sending a message.
---

# Update CRM - Log Outreach Activity

You log sent messages, update lead status, and rate against the 5-element framework.

## Input Format

User will say something like:
- "Update Sam Altman, sent on LinkedIn: 'Hey Sam...'"
- "Log message to Jane Doe: [message content]"
- "Sarah Chen sent on Email: [message]"

---

## Step 1: Load Configuration

Read `.claude/config/workspace.json` to get database collection URLs.

**If file doesn't exist:** Tell user to run `/setup-crm` first.

---

## Step 2: Find the Lead

Search for the person in Leads database using `mcp__notion__notion-search`:

```
query: [name]
data_source_url: [leads collection URL from config]
```

**If not found:** Offer to create a basic entry or suggest `/add-lead` first.

---

## Step 3: Create Message Entry

Use `mcp__notion__notion-create-pages` to log the message:

```json
{
  "parent": {
    "type": "data_source_id",
    "data_source_id": "[messages collection ID]"
  },
  "pages": [{
    "properties": {
      "Title": "[Name] ([Company]) - [brief angle]",
      "Platform": "[LinkedIn/Email/Twitter]",
      "date:Date:start": "[YYYY-MM-DD]",
      "date:Date:is_datetime": 0,
      "Result": "Pending",
      "Rating": [calculated rating 1-10]
    },
    "content": "```\n[Full message text]\n```"
  }]
}
```

---

## Step 4: Rate the Message

Score against the 5-element framework:

| Element | Points | Check |
|---------|--------|-------|
| **Vision** | 2 | Human problem stated without product pitch |
| **Framing** | 1 | "not selling" or similar disclaimer |
| **Weakness** | 3 | Admits confusion or asks for help (MOST IMPORTANT) |
| **Pedestal** | 2 | Specific reason why THEM |
| **Ask** | 2 | Clear question, preferably binary |

**Modifiers:**
- Pattern interrupt opener: +0.5
- Specific reference to their content: +1
- Generic flattery: -2
- Link in first message: -2
- Too long (5+ paragraphs): -1

Calculate total and set Rating property.

---

## Step 5: Update Lead Status

Use `mcp__notion__notion-update-page` on the lead:

```json
{
  "page_id": "[lead page ID]",
  "command": "update_properties",
  "properties": {
    "Status": "Contacted",
    "date:Last Contact:start": "[YYYY-MM-DD]",
    "date:Last Contact:is_datetime": 0
  }
}
```

---

## Step 6: Report Results

```
Logged: [Name] ([Company])

Platform: [Platform]
Rating: [X]/10
Status: [Old] → Contacted

Rating breakdown:
  [✓/✗] Vision (2pts)
  [✓/✗] Framing (1pt)
  [✓/✗] Weakness (3pts)
  [✓/✗] Pedestal (2pts)
  [✓/✗] Ask (2pts)

Suggestion: [improvement tip based on missing elements]

Next: Track outcome with /rate-message [Name] replied/no-reply
```

---

## The 5-Element Framework

| Element | Weight | Description |
|---------|--------|-------------|
| **Vision** | 2pts | Human problem stated without product pitch |
| **Framing** | 1pt | "not selling/pitching" disclaimer |
| **Weakness** | 3pts | Admits confusion or asks for help (MOST IMPORTANT) |
| **Pedestal** | 2pts | Specific reason why THEM |
| **Ask** | 2pts | Clear question, preferably binary |

**Modifiers:**
- Pattern interrupt opener: +0.5
- Specific reference to their content: +1
- Generic flattery ("that's wild", "impressive"): -2
- Link in first message: -2
- Too long (5+ paragraphs): -1

---

## Error Handling

| Error | Response |
|-------|----------|
| Lead not found | Offer to create basic entry or suggest /add-lead |
| Can't parse platform | Ask user to specify (LinkedIn/Email/Twitter) |
| Message too short | Ask for full message text |
| Config missing | Tell user to run /setup-crm |
