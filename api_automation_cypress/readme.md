---
name: qa-agent-api-cypress
description: End-to-end API test generation and execution agent using Cypress + fixtures
tools:
  - mcp__cypress__*
  - mcp__filesystem__*
mcp-servers:
  cypress:
    command: npx
    args: ["-y", "cypress-mcp", "--cwd", "."]
  filesystem:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-filesystem", "."]
---
"""
Project README: API Test Agent (Cypress + MCP)
"""

# API Test Agent (Copilot)

## Purpose
Design, generate, and execute API tests using Cypress. This folder exposes an AI agent that can create scaffolding, clients, fixtures, and specs, and can execute tests via Model Context Protocol (MCP) servers.

---

## Capabilities
- Generate a Cypress API test framework from templates
- Create reusable API domain client wrappers and utilities
- Generate test specs from OpenAPI or example payloads
- Execute tests and inspect results via the Cypress MCP server
- Read and write workspace files via the filesystem MCP server

---

## Agent files and MCP configuration

This repository includes the following agent integration files:

- `.mcp.json` — root MCP server configuration for `cypress` and `filesystem`.
- `.claude/agents/qa-agent-api-cypress.md` — Claude agent prompt and tooling config.
- `.cursor/rules/qa-agent-api-cypress.mdc` — Cursor agent rule file.
- `.github/agents/qa-agent-api-cypress.agent.md` — GitHub Copilot agent instructions.

When copying this project into a blank repository, keep these files in place so your editor/agent tooling can discover the project-specific agent behavior and MCP setup.

---

## Quick start

Prerequisites:

- Node.js 18+ installed
- Git or a way to clone this repository

1. Start the MCP servers (in separate terminals). The frontmatter at the top of this file defines the same servers.

Filesystem MCP server:

```bash
npx @modelcontextprotocol/server-filesystem
```

Cypress MCP server:

```bash
npx cypress-mcp --cwd .
```

2. Use the agent to generate the API test framework and create domain wrappers from request/response examples.

3. Run generated Cypress specs using the agent or via `npx cypress run`.

With both MCP servers running, the AI agent has access to the `mcp__cypress__*` and `mcp__filesystem__*` tools declared in the frontmatter and can generate, edit, and execute tests programmatically.

---

## Using the agent (summary)

- Ensure both MCP servers are running as shown above.
- Open your editor/agent that integrates with MCP-enabled tools and invoke the agent prompts.
- The agent will use the filesystem MCP to read/write files and the Cypress MCP tool to execute tests and gather results.

Example prompts you can give the agent:

- "Generate API client wrappers for the Users domain and add fixtures."
- "Create a spec that verifies the /users POST endpoint against the fixture file."
- "Run the API test suite and return the failing request/response details."

---

## Project layout
See `AGENT.md` for full scaffolding and templates. Key folders the agent will create/use:

```
cypress/
  fixtures/
    hosts.json
    <domain>/
      create_request.json
      create_response.json
  e2e/
    api/
      <domain>.spec.ts
  support/
    lib/
      <domain>.ts
    utils/
      requests.ts
      assertions.ts
      common.ts
      apiTracker.ts
cypress.config.ts
package.json
```

---

## More information
- Agent guidelines and templates: [AGENT.md](AGENT.md)
- Core utilities and examples are documented inside `AGENT.md` (API wrappers, requests, assertions, helpers).
