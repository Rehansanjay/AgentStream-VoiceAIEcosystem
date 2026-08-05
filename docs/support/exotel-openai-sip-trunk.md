# Connect Exotel SIP trunking to OpenAI Realtime SIP

This guide connects **Exotel SIP trunking** to **[OpenAI Realtime API over SIP](https://developers.openai.com/api/docs/guides/realtime-sip)** so callers dial an India Exotel DID and reach a Realtime voice agent. Integration is **inbound SIP trunking** with an **auth-based (digest)** posture on the Exotel trunk.

> **India market compliance:** For India deployments, keep **SIP signaling and media** on paths that remain **in India** (India-region telephony edges and, where the provider offers it, India or in-country data residency). If a Voice AI provider’s SIP signaling or media endpoints are **outside India**, prefer a **self-hosted** deployment of that stack in an **India region** (or an equivalent India-resident architecture) so signaling and media stay in-country. Confirm residency and lawful-interception requirements with your compliance team and with Exotel before go-live.

**GitHub repo (reference):** https://github.com/exotel/AgentStream-VoiceAIEcosystem

> **Applicability:** **API-driven** (OpenAI webhook + `/accept`; Exotel trunk APIs + Flow Connect).

> **Auth-based:** Prefer Exotel **`POST .../credentials`** (SIP digest). Do **not** add OpenAI media CIDRs to Exotel `whitelisted-ips`. OpenAI identifies the project via **`proj_…` in the SIP URI**, not digest peer auth.

> **Not Voicebot WSS:** This is Direct SIP to OpenAI — not Exotel Voicebot streaming and not Pipecat `ExotelFrameSerializer`.

> **Engineering detail:** [`openai/integrations/exotel-vsip/openai-exotel-voice-ai-connector.md`](../../openai/integrations/exotel-vsip/openai-exotel-voice-ai-connector.md)

> **Quickstart:** [`openai/integrations/exotel-vsip/QUICKSTART.md`](../../openai/integrations/exotel-vsip/QUICKSTART.md)

---

## OpenAI (from official docs)

| Topic | Detail |
|--------|--------|
| SIP guide | [Realtime API with SIP](https://developers.openai.com/api/docs/guides/realtime-sip) |
| Origination URI | `sip:$PROJECT_ID@sip.api.openai.com;transport=tls` (`proj_` prefix) |
| EU residency | `sip-eu.api.openai.com` (EU only — **not** an India residency option) |
| Webhook event | `realtime.call.incoming` |
| Accept | `POST /v1/realtime/calls/{call_id}/accept` |
| Control WS | `wss://api.openai.com/v1/realtime?call_id={call_id}` |
| Signaling | TCP/TLS port **5061** to GeoIP-routed SIP host |
| Media | SRTP (UDP) to OpenAI media CIDRs in the SIP guide |

---

## Architecture

```text
Caller → India PSTN → Exotel DID → Flow (Connect sip:<trunk_sid>)
  → sip:proj_…@sip.api.openai.com;transport=tls
  → OpenAI webhook (realtime.call.incoming)
  → POST /accept → Realtime session
  → optional wss://…/realtime?call_id=… for control
```

| Direction | Supported in this article |
|-----------|---------------------------|
| **Inbound** (PSTN → Exotel → OpenAI) | Yes |
| **Outbound** (OpenAI agent → Exotel → PSTN) | Out of scope (not covered like Twilio termination BYOT) |

---

## Prerequisites

### Exotel

- [my.in.exotel.com](https://my.in.exotel.com) with **SIP trunking** enabled
- Exophone (DID) in **E.164** (`+91…`); KYC complete
- API Key / Token / Account SID from [API credentials](https://my.in.exotel.com/apisettings/site#api-credentials)
- APIs: `https://api.in.exotel.com` (India)

### OpenAI

- Project with Realtime API access and API key
- HTTPS webhook URL + webhook signing secret
- Project ID (`proj_…`)

### Network

- Exotel → OpenAI: SIP **TLS** (5061); media **SRTP** as negotiated — confirm TLS/SRTP with Exotel for your account

---

## Part A — OpenAI

1. In [platform settings](https://platform.openai.com/settings) → Project → **Webhooks**, register your HTTPS endpoint.
2. Copy **project ID** (`proj_…`) from Project → **General**.
3. Implement the webhook handler:
   - Verify OpenAI webhook signature
   - On `realtime.call.incoming`, read `data.call_id`
   - Call **Accept** with model / instructions / voice (same shape as create client secret session config)
4. Optionally open the Realtime WebSocket with `call_id` to send `response.create` and handle tools/events.
5. Use **Reject**, **Refer**, or **Hangup** as needed ([guide](https://developers.openai.com/api/docs/guides/realtime-sip)).

Example accept:

```bash
curl -X POST "https://api.openai.com/v1/realtime/calls/${CALL_ID}/accept" \
  -H "Authorization: Bearer ${OPENAI_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
        "type": "realtime",
        "model": "gpt-realtime-2.1",
        "instructions": "You are a friendly phone support agent for Example Corp."
      }'
```

---

## Part B — Exotel APIs (auth-based inbound)

**Auth:** `API_KEY:API_TOKEN@api.in.exotel.com` · [`_exotel-trunk-api-snippets.md`](./_exotel-trunk-api-snippets.md)

### 1. Create trunk → map DID → digest credentials

```bash
# Create trunk (see shared snippets for full body)
# Map phone number
# Then:
curl -s -X POST \
  "https://${API_KEY}:${API_TOKEN}@${SUBDOMAIN}/v2/accounts/${ACCOUNT_SID}/trunks/${TRUNK_SID}/credentials" \
  -H "Content-Type: application/json" \
  -d '{
    "user_name": "openai_sip_user",
    "password": "REPLACE_WITH_STRONG_PASSWORD",
    "friendly_name": "openai_realtime"
  }'
```

Do **not** configure `whitelisted-ips` for OpenAI media ranges on this path.

### 2. Destination URI (inbound toward OpenAI)

Point the trunk at OpenAI’s SIP host over **TLS**. Align the SIP **user** with your `proj_…` (confirm with Exotel if the destination string must include `proj_…@` vs host-only + From/To mapping):

```bash
curl -s -X POST "https://${API_KEY}:${API_TOKEN}@${SUBDOMAIN}/v2/accounts/${ACCOUNT_SID}/trunks/${TRUNK_SID}/destination-uris" \
  -H "Content-Type: application/json" \
  -d '{
    "destinations": [
      { "destination": "sip.api.openai.com:5061;transport=tls" }
    ]
  }'
```

OpenAI’s documented carrier URI form is `sip:proj_…@sip.api.openai.com;transport=tls`. If Exotel requires the full user@host in `destination`, use the form Exotel documents for user-part destinations (coordinate with Exotel support).

### 3. Connect applet

**Dial whom:** **`sip:<trunk_sid>`** only — [Voice AI / SIP trunking Flow guide](https://support.exotel.com/support/solutions/articles/3000133452-flow-and-api-configuration-guide-for-voice-ai-contact-centre-platforms-via-exotel-virtual-sip-trunk).

---

## Test

1. Place a PSTN call to the Exotel DID.
2. Confirm webhook receives `realtime.call.incoming`.
3. Confirm `/accept` returns 200 and audio bridges.
4. Optional: watch Realtime WS events; hang up via API or caller.

---

## Troubleshooting

| Symptom | What to check |
|---------|----------------|
| No webhook | Project ID / destination URI; Flow Connect `sip:<trunk_sid>`; webhook URL reachable |
| Webhook but no audio | `/accept` body (model/type); TLS/SRTP negotiation; Exotel media path |
| 401 on accept | `OPENAI_API_KEY`; call_id from webhook |
| Slow answer / caller drops | Accept latency; keep webhook handler fast (async accept + WS) |
| India compliance concern | OpenAI global/EU SIP hosts — evaluate self-hosted India-region options per compliance note |

---

## References

| Resource | URL |
|----------|-----|
| OpenAI Realtime SIP | https://developers.openai.com/api/docs/guides/realtime-sip |
| OpenAI webhooks | https://developers.openai.com/api/docs/guides/webhooks |
| Exotel SIP API | https://docs.exotel.com/dynamic-sip-trunking/detailed-sip-trunking-api-reference |
| Exotel network / firewall | https://docs.exotel.com/dynamic-sip-trunking/network-and-firewall-configuration |
| Connector | [`openai/integrations/exotel-vsip/openai-exotel-voice-ai-connector.md`](../../openai/integrations/exotel-vsip/openai-exotel-voice-ai-connector.md) |
