---
name: gh-pr-comments
description: Managing GitHub pull request review comments through extraction, validation, fixes, and resolution workflows using gh CLI.
---

# GitHub PR Comments

Automates GitHub pull request review comment management using gh CLI and GraphQL APIs.

## Quick Start

```bash
# Get current PR context
PR_NUMBER=$(gh pr view --json number --jq '.number')
REPO_OWNER=$(gh repo view --json owner --jq '.owner.login')
REPO_NAME=$(gh repo view --json name --jq '.name')

# Extract unresolved review comments
gh api graphql -f query='
  query($owner: String!, $repo: String!, $number: Int!) {
    repository(owner: $owner, name: $repo) {
      pullRequest(number: $number) {
        reviewThreads(first: 100) {
          nodes {
            id
            isResolved
            isOutdated
            comments(first: 10) {
              nodes {
                id
                body
                author { login }
                createdAt
                path
                line
              }
            }
          }
        }
      }
    }
  }' -F owner="$REPO_OWNER" -F repo="$REPO_NAME" -F number="$PR_NUMBER" --jq '.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved == false and .isOutdated == false)'

# Resolve specific thread
gh api graphql -f query='
  mutation($threadId: ID!) {
    resolveReviewThread(input: {threadId: $threadId}) {
      thread { id isResolved }
    }
  }' -F threadId="$THREAD_ID"
```

## Managing PR Review Comments

### Extracting Unresolved Comments

**Workflow:**
1. Get current PR number and repo context
2. Query GraphQL API for unresolved review threads
3. Filter out resolved and outdated threads
4. Extract comment details (file, line, author, content)

**Required Data:**
- Thread ID (for resolution)
- File path and line number
- Comment body and author
- Creation timestamp

### Validating Review Comments

**Validation Checklist:**
- [ ] Addresses a real issue or improvement
- [ ] Technically accurate and sound
- [ ] Follows project conventions
- [ ] Appropriate for change context
- [ ] Benefit vs. complexity justified

**Common Invalid Patterns:**
- Style suggestions conflicting with project standards
- Security warnings without practical exploit
- Micro-optimizations with negligible impact
- Dependency updates breaking compatibility
- Suggestions that would break existing functionality

*See `references/validation-patterns.md` for detailed validation guidance*

### Applying Fixes and Remediation

**Fix Workflow:**
1. Parse target file and location from comment
2. Apply suggested modification using appropriate strategy
3. Validate change doesn't break functionality
4. Test the fix when applicable
5. Verify no new issues introduced

**Change Types:**
- **Code Changes**: Modify source files with proper syntax
- **Configuration Updates**: Update YAML, JSON, or other config files
- **Documentation**: Update README, inline docs, or API documentation

*See `references/fix-strategies.md` for detailed implementation patterns*

### Marking Comments Resolved

**Resolution Workflow:**
1. Confirm fix addresses the original concern
2. Verify no regression in functionality
3. Use GraphQL mutation to resolve thread
4. Validate resolution succeeded

**Resolution Command:**
```bash
gh api graphql -f query='
  mutation($threadId: ID!) {
    resolveReviewThread(input: {threadId: $threadId}) {
      thread { id isResolved }
    }
  }' -F threadId="$THREAD_ID"
```

## Complete Workflow Process

1. **Extract Comments**: Get all unresolved review threads
2. **Analyze Each**: Validate against project standards and context
3. **Apply Fixes**: Implement valid suggestions systematically
4. **Resolve Threads**: Mark addressed comments as resolved
5. **Verify Changes**: Ensure no new issues introduced

## Error Handling

- **GraphQL API failures**: Check `gh auth status` and repository permissions
- **Resolution failures**: Verify write access to repository
- **Fix application failures**: Validate file paths and permissions
- **Missing data**: Use `--jq` filters for graceful error handling

## Reference Materials

- **`references/validation-patterns.md`**: Detailed validation frameworks and common patterns
- **`references/fix-strategies.md`**: Standardized approaches for implementing changes
- **`references/workflow-checklists.md`**: Step-by-step checklists for each capability

## Requirements

- GitHub CLI (`gh`) installed and authenticated
- Write access to target repository
- Bash shell environment

---

**Note**: This skill works with any GitHub repository using standard GitHub APIs.