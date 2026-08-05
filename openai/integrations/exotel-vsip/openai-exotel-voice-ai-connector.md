# OpenAI Realtime SIP + Exotel vSIP — connector reference

Complements [`docs/support/exotel-openai-sip-trunk.md`](../../../docs/support/exotel-openai-sip-trunk.md).

> **India market compliance:** For India deployments, keep **SIP signaling and media** on paths that remain **in India** (India-region telephony edges and, where the provider offers it, India or in-country data residency). If a Voice AI provider’s SIP signaling or media endpoints are **outside India**, prefer a **self-hosted** deployment of that stack in an **India region** (or an equivalent India-resident architecture) so signaling and media stay in-country. Confirm residency and lawful-interception requirements with your compliance team and with Exotel before go-live.

---

## Roles

| System | Role |
|--------|------|
| **Exotel vSIP** | India PSTN carrier; trunk, DID, digest credentials, destination URI toward OpenAI |
| **OpenAI Realtime SIP** | Terminates SIP at `sip.api.openai.com`; fires webhook; Realtime session after `/accept` |

**Not this path:** Exotel Voicebot `wss://` media streaming / Pipecat `ExotelFrameSerializer`. Those use Exotel’s Voicebot JSON protocol, not OpenAI’s `call_id` Realtime WebSocket.

---

## Auth model (auth-based)

| Layer | What to use |
|-------|-------------|
| **Exotel trunk** | `POST .../credentials` (SIP digest). Prefer digest; do **not** `whitelisted-ips` OpenAI media CIDRs (Exotel ACL is `/32` only; OpenAI publishes ranges). |
| **OpenAI SIP** | Routing key = OpenAI **project ID** in the SIP URI user part (`sip:proj_…@sip.api.openai.com;transport=tls`). OpenAI does **not** expose a peer digest API for Realtime SIP. |

---

## OpenAI: project SIP target

1. [platform.openai.com](https://platform.openai.com/settings) → Project → **General** → copy `proj_…`.
2. Project → **Webhooks** → HTTPS URL for `realtime.call.incoming`.
3. Carrier / Exotel destination (Twilio-equivalent):

```text
sip:proj_YOUR_PROJECT_ID@sip.api.openai.com;transport=tls
```

EU data residency only (not India): `sip-eu.api.openai.com`. For India compliance, if OpenAI SIP/media is outside India, prefer self-hosted India-region alternatives per the compliance note above.

Signaling: TCP/TLS to DNS for `sip.api.openai.com` on port **5061**. Media: SRTP per [OpenAI SIP guide](https://developers.openai.com/api/docs/guides/realtime-sip).

---

## Webhook: accept the call

```bash
curl -X POST "https://api.openai.com/v1/realtime/calls/${CALL_ID}/accept" \
  -H "Authorization: Bearer ${OPENAI_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
        "type": "realtime",
        "model": "gpt-realtime-2.1",
        "instructions": "You are a friendly phone support agent."
      }'
```

Reject (optional):

```bash
curl -X POST "https://api.openai.com/v1/realtime/calls/${CALL_ID}/reject" \
  -H "Authorization: Bearer ${OPENAI_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"status_code": 486}'
```

---

## Optional: monitor over WebSocket

`GET wss://api.openai.com/v1/realtime?call_id={call_id}` with `Authorization: Bearer …`. Send `response.create` and other client events as needed.

```python
import asyncio
import json
import os
import websockets

AUTH = {"Authorization": "Bearer " + os.environ["OPENAI_API_KEY"]}

async def monitor(call_id: str):
    uri = f"wss://api.openai.com/v1/realtime?call_id={call_id}"
    async with websockets.connect(uri, additional_headers=AUTH) as ws:
        await ws.send(json.dumps({"type": "response.create"}))
        async for msg in ws:
            print(msg)

# asyncio.run(monitor("rtc_..."))
```

Call control: `POST .../refer`, `POST .../hangup` — see [Realtime SIP](https://developers.openai.com/api/docs/guides/realtime-sip).

---

## Exotel: trunk steps (summary)

1. `POST .../trunks` — create trunk  
2. `POST .../trunks/{TRUNK_SID}/phone-numbers` — map E.164 DID  
3. `POST .../trunks/{TRUNK_SID}/credentials` — digest (auth-based)  
4. `POST .../trunks/{TRUNK_SID}/destination-uris` — toward OpenAI TLS (see support article)  
5. Flow → Connect → **`sip:<trunk_sid>`**

Shared curls: [`docs/support/_exotel-trunk-api-snippets.md`](../../../docs/support/_exotel-trunk-api-snippets.md).

---

## References

- [Realtime API with SIP](https://developers.openai.com/api/docs/guides/realtime-sip)
- [Accept call](https://developers.openai.com/api/reference/resources/realtime/subresources/calls/methods/accept)
- [Exotel SIP API](https://docs.exotel.com/dynamic-sip-trunking/detailed-sip-trunking-api-reference)
