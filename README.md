# AgentStream — Voice AI Ecosystem

Practical integration guides and reference snippets for connecting **Exotel telephony (SIP trunking / Voice Streaming)** with popular Voice AI platforms (e.g., ElevenLabs, LiveKit, Retell, Bolna, Pipecat, Ultravox, Vapi, Smallest AI, Vocallabs, Rapida AI, NLPearl, OpenAI Realtime SIP, xAI Grok Voice Agent SIP).

> **India market compliance:** For India deployments, keep **SIP signaling and media** on paths that remain **in India** (India-region telephony edges and, where the provider offers it, India or in-country data residency). If a Voice AI provider’s SIP signaling or media endpoints are **outside India**, prefer a **self-hosted** deployment of that stack in an **India region** (or an equivalent India-resident architecture) so signaling and media stay in-country. Confirm residency and lawful-interception requirements with your compliance team and with Exotel before go-live.

## What’s in this repo

- **Support guides**: `docs/support/` — step-by-step setup for Exotel ↔ provider integrations.
- **Shared Exotel trunk API snippets**: `docs/support/_exotel-trunk-api-snippets.md` — reusable curl templates (create trunk, map DID, credentials, allowlist, destination URI).
- **OpenAPI (Voice AI subset)**: [`openapi/exotel-sip-trunk-voice-ai.yaml`](./openapi/exotel-sip-trunk-voice-ai.yaml) — machine-readable Exotel SIP trunk contract for Swagger UI, Postman, and client codegen ([`openapi/README.md`](./openapi/README.md)).
- **Provider “connector” docs / quickstarts**: `<provider>/integrations/exotel-vsip/` — implementation and runbooks for each provider’s flow.
- **WebRTC app setup**: `docs/integrations/webrtc-application-setup.md` — WebRTC application wiring notes.

## Quickstart

1. Copy `.env.example` → `.env` and fill in values locally.
2. Pick your provider guide from `docs/support/README.md`.
3. Follow the **Outbound** / **Inbound** flow steps in the chosen support article.

## Security

- **Do not commit secrets**: keep real credentials in `.env` (ignored by git).
- **Use placeholders in docs**: examples should stay as `API_KEY`, `API_TOKEN`, `SIP_PASS`, etc.

