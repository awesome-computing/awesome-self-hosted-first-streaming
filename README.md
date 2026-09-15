# Awesome Self-Hosted-First Streaming [![Awesome](https://awesome.re/badge.svg)](https://awesome.re) [![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md) [![License: CC0-1.0](https://img.shields.io/badge/License-CC0%201.0-lightgrey.svg)](LICENSE)

> Run your own ingest/transcode/delivery stack before reaching for a managed streaming SaaS. Justify the vendor.

Part of the [awesome-computing](https://github.com/awesome-computing) family — see the [org README](https://github.com/awesome-computing/.github) for the pattern this list follows: a clear thesis, honest decision heuristics, curated real resources, and a section on when to break the rule.

**Licensing note:** every software resource below is tagged with its actual license — `MIT`, `Apache-2.0`, `AGPL-3.0`, etc. Anything not genuinely open source is tagged `(proprietary)` explicitly rather than implied. Open *specs/protocols* (RTMP, HLS, DASH) aren't software and aren't license-tagged the same way — they're marked `(open spec)`.

## Contents

- [Introduction](#introduction)
- [Decision Heuristics](#decision-heuristics)
- [Ingest Protocols & Servers](#ingest-protocols--servers)
- [Modern Ingest Alternatives](#modern-ingest-alternatives)
- [Transcoding & Packaging](#transcoding--packaging)
- [Delivery Formats](#delivery-formats-open-specs)
- [Players](#players)
- [Full Self-Hosted Platforms](#full-self-hosted-platforms)
- [What You're Avoiding (the proprietary side)](#what-youre-avoiding-the-proprietary-side)
- [When You Actually Need Managed SaaS](#when-you-actually-need-managed-saas)
- [Further Reading](#further-reading)

## Introduction

Live and on-demand video streaming has a full open-protocol, open-source stack available at every stage: ingest (RTMP, SRT, WHIP), transcode/package (FFmpeg, GStreamer, Shaka Packager), delivery (HLS, LL-HLS, DASH, CMAF — all open specs), and playback (hls.js, Shaka Player, video.js, Media3/ExoPlayer). None of it is exotic; RTMP servers and FFmpeg pipelines have been production-grade for over a decade.

Despite that, teams often reach straight for a managed streaming API — Mux, Cloudflare Stream, AWS IVS, Wowza Cloud — even for workloads a self-hosted FOSS stack would handle comfortably, trading cost-at-scale, data ownership, and architectural control for convenience. That trade is sometimes the right one. This list exists to make it a deliberate choice instead of a default.

RTMP itself deserves a specific note: Adobe stopped actively developing Flash long ago and killed the player in 2020, but RTMP as an *ingest* protocol (camera/encoder → server) never went away — it's still what OBS, most hardware encoders, and most streaming platforms speak by default, even though playback moved entirely to HLS/DASH/WebRTC years earlier. "RTMP is dead" is a playback-side statement, not an ingest-side one.

## Decision Heuristics

**Self-host wins when:**
- Traffic is reasonably predictable, or you're comfortable provisioning for headroom yourself.
- Egress/transcode costs at your scale would meaningfully exceed a self-hosted stack's infrastructure + ops cost.
- You need full control over stream data — compliance, privacy, or contractual requirements that a third-party API complicates.
- You have (or are willing to build) real-time/low-latency requirements a generic managed API doesn't tune for.
- Your team already has the ops capacity (containers, load balancers, storage/CDN integration) to run production services.

**Managed SaaS wins when:**
- You're a small team without video-ops experience and shipping speed matters more than unit economics.
- Traffic is highly bursty/unpredictable (e.g. viral spikes) and you need instant global capacity without pre-provisioning.
- You need turnkey multi-DRM, enterprise compliance certifications, or global anycast delivery today, not after months of building it.
- Video is a small part of your product, not something worth a dedicated ops investment.

## Ingest Protocols & Servers

- **RTMP** (open, legacy-but-dominant ingest protocol; Adobe-originated, never formally an open standard body spec, but ubiquitously and openly implemented) *(open spec)*
  - [nginx-rtmp-module](https://github.com/arut/nginx-rtmp-module) `(BSD-2-Clause)` — the classic self-hosted RTMP ingest/relay module for nginx.
  - [SRS (Simple Realtime Server)](https://github.com/ossrs/srs) `(MIT)` — RTMP/WebRTC/SRT/HLS all-in-one media server, widely used in production.
  - [MediaMTX](https://github.com/bluenviron/mediamtx) `(MIT)` — ready-to-use RTMP/RTSP/HLS/WebRTC/SRT server and proxy, zero external dependencies.
  - [OvenMediaEngine](https://github.com/AirenSoft/OvenMediaEngine) `(AGPL-3.0)` — sub-second latency streaming server (WebRTC/LLHLS/SRT ingest and delivery).
  - [Ant Media Server](https://github.com/ant-media/Ant-Media-Server) `(open-core — Community Edition source available on GitHub; Enterprise is proprietary; verify current license terms before depending on it)`.

## Modern Ingest Alternatives

- [SRT (Secure Reliable Transport)](https://github.com/Haivision/srt) `(MPL-2.0)` — low-latency, packet-loss-resilient transport designed as a modern RTMP successor for contribution/ingest.
- **WHIP (WebRTC-HTTP Ingestion Protocol)** *(open spec, IETF)* — standardizes WebRTC as an ingest protocol; implemented by MediaMTX, OvenMediaEngine, and others above.
- **RIST (Reliable Internet Stream Transport)** *(open spec, RIST Forum)* — another modern, loss-resilient contribution protocol used in broadcast contexts.

## Transcoding & Packaging

- [FFmpeg](https://github.com/FFmpeg/FFmpeg) `(LGPL-2.1+, or GPL-2.1+ if built with --enable-gpl components)` — the de facto standard transcoding/muxing toolkit; almost everything else on this list shells out to it or embeds its libraries.
- [GStreamer](https://github.com/gstreamer/gstreamer) `(LGPL-2.1+)` — pipeline-based multimedia framework, common in embedded/hardware-accelerated transcoding.
- [Shaka Packager](https://github.com/shaka-project/shaka-packager) `(BSD-3-Clause, Google)` — packages media into DASH/HLS-compatible fragmented MP4/CMAF, including DRM encryption.

## Delivery Formats (open specs)

- **HLS (HTTP Live Streaming)** *(open spec, Apple-originated, now IETF RFC 8216)* — the most widely supported delivery format across devices.
- **LL-HLS (Low-Latency HLS)** *(open spec extension)* — sub-few-second latency variant of HLS.
- **MPEG-DASH** *(open ISO/IEC standard)* — vendor-neutral adaptive streaming standard, the non-Apple counterpart to HLS.
- **CMAF (Common Media Application Format)** *(open packaging standard)* — lets one set of encoded segments serve both HLS and DASH, cutting storage/transcode duplication.

## Players

- [hls.js](https://github.com/video-dev/hls.js) `(Apache-2.0)` — HLS playback in any browser via Media Source Extensions.
- [Shaka Player](https://github.com/shaka-project/shaka-player) `(Apache-2.0, Google)` — HLS + DASH playback with DRM support.
- [video.js](https://github.com/videojs/video.js) `(Apache-2.0)` — the standard open-source HTML5 video player framework, HLS/DASH via plugins.
- [AndroidX Media3 / ExoPlayer](https://github.com/androidx/media) `(Apache-2.0, Google)` — Android's open-source media playback library.

## Full Self-Hosted Platforms

- [Owncast](https://github.com/owncast/owncast) `(MIT)` — a complete self-hosted live streaming server with a built-in web player and chat; the closest thing to a drop-in FOSS Twitch alternative.
- [PeerTube](https://github.com/Chocobozzz/PeerTube) `(AGPL-3.0)` — federated (ActivityPub), self-hosted video hosting; VOD-focused FOSS alternative to YouTube/Vimeo, with live-streaming support too.
- [OBS Studio](https://github.com/obsproject/obs-studio) `(GPL-2.0)` — the open-source broadcast/encoder client that feeds RTMP/SRT/WHIP into everything above; the ingest side starts here for most self-hosted setups.

## What You're Avoiding (the proprietary side)

These are legitimate products and the right call under the "when managed SaaS wins" heuristics above — but they're closed, proprietary, hosted services, not something you can inspect, self-host, or audit:

- **Mux** `(proprietary)` — managed video API (encoding, storage, delivery, analytics).
- **Cloudflare Stream** `(proprietary)` — managed video API built on Cloudflare's network.
- **AWS IVS (Interactive Video Service)** `(proprietary)` — managed low-latency live streaming.
- **Wowza Streaming Cloud** `(proprietary)` — managed video API. (Note: Wowza also sells *Wowza Streaming Engine*, self-hostable but commercially licensed — not FOSS either, despite being something you run yourself.)

## When You Actually Need Managed SaaS

Be honest about this instead of treating self-hosting as always correct:

- You're pre-product-market-fit and video infrastructure isn't your differentiator — every hour spent on transcoder ops is an hour not spent on the product.
- Your traffic pattern is genuinely unpredictable at a scale where under-provisioning means an outage during your highest-value moment (e.g. a live event going viral).
- You need certified DRM/compliance (e.g. major studio content licensing requirements) that would take a dedicated team months to implement correctly in-house.
- You're a one- or two-person team and the operational surface of running FFmpeg farms, storage, and CDN integration reliably at 3am is not a trade you want.

## Further Reading

- [Adobe's official end-of-life notice for Flash Player](https://www.adobe.com/products/flashplayer/end-of-life.html) — useful for understanding why "RTMP is legacy" is a playback-side claim, not an ingest-side one.
- [Apple's HLS specification (RFC 8216)](https://datatracker.ietf.org/doc/html/rfc8216) — the open spec underlying most modern delivery.
- [CMAF overview](https://www.iso.org/standard/79106.html) — ISO/IEC 23000-19, the standard behind unified HLS/DASH packaging.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). New entries must be genuinely open source (include an accurate license tag) or, if proprietary, must go in the explicitly-labeled proprietary section — no unlabeled vendor listings.

## License

[CC0 1.0 Universal](LICENSE) — public domain dedication, consistent with the rest of the awesome-computing org.
