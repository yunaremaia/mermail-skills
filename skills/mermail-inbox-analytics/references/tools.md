# Tools

This skill uses read-only queries against the shared `mermail` MCP server. Tool ownership is shared — no unique tools.

## Conventions

- Pass structured arguments as **native JSON objects**. Never stringify an object into a string field such as `query`.
- Use the exact tool identifier exposed by the current host (for example `list_emails` or a host-qualified form like `Mermail:list_emails`). Do not manually add, strip, or invent prefixes inconsistently.
- Prefer mailbox `public_id` as `mailboxId` when the list tools return it.
- All tools in this skill are **read-only** — no external-effect or destructive operations.

## Tool Reference

| Tool | Purpose | Risk |
|------|---------|------|
| `list_mailboxes` | List all accessible mailboxes with their `public_id`, email, and status | read |
| `list_emails` | Fetch paginated email list with filtering (sender, date range, thread, etc.) | read |
| `get_email` | Fetch full email details including headers, body, attachments | read |
| `get_thread` | Fetch all emails in a thread by `threadId` | read |

## Query Patterns

### List all mailboxes
```json
{
  "tool": "list_mailboxes",
  "arguments": {}
}
```

### List emails (paginated, sorted by date)
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

### Filter by sender
```json
{
  "tool": "list_emails",
  "arguments": {
    "mailboxId": "mbx_abc123",
    "sender": "newsletter@example.com",
    "limit": 100
  }
}
```

### Date range filter
```json
{
  "tool": "list_emails",
  "arguments": {
    "mailboxId": "mbx_abc123",
    "after": "2026-07-01T00:00:00Z",
    "before": "2026-08-01T00:00:00Z",
    "limit": 1000
  }
}
```

### Fetch single email
```json
{
  "tool": "get_email",
  "arguments": {
    "emailId": "eml_xyz789"
  }
}
```

### Fetch thread
```json
{
  "tool": "get_thread",
  "arguments": {
    "threadId": "thr_abc123"
  }
}
```

## Response Shapes (Reference)

### Mailbox
```typescript
interface Mailbox {
  public_id: string;      // e.g., "mbx_abc123"
  email: string;          // e.g., "user@domain.com"
  name?: string;          // display name
  enabled: boolean;
  receiving: boolean;
  created_at: string;
}
```

### Email
```typescript
interface Email {
  id: string;             // e.g., "eml_xyz789"
  thread_id: string;      // e.g., "thr_abc123"
  mailbox_id: string;
  sender: { email: string; name?: string };
  recipients: { to: string[]; cc: string[]; bcc: string[] };
  subject: string;
  body_text?: string;
  body_html?: string;
  date: string;           // ISO 8601
  has_attachments: boolean;
  attachments?: Array<{
    filename: string;
    mime_type: string;
    size: number;
  }>;
  labels: string[];
  folder: string;
}
```

### Thread
```typescript
interface Thread {
  id: string;
  emails: Email[];
  participant_count: number;
  message_count: number;
  first_date: string;
  last_date: string;
}
```

## Analytics Computation Notes

All computations happen client-side in the agent:

1. **Sender frequency**: Group emails by `sender.email`, count occurrences
2. **Thread grouping**: Group by `thread_id`, compute length, participants
3. **Time histograms**: Extract hour/day from `date` field (ISO 8601)
4. **Attachment stats**: Count `has_attachments`, aggregate by `mime_type`
5. **Reply detection**: Compare `sender.email` against mailbox `email` to classify sent vs received
6. **Response latency**: For each thread, find first email from mailbox owner after initial received email

All outputs are aggregated — no individual email content exposed.