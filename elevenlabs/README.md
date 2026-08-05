# ElevenLabs

This area covers two ways to work with [ElevenLabs](https://elevenlabs.io/):

> **India market compliance:** For India deployments, keep **SIP signaling and media** on paths that remain **in India** (India-region telephony edges and, where the provider offers it, India or in-country data residency). If a Voice AI provider’s SIP signaling or media endpoints are **outside India**, prefer a **self-hosted** deployment of that stack in an **India region** (or an equivalent India-resident architecture) so signaling and media stay in-country. Confirm residency and lawful-interception requirements with your compliance team and with Exotel before go-live.

| Track | Folder | Purpose |
|--------|--------|---------|
| **API** | [`api/`](./api/) | Programmatic setup: API keys, SDKs, and first calls (TTS, voices, etc.). |
| **Dashboard** | [`dashboard/`](./dashboard/) | Manual setup in the ElevenLabs web UI: account, voices, projects, and settings. |

## Integrations (Exotel + ElevenLabs)

**Two approaches:** [integrations/README.md](./integrations/README.md) — **SIP trunk** vs **WebSocket (Voicebot + bridge)**.

| Integration | Path |
|-------------|------|
| **SIP trunk (vSIP)** — native ElevenLabs SIP | [`integrations/exotel-vsip/`](./integrations/exotel-vsip/) — [Quickstart](./integrations/exotel-vsip/QUICKSTART.md) · [Voice AI Connector](./integrations/exotel-vsip/elevenlabs-voice-ai-connector.md) |
| **WSS** — Exotel Voicebot → your bridge → ElevenLabs ConvAI WS | [`integrations/exotel-wss/`](./integrations/exotel-wss/) (see [Agent-Stream](https://github.com/exotel/Agent-Stream) for Exotel streaming reference) |

Start with whichever fits your workflow; you can combine both (e.g. configure voices in the dashboard, then call them from code via the API).
