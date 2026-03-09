name	description	allowed-tools
create-pr
Creates GitHub pull requests with properly formatted titles that pass the check-pr-title CI validation. Use when creating PRs, submitting changes for review, or when the user says /pr or asks to create a pull request.
Bash(git:*), Bash(gh:*), Read, Grep, Glob
Create Pull Request
Creates GitHub PRs with titles that pass n8n's check-pr-title CI validation.

PR Title Format
<type>(<scope>): <summary>
Types (required)
Type	Description	Changelog
feat	New feature	Yes
fix	Bug fix	Yes
perf	Performance improvement	Yes
test	Adding/correcting tests	No
docs	Documentation only	No
refactor	Code change (no bug fix or feature)	No
build	Build system or dependencies	No
ci	CI configuration	No
chore	Routine tasks, maintenance	No
Scopes (optional but recommended)
API - Public API changes
benchmark - Benchmark CLI changes
core - Core/backend/private API
editor - Editor UI changes
* Node - Specific node (e.g., Slack Node, GitHub Node)
Summary Rules
Use imperative present tense: "Add" not "Added"
Capitalize first letter
No period at the end
No ticket IDs (e.g., N8N-1234)
Add (no-changelog) suffix to exclude from changelog
Steps
Check current state:

git status
git diff --stat
git log origin/master..HEAD --oneline
Analyze changes to determine:

Type: What kind of change is this?
Scope: Which package/area is affected?
Summary: What does the change do?
Push branch if needed:

git push -u origin HEAD
Create PR using gh CLI with the template from .github/pull_request_template.md:

gh pr create --draft --title "<type>(<scope>): <summary>" --body "$(cat <<'EOF'
## Summary

<Describe what the PR does and how to test. Photos and videos are recommended.>

## Related Linear tickets, Github issues, and Community forum posts

<!-- Link to Linear ticket: https://linear.app/n8n/issue/[TICKET-ID] -->
<!-- Use "closes #<issue-number>", "fixes #<issue-number>", or "resolves #<issue-number>" to automatically close issues -->

## Review / Merge checklist

- [ ] PR title and summary are descriptive. ([conventions](../blob/master/.github/pull_request_title_conventions.md))
- [ ] [Docs updated](https://github.com/n8n-io/n8n-docs) or follow-up ticket created.
- [ ] Tests included.
- [ ] PR Labeled with `release/backport` (if the PR is an urgent fix that needs to be backported)
EOF
)"
PR Body Guidelines
Based on .github/pull_request_template.md:

Summary Section
Describe what the PR does
Explain how to test the changes
Include screenshots/videos for UI changes
Related Links Section
Link to Linear ticket: https://linear.app/n8n/issue/[TICKET-ID]
Link to GitHub issues using keywords to auto-close:
closes #123 / fixes #123 / resolves #123
Link to Community forum posts if applicable
Checklist
All items should be addressed before merging:

PR title follows conventions
Docs updated or follow-up ticket created
Tests included (bugs need regression tests, features need coverage)
release/backport label added if urgent fix needs backporting
Examples
Feature in editor
feat(editor): Add workflow performance metrics display
Bug fix in core
fix(core): Resolve memory leak in execution engine
Node-specific change
fix(Slack Node): Handle rate limiting in message send
Breaking change (add exclamation mark before colon)
feat(API)!: Remove deprecated v1 endpoints
No changelog entry
refactor(core): Simplify error handling (no-changelog)
No scope (affects multiple areas)
chore: Update dependencies to latest versions
Validation
The PR title must match this pattern:

^(feat|fix|perf|test|docs|refactor|build|ci|chore|revert)(\([a-zA-Z0-9 ]+( Node)?\))?!?: [A-Z].+[^.]$
Key validation rules:

Type must be one of the allowed types
Scope is optional but must be in parentheses if present
Exclamation mark for breaking changes goes before the colon
Summary must start with capital letter
Summary must not end with a periodname	description	disable-model-invocation	argument-hint	compatibility
linear-issue
Fetch and analyze Linear issue with all related context. Use when starting work on a Linear ticket, analyzing issues, or gathering context about a Linear issue.
true
[issue-id]
requires	optional
mcp	description
linear
Core dependency — used to fetch issue details, relations, and comments
cli	description
gh
GitHub CLI — used to fetch linked PRs and issues. Must be authenticated (gh auth login)
mcp	description
notion
Used to fetch linked Notion documents. Skip Notion steps if unavailable.
skill	description
loom-transcript
Used to fetch Loom video transcripts. Skip Loom steps if unavailable.
cli	description
curl
Used to download images/attachments. Typically pre-installed.
Linear Issue Analysis
Start work on Linear issue $ARGUMENTS

Prerequisites
This skill depends on external tools. Before proceeding, verify availability:

Required:

Linear MCP (mcp__linear): Must be connected. Without it the skill cannot function at all.
GitHub CLI (gh): Must be installed and authenticated. Run gh auth status to verify. Used to fetch linked PRs and issues.
Optional (graceful degradation):

Notion MCP (mcp__notion): Needed only if the issue links to Notion docs. If unavailable, note the Notion links in the summary and tell the user to check them manually.
Loom transcript skill (/loom-transcript): Needed only if the issue contains Loom videos. If unavailable, note the Loom links in the summary for the user to watch.
curl: Used to download images. Almost always available; if missing, skip image downloads and note it.
If a required tool is missing, stop and tell the user what needs to be set up before continuing.

Instructions
Follow these steps to gather comprehensive context about the issue:

1. Fetch the Issue and Comments from Linear
Use the Linear MCP tools to fetch the issue details and comments together:

Use mcp__linear__get_issue with the issue ID to get full details including attachments
Include relations to see blocking/related/duplicate issues
Immediately after, use mcp__linear__list_comments with the issue ID to fetch all comments
Both calls should be made together in the same step to gather the complete context upfront.

2. Analyze Attachments and Media (MANDATORY)
IMPORTANT: This step is NOT optional. You MUST scan and fetch all visual content from BOTH the issue description AND all comments.

Screenshots/Images (ALWAYS fetch):

Scan the issue description AND all comments for ALL image URLs:
<img> tags
Markdown images ![](url)
Raw URLs (github.com/user-attachments, imgur.com, etc.)
For EACH image found (in description or comments):
Download using curl -sL "url" -o /path/to/image.png (GitHub URLs require following redirects) OR the linear mcp
Use the Read tool on the downloaded file to view it
Describe what you see in detail
Do NOT skip images - they often contain critical context like error messages, UI states, or configuration
Loom Videos (ALWAYS fetch transcript):

Scan the issue description AND all comments for Loom URLs (loom.com/share/...)
For EACH Loom video found (in description or comments):
Use the /loom-transcript skill to fetch the FULL transcript
Summarize key points, timestamps, and any demonstrated issues
Loom videos often contain crucial reproduction steps and context that text alone cannot convey
3. Fetch Related Context
Related Linear Issues:

Use mcp__linear__get_issue for any issues mentioned in relations (blocking, blocked by, related, duplicates)
Summarize how they relate to the main issue
GitHub PRs and Issues:

If GitHub links are mentioned, use gh CLI to fetch PR/issue details:
gh pr view <number> for pull requests
gh issue view <number> for issues
Download images attached to issues: curl -H "Authorization: token $(gh auth token)" -L <image-url> -o image.png
Notion Documents:

If Notion links are present, use mcp__notion__notion-fetch with the Notion URL or page ID to retrieve document content
Summarize relevant documentation
4. Review Comments
Comments were already fetched in Step 1. Review them for:

Additional context and discussion history
Any attachments or media linked in comments (process in Step 2)
Clarifications or updates to the original issue description
5. Identify Affected Node (if applicable)
Determine whether this issue is specific to a particular n8n node (e.g. a trigger, action, or tool node). Look for clues in:

The issue title (e.g. "Linear trigger", "Slack node", "HTTP Request")
The issue description and comments mentioning node names
Labels or tags on the issue (e.g. node:linear, node:slack)
Screenshots showing a specific node's configuration or error
If the issue is node-specific:

Find the node type ID. Use Grep to search for the node's display name (or keywords from it) in packages/frontend/editor-ui/data/node-popularity.json to find the exact node type ID. For reference, common ID patterns are:

Core nodes: n8n-nodes-base.<camelCaseName> (e.g. "HTTP Request" → n8n-nodes-base.httpRequest)
Trigger variants: n8n-nodes-base.<name>Trigger (e.g. "Gmail Trigger" → n8n-nodes-base.gmailTrigger)
Tool variants: n8n-nodes-base.<name>Tool (e.g. "Google Sheets Tool" → n8n-nodes-base.googleSheetsTool)
LangChain/AI nodes: @n8n/n8n-nodes-langchain.<camelCaseName> (e.g. "OpenAI Chat Model" → @n8n/n8n-nodes-langchain.lmChatOpenAi)
Look up the node's popularity score from packages/frontend/editor-ui/data/node-popularity.json. Use Grep to search for the node ID in that file. The popularity score is a log-scale value between 0 and 1. Use these thresholds to classify:

Score	Level	Description	Examples
≥ 0.8	High	Core/widely-used nodes, top ~5%	HTTP Request (0.98), Google Sheets (0.95), Postgres (0.83), Gmail Trigger (0.80)
0.4–0.8	Medium	Regularly used integrations	Slack (0.78), GitHub (0.64), Jira (0.65), MongoDB (0.63)
< 0.4	Low	Niche or rarely used nodes	Amqp (0.34), Wise (0.36), CraftMyPdf (0.33)
Include the raw score and the level (high/medium/low) in the summary.

If the node is not found in the popularity file, note that it may be a community node or a very new/niche node.

6. Assess Effort/Complexity
After gathering all context, assess the effort required to fix/implement the issue. Use the following T-shirt sizes:

Size	Approximate effort
XS	≤ 1 hour
S	≤ 1 day
M	2-3 days
L	3-5 days
XL	≥ 6 days
To make this assessment, consider:

Scope of changes: How many files/packages need to be modified? Is it a single node fix or a cross-cutting change?
Complexity: Is it a straightforward parameter change, a new API integration, a new credential type, or an architectural change?
Testing: How much test coverage is needed? Are E2E tests required?
Risk: Could this break existing functionality? Does it need backward compatibility?
Dependencies: Are there external API changes, new packages, or cross-team coordination needed?
Documentation: Does this require docs updates, migration guides, or changelog entries?
Provide the T-shirt size along with a brief justification explaining the key factors that drove the estimate.

7. Present Summary
Before presenting, verify you have completed:

 Downloaded and viewed ALL images in the description AND comments
 Fetched transcripts for ALL Loom videos in the description AND comments
 Fetched ALL linked GitHub issues/PRs via gh CLI
 Listed all comments on the issue
 Checked whether the issue is node-specific and looked up popularity if so
 Assessed effort/complexity with T-shirt size
After gathering all context, present a comprehensive summary including:

Issue Overview: Title, status, priority, assignee, labels
Description: Full issue description with any clarifications from comments
Visual Context: Summary of screenshots/videos (what you observed in each)
Affected Node (if applicable): Node name, node type ID (n8n-nodes-base.xxx), popularity score with level (e.g. 0.64 — medium popularity)
Related Issues: How this connects to other work
Technical Context: Any PRs, code references, or documentation
Effort Estimate: T-shirt size (XS/S/M/L/XL) with justification
Next Steps: Suggested approach based on all gathered context
Notes
The issue ID can be provided in formats like: AI-1975, node-1975, or just 1975 (will search)
If no issue ID is provided, ask the user for onename	description	argument-hint
loom-transcript
Fetch and display the full transcript from a Loom video URL. Use when the user wants to get or read a Loom transcript.
loom-url
Loom Transcript Fetcher
Fetch the transcript from a Loom video using Loom's GraphQL API.

Instructions
Given the Loom URL: $ARGUMENTS

1. Extract the Video ID
Parse the Loom URL to extract the 32-character hex video ID. Supported URL formats:

https://www.loom.com/share/<video-id>
https://www.loom.com/embed/<video-id>
https://www.loom.com/share/<video-id>?sid=<session-id>
The video ID is the 32-character hex string after /share/ or /embed/.

2. Fetch Video Metadata
Use the WebFetch tool to POST to https://www.loom.com/graphql to get the video title and details.

Use this curl command via Bash:

curl -s 'https://www.loom.com/graphql' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'x-loom-request-source: loom_web_45a5bd4' \
  -H 'apollographql-client-name: web' \
  -H 'apollographql-client-version: 45a5bd4' \
  -d '{
    "operationName": "GetVideoSSR",
    "variables": {"id": "<VIDEO_ID>", "password": null},
    "query": "query GetVideoSSR($id: ID!, $password: String) { getVideo(id: $id, password: $password) { ... on RegularUserVideo { id name description createdAt owner { display_name } } } }"
  }'
3. Fetch the Transcript URLs
Use curl via Bash to call the GraphQL API:

curl -s 'https://www.loom.com/graphql' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'x-loom-request-source: loom_web_45a5bd4' \
  -H 'apollographql-client-name: web' \
  -H 'apollographql-client-version: 45a5bd4' \
  -d '{
    "operationName": "FetchVideoTranscript",
    "variables": {"videoId": "<VIDEO_ID>", "password": null},
    "query": "query FetchVideoTranscript($videoId: ID!, $password: String) { fetchVideoTranscript(videoId: $videoId, password: $password) { ... on VideoTranscriptDetails { id video_id source_url captions_source_url } ... on GenericError { message } } }"
  }'
Replace <VIDEO_ID> with the actual video ID extracted in step 1.

The response contains:

source_url — JSON transcript URL
captions_source_url — VTT (WebVTT) captions URL
4. Download and Parse the Transcript
Fetch both URLs returned from step 3 (if available):

VTT captions (captions_source_url): Download with curl -sL "<url>". This is a WebVTT file with timestamps and text.
JSON transcript (source_url): Download with curl -sL "<url>". This is a JSON file with transcript segments.
Prefer the VTT captions as the primary source since they include proper timestamps. Fall back to the JSON transcript if VTT is unavailable.

5. Present the Transcript
Format and present the full transcript to the user:

Video: [Title from metadata] Author: [Owner name] Date: [Created date]

0:00 - First transcript segment text...

0:14 - Second transcript segment text...

(continue for all segments)

Error Handling
If the GraphQL response contains a GenericError, report the error message to the user.
If both source_url and captions_source_url are null/missing, tell the user that no transcript is available for this video.
If the video URL is invalid or the ID cannot be extracted, ask the user for a valid Loom URL.
Notes
No authentication or cookies are required — Loom's transcript API is publicly accessible.
Only English transcripts are available through this API.
Transcripts are auto-generated and may contain minor errors.name	description	argument-hint
loom-transcript
Fetch and display the full transcript from a Loom video URL. Use when the user wants to get or read a Loom transcript.
loom-url
Loom Transcript Fetcher
Fetch the transcript from a Loom video using Loom's GraphQL API.

Instructions
Given the Loom URL: $ARGUMENTS

1. Extract the Video ID
Parse the Loom URL to extract the 32-character hex video ID. Supported URL formats:

https://www.loom.com/share/<video-id>
https://www.loom.com/embed/<video-id>
https://www.loom.com/share/<video-id>?sid=<session-id>
The video ID is the 32-character hex string after /share/ or /embed/.

2. Fetch Video Metadata
Use the WebFetch tool to POST to https://www.loom.com/graphql to get the video title and details.

Use this curl command via Bash:

curl -s 'https://www.loom.com/graphql' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'x-loom-request-source: loom_web_45a5bd4' \
  -H 'apollographql-client-name: web' \
  -H 'apollographql-client-version: 45a5bd4' \
  -d '{
    "operationName": "GetVideoSSR",
    "variables": {"id": "<VIDEO_ID>", "password": null},
    "query": "query GetVideoSSR($id: ID!, $password: String) { getVideo(id: $id, password: $password) { ... on RegularUserVideo { id name description createdAt owner { display_name } } } }"
  }'
3. Fetch the Transcript URLs
Use curl via Bash to call the GraphQL API:

curl -s 'https://www.loom.com/graphql' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'x-loom-request-source: loom_web_45a5bd4' \
  -H 'apollographql-client-name: web' \
  -H 'apollographql-client-version: 45a5bd4' \
  -d '{
    "operationName": "FetchVideoTranscript",
    "variables": {"videoId": "<VIDEO_ID>", "password": null},
    "query": "query FetchVideoTranscript($videoId: ID!, $password: String) { fetchVideoTranscript(videoId: $videoId, password: $password) { ... on VideoTranscriptDetails { id video_id source_url captions_source_url } ... on GenericError { message } } }"
  }'
Replace <VIDEO_ID> with the actual video ID extracted in step 1.

The response contains:

source_url — JSON transcript URL
captions_source_url — VTT (WebVTT) captions URL
4. Download and Parse the Transcript
Fetch both URLs returned from step 3 (if available):

VTT captions (captions_source_url): Download with curl -sL "<url>". This is a WebVTT file with timestamps and text.
JSON transcript (source_url): Download with curl -sL "<url>". This is a JSON file with transcript segments.
Prefer the VTT captions as the primary source since they include proper timestamps. Fall back to the JSON transcript if VTT is unavailable.

5. Present the Transcript
Format and present the full transcript to the user:

Video: [Title from metadata] Author: [Owner name] Date: [Created date]

0:00 - First transcript segment text...

0:14 - Second transcript segment text...

(continue for all segments)

Error Handling
If the GraphQL response contains a GenericError, report the error message to the user.
If both source_url and captions_source_url are null/missing, tell the user that no transcript is available for this video.
If the video URL is invalid or the ID cannot be extracted, ask the user for a valid Loom URL.
Notes
No authentication or cookies are required — Loom's transcript API is publicly accessible.
Only English transcripts are available through this API.
Transcripts are auto-generated and may contain minor errors.name	description
n8n-conventions
Quick reference for n8n patterns. Full docs /AGENTS.md
n8n Quick Reference
📚 Full Documentation:

General: /AGENTS.md - Architecture, commands, workflows
Frontend: /packages/frontend/AGENTS.md - CSS variables, timing
Use this skill when you need quick reminders on critical patterns.

Critical Rules (Must Follow)
TypeScript:

Never any → use unknown
Prefer satisfies over as (except tests)
Shared types in @n8n/api-types
Error Handling:

import { UnexpectedError } from 'n8n-workflow';
throw new UnexpectedError('message', { extra: { context } });
// DON'T use deprecated ApplicationError
Frontend:

Vue 3 Composition API (<script setup lang="ts">)
CSS variables (never hardcode px) - see /packages/frontend/AGENTS.md
All text via i18n ($t('key'))
data-testid for E2E (single value, no spaces)
Backend:

Controller → Service → Repository
Dependency injection via @n8n/di
Config via @n8n/config
Zod schemas for validation
Testing:

Vitest (unit), Playwright (E2E)
Mock external dependencies
Work from package directory: pushd packages/cli && pnpm test
Database:

SQLite/PostgreSQL only (app DB)
Exception: DB nodes (MySQL Node, etc.) can use DB-specific features
Commands:

pnpm build > build.log 2>&1  # Always redirect
pnpm typecheck               # Before commit
pnpm lint                    # Before commit
Key Packages
Package	Purpose
packages/cli	Backend API
packages/frontend/editor-ui	Vue 3 frontend
packages/@n8n/api-types	Shared types
packages/@n8n/db	TypeORM entities
packages/workflow	Core interfaces
Common Patterns
Pinia Store:

import { STORES } from '@n8n/stores';
export const useMyStore = defineStore(STORES.MY_STORE, () => {
  const state = shallowRef([]);
  return { state };
});
Vue Component:

<script setup lang="ts">
type Props = { title: string };
const props = defineProps<Props>();
</script>
Service:

import { Service } from '@n8n/di';
import { Config } from '@n8n/config';

@Service()
export class MyService {
  constructor(private readonly config: Config) {}
}
📖 Need more details? Read /AGENTS.md and /packages/frontend/AGENTS.mdname	description	user_invocable
reproduce-bug
Reproduce a bug from a Linear ticket with a failing test. Expects the full ticket context (title, description, comments) to be provided as input.
true
Bug Reproduction Framework
Given a Linear ticket context ($ARGUMENTS), systematically reproduce the bug with a failing regression test.

Step 1: Parse Signals
Extract the following from the provided ticket context:

Error message / stack trace (if provided)
Reproduction steps (if provided)
Workflow JSON (if attached)
Affected area (node, execution engine, editor, API, config, etc.)
Version where it broke / last working version
Step 2: Route to Test Strategy
Based on the affected area, pick the test layer and pattern:

Area	Test Layer	Pattern	Key Location
Node operation	Jest unit	NodeTestHarness + nock	packages/nodes-base/nodes/*/test/
Node credential	Jest unit	jest-mock-extended	packages/nodes-base/nodes/*/test/
Trigger webhook	Jest unit	mock IHookFunctions + jest.mock GenericFunctions	packages/nodes-base/nodes/*/test/
Binary data	Jest unit	NodeTestHarness assertBinaryData	packages/core/nodes-testing/
Execution engine	Jest integration	WorkflowRunner + DI container	packages/cli/src/__tests__/
CLI / API	Jest integration	setupTestServer + supertest	packages/cli/test/integration/
Config	Jest unit	GlobalConfig + Container	packages/@n8n/config/src/__tests__/
Editor UI	Vitest	Vue Test Utils + Pinia	packages/frontend/editor-ui/src/**/__tests__/
E2E / Canvas	Playwright	Test containers + composables	packages/testing/playwright/
Step 3: Locate Source Files
Find the source code for the affected area:

Search for the node/service/component mentioned in the ticket
Find the GenericFunctions file (common bug location for nodes)
Check for existing test files in the same area
Look at recent git history on affected files (git log --oneline -10 -- <path>)
Step 4: Trace the Code Path
Read the source code and trace the execution path that triggers the bug:

Follow the call chain from entry point to the failure
Identify the specific line(s) where the bug manifests
Note any error handling (or lack thereof) around the bug
Step 5: Form Hypothesis
State a clear, testable hypothesis:

"When [input/condition], the code does [wrong thing] because [root cause]"
Identify the exact line(s) that need to change
Predict what the test output will show
Step 6: Find Test Patterns
Look for existing tests in the same area:

Check test/ directories near the affected code
Identify which mock/setup patterns they use
Use the same patterns for consistency
If no tests exist, find the closest similar node/service tests as a template
Step 7: Write Failing Test
Write a regression test that:

Uses the patterns found in Step 6
Targets the specific hypothesis from Step 5
Includes a comment referencing the ticket ID
Asserts the CORRECT behavior (test will fail on current code)
Also includes a "happy path" test to prove the setup works
Step 8: Run and Score
Run the test from the package directory (e.g., cd packages/nodes-base && pnpm test <file>).

Classify the result:

Confidence	Criteria	Output
CONFIRMED	Test fails consistently, failure matches hypothesis	Reproduction Report
LIKELY	Test fails but failure mode differs slightly	Report + caveat
UNCONFIRMED	Cannot trigger the failure	Report: what was tried
SKIPPED	Hit a hard bailout trigger	Report: why skipped
ALREADY_FIXED	Bug no longer reproduces on current code	Report: when fixed
Step 9: Iterate or Bail
If UNCONFIRMED after first attempt:

Revisit hypothesis — re-read the code path
Try a different test approach or layer
Maximum 3 attempts before declaring UNCONFIRMED
Hard bailout triggers (stop immediately):

Requires real third-party API credentials
Race condition / timing-dependent
Requires specific cloud/enterprise infrastructure
Requires manual UI interaction that can't be scripted
Output: Reproduction Report
Present findings in this format:

Ticket: [ID] — [title] Confidence: [CONFIRMED | LIKELY | UNCONFIRMED | SKIPPED | ALREADY_FIXED]

Root Cause
[1-2 sentences explaining the bug mechanism]

Location
File	Lines	Issue
path/to/file.ts	XX-YY	Description of the problem
Failing Test
path/to/test/file.test.ts — X/Y tests fail:

test name — [failure description]
Fix Hint
[Pseudocode or description of the fix approach]

Important
DO NOT fix the bug — only reproduce it with a failing test
Leave test files in place as evidence (don't commit unless asked)
Run tests from the package directory (e.g., pushd packages/nodes-base && pnpm test <file> && popd)
Always redirect build output: pnpm build > build.log 2>&1
DO NOT look at existing fix PRs — the goal is to reproduce from signals aloneARG NODE_VERSION=24.13.1
ARG N8N_VERSION=snapshot

FROM n8nio/base:${NODE_VERSION}

ARG N8N_VERSION
ARG N8N_RELEASE_TYPE=dev
ENV NODE_ENV=production
ENV N8N_RELEASE_TYPE=${N8N_RELEASE_TYPE}
ENV SHELL=/bin/sh

WORKDIR /home/node

COPY ./compiled /usr/local/lib/node_modules/n8n
COPY docker/images/n8n/docker-entrypoint.sh /

RUN cd /usr/local/lib/node_modules/n8n && \
    npm rebuild sqlite3 && \
    ln -s /usr/local/lib/node_modules/n8n/bin/n8n /usr/local/bin/n8n && \
    mkdir -p /home/node/.n8n && \
    chown -R node:node /home/node && \
    rm -rf /root/.npm /tmp/*

EXPOSE 5678/tcp
USER node
ENTRYPOINT ["tini", "--", "/docker-entrypoint.sh"]

LABEL org.opencontainers.image.title="n8n" \
      org.opencontainers.image.description="Workflow Automation Tool" \
      org.opencontainers.image.source="https://github.com/n8n-io/n8n" \
      org.opencontainers.image.url="https://n8n.io" \
      org.opencontainers.image.version=${N8N_VERSION}------
name: n8n-developer
description: Use this agent for any n8n development task - frontend (Vue 3), backend (Node.js/TypeScript), workflow engine, node creation, or full-stack features. The agent automatically applies n8n conventions and best practices. Examples: <example>user: 'Add a new button to the workflow editor' assistant: 'I'll use the n8n-developer agent to implement this following n8n's design system.'</example> <example>user: 'Create an API endpoint for workflow export' assistant: 'I'll use the n8n-developer agent to build this API endpoint.'</example> <example>user: 'Fix the CSS issue in the node panel' assistant: 'I'll use the n8n-developer agent to fix this styling issue.'</example>
model: inherit
color: blue
---

You are an expert n8n developer with comprehensive knowledge of the n8n workflow automation platform. You handle both frontend (Vue 3 + Pinia + Design System) and backend (Node.js + TypeScript + Express + TypeORM) development.

## Core Expertise

**n8n Architecture**: Monorepo structure with pnpm workspaces, workflow engine (n8n-workflow, n8n-core), node development patterns, frontend (editor-ui package with Vue 3), backend (CLI package with Express), authentication flows, queue management, and event-driven patterns.

**Key Packages**:
- Frontend: packages/frontend/editor-ui (Vue 3 + Pinia), packages/frontend/@n8n/design-system, packages/frontend/@n8n/i18n
- Backend: packages/cli (Express + REST API), packages/core (workflow execution), packages/@n8n/db (TypeORM)
- Shared: packages/workflow, packages/@n8n/api-types

## Development Standards

**TypeScript**: Strict typing (never `any`), use `satisfies` over `as`, proper error handling with UnexpectedError from n8n-workflow.

**Frontend**: Vue 3 Composition API, Pinia stores, n8n design system components, CSS variables from design system, proper i18n with @n8n/i18n.

**Backend**: Controller-service-repository pattern, dependency injection with @n8n/di, @n8n/config for configuration, Zod schemas for validation, TypeORM with multi-database support.

## Workflow

1. **Analyze Requirements**: Identify affected packages and appropriate patterns using n8n conventions
   - If working from a Linear ticket, use Linear MCP (`mcp__linear-server__get_issue`) to fetch complete context
   - Review ticket description, comments, and linked GitHub issues
   - Use `gh` CLI and `git` commands for GitHub/git operations (e.g., `gh pr view`, `git log`)
2. **Plan Implementation**: Outline steps and dependencies
3. **Follow Patterns**: Apply n8n architectural patterns consistently
4. **Ensure Quality**: Run typecheck/lint, write tests, validate across databases
5. **Complete Implementation**: Provide working code with proper error handling and logging. Review for security vulnerabilities and only finalize when confident the solution is secure

Use pnpm for package management, work within appropriate package directories using pushd/popd, and build when type definitions change.

You deliver maintainable, well-typed code that integrates seamlessly with n8n's monorepo architecture.
name: n8n-linear-issue-triager
description: Use this agent proactively when a Linear issue is created, updated, or needs comprehensive analysis. This agent performs thorough issue investigation and triage including root cause analysis, severity assessment, and implementation scope identification.
model: inherit
color: red
---

You are an expert n8n Linear Issue Explorer and Analysis Agent, specializing in comprehensive investigation of Linear tickets and GitHub issues within the n8n workflow automation platform ecosystem.

**n8n Conventions**: This agent has deep knowledge of n8n conventions, architecture patterns, and best practices embedded in its expertise.

Your primary role is thorough investigation and context gathering to enable seamless handover to developers or implementation agents through comprehensive analysis and actionable intelligence.

## Core Mission
Provide thorough analysis and sufficient context for smooth handover - not implementation. Focus on investigation, root cause identification, and actionable intelligence gathering leveraging your deep n8n ecosystem knowledge.

## Investigation Capabilities

### 1. Deep Issue Analysis
- Fetch Linear ticket details including descriptions, comments, attachments, and linked resources
- Cross-reference related GitHub issues, pull requests, and community reports
- Examine and analyze git history and identify specific problematic commits to understand code evolution and potential regressions
- Analyze patterns and correlations across related issues within the n8n ecosystem
- Check for related issues or PRs with similar descriptions or file paths.

### 2. Root Cause Investigation
- Trace issues to specific commits, files, and line numbers across the monorepo
- Identify whether problems stem from recent changes, workflow engine updates, or node ecosystem changes
- Distinguish between configuration issues, code bugs, architectural problems, and node integration issues
- Analyze dependencies and cross-package impacts in TypeScript monorepo structure

### 3. Context Gathering
- **Implementation Area**: Clearly identify FRONTEND / BACKEND / BOTH / NODE ECOSYSTEM
- **Technical Scope**: Specific packages, files, workflow components, and code areas involved
- **User Impact**: Affected user segments, workflow types, and severity assessment
- **Business Context**: Customer reports, enterprise vs community impact, node usage patterns
- **Related Issues**: Historical context, similar resolved cases, and ecosystem-wide implications

### 4. Severity Assessment Framework
- **CRITICAL**: Data loss, silent failures, deployment blockers, workflow execution failures, security vulnerabilities
- **HIGH**: Core functionality broken, affects multiple users, monitoring/observability issues, node integration problems
- **MEDIUM**: UI/UX issues, non-critical feature problems, performance degradation, specific node issues
- **LOW**: Enhancement requests, minor bugs, cosmetic issues, node improvements

## Workflow

1. **Fetch Issue Details**: Get Linear ticket, comments, attachments, and related resources
   - Use Linear MCP tools (`mcp__linear-server__get_issue`, `mcp__linear-server__list_comments`) to fetch complete ticket data
   - Get all comments, attachments, and linked GitHub issues
   - Check for related Linear issues with similar symptoms
2. **Investigate Root Cause**: Trace to commits, files, and identify problematic changes
   - Use `git` commands to examine commit history, blame, and file changes
   - Use `gh` CLI to view PRs and issues (e.g., `gh pr view`, `gh issue view`)
   - Search codebase for related implementations
3. **Assess Severity**: Apply framework to determine priority level
4. **Generate Analysis**: Provide comprehensive handover report with actionable intelligence

## Investigation Output

Provide comprehensive analysis including:

1. **Root Cause Analysis**: Specific technical reason with commit/file references and ecosystem context
2. **Implementation Scope**: FRONTEND/BACKEND/BOTH/NODE with exact file paths and affected components
3. **Impact Assessment**: User segments affected, workflow scenarios impacted, and severity level
4. **Technical Context**: Architecture areas involved, workflow engine implications, node dependencies, related systems
5. **Investigation Trail**: Commits examined, patterns identified, related issues, ecosystem considerations
6. **Handover Intelligence**: Everything needed for developer or implementation agent to proceed immediately with full context

## Goal
Generate detailed investigative reports that provide complete context for immediate development handover, leveraging deep n8n ecosystem knowledge to ensure comprehensive analysis and actionable intelligence for complex workflow automation
platform issues.

## Important
**DO NOT post triage results to Linear.** Only generate the analysis as output. The user will decide what to share with the Linear ticket.
---
name: n8n-developer
description: Use this agent for any n8n development task - frontend (Vue 3), backend (Node.js/TypeScript), workflow engine, node creation, or full-stack features. The agent automatically applies n8n conventions and best practices. Examples: <example>user: 'Add a new button to the workflow editor' assistant: 'I'll use the n8n-developer agent to implement this following n8n's design system.'</example> <example>user: 'Create an API endpoint for workflow export' assistant: 'I'll use the n8n-developer agent to build this API endpoint.'</example> <example>user: 'Fix the CSS issue in the node panel' assistant: 'I'll use the n8n-developer agent to fix this styling issue.'</example>
model: inherit
color: blue
---
You are an expert n8n developer with comprehensive knowledge of the n8n workflow automation platform. You handle both frontend (Vue 3 + Pinia + Design System) and backend (Node.js + TypeScript + Express + TypeORM) development.

Core Expertise
n8n Architecture: Monorepo structure with pnpm workspaces, workflow engine (n8n-workflow, n8n-core), node development patterns, frontend (editor-ui package with Vue 3), backend (CLI package with Express), authentication flows, queue management, and event-driven patterns.

Key Packages:

Frontend: packages/frontend/editor-ui (Vue 3 + Pinia), packages/frontend/@n8n/design-system, packages/frontend/@n8n/i18n
Backend: packages/cli (Express + REST API), packages/core (workflow execution), packages/@n8n/db (TypeORM)
Shared: packages/workflow, packages/@n8n/api-types
Development Standards
TypeScript: Strict typing (never any), use satisfies over as, proper error handling with UnexpectedError from n8n-workflow.

Frontend: Vue 3 Composition API, Pinia stores, n8n design system components, CSS variables from design system, proper i18n with @n8n/i18n.

Backend: Controller-service-repository pattern, dependency injection with @n8n/di, @n8n/config for configuration, Zod schemas for validation, TypeORM with multi-database support.

Workflow
Analyze Requirements: Identify affected packages and appropriate patterns using n8n conventions
If working from a Linear ticket, use Linear MCP (mcp__linear-server__get_issue) to fetch complete context
Review ticket description, comments, and linked GitHub issues
Use gh CLI and git commands for GitHub/git operations (e.g., gh pr view, git log)
Plan Implementation: Outline steps and dependencies
Follow Patterns: Apply n8n architectural patterns consistently
Ensure Quality: Run typecheck/lint, write tests, validate across databases
Complete Implementation: Provide working code with proper error handling and logging. Review for security vulnerabilities and only finalize when confident the solution is secure
Use pnpm for package management, work within appropriate package directories using pushd/popd, and build when type definitions change.

You deliver maintainable, well-typed code that integrates seamlessly with n8n's monorepo architecture.# Node.js Release Working Group

## Release schedule

| Release  | Status              | Codename     |Initial Release | Active LTS Start | Maintenance Start | End-of-life               |
| :--:     | :---:               | :---:        | :---:          | :---:            | :---:             | :---:                     |
| [20.x][] | **Maintenance LTS** | [Iron][]     | 2023-04-18     | 2023-10-24       | 2024-10-22        | 2026-04-30                |
| [22.x][] | **Maintenance LTS** | [Jod][]      | 2024-04-24     | 2024-10-29       | 2025-10-21        | 2027-04-30                |
| [24.x][] | **Active LTS**      | [Krypton][]  | 2025-05-06     | 2025-10-28       | 2026-10-20        | 2028-04-30                |
| [25.x][] | **Current**         |              | 2025-10-15     | -                | 2026-04-01        | 2026-06-01                |

Dates are subject to change.

<p><img src="schedule.svg" alt="LTS Schedule"/></p>

The Release schedule is available also as a [JSON][] file.

### Release Phases

There are three phases that a Node.js release can be in: 'Current', 'Active
Long Term Support (LTS)', and 'Maintenance'. Odd-numbered release lines are not
promoted to LTS - they will not go through the 'Active LTS' or 'Maintenance'
phases.

 * Current - Should incorporate most of the non-major (non-breaking)
 changes that land on `nodejs/node` main branch.
 * Active LTS - New features, bug fixes, and updates that have been audited by
 the Release team and have been determined to be appropriate and stable for the
 release line.
 * Maintenance - Critical bug fixes and security updates. New features may be
 added at the discretion of the Release team - typically only in cases where
 the new feature supports migration to later release lines.

Changes required for critical security and bug fixes may lead to *semver-major*
changes landing within a release stream, such situations will be rare and will
land as *semver-minor*. Although, those changes should have a revert option included.

The term 'supported release lines' will be used to refer to all release lines
that are not End-of-Life.

### End-of-Life Releases

|  Release |      Status     |  Codename | Initial Release | Active LTS Start | Maintenance LTS Start | End-of-life |
|:--------:|:---------------:|:---------:|:---------------:|:----------------:|:---------------------:|:-----------:|
|  v0.10.x | **End-of-Life** |     -     |    2013-03-11   |         -        |       2015-10-01      |  2016-10-31 |
|  v0.12.x | **End-of-Life** |     -     |    2015-02-06   |         -        |       2016-04-01      |  2016-12-31 |
|  [4.x][] | **End-of-Life** | [Argon][] |    2015-09-08   |    2015-10-01    |       2017-04-01      |  2018-04-30 |
|  [5.x][] | **End-of-Life** |           |    2015-10-29   |         -        |                       |  2016-06-30 |
|  [6.x][] | **End-of-Life** | [Boron][] |    2016-04-26   |    2016-10-18    |       2018-04-30      |  2019-04-30 |
|  [7.x][] | **End-of-Life** |           |    2016-10-25   |         -        |                       |  2017-06-30 |
|  [8.x][] | **End-of-Life** | [Carbon][]|    2017-05-30   |    2017-10-31    |       2019-01-01      |  2019-12-31 |
|  [9.x][] | **End-of-Life** |           |    2017-10-01   |         -        |                       |  2018-06-30 |
| [10.x][] | **End-of-Life** |[Dubnium][]|    2018-04-24   |    2018-10-30    |       2020-05-19      |  2021-04-30 |
| [11.x][] | **End-of-Life** |           |    2018-10-23   |         -        |                       |  2019-06-01 |
| [12.x][] | **End-of-Life** | [Erbium][]|    2019-04-23   |    2019-10-21    |       2020-11-30      |  2022-04-30 |
| [13.x][] | **End-of-Life** |           |    2019-10-22   |         -        |                       |  2020-06-01 |
| [14.x][] | **End-of-Life** |[Fermium][]|    2020-04-21   |    2020-10-27    |       2021-10-19      |  2023-04-30 |
| [15.x][] | **End-of-Life** |           |    2020-10-20   |         -        |                       |  2021-06-01 |
| [16.x][] | **End-of-Life** |[Gallium][]|    2021-04-20   |    2021-10-26    |       2022-10-18      | [2023-09-11][nodejs16eol] |
| [17.x][] | **End-of-Life** |           |    2021-10-19   |         -        |                       |  2022-06-01 |
| [18.x][] | **End-of-Life** |[Hydrogen][]|   2022-04-19   |    2022-10-25    |       2023-10-18      |  2025-04-30 |
| [19.x][] | **End-of-Life** |           |    2022-10-18   |         -        |                       |  2023-06-01 |
| [21.x][] | **End-of-Life** |           |    2023-10-17   |         -        |       2024-04-01      |  2024-06-01 |
| [23.x][] | **End-of-Life** |           |    2024-10-15   |         -        |       2025-04-01      |  2025-06-01 |

## Mandate

The Release working group's purpose is:

* Management/execution of the release and support process for all releases.

Its responsibilities are:

* Define the release process.
* Define the content of releases.
* Generate and create releases.
* Test Releases.
* Manage the LTS and Current branches including backporting changes to
  these branches.
* Define the policy for what gets backported to release streams.

The Release working group is structured into teams and membership in
the working group does not automatically result in membership in these
teams. These teams are:

* Releasers team
* Backporters team
* CITGM team

The `releasers` team is entrusted with the secrets and CI access to be able
build and sign releases. **Additions to the releasers team must be approved
by the TSC following the process outlined in GOVERNANCE.md.**

The Release team manages the process/content of LTS releases
and the required backporting for these releases. Additions to the Release
team needs sign off from the rest of the Release team.

The Canary in the Gold Mine (CITGM) team maintains CITGM as one of
the key sanity checks for releases. This team maintains the CITGM
repository and works to keep CITGM builds running and passing regularly.
This also includes maintaining the CI jobs in collaboration with the Build
Working Group.

## Release Plan

New *semver-major* releases of Node.js are branched from `main` every six
months. New even-numbered versions are released in April and odd-numbered
versions in October.

In coordination with a new *odd-numbered* major release, the previous
*even-numbered* major version will transition to Long Term Support. The
transition to Long Term Support will happen in a *semver-minor* release and
should happen after the new major version is released.

Every even (LTS) major version will be actively maintained for 12 months from
the date it enters LTS coverage. Following those 12 months of active support,
the major version will transition into "maintenance" mode for 18 months. Prior
to Node.js 12, the active period was 18 months and the maintenance period 12
months. See [Releases Phases](#release-phases) for details of which changes
are expected to land during each release phase.

The exact date that a release will be moved to LTS, moved between LTS modes,
or deprecated will be chosen no later than the first day of the month it is to
change. If the release team plans to change the release date, it will be done
with no less than 14 days notice.

All LTS releases will be assigned a codename. A list of expected upcoming
codenames is available in [CODENAMES.md](./CODENAMES.md).

### LTS Staging Branches

Every LTS major version has two branches in the GitHub repository: a release
branch and a staging branch. The release branch is used to cut new releases.
Only members of the @nodejs/releasers team should land commits onto release branches.
The staging branch is used to land cherry-picked or backported commits from
main that need to be included in a future release. Only members of
@nodejs/backporters should land commits onto staging branches.

For example, for Node.js v4, there is a `v4.x` branch and a `v4.x-staging`
branch. When commits land in main that must be cherry-picked for a future
Node.js v4 release, those must be landed into the `v4.x-staging` branch. When
commits are backported for a future Node.js v4 release, those must come in the
form of pull requests opened against the `v4.x-staging` branch. **Commits are
only landed in the `v4.x` branch when a new `v4.x` release is being prepared.**

Generally, changes are expected to live in a *Current* release for at least 2
weeks before being backported. It is possible for a commit to land earlier at
the discretion of the Release working group.

[Argon]: https://nodejs.org/download/release/latest-argon/
[Boron]: https://nodejs.org/download/release/latest-boron/
[Carbon]: https://nodejs.org/download/release/latest-carbon/
[Dubnium]: https://nodejs.org/download/release/latest-dubnium/
[Erbium]: https://nodejs.org/download/release/latest-erbium/
[Fermium]: https://nodejs.org/download/release/latest-fermium/
[Gallium]: https://nodejs.org/download/release/latest-gallium/
[Hydrogen]: https://nodejs.org/download/release/latest-hydrogen/
[Iron]: https://nodejs.org/download/release/latest-iron/
[Jod]: https://nodejs.org/download/release/latest-jod/
[Krypton]: https://nodejs.org/download/release/latest-krypton/
[JSON]: schedule.json
[nodejs16eol]: https://nodejs.org/en/blog/announcements/nodejs16-eol/
[4.x]: https://nodejs.org/download/release/latest-v4.x/
[5.x]: https://nodejs.org/download/release/latest-v5.x/
[6.x]: https://nodejs.org/download/release/latest-v6.x/
[7.x]: https://nodejs.org/download/release/latest-v7.x/
[8.x]: https://nodejs.org/download/release/latest-v8.x/
[9.x]: https://nodejs.org/download/release/latest-v9.x/
[10.x]: https://nodejs.org/download/release/latest-v10.x/
[11.x]: https://nodejs.org/download/release/latest-v11.x/
[12.x]: https://nodejs.org/download/release/latest-v12.x/
[13.x]: https://nodejs.org/download/release/latest-v13.x/
[14.x]: https://nodejs.org/download/release/latest-v14.x/
[15.x]: https://nodejs.org/download/release/latest-v15.x/
[16.x]: https://nodejs.org/download/release/latest-v16.x/
[17.x]: https://nodejs.org/download/release/latest-v17.x/
[18.x]: https://nodejs.org/download/release/latest-v18.x/
[19.x]: https://nodejs.org/download/release/latest-v19.x/
[20.x]: https://nodejs.org/download/release/latest-v20.x/
[21.x]: https://nodejs.org/download/release/latest-v21.x/
[22.x]: https://nodejs.org/download/release/latest-v22.x/
[23.x]: https://nodejs.org/download/release/latest-v23.x/
[24.x]: https://nodejs.org/download/release/latest-v24.x/
[25.x]: https://nodejs.org/download/release/latest-v25.x/

The working group members are the union of the Releasers, Backporters
and CITGM team members listed below.

### Backporters team

<!-- ncu-team-sync.team(nodejs/backporters) -->

* [@aduh95](https://github.com/aduh95) - Antoine du Hamel
* [@BethGriggs](https://github.com/BethGriggs) - Beth Griggs
* [@codebytere](https://github.com/codebytere) - Shelley Vohr
* [@guybedford](https://github.com/guybedford) - Guy Bedford
* [@RafaelGSS](https://github.com/RafaelGSS) - Rafael Gonzaga
* [@richardlau](https://github.com/richardlau) - Richard Lau

<!-- ncu-team-sync end -->

## Releasers team

<!-- ncu-team-sync.team(nodejs/releasers) -->

* [@aduh95](https://github.com/aduh95) - Antoine du Hamel
* [@juanarbol](https://github.com/juanarbol) - Juan José
* [@marco-ippolito](https://github.com/marco-ippolito) - Marco Ippolito
* [@RafaelGSS](https://github.com/RafaelGSS) - Rafael Gonzaga
* [@richardlau](https://github.com/richardlau) - Richard Lau
* [@ruyadorno](https://github.com/ruyadorno) - Ruy Adorno
* [@targos](https://github.com/targos) - Michaël Zasso
* [@UlisesGascon](https://github.com/UlisesGascon) - Ulises Gascón

<!-- ncu-team-sync end -->

## CITGM team

* https://github.com/nodejs/citgm#citgm-team

## Emeritus

### LTS team
- [@addaleax](https://github.com/addaleax) - Anna Henningsen
- [@BethGriggs](https://github.com/BethGriggs) - Bethany Griggs
- [@bnoordhuis](https://github.com/bnoordhuis) - Ben Noordhuis
- [@danielleadams](https://github.com/danielleadams) - Danielle Adams
- [@ErisDS](https://github.com/ErisDS) - Hannah Wolfe
- [@Fishrock123](https://github.com/Fishrock123) - Jeremiah Senkpiel
- [@geek](https://github.com/geek) - Wyatt Preul
- [@gibfahn](https://github.com/gibfahn) - Gibson Fahnestock
- [@jasnell](https://github.com/jasnell) - James M Snell
- [@mhdawson](https://github.com/mhdawson) - Michael Dawson
- [@MylesBorins](https://github.com/MylesBorins) - Myles Borins
- [@othiym23](https://github.com/othiym23) - Forrest L Norvell
- [@rvagg](https://github.com/rvagg) - Rod Vagg
- [@sam-github](https://github.com/sam-github) - Sam Roberts
- [@shigeki](https://github.com/shigeki) - Shigeki Ohtsu
- [@srl295](https://github.com/srl295) - Steven R. Loomis
- [@trevnorris](https://github.com/trevnorris) - Trevor Norris
- [@yunong](https://github.com/yunong) - Yunong Xiao

### Releasers team
- [@bengl](https://github.com/bengl) - Bryan English
- [@BethGriggs](https://github.com/BethGriggs) - Bethany Griggs
- [@BridgeAR](https://github.com/BridgeAR) - Ruben Bridgewater
- [@cjihrig](https://github.com/cjihrig) - Colin Ihrig
- [@codebytere](https://github.com/codebytere) - Shelley Vohr
- [@danielleadams](https://github.com/danielleadams) - Danielle Adams
- [@evanlucas](https://github.com/evanlucas) - Evan Lucas
- [@Fishrock123](https://github.com/Fishrock123) - Jeremiah Senkpiel
- [@gibfahn](https://github.com/gibfahn) - Gibson Fahnestock
- [@jasnell](https://github.com/jasnell) - James M Snell
- [@MylesBorins](https://github.com/MylesBorins) - Myles Borins
- [@rvagg](https://github.com/rvagg) - Rod VaggThis file contains a list of codenames for LTS releases. Codenames for future releases are subject to change.

Argon (4.x 2015)
Boron (6.x 2016)
Carbon (8.x 2017)
Dubnium (10.x 2018)
Erbium (12.x 2019)
Fermium (14.x 2020)
Gallium (16.x 2021)
Hydrogen (18.x 2022)
Iron (20.x 2023)
Jod (22.x 2024)
Krypton (24.x 2025)
Lithium (26.x 2026)
Magnesium (28.x 2027)
Neon (30.x 2028)
Oxygen (32.x 2029)
Platinum (34.x 2030)
The release schedule is available as a JSON file.{
  "v0.8": {
    "start": "2012-06-25",
    "end": "2014-07-31"
  },
  "v0.10": {
    "start": "2013-03-11",
    "end": "2016-10-31"
  },
  "v0.12": {
    "start": "2015-02-06",
    "end": "2016-12-31"
  },
  "v4": {
    "start": "2015-09-08",
    "lts": "2015-10-12",
    "maintenance": "2017-04-01",
    "end": "2018-04-30",
    "codename": "Argon"
  },
  "v5": {
    "start": "2015-10-29",
    "maintenance": "2016-04-30",
    "end": "2016-06-30"
  },
  "v6": {
    "start": "2016-04-26",
    "lts": "2016-10-18",
    "maintenance": "2018-04-30",
    "end": "2019-04-30",
    "codename": "Boron"
  },
  "v7": {
    "start": "2016-10-25",
    "maintenance": "2017-04-30",
    "end": "2017-06-30"
  },
  "v8": {
    "start": "2017-05-30",
    "lts": "2017-10-31",
    "maintenance": "2019-01-01",
    "end": "2019-12-31",
    "codename": "Carbon"
  },
  "v9": {
    "start": "2017-10-01",
    "maintenance": "2018-04-01",
    "end": "2018-06-30"
  },
  "v10": {
    "start": "2018-04-24",
    "lts": "2018-10-30",
    "maintenance": "2020-05-19",
    "end": "2021-04-30",
    "codename": "Dubnium"
  },
  "v11": {
    "start": "2018-10-23",
    "maintenance": "2019-04-22",
    "end": "2019-06-01"
  },
  "v12": {
    "start": "2019-04-23",
    "lts": "2019-10-21",
    "maintenance": "2020-11-30",
    "end": "2022-04-30",
    "codename": "Erbium"
  },
  "v13": {
    "start": "2019-10-22",
    "maintenance": "2020-04-01",
    "end": "2020-06-01"
  },
  "v14": {
    "start": "2020-04-21",
    "lts": "2020-10-27",
    "maintenance": "2021-10-19",
    "end": "2023-04-30",
    "codename": "Fermium"
  },
  "v15": {
    "start": "2020-10-20",
    "maintenance": "2021-04-01",
    "end": "2021-06-01"
  },
  "v16": {
    "start": "2021-04-20",
    "lts": "2021-10-26",
    "maintenance": "2022-10-18",
    "end": "2023-09-11",
    "codename": "Gallium"
  },
  "v17": {
    "start": "2021-10-19",
    "maintenance": "2022-04-01",
    "end": "2022-06-01"
  },
  "v18": {
    "start": "2022-04-19",
    "lts": "2022-10-25",
    "maintenance": "2023-10-18",
    "end": "2025-04-30",
    "codename": "Hydrogen"
  },
  "v19": {
    "start": "2022-10-18",
    "maintenance": "2023-04-01",
    "end": "2023-06-01"
  },
  "v20": {
    "start": "2023-04-18",
    "lts": "2023-10-24",
    "maintenance": "2024-10-22",
    "end": "2026-04-30",
    "codename": "Iron"
  },
  "v21": {
    "start": "2023-10-17",
    "maintenance": "2024-04-01",
    "end": "2024-06-01"
  },
  "v22": {
    "start": "2024-04-24",
    "lts": "2024-10-29",
    "maintenance": "2025-10-21",
    "end": "2027-04-30",
    "codename": "Jod"
  },
  "v23": {
    "start": "2024-10-16",
    "maintenance": "2025-04-01",
    "end": "2025-06-01"
  },
  "v24": {
    "start": "2025-05-06",
    "lts": "2025-10-28",
    "maintenance": "2026-10-20",
    "end": "2028-04-30",
    "codename": "Krypton"
  },
  "v25": {
    "start": "2025-10-15",
    "maintenance": "2026-04-01",
    "end": "2026-06-01"
  },
  "v26": {
    "start": "2026-04-22",
    "lts": "2026-10-28",
    "maintenance": "2027-10-20",
    "end": "2029-04-30",
    "codename": ""
  }
}# Cross Project Council (CPC) Charter

## Section 1. Guiding Principle.

The OpenJS Foundation will operate transparently, openly, collaboratively, and ethically.
Project proposals, timelines, and status must not merely be open, but also easily visible to outsiders.

## Section 2. Evolution of OpenJS Foundation Governance.

Most large, complex open source communities have both a business and a technical governance model.
OpenJS Foundation’s technical leadership is the Cross Project Council (“CPC”).
OpenJS Foundation’s business leadership is the Board of Directors (the “Board”).

This Cross Project Council Charter reflects a carefully constructed, balanced role for the CPC and the Board in the governance of OpenJS Foundation.
The charter amendment process is for the OpenJS Foundation to propose changes using simple majority of the full OpenJS Foundation, the proposed changes being subject to review and approval by the Board.
The Board may additionally make amendments to the OpenJS Foundation charter at any time, though the Board will not interfere with day-to-day discussions, votes or meetings of the CPC.

The above notwithstanding, the CPC may make trivial changes to the charter without requiring board approval, including spelling and punctuation fixes, so long as they do not change the meaning of the text.

## Section 3. Board’s Role in Setting OpenJS Foundation’s Strategic Direction.

The Board will set the overall CPC Policy.
The policy will describe the overarching scope of the OpenJS Foundation initiative, OpenJS Foundation’s technical vision and direction and project release expectations in the form of expected cadence and intent.
The Board will use the CPC as a delegate body for governing technical implementation, individual project scope and direction while they remain within the scope and direction of the policies as described in the CPC Policy document and approved by the Board.

The Board sets the overall CPC policy through [Foundation mission and vision statements][] as well as within the [Foundation bylaws][].
The policy describes the overarching scope of the OpenJS Foundation initiatives, technical vision, and direction.
The Board uses the CPC as a delegate body for governing high-level technical policy and procedures while remaining within the scope and direction of the policies set by the Board.

## Section 4. Establishment of the CPC.

The CPC is an inclusive group with three types of participants:

* Observers
* Regular members
* Voting members

### Observers

Observers are free to attend meetings and participate in the work of the CPC as well as the consensus seeking process.

### Regular members

Regular members have made a commitment to be involved in an ongoing manner and take on roles and responsibilities as outlined below.
Regular member is implied when membership type is not specified.
Anyone who has been active in the work of the OpenJS Foundation, one of its member projects or other related activity as described in the CPC Governance may request to become a regular member by opening a PR to add themselves to the list of regular members.
Regular members remain for as long as they are active within the work of the CPC. Regular members who have not been active in GitHub, participated in meetings, or other work of the CPC for 3 months may be removed from the list of Regular members.
In addition, a Regular CPC member can be removed by voluntary resignation, or by a standard CPC motion.

### Voting members

### Current Structure (Through Fall 2026)

Voting members are selected as follows:

* Each Impact project may nominate up to two members through a process of their choosing. Once nominated the member must be ratified by the CPC Voting members before becoming a Voting member.
* Up to two Voting members may be nominated by the non-Impact projects based on a process set by the CPC.
* Up to two Voting members may be nominated by the Regular members. Once nominated these members must be ratified by the CPC Voting members before becoming a Voting member.

### New Structure (Effective Fall 2026 Election Cycle)

Beginning with the Fall 2026 election cycle, the CPC will transition from separate non-Impact project and Regular Member voting representatives to a unified class of Community Voting Members. Under this structure, voting members will be selected as follows:

* Each Impact project may nominate up to two Voting members through a process of their choosing. Once appointed, ratification occurs by opening an issue in the CPC repository announcing the appointment and following the [guidelines for merging PRs](./governance/GOVERNANCE.md#merging-prs-into-this-repository). Impact project voting members serve until they voluntarily step down, until their project appoints a replacement, or if they are removed by a CPC motion.
* Up to five Community Voting Members may be elected through a process defined in [Section 7. Elections](#section-7-elections). Community Voting Members must be Regular members of the CPC and must not currently be serving as a voting representative for an Impact project. 

### General Requirements and Expectations

Voting members are expected to make a time commitment which allows them to be responsive to CPC business, participate regularly in meetings and to participate in all voting matters (either by voting or specifically abstaining).
They are also expected to help to enable work of the regular CPC members by providing leadership, help with interactions with the board and Foundation staff and to generally help keep things moving.

Non Impact Voting members serve for a term of 1 year and must be re-nominated and ratified by the Voting CPC members each year. In addition, a Voting CPC member can be removed by voluntary resignation, by a standard CPC motion, or in accordance to the participation rules described for Regular members.

Changes to CPC membership should be posted in the agenda, and may be suggested as any other agenda item.

No more than one-fourth of the Voting CPC members may be affiliated with the same employer.
Removal or resignation of a Voting CPC member, or a change of employment by a Voting CPC member could cause more than one-fourth of the Voting CPC membership to be affiliated with the same employer.
Such a situation must be immediately remedied by the resignation or removal of one or more voting CPC members.

If a voting CPC member steps down before the end of their one year term or they are removed from the CPC by a CPC motion they may be replaced using the same process by which they were elected.
In this case the member will serve out the remainder of the term and must be re-confirmed at the end of the original one year term as if they had served for the full year.

The public portion of the CPC discussions and meetings will be open to all observers and members.

The CPC shall meet regularly using tools that enable participation by the community.
The meeting shall be directed by the CPC Chair or the CPC Vice Chair.
Responsibility for directing individual meetings may be delegated by the CPC Chair or by the CPC Vice Chair to any other CPC member.
The CPC Chair and CPC Vice Chair roles confer Voting member status, and will be selected by the CPC members through consensus or if necessary a vote as described in the section titled "Voting".
Minutes or an appropriate recording shall be taken and made available to the community through accessible public postings.

Voting members have the roles and responsibilities as outlined below.

## Section 5. Responsibilities and Expectations of the CPC

Subject to such policies as may be set by the Board, the CPC is responsible for:

   1. When needed, members of the CPC will be expected to create subcommittees or `collaboration spaces`.
      The CPC may delegate responsibilities and empower these groups to make decisions.
      Any of the responsibilities listed below not identified as being responsibilities of the Voting members may be delegated.
      For the remaining responsibilities, day-to-day work, investigation, and building recommendations may be delegated, however, the final responsibility will remain with the Voting members.
   1. Ensuring collaboration is the driving principle within a Project, between OpenJS Foundation Projects, and between OpenJS Foundation Projects and the broader community.
   1. Defining and maintaining neutral consensus for the technical vision for the OpenJS Foundation.
   1. Creating conceptual architecture for how the projects fit together.
   1. Voting members are responsible for approving new projects within the scope for OpenJS Foundation set by the Governing Board, as outlined in the Project Progression Process.
   1. Voting members are responsible for making final decisions on aligning, removing, or archiving projects, as outlined in Project Progression Process.
   1. Voting members are responsible for approving the charter (and any updates) for projects which are part of the OpenJS Foundation.
      As part of this review/approval the voting members will ensure that the updated charter is compatible with the mission of the OpenJS Foundation and reach consensus with the Board on more substantial changes before final approval by CPC.
   1. Voting members are responsible for approving funds for budgets delegated to the project.
   1. Voting members are responsible for approving new `collaboration spaces` as outlined in the OpenJS [COLLABORATION_NETWORK.md](./collaboration-spaces/COLLABORATION_NETWORK.md) process.
   1. Voting members are responsible for making final decisions on aligning `collaboration spaces`, removing or archiving `collaboration spaces`, as outlined in the OpenJS [COLLABORATION_NETWORK.md](./collaboration-spaces/COLLABORATION_NETWORK.md) process.
   1. Following and be up to date on Board/OpenJS Foundation initiatives and communicate them to the projects.
   1. Defining common practices to be implemented across OpenJS Foundation projects, if any.
   1. Mediating technical conflicts between OpenJS Foundation Projects when attempts to resolve those conflicts within the Project were unsuccessful.
   1. Managing the process for creating a yearly budget and approving a yearly budget to be presented to the board.
   1. The CPC serves as the OpenJS Foundation's primary technical liaison body with external open source projects, consortiums, and groups and is also responsible for ensuring that the OpenJS Foundation has appropriate technical participation in standards bodies by finding an encouraging members from individual projects to participate in these bodies.
   1. Managing the Individual membership program.
   1. Managing programs to improve inclusivity and diversity.
   1. Guiding projects into appropriate Foundation paths.
   1. Shared resources, tooling and funding for projects (Collaborator Summit Events, Travel fund).
   1. Supporting projects in their enforcement of Code of Conduct violations and escalation.
   1. Handling Code of Conduct violations for projects which do not have a documented process to do so.
   1. Developing a framework for community end-user feedback.
   1. Managing programs to offer mentorship to external individuals.
   1. Managing developer outreach programs.
   1. Supporting project advocacy and recognition programs.
   1. Identifying, recruiting and engaging prospective projects.
   1. Voting members are responsible for making a final decision if consensus cannot be reached among members.
   1. Members of the CPC, or committees consisting of subsets of the CPC membership will be expected to meet on a regular basis to discuss topics such as new project acceptance into the mentorship program, project graduation from the mentorship program, Project issues and conflicts, opportunities for collaboration between Projects, opportunities for the Foundation in the greater JavaScript community, etc.

<a name="section-6-non-responsibilities-of-the-cpc"></a>
<!-- this anchor tag exists to preserve links to the old name of the section -->

## Section 6. Responsibilities Delegated to OpenJS Foundation Projects

OpenJS Foundation Projects are self-governing entities. All policies and procedures for interacting with, contributing to and making decisions within a Project are defined, implemented, and documented by that Project’s Governing Body and community during the mentorship process. Specific technical decisions within a Project will be the responsibility of the Project and establishment and implementation of the decision making process will be done within the guidelines as defined by CPC charter. Those Project decisions and responsibilities include but are not limited to:

* Setting release dates.
* Release quality standards.
* Technical direction.
* Project governance and process.
* GitHub repository hosting.
* Conduct guidelines.
* Maintaining the list of additional Collaborators.
* Development process and any coding standards.
* Mediating technical conflicts between Collaborators.

Not withstanding the above, the Projects and the entire technical community will follow any processes as may be specified by the Board relating to the intake and license compliance review of contributions, including the OpenJS Foundation IP Policy.

## Section 7. Elections

For election of persons (such as the CPC Chair), a multiple-candidate method should be used, such as:

* [Condorcet][] or
* [Single Transferable Vote][]

Multiple-candidate methods may be reduced to simple election by plurality when there are only two candidates for one position to be filled.
No election is required if there is only one candidate and no objections to the candidates election.

The CPC will elect from amongst Regular and Voting members of the CPC:

* a CPC Chair and a CPC Vice Chair to work on building an agenda for CPC meetings
* CPC Directors to represent the Foundation's projects and related communities to the Board

The term for each of these roles is one year (as defined in the [OpenJS Foundation bylaws][]).

The CPC shall hold annual elections to select a CPC Chair, Vice Chair, and Directors.
The CPC can choose to hold those elections at different times of the year or concurrently.

There are no limits on the number of terms a CPC Chair, Vice Chair, or Director may serve.

The CPC Chair and Vice Chair may be (but are not required to be) CPC Directors.

### Who Votes

CPC Voting members at the time of the election may vote.

#### Community Voting Member Elections (Effective Fall 2026)

For the election of Community Voting Members, the electorate includes all CPC members: Impact Project Voting Members and Regular Members.


## Section 8. Board Representation

There are Board seats allocated for CPC members to represent the Foundation's projects and related communities to the Board.

CPC Board representatives are called CPC Directors.

The number of CPC Directors is defined in §4.3(d) and §4.3(e) of the [OpenJS Foundation bylaws][].

To be eligible, candidates must be Regular or Voting members of the CPC.

The CPC Voting Members will vote on candidates as described in the [Elections][] section.

## Section 9. Decision Making

The CPC follows a Lazy Consensus Seeking decision-making model inspired by the [Apache Software Foundation's model][ASF community].

Lazy consensus is achieved by making a proposal in a forum that will reach all members of the group who should have the opportunity to participate in reaching consensus and waiting for an appropriate amount of time given the nature of the decision for anyone to object. It is not necessary to get explicit approval to proceed from all members, silence indicates consent. However, objections must be resolved in order to achieve consensus.

## Section 10. Voting

Voting should only be done as a last resort.

If a proposal cannot reach consensus after a reasonable period of discussion, a voting member can call for objections to be overruled through an asynchronous vote. If this call is seconded by another voting member, an asynchronous vote must be organized. For the objections to be overruled, at least 2/3 of all voting members must vote in favor of overruling the objections.

This voting process is designed to make it difficult to move a proposal forward without consensus, but prevent someone from blocking progress through objections.

## Section 11. Definitions

**Project**: a technical collaboration effort that is organized through the project mentorship process and approved by the CPC.

**Contributors**: contribute code or other artifacts, but do not have the right to commit to the code base.
Contributors work with the Project's Collaborators to have code committed to the code base.
A Contributor may be promoted to a Collaborator by the Project's Governing Body.
Contributors should rarely be encumbered by the CPC or Board.

**Collaborator**: a Contributor within a Project that has made significant and valuable contributions and has been given commit-access to that Project repository.

**Governing Body**: a  group of Collaborators within a Project elected to represent the Project in an official decision making role as defined in the Project's governance policies.

## Section 12. Changes to this Document

Changes to this document require CPC consensus and approval from the OpenJS Foundation Board.

[ASF community]: https://community.apache.org/committers/decisionMaking.html#lazy-consensus
[Foundation mission and vision statements]:  https://openjsf.org/about/
[Foundation bylaws]:  https://bylaws.openjsf.org
[Elections]: #section-7-elections
[Condorcet]: http://en.wikipedia.org/wiki/Condorcet_method
[Single Transferable Vote]: http://en.wikipedia.org/wiki/Single_transferable_vote
[OpenJS Foundation bylaws]: https://bylaws.openjsf.org/### Accessibility Skills Student Series Spring 2025 Random Prize Draw Terms and Conditions

**1. Promoter**
The promoter of this prize draw is GitHub Education, a division of GitHub, Inc., located at 88 Colin P. Kelly Jr. Street, San Francisco, CA 94107, USA.

**2. Eligibility**

The prize draw is open to verified registered students of GitHub Education who are at least 13 years old.
- Participants under 18 years must have parental or guardian consent to enter.
- Employees of GitHub, Microsoft, their immediate families, and anyone otherwise connected with the organization are not eligible to enter.
- The prize draw is void in countries where such promotions are prohibited by law. This includes, but is not limited to, the following countries:
  - Belgium
  - Poland
  - Malaysia
  - Brunei
  - Bangladesh
  - Italy
  - Any other country where local laws prohibit free entry random prize draws

**3. Entry Period**

- The entry period begins on April 14, 2025, 11:59 PM EST and ends on May 15, 2025, at 11:59 PM EST.

**4. How to Enter**

- To to receive an entry, participants must visit the designated forum discussion and complete the challenge for that week

**5. Free Entry**

- Participation in the prize draw is free, and no purchase is necessary to enter or win.

**6. Parental/Guardian Consent**

- If you are under 18 years old, you must have parental or guardian consent to enter the prize draw. By entering, you confirm that you have obtained such consent.

**7. Selection of Winners**

- Up to 10 winners will be selected at random from all eligible entries between May 15 and May 22, 2025.
- The draw will be conducted using a verifiably random process under the supervision of an independent person or by an independent entity.

**8. Notification of Winners**

- Winners will be notified via the email associated with their GitHub account by May 31, 2025.
- Winners must respond to the notification within 7 days to claim their prize. Failure to respond within this period may result in forfeiture of the prize, and GitHub Education reserves the right to select an alternative winner.

**9. Prizes**

- Each winner will receive GitHub swag, which may include items such as T-shirts, stickers, or other branded merchandise.
- Prizes are non-transferable and no cash alternative is available.
- GitHub Education reserves the right to substitute any prize with another of equivalent value without giving notice.

**10. Data Protection**

- Participants’ personal data will be used solely for the purposes of administering the prize draw and in accordance with GitHub’s privacy policy.

**11. General Conditions**

- GitHub Education reserves the right to cancel or amend the prize draw and these terms and conditions without notice in the event of a catastrophe, war, civil or military disturbance, or any actual or anticipated breach of any applicable law or regulation or any other event outside GitHub Education’s control.
- GitHub Education is not responsible for inaccurate prize details supplied to any participant by any third party connected with this prize draw.

**12. Governing Law**

- This prize draw and these terms and conditions will be governed by the laws of the country where the participant resides, provided that such laws do not conflict with the laws of the United States. Any disputes will be subject to the non-exclusive jurisdiction of the courts of the country of the participant’s residence.

**13. Limitation of Liability**

- By entering, participants agree to release and hold harmless GitHub Education, GitHub, Inc., and Microsoft from any liability, illness, injury, death, loss, litigation, claim, or damage that may occur, directly or indirectly, whether caused by negligence or not, from such entrant’s participation in the prize draw and/or their acceptance, possession, use, or misuse of any prize or any portion thereof.

**14. Sponsor**

- The prize draw is sponsored by GitHub Education.

For further information or any questions regarding the prize draw, please contact [morganersery1@github.com].# MICROSOFT RESPONSIBLE AI FOR OPEN SOURCE SWEEPSTAKES  
## OFFICIAL RULES  

### SPONSOR  
These Official Rules (“Rules”) govern the operation of the Microsoft Responsible AI for Open Source Sweepstakes (“Sweepstakes”). Microsoft Corporation, One Microsoft Way, Redmond, WA, 98052, USA, is the Sweepstakes sponsor (“Sponsor”).  

### DEFINITIONS  
In these Rules, "Microsoft", "we", "our", and "us" refer to Sponsor and “you” and "yourself" refers to a Sweepstakes participant, or the parent/legal guardian of any Sweepstakes entrant who has not reached the age of majority to contractually obligate themselves in their legal place of residence. By entering you (your parent/legal guardian if you are not the age of majority in your legal place of residence) agree to be bound by these Rules.  

### ENTRY PERIOD  
The Sweepstakes starts at 12:00 a.m. Pacific Time (PT) on July 24, 2025, and ends at 11:59 p.m. PT on August 21, 2025 (“Entry Period”).  

### ELIGIBILITY  
To enter, you must be 18 years of age or older. If you are 18 years of age or older but have not reached the age of majority in your legal place of residence, then you must have consent of a parent/legal guardian.  

Employees and directors of Microsoft Corporation and its subsidiaries, affiliates, advertising agencies, and Sweepstakes Parties are not eligible, nor are persons involved in the execution or administration of this promotion, or the family members of each above (parents, children, siblings, spouse/domestic partners, or individuals residing in the same household). Void in Cuba, Iran, North Korea, Sudan, Syria, Region of Crimea, Russia, and where prohibited.  

### HOW TO ENTER  
No Purchase Necessary.  

You will receive one Sweepstakes entry when you participate in the marked discussion on the GitHub Community during our entry period. There will be three (3) different challenges in total where you’ll be able to comment on the discussion stating that you completed the challenge. Challenges vary from sharing insights of codebase deep dive with AI, rewriting a Pull Request, and posting your learnings from an AI-generated code review. To participate in the GitHub Community, you must have a free GitHub account which you can sign up for at GitHub.com.  

The entry limit is one entry per challenge (there are three challenges).  

Any attempt by you to obtain more than the stated number of entries by using multiple/different accounts, identities, registrations, logins, or any other methods will void your entries and you may be disqualified. Use of any automated system to participate is prohibited.  

We are not responsible for excess, lost, late, or incomplete entries. If disputed, entries will be deemed submitted by the “authorized account holder” of the email address, social media account, or other method used to enter. The “authorized account holder” is the natural person assigned to an email address by an internet or online service provider, or other organization responsible for assigning email addresses.  

### WINNER SELECTION AND NOTIFICATION  
Pending confirmation of eligibility, potential prize winners will be selected by Microsoft or their Agent in a random drawing from among all eligible entries received within 7 days following the Entry Period.  

Winners will be notified via the contact information provided during entry no more than 7 days following the drawing with prize claim instructions, including submission deadlines. If a selected winner cannot be contacted, is ineligible, fails to claim a prize or fails to return any forms, the selected winner will forfeit their prize and an alternate winner will be selected time allowing. If you are a potential winner and you are 18 or older but have not reached the age of majority in your legal place of residence, we may require your parent/legal guardian to sign all required forms on your behalf. Only three alternate winners will be selected, after which unclaimed prizes will remain unawarded.  

### PRIZES  
The following prizes will be awarded:  

Ten (10) Grand Prizes. Each winner will receive:  
A(n) GitHub Shop voucher. Approximate Retail Value (ARV) $30.00.  

The ARV of electronic prizes is subject to price fluctuations in the consumer marketplace based on, among other things, any gap in time between the date the ARV is estimated for purposes of these Official Rules and the date the prize is awarded or redeemed. We will determine the value of the prize to be the fair market value at the time of prize award.  

The total Approximate Retail Value (ARV) of all prizes: $300  

We will only award one (1) prize per person/company during the Entry Period. No more than the stated number of prizes will be awarded. No substitution, transfer, or assignment of prize permitted, except that Microsoft reserves the right to substitute a prize of equal or greater value in the event the offered prize is unavailable. Microsoft products awarded as prizes are awarded “AS IS” and WITHOUT WARRANTY OF ANY KIND, express or implied (including any implied warranty of merchantability or fitness for a particular purpose); you assume the entire risk of quality and performance, and should the prizes prove defective, you assume the entire cost of all necessary servicing or repair. This is so even if the Microsoft product mentions a warranty on its packaging, in a manual, or in marketing materials; no warranty applies to Microsoft products awarded as prizes.  

Microsoft does not give any warranty of any kind, express or implied (including any implied warranty of merchantability or fitness for a particular purpose) on products made by a company other than Microsoft that are awarded as prizes. Please contact the manufacturer to see if it is covered by that company’s warranty.  

Prizes will be sent no later than 28 days after winner selection. Prize winners may be required to complete and return prize claim and/or tax forms (“Forms”) within the deadline stated in the winner notification. Taxes on the prize, if any, are the sole responsibility of the winner, who is advised to seek independent counsel regarding the tax implications of accepting a prize. By accepting a prize, you agree that Microsoft may use your entry, name, image and hometown online and in print, or in any other media, in connection with this Sweepstakes without payment or compensation to you, except where prohibited by law.  

### ODDS  
The odds of winning are based on the number of eligible entries received.  

### GENERAL CONDITIONS AND RELEASE OF LIABILITY  
To the extent allowed by law, by entering you agree to release and hold harmless Microsoft and its respective parents, partners, subsidiaries, affiliates, employees, and agents from any and all liability or any injury, loss, or damage of any kind arising in connection with this Sweepstakes or any prize won.  

All local laws apply. The decisions of Microsoft are final and binding.  

We reserve the right to cancel, change, or suspend this Sweepstakes for any reason, including cheating, technology failure, catastrophe, war, or any other unforeseen or unexpected event that affects the integrity of this Sweepstakes, whether human or mechanical. If the integrity of the Sweepstakes cannot be restored, we may select winners from among all eligible entries received before we had to cancel, change or suspend the Sweepstakes.  

If you attempt or we have strong reason to believe that you have compromised the integrity or the legitimate operation of this Sweepstakes by cheating, hacking, creating a bot or other automated program, or by committing fraud in any way, we may seek damages from you to the full extent of the law and you may be banned from participation in future Microsoft promotions.  

### USE OF YOUR ENTRY  
Personal data you provide while entering this Sweepstakes will be used by Microsoft and/or its agents and prize fulfillers acting on Microsoft’s behalf only for the administration and operation of this Sweepstakes and in accordance with the Microsoft Privacy Statement.  

### GOVERNING LAW  
This Sweepstakes will be governed by the laws of the State of Washington, and you consent to the exclusive jurisdiction and venue of the courts of the State of Washington for any disputes arising out of this Sweepstakes.  

### WINNERS LIST  
Send an email to lgalante@microsoft.com with the subject line “Responsible AI for Open Source Sweepstakes winners” within 30 days of August 21, 2025 to receive a list of winners that received a prize worth $25.00 or more.  **MICROSOFT GITHUB TAKE YOUR SKILLS TO NEW HEIGHTS**   
**WITH OUR GITHUB COPILOT SKILLS JOURNEY SWEEPSTAKES**  
**OFFICIAL RULES**

1. **SPONSOR**  

   These Official Rules (“Rules”) govern the operation of the Microsoft GitHub Take your skills to new heights with our GitHub Copilot Skills Journey Sweepstakes (“Sweepstakes”). Microsoft Corporation, One Microsoft Way, Redmond, WA, 98052, USA, is the Sweepstakes sponsor (“Sponsor”).

2. **DEFINITIONS**  

   In these Rules, “Microsoft”, “we”, “our”, and “us”, refer to Sponsor and “you” and “yourself” refers to a Sweepstakes participant, or the parent/legal guardian of any Sweepstakes entrant who has not reached the age of majority to contractually obligate themselves in their legal place of residence. “Event” refers to the Take your skills to new heights with our GitHub Copilot Skills Journey held online from January 20-February 17 (the “Program”). By entering you (your parent/legal guardian if you are not the age of majority in your legal place of residence) agree to be bound by these Rules.


3. **ENTRY PERIOD**  

   The Sweepstakes will operate during regular Program hours from January 20, 2026 to February 17, 2026 (“Entry Period”). 

4. **ELIGIBILITY**  

   Open to any registered Event attendee fourteen (14) years of age or older with an active GitHub account. If you are fourteen (14) years of age or older but have not reached the age of majority in your legal place of residence, then you must have the consent of a parent/legal guardian.  

   Employees and directors of Microsoft Corporation and its subsidiaries, affiliates, advertising agencies, and Sweepstakes Parties are not eligible, nor are persons involved in the execution or administration of this Sweepstakes or the family members of each above (parents, children, siblings, spouse/domestic partners, or individuals residing in the same household). Void in Cuba, Iran, North Korea, Sudan, Syria, Region of Crimea, Russia, and where otherwise prohibited by law. 

5. **HOW TO ENTER**  

   No Purchase Necessary. During your participation in the Program, you will develop an application using GitHub Copilot Agent (your “App”). The requirements for your App will be shared during the Program. You will receive entries by visiting the web site for the Sweepstakes at [https://github.com/orgs/community/discussions/184217](https://nam12.safelinks.protection.outlook.com/?url=https%3A%2F%2Fgithub.com%2Forgs%2Fcommunity%2Fdiscussions%2F184217&data=05%7C02%7Cjragen%40shb.com%7C214d5efcaa094978688708de553ee252%7C7be5e27659ab444899e76ab9030adfbf%7C1%7C0%7C639041925774084398%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=s63vyFIZ7X%2BxljukEXS312tdo8%2BHdfCnxuNT4hqa1%2B4%3D&reserved=0) (“Sweepstakes Website”) and completing the following actions during each week of the Program:  


| Week | Action |
| :---- | :---- |
| Week 1 | Access the weekly comment and follow the instructions to receive one (1) entry |
| Week 2 | Access the weekly comment and follow the instructions to receive one (1) entry |
| Week 3 | Access the weekly comment and follow the instructions to receive one (1) entry |
| Week 4 | Share via comment a link to the final version of your App to receive one (1) entry |



   The entry limit is one (1) per person per week, and a total of four (4) entries overall. Any attempt by you to obtain more than the stated number of entries by using multiple/different accounts, identities, registrations, logins, or any other methods will void your entries and you may be disqualified. Use of any automated system to participate is prohibited.



   We are not responsible for excess, lost, late, or incomplete entries. If disputed, entries will be deemed submitted by the “authorized account holder” of the email address, social media account, or other method used to enter. The “authorized account holder” is the natural person assigned to an email address by an internet or online service provider, or other organization responsible for assigning email addresses. 

6. **WINNER SELECTION AND NOTIFICATION**  

   Pending confirmation of eligibility, potential prize winners will be selected by Microsoft or our designated agent within seven (7) days following the event from among all eligible entries received.

          
Potential winners will be notified via the contact information provided during entry no more than seven (7) days following the drawing with prize claim instructions, including submission deadlines.

Prizes will be sent no later than twenty-eight (28) days after winner selection. Prize winners may be required to complete and return prize claim and/or tax forms (“Forms”) within the deadline stated in the winner notification. If a selected winner cannot be contacted, is ineligible, fails to claim a prize or fails to return any Forms, the selected winner will forfeit their prize and an alternate winner will be selected. Only three (3) alternate winners will be selected, after which unclaimed prizes will remain unawarded.

7. **PRIZES**  


The following prizes will be awarded: 

**Ten (10) Grand Prizes.** Each winner will receive a gift card to the GitHub shop. Approximate Retail Value (ARV) $30.00 USD.

The ARV of electronic prizes is subject to price fluctuations in the consumer marketplace based on, among other things, any gap in time between the date the ARV is estimated for purposes of these Official Rules and the date the prize is awarded or redeemed. We will determine the value of the prize to be the fair market value at the time of prize award.

The total Approximate Retail Value (ARV) of all prizes: **$300.00 USD**

We will only award one (1) prize per person. No more than the stated number of prizes will be awarded. No substitution, transfer, or assignment of prize permitted, except that Microsoft reserves the right to substitute a prize of equal or greater value in the event the offered prize is unavailable. Microsoft products awarded as prizes are awarded “AS IS” and WITHOUT WARRANTY OF ANY KIND, express or implied (including, but not limited to, any implied warranty of merchantability, fitness for a particular purpose, or non-infringement); you assume the entire risk of quality and performance, and should the prizes prove defective, you assume the entire cost of all necessary servicing or repair. This is so even if the Microsoft product mentions a warranty on its packaging, in a manual, or in marketing materials; no warranty applies to Microsoft products awarded as prizes. Microsoft does not give any warranty of any kind, express or implied (including any implied warranty of merchantability or fitness for a particular purpose) on products made by a company other than Microsoft that are awarded as prizes. Please contact the manufacturer to see if it is covered by that company’s warranty.

Taxes on the prize, if any, are the sole responsibility of the winner, who is advised to seek independent counsel regarding the tax implications of accepting a prize. By accepting a prize, you agree that Microsoft may use your entry, name, image and hometown online and in print, or in any other media, in connection with this Sweepstakes without payment or compensation to you, except where prohibited by law.

8. **ODDS**  

   The odds of winning are based on the number and/or quality of eligible entries received. 

9. **GENERAL CONDITIONS AND RELEASE OF LIABILITY**  

   To the extent allowed by law, by entering you agree to release and hold harmless Microsoft and its respective parents, partners, subsidiaries, affiliates, employees, and agents from any and all liability or any injury, loss, or damage of any kind arising in connection with this Sweepstakes or any prize won.  

   All federal, state, and local laws and regulations apply. In the event of a dispute, the decisions of Microsoft are final and binding.  

   We reserve the right to cancel, change, or suspend this Sweepstakes for any reason without prior notice, including (but not limited to) cheating, technology failure, catastrophe, war, or any other unforeseen or unexpected event that affects the integrity of this Sweepstakes, whether human or mechanical. If the integrity of the Sweepstakes cannot be restored, we may select winners from among all eligible entries received before we had to cancel, change or suspend the Sweepstakes.   

   If you attempt or we have strong reason to believe that you have compromised the integrity or the legitimate operation of this Sweepstakes by cheating, hacking, creating a bot or other automated program, or by committing fraud in any way, we may seek damages from you to the full extent of the law and you may be banned from participation in future Microsoft promotions. 

10. **GOVERNING LAW**  

    This Sweepstakes will be governed by the laws of the State of Washington, and you consent to the exclusive jurisdiction and venue of the courts of the State of Washington for any disputes arising out of this Sweepstakes.

11. **WINNERS LIST**  

    Send an email to [ghostinhershell@github.com](mailto:ghostinhershell@github.com) with the subject line “Take your skills to new heights with our GitHub Copilot Skills Journey winners” within thirty (30) days of February 28, 2026 to receive a list of winners that received a prize worth $25.00 USD or more.

12. **PRIVACY**

    Your privacy is important to us. Microsoft uses the personal data you provide on this form to notify you of important information about our products, upgrades and enhancements, and to send you information about other Microsoft products and services. We share your personal data with your consent or as necessary to complete any transaction or provide any service you have requested. We also share data with Microsoft-controlled affiliates and subsidiaries; with vendors working on our behalf; when required by law or to respond to legal process; to protect our customers; to protect lives; to maintain the security of our services; and to protect the rights or property of Microsoft. Microsoft is committed to protecting the security of your personal data. We use a variety of security technologies and procedures to help protect your personal information from unauthorized access, use, or disclosure. Your personal data is never shared outside the company without your permission, except under conditions explained above. For more information about Microsoft’s Privacy Practices, see [https://aka.ms.privacy](https://aka.ms.privacy). 

    If you believe that Microsoft has not adhered to this statement, please contact us at [https://aka.ms/privacyresponse](https://aka.ms/privacyresponse). We will respond to questions within thirty (30) days.For further information or any questions regarding the prize draw, please contact [queenofcorgis@github.com].### Meet the Maintainers March 2025 Random Prize Draw Terms and Conditions

**1. Promoter**
The promoter of this prize draw is GitHub Community, a division of GitHub, Inc., located at 88 Colin P. Kelly Jr. Street, San Francisco, CA 94107, USA.

**2. Eligibility**

The prize draw is open to verified registered GitHub users who are at least 13 years old.
- Employees of GitHub, Microsoft, their immediate families, and anyone otherwise connected with the organization are not eligible to enter.
- Participants under 18 years must have parental or guardian consent to enter.
- The prize draw is void in countries where such promotions are prohibited by law. This includes, but is not limited to, the following countries:
  - Belgium
  - Poland
  - Malaysia
  - Brunei
  - Bangladesh
  - Italy
  - Any other country where local laws prohibit free entry random prize draws

**3. Entry Period**

- The entry period begins on March 11, 2025 11:59 PM EST and ends on March 18, 2025, at 11:59 PM EST.

**4. How to Enter**

- To enter, participants must visit the designated forum discussion and ask a question. 
- Entries must be original and not generated entirely by AI tools. AI tools may be used for grammar and spelling checks only.

**5. Free Entry**

- Participation in the prize draw is free, and no purchase is necessary to enter or win.

**6. Parental/Guardian Consent**

- If you are under 18 years old, you must have parental or guardian consent to enter the prize draw. By entering, you confirm that you have obtained such consent.

**7. Selection of Winners**

- Up to 5 winners will be selected at random from all eligible entries between March 11 and March 18, 2025.
- The draw will be conducted using a verifiably random process under the supervision of an independent person or by an independent entity.

**8. Notification of Winners**

- Winners will be notified via the discussion by March 25, 2025. 
- Winners must fill out the form within 7 days to claim their prize. Failure to respond within this period may result in forfeiture of the prize, and GitHub Education reserves the right to select an alternative winner.

**9. Prizes**

- Each winner will receive a Github Shop voucher
- Prizes are non-transferable and no cash alternative is available.
- GitHub Community reserves the right to substitute any prize with another of equivalent value without giving notice.

**10. Data Protection**

- Participants’ personal data will be used solely for the purposes of administering the prize draw and in accordance with GitHub’s privacy policy.

**11. General Conditions**

- GitHub Community reserves the right to cancel or amend the prize draw and these terms and conditions without notice in the event of a catastrophe, war, civil or military disturbance, or any actual or anticipated breach of any applicable law or regulation or any other event outside GitHub Education’s control.
- GitHub Community is not responsible for inaccurate prize details supplied to any participant by any third party connected with this prize draw.

**12. Governing Law**

- This prize draw and these terms and conditions will be governed by the laws of the country where the participant resides, provided that such laws do not conflict with the laws of the United States. Any disputes will be subject to the non-exclusive jurisdiction of the courts of the country of the participant’s residence.

**13. Limitation of Liability**

- By entering, participants agree to release and hold harmless GitHub Community, GitHub, Inc., and Microsoft from any liability, illness, injury, death, loss, litigation, claim, or damage that may occur, directly or indirectly, whether caused by negligence or not, from such entrant’s participation in the prize draw and/or their acceptance, possession, use, or misuse of any prize or any portion thereof.

**14. Sponsor**

- The prize draw is sponsored by GitHub Community.

For further information or any questions regarding the prize draw, please contact [queenofcorgis@github.com].![Banner image](https://user-images.githubusercontent.com/10284570/173569848-c624317f-42b1-45a6-ab09-f0ea3c247648.png)

# n8n - Secure Workflow Automation for Technical Teams

n8n is a workflow automation platform that gives technical teams the flexibility of code with the speed of no-code. With 400+ integrations, native AI capabilities, and a fair-code license, n8n lets you build powerful automations while maintaining full control over your data and deployments.

![n8n.io - Screenshot](https://raw.githubusercontent.com/n8n-io/n8n/master/assets/n8n-screenshot-readme.png)

## Key Capabilities

- **Code When You Need It**: Write JavaScript/Python, add npm packages, or use the visual interface
- **AI-Native Platform**: Build AI agent workflows based on LangChain with your own data and models
- **Full Control**: Self-host with our fair-code license or use our [cloud offering](https://app.n8n.cloud/login)
- **Enterprise-Ready**: Advanced permissions, SSO, and air-gapped deployments
- **Active Community**: 400+ integrations and 900+ ready-to-use [templates](https://n8n.io/workflows)

## Quick Start

Try n8n instantly with [npx](https://docs.n8n.io/hosting/installation/npm/) (requires [Node.js](https://nodejs.org/en/)):

```
npx n8n
```

Or deploy with [Docker](https://docs.n8n.io/hosting/installation/docker/):

```
docker volume create n8n_data
docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

Access the editor at http://localhost:5678

## Resources

- 📚 [Documentation](https://docs.n8n.io)
- 🔧 [400+ Integrations](https://n8n.io/integrations)
- 💡 [Example Workflows](https://n8n.io/workflows)
- 🤖 [AI & LangChain Guide](https://docs.n8n.io/advanced-ai/)
- 👥 [Community Forum](https://community.n8n.io)
- 📖 [Community Tutorials](https://community.n8n.io/c/tutorials/28)

## Support

Need help? Our community forum is the place to get support and connect with other users:
[community.n8n.io](https://community.n8n.io)

## License

n8n is [fair-code](https://faircode.io) distributed under the [Sustainable Use License](https://github.com/n8n-io/n8n/blob/master/LICENSE.md) and [n8n Enterprise License](https://github.com/n8n-io/n8n/blob/master/LICENSE_EE.md).

- **Source Available**: Always visible source code
- **Self-Hostable**: Deploy anywhere
- **Extensible**: Add your own nodes and functionality

[Enterprise licenses](mailto:license@n8n.io) available for additional features and support.

Additional information about the license model can be found in the [docs](https://docs.n8n.io/sustainable-use-license/).

## Contributing

Found a bug 🐛 or have a feature idea ✨? Check our [Contributing Guide](https://github.com/n8n-io/n8n/blob/master/CONTRIBUTING.md) for a setup guide & best practices.

## Join the Team

Want to shape the future of automation? Check out our [job posts](https://n8n.io/careers) and join our team!

## What does n8n mean?

**Short answer:** It means "nodemation" and is pronounced as n-eight-n.

**Long answer:** "I get that question quite often (more often than I expected) so I decided it is probably best to answer it here. While looking for a good name for the project with a free domain I realized very quickly that all the good ones I could think of were already taken. So, in the end, I chose nodemation. 'node-' in the sense that it uses a Node-View and that it uses Node.js and '-mation' for 'automation' which is what the project is supposed to help with. However, I did not like how long the name was and I could not imagine writing something that long every time in the CLI. That is when I then ended up on 'n8n'." - **Jan Oberhauser, Founder and CEO, n8n.io**
## [docker compose up --build -d
npm install -g @google/gemini-cli@preview# En la Etapa 2
RUN pip install --no-cache-dir -r requirements.txt && \
    find /usr/local -depth \
		\( \
			\( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
			-o \
			\( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
		\) -exec rm -rf '{}' +
docker logs -f sentinel_omega_container
npm uninstall -g @google/gemini-cli
```]>
docker compose up --build -d
docker exec -it sentinel_omega_container /bin/bash
## [docker compose up --build -d
npm install -g @google/gemini-cli@preview# En la Etapa 2
RUN pip install --no-cache-dir -r requirements.txt && \
    find /usr/local -depth \
		\( \
			\( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
			-o \
			\( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
		\) -exec rm -rf '{}' +
docker logs -f sentinel_omega_container
npm uninstall -g @google/gemini-cli
```]>
docker compose up --build -d
docker exec -it sentinel_omega_container /bin/bash
packages/nodes-base/nodes/Xml/Xml.node.tsARG NODE_VERSION=24.13.1
ARG PYTHON_VERSION=3.13

# ==============================================================================
# STAGE 1: JavaScript runner (@n8n/task-runner) artifact from CI
# ==============================================================================
FROM node:${NODE_VERSION}-alpine AS javascript-runner-builder
COPY ./dist/task-runner-javascript /app/task-runner-javascript

WORKDIR /app/task-runner-javascript

RUN corepack enable pnpm

# Remove `catalog` and `workspace` references from package.json to allow `pnpm add` in extended images
RUN node -e "const pkg = require('./package.json'); \
    Object.keys(pkg.dependencies || {}).forEach(k => { \
        const val = pkg.dependencies[k]; \
        if (val === 'catalog:' || val.startsWith('catalog:') || val.startsWith('workspace:')) \
            delete pkg.dependencies[k]; \
    }); \
    Object.keys(pkg.devDependencies || {}).forEach(k => { \
        const val = pkg.devDependencies[k]; \
        if (val === 'catalog:' || val.startsWith('catalog:') || val.startsWith('workspace:')) \
            delete pkg.devDependencies[k]; \
    }); \
    delete pkg.devDependencies; \
    require('fs').writeFileSync('./package.json', JSON.stringify(pkg, null, 2));"

# Install moment (special case for backwards compatibility)
RUN rm -f node_modules/.modules.yaml && \
    pnpm add moment@2.30.1 --prod --no-lockfile

# ==============================================================================
# STAGE 2: Python runner build (@n8n/task-runner-python) with uv
# Produces a relocatable venv tied to the python version used
# ==============================================================================
FROM python:${PYTHON_VERSION}-alpine AS python-runner-builder
ARG TARGETPLATFORM
ARG UV_VERSION=0.8.14

RUN set -e; \
  case "$TARGETPLATFORM" in \
    "linux/amd64") UV_ARCH="x86_64-unknown-linux-musl" ;; \
    "linux/arm64") UV_ARCH="aarch64-unknown-linux-musl" ;; \
    *) echo "Unsupported platform: $TARGETPLATFORM" >&2; exit 1 ;; \
  esac; \
  mkdir -p /tmp/uv && cd /tmp/uv; \
  wget -q "https://github.com/astral-sh/uv/releases/download/${UV_VERSION}/uv-${UV_ARCH}.tar.gz"; \
  wget -q "https://github.com/astral-sh/uv/releases/download/${UV_VERSION}/uv-${UV_ARCH}.tar.gz.sha256"; \
  sha256sum -c "uv-${UV_ARCH}.tar.gz.sha256"; \
        tar -xzf "uv-${UV_ARCH}.tar.gz"; \
  install -m 0755 "uv-${UV_ARCH}/uv" /usr/local/bin/uv; \
  cd / && rm -rf /tmp/uv

WORKDIR /app/task-runner-python

COPY packages/@n8n/task-runner-python/pyproject.toml \
     packages/@n8n/task-runner-python/uv.lock** \
     packages/@n8n/task-runner-python/.python-version** \
     ./

RUN uv venv
RUN uv sync \
                        --frozen \
                        --no-editable \
                        --no-install-project \
                        --no-dev \
                        --all-extras

COPY packages/@n8n/task-runner-python/ ./
RUN uv sync \
      --frozen \
                        --no-dev \
                        --all-extras \
      --no-editable

# Install the python runner package itself into site packages. We can remove the src directory then
RUN uv pip install . && rm -rf /app/task-runner-python/src

# ==============================================================================
# STAGE 3: Task Runner Launcher download
# ==============================================================================
FROM alpine:3.22 AS launcher-downloader
ARG TARGETPLATFORM
ARG LAUNCHER_VERSION=1.4.3

RUN set -e; \
    case "$TARGETPLATFORM" in \
        "linux/amd64") ARCH_NAME="amd64" ;; \
        "linux/arm64") ARCH_NAME="arm64" ;; \
        *) echo "Unsupported platform: $TARGETPLATFORM" && exit 1 ;; \
    esac; \
    mkdir /launcher-temp && cd /launcher-temp; \
    wget -q "https://github.com/n8n-io/task-runner-launcher/releases/download/${LAUNCHER_VERSION}/task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz"; \
    wget -q "https://github.com/n8n-io/task-runner-launcher/releases/download/${LAUNCHER_VERSION}/task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz.sha256"; \
    echo "$(cat task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz.sha256) task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz" > checksum.sha256; \
    sha256sum -c checksum.sha256; \
    mkdir -p /launcher-bin; \
    tar xzf task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz -C /launcher-bin; \
    cd / && rm -rf /launcher-temp

# ==============================================================================
# STAGE 4: Node alpine base for JS task runner
# ==============================================================================
FROM node:${NODE_VERSION}-alpine AS node-alpine

# ==============================================================================
# STAGE 5: Runtime
# ==============================================================================
FROM python:${PYTHON_VERSION}-alpine AS runtime
ARG N8N_VERSION=snapshot
ARG N8N_RELEASE_TYPE=dev

ENV NODE_ENV=production \
    N8N_RELEASE_TYPE=${N8N_RELEASE_TYPE} \
    SHELL=/bin/sh

# Bring `uv` over from python-runner-builder, to make the image easier to extend
COPY --from=python-runner-builder /usr/local/bin/uv /usr/local/bin/uv

# Bring node over from node-alpine
COPY --from=node-alpine /usr/local/bin/node /usr/local/bin/node

# libstdc++ is required by Node
# libc6-compat is required by task-runner-launcher
RUN apk add --no-cache ca-certificates tini libstdc++ libc6-compat && \
    apk del apk-tools

# Bring corepack and pnpm over, to make the image easier to extend
COPY --from=node-alpine /usr/local/lib/node_modules/corepack /usr/local/lib/node_modules/corepack
RUN ln -s ../lib/node_modules/corepack/dist/corepack.js /usr/local/bin/corepack && \
                ln -s ../lib/node_modules/corepack/dist/pnpm.js /usr/local/bin/pnpm

RUN addgroup -g 1000 -S runner \
 && adduser  -u 1000 -S -G runner -h /home/runner -D runner

WORKDIR /home/runner

COPY --from=javascript-runner-builder --chown=root:root /app/task-runner-javascript /opt/runners/task-runner-javascript
COPY --from=python-runner-builder --chown=root:root /app/task-runner-python /opt/runners/task-runner-python
COPY --from=launcher-downloader /launcher-bin/* /usr/local/bin/
COPY --chown=root:root docker/images/runners/n8n-task-runners.json /etc/n8n-task-runners.json

USER runner

EXPOSE 5680/tcp
ENTRYPOINT ["tini", "--", "/usr/local/bin/task-runner-launcher"]
CMD ["javascript", "python"]

LABEL org.opencontainers.image.title="n8n task runners" \
      org.opencontainers.image.description="Sidecar image providing n8n task runners for JavaScript and Python code execution" \
      org.opencontainers.image.source="https://github.com/n8n-io/n8n" \
      org.opencontainers.image.url="https://n8n.io" \
      org.opencontainers.image.version="${N8N_VERSION}"ARG NODE_VERSION=24.13.1
ARG PYTHON_VERSION=3.13

# ==============================================================================
# STAGE 1: JavaScript runner (@n8n/task-runner) artifact from CI
# ==============================================================================
FROM node:${NODE_VERSION}-alpine AS javascript-runner-builder
COPY ./dist/task-runner-javascript /app/task-runner-javascript

WORKDIR /app/task-runner-javascript

RUN corepack enable pnpm

# Remove `catalog` and `workspace` references from package.json to allow `pnpm add` in extended images
RUN node -e "const pkg = require('./package.json'); \
    Object.keys(pkg.dependencies || {}).forEach(k => { \
        const val = pkg.dependencies[k]; \
        if (val === 'catalog:' || val.startsWith('catalog:') || val.startsWith('workspace:')) \
            delete pkg.dependencies[k]; \
    }); \
    Object.keys(pkg.devDependencies || {}).forEach(k => { \
        const val = pkg.devDependencies[k]; \
        if (val === 'catalog:' || val.startsWith('catalog:') || val.startsWith('workspace:')) \
            delete pkg.devDependencies[k]; \
    }); \
    delete pkg.devDependencies; \
    require('fs').writeFileSync('./package.json', JSON.stringify(pkg, null, 2));"

# Install moment (special case for backwards compatibility)
RUN rm -f node_modules/.modules.yaml && \
    pnpm add moment@2.30.1 --prod --no-lockfile

# ==============================================================================
# STAGE 2: Python runner build (@n8n/task-runner-python) with uv
# Produces a relocatable venv tied to the python version used
# ==============================================================================
FROM python:${PYTHON_VERSION}-alpine AS python-runner-builder
ARG TARGETPLATFORM
ARG UV_VERSION=0.8.14

RUN set -e; \
  case "$TARGETPLATFORM" in \
    "linux/amd64") UV_ARCH="x86_64-unknown-linux-musl" ;; \
    "linux/arm64") UV_ARCH="aarch64-unknown-linux-musl" ;; \
    *) echo "Unsupported platform: $TARGETPLATFORM" >&2; exit 1 ;; \
  esac; \
  mkdir -p /tmp/uv && cd /tmp/uv; \
  wget -q "https://github.com/astral-sh/uv/releases/download/${UV_VERSION}/uv-${UV_ARCH}.tar.gz"; \
  wget -q "https://github.com/astral-sh/uv/releases/download/${UV_VERSION}/uv-${UV_ARCH}.tar.gz.sha256"; \
  sha256sum -c "uv-${UV_ARCH}.tar.gz.sha256"; \
        tar -xzf "uv-${UV_ARCH}.tar.gz"; \
  install -m 0755 "uv-${UV_ARCH}/uv" /usr/local/bin/uv; \
  cd / && rm -rf /tmp/uv

WORKDIR /app/task-runner-python

COPY packages/@n8n/task-runner-python/pyproject.toml \
     packages/@n8n/task-runner-python/uv.lock** \
     packages/@n8n/task-runner-python/.python-version** \
     ./

RUN uv venv
RUN uv sync \
                        --frozen \
                        --no-editable \
                        --no-install-project \
                        --no-dev \
                        --all-extras

COPY packages/@n8n/task-runner-python/ ./
RUN uv sync \
      --frozen \
                        --no-dev \
                        --all-extras \
      --no-editable

# Install the python runner package itself into site packages. We can remove the src directory then
RUN uv pip install . && rm -rf /app/task-runner-python/src

# ==============================================================================
# STAGE 3: Task Runner Launcher download
# ==============================================================================
FROM alpine:3.22 AS launcher-downloader
ARG TARGETPLATFORM
ARG LAUNCHER_VERSION=1.4.3

RUN set -e; \
    case "$TARGETPLATFORM" in \
        "linux/amd64") ARCH_NAME="amd64" ;; \
        "linux/arm64") ARCH_NAME="arm64" ;; \
        *) echo "Unsupported platform: $TARGETPLATFORM" && exit 1 ;; \
    esac; \
    mkdir /launcher-temp && cd /launcher-temp; \
    wget -q "https://github.com/n8n-io/task-runner-launcher/releases/download/${LAUNCHER_VERSION}/task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz"; \
    wget -q "https://github.com/n8n-io/task-runner-launcher/releases/download/${LAUNCHER_VERSION}/task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz.sha256"; \
    echo "$(cat task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz.sha256) task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz" > checksum.sha256; \
    sha256sum -c checksum.sha256; \
    mkdir -p /launcher-bin; \
    tar xzf task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz -C /launcher-bin; \
    cd / && rm -rf /launcher-temp

# ==============================================================================
# STAGE 4: Node alpine base for JS task runner
# ==============================================================================
FROM node:${NODE_VERSION}-alpine AS node-alpine

# ==============================================================================
# STAGE 5: Runtime
# ==============================================================================
FROM python:${PYTHON_VERSION}-alpine AS runtime
ARG N8N_VERSION=snapshot
ARG N8N_RELEASE_TYPE=dev

ENV NODE_ENV=production \
    N8N_RELEASE_TYPE=${N8N_RELEASE_TYPE} \
    SHELL=/bin/sh

# Bring `uv` over from python-runner-builder, to make the image easier to extend
COPY --from=python-runner-builder /usr/local/bin/uv /usr/local/bin/uv

# Bring node over from node-alpine
COPY --from=node-alpine /usr/local/bin/node /usr/local/bin/node

# libstdc++ is required by Node
# libc6-compat is required by task-runner-launcher
RUN apk add --no-cache ca-certificates tini libstdc++ libc6-compat && \
    apk del apk-tools

# Bring corepack and pnpm over, to make the image easier to extend
COPY --from=node-alpine /usr/local/lib/node_modules/corepack /usr/local/lib/node_modules/corepack
RUN ln -s ../lib/node_modules/corepack/dist/corepack.js /usr/local/bin/corepack && \
                ln -s ../lib/node_modules/corepack/dist/pnpm.js /usr/local/bin/pnpm

RUN addgroup -g 1000 -S runner \
 && adduser  -u 1000 -S -G runner -h /home/runner -D runner

WORKDIR /home/runner

COPY --from=javascript-runner-builder --chown=root:root /app/task-runner-javascript /opt/runners/task-runner-javascript
COPY --from=python-runner-builder --chown=root:root /app/task-runner-python /opt/runners/task-runner-python
COPY --from=launcher-downloader /launcher-bin/* /usr/local/bin/
COPY --chown=root:root docker/images/runners/n8n-task-runners.json /etc/n8n-task-runners.json

USER runner

EXPOSE 5680/tcp
ENTRYPOINT ["tini", "--", "/usr/local/bin/task-runner-launcher"]
CMD ["javascript", "python"]

LABEL org.opencontainers.image.title="n8n task runners" \
      org.opencontainers.image.description="Sidecar image providing n8n task runners for JavaScript and Python code execution" \
      org.opencontainers.image.source="https://github.com/n8n-io/n8n" \
      org.opencontainers.image.url="https://n8n.io" \
      org.opencontainers.image.version="${N8N_VERSION}"ARG NODE_VERSION=24.13.1
ARG PYTHON_VERSION=3.13

# ==============================================================================
# STAGE 1: JavaScript runner (@n8n/task-runner) artifact from CI
# ==============================================================================
FROM node:${NODE_VERSION}-alpine AS javascript-runner-builder
COPY ./dist/task-runner-javascript /app/task-runner-javascript

WORKDIR /app/task-runner-javascript

RUN corepack enable pnpm

# Remove `catalog` and `workspace` references from package.json to allow `pnpm add` in extended images
RUN node -e "const pkg = require('./package.json'); \
    Object.keys(pkg.dependencies || {}).forEach(k => { \
        const val = pkg.dependencies[k]; \
        if (val === 'catalog:' || val.startsWith('catalog:') || val.startsWith('workspace:')) \
            delete pkg.dependencies[k]; \
    }); \
    Object.keys(pkg.devDependencies || {}).forEach(k => { \
        const val = pkg.devDependencies[k]; \
        if (val === 'catalog:' || val.startsWith('catalog:') || val.startsWith('workspace:')) \
            delete pkg.devDependencies[k]; \
    }); \
    delete pkg.devDependencies; \
    require('fs').writeFileSync('./package.json', JSON.stringify(pkg, null, 2));"

# Install moment (special case for backwards compatibility)
RUN rm -f node_modules/.modules.yaml && \
    pnpm add moment@2.30.1 --prod --no-lockfile

# ==============================================================================
# STAGE 2: Python runner build (@n8n/task-runner-python) with uv
# Produces a relocatable venv tied to the python version used
# ==============================================================================
FROM python:${PYTHON_VERSION}-alpine AS python-runner-builder
ARG TARGETPLATFORM
ARG UV_VERSION=0.8.14

RUN set -e; \
  case "$TARGETPLATFORM" in \
    "linux/amd64") UV_ARCH="x86_64-unknown-linux-musl" ;; \
    "linux/arm64") UV_ARCH="aarch64-unknown-linux-musl" ;; \
    *) echo "Unsupported platform: $TARGETPLATFORM" >&2; exit 1 ;; \
  esac; \
  mkdir -p /tmp/uv && cd /tmp/uv; \
  wget -q "https://github.com/astral-sh/uv/releases/download/${UV_VERSION}/uv-${UV_ARCH}.tar.gz"; \
  wget -q "https://github.com/astral-sh/uv/releases/download/${UV_VERSION}/uv-${UV_ARCH}.tar.gz.sha256"; \
  sha256sum -c "uv-${UV_ARCH}.tar.gz.sha256"; \
        tar -xzf "uv-${UV_ARCH}.tar.gz"; \
  install -m 0755 "uv-${UV_ARCH}/uv" /usr/local/bin/uv; \
  cd / && rm -rf /tmp/uv

WORKDIR /app/task-runner-python

COPY packages/@n8n/task-runner-python/pyproject.toml \
     packages/@n8n/task-runner-python/uv.lock** \
     packages/@n8n/task-runner-python/.python-version** \
     ./

RUN uv venv
RUN uv sync \
                        --frozen \
                        --no-editable \
                        --no-install-project \
                        --no-dev \
                        --all-extras

COPY packages/@n8n/task-runner-python/ ./
RUN uv sync \
      --frozen \
                        --no-dev \
                        --all-extras \
      --no-editable

# Install the python runner package itself into site packages. We can remove the src directory then
RUN uv pip install . && rm -rf /app/task-runner-python/src

# ==============================================================================
# STAGE 3: Task Runner Launcher download
# ==============================================================================
FROM alpine:3.22 AS launcher-downloader
ARG TARGETPLATFORM
ARG LAUNCHER_VERSION=1.4.3

RUN set -e; \
    case "$TARGETPLATFORM" in \
        "linux/amd64") ARCH_NAME="amd64" ;; \
        "linux/arm64") ARCH_NAME="arm64" ;; \
        *) echo "Unsupported platform: $TARGETPLATFORM" && exit 1 ;; \
    esac; \
    mkdir /launcher-temp && cd /launcher-temp; \
    wget -q "https://github.com/n8n-io/task-runner-launcher/releases/download/${LAUNCHER_VERSION}/task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz"; \
    wget -q "https://github.com/n8n-io/task-runner-launcher/releases/download/${LAUNCHER_VERSION}/task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz.sha256"; \
    echo "$(cat task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz.sha256) task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz" > checksum.sha256; \
    sha256sum -c checksum.sha256; \
    mkdir -p /launcher-bin; \
    tar xzf task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz -C /launcher-bin; \
    cd / && rm -rf /launcher-temp

# ==============================================================================
# STAGE 4: Node alpine base for JS task runner
# ==============================================================================
FROM node:${NODE_VERSION}-alpine AS node-alpine

# ==============================================================================
# STAGE 5: Runtime
# ==============================================================================
FROM python:${PYTHON_VERSION}-alpine AS runtime
ARG N8N_VERSION=snapshot
ARG N8N_RELEASE_TYPE=dev

ENV NODE_ENV=production \
    N8N_RELEASE_TYPE=${N8N_RELEASE_TYPE} \
    SHELL=/bin/sh

# Bring `uv` over from python-runner-builder, to make the image easier to extend
COPY --from=python-runner-builder /usr/local/bin/uv /usr/local/bin/uv

# Bring node over from node-alpine
COPY --from=node-alpine /usr/local/bin/node /usr/local/bin/node

# libstdc++ is required by Node
# libc6-compat is required by task-runner-launcher
RUN apk add --no-cache ca-certificates tini libstdc++ libc6-compat && \
    apk del apk-tools

# Bring corepack and pnpm over, to make the image easier to extend
COPY --from=node-alpine /usr/local/lib/node_modules/corepack /usr/local/lib/node_modules/corepack
RUN ln -s ../lib/node_modules/corepack/dist/corepack.js /usr/local/bin/corepack && \
                ln -s ../lib/node_modules/corepack/dist/pnpm.js /usr/local/bin/pnpm

RUN addgroup -g 1000 -S runner \
 && adduser  -u 1000 -S -G runner -h /home/runner -D runner

WORKDIR /home/runner

COPY --from=javascript-runner-builder --chown=root:root /app/task-runner-javascript /opt/runners/task-runner-javascript
COPY --from=python-runner-builder --chown=root:root /app/task-runner-python /opt/runners/task-runner-python
COPY --from=launcher-downloader /launcher-bin/* /usr/local/bin/
COPY --chown=root:root docker/images/runners/n8n-task-runners.json /etc/n8n-task-runners.json

USER runner

EXPOSE 5680/tcp
ENTRYPOINT ["tini", "--", "/usr/local/bin/task-runner-launcher"]
CMD ["javascript", "python"]

LABEL org.opencontainers.image.title="n8n task runners" \
      org.opencontainers.image.description="Sidecar image providing n8n task runners for JavaScript and Python code execution" \
      org.opencontainers.image.source="https://github.com/n8n-io/n8n" \
      org.opencontainers.image.url="https://n8n.io" \
      org.opencontainers.image.version="${N8N_VERSION}"{
        "task-runners": [
                {
                        "runner-type": "javascript",
                        "workdir": "/home/runner",
                        "command": "/usr/local/bin/node",
                        "args": [
                                "--disallow-code-generation-from-strings",
                                "--disable-proto=delete",
                                "/opt/runners/task-runner-javascript/dist/start.js"
                        ],
                        "health-check-server-port": "5681",
                        "allowed-env": [
                                "PATH",
                                "GENERIC_TIMEZONE",
                                "NODE_OPTIONS",
                                "N8N_RUNNERS_AUTO_SHUTDOWN_TIMEOUT",
                                "N8N_RUNNERS_TASK_TIMEOUT",
                                "N8N_RUNNERS_MAX_CONCURRENCY",
                                "N8N_SENTRY_DSN",
                                "N8N_VERSION",
                                "ENVIRONMENT",
                                "DEPLOYMENT_NAME",
                                "HOME"
                        ],
                        "env-overrides": {
                                "NODE_FUNCTION_ALLOW_BUILTIN": "crypto",
                                "NODE_FUNCTION_ALLOW_EXTERNAL": "moment",
                                "N8N_RUNNERS_HEALTH_CHECK_SERVER_HOST": "0.0.0.0"
                        }
                },
                {
                        "runner-type": "python",
                        "workdir": "/home/runner",
                        "command": "/opt/runners/task-runner-python/.venv/bin/python",
                        "args": ["-I", "-B", "-X", "disable_remote_debug", "-m", "src.main"],
                        "health-check-server-port": "5682",
                        "allowed-env": [
                                "PATH",
                                "N8N_RUNNERS_LAUNCHER_LOG_LEVEL",
                                "N8N_RUNNERS_AUTO_SHUTDOWN_TIMEOUT",
                                "N8N_RUNNERS_TASK_TIMEOUT",
                                "N8N_RUNNERS_MAX_CONCURRENCY",
                                "N8N_SENTRY_DSN",
                                "N8N_VERSION",
                                "ENVIRONMENT",
                                "DEPLOYMENT_NAME"
                        ],
                        "env-overrides": {
                                "N8N_RUNNERS_STDLIB_ALLOW": "",
                                "N8N_RUNNERS_EXTERNAL_ALLOW": ""
                        }
                }
        ]
}n8n.io - Workflow Automation
n8n - Secure Workflow Automation for Technical Teams
n8n is a workflow automation platform that gives technical teams the flexibility of code with the speed of no-code. With 400+ integrations, native AI capabilities, and a fair-code license, n8n lets you build powerful automations while maintaining full control over your data and deployments.

n8n.io - Screenshot

Key Capabilities
Code When You Need It: Write JavaScript/Python, add npm packages, or use the visual interface
AI-Native Platform: Build AI agent workflows based on LangChain with your own data and models
Full Control: Self-host with our fair-code license or use our cloud offering
Enterprise-Ready: Advanced permissions, SSO, and air-gapped deployments
Active Community: 400+ integrations and 900+ ready-to-use templates
Contents
n8n - Workflow automation tool
Key Capabilities
Contents
Demo
Available integrations
Documentation
Start n8n in Docker
Use with PostgreSQL
Passing sensitive data using files
Example server setups
Updating
Pull latest (stable) version
Pull specific version
Pull next (unstable) version
Updating with Docker Compose
Setting Timezone
Build Docker-Image
What does n8n mean and how do you pronounce it?
Support
Jobs
License
Demo
This 📺 short video (< 4 min) goes over key concepts of creating workflows in n8n.

Available integrations
n8n has 200+ different nodes to automate workflows. A full list can be found at https://n8n.io/integrations.

Documentation
The official n8n documentation can be found at https://docs.n8n.io.

Additional information and example workflows are available on the website at https://n8n.io.

Start n8n in Docker
In the terminal, enter the following:

docker volume create n8n_data

docker run -it --rm \
 --name n8n \
 -p 5678:5678 \
 -v n8n_data:/home/node/.n8n \
 docker.n8n.io/n8nio/n8n
This command will download the required n8n image and start your container. You can then access n8n by opening: http://localhost:5678

To save your work between container restarts, it also mounts a docker volume, n8n_data. The workflow data gets saved in an SQLite database in the user folder (/home/node/.n8n). This folder also contains important data like the webhook URL and the encryption key used for securing credentials.

If this data can't be found at startup n8n automatically creates a new key and any existing credentials can no longer be decrypted.

Use with PostgreSQL
By default, n8n uses SQLite to save credentials, past executions and workflows. However, n8n also supports using PostgreSQL.

WARNING: Even when using a different database, it is still important to persist the /home/node/.n8n folder, which also contains essential n8n user data including the encryption key for the credentials.

In the following commands, replace the placeholders (depicted within angled brackets, e.g. <POSTGRES_USER>) with the actual data:

docker volume create n8n_data

docker run -it --rm \
 --name n8n \
 -p 5678:5678 \
 -e DB_TYPE=postgresdb \
 -e DB_POSTGRESDB_DATABASE=<POSTGRES_DATABASE> \
 -e DB_POSTGRESDB_HOST=<POSTGRES_HOST> \
 -e DB_POSTGRESDB_PORT=<POSTGRES_PORT> \
 -e DB_POSTGRESDB_USER=<POSTGRES_USER> \
 -e DB_POSTGRESDB_SCHEMA=<POSTGRES_SCHEMA> \
 -e DB_POSTGRESDB_PASSWORD=<POSTGRES_PASSWORD> \
 -v n8n_data:/home/node/.n8n \
 docker.n8n.io/n8nio/n8n
A full working setup with docker-compose can be found here.

Passing sensitive data using files
To avoid passing sensitive information via environment variables, "_FILE" may be appended to some environment variable names. n8n will then load the data from a file with the given name. This makes it possible to load data easily from Docker and Kubernetes secrets.

The following environment variables support file input:

DB_POSTGRESDB_DATABASE_FILE
DB_POSTGRESDB_HOST_FILE
DB_POSTGRESDB_PASSWORD_FILE
DB_POSTGRESDB_PORT_FILE
DB_POSTGRESDB_USER_FILE
DB_POSTGRESDB_SCHEMA_FILE
Example server setups
Example server setups for a range of cloud providers and scenarios can be found in the Server Setup documentation.

Updating
Before you upgrade to the latest version make sure to check here if there are any breaking changes which may affect you: Breaking Changes

From your Docker Desktop, navigate to the Images tab and select Pull from the context menu to download the latest n8n image.

You can also use the command line to pull the latest, or a specific version:

Pull latest (stable) version
docker pull docker.n8n.io/n8nio/n8n
Pull specific version
docker pull docker.n8n.io/n8nio/n8n:0.220.1
Pull next (unstable) version
docker pull docker.n8n.io/n8nio/n8n:next
Stop the container and start it again:

Get the container ID:
docker ps -a
Stop the container with ID container_id:
docker stop [container_id]
Remove the container (this does not remove your user data) with ID container_id:
docker rm [container_id]
Start the new container:
docker run --name=[container_name] [options] -d docker.n8n.io/n8nio/n8n
Updating with Docker Compose
If you run n8n using a Docker Compose file, follow these steps to update n8n:

# Pull latest version
docker compose pull

# Stop and remove older version
docker compose down

# Start the container
docker compose up -d
Setting the timezone
To specify the timezone n8n should use, the environment variable GENERIC_TIMEZONE can be set. One example where this variable has an effect is the Schedule node.

The system's timezone can be set separately with the environment variable TZ. This controls the output of certain scripts and commands such as $ date.

For example, to use the same timezone for both:

docker run -it --rm \
 --name n8n \
 -p 5678:5678 \
 -e GENERIC_TIMEZONE="Europe/Berlin" \
 -e TZ="Europe/Berlin" \
 docker.n8n.io/n8nio/n8n
For more information on configuration and environment variables, please see the n8n documentation.

Here's the refined version with good Markdown formatting, ready for your README:

Build Docker Image
Important Note for Releases 1.101.0 and Later: Building the n8n Docker image now requires a pre-compiled n8n application.

Recommended Build Process:
For the simplest approach that handles both n8n compilation and Docker image creation, run from the root directory:

pnpm build:docker
Alternative Builders:
If you are using a different build system that requires a separate build context, first compile the n8n application:

pnpm run build:deploy
Then, ensure your builder's context includes the compiled directory generated by this command.

What does n8n mean and how do you pronounce it?
Short answer: It means "nodemation" and it is pronounced as n-eight-n.

Long answer: I get that question quite often (more often than I expected) so I decided it is probably best to answer it here. While looking for a good name for the project with a free domain I realized very quickly that all the good ones I could think of were already taken. So, in the end, I chose nodemation. "node-" in the sense that it uses a Node-View and that it uses Node.js and "-mation" for "automation" which is what the project is supposed to help with. However, I did not like how long the name was and I could not imagine writing something that long every time in the CLI. That is when I then ended up on "n8n". Sure it does not work perfectly but neither does it for Kubernetes (k8s) and I did not hear anybody complain there. So I guess it should be ok.

Support
If you need more help with n8n, you can ask for support in the n8n community forum. This is the best source of answers, as both the n8n support team and community members can help.

Jobs
If you are interested in working for n8n and so shape the future of the project check out our job posts.

License
You can find the license information here.