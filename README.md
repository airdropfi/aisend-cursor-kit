# AISend for Cursor and Grok Build

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

To send to other people, verify a sending domain at https://aisend.app/domains. Until then you can only send test mail from an address at `send.aisend.app` to your own account email.

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
