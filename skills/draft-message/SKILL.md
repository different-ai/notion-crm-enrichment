---
name: draft-message
description: Draft a personalized outreach message for a lead. Fetches lead data from Notion, reads Feedback Insights for what's working, and applies the 5-element framework.
---

# Draft Message - Personalized Outreach That Gets Replies

You draft cold outreach messages using the lead's context from Notion and learnings from past messages.

## Input Format

User will say something like:
- "/draft-message Sarah Chen"
- "Draft a message for Sam Altman"
- "Write outreach for Jane at TechCorp"

---

## Step 1: Load Configuration

Read `.claude/config/workspace.json` to get:
- Leads database collection URL
- Feedback Insights page ID

**If file doesn't exist:** Tell user to run `/setup-crm` first.

---

## Step 2: Fetch Lead Data

Search for the lead using `mcp__notion__notion-search`:

```
query: [lead_name]
data_source_url: [leads collection URL from config]
```

Then fetch the full lead page to get:
- Name, Company
- Context
- PULL Analysis
- LinkedIn URL
- Any previous messages (check status)

**If not found:** Tell user to run `/add-lead [Name]` first.

---

## Step 3: Read Feedback Insights (CRITICAL)

Fetch the Feedback Insights page using `mcp__notion__notion-fetch`:

```
id: [feedback_insights_page_id from config]
```

Extract:
- **What's Working:** Patterns to apply
- **What's Failing:** Anti-patterns to avoid
- **Proven Templates:** Starting points for each element

**If empty or no data:** Use default templates from this skill.

---

## Step 4: Apply the 5-Element Framework

**Every message MUST contain these 5 elements:**

### 1. VISION (2 points)
The human problem WITHOUT mentioning your product.
- ✅ "startups leave money sitting in checking accounts"
- ❌ "we offer 4% APY"

### 2. FRAMING (1 point)
Disarm them - you're NOT selling.
- ✅ "not trying to pitch you anything"
- ✅ "still figuring this out"

### 3. WEAKNESS (3 points - MOST IMPORTANT)
Admit you're STUCK or CONFUSED on something.
- ✅ "honestly stuck on whether founders even care about yield"
- ✅ "trying to figure out if this is a real pain point"

### 4. PEDESTAL (2 points)
Why THEM specifically - not generic flattery.
- ✅ "you raised your series a last year so you've been through this"
- ❌ "love what you're building"

### 5. ASK (2 points)
Ask for HELP, not a call. Make it easy to answer.
- ✅ "is it 'park it and forget it' or actively optimizing?"
- ❌ "would love 15 min of your time"

---

## Step 5: Apply Style Rules

### Format
- **Lowercase everything** (except proper nouns if needed)
- **Plain text only** - no formatting, no bullets
- **Short, punchy sentences**
- **Sign off:** Just "best," or your name

### Subject Line (if email)
- 2-4 words, lowercase
- Good: `quick q`, `random`, `treasury stuff`
- Bad: `Quick Question About Treasury Management`

### Anti-Patterns (NEVER DO)
- No links in first message
- No "15 min call" or "coffee"
- No "feedback" (use "help" or "advice")
- No generic flattery ("love what you're building")

---

## Step 6: Craft the Message

Use the PULL analysis from the lead to personalize:

```
subject: [2-4 words lowercase]

[optional: what you're working on - establishes credibility]

anyways. [or similar transition]

[VISION: the human problem]
[FRAMING: not selling]
[WEAKNESS: what you're stuck on - be specific]

[PEDESTAL: why them - use their PULL context]

[ASK: easy question, binary if possible]

best,
[name]
```

---

## Step 7: Self-Rate Before Presenting

Score the draft against the framework:

| Element | Score | Notes |
|---------|-------|-------|
| Vision | /2 | |
| Framing | /1 | |
| Weakness | /3 | |
| Pedestal | /2 | |
| Ask | /2 | |
| **Total** | /10 | |

**Modifiers:**
- Pattern interrupt: +0.5
- Specific reference: +1
- Generic flattery: -2
- Link included: -2

**If score < 6:** Revise before presenting.

---

## Step 8: Present Draft

Output format:

```
## Draft for [Name] at [Company]

**Platform:** [LinkedIn / Email]
**Subject:** [subject line]

---

[The actual message]

---

**Self-Rating:** [X]/10

| Element | Score |
|---------|-------|
| Vision | [/2] |
| Framing | [/1] |
| Weakness | [/3] |
| Pedestal | [/2] |
| Ask | [/2] |

**Why this works:**
- [Key strength from PULL connection]
- [What makes it memorable]

**Next steps:**
1. Review and edit as needed
2. Send manually on [Platform]
3. Run: /update-crm [Name] sent: "[your message]"
```

---

## Example Output

```
## Draft for Sarah Chen at TechCorp

**Platform:** LinkedIn
**Subject:** quick q

---

hey sarah

been thinking about how startups manage their cash lately.

not trying to pitch you anything - genuinely just trying to understand this better.

honestly stuck on whether founders even think about yield on their operating cash or if it's just "park it at mercury and forget it." like is this actually a pain point or am i overthinking it?

you just closed your series a so you've probably had to make this exact call recently.

curious - do you actively think about optimizing your treasury or is it more set-and-forget? even a one-liner would help.

best,
ben

---

**Self-Rating:** 8/10

| Element | Score |
|---------|-------|
| Vision | 2/2 |
| Framing | 1/1 |
| Weakness | 3/3 |
| Pedestal | 2/2 |
| Ask | 2/2 |

Modifiers: -2 (none applied)

**Why this works:**
- Connects to her recent Series A (PULL: Project)
- Binary ask makes it easy to respond
- Genuine curiosity, not sales pitch

**Next steps:**
1. Review and edit as needed
2. Send on LinkedIn
3. Run: /update-crm Sarah Chen sent on LinkedIn: "[your message]"
```

---

## Error Handling

| Error | Response |
|-------|----------|
| Lead not found | Suggest `/add-lead [Name]` |
| No PULL analysis | Draft with available context, note weakness |
| No Feedback Insights | Use default templates |
| Score too low | Auto-revise and present improved version |
