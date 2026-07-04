---
name: linear
description: "Trigger: Linear issues, tickets, sprints, cycles, projects, backlog, AppStride team. Manage Linear.app issues, cycles, and projects via GraphQL API."
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

# Skill: Linear

Manage Linear.app issues, cycles, projects, and team workflows for the AppStride team via the GraphQL API.

## Activation Contract

Activate when:
- User mentions Linear issues, tickets, sprints, cycles, or backlog
- User asks to create, update, list, assign, or comment on issues
- User references the AppStride team project management
- User asks for a sprint/cycle summary or project progress

## Hard Rules

- **Always preview** before executing mutations (create, update, delete). Show what will be sent and ask confirmation.
- Use `$LINEAR_API_KEY` from environment. Never hardcode tokens.
- Team: "Equipo AppStride" — resolve team ID on first use and cache it.
- All API calls go to `https://api.linear.app/graphql` via POST.
- Auth header: `Authorization: $LINEAR_API_KEY` (no Bearer prefix for personal API keys).
- Use queries from `assets/queries.graphql` and mutations from `assets/mutations.graphql`.
- Parse responses with `jq`. Always check for `.errors` in response before presenting data.
- When listing issues, default to showing: identifier, title, state, priority, assignee.
- Priority values: 0=No priority, 1=Urgent, 2=High, 3=Medium, 4=Low.

## Decision Gates

| Need | Action |
|------|--------|
| List/read issues, cycles, projects | Use queries from `assets/queries.graphql` |
| Create/update/delete anything | Preview first, then use `assets/mutations.graphql` |
| Filter issues by state, assignee, label | Use GraphQL filter syntax (see `references/api-notes.md`) |
| Link to GitHub PR/branch | Use `attachmentCreate` mutation with the GitHub URL |
| Get team summary/standup | Combine active cycle issues + in-progress states |

## Execution Steps

1. Source the environment: verify `$LINEAR_API_KEY` is set.
2. Resolve the AppStride team ID (cache after first call).
3. Build the GraphQL query/mutation from the assets templates.
4. For reads: execute immediately, format output as a table.
5. For writes: show preview of what will be created/changed, wait for confirmation, then execute.
6. Parse response, check for errors, present results.

## Available Operations

### Queries (Read)
| Operation | Description |
|-----------|-------------|
| `viewer` | Current authenticated user info |
| `teams` | List all teams (resolve AppStride team ID) |
| `team_issues` | List issues for the team with filters |
| `my_issues` | Issues assigned to the current user |
| `issue_detail` | Full detail of a single issue by ID/identifier |
| `cycles` | List team cycles (sprints) |
| `active_cycle` | Current active cycle with its issues |
| `projects` | List team projects with progress |
| `project_issues` | Issues within a specific project |
| `labels` | List available labels |
| `workflow_states` | List team workflow states (statuses) |
| `users` | List team members |
| `search_issues` | Search issues by text |
| `comments` | Get comments on an issue |

### Mutations (Write — always preview first)
| Operation | Description |
|-----------|-------------|
| `issue_create` | Create a new issue |
| `issue_update` | Update issue (title, state, assignee, priority, labels) |
| `issue_delete` | Archive/delete an issue |
| `comment_create` | Add a comment to an issue |
| `issue_assign` | Assign an issue to a team member |
| `label_create` | Create a new label |
| `cycle_create` | Create a new cycle |
| `attachment_create` | Link external resource (GitHub PR/branch) to an issue |
| `sub_issue_create` | Create a sub-issue under a parent |

## Output Contract

- Tables for lists (identifier | title | state | priority | assignee)
- Structured detail for single items
- Preview block before any mutation
- Confirmation message after successful mutation with the created/updated entity URL

## References

- `assets/queries.graphql` — All read queries
- `assets/mutations.graphql` — All write mutations
- `references/api-notes.md` — API filtering, pagination, rate limits, and conventions
