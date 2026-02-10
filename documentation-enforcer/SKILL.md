---
name: documentation-enforcer
description: |
  Use when reviewing code for documentation quality, clarity, and organization.
  Reviews all source files and generates unified diffs with suggested comments/
  docstrings and a review summary. Clarifies ambiguous intent before generating
  suggestions. Outputs diffs only; does not modify files automatically.
---

# Documentation Enforcer Skill

## Purpose
This skill reviews the source code in a repository to ensure:
- All functions, classes, modules, and logic blocks are well documented.
- Docstrings/comments explain *why* logic exists, expected behaviors, and edge cases.
- Documentation follows best practices appropriate to the language.
- Changes are output as **unified diffs** and a summary report *without modifying files*.

## When to Use
Use this skill when preparing code for review or commit, or when ensuring that
code is maintainable and understandable for future developers.

Suggested trigger phrases:
- Review code for documentation quality
- Ensure excellent documentation across the codebase
- Provide documentation diff and summary
- Explain logic and edge cases

## Workflow

1. **Scan All Source Files**
   - Walk the directory tree.
   - Detect function, class, and module definitions.
   - Flag missing or inadequate docstrings/comments.

2. **Ask Clarifying Questions**
   If any intent, purpose, or edge behavior is unclear:
   - Ask specific questions (e.g., “What is the intended behavior of `foo()`?”).
   - Wait for responses before continuing the review.

3. **Generate Suggested Diffs**
   - For each file that needs documentation improvements:
     - Produce unified diffs that add or improve docstrings/comments.
     - Follow common best practices per language (e.g., JSDoc for JS, Python docstrings).
     - Include rationale in diff comments for clarity.

4. **Summarize Findings**
   - List reviewed files.
   - Highlight changes suggested.
   - Note any remaining unclear items needing manual attention.

## Output Examples

### Unified Diff Example
```diff
--- a/src/utils.py
+++ b/src/utils.py
@@ -12,6 +12,18 @@ def compute_score(x, y):
     result = x * y

+    # Document purpose of fallback handling:
+    # When x or y are negative, we apply a stable fallback to
+    # ensure consistent results. This ensures that edge logic
+    # remains explicit for future maintainers.
+
+    # Expected parameters:
+    # - x: non-negative numeric
+    # - y: non-negative numeric
+    # Clarify if negative inputs should raise exceptions.
```

## Summary Example
### Documentation Enforcer Summary
------------------------------
Reviewed 24 files
Generated diff suggestions for 8 files
Clarified ambiguous logic in:
- compute_score (needs parameter intent)
Outstanding items:
- Clarify intended behavior for parse_input edge cases
Please review diffs and apply manually.


## Best Practices
- Comments explain why, not just what.
- Language-appropriate docstring formats.
- Explicit edge case explanations.
- Ask questions when information is missing.

## Clarifying Question Protocol
If intent is ambiguous:
1. Pause the analysis.
2. Ask focused questions about goals or expected behaviors.
3. Resume after user clarification.