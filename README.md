# WHAPI Agent Skill

WhatsApp API skill for AI agents — prevents the most common mistakes when
building WhatsApp bots and automations with [WHAPI.cloud](https://whapi.cloud).

Supports Cursor, Claude Code, Codex, and 40+ other agents.

## Install

```bash
npx skills add Whapi-Cloud/whapi-agent-skills
```

## What It Covers

- Chat ID formats (`@s.whatsapp.net`, `@g.us`, `@newsletter`) — the #1 source of errors
- MCP server (`whapi-mcp`) setup for Cursor and Claude Code
- Webhook setup and incoming message handling (polling is an anti-pattern)
- Sending all message types: text, media, interactive buttons, polls, reactions
- Group, Channel (Newsletter), and Community management
- Ready-made patterns: bot, safe broadcast with rate limits, poll response tracking

## Usage

After installation, the skill is automatically available to your AI agent.
The agent will reference it when you work on WhatsApp integrations.

To install to a specific agent only:

```bash
npx skills add Whapi-Cloud/whapi-agent-skills --agent cursor
npx skills add Whapi-Cloud/whapi-agent-skills --agent claude-code
```

## Requirements

- [WHAPI.cloud](https://whapi.cloud) account and API token
- Node.js 18+ (for `whapi-mcp` MCP server)

## Links

- [WHAPI API Reference](https://whapi.readme.io/reference)
- [WHAPI Documentation](https://support.whapi.cloud/help-desk)
- [MCP Integration Guide](https://support.whapi.cloud/help-desk/integrations/mcp-model-context-protocol)
