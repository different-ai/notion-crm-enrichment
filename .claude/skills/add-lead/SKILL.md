---
name: add-lead
description: Add a new lead to your Notion CRM with PULL analysis. You provide the context, the skill structures it and creates the entry.
---

# Add Lead - Create Enriched Lead Entries

You add new leads to the CRM with structured context for personalized outreach.

## Input Format

User will say something like:
- "Sam Altman from OpenAI"
- "Add lead: Jane Doe, CEO at TechCorp"
- "Sarah Chen, just raised Series A at Acme"

---

## Step 1: Load Configuration

Read `.claude/config/workspace.json` to get database collection URLs.

**If file doesn't exist:** Tell user to run `/setup-crm` first.

---

## Step 2: Check for Duplicates

Search the Leads database for the person's name using `mcp__notion__notion-search`:

```
query: [person_name]
data_source_url: [leads collection URL from config]
```

**If found:** Ask user if they want to update the existing entry or skip.

---

## Step 3: Gather Context from User

Ask the user for key information (or extract from their input):

1. **Basic Info**
   - Full name
   - Company
   - Role (if known)
   - LinkedIn URL (if available)

2. **PULL Analysis** (prompt user if not provided)
   - **P (Project):** What are they working on?
   - **U (Unavoidable):** Why is this urgent for them?
   - **L (Looking at):** What alternatives exist?
   - **L (Limitation):** Why do alternatives fall short?

3. **Hooks** (specific angles for outreach)
   - Recent news, funding, announcements
   - Quotes from interviews or posts
   - Shared connections or context

---

## Step 4: Create Lead in Notion

Use `mcp__notion__notion-create-pages`:

```json
{
  "parent": {
    "type": "data_source_id",
    "data_source_id": "[leads collection ID from config]"
  },
  "pages": [{
    "properties": {
      "Name": "[Full Name]",
      "Company": "[Company Name]",
      "LinkedIn": "[LinkedIn URL if provided]",
      "Status": "New",
      "ICP": "[Best matching segment]",
      "Context": "[One-paragraph summary]",
      "PULL Analysis": "[Formatted PULL analysis]"
    }
  }]
}
```

---

## Context Field Format

```
[Role] at [Company] - [Key differentiator or recent event]

PULL:
• P: [Project they're working on]
• U: [Why it's urgent]
• L: [What they're looking at]
• L: [Why those fall short]

Hooks:
• [Specific angle 1]
• [Specific angle 2]
```

---

## Step 5: Report Results

```
Added: [Name] ([Company])

Status: New
ICP: [Segment]

PULL Summary:
• Project: [brief]
• Urgency: [brief]

Next: /draft-message [Name]
```

---

## Example Flow

**User:** Add lead Sarah Chen, CEO at TechCorp, just raised $10M Series A

**Skill Response:**

```
I'll create a lead for Sarah Chen. Let me gather some context:

What's Sarah working on right now? (Project)
> [User provides or skill suggests based on funding]

Why is this urgent for them? (Unavoidable)
> [User provides or skill suggests: "Post-raise pressure to deploy capital efficiently"]

What alternatives are they likely using? (Looking at)
> [User provides or skill suggests: "Traditional banks, Mercury"]

Why do those fall short? (Limitation)
> [User provides or skill suggests: "Low yield, not optimized for startup cash"]
```

Then creates the lead with structured PULL analysis.

---

## Quality Rules

1. **User provides context:** Don't make up information
2. **Structure it well:** Format PULL analysis consistently
3. **Check duplicates:** Always search before creating
4. **Keep it actionable:** Context should enable personalized outreach
