# Demo Script: Mermail Inbox Analytics Skill

## Overview
This script demonstrates the mermail-inbox-analytics skill for the Mermail bounty.
The skill analyzes email patterns, volume trends, top senders, and conversation insights.

## Demo Structure (2-5 minute video)

### 1. Introduction (0:00-0:30)
- Brief explanation of what the skill does
- Show the skill name and description
- Mention it's a read-only analytics skill for Mermail

### 2. Setup & Configuration (0:30-1:00)
- Show the skill configuration in SKILL.md
- Highlight that it requires MERMAIL_API_KEY
- Show the tools it uses (list_mailboxes, list_emails, get_email, get_thread)

### 3. Demonstration (1:00-3:30)
#### Part A: Mailbox Discovery (1:00-1:30)
- Demonstrate list_mailboxes tool call
- Show example response with mailbox details
- Explain how user selects target mailbox

#### Part B: Email Analysis (1:30-2:30)
- Demonstrate list_emails call with parameters
- Show fetching recent emails (last 30 days, limit 500)
- Explain the read-only nature of the queries

#### Part C: Analytics Computation (2:30-3:00)
- Show how the skill computes:
  * Sender frequency analysis
  * Thread grouping and analysis
  * Time-of-day patterns
  * Attachment statistics
  * Reply rate calculation

#### Part D: Results Presentation (3:00-3:30)
- Show example output format:
  * Volume report (daily/weekly counts)
  * Top senders table
  * Thread insights
  * Attachment summary
  * Actionable recommendations

### 4. Use Cases & Benefits (3:30-4:00)
- "Analyze my inbox for the last 30 days — who emails me most?"
- "What time of day do I get the most emails?"
- "Show me threads with >5 replies — which ones are still open?"
- "Attachment stats: how many PDFs vs images, from whom?"

### 5. Security & Privacy (4:00-4:30)
- Emphasize read-only operations
- No external effects or data modification
- Aggregated outputs only - no PII exposure
- Follows Mermail security guidelines

### 6. Conclusion (4:30-5:00)
- Summary of skill capabilities
- Call to action: Try the skill for your own inbox analysis
- Thank you and contact information

## Example Data Structures

### Mailbox Response:
```json
{
  "public_id": "mbx_abc123",
  "email": "user@example.com",
  "name": "Personal Mailbox",
  "enabled": true,
  "receiving": true,
  "created_at": "2026-01-15T10:30:00Z"
}
```

### Email Response:
```json
{
  "id": "eml_xyz789",
  "thread_id": "thr_abc123",
  "mailbox_id": "mbx_abc123",
  "sender": {
    "email": "sender@example.com",
    "name": "John Sender"
  },
  "subject": "Meeting tomorrow at 3pm",
  "body_text": "Hi, just confirming our meeting...",
  "date": "2026-08-28T14:30:00Z",
  "has_attachments": false,
  "labels": ["important", "work"]
}
```

### Analytics Output Example:
```
📊 INBOX ANALYTICS REPORT
=========================

📧 VOLUME METRICS
- Total emails analyzed: 1,245
- Daily average: 41.5 emails/day
- Peak hour: 10:00 AM (23% of volume)
- Busiest day: Wednesday

👥 TOP SENDERS
1. newsletter@example.com - 324 emails (26.0%)
2. team@company.com - 189 emails (15.2%)
3. notifications@service.com - 156 emails (12.5%)

🧵 THREAD INSIGHTS
- Total threads: 412
- Average thread length: 3.2 emails
- Longest thread: 24 emails (Project discussion)
- Threads with >5 replies: 23 (5.6%)

📎 ATTACHMENT STATS
- Emails with attachments: 234 (18.8%)
- Most common types: PDF (45%), JPEG (30%), PNG (15%)
- Total attachment size: 2.4 GB

💡 RECOMMENDATIONS
- Create filter for newsletter@example.com → "Newsletters" label
- Consider unsubscribing from low-value senders
- Set up triage rules for high-volume senders
```

## Technical Implementation Notes
- All computations happen client-side in the agent
- Uses structured JSON arguments for tool calls
- Follows Mermail MCP server conventions
- No external-effect tools - purely read-only
- Output is aggregated statistics only
