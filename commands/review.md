# Comprehensive PR Review

Run a comprehensive pull request review using multiple specialized agents in parallel.

**Arguments:** "$ARGUMENTS"

## Review Workflow

### Step 1: Identify the PR

First, determine which PR to review:

1. If a PR number or URL is provided in arguments, use that
2. Otherwise, check if current branch has an open PR: `gh pr view --json number,title,body,url`
3. If no PR exists, review the current branch changes against main

### Step 2: Get PR Information

```bash
# Get PR details (adjust number as needed)
gh pr view <number> --json title,body,files,additions,deletions,baseRefName,headRefName

# Get the diff
gh pr diff <number>

# List changed files
gh pr diff <number> --name-only
```

### Step 3: Launch Review Agents in Parallel

Launch ALL of the following agents simultaneously using the Task tool. Each agent should receive:
- The PR number or branch name
- The list of changed files
- Instructions to focus only on the PR changes

**Agents to launch (ALL in parallel):**

1. **pr-description-checker** - Verify PR description matches actual changes
2. **architecture-reviewer** - Evaluate design and suggest better approaches
3. **security-reviewer** - Find security vulnerabilities
4. **comment-analyzer** - Check comment accuracy and documentation
5. **test-reviewer** - Review test coverage
6. **error-handler-reviewer** - Find silent failures
7. **type-reviewer** - Analyze type design (if TypeScript files changed)
8. **code-reviewer** - General code quality
9. **code-simplifier** - Suggest simplifications

For each agent, provide this context:
- PR number: X
- Changed files: [list]
- Focus: Only analyze the changes in this PR, not the entire codebase

### Step 4: Aggregate Results

After all agents complete, compile their findings into a single report:

```markdown
# PR Review Summary

**PR:** #<number> - <title>
**Author:** <author>
**Branch:** <head> -> <base>
**Files Changed:** <count> (+<additions>/-<deletions>)

---

## Critical Issues (X found)

Must fix before merge.

| # | Agent | Issue | Location | Action |
|---|-------|-------|----------|--------|
| 1 | security-reviewer | SQL injection vulnerability | file.ts:42 | Use parameterized query |
| 2 | code-reviewer | Null pointer exception | api.ts:15 | Add null check |

---

## Important Issues (X found)

Should fix before merge.

| # | Agent | Issue | Location | Action |
|---|-------|-------|----------|--------|
| 1 | error-handler-reviewer | Silent failure in catch | service.ts:88 | Log and rethrow |

---

## Suggestions (X found)

Nice to have improvements.

| # | Agent | Suggestion | Location |
|---|-------|------------|----------|
| 1 | code-simplifier | Can simplify nested conditionals | utils.ts:23 |
| 2 | type-reviewer | Consider using branded type | types.ts:5 |

---

## PR Description Accuracy

<Result from pr-description-checker>

- [ ] All claimed changes are implemented
- [ ] No undocumented changes
- Accuracy Score: X/10

---

## Test Coverage

<Summary from test-reviewer>

- Coverage estimate: X%
- Missing critical tests: X
- Test quality: GOOD/ADEQUATE/POOR

---

## Security Assessment

<Summary from security-reviewer>

- Risk Level: CRITICAL/HIGH/MEDIUM/LOW/NONE
- Vulnerabilities: X critical, X high, X medium, X low

---

## Positive Observations

What's done well in this PR:

- <From various agents>
- <Good patterns found>
- <Well-tested areas>

---

## Action Plan

### Before Merge (Required)

1. [ ] Fix: <Critical issue 1> at file:line
2. [ ] Fix: <Critical issue 2> at file:line

### Recommended

3. [ ] Address: <Important issue 1> at file:line
4. [ ] Address: <Important issue 2> at file:line

### Optional

5. [ ] Consider: <Suggestion 1> at file:line

---

## Review Metadata

| Agent | Status | Findings |
|-------|--------|----------|
| pr-description-checker | Done | X issues |
| architecture-reviewer | Done | X issues |
| security-reviewer | Done | X issues |
| comment-analyzer | Done | X issues |
| test-reviewer | Done | X issues |
| error-handler-reviewer | Done | X issues |
| type-reviewer | Done | X issues |
| code-reviewer | Done | X issues |
| code-simplifier | Done | X suggestions |
```

## Usage Examples

**Review current branch's PR:**
```
/asreview:review
```

**Review specific PR:**
```
/asreview:review 458
/asreview:review https://github.com/owner/repo/pull/458
```

## Notes

- All agents run in parallel for faster reviews
- Each agent focuses on its specialty for deep analysis
- Results include file:line references for easy navigation
- Action plan prioritizes by severity
