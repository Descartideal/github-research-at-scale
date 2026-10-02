# GitHub collection and quota handling

## Repository discovery

REST search exposes at most 1,000 results for a query and up to 100 items per page. These limits describe that endpoint, not every connector. Use returned pagination links or cursors and retain `total_count` and `incomplete_results` when available. Never equate total matches with retrieved repositories.

For an explicitly requested broad collection, partition repository queries into non-overlapping supported ranges, such as `created:` dates. A daily interval exceeding the cap needs a second dimension such as a disjoint `stars:` range. Do not recurse forever on an unsplittable interval. Respect a finite query budget, deduplicate IDs across shards, and record unresolved or incomplete shards.

Example date partitions:

```text
agent language:typescript created:2026-01-01..2026-06-30
agent language:typescript created:2026-07-01..2026-12-31
```

These cover only that explicit year; they cannot support a claim about all repositories. Account for repositories outside the chosen bounds. Search results can change during pagination, especially when partitioning by mutable stars or activity. Partitioning reduces truncation but does not guarantee a consistent complete snapshot.

Persist fields already returned, such as identity, description, language, stars, topics, and dates, when available. A connector may expose only a subset. Treat `updated_at` and `pushed_at` according to their meanings; neither alone proves active maintenance. Unknown or metadata-declared licenses may require file verification on the shortlist.

## Rate limits and failures

Use live response headers or connector diagnostics rather than hardcoded quotas. Core, search, code search, and GraphQL have different accounting, and secondary throttling can occur before the primary budget is depleted.

- For a confirmed rate limit, respect `Retry-After`; if primary remaining is zero, wait until the reported reset time. Without those signals, use bounded backoff appropriate to the provider response.
- Do not interpret every 403 as throttling: inspect whether the cause is permission, authentication, policy, or a confirmed rate limit. Fix a query error rather than retrying it unchanged.
- Stop repeated failures after a small bounded retry budget. Continue filtering cached data or reading existing clones; report the blocked source. Do not create a retry storm or rotate identities to evade limits.
- Batch missing metadata when an available API supports it, respecting its own cost and size constraints. One GraphQL request is not automatically cheap and does not replace cross-repository code search.

## Caches and gateway boundaries

Choose freshness according to the question. Cache by endpoint/backend, parameters, authentication scope, and revision where relevant. Keep tokens out of cache keys, artifacts, and reports; isolate private data from public outputs. Revalidate stale data where freshness affects the conclusion.

Where supported, save ETags and use conditional requests. Correctly authenticated requests returning 304 can save primary quota; they still create network traffic and are not a cure for secondary limits. If implementing a gateway separately, centralize request budgets and identical in-flight request coalescing. This skill itself does not implement that infrastructure.

## Primary references

- [GitHub REST search](https://docs.github.com/en/rest/search/search)
- [REST API rate limits](https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api)
- [REST API best practices](https://docs.github.com/en/rest/using-the-rest-api/best-practices-for-using-the-rest-api)

Consult the current documentation for numeric quotas and endpoint-specific behavior rather than treating remembered values as configuration.
