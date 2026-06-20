---
name: qa-agent-api-playwright
description: End-to-end API test generation and execution agent using Playwright + fixtures
tools:
  - mcp__playwright__*
  - mcp__filesystem__*
mcp-servers:
  playwright:
    command: npx
    args: ["@playwright/mcp"]
  filesystem:
    command: npx
    args: ["@modelcontextprotocol/server-filesystem"]
---
"""
Project README: API Test Agent (Playwright + MCP)
"""

# API Test Agent (Copilot)

## Purpose
Design, generate, and execute API tests using Playwright. This repository exposes an AI agent that can create scaffolding, clients, fixtures, and specs, and can execute tests via Model Context Protocol (MCP) servers.

---

## Capabilities
- Generate a Playwright API test framework from templates
- Create reusable API domain client wrappers and utilities
- Generate test specs from OpenAPI or example payloads
- Execute tests and instrument requests via the Playwright MCP server
- Read and write workspace files via the filesystem MCP server

---

## Agent files and MCP configuration

This repository includes the following agent integration files:

- `.mcp.json` — root MCP server configuration for `playwright` and `filesystem`.
- `.claude/agents/qa-agent-api-playwright.md` — Claude agent prompt and tooling config.
- `.cursor/rules/qa-agent-api-playwright.mdc` — Cursor agent rule file.
- `.github/agents/qa-agent-api-playwright.agent.md` — GitHub Copilot agent instructions.

When copying this project into a blank repository, keep these files in place so your editor/agent tooling can discover the project-specific agent behavior and MCP setup.

---

## Quick start

Prerequisites:

- Node.js 18+ (or current LTS) installed
- Git or a way to clone this repository

1. Start the MCP servers (in separate terminals). The frontmatter at the top of this file defines the same servers.

Filesystem MCP server:

```bash
npx @modelcontextprotocol/server-filesystem
```

Playwright MCP server:

```bash
npx @playwright/mcp
```

2. for new repositry. use qa-agent-api-playwright to generate api test and create folder structure for following set of curls.

3. for existing repository. use qa-agent-api-playwright to generate api test for following set of curls.

With both MCP servers running, the AI agent has access to the `mcp__playwright__*` and `mcp__filesystem__*` tools declared in the frontmatter and can generate, edit, and execute tests programmatically.

---


## Using the agent (summary)

- Ensure both MCP servers are running as shown above.
- Open your editor/agent that integrates with MCP-enabled tools and invoke the agent prompts (examples are in `AGENT.md`).
- The agent will use the filesystem MCP to read/write files and the Playwright MCP tool to execute tests and gather responses.

Example prompts you can give the agent:

- "Generate API client wrappers for the Users domain and add fixtures."
- "Create a spec that verifies the /users POST endpoint against the fixture file." 
- "Run the API test suite and return the failing request/response details."

---

## Project layout
See `AGENT.md` for full scaffolding and templates. Key folders the agent will create/use:

```
config/
libs/
  └── utils/
test_data/
spec/api/
playwright.config.ts
global-setup.ts
package.json
```

---

## More information
- Agent guidelines and templates: [AGENT.md](AGENT.md)
- Core utilities and examples are documented inside `AGENT.md` (API wrappers, requests, assertions, helpers).


