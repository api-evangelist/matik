---
name: Generate a presentation from a Matik template
description: >-
  Browse Matik templates, resolve their required inputs (running any data-source
  query callbacks), generate a presentation/document/spreadsheet, and retrieve the
  download URL. Grounded in the Matik MCP server tools and the Matik External REST API.
api: mcp/matik-mcp.yml
source: https://help.matik.io/hc/en-us/articles/48790661770267--BETA-Matik-MCP-Server
operations:
  - get_all_templates
  - get_inputs_for_template
  - query_callback
  - generate_content
rest_equivalents:
  - GET /api/1.0/templates
  - POST /api/1.0/templates/{template_id}/inputs
  - POST /api/1.0/presentations
  - GET /api/1.0/presentations/{presentation_id}
---

# Generate a presentation from a Matik template

Use this to produce a data-driven presentation, document, or spreadsheet from an
existing Matik template.

## Auth
Authenticate with OAuth 2.0 (authorization code). Present `Authorization: Bearer
{access_token}` against `https://app.matik.io/api/1.0/`. The `producer` scope is
required to manage templates; `consumer` is enough to generate from existing ones.
Via the MCP server, OAuth uses dynamic client registration — authorize once with a
valid Matik account.

## Steps
1. **List templates** — call `get_all_templates` (REST: `GET /templates`) to get
   available templates with their names and IDs. Choose a `template_id`.
2. **Get required inputs** — call `get_inputs_for_template` with `template_id`
   (REST: `POST /templates/{template_id}/inputs`). If you set values that other
   inputs depend on, pass `input_values_by_id` and call again so dependent options
   recompute.
3. **Resolve data queries** — if step 2 returns a `query_callback_url`, call
   `query_callback` with it. It polls until the underlying data-source lookups
   finish (seconds to minutes) and returns updated input options.
4. **Generate** — call `generate_content` with `template_id` and
   `input_values_by_name` (REST: `POST /presentations`). The operation is
   asynchronous: poll the status URL (from the `Location` header) — `200` means
   still processing, `303` means done.
5. **Download** — when status is `done`, use the returned download URL to fetch the
   generated file.

## Conventions
- Long-running calls are poll-based (200 = in progress, 303 = complete). See
  `conventions/matik-conventions.yml`.
- List endpoints paginate with `page` (0-indexed) and `per_page` (1–100, default 20).
- No idempotency-key contract is documented; avoid blind retries of `generate_content`.
