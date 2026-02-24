---
name: pr-triage
description: Triage outstanding PR review requests and do preliminary reviews for urgent ones
disable-model-invocation: true
argument-hint: "[optional: iree|llvm|all]"
allowed-tools: Bash(gh:*), Read, Grep, Glob, WebFetch
---

# PR Triage Skill

Analyze outstanding PR review requests, prioritize them, and do preliminary reviews for urgent PRs.

## Scope

By default, check both IREE and LLVM repos. If $ARGUMENTS is provided:
- `iree` - only IREE PRs
- `llvm` - only LLVM PRs
- `all` - both (default)

## Process

### Phase 1: Fetch and Categorize

#### 1.1 Fetch Outstanding PRs

```bash
# IREE PRs where user is requested reviewer
gh pr list --repo iree-org/iree --search "review-requested:@me" --json number,title,author,createdAt,additions,deletions,url,labels,reviewDecision,comments

# LLVM PRs where user is requested reviewer
gh pr list --repo llvm/llvm-project --search "review-requested:@me" --json number,title,author,createdAt,additions,deletions,url,labels,reviewDecision,comments
```

#### 1.2 Categorize PRs

**Priority Levels:**

- **URGENT** (will get preliminary review):
  - Marked as urgent/blocker
  - Very old (>7 days) with no reviews
  - Small fix (<100 lines) waiting on you
  - Author has responded to previous comments

- **NEEDS ATTENTION**:
  - Other reviewers have approved, waiting on you
  - Medium age (3-7 days)
  - Medium size (100-500 lines)

- **NORMAL**:
  - Recent PRs (<3 days)
  - Large PRs that need dedicated time
  - Has other reviewers actively engaged

- **LOW**:
  - Draft PRs
  - WIP or experimental
  - Already has extensive review coverage

### Phase 2: Preliminary Reviews (Top 2-3 Urgent PRs)

For each PR in the URGENT category (up to 3), do a quick preliminary review:

#### 2.1 Fetch PR Details
```bash
gh pr view <number> --repo <repo>
gh pr diff <number> --repo <repo>
gh pr view <number> --repo <repo> --comments
```

#### 2.2 Quick Analysis

Apply Mahesh's review style principles:

**Check for:**
1. **Missing context** - Does PR description explain the "why"?
2. **Test coverage** - Are there tests for the changes?
3. **Obvious issues** - Logic errors, missing error handling
4. **Footguns** - Potential issues that will bite developers later
5. **Scope creep** - Changes that should be follow-ups

**Flag with questions:**
- "Why X?" for unclear design decisions
- "Is this intentional?" for potentially problematic changes
- "Missing test for..." for untested code paths

**Note areas of expertise:**
- Codegen changes
- LinalgExt/Linalg changes
- Tiling/distribution
- Stream dialect
- MLIR patterns

#### 2.3 Preliminary Review Output

For each urgent PR, provide:
```
### PR #12345: "Title"
**Author:** @author | **Size:** +45/-10 | **Age:** 5 days

**Summary:** One paragraph explaining what the PR does.

**Quick Assessment:**
- [ ] Description adequate
- [ ] Tests included
- [ ] No obvious issues

**Concerns to investigate:**
1. Line 42 in file.cpp: Why is X done this way?
2. Missing test for edge case Y
3. This changes behavior of Z - intentional?

**Review effort:** Quick / Moderate / Deep dive needed
```

### Phase 3: Output Summary

```
# PR Review Queue - [Date]

## Summary
- Total PRs pending: X
- Urgent (reviewed below): Y
- Needs attention: Z
- Can wait: W

---

## Preliminary Reviews (Urgent PRs)

[Detailed preliminary reviews for top 2-3 urgent PRs]

---

## Other PRs Needing Attention

| PR | Repo | Title | Author | Age | Size | Note |
|----|------|-------|--------|-----|------|------|
| #123 | IREE | "Fix X" | @foo | 4d | S | Author responded |
| #456 | LLVM | "Add Y" | @bar | 5d | M | Other approvals |

---

## Normal Priority

[Brief list]

---

## Low Priority

[Brief list]

---

## Recommended Action

1. **Start with PR #12345** - Small fix, author waiting on response
2. **Then PR #12346** - You're the blocking reviewer
3. **Schedule time for PR #12347** - Large change, needs deep review
```

## Review Style Guidelines (from code-reviews skill)

When doing preliminary reviews, apply these patterns:

- **Ask "why" questions** - Don't assume, ask for clarification
- **Flag potential footguns** - Issues that will bite developers later
- **Note missing descriptions** - "Fairly big change. This needs more description"
- **Suggest follow-ups** - Don't try to fix everything in one PR
- **Check upstream self-containment** (LLVM PRs) - Must be reproducible without IREE

## What This Skill Does NOT Do

- Post comments to PRs (triage only, you decide what to post)
- Approve or request changes
- Do exhaustive line-by-line review (that's for focused review time)
- Review more than 3 PRs in depth (keeps triage quick)
