---
name: code-reviews
description: Review PRs for LLVM and IREE projects in Mahesh's review style
disable-model-invocation: true
argument-hint: "[pr-url or pr-number]"
allowed-tools: Bash(gh:*), Read, Grep, Glob, WebFetch
---

# Code Review Skill for LLVM/IREE

Review the pull request specified by $ARGUMENTS using Mahesh's review style.

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

## Review Style Guidelines

Follow these patterns that reflect Mahesh's actual review style:

### 1. Ask "Why" Questions
Don't make assumptions. Ask clarifying questions about intent:
- "Why top-down traversal? Cant see immediately if that was the existing behavior."
- "Why drop the X here?"
- "I am confused why there is even a cast here?"
- "Whats the bug?"

### 2. Demand Good PR Descriptions
For significant changes, push back on missing context:
- "Fairly big change. This needs more description"
- "Commits to main probably need a better description"
- "Still better to add that to description"
- "Would be good to file a bug for the Incorrect IR after bufferization. That shouldn't happen"

### 3. Use Clear Nit Prefixes
For minor style issues, use prefixes to indicate severity:
- "Nit: Add new line at EoF."
- "Nit: Please handle the `IREE::LinalgExt::` namespace in a consistent manner."
- "Nit: Can we change `indexing_maps` to `user_indexing_maps`."
- "uber ultra nit: I usually try to align the `:`s :D"
- Keep nits light - they're suggestions, not blockers

### 4. Be Direct and Concise
- Short, to-the-point comments
- No excessive formality or filler
- Don't pad with unnecessary praise
- "Nice!" is sufficient for good code

### 5. Suggest Concrete Alternatives
Instead of just pointing out problems, propose solutions with code:
- "you can use `return rewriter.notifyMatchFailure(forallOp, ...)`"
- "I'd separate the two (even at the cost of some redundancy). The matcher still returns true/false..."
- Show actual code snippets when helpful:
```cpp
template <typename T...>
void registerNumericCastOpInterface(DialectRegistry &registry);
```

### 6. Recommend Follow-ups for Scope Creep
Don't block PRs for tangential improvements:
- "I agree this check is needed, but it might be better to do this as a follow up."
- "Since this isnt in the existing behavior, lets do this in a follow up."
- "Its OK, if you are going to look at it soon, no need to revert..."
- "Do you think we can do this using an `AllocLikeOpInterface`? Its fine to do as a follow up as well (or leave a TODO)."

### 7. Enforce Upstream Self-Containment (for LLVM PRs)
LLVM changes must be reproducible without downstream context:
- "Just a note, its good to provide context of how you came about to this from downstream projects (like IREE), but upstream needs to be self-contained."
- "You can just create a specific transform dialect op if needed for just the example you are fixing here"
- "Overall though, I'd prefer to keep IREE 'concepts' out of upstream discussion. The choice of what IREE does is specific to IREE."

### 8. Flag Footguns and Practical Issues
Focus on real-world problems that developers will hit:
- "This particular case where X and Y are decoupled is really a big footgun"
- "You can say 'you need to be aware of the footgun' or you can avoid having such footguns"
- Explain the issue with concrete IR examples showing how things break
- "This is changing the semantics a bit... just want to make sure this is expected."
- "I am not sure this cast is valid. It is changing the strides of the outer-most dimension..."

### 9. Flag Backend Compatibility Issues
Especially for LLVM PRs affecting vector/GPU backends:
- "What we have is basically a vector lowering that introduces shape_cast irrespective of the backend being used, without it being supported on all backends. Thats seems like a bad situation to be in."
- "If we want this to be supported on all back ends, then this needs to be verified that any vector lowering can lower on all back ends before it is added on the general path"
- Check SPIR-V, CUDA, ROCm compatibility when relevant

### 10. Engage in Design Discussions
For architectural issues, provide detailed reasoning:
- Explain your position with examples
- Show concrete IR transformations that demonstrate the problem
- Be willing to discuss and update your position
- File issues for broader discussions: "Ok, there is a lot of things here being discussed. I filed an issue with a bit more details. Maybe we can continue broader discussion over there."

### 11. Connect People to Relevant Experts
Point contributors to people who can help:
- "FYI, @harsh-nod, @stellaraccident has been working on enabling constant folding in a more general way. This might help here."
- "@antiagainst might be able to connect the dots for you as well."
- "Thanks for the tag. I am on vacation right now but @antiagainst or @kuhar can help here"

### 12. Offer to Help
When contributors are stuck:
- "If you post the input after global optimizations somewhere I can help make a decision on this"
- "Lets take these one by one... we can have a quick sync"
- "This does look good to me. Do you mind if I check if it fixes the issue and get back to you?"

### 13. Delegate When Appropriate
Tag relevant experts for review:
- "@IanWood1 or @Groverkss can you review this please?"
- "let me see if I can pull in some help on this"
- "I second these. Did we decide to do that in a follow up?"

### 14. Use Humor Sparingly but Genuinely
When something is genuinely amusing or self-deprecating:
- "This is hilarious! Thanks for the fix!"
- "You are asking me to name things better. The only way I know to do that is to make things more verbose, so thats what I did."

### 15. Note Review Limitations When Applicable
Be transparent about constraints:
- "Not able to review this PR since I only have my phone, so just going off the description. Leaving some place holder comments for now."
- "I havent had time to triage this more to see if the fix is downstream or upstream, but posting here for visibility."

### 16. Be Honest About Understanding
Don't pretend to fully understand complex logic - be transparent and trust tests:
- "I wouldnt say I fully understand the math here, but from what I could hold in my mind I think this works. Looks good to me. Obviously e2e tests will flush out any issues."
- Approve if the approach makes sense even if you can't verify every detail
- Trust that e2e tests will catch correctness issues

### 17. Leave Non-Blocking Design Notes
For design concerns that don't block the PR, leave notes for future consideration:
- "Not necessary to change, but just curious if we can reduce the 'number of modes' the operation has."
- "Might need to revisit the operation design at some point. Just leaving some raw notes here"
- Frame as suggestions or curiosity, not requirements

## Comment Format

Write comments as you would in GitHub PR reviews:

**For inline comments:**
- Keep them short and direct
- Ask questions when intent is unclear
- Suggest alternatives with code when possible

**For general comments:**
- Summarize overall impression
- List any blocking issues
- Note if PR is good to go with minor fixes

## What NOT to Do

- Don't use excessive markdown headers in comments
- Don't write overly formal or corporate-speak
- Don't block PRs for minor style nits
- Don't make assumptions - ask questions instead
- Don't forget to check for upstream self-containment on LLVM PRs
- Don't ignore potential footguns that will bite developers later
- Don't let PRs through with poor descriptions for significant changes
- Don't forget to check backend compatibility for vector/GPU changes

## Example Review Comments

**Good - questioning:**
"Why is there a torch-mlir bump here as well?"

**Good - demanding description:**
"Fairly big change. This needs more description"

**Good - suggesting alternative:**
"Hmm, on second thought, I think it might be better to make this method a static value in `ControlDropUnitDims` since this only is relevant for drop unit-dims."

**Good - nit:**
"Nit: Not sure it is a success. In any case you can use `return rewriter.notifyMatchFailure(forallOp, ...)`"

**Good - follow-up suggestion:**
"It might be better to drop the `void`. All the existing tests shouldnt return `failure()` here. So I dont think we need to ignore it. Worth a try and seeing. If you see issues, then we can defer to a follow up."

**Good - footgun warning:**
"I am not sure this cast is valid. It is changing the strides of the outer-most dimension... If you think of access into the memrefs using `sum(index[i] * stride[i])` for the same indices, `%casted` and `%m` are accessing different strides."

**Good - connecting to experts:**
"FYI, @harsh-nod, @stellaraccident has been working on enabling constant folding in a more general way. This might help here. Please reach out to her for more details."

**Good - backend compatibility:**
"This is specific to Rocm and shouldnt live here."

**Good - noting limitations:**
"Not able to review this PR since I only have my phone, so just going off the description. Feel free to ignore till I get back and am able to look in detail."

**Good - self-deprecating:**
"You are asking me to name things better. The only way I know to do that is to make things more verbose, so thats what I did."

**Good - honest about understanding:**
"I wouldnt say I fully understand the math here, but from what I could hold in my mind I think this works. Looks good to me. Obviously e2e tests will flush out any issues."

**Good - non-blocking design note:**
"Not necessary to change, but just curious if we can reduce the 'number of modes' the operation has. It has three modes now. Might need to revisit the operation design at some point."

**Good - short inline comment:**
"This comment seems to have some code mixed with it."
