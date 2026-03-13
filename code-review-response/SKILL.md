---
name: code-review-response
description: |
  Use when processing review comments on an open pull request.
  Fetches PR comments, triages each suggestion (from any reviewer — human,
  CodeRabbit, Sentry, or other bots), applies fixes (one commit per comment),
  runs tests, replies with commit references, and resolves review threads.
  Does not bundle fixes.
---

# PR Review Comment Resolver

## Purpose
This skill processes **all open review comments** on the current branch's PR
in a safe, structured, and auditable way — regardless of who left the comment
(human reviewers, CodeRabbit, Sentry, or any other bot).

It ensures:
- Each review comment is evaluated against the current code state.
- Valid fixes are implemented with proper test coverage.
- Exactly **one commit per comment**.
- Review threads are replied to and resolved.
- A final summary table is produced.

---

## When to Use

Use this skill when:
- A PR has unresolved review comments from any source.
- You want clean, reviewable commits tied to specific feedback.
- You need structured triage and resolution of review threads.

Suggested trigger phrases:
- Process PR review comments
- Resolve review comments
- Apply review fixes
- Triage PR feedback

---

## Workflow

### 1. Identify the Repository and Pull Request

Detect the repo from the current git remote:

```bash
gh repo view --json nameWithOwner --jq '.nameWithOwner'
```

Find the PR for the current branch:

```bash
gh pr list --head "$(git branch --show-current)" --json number,title,url
```

Store the `OWNER/REPO` and `PR_NUMBER` for use in all subsequent commands.

---

### 2. Fetch Review Comments

```bash
gh api repos/{owner}/{repo}/pulls/{PR_NUMBER}/comments \
  --jq '.[] | {id: .id, path: .path, line: .line, body: .body, user: .user.login, in_reply_to_id: .in_reply_to_id}'
```

Also fetch top-level PR review bodies (these are not line-level comments):

```bash
gh api repos/{owner}/{repo}/pulls/{PR_NUMBER}/reviews \
  --jq '.[] | select(.body != "" and .body != null) | {id: .id, body: .body, user: .user.login, state: .state}'
```

Filter rules:
- Only process **top-level comments** (`in_reply_to_id: null`)
- Process comments from **all authors** — humans, `coderabbitai[bot]`, `sentry-io[bot]`, or any other user/bot
- Skip comments already resolved by a reply (e.g., "Risk accepted", "Won't fix")
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
| Already resolved by reply | Skip (note in summary) |
| Already fixed by prior push | Skip (note in summary) |
| Disagree / Not applicable | Flag to user for decision |
| Informational only (no action needed) | Skip (note in summary) |

If unsure, ask the user before proceeding.

---

### 5. Fix, Test, Commit (One Commit Per Comment)

Do NOT bundle multiple fixes.

For each valid comment:

#### 1. Implement Minimal Fix
- Modify only relevant files.
- Avoid unrelated refactors.

#### 2. Update Tests (If Needed)
- Add or adjust tests to cover change.

#### 3. Run Tests

Run the project's test suite for the affected module. Detect the test runner from the project (e.g., `pytest`, `jest`, `go test`, `mix test`, etc.) and run the relevant subset.

All tests must pass before committing.

#### 4. Commit With Clear Message

```bash
git add <files> && git commit -m "$(cat <<'EOF'
fix(<scope>): Short description of fix

Addresses review comment by <reviewer>.
Explanation of the change and why it addresses the feedback.

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

Commit rules:
- One commit per review comment
- Reference ticket ID in scope if available (e.g., `fix(HAN-1234):`)
- Name the reviewer in the commit body
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
gh api repos/{owner}/{repo}/pulls/{PR_NUMBER}/comments/{COMMENT_ID}/replies \
  -X POST -f body="Fixed in <short-sha> — <brief explanation>."
```

If already resolved or accepted:

```bash
gh api repos/{owner}/{repo}/pulls/{PR_NUMBER}/comments/{COMMENT_ID}/replies \
  -X POST -f body="Already resolved — <reason>."
```

---

### 8. Resolve Review Threads

Fetch threads:

```bash
gh api graphql -f query='query {
  repository(owner: "{owner}", name: "{repo}") {
    pullRequest(number: {PR_NUMBER}) {
      reviewThreads(first: 50) {
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

Resolve unresolved threads that have been addressed:

```bash
gh api graphql -f query='mutation {
  resolveReviewThread(input: {threadId: "{THREAD_ID}"}) {
    thread { isResolved }
  }
}'
```

Note:
Some bots (e.g., CodeRabbit) auto-resolve threads after detecting a fix.
Always check `isResolved` before manually resolving.

---

## Final Output

Produce a summary table:

| # | Reviewer | Comment | Commit | Status |
|---|----------|---------|--------|--------|
| 1 | coderabbitai[bot] | Description | `<sha>` | Resolved |
| 2 | @teammate | Description | N/A — accepted by reviewer | Skipped |
| 3 | sentry-io[bot] | Description | `<sha>` | Resolved |
| ... | ... | ... | ... | ... |

---

## Best Practices

- Never assume a comment is still valid.
- One commit per comment.
- Keep diffs minimal and reviewable.
- Do not silently ignore disagreements — escalate to user.
- Treat human reviewer comments with the same rigor as bot comments.
- Always produce a final structured summary.
