# OpenAPI — Exotel SIP trunk (Voice AI subset)

Machine-readable [OpenAPI 3.1](https://swagger.io/specification/) description of the Exotel SIP trunk APIs used across the [AgentStream Voice AI Ecosystem](https://github.com/exotel/AgentStream-VoiceAIEcosystem) guides.

| File | Purpose |
|------|---------|
| [`exotel-sip-trunk-voice-ai.yaml`](./exotel-sip-trunk-voice-ai.yaml) | Create trunk, map DID, credentials (digest), whitelist IP, destination URIs |

This is a **curated subset** for Voice AI integrations — not a full Exotel API catalog. Authoritative product docs: [Exotel SIP trunking API reference](https://docs.exotel.com/dynamic-sip-trunking/detailed-sip-trunking-api-reference).

## Quick use

**Swagger Editor (browser):** paste or load the YAML at [editor.swagger.io](https://editor.swagger.io/).

**Postman:** Import → Link / file → select `exotel-sip-trunk-voice-ai.yaml`.

**Codegen (example):**

```bash
npx @openapitools/openapi-generator-cli generate \
  -i openapi/exotel-sip-trunk-voice-ai.yaml \
  -g python \
  -o /tmp/exotel-sip-trunk-client
```

## Auth

HTTP Basic: **API Key** + **API Token** from [API Settings (India)](https://my.in.exotel.com/apisettings/site#api-credentials) (or your cluster).

Servers in the spec:

- `https://api.in.exotel.com` — India
- `https://api.exotel.com` — Singapore / other clusters

## Related human docs

- Shared curls: [`docs/support/_exotel-trunk-api-snippets.md`](../docs/support/_exotel-trunk-api-snippets.md)
- Provider matrix: [`docs/support/README.md`](../docs/support/README.md)
