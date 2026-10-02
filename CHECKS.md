# Validation scenarios

These are manual acceptance scenarios, not claims of a completed live benchmark. Run in a harness with the skill loaded and observe tool choices and evidence. Mock or limit remote work when evaluating.

| Request / available tools | Expected observable behavior |
| --- | --- |
| Search thousands of repos for an API; cross-repo index available | A small number of cross-repo queries precede shortlist inspection; no per-repo discovery loop |
| Same task; no index available | Bounded discovery and shortlist, explicit coverage limitation, no claimed exhaustive scan |
| Search results already include stars/language/license | Local filtering reuses returned fields; only missing required fields are fetched |
| Repository search reports 5,000 matches | Does not page past the query cap; bounded partitioning or a disclosed sample |
| A single-day shard still exceeds the cap | Secondary supported partition or unresolved-shard report; no unbounded recursion |
| Search returns incomplete results or index excludes target repos | Completeness is not claimed; absence is not treated as proof |
| Confirmed 429 or rate-limited 403 | Header-aware bounded retry or cached/local continuation |
| Permission-denied 403 | Permission diagnosis, no blind rate-limit retry loop |
| Known repo with local user changes | Reuses checkout safely, records revision, does not overwrite changes |
| Multiple agents already authorized | Shared candidates and request budget; partitioned inspection |

Structural checks: valid YAML frontmatter with matching folder/name; referenced files exist; no scaffold placeholders; UI prompt names the skill; no credentials or source transcript in the public repository.

If the Codex skill-creator validator is installed, run its `scripts/quick_validate.py` against `skills/github-research-at-scale`. No external runtime dependencies are required to use the skill.
