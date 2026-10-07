# ZenMode MCP server

[ZenMode](https://www.zen-mode.io) is LinkedIn outreach software. Its remote MCP server lets an AI assistant
work with your ZenMode account in plain language: read campaigns, contacts and conversations, add or remove
people in a campaign, pause or exclude a contact, and change who a campaign finds.

It works with Claude, ChatGPT and other MCP clients.

- **Server address:** `https://www.zen-mode.io/api/mcp`
- **Transport:** Streamable HTTP
- **Sign-in:** approve the connection in your client (OAuth), or use a ZenMode API key as a Bearer token
- **Plan:** Practitioner and above. The MCP server and the REST API are not on Novice.
- **Contact:** hello@zen-mode.io

This repository holds documentation only. The server runs at the address above and its source is not published here.

## What it can do

Fifteen tools. Everything is scoped to the ZenMode account you sign in with.

**Read**

| tool | what it returns |
|---|---|
| `get_account_summary` | plan, whether the ZenMode desktop app has checked in recently, campaigns by status, and the last 30 days of results |
| `list_campaigns` | your campaigns with status, daily limit and results for a date window |
| `get_campaign` | one campaign: type, status, daily limit, business hours, targeting, contacts per stage, results |
| `search_contacts` | contacts across your campaigns by name, company, job title or LinkedIn URL |
| `get_contact` | one contact: profile details, campaign, status, outreach state, do-not-contact status |
| `get_conversation` | the LinkedIn conversation history with one contact, oldest first |
| `get_campaign_targeting` | who a campaign is set to find |
| `get_safety_status` | what ZenMode's own limits say about the account right now |
| `get_setup_status` | where the account is in setup, and the next step |
| `get_desktop_app` | the desktop app download and install command |
| `start_trial` | a checkout link for the free 14-day trial (accounts with no plan only) |

**Change** (each is marked as a change, so a client can ask you to confirm it)

| tool | what it does |
|---|---|
| `add_leads_to_campaign` | add up to 100 people to a campaign by LinkedIn profile URL; invalid URLs, your do-not-contact list, the campaign's exclusion list and duplicates are refused and reported per person |
| `remove_leads_from_campaign` | remove up to 100 contacts from a campaign |
| `set_contact_outreach_state` | pause a contact, put them on your do-not-contact list, or mark them not interested |
| `update_campaign_targeting` | change job titles, locations, industries and companies a campaign finds |

## What it cannot do

There is no tool that sends a LinkedIn message or a connection request. Adding people to a campaign puts them
in that campaign's worklist. ZenMode's desktop app then works through the list later, inside the account's
daily and weekly limits and business hours, and only while the desktop app is running.

If the account is restricted, paused for the week, cooling down or switched off, a change that would widen who
can be contacted is refused with the reason and when it lifts.

## Set up

### Claude (claude.ai)

1. Open **Customize → Connectors**, click **+**, then **Add custom connector**. On a Team or Enterprise plan an
   owner adds it under **Organization settings → Connectors → Add → Custom → Web**.
2. Name it **ZenMode** and paste `https://www.zen-mode.io/api/mcp`. Leave the advanced OAuth settings empty.
3. Click **Connect**. Sign in to ZenMode if asked, choose **Read only** (the default) or **Read and change**,
   pick the API key to connect with, and click **Allow**.
4. In a new chat, open the **+** menu, go to **Connectors**, and switch **ZenMode** on.

### Claude Code

```bash
claude mcp add --transport http zenmode https://www.zen-mode.io/api/mcp --header "Authorization: Bearer YOUR_ZENMODE_API_KEY"
```

Run `claude mcp list` to check it connected.

### ChatGPT

Add ZenMode as a custom MCP server in ChatGPT, approve it on a ZenMode page, and keep the scopes on read only
unless you want ChatGPT to make changes. Step by step, from a real run on a Plus account:
https://www.zen-mode.io/integrations/chatgpt. Which ChatGPT plans can add a custom MCP server is up to OpenAI.

### Cursor

Add this to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "zenmode": {
      "url": "https://www.zen-mode.io/api/mcp",
      "headers": {
        "Authorization": "Bearer ${env:ZENMODE_API_KEY}"
      }
    }
  }
}
```

Set `ZENMODE_API_KEY` in your environment so the key is not stored in the file.

### VS Code (GitHub Copilot)

Add this to your user `mcp.json`. VS Code asks for the key and keeps it out of the file:

```json
{
  "servers": {
    "zenmode": {
      "type": "http",
      "url": "https://www.zen-mode.io/api/mcp",
      "headers": {
        "Authorization": "Bearer ${input:zenmode-api-key}"
      }
    }
  },
  "inputs": [
    {
      "type": "promptString",
      "id": "zenmode-api-key",
      "description": "ZenMode API key",
      "password": true
    }
  ]
}
```

### Gemini CLI

```bash
gemini mcp add --scope user --transport http -H "Authorization: Bearer YOUR_ZENMODE_API_KEY" zenmode https://www.zen-mode.io/api/mcp
gemini mcp list
```

Create an API key in the ZenMode dashboard at https://www.zen-mode.io/dashboard/api. Revoking the key ends the
connection.

### Grok Build

This repository is also a Grok Build plugin: `.grok-plugin/plugin.json` and `.mcp.json` point Grok Build at the
same server address. Sign in to ZenMode when it first connects.

## Example prompts

- "How are my LinkedIn campaigns doing over the last 30 days?" uses `list_campaigns`.
- "Find everyone at Acme I've been talking to and show me the latest conversation." uses `search_contacts`, then `get_conversation`.
- "Add these five LinkedIn profiles to my Q4 founders campaign and tell me if any were refused." uses `add_leads_to_campaign`.

## Limits

Per ZenMode API key (a connection made through sign-in shares the budget of the key it uses):

- 60 tool calls a minute
- 10 changes a minute and 300 a day
- 500 leads added per day, up to 100 per call

## Links

- Setup guides for every client: https://www.zen-mode.io/integrations/mcp
- Guide for Claude: https://www.zen-mode.io/integrations/claude
- Pricing: https://www.zen-mode.io/pricing
- Privacy policy: https://www.zen-mode.io/privacy
- Terms: https://www.zen-mode.io/terms
- Service status: https://www.zen-mode.io/status
- `llms.txt`: https://www.zen-mode.io/llms.txt

## Licence

The documentation in this repository is licensed under
[Creative Commons Attribution 4.0 International](LICENSE). That licence covers the text of these files only.
It does not cover the ZenMode name or logo, and it does not grant any right to the ZenMode software or service.
