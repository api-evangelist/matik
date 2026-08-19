---
name: Run a Matik dynamic content query
description: >-
  Find a Matik dynamic content item, resolve its inputs (running any data-source
  query callbacks), run its parameterized query against the connected data sources,
  and read the results. Grounded in the Matik MCP server tools and REST API.
api: mcp/matik-mcp.yml
source: https://help.matik.io/hc/en-us/articles/48790661770267--BETA-Matik-MCP-Server
operations:
  - search_dynamic_content
  - get_dynamic_content_by_name
  - get_inputs_for_dynamic_content
  - query_callback
  - run_dynamic_content
rest_equivalents:
  - GET /api/1.0/dynamic_content
  - GET /api/1.0/dynamic_content/name/:dynamic_content_name
  - POST /api/1.0/dynamic_content/:dynamic_content_id/inputs
  - GET /api/1.0/dynamic_content/:dynamic_content_id/inputs/options/:bulk_query_id
  - POST /api/1.0/dynamic_content/run/:dynamic_content_id
  - GET /api/1.0/dynamic_content/query_status/:query_run_id
---

# Run a Matik dynamic content query

Use this to run a parameterized Matik dynamic content item against your connected
data sources and read back the results.

## Auth
OAuth 2.0 bearer token against `https://app.matik.io/api/1.0/`. See
`authentication/matik-authentication.yml`.

## Steps
1. **Find the item** — call `search_dynamic_content` with a `search_term` (paginated
   via `page`/`per_page`), or `get_dynamic_content_by_name` if you know the exact
   name, to obtain the `dynamic_content_id`.
2. **Get inputs** — call `get_inputs_for_dynamic_content` with `dynamic_content_id`
   to see the required input fields.
3. **Resolve data queries** — if a `query_callback_url` is returned, call
   `query_callback` and let it poll to completion.
4. **Run** — call `run_dynamic_content` with the `dynamic_content_id` and input
   values to execute the query against the data source.
5. **Read results** — return the query results to the caller.

## Conventions
- Pagination: `page` (0-indexed), `per_page` (1–100, default 20).
- Query resolution and runs are poll-based; wait for completion before reading.
- `503` on a data-source connection test means the upstream warehouse/BI system is
  unreachable, not Matik — retry with backoff.
- See `conventions/matik-conventions.yml`, `data-model/matik-data-model.yml`,
  `errors/matik-problem-types.yml` and `mcp/matik-tool-crosswalk.yml`.
