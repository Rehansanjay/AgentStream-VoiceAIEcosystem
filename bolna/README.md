# Bolna AI (Voice AI Ecosystem)

Documentation for integrating **[Bolna AI](https://www.bolna.ai/)** with telephony in this workspace.

> **India market compliance:** For India deployments, keep **SIP signaling and media** on paths that remain **in India** (India-region telephony edges and, where the provider offers it, India or in-country data residency). If a Voice AI provider’s SIP signaling or media endpoints are **outside India**, prefer a **self-hosted** deployment of that stack in an **India region** (or an equivalent India-resident architecture) so signaling and media stay in-country. Confirm residency and lawful-interception requirements with your compliance team and with Exotel before go-live.

| Integration | Path |
|-------------|------|
| **Exotel vSIP + Bolna** (SIP BYOT) | [`integrations/exotel-vsip/README.md`](integrations/exotel-vsip/README.md) |

Official Bolna references:

- [Bring Your Own SIP Trunk](https://www.bolna.ai/docs/sip-trunking/introduction)
- [Connect Exotel account (dashboard)](https://www.bolna.ai/docs/exotel-connect-provider)
- [Create SIP Trunk API](https://www.bolna.ai/docs/api-reference/sip-trunks/create)
