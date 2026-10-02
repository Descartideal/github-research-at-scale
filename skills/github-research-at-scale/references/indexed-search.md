# Indexed cross-repository search

Use this reference for Sourcegraph-style search. Check the connected server's schema and deployed syntax before running queries. A private instance may index only internal repositories; a public instance may not index the requested projects. A browser search page does not imply an available MCP endpoint.

## Query examples

Find a literal API call across TypeScript repositories:

```text
context:global patterntype:literal registerTool( lang:typescript select:repo count:100
```

Find documentation mentions:

```text
context:global patterntype:literal "agent team" file:README select:repo count:100
```

Inspect one identified repository:

```text
repo:^github\.com/OWNER/REPO$ patterntype:literal registerTool( lang:typescript count:50
```

Replace OWNER/REPO with an actual repository and regex-escape its name. `repo:` and `file:` use backend-specific matching rules; do not pass arbitrary names into regex unchanged. `select:repo` finds identities; remove it when you need matching file evidence.

Begin with bounded result counts. Inspect backend completion flags, warnings, timeouts, and repository coverage. Increasing `count:` or using `count:all` does not guarantee exhaustive results and may materially increase backend work. When needed, partition by supported filters and retain each partition's completion state. Do not send enormous owner/name alternations that exceed query limits.

Use a semantic search tool for conceptual discovery only if the server offers one. Confirm shortlisted results with concrete files and versions. If results are too broad, add discriminating terms or file/language filters; if too narrow, relax one filter at a time. Avoid reissuing unchanged searches unless freshness or an earlier incomplete response justifies it.

Official syntax reference: [Sourcegraph search queries](https://sourcegraph.com/docs/code-search/queries). Verify deployment-specific availability; this skill provides no coverage guarantee.
