# spaceship-skills

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![MCP](https://img.shields.io/badge/MCP-compatible-purple.svg)](https://modelcontextprotocol.io)
[![CI](https://github.com/bartwaardenburg/spaceship-skills/actions/workflows/validate.yml/badge.svg)](https://github.com/bartwaardenburg/spaceship-skills/actions/workflows/validate.yml)

Agent skills for managing domains via the [Spaceship](https://spaceship.com) registrar API. Works with any agent that supports the [Agent Skills](https://agentskills.io) specification — Claude Code, Cursor, OpenAI Codex, Windsurf, GitHub Copilot, Gemini CLI, Amp, and more.

> **Note:** This is an unofficial, community-maintained project and is not affiliated with or endorsed by Spaceship.

## Features

- **48 Spaceship tools** organized into step-by-step workflows for domain management
- **9 tool categories** — DNS records, domain lifecycle, contacts, privacy, nameservers, SellerHub, and more
- **Common recipes** for Google Workspace, Microsoft 365, Vercel, Netlify, and other popular services
- **Gotcha documentation** covering async operations, financial safeguards, and edge cases
- **Companion skill** to the [spaceship-mcp](https://github.com/bartwaardenburg/spaceship-mcp) server

## Prerequisites

This skill requires the [spaceship-mcp](https://github.com/bartwaardenburg/spaceship-mcp) MCP server to be installed and configured with your Spaceship API credentials. See the [spaceship-mcp installation guide](https://github.com/bartwaardenburg/spaceship-mcp#installation) for setup instructions.

## Installation

### Claude Code

```bash
/install bartwaardenburg/spaceship-skills
```

### Cursor

Clone into Cursor's skills directory:

```bash
git clone https://github.com/bartwaardenburg/spaceship-skills.git ~/.cursor/skills/spaceship-skills
```

### OpenAI Codex

Clone into the Codex skills directory:

```bash
git clone https://github.com/bartwaardenburg/spaceship-skills.git ~/.agents/skills/spaceship-skills
```

### Windsurf

Clone into the Windsurf skills directory:

```bash
git clone https://github.com/bartwaardenburg/spaceship-skills.git ~/.codeium/windsurf/skills/spaceship-skills
```

### GitHub Copilot / VS Code

Clone into your project or user skills directory:

```bash
git clone https://github.com/bartwaardenburg/spaceship-skills.git ~/.agents/skills/spaceship-skills
```

### Gemini CLI

```bash
gemini skills install https://github.com/bartwaardenburg/spaceship-skills.git
```

### Amp

Clone into the Amp skills directory:

```bash
git clone https://github.com/bartwaardenburg/spaceship-skills.git ~/.config/agents/skills/spaceship-skills
```

### Other Agents

Clone or copy the skill directory into your agent's skills location. This skill follows the open [Agent Skills](https://agentskills.io) specification and works with any compatible agent.

## Available Skills

| Skill | Description |
|---|---|
| [spaceship-domains](spaceship-domains/) | Manage domains, DNS records, contacts, privacy, nameservers, transfers, and SellerHub marketplace listings |

## What's Included

### spaceship-domains

Covers the full Spaceship domain management API across 9 categories:

| Category | Key Operations |
|---|---|
| Domain Lookup | Check availability and pricing for up to 20 domains |
| Domain Info | List and inspect domains in your account |
| Domain Settings | Auto-renewal, transfer locks, nameservers, auth codes |
| Domain Lifecycle | Register, renew, transfer, and restore domains |
| DNS Management | List, save, delete, and verify DNS records in bulk |
| DNS Record Creation | Create individual records across 13 types (A, AAAA, CNAME, MX, TXT, SRV, NS, ALIAS, CAA, HTTPS, PTR, SVCB, TLSA) |
| Contacts & Privacy | Contact profiles, WHOIS privacy, email protection |
| Personal Nameservers | Vanity/glue record nameservers |
| SellerHub Marketplace | List domains for sale, manage pricing, generate checkout links |

### Reference Documentation

Each skill includes structured reference files:

- **[API Reference](spaceship-domains/references/api-reference.md)** — complete parameter specifications for all 48 tools
- **[Gotchas](spaceship-domains/references/gotchas.md)** — common pitfalls, edge cases, and correct usage patterns
- **[Patterns](spaceship-domains/references/patterns.md)** — DNS recipes for popular services (Google Workspace, Microsoft 365, Vercel, Netlify, etc.)

## Example Prompts

Once installed, you can use natural language:

- "Register example.com with WHOIS privacy enabled"
- "Set up DNS for example.com with Google Workspace email"
- "Point example.com to my Vercel deployment"
- "Transfer example.com to Spaceship"
- "List my domains for sale on SellerHub"
- "Check if the DNS records for example.com are configured correctly"

## Contributing

See [CLAUDE.md](CLAUDE.md) for repository structure, skill creation guidelines, and quality standards.

## Related

- [spaceship-mcp](https://github.com/bartwaardenburg/spaceship-mcp) — The MCP server that powers these skills

## License

MIT — see [LICENSE](LICENSE) for details.
