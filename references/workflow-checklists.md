# GitHub PR Comments Workflow Checklists

Step-by-step checklists for the 4 core GitHub PR Comments capabilities.

## 1. Extracting Unresolved Comments Workflow

**Prerequisites:**
- [ ] GitHub CLI (`gh`) installed and authenticated
- [ ] Write access to target repository
- [ ] Current working directory in repository

**Extraction Steps:**
1. **Get PR Context**
   ```bash
   PR_NUMBER=$(gh pr view --json number --jq '.number')
   REPO_OWNER=$(gh repo view --json owner --jq '.owner.login')
   REPO_NAME=$(gh repo view --json name --jq '.name')
   ```

2. **Query Unresolved Threads**
   ```bash
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
   ```

3. **Validate Extraction Results**
   - [ ] Got JSON response with thread data
   - [ ] Each thread has required fields (id, comments, file path, line)
   - [ ] No GraphQL errors in output
   - [ ] Filter correctly excludes resolved and outdated threads

4. **Store Results**
   - [ ] Save thread IDs for later resolution
   - [ ] Extract file paths and line numbers for fix targeting
   - [ ] Capture comment content and author for validation
   - [ ] Note creation timestamps for priority assessment

## 2. Validating Review Comments Workflow

**Pre-Validation Setup:**
- [ ] Have extracted comment data from previous step
- [ ] Understand project coding standards and conventions
- [ ] Access to validation patterns reference (`validation-patterns.md`)

**Validation Process:**
For each unresolved comment:

1. **Initial Assessment**
   - [ ] Comment addresses a real issue or improvement opportunity
   - [ ] Proposed solution is technically correct and sound
   - [ ] Follows established project conventions and patterns
   - [ ] Appropriate for the specific change context
   - [ ] Benefits justify implementation cost and complexity

2. **Comment Type Validation**
   - [ ] **Automated Tool Comment**: Check tool-specific valid/invalid patterns
   - [ ] **Human Reviewer**: Assess feedback quality and specificity
   - [ ] **Security Issue**: Verify exploit potential and impact
   - [ ] **Performance Suggestion**: Evaluate measurable benefit
   - [ ] **Style Suggestion**: Check alignment with project standards

3. **Context-Specific Checks**
   - [ ] **Code Changes**: Validate logic, syntax, and integration
   - [ ] **Configuration Updates**: Check environment compatibility
   - [ ] **Documentation**: Verify accuracy and clarity
   - [ ] **Dependencies**: Assess compatibility and maintenance impact

4. **Decision & Documentation**
   - [ ] **Valid**: Mark for implementation with fix strategy
   - [ ] **Invalid**: Document reasoning and prepare challenge response
   - [ ] **Partial**: Note modifications needed for acceptance
   - [ ] **Unclear**: Request clarification from reviewer

**Validation Output:**
- [ ] List of validated comments with implementation notes
- [ ] List of challenged comments with reasoning
- [ ] Priority order for implementation
- [ ] Required fix strategies for each valid comment

## 3. Applying Fixes and Remediation Workflow

**Pre-Fix Setup:**
- [ ] Have validated comment list with priorities
- [ ] Access to fix strategies reference (`fix-strategies.md`)
- [ ] Development environment properly configured
- [ ] Current branch checked out and up to date

**Fix Implementation:**
For each validated comment:

1. **Preparation**
   - [ ] Parse target file and line number from comment
   - [ ] Select appropriate fix strategy based on issue type
   - [ ] Identify language-specific patterns needed
   - [ ] Create backup or branch if change is complex

2. **Apply Fix**
   - [ ] **Single Line**: Use sed or direct file editing
   - [ ] **Multi-line**: Use heredoc or direct block replacement
   - [ ] **Language-Specific**: Apply language-appropriate patterns
   - [ ] **Configuration**: Update YAML/JSON with proper syntax
   - [ ] **Documentation**: Update content while preserving format

3. **Validation**
   - [ ] **Syntax Check**: Code compiles/runs without errors
   - [ ] **Linter Check**: No new linting violations introduced
   - [ ] **Functionality Test**: Tests pass if applicable
   - [ ] **Manual Verification**: Fix addresses original concern
   - [ ] **Regression Check**: No new issues introduced

4. **Documentation**
   - [ ] Update relevant documentation if behavior changed
   - [ ] Add comments explaining complex changes if needed
   - [ ] Note dependencies or side effects introduced

**Post-Fix Process:**
- [ ] Stage changes for commit
- [ ] Create descriptive commit message referencing comment
- [ ] Run full test suite if available
- [ ] Prepare for thread resolution

## 4. Marking Comments Resolved Workflow

**Pre-Resolution Setup:**
- [ ] All valid fixes have been implemented and tested
- [ ] Have thread IDs for resolved comments
- [ ] GitHub CLI authentication is current and valid
- [ ] Write permissions verified on repository

**Resolution Process:**
For each addressed comment thread:

1. **Verification**
   - [ ] Fix successfully implemented and tested
   - [ ] No regressions or new issues introduced
   - [ ] Thread content is fully addressed by changes
   - [ ] Ready for reviewer confirmation

2. **Resolution Command**
   ```bash
   gh api graphql -f query='
     mutation($threadId: ID!) {
       resolveReviewThread(input: {threadId: $threadId}) {
         thread {
           id
           isResolved
         }
       }
     }' -F threadId="$THREAD_ID"
   ```

3. **Validation**
   - [ ] GraphQL mutation returns success
   - [ ] Thread `isResolved` field shows `true`
   - [ ] No error messages in command output
   - [ ] GitHub UI shows thread as resolved

4. **Documentation**
   - [ ] Log resolved thread ID for audit trail
   - [ ] Note any issues or partial resolutions
   - [ ] Update tracking spreadsheet if used

**Error Handling:**
- [ ] **GraphQL Error**: Check `gh auth status` and permissions
- [ ] **Thread Not Found**: Verify correct thread ID format
- [ ] **Permission Denied**: Confirm write access to repository
- [ ] **Network Issues**: Retry after checking connectivity

## Complete Workflow Summary

**Sequence of Operations:**
1. **Extract** all unresolved PR review comments using GraphQL API
2. **Validate** each comment against project standards and technical merit
3. **Apply** fixes using appropriate strategies for each valid comment
4. **Resolve** threads when fixes successfully address the concerns
5. **Verify** all changes work together and no new issues introduced

**Quality Gates:**
- [ ] All comments processed (valid implemented, invalid challenged)
- [ ] All fixes tested and validated
- [ ] All resolved threads show as resolved in GitHub
- [ ] No regressions or new issues introduced
- [ ] Documentation updated as needed

**Success Metrics:**
- Number of unresolved comments reduced to zero
- All implemented fixes pass validation checks
- Review threads properly resolved in GitHub UI
- No new issues introduced by implemented fixes