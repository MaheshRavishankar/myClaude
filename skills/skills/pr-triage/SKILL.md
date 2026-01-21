---
name: pr-triage
description: Triage outstanding PR review requests for LLVM and IREE
disable-model-invocation: true
argument-hint: "[optional: iree|llvm|all]"
allowed-tools: Bash(gh:*), Read, Grep, Glob
---

# PR Triage Skill

Analyze outstanding PR review requests and prioritize them.

## Scope

By default, check both IREE and LLVM repos. If $ARGUMENTS is provided:
- `iree` - only IREE PRs
- `llvm` - only LLVM PRs
- `all` - both (default)

## Process

### 1. Fetch Outstanding PRs

```bash
# IREE PRs where user is requested reviewer
gh pr list --repo iree-org/iree --search "review-requested:@me" --json number,title,author,createdAt,additions,deletions,url,labels,reviewDecision,comments

# LLVM PRs where user is requested reviewer
gh pr list --repo llvm/llvm-project --search "review-requested:@me" --json number,title,author,createdAt,additions,deletions,url,labels,reviewDecision,comments
```

### 2. For Each PR, Gather Quick Info

- **Size**: additions + deletions
- **Age**: days since created
- **Activity**: number of comments, whether there's already discussion
- **Review status**: pending, changes requested, approved by others

### 3. Categorize PRs

**Priority Levels:**

- **URGENT**:
  - Marked as urgent/blocker
  - Very old (>7 days) with no reviews
  - Small fix (<50 lines) waiting on you

- **NEEDS ATTENTION**:
  - Author has responded to previous comments
  - Other reviewers have approved, waiting on you
  - Medium age (3-7 days)

- **NORMAL**:
  - Recent PRs (<3 days)
  - Large PRs that need dedicated time
  - Has other reviewers actively engaged

- **LOW**:
  - Draft PRs
  - WIP or experimental
  - Already has extensive review coverage

### 4. Quick Analysis Per PR

For each PR, provide:
1. One-line summary of what it does
2. Size category (small/medium/large)
3. Key files changed (focus on areas of expertise)
4. Any red flags (breaking changes, missing tests, etc.)
5. Estimated review effort (quick/moderate/deep)

### 5. Output Format

```
## PR Review Queue - [Date]

### URGENT (X PRs)
- **#12345** [IREE] "Title" by @author (3 days, 45 lines)
  - Summary: Fixes X in Y
  - Effort: Quick review
  - Note: Author addressed your previous comments

### NEEDS ATTENTION (X PRs)
...

### NORMAL (X PRs)
...

### LOW PRIORITY (X PRs)
...

---
Total: X PRs pending review
Recommended focus: Start with #12345 and #12346
```

## Expertise Areas to Flag

When triaging, note if PR touches:
- Codegen (compiler/src/iree/compiler/Codegen/)
- LinalgExt (compiler/src/iree/compiler/Dialect/LinalgExt/)
- Stream dialect
- Dispatch creation
- Tiling/distribution
- MLIR core (for LLVM PRs)
- Linalg dialect (for LLVM PRs)

## Quick Review Hints

For PRs that look straightforward, provide a quick review hint:
- "Looks like a mechanical refactor - verify no behavior change"
- "Test-only change - check coverage"
- "New feature - check for tests and documentation"
- "Bug fix - look for regression test"

## What NOT to Do

- Don't do full reviews - this is triage only
- Don't post comments to PRs
- Don't approve or request changes
- Keep summaries brief - one line per PR
