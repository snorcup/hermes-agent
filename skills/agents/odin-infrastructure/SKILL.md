---
name: odin-infrastructure
description: "Use when deploying, inspecting, or troubleshooting Odin infrastructure: systemd services, Tailscale certs, sidecars, service health checks, remote hosts, deprecated still Whisper paths, and Odin-specific network wiring."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [odin, infrastructure, systemd, tailscale, sidecar, deployment]
    related_skills: [odin-agent, odin-audio-lifecycle, odin-termux-edge]
---

# Odin Infrastructure

## Overview

This skill covers Odin-specific deployment and infrastructure. Keep it separate from generic Hermes Agent operations: Odin has its own bot service, audio/ingest sidecars, Tailscale endpoints, and historical still-based Whisper deployment notes.

## When to Use

Use this skill for:

- Odin service deployment or systemd troubleshooting
- Tailscale certificate setup for Odin sidecars
- Browser sidecar hosting and HTTPS wiring
- Odin ingest endpoint/network checks
- Remote host migration or deprecation cleanup
- Legacy still Whisper offload inspection

Use `odin-termux-edge` for Pixel/Termux mechanics and `odin-audio-lifecycle` for listening state behavior.

## Service Model

Common Odin services:

- `odin` — main bot service; restart after changes to main Odin bot code/config.
- `odin-sidcar.service` / sidecar variants — browser/audio sidecar paths; spelling may be historical.
- Edge agent — runs on Pixel/Termux, not as a normal Linux systemd service.

Always inspect live service names and logs before changing anything; Odin deployments have had historical naming drift.

## Tailscale HTTPS Pattern

For browser mic sidecars on private Tailscale hosts:

1. Generate certs with `tailscale cert <hostname>.tail8a8496.ts.net`.
2. Copy `.crt` and `.key` into the service/project directory.
3. Set private key permissions to `600`.
4. Load them with Python `ssl.SSLContext.load_cert_chain()` or equivalent.
5. Bind the sidecar to `0.0.0.0` only when network exposure is intended and access is controlled.

Pitfall: `/var/lib/tailscale/certs/` is usually not readable by the service user. Copy certs into the project directory rather than pointing the service at Tailscale's internal cert store.

## Deprecated still Whisper Path

The still-based Whisper offload sidecar is deprecated. The Pixel 9 Pro edge path is primary. Do not deploy new Odin transcription to still unless the user explicitly asks for legacy debugging or migration archaeology.

If inspecting legacy still setup:

- Check disk before downloading models; still has historically been space-constrained.
- Treat old `odin_sidcar.py` / still deployment docs as historical.
- Prefer disabling stale still sidecars once replacement is verified.

## Deployment Discipline

- Check current config, service files, and logs before editing.
- Verify health endpoints before and after restart.
- Avoid broad kill patterns on remote/Termux hosts.
- Keep secrets in env files, not committed docs.
- Record exact hostnames, ports, and service names in notes when they differ from historical docs.

## Reference Files

- `references/still-whisper-deployment.md`
- `references/browser-audio-capture.md`
- `references/transcript-ingestion-service.md`
- `references/web-ui-control.md`

## Verification Checklist

- [ ] Correct host and service identified from live state.
- [ ] Logs checked before restart/change.
- [ ] Tailscale cert/key path readable by the service user.
- [ ] Health endpoint responds after change.
- [ ] Deprecated still path was not used for new work unless explicitly requested.
