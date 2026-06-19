File	IDE / Platform	How it activates
.github/agents/api-test-agent.agent.md	GitHub Copilot (VS Code + cloud)	Select the agent in Copilot's agent picker
.cursor/rules/api-test-agent.mdc	Cursor	Auto-attaches when spec/libs/test_data files are open
.claude/agents/api-test-agent.md	Claude Code	Invoked via /api-test-agent or triggered proactively
.mcp.json	All IDEs (MCP config)	Declares Playwright + Filesystem MCP servers
MCP execution
The .mcp.json wires up two MCP servers that the agent uses to actually execute things — not just generate code:

@playwright/mcp — can launch and drive Playwright (navigate, assert, run scripts)
@modelcontextprotocol/server-filesystem — reads and writes project files (fixtures, wrapper classes, spec files)
The GitHub Copilot agent file declares both servers inline in its mcp-servers: frontmatter block. The Claude Code agent references them as mcp__playwright__* and mcp__filesystem__* in its tools: list.

Sharing without the codebase
All three agent files are self-contained — they embed the full patterns, templates, and checklist. Drop any single one into the matching IDE location and the agent can bootstrap a new framework from zero.