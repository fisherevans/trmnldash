# Clear Dependabot alerts on the runtime Python deps

**Date:** 2026-10-21
**Type:** infra
**Version:** v1.7.2
**Author:** claude-code

## What changed

`uv.lock` bumps pillow 12.2.0 -> 13.0.0, aiohttp 3.13.5 -> 3.14.5, anyio 4.13.0 -> 4.14.2+,
cryptography 48.0.0 -> 50.0.2, httplib2 0.31.2 -> 0.32.0, pyasn1 0.6.3 -> 0.6.4, urllib3
2.7.0 -> 2.8.0, multidict 6.7.1 -> 7.1.0 and oauthlib 3.3.1 -> 4.0.0. `requires-python`
moves from `>=3.10` to `>=3.12`, which is what the image has always run. No code change.

## Why

Dependabot opened 60 alerts against `uv.lock`, all runtime scope
([nottingham-cloud#800](https://github.com/fisherevans/nottingham-cloud/issues/800)).
What was actually reachable, given `trmnl.fisher.sh` is public through the Cloudflare tunnel:

- **aiohttp** is the HTTP server (`engine/server.py`), so the server-side advisories
  (missing header timeout, multipart fd exhaustion, unbounded memory/CPU, pipelining, request
  smuggling) applied to an internet-facing listener. Cloudflare in front blunts the smuggling
  ones; the DoS ones were real. The client-side aiohttp advisories do not apply - outbound
  fetches use httpx.
- **pillow** only ever decodes the PNG our own Chromium produces, then quantizes it. None of
  the decoder advisories (JPEG, TIFF, GIF, PDF, EPS, fonts) are reachable from untrusted input.
- **anyio, urllib3, httplib2, pyasn1, cryptography, oauthlib** sit under httpx and the Google
  Calendar client, talking only to Open-Meteo, NWS, Home Assistant and Google. Low exposure.

The `requires-python` bump is what lets the lock carry a single pillow 13 resolution; with
`>=3.10` uv kept a pillow 12.3 fork for 3.10 that Dependabot would keep flagging.

Pillow 13 is a major bump, so every fixture (`data*.json` through `weather_landscape`, and the
OG `calendar_agenda` + `weather_compact` hstack at 2-bit) was rendered on both lockfiles and
diffed: pixel-identical. `trmnldash serve` was smoke-tested on aiohttp 3.14.5 (healthz 200,
PNG served, unknown path 404).
