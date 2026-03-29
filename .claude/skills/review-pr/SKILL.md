---
name: review-pr
description: Run parallel code quality, test quality, and security reviews against local changes or branch diff. Use when the user wants to review their work before committing or creating a PR.
---

# Review PR

Run three parallel code reviews (code quality, test quality, security) and present a unified summary with proposed next actions.

## Step 1: Detect Review Scope

Determine what to review by inspecting git state:

```bash
# Check for uncommitted changes
git diff --stat HEAD

# Check for branch commits ahead of main
git log --oneline main..HEAD
```

**Decision logic:**

- If uncommitted changes exist but NO branch commits → scope is **uncommitted changes**. Generate diff with: `git diff HEAD`
- If branch commits exist (with or without uncommitted changes) → scope is **branch diff**. Generate diff with: `git diff main`
- If neither → print "Nothing to review." and stop.

Save the diff output and the changed file list (`git diff --name-only` with the same arguments) for use in Step 3.

Print the detected scope to the user:

> "Reviewing [uncommitted changes | branch diff against main] — [N] files changed."

## Step 2: Run Coverage

Run the test suite with coverage before dispatching reviewers:

```bash
npm run test:coverage:generate 2>&1 | tee /tmp/coverage-text.txt
```

After the command completes, check for coverage output:

- `/tmp/coverage-text.txt`
- `coverage/coverage-summary.json` (if generated)

If the coverage run fails, print a warning and continue:

> "Warning: Coverage run failed. Reviews will proceed without coverage data."

## Step 3: Read Review Prompts

Read the three prompt files from the skill directory:

- `.claude/skills/review-pr/prompts/code-quality.md`
- `.claude/skills/review-pr/prompts/test-quality.md`
- `.claude/skills/review-pr/prompts/security.md`

## Step 4: Dispatch Parallel Review Agents

Dispatch three agents **in parallel** using the Agent tool. Each agent receives its review prompt, the diff, and the file list as context.

**IMPORTANT:** All three Agent calls MUST be in a single message to run in parallel.

### Agent 1: Code Quality

```
Description: "Review code quality"
Prompt: |
  <review-prompt>
  [contents of prompts/code-quality.md]
  </review-prompt>

  <diff>
  [diff output from Step 1]
  </diff>

  <changed-files>
  [file list from Step 1]
  </changed-files>

  <coverage-text>
  [contents of coverage text, or "Coverage data not available." if missing]
  </coverage-text>
```

### Agent 2: Test Quality

```
Description: "Review test quality"
Prompt: |
  <review-prompt>
  [contents of prompts/test-quality.md]
  </review-prompt>

  <diff>
  [diff output from Step 1]
  </diff>

  <changed-files>
  [file list from Step 1]
  </changed-files>

  <coverage-text>
  [contents of coverage text, or "Coverage data not available." if missing]
  </coverage-text>
```

### Agent 3: Security

```
Description: "Review security"
Prompt: |
  <review-prompt>
  [contents of prompts/security.md]
  </review-prompt>

  <diff>
  [diff output from Step 1]
  </diff>

  <changed-files>
  [file list from Step 1]
  </changed-files>
```

Note: The security agent does NOT receive coverage data.

## Step 5: Summarize Results

After all three agents return, present a unified summary:

```markdown
## Review Summary

| Review       | Grade | Key Findings |
| ------------ | ----- | ------------ |
| Code Quality | [X]   | [N findings] |
| Test Quality | [X]   | [N findings] |
| Security     | [X]   | [N findings] |

---

### Code Quality (Grade: [X])

[condensed findings from agent 1]

### Test Quality (Grade: [X])

[condensed findings from agent 2]

### Security (Grade: [X])

[condensed findings from agent 3]

---

### Proposed Next Actions

- [ ] [Actionable item derived from finding] ([Review Name] #[N])
- [ ] [Actionable item derived from finding] ([Review Name] #[N])
```

**Rules for Proposed Next Actions:**

- Only include actionable items (Critical, High, Medium severity or grade-impacting findings)
- Omit Info-level and minor suggestions
- Order by priority: Critical first, then High, then Medium
- Each item references the source review and finding number
