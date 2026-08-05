# Voice AI Ecosystem: OpenAI Realtime SIP + Exotel vSIP

Connect **[Exotel](https://exotel.com/)** Virtual SIP Trunking to **[OpenAI Realtime SIP](https://developers.openai.com/api/docs/guides/realtime-sip)** so India PSTN callers reach a Realtime voice session.

**Path:** inbound SIP trunk only (auth-based / digest on the Exotel trunk). This is **not** Exotel Voicebot WebSocket streaming — do not use Pipecat `ExotelFrameSerializer` on this path.

| Doc | Use when |
|-----|----------|
| [**QUICKSTART.md**](./QUICKSTART.md) | Short inbound checklist |
| [**openai-exotel-voice-ai-connector.md**](./openai-exotel-voice-ai-connector.md) | Project URI, webhook accept, Realtime WS snippet |
| [**exotel-openai-sip-trunk.md**](../../../docs/support/exotel-openai-sip-trunk.md) | End-to-end support article |

## Publishing

Do not commit live API keys, webhook secrets, or SIP passwords.
