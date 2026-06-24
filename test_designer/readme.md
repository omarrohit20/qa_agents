---
name: qa-agent-test-designer
description: QA planning and test strategy agent for polyglot, microservices, and web applications
tools:
  - mcp__filesystem__*
  - mcp__atlassian__*
  - mcp__azure-devops__*
mcp-servers:
  filesystem:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-filesystem", "."]
  atlassian:
    command: npx
    args: ["-y", "mcp-remote@latest", "https://mcp.atlassian.com/v1/mcp/authv2"]
  azure-devops:
    command: npx
    args: ["-y", "@azure-devops/mcp", "YOUR_ORG"]
---
"""
Project README: QA Test Designer Agent
"""

# QA Test Designer Agent

## Purpose
Generate QA strategy, test planning, scenarios, and detailed test cases for polyglot systems, microservices, and web applications.

---

## Capabilities
- Draft QA strategy and scope documents
- Create a test plan covering environments, levels, and exit criteria
- Define test scenarios for microservices, API, web, and cross-service flows
- Produce detailed test cases with steps, expected results, priority, and traceability
- Resolve requirements from Jira or Azure DevOps via local token files or MCP servers
- Use filesystem MCP to read and write workspace artifacts

---

## Ticket context resolution

Before generating QA artifacts, resolve requirements from Jira or Azure DevOps.

### Local context files

| File | Purpose |
|------|---------|
| `context/ticket.txt` | Ticket or work item ID and source (`jira` or `azure`) |
| `context/jira.config` | Jira site URL, email, project key (for API token auth) |
| `context/jira.token` | Jira API token — optional, gitignored |
| `context/azure.config` | Azure DevOps organization, project, team |
| `context/azure.pat` | Azure DevOps PAT — optional, gitignored |

Copy the `.example` files in `context/` to create your local files. See `context/ticket.txt.example`, `context/jira.config.example`, and `context/azure.config.example`.

`context/ticket.txt` format:

```
source: jira
id: PROJ-123
```

or

```
source: azure
id: 12345
```

### Resolution workflow

1. Read `context/ticket.txt` (or use the ticket ID from the user prompt).
2. Determine the source (`jira` or `azure`).
3. **If a token file is present**, fetch the ticket via REST API:
   - Jira: use `context/jira.token` + `context/jira.config` → Jira REST API (`/rest/api/3/issue/{id}`)
   - Azure: use `context/azure.pat` + `context/azure.config` → Azure DevOps REST API (work item by ID)
4. **If the token file is missing or empty**, fetch via MCP:
   - Jira: use the **Atlassian MCP** server (`atlassian`) — complete OAuth when prompted; search or get the issue by key
   - Azure: use the **Azure DevOps MCP** server (`azure-devops`) — interactive or `azcli` auth; get work item by ID. Update `YOUR_ORG` in `.mcp.json` or match `context/azure.config`.
5. Extract acceptance criteria, priority, labels, linked items, environment notes, and existing coverage references.
6. Use extracted ACs as the source of truth for all QA artifacts. Reference the ticket ID in every output file.

Do not proceed with artifact generation until ticket context is loaded or the user confirms there is no ticket source.

---

## Design guidance

Apply all required design techniques:

- Happy path: at least one scenario per AC
- Negative scenarios: invalid inputs, missing required fields, unauthorized access, expired sessions, wrong format, out-of-range values
- Boundary analysis: empty, 1 char, max length, max+1, 0, negative, decimal, large, min/max, past/today/future, leap year, timezone edge, end of month
- Edge cases: concurrent actions, repeated actions, mid-flow navigation, deleted dependencies, special characters, Unicode, RTL, HTML injection
- Integration: adjacent feature interactions, upstream/downstream impact
- Non-functional: performance with large datasets, accessibility, cross-browser/cross-device behavior

Always generate the complete artifact set: `qa/test-strategy.md`, `qa/test-plan.md`, `qa/test-scenarios.md`, and `qa/test-cases.md`.

Use the exact output template below when writing test scenarios.

```
## Test Scenarios

**Ticket**: [ID]
**Feature**: [name]
**Total scenarios**: [count]
**Generated**: [date]

### Scenario Table

| ID | Category | Scenario | Steps | Expected Result | AC Ref | Priority |
|---|---|---|---|---|---|---|
| TS-001 | Happy Path | [name] | 1. Step
2. Step
3. Step | [observable outcome] | AC-1 | Must Test |
| TS-002 | Negative | [name] | 1. Step
2. Step | [observable outcome] | AC-1 | Must Test |
| TS-003 | Boundary | [name] | 1. Step
2. Step | [observable outcome] | AC-2 | Should Test |
| TS-004 | Edge Case | [name] | 1. Step
2. Step | [observable outcome] | AC-2 | Could Test |

### AC Coverage Matrix

| AC | Happy Path | Negative | Boundary | Edge Case | Integration |
|---|---|---|---|---|---|
| AC-1 | TS-001 | TS-002 | TS-005 | TS-008 | — |
| AC-2 | TS-003 | TS-006 | TS-007 | TS-009 | TS-010 |

### Test Data Requirements
- [Specific data needed — exact values, not descriptions]
- [Preconditions to set up before running]

### Risks and Gaps
- [Anything that cannot be fully tested and why]
- [Assumptions made about ambiguous AC]
- [Exploratory testing recommendations]

---
*To automate: run the automation-writer agent against this file.*
*To validate manually: run the manual-validator agent against this file.*
```

## Priority definitions

- Must Test: Core functionality, direct AC mapping, high risk
- Should Test: Important but lower risk, boundary cases for critical fields
- Could Test: Unlikely edge cases, cosmetic validation

## Rules

- Every AC must have at least one happy path AND one negative scenario.
- Scenarios must be independent — no shared state or execution order dependency.
- Steps must be specific. Not "enter data" -> "enter 'john@example.com' in the Email field".
- Expected results must be observable. Not "it works" -> "green toast displays 'Profile updated'".
- If AC is ambiguous, create scenarios for both interpretations and mark `[ASSUMPTION]`.
- Target 10-20 scenarios per ticket. More is fine if AC is broad.
- Do NOT write automation code — that is the automation-writer's job.

---

## Agent files and MCP configuration

This folder includes the following agent integration files:

- `.mcp.json` — root MCP server configuration for `filesystem`, `atlassian`, and `azure-devops`.
- `.claude/agents/qa-agent-test-designer.md` — Claude agent prompt and tooling config.
- `.cursor/rules/qa-agent-test-designer.mdc` — Cursor agent rule file.
- `.github/agents/qa-agent-test-designer.agent.md` — GitHub Copilot agent instructions.

When using this agent, keep these files in place so editor/agent tooling can discover the project-specific agent behavior.

---

## Quick start

Prerequisites:

- Node.js 18+ installed (Node.js 20+ recommended for Azure DevOps MCP)
- Git or workspace access to the `test_designer` folder
- Access to Jira (Atlassian Cloud) and/or Azure DevOps, if fetching ticket context

1. Copy context examples and configure ticket source (optional but recommended):

```bash
cp context/ticket.txt.example context/ticket.txt
cp context/jira.config.example context/jira.config   # if using Jira with API token
cp context/azure.config.example context/azure.config # if using Azure with PAT
```

2. For token-based auth, create credential files (gitignored):

```bash
echo "your-jira-api-token" > context/jira.token
echo "your-azure-devops-pat" > context/azure.pat
```

3. Replace `YOUR_ORG` in `.mcp.json` with your Azure DevOps organization name (required when using Azure MCP without a PAT).

4. Start MCP servers via your editor's MCP integration, or manually:

Filesystem:

```bash
npx @modelcontextprotocol/server-filesystem .
```

Atlassian (OAuth — used when `context/jira.token` is absent):

```bash
npx -y mcp-remote@latest https://mcp.atlassian.com/v1/mcp/authv2
```

Azure DevOps (interactive — used when `context/azure.pat` is absent):

```bash
npx -y @azure-devops/mcp YOUR_ORG
```

5. Use the QA Test Designer agent to generate or update strategy and test planning artifacts.

---

## Using the agent

- Ensure MCP servers are running (`filesystem` always; `atlassian` and/or `azure-devops` when fetching ticket context without local tokens).
- Set `context/ticket.txt` or include a ticket/work item ID in your prompt.
- Open your editor/agent with MCP integration and invoke the agent using the prompt name `qa-agent-test-designer`.
- Ask the agent to generate or improve QA planning artifacts such as `qa/test-strategy.md`, `qa/test-plan.md`, `qa/test-scenarios.md`, and `qa/test-cases.md`.

Example prompts:

- "Fetch PROJ-123 from Jira and create test scenarios with full AC coverage."
- "Load Azure work item 12345 and draft a test plan for API, integration, and end-to-end coverage."
- "Create a QA strategy for a polyglot microservices architecture from ticket PROJ-456."

---

## Artifact layout

```
context/ticket.txt          # ticket source (optional)
context/jira.config         # Jira site config (optional)
context/jira.token          # Jira API token (optional, gitignored)
context/azure.config        # Azure DevOps config (optional)
context/azure.pat           # Azure PAT (optional, gitignored)
qa/test-strategy.md
qa/test-plan.md
qa/test-scenarios.md
qa/test-cases.md
```

---

## More information
- Use the `.claude`, `.cursor`, and `.github` agent files to keep guidance consistent across editor tooling.
- Store QA artifacts under `qa/` for organized strategy and execution planning.
- Atlassian MCP docs: https://support.atlassian.com/atlassian-rovo-mcp-server/docs/getting-started-with-the-atlassian-remote-mcp-server/
- Azure DevOps MCP docs: https://github.com/microsoft/azure-devops-mcp
