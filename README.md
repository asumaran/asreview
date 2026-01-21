# asreview

A comprehensive PR review toolkit for Claude Code with specialized parallel agents.

## Features

- Analyzes only PR/branch changes
- Verifies PR description matches actual code changes
- Evaluates architecture and suggests better approaches
- Security vulnerability detection (XSS, SQL injection, etc.)
- Comment accuracy and maintainability analysis
- Test coverage review
- Silent failure detection
- Type design analysis
- General code quality review
- Code simplification suggestions

## Installation

Add to your Claude Code settings (`~/.claude/settings.json`):

```json
{
  "plugins": [
    "/path/to/asreview"
  ]
}
```

Or for project-specific use, add to `.claude/settings.json` in your project.

## Usage

### Full PR Review

```
/asreview:review
```

Or with a specific PR:

```
/asreview:review 458
/asreview:review https://github.com/owner/repo/pull/458
```

### Review Specific Aspects

```
/asreview:review security      # Security vulnerabilities only
/asreview:review tests         # Test coverage only
/asreview:review architecture  # Architecture review only
```

## Agents

The plugin includes specialized agents that run in parallel:

| Agent | Focus |
|-------|-------|
| `pr-description-checker` | Verifies PR description vs actual changes |
| `architecture-reviewer` | Evaluates design and suggests alternatives |
| `security-reviewer` | XSS, injection, OWASP vulnerabilities |
| `comment-analyzer` | Comment accuracy and documentation |
| `test-reviewer` | Test coverage quality and completeness |
| `error-handler-reviewer` | Silent failures and error handling |
| `type-reviewer` | Type design and invariants |
| `code-reviewer` | General code quality and guidelines |
| `code-simplifier` | Clarity and maintainability improvements |

## Output Format

Results are grouped by severity:

1. **Critical Issues** - Must fix before merge
2. **Important Issues** - Should fix
3. **Suggestions** - Nice to have
4. **Positive Observations** - What's well done

Each finding includes:
- File path and line number
- Issue description
- Recommended action

## License

MIT
