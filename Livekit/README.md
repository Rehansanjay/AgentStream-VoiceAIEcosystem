# LiveKit integrations

This folder contains LiveKit-related integration guides and examples.

> **India market compliance:** For India deployments, keep **SIP signaling and media** on paths that remain **in India** (India-region telephony edges and, where the provider offers it, India or in-country data residency). If a Voice AI provider’s SIP signaling or media endpoints are **outside India**, prefer a **self-hosted** deployment of that stack in an **India region** (or an equivalent India-resident architecture) so signaling and media stay in-country. Confirm residency and lawful-interception requirements with your compliance team and with Exotel before go-live.

## Exotel vSIP

- Overview: [`Livekit/integrations/exotel-vsip/README.md`](./integrations/exotel-vsip/README.md)
- Quickstart: [`Livekit/integrations/exotel-vsip/QUICKSTART.md`](./integrations/exotel-vsip/QUICKSTART.md)
- Support article: [`docs/support/exotel-livekit-sip-trunk.md`](../docs/support/exotel-livekit-sip-trunk.md)

## Example project

- Outbound caller sample + Exotel troubleshooting: [`Livekit/livekit-outbound-caller-agent/`](./livekit-outbound-caller-agent/)

