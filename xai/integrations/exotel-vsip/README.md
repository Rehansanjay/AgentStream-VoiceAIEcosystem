# Voice AI Ecosystem: xAI Grok Voice Agent SIP + Exotel vSIP

Connect **[Exotel](https://exotel.com/)** Virtual SIP Trunking to **[xAI Speech to Speech / Voice Agent SIP](https://docs.x.ai/developers/model-capabilities/audio/voice-agent/sip)** using **BYO trunk** (`origin: "byo_trunk"`) and **SIP digest auth**.

**Path:** inbound SIP trunk only (auth-based digest on xAI + Exotel). This is **not** Exotel Voicebot WebSocket streaming — do not use Pipecat `ExotelFrameSerializer` on this path.

| Doc | Use when |
|-----|----------|
| [**QUICKSTART.md**](./QUICKSTART.md) | Short inbound checklist |
| [**xai-exotel-voice-ai-connector.md**](./xai-exotel-voice-ai-connector.md) | `POST /v2/phone-numbers`, digest, webhook + WS snippet |
| [**exotel-xai-sip-trunk.md**](../../../docs/support/exotel-xai-sip-trunk.md) | End-to-end support article |

## Publishing

Do not commit live API keys, webhook secrets, or SIP passwords.
