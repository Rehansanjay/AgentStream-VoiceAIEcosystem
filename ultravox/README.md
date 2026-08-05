# Voice AI Ecosystem: Ultravox

**[Ultravox](https://www.ultravox.ai/)** is a real-time voice AI platform. For **telephony**, Ultravox supports a **native Exotel** call medium (Voice Streaming), a **generic SIP** integration, and other providers — see [Telephony overview](https://docs.ultravox.ai/telephony/overview).

> **India market compliance:** For India deployments, keep **SIP signaling and media** on paths that remain **in India** (India-region telephony edges and, where the provider offers it, India or in-country data residency). If a Voice AI provider’s SIP signaling or media endpoints are **outside India**, prefer a **self-hosted** deployment of that stack in an **India region** (or an equivalent India-resident architecture) so signaling and media stay in-country. Confirm residency and lawful-interception requirements with your compliance team and with Exotel before go-live.

| Topic | Doc |
|-------|-----|
| **Exotel vSIP + Ultravox (SIP BYOT)** | [`integrations/exotel-vsip/`](./integrations/exotel-vsip/) |
| **Support article** | [`docs/support/exotel-ultravox-sip-trunk.md`](../docs/support/exotel-ultravox-sip-trunk.md) |

## Official Ultravox docs

- [SIP guide](https://docs.ultravox.ai/telephony/sip) — incoming (allowlist / registration), outgoing `medium.sip`, codecs, billing notes  
- [Telephony platforms](https://docs.ultravox.ai/telephony/telephony-platforms) — native `exotel` medium vs SIP  
- [Get SIP configuration](https://docs.ultravox.ai/api-reference/sip/sip-get) — account SIP `domain`, `allowedCidrRanges`

## Publishing

Do not commit live API keys or SIP passwords.
