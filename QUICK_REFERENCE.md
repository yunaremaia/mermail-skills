# QUICK REFERENCE: Mermail Inbox Analytics Demo

## FLOW
1. INTRO (0:00-0:30) - What problem it solves
2. SETUP (0:30-1:00) - API key, read-only tools
3. DEMO (1:00-3:30) - Mailbox → Fetch → Analyze → Results
4. SECURITY (3:30-4:00) - Privacy guarantees
5. CONCLUSION (4:00-5:00) - Summary + CTA

## KEY POINTS TO EMPHASIZE
- Read-only: list_mailboxes, list_emails, get_email, get_thread ONLY
- No external effects - safe to use
- Privacy: aggregates only, no email content exposed
- Natural language questions → structured insights
- Actionable recommendations for inbox optimization

## EXAMPLE QUERIES TO SHOW
1. list_mailboxes: {}
2. list_emails: {mailboxId: "mbx_abc123", limit: 500, sort: date DESC}
3. (Analysis happens client-side)
4. Results: Volume, top senders, threads, attachments, recommendations

## SAMPLE OUTPUT TO DISPLAY
📊 INBOX ANALYTICS REPORT
📧 Volume: 1,245 emails (41.5/day)
👥 Top sender: newsletter@example.com (324 emails)
🧵 Threads: 412 total, avg 3.2 emails
📎 Attachments: 234 emails (PDF 45%, JPEG 30%)
💡 Tip: Filter newsletters to reduce inbox noise

## EXAMPLE PROMPTS TO MENTION
- "Who emails me most in the last 30 days?"
- "What's my peak email hour?"
- "Show long-running threads (>5 replies)"
- "Attachment types and sizes by sender"
