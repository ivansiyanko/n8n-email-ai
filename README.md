# AI Email Classifier & Auto-Responder

![n8n](https://img.shields.io/badge/n8n-workflow-FF6D5A?logo=n8n&logoColor=white)
![AI Powered](https://img.shields.io/badge/AI-GPT--4o--mini-412991?logo=openai&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)

An n8n workflow that automatically reads your inbox, classifies every email by category, priority, and sentiment using GPT-4o-mini, then takes the right action — auto-reply, draft a response, forward to the right team, flag for review, or archive.

## What It Does

```
┌──────────────┐    ┌────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│  📬 Check    │───▶│  📧 Fetch      │───▶│  🧹 Parse &      │───▶│  🤖 AI Classify  │
│  Inbox (5m)  │    │  Unread Emails │    │  Prepare Email   │    │  with GPT-4o     │
└──────────────┘    └────────────────┘    └──────────────────┘    └────────┬─────────┘
                                                                           │
                    ┌──────────────────────────────────────────────────────┘
                    ▼
          ┌──────────────────┐    ┌──────────────────────────────────────────┐
          │  📊 Log to       │    │  🔀 Route by Suggested Action            │
          │  Google Sheets   │    │                                          │
          └──────────────────┘    │  ├─ ✅ Auto Reply → Send email + log     │
                                  │  ├─ ✏️ Draft Reply → Slack for review    │
                                  │  ├─ 📨 Forward → Route to team channel   │
                                  │  ├─ 🚩 Flag → Slack review alert         │
                                  │  └─ 🗑️ Archive → Log and skip           │
                                  └──────────────────────────────────────────┘
```

## Email Categories

| Category | Description | Example |
|----------|-------------|---------|
| `meeting` | Calendar invites, scheduling | "Can we meet Thursday at 2pm?" |
| `support` | Help requests, bug reports | "I'm having trouble logging in" |
| `sales` | Inquiries, partnerships, pricing | "What are your enterprise plans?" |
| `billing` | Invoices, payment issues | "My last payment didn't go through" |
| `newsletter` | Marketing, promotional content | Weekly digest from a SaaS tool |
| `spam` | Unwanted solicitation, phishing | "You've won a $1000 gift card!" |
| `internal` | Team updates, HR notices | "All-hands meeting next Monday" |
| `urgent` | Security alerts, outages | "Production server is down" |
| `other` | Anything else | Miscellaneous |

## Smart Actions

The AI doesn't just classify — it decides what to do:

- **Auto Reply** — Sends a professional automated response for routine emails (meeting confirmations, acknowledgments). Logs to Slack.
- **Draft Reply** — AI writes a draft response and posts it to Slack `#email-drafts` for a human to review and send.
- **Forward to Team** — Routes to the right Slack channel (`#support-inbox`, `#sales-inbox`, `#billing-inbox`, `#engineering-inbox`, or `#management-inbox`).
- **Flag for Review** — Low-confidence or complex emails get flagged in `#email-review` with full context.
- **Archive** — Spam and newsletters are logged and skipped.

## Setup

### Prerequisites

- [n8n](https://n8n.io/) (self-hosted or cloud)
- OpenAI API key (GPT-4o-mini)
- IMAP email account (Gmail, Outlook, or any IMAP provider)
- SMTP credentials for sending replies
- Google Sheets API access
- Slack workspace with a bot token

### Installation

1. **Import the workflow** — Copy `workflow.json` and import it in n8n (Settings → Import Workflow)

2. **Set up credentials** in n8n:
   - **IMAP** — your email inbox credentials
   - **SMTP** — for sending auto-replies
   - **OpenAI** — your API key
   - **Google Sheets** — OAuth2 connection
   - **Slack** — Bot token with `chat:write` scope

3. **Create a Google Sheet** with these columns:

   | Date | From | Subject | Category | Priority | Sentiment | Summary | Action | Forward To | Confidence | Classified At |
   |------|------|---------|----------|----------|-----------|---------|--------|------------|------------|---------------|

4. **Set environment variables** in n8n:
   ```
   IMAP_HOST=imap.gmail.com
   IMAP_USER=your@email.com
   IMAP_PASSWORD=your-app-password
   SMTP_FROM=your@email.com
   GOOGLE_SHEET_ID=your-sheet-id
   SLACK_CHANNEL_DRAFTS=#email-drafts
   SLACK_CHANNEL_REVIEW=#email-review
   ```

5. **Create Slack channels:**
   - `#email-drafts` — for AI-drafted replies awaiting human review
   - `#email-review` — for flagged emails and archive logs
   - `#support-inbox`, `#sales-inbox`, `#billing-inbox`, `#engineering-inbox`, `#management-inbox` — for forwarded emails

6. **Activate the workflow** — it checks your inbox every 5 minutes

### Gmail Setup

If using Gmail, you'll need an App Password:

1. Go to [Google Account Security](https://myaccount.google.com/security)
2. Enable 2-Step Verification
3. Generate an App Password for "Mail"
4. Use `imap.gmail.com` (port 993) and `smtp.gmail.com` (port 587)

## Example Output

### Slack Draft Review Message

```
✏️ Draft Reply Needed
────────────────────
From: sarah@acmecorp.com
Category: sales (medium priority)

Subject: Enterprise pricing for 50 seats
Summary: Potential enterprise customer asking about volume pricing and onboarding support for their team.

Suggested Draft:
> Hi Sarah, thank you for your interest in our enterprise plan!
> For teams of 50+, we offer custom pricing with dedicated onboarding
> support. I'd love to schedule a quick call to discuss your needs.
> Would Thursday or Friday work for you?

Confidence: 0.92 | Sentiment: positive
```

### Google Sheets Log

| Date | From | Subject | Category | Priority | Sentiment | Action | Confidence |
|------|------|---------|----------|----------|-----------|--------|------------|
| 2026-04-06 | sarah@acmecorp.com | Enterprise pricing | sales | medium | positive | draft_reply | 0.92 |
| 2026-04-06 | noreply@newsletter.io | Weekly digest | newsletter | low | neutral | archive | 0.98 |
| 2026-04-06 | ops@company.com | Server alert | urgent | high | negative | flag_for_review | 0.87 |

## Customization

### Adjust Check Frequency

Edit the Schedule Trigger node to change from every 5 minutes to any interval.

### Add More Categories

Edit the system prompt in the "AI Classify Email" node to add custom categories, then add matching branches in the Switch node.

### Change the AI Model

Swap `gpt-4o-mini` for `gpt-4o` for higher accuracy (at higher cost), or use a local model via Ollama for zero API costs.

### Add More Channels

Map additional teams in the "Slack Forward to Team" node by editing the channel routing expression.

### Integrate with Other Tools

Replace Slack with Microsoft Teams, Discord, or any notification service. Replace Google Sheets with Airtable, Notion, or a database.

## Cost Estimate

With GPT-4o-mini at ~$0.15/1M input tokens:
- **100 emails/day** ≈ $0.02/day ($0.60/month)
- **500 emails/day** ≈ $0.10/day ($3.00/month)
- **1000 emails/day** ≈ $0.20/day ($6.00/month)

## Author

**Ivan Siyanko** — [siyanko.com](https://siyanko.com)

## License

MIT — see [LICENSE](LICENSE)
