---
description: Create a git commit following best practices
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(git add:*), Bash(git commit:*), Bash(git show:*), Task
---

Create a git commit following the project's best practices and commit message conventions.

You MUST follow these steps:

1. Run `git status` to see all untracked and modified files
2. Run `git diff` to see both staged and unstaged changes
3. Run `git log --oneline -10` to review recent commit message style
4. Analyze the changes and determine:
   - The nature of changes (feature, fix, refactor, docs, etc.)
   - What files SHOULD be staged
   - Any files that MUST NOT be committed (secrets, .env, etc.)
5. Stage the appropriate files using `git add`
6. **REQUIRED: Run quality checks before committing**
   - Use the Task tool with `subagent_type: "quality-orchestrator"` to run all quality checks
   - The quality-orchestrator will review changed files and automatically fix any issues
   - You MUST wait for the quality-orchestrator to complete and confirm all files pass
   - If issues cannot be resolved, you MUST NOT proceed with the commit
7. Create a commit message that:
   - Follows the repository's commit style
   - Is concise (1-2 sentences)
   - Focuses on "why" rather than "what"
   - Ends with the Claude Code attribution:
     ```
     > Generated with [Claude Code](https://claude.com/claude-code)

     Co-Authored-By: Claude <noreply@anthropic.com>
     ```
8. Execute the commit using a HEREDOC format:
   ```bash
   git commit -m "$(cat <<'EOF'
   Your commit message here.

   > Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude <noreply@anthropic.com>
   EOF
   )"
   ```
9. Run `git status` after the commit to verify success

**Important constraints:**
- You MUST ONLY use git commands (status, diff, log, add, commit, show) and the Task tool for quality checks
- You MUST NOT push to remote unless explicitly requested
- You MUST NOT use git commands with `-i` flag (interactive mode not supported)
- You MUST WARN if user tries to commit sensitive files (.env, credentials, etc.)
- You MUST NOT commit if there are no changes
- You MUST keep commit message concise and meaningful
- **You MUST run quality-orchestrator before committing** - this is non-negotiable
- If quality-orchestrator reports issues that cannot be fixed, you MUST NOT commit
