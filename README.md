# Notion CRM Enrichment

AI-powered outreach CRM that lives in Notion. Add leads, draft messages, track outcomes, and learn what works.

## The Flow

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           NOTION CRM ENRICHMENT - FULL FLOW                         │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│                              ┌──────────────────┐                                   │
│                              │   NOTION (Hub)   │                                   │
│                              ├──────────────────┤                                   │
│                              │ • Leads DB       │                                   │
│                              │ • Messages DB    │                                   │
│                              │ • Insights Page  │◄─────────────────────────┐        │
│                              └────────┬─────────┘                          │        │
│                                       │                                    │        │
│         ┌─────────────────────────────┼─────────────────────────────┐      │        │
│         │                             │                             │      │        │
│         ▼                             ▼                             ▼      │        │
│  ┌─────────────┐              ┌─────────────┐              ┌─────────────┐ │        │
│  │ /add-lead   │              │/draft-message│              │ /update-crm │ │        │
│  │             │              │             │              │             │ │        │
│  │ 1. Research │   writes     │ 1. READ     │   writes     │ 1. Find     │ │        │
│  │    (manual) │──────────►   │    Lead     │              │    lead     │ │        │
│  │ 2. PULL     │   Lead to    │ 2. READ ◄───────────────── │ 2. Log msg  │ │        │
│  │    analysis │   Notion     │    Insights │  reads       │ 3. Rate msg │ │        │
│  │ 3. Create   │              │ 3. Draft    │  insights    │ 4. Update   │ │        │
│  │    Lead     │              │    message  │              │    status   │ │        │
│  └─────────────┘              └──────┬──────┘              └──────┬──────┘ │        │
│                                      │                            │        │        │
│                                      ▼                            │        │        │
│                               ┌─────────────┐                     │        │        │
│                               │   USER      │                     │        │        │
│                               │             │                     │        │        │
│                               │ • Reviews   │                     │        │        │
│                               │ • Edits     │                     │        │        │
│                               │ • Sends     │─────────────────────┘        │        │
│                               │   manually  │                              │        │
│                               └──────┬──────┘                              │        │
│                                      │                                     │        │
│                                      ▼                                     │        │
│                               ┌─────────────┐                              │        │
│                               │/rate-message│                              │        │
│                               │             │                              │        │
│                               │ 1. Log      │                              │        │
│                               │    outcome  │                              │        │
│                               │ 2. Analyze  │                              │        │
│                               │    patterns │                              │        │
│                               │ 3. UPDATE ──┼──────────────────────────────┘        │
│                               │    Insights │  writes new learnings                 │
│                               └─────────────┘  to Notion Insights Page              │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

## The Learning Loop

```
                    ┌────────────────────────────────────────┐
                    │                                        │
                    ▼                                        │
            ┌───────────────┐                                │
            │ Insights Page │ (Notion)                       │
            │               │                                │
            │ • What works  │                                │
            │ • What fails  │                                │
            │ • Templates   │                                │
            └───────┬───────┘                                │
                    │ READ                                   │ WRITE
                    ▼                                        │
            ┌───────────────┐      ┌───────────────┐   ┌─────┴─────────┐
            │/draft-message │ ───► │    USER       │ ─►│ /rate-message │
            │               │      │ sends message │   │               │
            │ applies       │      └───────────────┘   │ logs outcome  │
            │ learnings     │                          │ finds patterns│
            └───────────────┘                          └───────────────┘
```

## Quick Start

### 1. Prerequisites

- [Claude Code](https://claude.ai/code) CLI installed
- Notion account (free tier works)

### 2. Configure Notion MCP

Add the Notion MCP server to your Claude Code:

```bash
claude mcp add notion --url https://mcp.notion.com/mcp
```

When prompted, authenticate with your Notion account.

### 3. Clone and Setup

```bash
git clone https://github.com/different-ai/notion-crm-enrichment
cd notion-crm-enrichment

# Start Claude Code
claude

# Run the setup wizard
> /setup-crm
```

The setup skill will:
1. ✓ Verify your Notion connection
2. ✓ Create an "Outreach CRM" page with databases
3. ✓ Create a "Feedback Insights" page for learning
4. ✓ Save configuration for other skills

## Usage

### Step 1: Add a Lead

```
/add-lead Sarah Chen, CEO at TechCorp
```

You provide the context, the skill:
- Creates a lead entry in Notion
- Prompts you for PULL analysis
- Stores hooks for personalization

### Step 2: Draft a Message

```
/draft-message Sarah Chen
```

The skill:
- Fetches Sarah's lead data from Notion
- **Reads Feedback Insights** for what's working
- Drafts a message using the 5-element framework
- Outputs a ready-to-send message

### Step 3: Log the Sent Message

```
/update-crm Sarah Chen sent on LinkedIn: "hey sarah, saw your series a announcement..."
```

The skill:
- Creates a message entry in Messages DB
- Rates your message (1-10) against the framework
- Updates Sarah's status to "Contacted"
- Links the message to the lead

### Step 4: Rate the Outcome

```
/rate-message Sarah replied: "thanks for reaching out, let's chat"
```

The skill:
- Updates message result to "Reply"
- Analyzes what worked in this message
- **Writes to Feedback Insights** for future drafts

## The 5-Element Framework

Every message is rated against these elements:

| Element | Points | What It Means |
|---------|--------|---------------|
| **Vision** | 2 | Human problem stated without product pitch |
| **Framing** | 1 | "not selling" disclaimer |
| **Weakness** | 3 | Admits confusion or asks for help (MOST IMPORTANT) |
| **Pedestal** | 2 | Specific reason why THEM |
| **Ask** | 2 | Clear question, preferably binary |

**Modifiers:**
- Pattern interrupt opener: +0.5
- Specific reference to their content: +1
- Generic flattery ("love what you're building"): -2
- Link in first message: -2

## The PULL Framework

Leads are analyzed through this lens:

| Letter | Meaning | Question |
|--------|---------|----------|
| **P** | Project | What are they actively working on? |
| **U** | Unavoidable | Why can't they ignore this? |
| **L** | Looking at | What alternatives are they considering? |
| **L** | Limitation | Why do those alternatives fall short? |

## Notion Structure

After `/setup-crm`, you'll have:

```
Outreach CRM (Page)
├── Leads (Database)
│   ├── Name, Company, LinkedIn, Status
│   ├── Context, PULL Analysis
│   └── Last Contact
│
├── Messages (Database)
│   ├── Title, Platform, Date
│   ├── Result (Pending/Reply/No Reply)
│   ├── Rating (1-10)
│   └── Content (full message text)
│
└── Feedback Insights (Page)
    ├── ## What's Working
    ├── ## What's Failing
    ├── ## Proven Templates
    └── ## Stats
```

## File Structure

```
.
├── README.md
├── .gitignore
├── .claude/
│   ├── skills/
│   │   ├── setup-crm/
│   │   │   └── SKILL.md       # Setup wizard
│   │   ├── add-lead/
│   │   │   └── SKILL.md       # Add leads
│   │   ├── draft-message/
│   │   │   └── SKILL.md       # Draft messages
│   │   ├── update-crm/
│   │   │   └── SKILL.md       # Log messages
│   │   └── rate-message/
│   │       └── SKILL.md       # Rate outcomes
│   └── config/
│       └── workspace.json     # Generated config (gitignored)
└── sample/
    └── test-data.txt          # Sample data for testing
```

## Commands Reference

| Command | What It Does |
|---------|--------------|
| `/setup-crm` | Bootstrap wizard - run once |
| `/add-lead [Name] from [Company]` | Add a new lead to CRM |
| `/draft-message [Name]` | Draft a message for a lead |
| `/update-crm [Name] sent: "..."` | Log a sent message |
| `/rate-message [Name] replied/no-reply` | Log outcome and learn |

## License

MIT
