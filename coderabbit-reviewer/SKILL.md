---
name: coderabbit-reviewer
description: |
  Use when processing CodeRabbit review comments on an open pull request.
  Fetches PR comments, triages each CodeRabbit suggestion, applies fixes
  (one commit per comment), runs tests, replies with commit references,
  and resolves review threads. Does not bundle fixes.
---

# CodeRabbit Reviewer Skill

## Purpose
This skill processes CodeRabbit review comments on the current branch’s PR
in a safe, structured, and auditable way.

It ensures:
- Each review comment is evaluated against the current code state.
- Valid fixes are implemented with proper test coverage.
- Exactly **one commit per comment**.
- Review threads are replied to and resolved.
- A final summary table is produced.

---

## When to Use

Use this skill when:
- A PR has automated CodeRabbit feedback.
- You want clean, reviewable commits.
- You need structured triage and resolution of review threads.

Suggested trigger phrases:
- Process CodeRabbit comments
- Resolve CodeRabbit review
- Apply CodeRabbit fixes
- Triage PR review comments

---

## Workflow

### 1. Locate the Pull Request

```bash
gh pr list --head <current-branch> --json number,title,url
```

---

### 2. Fetch Review Comments

```bash
gh api repos/Handshaik/handshaik-app/pulls/<PR_NUMBER>/comments \
  --jq '.[] | {id: .id, path: .path, line: .line, body: .body, user: .user.login, in_reply_to_id: .in_reply_to_id}'
```

Filter rules:
- Only process **top-level comments** (`in_reply_to_id: null`)
- Only process comments from `coderabbitai[bot]`
- Skip comments already resolved by a human reply (e.g., “Risk accepted”)
- Skip comments already addressed in a prior push

---

### 3. Validate Current Code State

Before making changes:
- Read the referenced file.
- Inspect the exact lines mentioned.
- Confirm the issue still exists.

Never assume the comment is still valid.

---

### 4. Triage Each Comment

Classify:

| Classification | Action |
|---------------|--------|
| Valid fix needed | Implement fix |
| Already resolved by human reply | Skip (note in summary) |
| Already fixed by prior push | Skip (note in summary) |
| Disagree / Not applicable | Flag to user for decision |

If unsure, ask the user before proceeding.

---

### 5. Fix, Test, Commit (One Commit Per Comment)

⚠️ Do NOT bundle multiple fixes.

For each valid comment:

#### 1. Implement Minimal Fix
- Modify only relevant files.
- Avoid unrelated refactors.

#### 2. Update Tests (If Needed)
- Add or adjust tests to cover change.

#### 3. Run Tests

```bash
uv run pytest src/apps/<module>/tests/ -v --tb=short
```

All tests must pass before committing.

#### 4. Commit With Clear Message

```bash
git add <files> && git commit -m "$(cat <<'EOF'
fix(HAN-XXXX): Short description of fix

Explanation of the change and why it addresses the review comment.

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

Commit rules:
- One commit per review comment
- Reference ticket (HAN-XXXX)
- Clear reasoning in body

---

### 6. Push Commits

```bash
git push
```

---

### 7. Reply to Each Review Comment

If fixed:

```bash
gh api repos/Handshaik/handshaik-app/pulls/<PR_NUMBER>/comments/<COMMENT_ID>/replies \
  -X POST -f body="Fixed in <short-sha> — <brief explanation>."
```

If already resolved or accepted:

```bash
gh api repos/Handshaik/handshaik-app/pulls/<PR_NUMBER>/comments/<COMMENT_ID>/replies \
  -X POST -f body="Already resolved — <reason>."
```

---

### 8. Resolve Review Threads

Fetch threads:

```bash
gh api graphql -f query='query {
  repository(owner: "Handshaik", name: "handshaik-app") {
    pullRequest(number: <PR_NUMBER>) {
      reviewThreads(first: 20) {
        nodes {
          id
          isResolved
          comments(first: 1) { nodes { databaseId } }
        }
      }
    }
  }
}'
```

Resolve unresolved threads:

```bash
gh api graphql -f query='mutation {
  resolveReviewThread(input: {threadId: "<THREAD_ID>"}) {
    thread { isResolved }
  }
}'
```

Note:
CodeRabbit often auto-resolves threads after detecting a fix.
Always check `isResolved` before manually resolving.

---

## Final Output

Produce a summary table:

| # | Comment | Commit | Status |
|---|---------|--------|--------|
| 1 | Description | `<sha>` | Resolved |
| 2 | Description | N/A — accepted by reviewer | Resolved |
| ... | ... | ... | ... |

---

## Best Practices

- Never assume a comment is still valid.
- One commit per comment.
- Keep diffs minimal and reviewable.
- Do not silently ignore disagreements — escalate to user.
- Always produce a final structured summary.
