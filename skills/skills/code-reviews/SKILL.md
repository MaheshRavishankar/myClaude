---
name: code-reviews
description: Review PRs for LLVM and IREE projects
disable-model-invocation: true
argument-hint: "[pr-url or pr-number]"
allowed-tools: Bash(gh:*), Read, Grep, Glob, WebFetch
---

# Code Review Skill for LLVM/IREE

Review the pull request specified by $ARGUMENTS.

## Determine PR Source

- If the argument is a GitHub URL, extract the repo and PR number
- If it's just a number, determine context from current working directory:
  - IREE repo: use `iree-org/iree`
  - LLVM repo: use `llvm/llvm-project`

## Review Process

1. **Fetch PR Information**
   ```bash
   gh pr view <number> --repo <repo>
   gh pr diff <number> --repo <repo>
   gh pr view <number> --repo <repo> --comments
   ```

2. **Analyze Changes**
   - Understand the purpose from PR description
   - Review each changed file
   - Check test coverage

3. **LLVM/IREE Specific Checks**
   - TableGen definitions (.td files): Verify operation definitions, constraints
   - C++ passes: Check for proper pattern matching, error handling
   - MLIR tests (.mlir files): Verify FileCheck patterns, edge cases
   - CMake/Bazel: Build system consistency

## Review Criteria

### Code Quality
- Logic correctness and edge case handling
- Memory management (especially in C++ runtime code)
- Error handling with proper status propagation
- Thread safety where applicable

### MLIR/Compiler Patterns
- Proper use of OpBuilder and rewriter patterns
- Correct dialect dependencies
- Appropriate use of interfaces vs traits
- Pass registration and pipeline integration

### Testing
- Unit tests for new functionality
- Lit tests for compiler passes
- Edge cases covered
- Negative tests where appropriate

### Style
- Follows LLVM/IREE coding conventions
- Comments explain "why" not "what"
- No unnecessary changes outside PR scope

## Output Format

Provide a structured review:

### Summary
Brief description of what the PR does.

### Positive Aspects
What the PR does well.

### Issues Found
- **Critical**: Must fix before merge
- **Major**: Should fix, may block merge
- **Minor**: Suggestions for improvement
- **Nit**: Style/preference (optional)

### Questions
Any clarifications needed from the author.

### Verdict
Overall assessment: Approve / Request Changes / Need Discussion
