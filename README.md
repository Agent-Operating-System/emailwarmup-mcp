# BobWork Email Warm-up MCP server

<img src="logo.png" width="96" alt="BobWork Email Warm-up">

Remote MCP server for [BobWork Email Warm-up](https://emailwarmup.bobwork.ai/): warm up any mailbox (Gmail, Google Workspace, Microsoft 365, Zoho, or any IMAP/SMTP), read inbox placement and spam rate, check SPF/DKIM/DMARC, and run cold email campaigns from your own mailboxes. Built so an AI agent can do the whole job for its human: sign up, connect mailboxes, start sending, and get a checkout link when payment is needed.

This repository holds the docs and the registry file. The server itself is hosted, so there is nothing to install or run.

- Endpoint: `https://app-emailwarmup.bobwork.ai/mcp` (Streamable HTTP)
- Auth: `Authorization: Bearer bwk_...`
- REST API: `https://app-emailwarmup.bobwork.ai/api/v1` ([OpenAPI](https://emailwarmup.bobwork.ai/openapi.json))
- Agent guide: [llms-full.txt](https://emailwarmup.bobwork.ai/llms-full.txt)

## Get an API key with no browser

An agent can create the account for its human. The human only has to read one code from their inbox.

```bash
curl -X POST https://app-emailwarmup.bobwork.ai/api/v1/signup/code \
  -H "Content-Type: application/json" -d '{"email": "you@company.com"}'
# -> {"nonce": "..."}; a 6-digit code is emailed to you@company.com

curl -X POST https://app-emailwarmup.bobwork.ai/api/v1/signup/verify \
  -H "Content-Type: application/json" -d '{"nonce": "...", "code": "123456"}'
# -> {"api_key": "bwk_..."}  (shown once)
```

Or sign in at https://app-emailwarmup.bobwork.ai/login and create a key under API.

## Add it to your client

Claude Code:

```bash
claude mcp add --transport http email-warmup https://app-emailwarmup.bobwork.ai/mcp \
  --header "Authorization: Bearer bwk_YOUR_KEY"
```

Cursor, Windsurf, or any client that takes JSON:

```json
{
  "mcpServers": {
    "email-warmup": {
      "url": "https://app-emailwarmup.bobwork.ai/mcp",
      "headers": { "Authorization": "Bearer bwk_YOUR_KEY" }
    }
  }
}
```

Claude Desktop (through mcp-remote):

```json
{
  "mcpServers": {
    "email-warmup": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://app-emailwarmup.bobwork.ai/mcp", "--header", "Authorization: Bearer bwk_YOUR_KEY"]
    }
  }
}
```

## Tools (39)

| Tool | What it does |
|---|---|
| `get_account` | Your account: email, credit balance, free mailbox allowance (10, or 20 for accounts created before 27 October 2026, plus referral bonus), paid extras and your... |
| `list_mailboxes` | Every connected mailbox with warm-up day, status, 7-day spam rate, today's and last 7 days' numbers, and whether it is ready for outbound. |
| `get_mailbox` | One mailbox's warm-up status and health. |
| `mailbox_activity` | The last 100 warm-up events for a mailbox (sent, received, replied, rescued from spam, errors). |
| `warmup_stats` | Totals across your mailboxes: today and last 7 days (sent, received, rescued, replied), active and ready counts. |
| `list_providers` | IMAP/SMTP presets (Gmail app password, Yahoo, iCloud, Zoho, Lark, SES, SendGrid, Mailgun…) with hosts and ports, plus the Google and Zoho API routes. |
| `connect_imap_mailbox` | Connect any mailbox with IMAP + SMTP. The login is tested before it is saved. Use an app password when two-factor sign-in is on. Give a preset (see list_providers) or... |
| `connect_zoho_mailbox` | Connect a Zoho Mail mailbox through the Zoho API (works on Zoho's free plan). Needs a Self Client id, secret and a generated code from api-console.zoho.com. |
| `google_connect_link` | Get a link the user opens to connect Gmail / Google Workspace with Google sign-in (one click, any device). The mailbox appears in list_mailboxes a few minutes after... |
| `update_mailbox` | Change a mailbox's display name, daily warm-up cap (1-100, null = default) or time zone (IANA, e.g. Europe/London). |
| `pause_mailbox` | Pause warm-up for a mailbox. |
| `resume_mailbox` | Resume warm-up for a mailbox. |
| `test_mailbox` | Sign in to the mailbox now and report whether the stored login still works. |
| `remove_mailbox` | Remove a mailbox and delete its stored login. Irreversible: ask the user first, then pass confirm=true. |
| `set_pool_mode` | Shared network (default: warm with every mailbox on the service) or private pool (only your own mailboxes write to each other; needs 2+). |
| `domain_health` | SPF, DKIM selector, DMARC policy and MX for every domain you have connected. |
| `check_spf` | Look up a domain's SPF record. |
| `check_dkim` | Find a domain's DKIM key: tries the selector you give, selectors seen in real mail, then 45 common ones. |
| `check_dmarc` | Look up a domain's DMARC record and policy. |
| `get_credits` | Prepaid credit balance, prices and recent charges (Outbound emails, AI drafts). |
| `list_campaigns` | Your Outbound campaigns with status and stats. |
| `get_campaign` | One campaign: its emails, mailboxes, stats and time left. |
| `create_campaign` | Create a draft Outbound campaign (nothing is sent). Emails can use {first_name}, {company} and any contact field. Any connected mailbox can send; one that is still... |
| `update_campaign` | Edit a paused or draft campaign. Fields you leave out keep their value. |
| `add_contacts` | Add up to 1,000 contacts to a campaign. Each needs email; other fields (first_name, company…) fill the placeholders. Duplicates and unsubscribed addresses are skipped. |
| `list_campaign_contacts` | Contacts in a campaign and where each is in the sequence (first 200). |
| `start_campaign` | Start or resume a campaign. This SENDS real email and uses credits. Confirm with the user first. accept_sending_rules=true means the user agrees to... |
| `pause_campaign` | Pause a running campaign. |
| `get_billing_status` | Payment status after you gave the user a checkout link: credit balance, whether the extra-mailbox subscription is active, mailboxes vs the free limit, and checkouts... |
| `create_billing_checkout` | Get a Stripe Checkout url for extra mailboxes beyond the free limit (monthly, or yearly = 10 months' price for 12). The USER opens it and pays; you cannot. Then poll... |
| `create_credits_checkout` | Get a Stripe Checkout url to top up prepaid credits for Outbound sending and AI drafts (one-off $5, $10, $25 or $100; credits never expire). The USER opens it and... |
| `list_bought_domains` | Domains bought in the domain shop, their hosted mailboxes, renewal date and price, plus IMAP/SMTP settings. |
| `search_domains` | Find available lookalike domains for cold email, with first-term and renewal prices. |
| `buy_domains` | Buy domains with up to 3 mailboxes each. Returns a Stripe checkout link for the person to pay; setup starts once paid. Needs accept_sending_rules true. |
| `add_hosted_mailboxes` | Add mailboxes to a domain you bought here ($3 per mailbox per month). Returns a Stripe checkout link. |
| `hosted_mailbox_login` | Password, IMAP and SMTP settings for one of your hosted mailboxes. |
| `list_dns_records` | DNS records on a domain you bought here. |
| `add_dns_record` | Add a DNS record to a domain you bought here. name = subdomain ('' for the domain itself). Types A, AAAA, CNAME, ALIAS, MX, TXT, CAA, SRV. |
| `delete_dns_record` | Delete a DNS record by id (from list_dns_records). Deleting MX, SPF, DKIM or DMARC stops the mailboxes working. |

## Pricing

- Warm-up: free for up to 10 mailboxes, no credit card. Extra mailboxes are billed per mailbox per month, cheaper at volume; yearly billing is 10 months for 12. Current prices: https://emailwarmup.bobwork.ai/pricing
- Cold email: $2 per 1,000 emails sent, from prepaid credits ($10, $25, $100 packs).
- When payment is needed, the API and MCP tools return a Stripe checkout link for the human to open, and `get_billing_status` confirms it went through.

## Support

info@bobwork.ai. BobWork is a product of BobOffice, LLC.
