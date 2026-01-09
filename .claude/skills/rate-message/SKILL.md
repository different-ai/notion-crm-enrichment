---
name: rate-message
description: Log message outcomes (reply/no reply), analyze patterns, and update Feedback Insights in Notion. This is the learning engine of the CRM.
---

# Rate Message - Learn What Works

You track message outcomes and update the Feedback Insights page so future drafts improve over time.

## Input Format

User will say something like:
- "/rate-message Sarah replied: 'thanks for reaching out'"
- "Sarah got back to me, she wants to chat"
- "No reply from Sam after a week"
- "Mark Jane's message as no reply"

---

## Step 1: Load Configuration

Read `.claude/config/workspace.json` to get:
- Messages database collection URL
- Feedback Insights page ID

**If file doesn't exist:** Tell user to run `/setup-crm` first.

---

## Step 2: Find the Message

Search Messages database for the lead using `mcp__notion__notion-search`:

```
query: [lead_name]
data_source_url: [messages collection URL from config]
```

Find the most recent message for this lead with Result = "Pending".

**If not found:** Ask if they logged the message with `/update-crm` first.

---

## Step 3: Update Message Result

Use `mcp__notion__notion-update-page` to update the message:

```json
{
  "page_id": "[message_page_id]",
  "command": "update_properties",
  "properties": {
    "Result": "[Reply / No Reply / Meeting]"
  }
}
```

Also fetch the full message content for analysis.

---

## Step 4: Update Lead Status

Find and update the lead's status based on outcome:

| Outcome | New Lead Status |
|---------|-----------------|
| Reply | Replied |
| No Reply | Contacted (no change) |
| Meeting | Meeting |

Use `mcp__notion__notion-update-page` on the lead.

---

## Step 5: Analyze the Message

If the message got a reply, analyze what worked:

### Element Analysis
Review the message against the 5-element framework:
- Which elements were present?
- Which were strongest?
- Any patterns that stand out?

### Pattern Detection
Look for:
- **Hook type:** What opened the message?
- **Weakness style:** How explicit was the vulnerability?
- **Pedestal specificity:** How personalized?
- **Ask type:** Binary vs open-ended?

### If No Reply
Note potential issues:
- Missing elements?
- Generic language?
- Too long?
- Wrong timing?

---

## Step 6: Update Feedback Insights (CRITICAL)

Fetch the current Feedback Insights page, then update it with new learnings.

### For Replies (What's Working)

Add to "What's Working" section:
```
- [Pattern]: "[Example from message]" → Reply ([Date])
```

Example:
```
- Binary ask + explicit weakness: "is it X or Y? honestly stuck on..." → Reply (2024-01-15)
```

### For No Replies (What's Failing)

Add to "What's Failing" section if you identify an anti-pattern:
```
- [Anti-pattern]: "[Example]" → No Reply ([Date])
```

### Update Stats

Update the Stats section:
```
| Metric | Value |
|--------|-------|
| Total Messages | [+1] |
| Reply Rate | [recalculate] |
| Avg Rating | [recalculate] |
```

Use `mcp__notion__notion-update-page` with `replace_content_range` to update specific sections.

---

## Step 7: Report Results

```
Logged: [Reply/No Reply] from [Name]

Message Rating: [X]/10
Lead Status: [Old] → [New]

Analysis:
[What likely worked / What might have failed]

Pattern noted:
[Key learning added to Feedback Insights]

Feedback Insights updated: [Notion URL]

---

Stats:
• Total Messages: [N]
• Reply Rate: [X]%
• This message: [Above/Below] average
```

---

## Batch Analysis

If user says "analyze my last 5 messages" or "what's working?":

1. Fetch recent messages from Messages database
2. Filter to those with outcomes (Reply or No Reply)
3. Analyze patterns across the batch:
   - Reply rate by element strength
   - Common patterns in successful messages
   - Common issues in failed messages
4. Update Feedback Insights with aggregated learnings
5. Present summary with actionable recommendations

---

## Example Flow

**User:** Sarah replied: "hey thanks for reaching out, actually yeah we've been thinking about this"

**Skill Response:**

```
Logged: Reply from Sarah Chen

Message Rating: 8/10
Lead Status: Contacted → Replied

Analysis:
✓ Strong weakness element ("honestly stuck on...")
✓ Binary ask made it easy to respond
✓ Specific pedestal (referenced her Series A)
→ Her response confirms the pain point is real

Pattern noted:
Added to "What's Working":
- Binary ask + Series A reference → Reply (2024-01-15)

Feedback Insights updated: [Notion URL]

---

Stats:
• Total Messages: 12
• Reply Rate: 33% (4/12)
• This message: Above average (rated 8/10, avg 6.5)

Next: Would you like to book a meeting? Update with:
/rate-message Sarah meeting booked
```

---

## Error Handling

| Error | Response |
|-------|----------|
| Message not found | Suggest `/update-crm` to log first |
| Multiple pending messages | Ask which one to update |
| Insights page missing | Create minimal structure and continue |
| Can't parse outcome | Ask for clarification (reply/no reply/meeting) |
