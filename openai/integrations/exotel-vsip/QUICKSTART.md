# Quickstart — OpenAI Realtime SIP + Exotel SIP trunking (auth-based)

Goal: first successful **inbound** PSTN call: **Exotel DID → OpenAI Realtime SIP → your webhook accepts → agent speaks**.

## Prereqs

- Exotel: SIP trunking enabled, DID active (E.164), India edge known
- OpenAI: project with Realtime access, webhook endpoint, API key
- Public HTTPS webhook that can call OpenAI `/accept` quickly

Shared Exotel API snippets:

- [`docs/support/_exotel-trunk-api-snippets.md`](../../../docs/support/_exotel-trunk-api-snippets.md)

## Inbound (PSTN → Exotel → OpenAI)

1. **OpenAI**
   - Project → **Webhooks** → endpoint for `realtime.call.incoming`
   - Copy project ID (`proj_…`) from Project → **General**
2. **Exotel**
   - Create trunk → map DID
   - `POST .../credentials` (digest — auth-based; do **not** ACL OpenAI CIDRs)
   - `POST .../destination-uris` toward `sip.api.openai.com:5061;transport=tls` (SIP user = `proj_…`)
   - Flow → **Connect** → Dial whom = `sip:<trunk_sid>`
3. **Webhook**
   - On `realtime.call.incoming` → `POST /v1/realtime/calls/{call_id}/accept` with model/instructions
   - Optional: open `wss://api.openai.com/v1/realtime?call_id=…` for control events
4. Call the Exotel DID and confirm the agent answers.

## If calls fail

- Wrong `proj_…` in destination / SIP user
- Webhook slow or `/accept` failed (401 key, bad body)
- TLS/5061 mismatch on Exotel destination URI
- Full guide: [`docs/support/exotel-openai-sip-trunk.md`](../../../docs/support/exotel-openai-sip-trunk.md)

## Links

- [OpenAI Realtime SIP](https://developers.openai.com/api/docs/guides/realtime-sip)
- Connector: [`openai-exotel-voice-ai-connector.md`](./openai-exotel-voice-ai-connector.md)
