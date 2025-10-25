# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a Claude Code plugin repository that provides standardized configurations, commands, agents, and MCP server integrations for elsif projects. It follows the Claude Code plugin structure and includes quality assurance workflows.

## Project Structure

```
claude-standard/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata (name, version, author)
├── commands/git/
│   └── commit.md           # /git:commit slash command
├── agents/
│   ├── quality-orchestrator.md        # Coordinates quality workflows
│   ├── claude-code-quality-runner.md  # Reviews config files
│   └── claude-code-quality-resolver.md # Fixes config issues
├── skills/                  # (Empty - for future agent skills)
├── hooks/                   # (Empty - for event handlers)
└── .mcp.json               # MCP server configurations
```

## Key Components

### Quality Assurance System

This plugin implements an automated quality review system with three specialized agents:

**quality-orchestrator** - Coordination agent that:
- Discovers runner/resolver pairs dynamically across all working directories
- Analyzes changed files (via `git diff --name-only HEAD`)
- Matches files to appropriate quality runners based on patterns
- Coordinates iterative review-fix-verify workflows (max 5 iterations)
- Cannot invoke subagents directly; provides Task tool invocation instructions to main thread
- Tracks iteration state and detects infinite loops

**claude-code-quality-runner** - Analysis agent (read-only) that:
- Reviews `.claude/settings.json` for dangerous permissions (e.g., `Bash(*:*)`)
- Checks configuration files for RFC 2119 compliance (MUST, SHOULD, MAY keywords)
- Validates minimum permissions for commands, agents, and skills
- Produces structured markdown reports without making modifications
- Tools: `Read, Glob, Grep`

**claude-code-quality-resolver** - Remediation agent that:
- Parses quality runner reports and applies suggested fixes
- Fixes security vulnerabilities, RFC 2119 violations, and permission issues
- Applies fixes in order: CRITICAL → HIGH → MEDIUM → LOW
- Generates remediation reports showing successful/partial/failed fixes
- Tools: `Read, Write, Edit, Glob`

### Slash Commands

**`/git:commit`** - Creates git commits with mandatory quality checks:
- Runs `quality-orchestrator` before every commit (non-negotiable)
- Reviews recent commit messages to match repository style
- Warns about sensitive files (.env, credentials)
- Scoped permissions: Only allows `git status/diff/log/add/commit/show` and Task tool
- Includes Claude Code attribution in commit messages

### MCP Servers

Pre-configured MCP servers in `.mcp.json`:
- **filesystem**: File system operations with read/write access (scoped to `${CLAUDE_PLUGIN_ROOT}`)
- **git**: Git repository operations and version control
- **sequential-thinking**: Enhanced reasoning and step-by-step problem solving
- **time**: Current time and date information

All servers use `npx -y` to auto-install and run. No additional environment variables required.

## Architecture Patterns

### Dynamic Agent Discovery

The quality orchestrator uses pattern-based discovery instead of hardcoded paths:
- Runners: Files matching `*-runner.md` or `*runner*.md`
- Resolvers: Files matching `*-resolver.md` or `*resolver*.md`
- Pairing: Matches by name prefix, file patterns, or domain keywords

### Coordination Model

Subagents cannot invoke other subagents. The orchestrator provides explicit instructions to the main thread using four structured response formats:
1. **Format 1**: Initial analysis with runner invocation instructions
2. **Format 2**: Runner results analysis with resolver invocation instructions
3. **Format 3**: Verification request after resolver fixes
4. **Format 4**: Final report (success/partial/failure)

### RFC 2119 Compliance

Configuration files use RFC 2119 keywords to define requirement levels:
- **MUST/REQUIRED/SHALL**: Absolute requirements
- **MUST NOT/SHALL NOT**: Absolute prohibitions
- **SHOULD/RECOMMENDED**: Strong recommendations (exceptions allowed)
- **MAY/OPTIONAL**: Discretionary items

The quality runner enforces consistent use of these keywords across all configuration files.

## Working with This Repository

### Making Changes to Configuration Files

When modifying files in `.claude/`, `agents/`, `commands/`, or `skills/`:
1. Changes are automatically reviewed before commits via `/git:commit`
2. Quality runner identifies security, compliance, and permission issues
3. Quality resolver applies fixes automatically
4. Verification runs confirm issues are resolved
5. Process iterates up to 5 times until all checks pass

### Adding New Agents

New agents should:
- Include YAML frontmatter with `name`, `description`, `tools`, and `model` fields
- Use RFC 2119 keywords for requirements (MUST, SHOULD, MAY)
- Request only minimum necessary tools
- Include `*-runner.md` suffix for analysis agents (read-only)
- Include `*-resolver.md` suffix for remediation agents

### Security Principles

- **Scope bash commands**: Use `Bash(git *:*)` instead of `Bash(*:*)`
- **Minimize permissions**: Only grant tools actually needed
- **Path restrictions**: Limit file operations to specific directories
- **No credential commits**: Quality checks warn about .env, credentials.json, etc.
