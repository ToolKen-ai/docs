# Toolken Docs (Mintlify) -- Authoring Guide

## What this is

The Mintlify docs site for docs.toolken.ai. `docs.json` is the nav source of truth. Modeled on Helicone (structure) + Portkey (feature docs). Monorepo `docs/` mirrors to `ToolKen-ai/docs` via `.github/workflows/docs-mirror.yml`.

## IA (3 anchors)

- **Documentation**: Getting Started · Observability · Providers & Routing · Rate Limits · Integrations (Agent frameworks / Coding tools / SDKs) · Use cases
- **API Reference**: Gateway API (Overview/Headers/Errors) · AI Gateway (endpoint pages) · Models · Toolken API (stub)
- **FAQ**

## Voice (human, not AI)

No em-dashes. Short sentences. Lead with the concrete thing. No hype words (unleash/seamless/effortless/powerful/robust). Contractions ok. One job per page. Honest about latency and coming-soon.

## Hard content rules (NEVER violate)

- NEVER invent features/endpoints/config/integrations -- the codebase is the source of truth.
- NEVER name pricing-data sources (no LiteLLM/Portkey/"OpenRouter as our source"). Pricing = "our catalog".
- NEVER expose internal infra: Redis, JSONB, TimescaleDB, proxy_logs, Sidekiq, escrow, column names, hypertables. Talk product, not plumbing.
- NEVER put internal ticket IDs (HC-xxxx) in user-facing pages.
- Mark coming-soon features; never demo them as working. Honest latency: "a single edge hop", never "zero latency".
- No image reference without a real asset (no placeholders). Existing screenshots: `images/cost-by-feature.png`, `cost-by-customer.png`, `dashboard-overview.png`, `quickstart-dashboard.png`.

## Verified product facts (canonical reference)

| Topic | Fact |
|---|---|
| Base URL | `https://gateway.toolken.ai/v1`. Anthropic SDK uses `https://gateway.toolken.ai` (no `/v1`; SDK appends `/v1/messages`). |
| Auth | `X-Toolken-Key` (docs use `tk_live_`). Provider key (BYOK) in `Authorization` / `x-api-key`, forwarded untouched, never stored. |
| Attribution headers | `X-Toolken-Metadata` (JSON; primary channel), `X-Toolken-Metadata-*` (kebab to snake_case; e.g. `X-Toolken-Metadata-Agent`, `X-Toolken-Metadata-Feature`, `X-Toolken-Metadata-Customer-Id`). Legacy `X-Toolken-Meta-*` accepted for compat. `X-Toolken-Provider` forces routing; `X-Toolken-Model` is the model-routing hint. Both stripped before forwarding. No bare `X-Toolken-Feature` or `X-Toolken-Customer-Id` aliases. |
| Reserved/promoted keys | Only `environment` and `session_id` are reserved (stored as typed columns `_environment`/`_session_id`). `feature` and `customer_id` are generic metadata keys, not reserved dimensions. |
| Metadata limits | `gateway/src/config/constants.ts`: 16384 bytes, 64 keys, key regex `^_?[a-z][a-z0-9_]{0,63}$`. Malformed `X-Toolken-Metadata` JSON is silently ignored (NOT 422); 422 fires only on valid-JSON-but-invalid-key/size. |
| Supported endpoints | `gateway/src/providers/index.ts` OPENAI_PATHS/ANTHROPIC_PATHS: `/v1/chat/completions`, `/v1/embeddings`, `/v1/responses`, `/v1/models`, `/v1/audio`, `/v1/images`, and Anthropic `/v1/messages` + `/v1/models`. `/v1/models` default = our own catalog (OpenAI shape, or Anthropic shape on `anthropic-version`/`anthropic-beta` hint; only `X-Toolken-Key` required); `X-Toolken-Provider: <slug>` = live passthrough to that provider (requires BYOK provider auth). NO `/v1/batches`. |
| Providers | 13 native providers + OpenRouter (breadth). No model-count headline, no pricing-source name. |
| Multimodal | Embeddings work and are cost-tracked (token-priced). Images and audio pass through but are NOT cost-tracked yet (keep the "coming soon" disclaimer). |
| Privacy | We do NOT store prompts or completions. The gateway logs request metadata only (model, provider, tokens, cost, latency, status, attribution tags). `LogEvent` in `gateway/src/types.ts` has no content fields. |
| Rate limits | Enforced but fail OPEN; auth fails CLOSED. Self-serve rate-limit config is coming soon. |
| Error body shape | `{ error: { message, type, param, code } }`. Real codes live in `gateway/src/errors/` + middleware: 401 `missing_api_key`/`invalid_api_key`, 403 `tenant_suspended`, 413 `payload_too_large`, 400 `invalid_provider`/`incompatible_path`, 422 `invalid_metadata`, 429 `rate_limited`/`concurrent_stream_limit`, 502 `provider_unreachable`/`response_too_large`, 503 `auth_unavailable`/`tenant_unavailable`, 504 `gateway_timeout`. |

## Mintlify patterns (hard-won)

- **API Reference endpoint pages: write MANUAL pages** -- frontmatter `api: "POST https://..."` + `<ParamField>`/`<ResponseField>` + `<RequestExample>`/`<ResponseExample>`. The spec-driven `openapi:` frontmatter approach rendered EMPTY here; manual pages always render. (`docs/api-reference/openapi.json` is kept as a machine-readable reference only.)
- If you do use an OpenAPI spec: declare `openapi` INSIDE the nav anchor/group (not top-level); OpenAPI 3.1 forbids `nullable: true` (use `type: ["x","null"]`); per-page frontmatter needs the spec path when the spec is not at root.
- `mint dev` does NOT hot-reload `docs.json` or the OpenAPI spec -- RESTART it after nav/spec edits. MDX content does hot-reload.
- Every nav slug must have a matching MDX file or you get a 404.
- Components: `<Steps>`, `<CardGroup>/<Card>`, `<CodeGroup>` (language tabs), `<Accordion>/<AccordionGroup>` (FAQ), `<Frame caption>`, `<Note>/<Tip>/<Warning>`, `<ParamField>/<ResponseField>`, `<RequestExample>/<ResponseExample>`, `<Table>`. Validate icon names against Mintlify's set.

## Verifying integrations & API examples (CRITICAL -- verify by EXECUTION)

- Third-party tool capability (does it forward a custom header? what config key?) MUST be verified by RUNNING it, not by docs or web search. Open web search fabricated GitHub issue/PR numbers, versions, and config in this repo's history.
- Harness: `.eng/integration-verify/` (a Docker header-echo that logs inbound headers + per-tool configs). To verify a new tool: point its base_url at the echo, set the custom header via the tool's mechanism, run one prompt, `grep -i x-toolken-key` the echo log.
- Verified status:

| Tool | Status | Notes |
|---|---|---|
| OpenClaw | ✅ | headers map |
| Hermes | coming soon | -- |
| Claude Code | ✅ | `ANTHROPIC_CUSTOM_HEADERS` |
| Codex | ✅ | `config.toml http_headers`, uses `/v1/responses` |
| Continue | ✅ | `requestOptions.headers` |
| Cursor | coming soon | -- |
| OpenAI SDK | ✅ | `default_headers` |
| Anthropic SDK | ✅ | `default_headers` |
| CrewAI | ✅ | `LLM(extra_headers=)` |
| LangGraph | ✅ | `ChatOpenAI(default_headers=)` |

- API examples: verify against the real gateway routed to `mock-llm` (`docker compose --profile e2e up ... mock-llm gateway`). Known quirks: Ruby `ruby-openai` needs `uri_base:` WITH a trailing slash + `extra_headers:` (`base_url`/`default_headers` are ignored); Go `openai-go` strips the `/v1` path, so use raw `net/http` to `/v1/chat/completions` (or a base URL that keeps `/v1/`).

## Sync

`landing/src/data/landing.ts` carries its own multi-language code samples (Node/Python/curl/Ruby) -- keep them consistent with the docs (base URLs, headers, Ruby trailing slash).

## Build order

Wave 1 (shipped): all sections above. Coming-soon (document when shipped): response cache, fallbacks/routing, vault & scoped keys, native multimodal cost tracking.
