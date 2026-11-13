# AISend for Cursor and Grok Build

> AISend is a paid transactional email API built for AI agents to operate and humans to supervise: an agent sends from your verified domain, checks delivery and validates addresses over MCP or a Resend-compatible API, while you approve, review and can revoke access.

## AISend at a glance

- **What it is:** AISend is a paid transactional email API built for AI agents to operate and humans to supervise: an agent sends from your verified domain, checks delivery and validates addresses over MCP or a Resend-compatible API, while you approve, review and can revoke access.
- **Primary user:** An AI agent (ChatGPT, Claude, Cursor, Gemini CLI or any MCP client) operates it: sends email, checks delivery, validates addresses. A human tags the work, approves the connection, reviews what the agent did and can revoke access.
- **Who pays for it:** Product and engineering teams at SaaS companies and agencies that send password resets, receipts and notifications from their own domains, and individuals or small businesses who want an assistant to send and track email from a domain they own.
- **What it does not do:** Cold outreach, purchased lists, bulk marketing or newsletters. No campaigns or automations. No free plan and no free sending: an account needs an active paid plan before it can send. Sending to anyone other than your own account address needs a domain you have verified.
- **Price:** From $14/month (Pro, 50,000 emails/month), billed monthly per account. Business $63/month for 100,000 emails/month; Scale $149/month for 500,000 emails/month (daily cap 5,000, about 150,000 a month today). No per-email overage charges. No free plan.
- **Integrations:** MCP (hosted https://api.aisend.app/mcp and local npx aisend-mcp), Resend-compatible REST API, Node.js SDK (aisend-email), HMAC-SHA256 signed webhooks, delivery through AWS SES.
- **Security and data:** Sends through AWS SES. Data encrypted in transit (TLS); API keys stored hashed. This kit signs in with an AISend API key sent as a Bearer header (no OAuth): the key has your account's API access, does not expire and can be revoked on the API Keys page; your plan's daily cap applies (Pro 2,500, Business 5,000, Scale 5,000 a day).
- **Start in minutes:** Create an account (free), add a domain you own and publish its DNS records (propagation time varies), subscribe to a plan, then connect an assistant at https://aisend.app/mcp (about a minute) or create an API key.
- **MCP:** https://api.aisend.app/mcp (Streamable HTTP, OAuth or API key). Official MCP Registry name: app.aisend/aisend.

## Connect in 60 seconds

1. Create an API key at https://aisend.app/api-keys and export it: `export AISEND_API_KEY=YOUR_API_KEY`, then restart Cursor.
2. Add to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "aisend": {
      "url": "https://api.aisend.app/mcp",
      "headers": { "Authorization": "Bearer ${env:AISEND_API_KEY}" }
    }
  }
}
```

You need an AISend account. Sending needs an active paid plan. Until you verify a domain, a paid account can only send from `send.aisend.app` to its own account email. To send to anyone else, verify a sending domain you own.

## Example prompts

You tag the work; the agent calls the tools; you review and approve.

- `Assign to AISend: email billing@client.example the approved invoice reminder from our verified domain, idempotency key invoice-2026-11-reminder. Show me the exact message first and send only after I say yes.`
- `Use AISend to list my verified sending domains and tell me which one to send from. Do not send anything.`
- `Use AISend to show the status of my last 10 emails and flag any that bounced or complained, so I can review them.`
- `Use AISend to validate these addresses and list the disposable or role-based ones. Show me the list so I can decide.`

## Human oversight: what you can see, approve and revoke

- **Revoke access.** The key is on the API Keys page of the AISend dashboard; revoke it there and the next call fails with 401. It does not expire on its own.
- **See what was sent.** The Emails page lists every sent email with its delivery status, and `list_emails` shows the same to the agent.
- **Approve each send, in clients that ask.** `send_email` is annotated destructive, so Cursor's tool-call approval (on by default) asks before a send; that prompt is Cursor's setting, not enforced by AISend. The kit's rule and skills also tell the agent to show the message and wait for a clear yes.
- **Bound the blast radius.** Your plan's daily cap (Pro 2,500, Business 5,000, Scale 5,000), quotas, new-account ramp-up, new-domain caps, the verified-domain requirement, suppression and the content gate apply to every send. There is no separate per-connection cap for a key you create yourself.
- **Watch events.** Webhooks deliver sent, delivered, opened, clicked, bounced and complained events, signed with HMAC-SHA256, to an endpoint you control.

## Links

- Product: https://aisend.app
- MCP page with the agent quickstart (markdown: https://aisend.app/mcp.md): https://aisend.app/mcp
- Pricing: https://aisend.app/pricing
- Official MCP Registry name: `app.aisend/aisend`
- MCP server card: https://aisend.app/.well-known/mcp/server-card.json
- llms.txt: https://aisend.app/llms.txt
- Other ways in: [Claude Code plugin](https://github.com/airdropfi/aisend-claude-plugin), [Gemini CLI extension](https://github.com/airdropfi/aisend-gemini-extension), npm [`aisend-mcp`](https://www.npmjs.com/package/aisend-mcp)

## Details

Send email from your own verified domain with [AISend](https://aisend.app) from Cursor or Grok Build. The agent shows you the exact message first, and nothing is sent until you say yes. Sending is real and cannot be recalled, and your account's sending limits apply.

## What it contains

| Component | Name | What it does |
|---|---|---|
| MCP server | `aisend` (`https://api.aisend.app/mcp`) | The AISend tools, authenticated with your API key as a Bearer header. |
| Rule | `rules/aisend.mdc` | Show before send, idempotency key, verified domains only, honest delivery status. |
| Skill | `send-an-email` | Pick a verified sender, show the full message, send after your yes, check status. |
| Skill | `check-deliverability` | Delivery status of recent mail and address validation. Sends nothing. |

## Sign-in: API key, not OAuth

AISend's OAuth sign-in only accepts redirects for claude.ai, ChatGPT and local loopback addresses (`http://localhost`, `127.0.0.1`, `[::1]`). Cursor's own `cursor://` callback is not on that list, so this kit uses your API key instead.

1. Create a key at https://aisend.app/api-keys.
2. Put it in an environment variable that Cursor can see, for example `export AISEND_API_KEY=YOUR_API_KEY` in your shell profile, then restart Cursor. Never commit the key.

A key you create yourself has your account's full API access and does not expire; revoke it on the API Keys page when you no longer need it.

## Install in Cursor

1. Add this to `~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (one project):

   ```json
   {
     "mcpServers": {
       "aisend": {
         "url": "https://api.aisend.app/mcp",
         "headers": {
           "Authorization": "Bearer ${env:AISEND_API_KEY}"
         }
       }
     }
   }
   ```

   Or use the [install link](https://cursor.com/link/mcp/install?name=aisend&config=eyJ1cmwiOiJodHRwczovL2FwaS5haXNlbmQuYXBwL21jcCIsImhlYWRlcnMiOnsiQXV0aG9yaXphdGlvbiI6IkJlYXJlciBZT1VSX0FQSV9LRVkifX0%3D): it adds the server with the placeholder `Bearer YOUR_API_KEY`, which you then replace with your key (or with `Bearer ${env:AISEND_API_KEY}`) in Cursor's MCP settings.
2. Open Customize, then MCPs, and check that `aisend` shows its tools.
3. Optional, the rule: copy `rules/aisend.mdc` to `.cursor/rules/aisend.mdc` in your project. It is set to "Agent Decides", so Cursor applies it when you ask about email.
4. Optional, the skills: copy the folders in `skills/` to `~/.cursor/skills/`.

Cursor asks you to approve MCP tool calls by default. Leave that on: sending mail is real.

## Install in Grok Build

```bash
grok mcp add --transport http aisend https://api.aisend.app/mcp --header "Authorization: Bearer YOUR_API_KEY"
```

Grok Build also loads `.cursor/mcp.json`.

## Add a domain first

To send to other people, verify a sending domain at https://aisend.app/domains. Sending needs an active paid plan. Until you verify a domain, a paid account can only send from `send.aisend.app` to its own account email.

## Use

Try the prompts in [examples/prompts.md](examples/prompts.md).

## What it will and will not do

- It shows the whole message and waits for a clear yes before every send, and uses an idempotency key so a retry never sends a second copy.
- It sends only from a domain you verified, and refuses bulk mail to purchased lists and deceptive or unsolicited mail.
- It reports delivery status exactly as AISend returns it.
- The hosted server cannot create API keys or accounts, and the agent never asks you to paste a key into the chat.

## Data and privacy

The MCP server receives what the tools need: the messages and addresses you ask the agent to send or look up, and the email, domain and key-name data on your AISend account. This repository stores nothing and runs no code on your machine; it is a server address, Markdown instructions and a logo. See the [privacy policy](https://aisend.app/privacy) and [terms](https://aisend.app/terms).

## Layout

```text
.cursor-plugin/plugin.json   Cursor Plugin manifest (also read by Grok Build)
mcp.json                     MCP server for Cursor (API key from AISEND_API_KEY)
.mcp.json                    the same server for Grok Build and other loaders
rules/aisend.mdc             rule
skills/                      two skills (SKILL.md)
assets/logo.png              1024x1024 logo
examples/prompts.md          example prompts
```

## Support

https://aisend.app/docs or hello@aisend.app.

## License

MIT for the files in this repository (see `LICENSE`). The AISend service is governed by its terms.
