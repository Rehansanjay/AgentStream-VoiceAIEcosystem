# Connect Exotel SIP trunking to xAI Grok Voice Agent (SIP)

This guide connects **Exotel SIP trunking** to **[xAI Speech to Speech / Voice Agent SIP](https://docs.x.ai/developers/model-capabilities/audio/voice-agent/sip)** using **BYO trunk** registration and **SIP digest (auth-based)** authentication. Callers dial an India Exotel DID; Exotel sends SIP to xAI; your webhook joins the Realtime WebSocket.

> **India market compliance:** For India deployments, keep **SIP signaling and media** on paths that remain **in India** (India-region telephony edges and, where the provider offers it, India or in-country data residency). If a Voice AI provider’s SIP signaling or media endpoints are **outside India**, prefer a **self-hosted** deployment of that stack in an **India region** (or an equivalent India-resident architecture) so signaling and media stay in-country. Confirm residency and lawful-interception requirements with your compliance team and with Exotel before go-live.

**GitHub repo (reference):** https://github.com/exotel/AgentStream-VoiceAIEcosystem

> **Applicability:** **API-driven** (xAI `POST /v2/phone-numbers` + webhook + Realtime WS; Exotel trunk APIs + Flow Connect).

> **Auth-based:** Use **digest** on both sides — xAI `sip_auth.auth_username` / `auth_password` and Exotel `POST .../credentials` with the **same** values. Do **not** use `allowed_addresses` / IP ACL as the primary model.

> **Not Voicebot WSS:** Direct SIP to `sip.voice.x.ai` — not Exotel Voicebot streaming and not Pipecat `ExotelFrameSerializer`.

> **Engineering detail:** [`xai/integrations/exotel-vsip/xai-exotel-voice-ai-connector.md`](../../xai/integrations/exotel-vsip/xai-exotel-voice-ai-connector.md)

> **Quickstart:** [`xai/integrations/exotel-vsip/QUICKSTART.md`](../../xai/integrations/exotel-vsip/QUICKSTART.md)

---

## xAI (from official docs)

| Topic | Detail |
|--------|--------|
| SIP guide | [SIP Phone Calls](https://docs.x.ai/developers/model-capabilities/audio/voice-agent/sip) |
| Register number | `POST /v2/phone-numbers` with `origin: "byo_trunk"` ([Voice API](https://docs.x.ai/developers/rest-api-reference/inference/voice)) |
| Destination | `sip:{number}@sip.voice.x.ai;transport=tls` |
| Auth | Digest (`auth_username` / `auth_password`) **or** `allowed_addresses` — **this guide uses digest only** |
| Webhook | `realtime.call.incoming` + Standard Webhooks signing secret (once) |
| Join call | `wss://api.x.ai/v1/realtime?call_id={call_id}` → `session.update` → `response.create` |
| Control | `POST /v1/realtime/calls/{id}/refer`, `/hangup` |
| Codecs | G.711 µ-law / A-law (and G.722 where negotiated) |

---

## Architecture

```text
Caller → India PSTN → Exotel DID → Flow (Connect sip:<trunk_sid>)
  → sip:+91…@sip.voice.x.ai;transport=tls
  → xAI webhook (realtime.call.incoming)
  → wss://api.x.ai/v1/realtime?call_id=… (session.update + response.create)
```

| Direction | Supported in this article |
|-----------|---------------------------|
| **Inbound** (PSTN → Exotel → xAI) | Yes |
| **Outbound** (xAI agent → Exotel → PSTN) | Out of scope |

---

## Prerequisites

### Exotel

- SIP trunking enabled; DID in **E.164**; India API host `api.in.exotel.com`
- Digest username/password you will also set on xAI

### xAI

- API key; HTTPS webhook; ability to store `dispatch_signing_secret`

### Network

- Exotel → xAI: SIP **TLS**; media per SDP — confirm TLS/SRTP with Exotel for your account

---

## Part A — xAI (digest BYO)

1. Deploy a public HTTPS webhook that verifies Standard Webhooks signatures.
2. Register the Exotel DID:

```bash
curl -s -X POST "https://api.x.ai/v2/phone-numbers" \
  -H "Authorization: Bearer ${XAI_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "origin": "byo_trunk",
    "name": "Exotel India DID",
    "phone_number": "+9198XXXXXXXX",
    "sip_auth": {
      "auth_username": "SIP_USER",
      "auth_password": "SIP_PASS"
    },
    "webhook": {
      "url": "https://example.com/xai/webhook"
    }
  }'
```

3. Save `webhook.dispatch_signing_secret` immediately (not returned again).
4. On `realtime.call.incoming`, open the Realtime WebSocket with `call_id`, send `session.update` (voice, instructions), then `response.create`.

There is **no** separate `/accept` REST step (unlike OpenAI Realtime SIP).

---

## Part B — Exotel APIs (auth-based inbound)

**Auth:** `API_KEY:API_TOKEN@api.in.exotel.com` · [`_exotel-trunk-api-snippets.md`](./_exotel-trunk-api-snippets.md)

### 1. Create trunk → map DID → digest credentials

Use the **same** `user_name` / `password` as xAI `sip_auth`:

```bash
curl -s -X POST \
  "https://${API_KEY}:${API_TOKEN}@${SUBDOMAIN}/v2/accounts/${ACCOUNT_SID}/trunks/${TRUNK_SID}/credentials" \
  -H "Content-Type: application/json" \
  -d '{
    "user_name": "SIP_USER",
    "password": "SIP_PASS",
    "friendly_name": "xai_voice"
  }'
```

Skip `whitelisted-ips` unless Exotel support requires ACL for your design.

### 2. Destination URI (inbound toward xAI)

```bash
curl -s -X POST "https://${API_KEY}:${API_TOKEN}@${SUBDOMAIN}/v2/accounts/${ACCOUNT_SID}/trunks/${TRUNK_SID}/destination-uris" \
  -H "Content-Type: application/json" \
  -d '{
    "destinations": [
      { "destination": "sip.voice.x.ai:5061;transport=tls" }
    ]
  }'
```

xAI’s documented URI includes the E.164 user: `sip:{number}@sip.voice.x.ai;transport=tls`. Confirm with Exotel how to encode the user part if host-only destination is insufficient.

### 3. Connect applet

**Dial whom:** **`sip:<trunk_sid>`** — [Voice AI / SIP trunking Flow guide](https://support.exotel.com/support/solutions/articles/3000133452-flow-and-api-configuration-guide-for-voice-ai-contact-centre-platforms-via-exotel-virtual-sip-trunk).

---

## Test

1. Call the Exotel DID from a phone.
2. Confirm webhook receives `realtime.call.incoming` and signature verifies.
3. Confirm WebSocket joins and agent audio plays to the caller.
4. Optional: DTMF over SIP (xAI buffers digits; see [SIP guide](https://docs.x.ai/developers/model-capabilities/audio/voice-agent/sip)).

---

## Troubleshooting

| Symptom | What to check |
|---------|----------------|
| 401 / auth failure toward xAI | Digest match Exotel credentials ↔ xAI `sip_auth`; Exotel UAC digest toward destination |
| No webhook | Destination URI / Flow Connect; number registered as `byo_trunk` with correct E.164 |
| Invalid signature | Wrong/missing `dispatch_signing_secret`; verify raw body |
| No audio after webhook | WS never connected; missing `session.update` / `response.create` |
| India compliance concern | `sip.voice.x.ai` may be outside India — prefer self-hosted India-region options per compliance note |

---

## References

| Resource | URL |
|----------|-----|
| xAI SIP Phone Calls | https://docs.x.ai/developers/model-capabilities/audio/voice-agent/sip |
| xAI Voice API | https://docs.x.ai/developers/rest-api-reference/inference/voice |
| Exotel SIP API | https://docs.exotel.com/dynamic-sip-trunking/detailed-sip-trunking-api-reference |
| Exotel network / firewall | https://docs.exotel.com/dynamic-sip-trunking/network-and-firewall-configuration |
| Connector | [`xai/integrations/exotel-vsip/xai-exotel-voice-ai-connector.md`](../../xai/integrations/exotel-vsip/xai-exotel-voice-ai-connector.md) |
