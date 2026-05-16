# NLPM Audit: anthropics/claude-code
**Date**: 2026-04-06  |  **Artifacts**: 60  |  **Strategy**: batched
**NL Score**: 88/100
**Security**: CLEAR
**Bugs**: 7  |  **Quality Issues**: 15  |  **Security Findings**: 5

## NL Score Summary
| File | Type | Score | Top Issue |
|------|------|-------|-----------|
| plugins/agent-sdk-dev/agents/agent-sdk-verifier-ts.md | agent | 73 | No example blocks in description; missing color |
| plugins/agent-sdk-dev/agents/agent-sdk-verifier-py.md | agent | 73 | No example blocks in description; missing color |
| plugins/feature-dev/agents/code-explorer.md | agent | 75 | No example blocks in description |
| plugins/feature-dev/agents/code-architect.md | agent | 75 | No example blocks in description |
| plugins/feature-dev/agents/code-reviewer.md | agent | 77 | No example blocks in description |
| plugins/pr-review-toolkit/agents/code-simplifier.md | agent | 82 | Missing output format; no color declared |
| plugins/plugin-dev/agents/plugin-validator.md | agent | 83 | Trailing editorial artifact in system prompt |
| plugins/feature-dev/commands/feature-dev.md | command | 83 | No allowed-tools declared; vague language |
| plugins/agent-sdk-dev/commands/new-sdk-app.md | command | 83 | No allowed-tools declared; vague language |
| plugins/plugin-dev/commands/create-plugin.md | command | 84 | Heavy vague-word density capped at -16 |
| plugins/plugin-dev/agents/agent-creator.md | agent | 86 | Vague quantifiers: appropriate, relevant, comprehensive |
| plugins/code-review/commands/code-review.md | command | 86 | Vague quantifiers throughout |
| plugins/ralph-wiggum/commands/ralph-loop.md | command | 88 | Minimal body; no output format |
| plugins/plugin-dev/agents/skill-reviewer.md | agent | 88 | Vague language: substantial, appropriate, relevant |
| plugins/learning-output-style/hooks/hooks.json | hook | 88 | Missing matcher on SessionStart hook |
| plugins/explanatory-output-style/hooks/hooks.json | hook | 88 | Missing matcher on SessionStart hook |
| plugins/ralph-wiggum/commands/cancel-ralph.md | command | 90 | Minor: no argument-hint |
| plugins/hookify/hooks/hooks.json | hook | 90 | No matchers on global hooks |
| plugins/ralph-wiggum/hooks/hooks.json | hook | 90 | No matcher on Stop hook |
| plugins/pr-review-toolkit/agents/type-design-analyzer.md | agent | 90 | Invalid color "pink" (BUG) |
| plugins/claude-opus-4-5-migration/skills/claude-opus-4-5-migration/SKILL.md | skill | 90 | Description uses "Use when" not third-person form |
| plugins/frontend-design/skills/frontend-design/SKILL.md | skill | 90 | Description uses "Use this skill when" (second person) |
| plugins/hookify/commands/help.md | command | 90 | Step numbering skips 4 (typo: "1. 2. 3. 4." jumps) |
| plugins/hookify/commands/hookify.md | command | 92 | Launches general-purpose subagent instead of dedicated conversation-analyzer |
| plugins/pr-review-toolkit/agents/silent-failure-hunter.md | agent | 92 | Vague language: appropriate, specific |
| plugins/hookify/agents/conversation-analyzer.md | agent | 92 | Vague language: appropriate |
| plugins/pr-review-toolkit/agents/code-reviewer.md | agent | 92 | Vague language: specific, significant |
| plugins/pr-review-toolkit/agents/pr-test-analyzer.md | agent | 92 | Vague language: adequate, appropriate |
| plugins/pr-review-toolkit/agents/comment-analyzer.md | agent | 94 | Minor: vague (specific, relevant) |
| plugins/hookify/commands/list.md | command | 92 | Clean |
| plugins/hookify/commands/configure.md | command | 92 | Clean |
| plugins/hookify/skills/writing-rules/SKILL.md | skill | 93 | Clean |
| .claude/commands/triage-issue.md | command | 92 | Vague: appropriate, relevant |
| .claude/commands/commit-push-pr.md | command | 93 | Clean |
| .claude/commands/dedupe.md | command | 92 | No $ARGUMENTS; relies on ambient context |
| plugins/pr-review-toolkit/commands/review-pr.md | command | 94 | Minor vague language |
| plugins/ralph-wiggum/commands/help.md | command | 95 | Missing allowed-tools |
| plugins/commit-commands/commands/clean_gone.md | command | 95 | Missing allowed-tools |
| plugins/commit-commands/commands/commit-push-pr.md | command | 93 | Clean |
| plugins/commit-commands/commands/commit.md | command | 93 | Clean |
| plugins/plugin-dev/skills/plugin-structure/SKILL.md | skill | 94 | Clean |
| plugins/plugin-dev/skills/hook-development/SKILL.md | skill | 94 | Clean |
| plugins/plugin-dev/skills/skill-development/SKILL.md | skill | 93 | Clean |
| plugins/plugin-dev/skills/plugin-settings/SKILL.md | skill | 93 | Clean |
| plugins/plugin-dev/skills/agent-development/SKILL.md | skill | 93 | Clean |
| plugins/plugin-dev/skills/command-development/SKILL.md | skill | 93 | Clean |
| plugins/plugin-dev/skills/mcp-integration/SKILL.md | skill | 93 | Clean |
| plugins/security-guidance/hooks/hooks.json | hook | 93 | Clean |
| plugins/hookify/.claude-plugin/plugin.json | manifest | 93 | Clean |
| plugins/feature-dev/.claude-plugin/plugin.json | manifest | 93 | Clean |
| plugins/learning-output-style/.claude-plugin/plugin.json | manifest | 93 | Clean |
| plugins/pr-review-toolkit/.claude-plugin/plugin.json | manifest | 93 | Clean |
| plugins/ralph-wiggum/.claude-plugin/plugin.json | manifest | 93 | Clean |
| plugins/claude-opus-4-5-migration/.claude-plugin/plugin.json | manifest | 93 | Clean |
| plugins/frontend-design/.claude-plugin/plugin.json | manifest | 93 | Clean |
| plugins/explanatory-output-style/.claude-plugin/plugin.json | manifest | 93 | Clean |
| plugins/agent-sdk-dev/.claude-plugin/plugin.json | manifest | 93 | Clean |
| plugins/security-guidance/.claude-plugin/plugin.json | manifest | 93 | Clean |
| plugins/code-review/.claude-plugin/plugin.json | manifest | 93 | Clean |
| plugins/commit-commands/.claude-plugin/plugin.json | manifest | 93 | Clean |

## Security Scan
| Severity | Count |
|----------|-------|
| Critical | 0 |
| High | 0 |
| Medium | 2 |
| Low | 3 |

### Execution Surface Inventory
| Surface | Files |
|---------|-------|
| Hook JSON configs | plugins/hookify/hooks/hooks.json, plugins/learning-output-style/hooks/hooks.json, plugins/ralph-wiggum/hooks/hooks.json, plugins/explanatory-output-style/hooks/hooks.json, plugins/security-guidance/hooks/hooks.json |
| Hook scripts (Python) | plugins/hookify/hooks/pretooluse.py, plugins/hookify/hooks/posttooluse.py, plugins/hookify/hooks/stop.py, plugins/hookify/hooks/userpromptsubmit.py, plugins/security-guidance/hooks/security_reminder_hook.py, examples/hooks/bash_command_validator_example.py |
| Hook scripts (Bash) | plugins/learning-output-style/hooks-handlers/session-start.sh, plugins/explanatory-output-style/hooks-handlers/session-start.sh, plugins/ralph-wiggum/hooks/stop-hook.sh |
| Utility scripts (Bash) | plugins/ralph-wiggum/scripts/setup-ralph-loop.sh, scripts/gh.sh, scripts/comment-on-duplicates.sh, scripts/edit-issue-labels.sh, plugins/plugin-dev/skills/*/scripts/*.sh, plugins/plugin-dev/skills/*/examples/*.sh |
| Hook core modules (Python) | plugins/hookify/core/config_loader.py, plugins/hookify/core/rule_engine.py |
| Devcontainer scripts | .devcontainer/init-firewall.sh |
| Package manifests | None (no package.json or requirements.txt at repo root) |
| MCP configs | None (.mcp.json not found) |

### Security Findings
| # | Severity | File | Line | Pattern | Description |
|---|----------|------|------|---------|-------------|
| 1 | Medium | plugins/security-guidance/hooks/security_reminder_hook.py | 14 | file-write-outside-repo | DEBUG_LOG_FILE hardcoded to `/tmp/security-warnings-log.txt`; writes debug output to global /tmp directory outside the repo |
| 2 | Medium | plugins/security-guidance/hooks/security_reminder_hook.py | 131 | file-write-outside-repo | State files written to `~/.claude/security_warnings_state_{session_id}.json`; writes to user home directory outside repo |
| 3 | Low | plugins/hookify/hooks/pretooluse.py | 60-70 | broad-except-fail-open | Bare `except Exception` catches all errors and exits 0 with a systemMessage, silently allowing tool execution even on unexpected errors in hook logic |
| 4 | Low | plugins/hookify/hooks/stop.py | 47-55 | broad-except-fail-open | Same bare `except Exception` fail-open pattern; stop hook errors are silently swallowed |
| 5 | Low | plugins/ralph-wiggum/scripts/setup-ralph-loop.sh | 140-149 | user-input-heredoc | User-supplied `$PROMPT` and `$COMPLETION_PROMISE` are expanded unquoted inside a heredoc that writes to `.claude/ralph-loop.local.md`; a prompt containing `---` could corrupt YAML frontmatter parsing |

## Bugs (PR-worthy)
| # | File | Issue | Impact |
|---|------|-------|--------|
| 1 | plugins/plugin-dev/agents/plugin-validator.md | Trailing editorial artifact at end of file: "Excellent work! The agent-development skill is now complete…" — leftover conversational text from authoring session | System prompt is contaminated; agent will emit congratulatory text when invoked |
| 2 | plugins/pr-review-toolkit/agents/type-design-analyzer.md | `color: pink` in frontmatter; valid values are blue/cyan/green/yellow/magenta/red | Plugin may fail to register the agent or render with default/broken color |
| 3 | plugins/feature-dev/agents/code-explorer.md | Description has no `<example>` blocks — agent will not auto-trigger reliably | Reduced discoverability; agent must be invoked explicitly by name |
| 4 | plugins/feature-dev/agents/code-reviewer.md | Description has no `<example>` blocks | Same — no proactive triggering |
| 5 | plugins/feature-dev/agents/code-architect.md | Description has no `<example>` blocks | Same — no proactive triggering |
| 6 | plugins/agent-sdk-dev/agents/agent-sdk-verifier-ts.md | Description has no `<example>` blocks | Agent requires explicit invocation; onboarding friction |
| 7 | plugins/agent-sdk-dev/agents/agent-sdk-verifier-py.md | Description has no `<example>` blocks | Same |

## Security Fixes (PR-worthy, Medium/Low only)
| # | File | Issue | Suggested Fix |
|---|------|-------|---------------|
| 1 | plugins/security-guidance/hooks/security_reminder_hook.py | DEBUG_LOG_FILE writes to global /tmp | Move debug logging behind an env flag (`SECURITY_HOOK_DEBUG=1`) or disable by default; remove file or write to project-scoped tmp |
| 2 | plugins/security-guidance/hooks/security_reminder_hook.py | State files in ~/.claude are persistent and accumulate | Already cleaned up after 30 days; consider documenting this or exposing cleanup via `ENABLE_SECURITY_REMINDER=0` override |
| 3 | plugins/hookify/hooks/pretooluse.py | Broad `except Exception` silently swallows unexpected errors | Consider logging unexpected exception types to stderr before exiting 0, to help users diagnose rule-engine bugs |
| 4 | plugins/hookify/hooks/stop.py | Same broad except pattern | Same fix as #3 |
| 5 | plugins/ralph-wiggum/scripts/setup-ralph-loop.sh | Unquoted `$PROMPT` in heredoc could corrupt YAML if prompt contains `---` | Wrap the YAML frontmatter section in a quoted heredoc (`<<'EOF'` for the static part) and write `$PROMPT` after the closing delimiter rather than inline |

## Quality Issues (informational)
| # | File | Issue | Penalty |
|---|------|-------|---------|
| 1 | plugins/pr-review-toolkit/agents/code-simplifier.md | Missing output format specification — agent body describes what it does but not what it returns | -10 |
| 2 | plugins/agent-sdk-dev/agents/agent-sdk-verifier-ts.md | Missing `color` in frontmatter | -5 |
| 3 | plugins/agent-sdk-dev/agents/agent-sdk-verifier-py.md | Missing `color` in frontmatter | -5 |
| 4 | plugins/feature-dev/commands/feature-dev.md | Missing `allowed-tools` — command uses Task and Bash tools but does not declare them | -5 |
| 5 | plugins/ralph-wiggum/commands/help.md | Missing `allowed-tools` (informational help command, low risk) | -5 |
| 6 | plugins/agent-sdk-dev/commands/new-sdk-app.md | Missing `allowed-tools` — uses Bash, WebFetch, Write but does not declare them | -5 |
| 7 | plugins/commit-commands/commands/clean_gone.md | Missing `allowed-tools` — executes bash git commands without declaration | -5 |
| 8 | plugins/frontend-design/skills/frontend-design/SKILL.md | Description uses second-person "Use this skill when" instead of third-person "This skill should be used when" | -5 |
| 9 | plugins/claude-opus-4-5-migration/skills/claude-opus-4-5-migration/SKILL.md | Description uses "Use when" (second person) instead of "This skill should be used when" (third person) | -5 |
| 10 | plugins/plugin-dev/agents/agent-creator.md | Vague quantifiers: "appropriate" (×3), "relevant" (×2), "comprehensive" (×2) = -14 | -14 |
| 11 | plugins/plugin-dev/agents/skill-reviewer.md | Vague quantifiers: "substantial", "appropriate", "relevant", "specific" = -12 | -12 |
| 12 | plugins/feature-dev/agents/code-explorer.md | Vague quantifiers: "comprehensive" (×3), "specific" (×2) = -10 | -10 |
| 13 | plugins/code-review/commands/code-review.md | Vague quantifiers: "appropriate", "specific", "relevant", "significant" = -14 | -14 |
| 14 | plugins/hookify/commands/hookify.md | Uses `general-purpose` subagent inline instead of the dedicated `conversation-analyzer` agent from the same plugin — bypasses plugin-specific expertise | -0 (cross-component) |
| 15 | plugins/hookify/commands/help.md | Step numbering typo: "3." is followed by "4." skipping the actual step 3 label (line 171) | -2 |

## Cross-Component
| # | Finding |
|---|---------|
| 1 | **Duplicate agent name `code-reviewer`**: Both `plugins/feature-dev/agents/code-reviewer.md` and `plugins/pr-review-toolkit/agents/code-reviewer.md` declare `name: code-reviewer`. When both plugins are installed simultaneously, one will shadow the other, making the shadowed agent unreachable. |
| 2 | **hookify command bypasses dedicated agent**: `plugins/hookify/commands/hookify.md` launches an inline `general-purpose` subagent prompt to analyze conversations rather than using the `conversation-analyzer` agent in `plugins/hookify/agents/conversation-analyzer.md`. The dedicated agent has richer analysis patterns and is the canonical entry point — the command should delegate to it. |
| 3 | **code-review command depends on external unconfigured MCP**: `plugins/code-review/commands/code-review.md` references `mcp__github_inline_comment__create_inline_comment` in its allowed-tools and uses it in step 9, but no MCP server providing this tool is declared in `plugins/code-review/.claude-plugin/plugin.json` or `.mcp.json`. Users who install this plugin without the corresponding MCP server will hit a silent runtime failure at step 9. |

## Recommendation
CLEAR — submit PRs for all bugs and medium/low security fixes.

The repo scores 88/100. Security is clean with no Critical or High findings. Seven bugs are all straightforward fixes (artifact cleanup, one invalid color string, five agents missing `<example>` blocks). The two Medium security findings in `security_reminder_hook.py` are low-risk (expected behavior, no credentials involved) and can be addressed with a simple env-flag guard on debug logging.

Priority order:
1. Fix `plugin-validator.md` trailing text (corrupts system prompt)
2. Fix `type-design-analyzer.md` invalid color `pink`
3. Add `<example>` blocks to the five example-free agents
4. Address Medium security findings (debug log to /tmp)
5. Resolve the `code-reviewer` agent name collision across plugins
6. Fix hookify command to delegate to `conversation-analyzer` agent
7. Document the external MCP dependency in `code-review` plugin
