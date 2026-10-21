# Refresh CA stores after the eMudhra intermediate compromise

**Date:** 2026-10-21
**Type:** infra
**Version:** v1.7.1
**Author:** claude-code

## What changed

`certifi` in `uv.lock` moves from 2026.4.22 to 2026.10.19, and the image is rebuilt so
apt pulls Debian's `ca-certificates 20250419+deb13u1`. No code change.

## Why

eMudhra's intermediate CAs were compromised on 2026-10-20. `requests`/`httpx` verify
against certifi, not the system store, and certifi only dropped `emSign ECC Root CA - G3`
in 2026.10.18. A rebuild alone would have kept the old certifi because the lockfile pins it.
The Debian package also distrusts the Sectigo `AAA Certificate Services` cross-sign.

The Playwright Chromium build carries its own root store and is not changed by this.

## Related

- fisherevans/nottingham-cloud#800
