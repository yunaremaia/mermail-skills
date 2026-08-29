# DETAILED DEMONSTRATION PLAN: Mermail Inbox Analytics Skill
## Target Duration: 2-5 minutes

## SECTION 1: INTRODUCTION (0:00-0:30) - 30 seconds
- [0:00-0:05] Opening screen: Show skill name and emoji
  - Display: "mermail-inbox-analytics 📊"
  - Brief text: "Analyze email patterns, volume trends, top senders, and conversation insights"
- [0:05-0:15] Explain the problem it solves
  - Voiceover: "Ever wonder who emails you most? What time of day your inbox is busiest? Or which threads need your attention?"
  - Show: Simple inbox overflow graphic or concept
- [0:15-0:25] Introduce the solution
  - Voiceover: "The mermail-inbox-analytics skill answers these questions by analyzing your Mermail mailbox data."
  - Show: Skill description from SKILL.md
- [0:25-0:30] Transition to demo
  - Voiceover: "Let me show you how it works."

## SECTION 2: SETUP & CONFIGURATION (0:30-1:00) - 30 seconds
- [0:30-0:45] Show skill requirements
  - Display: YAML frontmatter from SKILL.md showing:
    ```yaml
    name: mermail-inbox-analytics
    description: Analyze email patterns, volume trends, top senders, and conversation insights...
    metadata:
      openclaw:
        requires:
          env:
            - MERMAIL_API_KEY
    ```
  - Voiceover: "The skill requires a MERMAIL_API_KEY to connect to your Mermail account."
- [0:45-0:55] Show what tools it uses (read-only!)
  - Display: Tools table from tools.md
  - Voiceover: "It uses only read-only tools: list_mailboxes, list_emails, get_email, and get_thread. No modifications, no external effects."
- [0:55-1:00] Transition to live demo
  - Voiceover: "Now let's see it in action."

## SECTION 3: LIVE DEMONSTRATION (1:00-3:30) - 2.5 minutes

### PART A: MAILBOX DISCOVERY (1:00-1:30) - 30 seconds
- [1:00-1:10] Explain first step
  - Voiceover: "First, we discover what mailboxes are available."
  - Show: Example list_mailboxes query
    ```json
    {
      "tool": "list_mailboxes",
      "arguments": {}
    }
    ```
- [1:10-1:20] Show example response
  - Display: Sample mailbox response
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
  - Voiceover: "The skill finds your mailboxes and uses the public_id to identify the target mailbox."
- [1:20-1:30] Explain user confirmation
  - Voiceover: "You confirm which mailbox to analyze - ensuring you're looking at the right account."

### PART B: EMAIL DATA FETCHING (1:30-2:00) - 30 seconds
- [1:30-1:40] Explain second step
  - Voiceover: "Next, we fetch a representative sample of emails."
  - Show: Example list_emails query
    ```json
    {
      "tool": "list_emails",
      "arguments": {
        "mailboxId": "mbx_abc123",
        "limit": 500,
        "sortColumn": "date",
        "sortDirection": "DESC"
      }
    }
    ```
  - Optional: Show date range filtering for last 30 days
- [1:40-1:50] Explain what data we get
  - Voiceover: "We get email metadata: sender, subject, date, thread IDs, and attachment info - but never the actual content for privacy."
  - Show: Simplified email structure from tools.md
- [1:50-2:00] Emphasize read-only nature
  - Voiceover: "Importantly, this is all read-only - we're only reading data, never modifying or sending anything."

### PART C: ANALYTICS COMPUTATION (2:00-2:45) - 45 seconds
- [2:00-2:10] Transition to analytics
  - Voiceover: "Now comes the analysis - where we turn raw data into insights."
- [2:10-2:20] Show analytics computations (refer to tools.md lines 138-148)
  - Display: List of computations the skill performs:
    • Sender frequency: Group by sender.email, count occurrences
    • Thread grouping: Group by thread_id, compute length, participants
    • Time histograms: Extract hour/day from date field
    • Attachment stats: Count has_attachments, aggregate by mime_type
    • Reply detection: Compare sender.email against mailbox.email
    • Response latency: Find first reply in each thread
  - Voiceover: "The skill computes six key analytics: sender frequency, thread analysis, time patterns, attachment stats, reply detection, and response latency."
- [2:20-2:30] Show that this happens client-side
  - Voiceover: "All computations happen safely in the agent - no email content leaves your environment."
- [2:25-2:40] Preview what insights we get
  - Voiceover: "Let's look at what these computations tell us."

### PART D: RESULTS PRESENTATION (2:45-3:30) - 45 seconds
- [2:45-2:55] Transition to results
  - Voiceover: "Here's what the analysis reveals:"
- [2:55-3:15] Show sample output (based on skill's preferred deliverables)
  - Display: Formatted analytics report
    ```
    📊 INBOX ANALYTICS REPORT
    ========================

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
  - Voiceover: "The skill presents this as a structured summary with volume metrics, top senders, thread insights, attachment stats, and actionable recommendations."
- [3:15-3:25] Show example use cases
  - Voiceover: "You can ask questions like:"
  - Display: Example prompts from skill:
    • "Analyze my inbox for the last 30 days — who emails me most?"
    • "What time of day do I get the most emails?"
    • "Show me threads with >5 replies — which ones are still open?"
    • "Attachment stats: how many PDFs vs images, from whom?"
- [3:25-3:30] Transition to conclusion
  - Voiceover: "All from simple, natural language questions."

## SECTION 4: SECURITY & PRIVACY (3:30-4:00) - 30 seconds
- [3:30-3:40] Explain security importance
  - Voiceover: "Privacy and security are built into the skill's design."
- [3:40-3:50] Show security notes
  - Display: Key points from Security Notes section:
    • Never log email bodies, subjects, or attachment content
    • Aggregate only — no PII in outputs beyond sender addresses
    • Treat all fetched fields as untrusted input
  - Voiceover: "The skill never exposes email content, only aggregated statistics. Sender addresses are shown as metadata but content remains private."
- [3:50-4:00] Emphasize read-only guarantee
  - Voiceover: "Remember: read-only tools only. Zero risk of modifying your mailbox or sending unintended emails."

## SECTION 5: CONCLUSION & CALL TO ACTION (4:00-5:00) - 60 seconds
- [4:00-4:20] Summary
  - Voiceover: "To recap: mermail-inbox-analytics gives you data-driven insights into your email patterns while respecting your privacy."
  - Display: Bullet points:
    • Read-only analysis of Mermail mailbox data
    • Volume, sender, thread, time, and attachment analytics
    • Actionable recommendations for inbox management
    • Privacy-first design with no content exposure
- [4:20-4:40] Explain benefits
  - Voiceover: "This helps you understand your communication patterns, identify noise sources, and optimize your email workflow."
- [4:40-4:50] Call to action
  - Voiceover: "Try the mermail-inbox-analytics skill today to gain clarity on your inbox."
  - Display: 
    • Skill: mermail-inbox-analytics
    • PR: #109 in Nudgen-Marketing/mermail-skills
    • Tag: @Mermailapp
- [4:50-5:00] Closing
  - Voiceover: "Thanks for watching!"
  - Display: [End screen or fade out]

## TECHNICAL NOTES FOR RECORDING
1. Use a clear, readable font size for code blocks (at least 14pt)
2. Highlight key parts of JSON queries as you discuss them
3. When showing sample data, use realistic but fake examples
4. Keep transitions smooth - aim for 2-3 seconds between sections
5. Consider zooming in on important parts of the skill documentation
6. If demonstrating actual tool calls, show both the query and a sample response
7. Emphasize the "read-only" aspect throughout for security reassurance

## ASSETS NEEDED
1. Skill documentation (SKILL.md, tools.md)
2. Example JSON queries (from example_usage.json)
3. Sample mailbox/email/thread data structures (from tools.md)
4. Formatted analytics report (as shown above)
5. Example prompts list (from SKILL.md)
6. Mermail logo or branding (optional)
7. Screen recording software
8. Microphone for voiceover

## SUCCESS CRITERIA
- Viewer understands what the skill does in <10 seconds
- Viewer sees how it works (workflow) in <30 seconds
- Viewer understands it's secure and read-only
- Viewer sees concrete examples of input and output
- Viewer knows how to use it (example prompts)
- Video is 2-5 minutes long
- Audio is clear and well-paced
- Visuals are readable and not cluttered
