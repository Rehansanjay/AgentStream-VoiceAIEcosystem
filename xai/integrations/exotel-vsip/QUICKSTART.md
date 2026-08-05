# Quickstart — xAI Voice Agent SIP + Exotel SIP trunking (auth-based)

Goal: first successful **inbound** PSTN call: **Exotel DID → xAI SIP → your webhook → Realtime WS joins → agent speaks**.

## Prereqs

- Exotel: SIP trunking enabled, DID active (E.164)
- xAI: API key; public HTTPS webhook
- Same SIP digest username/password planned for Exotel credentials and xAI `sip_auth`

Shared Exotel API snippets:

- [`docs/support/_exotel-trunk-api-snippets.md`](../../../docs/support/_exotel-trunk-api-snippets.md)

## Inbound (PSTN → Exotel → xAI)

1. **xAI**
   - `POST https://api.x.ai/v2/phone-numbers` with `origin: "byo_trunk"`, E.164 = Exotel DID, `webhook.url`, and **`sip_auth.auth_username` / `auth_password`** (digest only — no `allowed_addresses`)
   - Store `dispatch_signing_secret` (returned once)
2. **Exotel**
   - Create trunk → map DID
   - `POST .../credentials` with the **same** digest as xAI `sip_auth`
   - `POST .../destination-uris` toward `sip.voice.x.ai:5061;transport=tls` (SIP user = registered E.164)
   - Flow → **Connect** → Dial whom = `sip:<trunk_sid>`
3. **Webhook**
   - Verify Standard Webhooks signature
   - Open `wss://api.x.ai/v1/realtime?call_id=…` → `session.update` → `response.create`
4. Call the Exotel DID and confirm the agent answers.

## If calls fail

- Digest mismatch between Exotel credentials and xAI `sip_auth`
- Wrong destination host/port/transport or SIP user (E.164)
- Webhook secret / signature failure; WS never joined
- Full guide: [`docs/support/exotel-xai-sip-trunk.md`](../../../docs/support/exotel-xai-sip-trunk.md)

## Links

- [xAI SIP Phone Calls](https://docs.x.ai/developers/model-capabilities/audio/voice-agent/sip)
- Connector: [`xai-exotel-voice-ai-connector.md`](./xai-exotel-voice-ai-connector.md)
