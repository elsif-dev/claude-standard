# Claude Standard Plugin

A Claude Code plugin repository.

## Plugin Structure

This repository follows the Claude Code plugin structure:

```
claude-standard/
├── plugins/
│   ├── .claude-plugin/
│   │   └── plugin.json          # Plugin metadata and configuration
│   ├── commands/                 # Custom slash commands (markdown files)
│   │   └── git/
│   │       └── commit.md        # /git:commit command
│   ├── agents/                   # Custom agent definitions
│   │   ├── quality-orchestrator.md
│   │   ├── claude-code-quality-runner.md
│   │   └── claude-code-quality-resolver.md
│   ├── skills/                   # Agent Skills with SKILL.md files
│   └── hooks/                    # Event handlers (hooks.json)
└── .mcp.json                # MCP server configurations
```

## Plugin Components

### Commands

Place markdown files in the `plugins/commands/` directory to create custom slash commands. Each file becomes a command that Claude can execute.

#### Git Commit (`plugins/commands/git/commit.md`)

A slash command `/git:commit` that creates git commits following best practices:
- **Quality Checks**: Automatically invokes the quality-orchestrator before committing
- **Commit Message Conventions**: Follows repository style and includes Claude Code attribution
- **Safety**: Warns about sensitive files, validates changes, prevents empty commits
- **Scoped Permissions**: Only allows specific git commands (status, diff, log, add, commit, show)

The command MUST run quality checks before every commit to ensure code quality and compliance.

### Agents

This plugin includes specialized subagents in the `plugins/agents/` directory:

#### Quality Orchestrator (`plugins/agents/quality-orchestrator.md`)

A generic quality orchestration agent that:
- **Discovers** runner/resolver pairs dynamically across all working directories
- **Coordinates** quality review workflows before commits
- **Pairs** runners (that find issues) with resolvers (that fix them) by matching names and file patterns
- **Works** across different project structures without hardcoded paths

Unlike directory-specific orchestrators, this agent searches for quality agents by pattern:
- Runners: `*-runner.md` or `*runner*.md`
- Resolvers: `*-resolver.md` or `*resolver*.md`

Use it proactively before commits to ensure code quality through automated review and fix workflows.

#### Claude Code Quality Runner (`plugins/agents/claude-code-quality-runner.md`)

Reviews Claude Code configuration files for security, compliance, and best practices:
- **Security Review**: Checks `.claude/settings.json` for dangerous permissions and overly broad tool access
- **RFC 2119 Compliance**: Ensures proper use of requirement keywords (MUST, SHOULD, MAY) in configuration files
- **Minimum Permissions**: Verifies commands, agents, and skills use only necessary tools

Automatically invoked by the quality-orchestrator for `.claude/` configuration files. Produces structured reports without making modifications.

#### Claude Code Quality Resolver (`plugins/agents/claude-code-quality-resolver.md`)

Automatically fixes issues identified by the quality-runner:
- **Security Fixes**: Restricts overly permissive tool access in settings
- **RFC 2119 Compliance Fixes**: Updates informal language to proper RFC 2119 keywords
- **Permission Scoping Fixes**: Reduces tool access to minimum necessary permissions

Paired with the quality-runner by the orchestrator. Applies fixes automatically and reports results.

### Skills
Create agent Skills in the `plugins/skills/` directory. Each skill should have a `SKILL.md` file that extends Claude's capabilities.

### Hooks
Configure event handlers in `plugins/hooks/hooks.json` to respond to Claude Code events like tool calls or user interactions.

### MCP Servers
This plugin includes standard MCP (Model Context Protocol) servers configured in `.mcp.json`:

- **filesystem**: File system operations with read/write access
- **git**: Git repository operations and version control
- **sequential-thinking**: Enhanced reasoning and step-by-step problem solving
- **time**: Current time and date information

These servers activate automatically when the plugin is enabled.

## Installation

To use this plugin with Claude Code:

1. Copy or clone this repository to your plugins directory
2. Configure via a marketplace manifest, or
3. Install directly using the Claude Code CLI

## Configuration

### Plugin Settings

Edit `plugins/.claude-plugin/plugin.json` to customize:
- Plugin name and version
- Author information
- Description and keywords
- Custom paths for components

### Environment Variables

The configured MCP servers require no additional environment variables. All servers will work out of the box when the plugin is enabled.

## Resources

- [Claude Code Plugin Documentation](https://docs.claude.com/en/docs/claude-code/plugins.md)
- [Plugin Reference](https://docs.claude.com/en/docs/claude-code/plugins-reference.md)
- [Skills Guide](https://docs.claude.com/en/docs/claude-code/skills.md)
- [MCP Server Documentation](https://docs.claude.com/en/docs/claude-code/mcp.md)
