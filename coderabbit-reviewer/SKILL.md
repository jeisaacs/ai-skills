# Resolve CodeRabbit Reviews

Process CodeRabbit review comments on the current branch's PR. For each comment: assess, fix if needed, commit, push, reply with the commit ref, and resolve the thread.

## Steps

### 1. Find the PR

```bash
gh pr list --head <current-branch> --json number,title,url
```

### 2. Fetch all review comments

```bash
gh api repos/Handshaik/handshaik-app/pulls/<PR_NUMBER>/comments \
  --jq '.[] | {id: .id, path: .path, line: .line, body: .body, user: .user.login, in_reply_to_id: .in_reply_to_id}'
```

- Filter to top-level comments only (`in_reply_to_id: null`) from `coderabbitai[bot]`.
- Also check for human replies (e.g. "The risk is accepted") that already resolve a comment — skip those.

### 3. Read the current state of each file mentioned

Before making any fix, always `Read` the file at the lines referenced by the comment. Understand the current code — it may have been modified by the user or a linter since the comment was posted.

### 4. Triage each comment

For each CodeRabbit comment, classify it:

| Classification | Action |
|---|---|
| Valid fix needed | Implement the fix |
| Already resolved by human reply | Skip — note it in the summary |
| Already fixed by a prior push | Skip — note it in the summary |
| Disagree / not applicable | Flag to the user for a decision |

### 5. Fix, test, commit — one per comment

For each comment that needs a fix:

1. **Make the code change** — edit the source file(s).
2. **Update tests if needed** — add/modify tests to cover the change.
3. **Run tests** — `uv run pytest src/apps/<module>/tests/ -v --tb=short` and confirm all pass.
4. **Commit with a descriptive message** referencing the ticket:
   ```bash
   git add <files> && git commit -m "$(cat <<'EOF'
   fix(HAN-XXXX): Short description of what was fixed

   Explanation of the change and why it addresses the review comment.

   Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
   EOF
   )"
   ```

Do NOT bundle multiple comment fixes into one commit. One commit per comment.

### 6. Push all commits

```bash
git push
```

### 7. Reply to each comment with the commit reference

```bash
gh api repos/Handshaik/handshaik-app/pulls/<PR_NUMBER>/comments/<COMMENT_ID>/replies \
  -X POST -f body="Fixed in <short-sha> — <brief explanation of the fix>."
```

For comments that were already resolved or accepted:
```bash
gh api repos/Handshaik/handshaik-app/pulls/<PR_NUMBER>/comments/<COMMENT_ID>/replies \
  -X POST -f body="Already resolved — <reason>."
```

### 8. Resolve comment threads

Fetch thread IDs and resolve any that aren't auto-resolved:

```bash
# Get thread IDs
gh api graphql -f query='query {
  repository(owner: "Handshaik", name: "handshaik-app") {
    pullRequest(number: <PR_NUMBER>) {
      reviewThreads(first: 20) {
        nodes { id isResolved comments(first: 1) { nodes { databaseId } } }
      }
    }
  }
}'

# Resolve unresolved threads
gh api graphql -f query='mutation { resolveReviewThread(input: {threadId: "<THREAD_ID>"}) { thread { isResolved } } }'
```

Note: CodeRabbit often auto-resolves threads when it detects a new push that addresses its comment. Check `isResolved` before manually resolving.

### 9. Report summary

Present a table to the user:

| # | Comment | Commit | Status |
|---|---------|--------|--------|
| 1 | Description | `<sha>` | Resolved |
| 2 | Description | N/A — accepted by reviewer | Resolved |
| ... | ... | ... | ... |
