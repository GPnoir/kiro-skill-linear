# Linear API Notes

## Endpoint

```
POST https://api.linear.app/graphql
```

## Authentication

Personal API key (no Bearer prefix):
```
Authorization: <API_KEY>
```

Generate at: Settings → Account → Security & Access → Personal API Keys

## curl Template

```bash
curl -s -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: $LINEAR_API_KEY" \
  --data '{"query": "<GRAPHQL_QUERY>", "variables": {}}' \
  https://api.linear.app/graphql | jq '.'
```

With variables:
```bash
curl -s -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: $LINEAR_API_KEY" \
  --data '{
    "query": "query($teamId: String!) { team(id: $teamId) { name } }",
    "variables": {"teamId": "YOUR_TEAM_ID"}
  }' \
  https://api.linear.app/graphql | jq '.'
```

## Priority Values

| Value | Label |
|-------|-------|
| 0 | No priority |
| 1 | Urgent |
| 2 | High |
| 3 | Medium |
| 4 | Low |

## Workflow State Types

| Type | Description |
|------|-------------|
| `backlog` | Not yet planned |
| `unstarted` | Planned but not started (Todo) |
| `started` | In progress |
| `completed` | Done |
| `canceled` | Won't do |

## Issue Identifiers

Issues can be referenced by:
- UUID: `"590a1127-f98b-49fc-ba74-2df8751c089e"`
- Identifier: `"APP-123"` (team key + number)

Both work in queries and mutations.

## Pagination

Uses cursor-based pagination:
```graphql
issues(first: 50, after: "cursor_value") {
  nodes { ... }
  pageInfo {
    hasNextPage
    endCursor
  }
}
```

Default: 50 items. Max per request: 250.

## Filtering

### Comparators (all fields)
- `eq`, `neq` — equals / not equals
- `in`, `nin` — in / not in array

### Numeric/date additional
- `lt`, `lte`, `gt`, `gte`

### String additional
- `contains`, `notContains`, `containsIgnoreCase`
- `startsWith`, `notStartsWith`
- `endsWith`, `notEndsWith`
- `eqIgnoreCase`, `neqIgnoreCase`

### Null check
- `null: true` / `null: false`

### Logical operators
- Default: all conditions AND
- Use `or: [...]` for OR logic

### Relationship filters
```graphql
# By assignee email
issues(filter: { assignee: { email: { eq: "user@appstride.com" } } })

# By label name
issues(filter: { labels: { name: { eq: "Bug" } } })

# By state type
issues(filter: { state: { type: { eq: "started" } } })
```

### Relative time (ISO 8601 durations)
```graphql
# Due in next 2 weeks
issues(filter: { dueDate: { lt: "P2W" } })

# Completed in last 2 weeks
issues(filter: { completedAt: { gt: "-P2W" } })
```

## Attachments (GitHub linking)

Link a GitHub PR to a Linear issue:
```graphql
mutation {
  attachmentCreate(input: {
    issueId: "ISSUE_UUID"
    title: "PR #42: Fix login bug"
    subtitle: "Open"
    url: "https://github.com/appstride/repo/pull/42"
    iconUrl: "https://github.githubassets.com/favicons/favicon-dark.png"
  }) {
    success
    attachment { id }
  }
}
```

Key behaviors:
- Same URL + same issue = update (idempotent)
- Can query by URL: `attachmentsForURL(url: "...")`
- Supports rich metadata with `messages` and `attributes`
- Date formatting in subtitles: `"Opened {createdAt__since}"` → "Opened 2 days ago"

## Mentions in Markdown

Use plain Linear URLs to create mentions:
```markdown
https://linear.app/appstride/profiles/username
https://linear.app/appstride/issue/APP-123/issue-title
```

These render as @mentions inside Linear.

## Rate Limiting

- 1,500 requests per hour per API key
- Complexity-based: simple queries cost less
- Check `X-RateLimit-Remaining` header
- If rate limited, wait and retry with exponential backoff

## Webhooks

Supported events: Issues, Comments, Attachments, Projects, Cycles, Labels, Users.
Register via: Settings → API → Webhooks (or programmatically via mutation).

## Error Handling

Always check response for errors:
```bash
response=$(curl -s ...)
errors=$(echo "$response" | jq '.errors')
if [ "$errors" != "null" ]; then
  echo "Error: $(echo "$errors" | jq -r '.[0].message')"
fi
```

GraphQL can return partial success (200 status with both `data` and `errors`).

## Ordering

Most collections support `orderBy`:
- `createdAt` (default)
- `updatedAt`
- `priority`

## Archived Resources

Hidden by default. Include with: `includeArchived: true`

## Collapsible Sections (issue descriptions)

```markdown
+++ Section title
Hidden content here
+++
```

## Team Context

- Workspace: AppStride
- Team: "Equipo AppStride"
- Resolve team ID once with `Teams` query, then cache for session
