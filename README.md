# Toolken Docs

Product documentation for [Toolken](https://toolken.ai), built with [Mintlify](https://mintlify.com).

## Information Architecture

| Section | Pages |
|---------|-------|
| Get Started | What is Toolken, Quickstart, Auth, First Cost Report |
| Attribution & Metadata | Overview, Headers Reference, Recipes, Per-Run & Per-Step |
| Integrations | Choose Your Path, OpenClaw, Hermes, CrewAI, LangGraph, API Quickstart, OpenAI-Compat Clients |
| Rate Limits | Rate Limits |
| Providers | Overview, BYOK, Model Routing |
| Concepts | How It Works, Why Edge, Metadata Model, Pricing |
| Troubleshooting | Error Codes, Cost Not Showing, FAQ |
| API Reference | Introduction, Endpoints, Errors |
| Cookbook | Overview |

## Local Preview

Never run `npm` directly on the host. Use a throwaway Docker container:

```bash
docker run --rm -it \
  -v "$PWD":/docs \
  -w /docs \
  -p 3000:3000 \
  node:20 \
  npx -y mint@latest dev
```

Then open http://localhost:3000.

## Deployment

This directory (`docs/`) is mirrored to the **ToolKen-ai/docs** GitHub repository by `.github/workflows/docs-mirror.yml`. Mintlify deploys automatically from that repository on every push to `main`.

## Logo & Favicon

The `logo/` and `favicon.svg` files are placeholder SVGs. Replace with official Toolken brand assets when available.
