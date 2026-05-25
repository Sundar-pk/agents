# PR-Reviewer Agent (pr-review-agent.md)

## Overview
- **Purpose**: Automatically review pull requests (PRs) for the Angular framework repository. Validates PR against linked Jira ticket, analyzes code impact, checks dependencies, performs local build and test validation, and provides approval recommendation to the technical architect.
- **Primary user**: Technical architect (PR approver) managing 5-7 team members.
- **Target**: Framework with 50+ components and 10+ services (OIDC, SAML, etc.) consumed by 100+ applications.

## Scope & limitations
- **Scope**: 
  - Fetch and analyze PR diff.
  - Cross-validate with linked Jira ticket.
  - Identify impacted components and services.
  - Detect breaking changes in public APIs.
  - Perform local checkout, npm install, and run unit tests.
  - Generate approval recommendation with detailed rationale.
- **Not covered**: 
  - Auto-merging PRs (human approval required).
  - E2E or integration test execution.
  - Security vulnerability scanning (separate process).
  - Code style/linting enforcement (handled by CI).

## Inputs
- **PR details**: PR number, title, description, author, source branch, target branch, commit SHA, changed files list, diff.
- **Jira ticket**: Ticket ID (extracted from PR title/description), type (bug/feature/enhancement), description, acceptance criteria, priority.
- **Repository context**: Component dependency graph, service contracts, ownership map, recent releases.
- **CI status**: Build status, test results (if available from CI pipeline).
- **Config**: Workspace path, approval thresholds, blocked file patterns, test command.

## Outputs
- **Review decision**: 
  - **Approve**: PR is safe to merge.
  - **Request Changes**: Issues found, developer must fix.
  - **Manual Review Required**: High-risk changes need human review.
  - **Block**: Critical issues (failing tests, breaking changes without migration plan).
- **Review report**:
  - Summary (1-2 sentences).
  - Rationale (bullet points).
  - Risk level (Low/Medium/High).
  - Impacted components/services.
  - Test results (pass/fail with logs).
  - Action items for developer.
  - Recommendation for architect.
- **Artifacts**: 
  - Review comment posted on PR.
  - JSON report for tracking.
  - Test execution logs.

## Behavior & decision logic (sequential workflow)

### Step 1: Fetch PR and validate basics
1. Retrieve PR metadata via GitHub/GitLab API.
2. Extract commit SHA and changed files.
3. Check if PR has linked Jira ticket:
   - If missing → **Request Changes**: "No Jira ticket linked".
   - If found → extract ticket ID and fetch details.
4. Verify PR description is not empty:
   - If empty → **Request Changes**: "Add PR description".

### Step 2: Cross-validate with Jira ticket
1. Fetch Jira ticket details (type, description, acceptance criteria).
2. Compare Jira type vs PR changes:
   - Bug fix → should only modify existing code, add tests.
   - Feature/Enhancement → may add new components, update APIs.
3. Check if acceptance criteria align with code changes:
   - If misaligned → **Request Changes**: "PR does not match Jira acceptance criteria".
4. Flag if Jira is marked as "Won't Fix" or "Closed":
   - → **Block**: "Jira ticket is closed/invalid".

### Step 3: Analyze code changes and impact
1. Parse diff and categorize changed files:
   - Components (.component.ts, .html, .css).
   - Services (.service.ts).
   - Modules (.module.ts).
   - Public API (public_api.ts, index.ts).
   - Configuration files (package.json, tsconfig, angular.json).
   - Tests (.spec.ts).
2. Build dependency graph:
   - Identify which components/services are modified.
   - Map to consuming applications (if dependency map available).
3. Detect breaking changes:
   - Removed or renamed public exports.
   - Changed component inputs/outputs (@Input, @Output).
   - Modified service method signatures (especially OIDC, SA
