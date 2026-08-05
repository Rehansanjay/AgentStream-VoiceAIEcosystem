# xAI Voice Agent SIP + Exotel vSIP — connector reference

Complements [`docs/support/exotel-xai-sip-trunk.md`](../../../docs/support/exotel-xai-sip-trunk.md).

> **India market compliance:** For India deployments, keep **SIP signaling and media** on paths that remain **in India** (India-region telephony edges and, where the provider offers it, India or in-country data residency). If a Voice AI provider’s SIP signaling or media endpoints are **outside India**, prefer a **self-hosted** deployment of that stack in an **India region** (or an equivalent India-resident architecture) so signaling and media stay in-country. Confirm residency and lawful-interception requirements with your compliance team and with Exotel before go-live.

---

## Roles

| System | Role |
|--------|------|
| **Exotel vSIP** | India PSTN carrier; trunk, DID, digest credentials, destination URI toward xAI |
| **xAI Voice Agent SIP** | Terminates SIP at `sip.voice.x.ai`; fires webhook; you join via Realtime WebSocket |

**Not this path:** Exotel Voicebot `wss://` / Pipecat `ExotelFrameSerializer`.

---

## Auth model (auth-based — digest only)

| Layer | What to use |
|-------|-------------|
| **xAI phone number** | `sip_auth.auth_username` + `sip_auth.auth_password` on `POST /v2/phone-numbers`. Do **not** use `allowed_addresses` as the primary model. |
| **Exotel trunk** | `POST .../credentials` with the **same** username/password. Prefer digest; skip `whitelisted-ips` unless Exotel support requires it. |

Confirm with Exotel that the platform can present digest when sending INVITEs to the destination URI (UAC digest toward xAI).

---

## xAI: register BYO number

`POST https://api.x.ai/v2/phone-numbers` with `Authorization: Bearer $XAI_API_KEY`.

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

Response includes `webhook.dispatch_signing_secret` (**once only**) and `phone_number.sip_host` (typically `sip.voice.x.ai`).

Carrier destination (per [xAI SIP](https://docs.x.ai/developers/model-capabilities/audio/voice-agent/sip)):

```text
sip:+9198XXXXXXXX@sip.voice.x.ai;transport=tls
```

---

## Webhook → Realtime WebSocket

On `realtime.call.incoming`, verify Standard Webhooks headers with the signing secret, then:

```python
import asyncio
import json
import os
import websockets

AUTH = {"Authorization": "Bearer " + os.environ["XAI_API_KEY"]}

async def handle_sip_call(call_id: str):
    uri = f"wss://api.x.ai/v1/realtime?call_id={call_id}"
    async with websockets.connect(uri, additional_headers=AUTH) as ws:
        await ws.send(json.dumps({
            "type": "session.update",
            "session": {
                "voice": "eve",
                "instructions": "You are a helpful phone support agent.",
                "turn_detection": {"type": "server_vad"},
            },
        }))
        await ws.send(json.dumps({"type": "response.create"}))
        async for msg in ws:
            print(json.loads(msg).get("type"))

# asyncio.run(handle_sip_call("00000000-0000-0000-0000-000000000000"))
```

Unlike OpenAI Realtime SIP, xAI does **not** require a separate `/accept` REST call before joining the WebSocket.

Call control:

```bash
curl -X POST "https://api.x.ai/v1/realtime/calls/${CALL_ID}/refer" \
  -H "Authorization: Bearer ${XAI_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"target_uri": "sip:agent@example.com"}'

curl -X POST "https://api.x.ai/v1/realtime/calls/${CALL_ID}/hangup" \
  -H "Authorization: Bearer ${XAI_API_KEY}"
```

---

## Exotel: trunk steps (summary)

1. `POST .../trunks` — create trunk  
2. `POST .../trunks/{TRUNK_SID}/phone-numbers` — map same E.164 as xAI  
3. `POST .../trunks/{TRUNK_SID}/credentials` — **same** digest as xAI `sip_auth`  
4. `POST .../trunks/{TRUNK_SID}/destination-uris` — `sip.voice.x.ai:5061;transport=tls`  
5. Flow → Connect → **`sip:<trunk_sid>`**

Shared curls: [`docs/support/_exotel-trunk-api-snippets.md`](../../../docs/support/_exotel-trunk-api-snippets.md).

---

## References

- [xAI SIP Phone Calls](https://docs.x.ai/developers/model-capabilities/audio/voice-agent/sip)
- [xAI Voice API — phone numbers](https://docs.x.ai/developers/rest-api-reference/inference/voice)
- [Exotel SIP API](https://docs.exotel.com/dynamic-sip-trunking/detailed-sip-trunking-api-reference)
