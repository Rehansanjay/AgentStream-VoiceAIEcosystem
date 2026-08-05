# Retell AI (Voice AI Ecosystem)

Documentation for integrating **[Retell AI](https://www.retellai.com/)** with telephony in this workspace.

> **India market compliance:** For India deployments, keep **SIP signaling and media** on paths that remain **in India** (India-region telephony edges and, where the provider offers it, India or in-country data residency). If a Voice AI provider’s SIP signaling or media endpoints are **outside India**, prefer a **self-hosted** deployment of that stack in an **India region** (or an equivalent India-resident architecture) so signaling and media stay in-country. Confirm residency and lawful-interception requirements with your compliance team and with Exotel before go-live.

| Integration | Path |
|-------------|------|
| **Exotel vSIP + Retell** (SIP trunk) | [`integrations/exotel-vsip/README.md`](integrations/exotel-vsip/README.md) |

Official Retell reference: [Custom telephony overview](https://docs.retellai.com/deploy/custom-telephony).
