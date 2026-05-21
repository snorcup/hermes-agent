---
name: odin-audio-lifecycle
description: "Use when changing or debugging Odin audio/listening behavior: session lifecycle transitions, automatic/manual listening, browser MediaRecorder capture, transcript ingest, terminology, or 409/manual-mode fixes."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [odin, audio, listening, lifecycle, browser, ingest]
    related_skills: [odin-agent, odin-termux-edge, odin-infrastructure]
---

# Odin Audio Lifecycle

## Overview

Odin audio is lifecycle-driven. The user-facing concept is "listening" rather than "recording". Automatic lifecycle behavior should cover normal sessions, while manual commands must remain available and must not be blocked by active-session checks.

## When to Use

Use this skill for:

- `idle` / `pre_session` / `standby` / `active` / `post_session` transitions
- automatic start/stop listening behavior
- manual `start listening` / `stop listening` command handling
- browser MediaRecorder sidecar capture
- transcript ingest service behavior
- terminology and user-facing audio UX
- fixing manual-start 409 failures

Use `odin-termux-edge` instead for Pixel/Termux recording mechanics or whisper.cpp details.

## Lifecycle Rules

- Entering `standby` or `active` starts listening automatically.
- Entering `idle` or `post_session` stops listening automatically.
- Manual commands override automatic behavior:
  - start: `start listening`, `begin listening`, `start audio`
  - stop: `stop listening`, `end listening`, `stop audio`
- Manual starts must set `capture.manual_mode = True` or equivalent so active-session requirements are bypassed.

## Browser Capture Notes

Browser capture is for desktop/laptop mic input. Requirements:

- HTTPS is required for browser mic permission unless on localhost.
- Tailscale certs are preferred for private-network HTTPS.
- MediaRecorder chunk interval around 2000ms works well.
- Auth must be sent on every upload.
- Chunks can be buffered in memory and transcribed/forwarded asynchronously so `/upload` stays responsive.

## Ingest Pattern

Prefer a lightweight ingest service for transcript intake instead of bloating the main bot. Edge/browser components POST transcripts to the ingest endpoint, which correlates them with the current Odin session and forwards relevant text into Odin.

## Common Pitfalls

1. Requiring an active session for explicit manual listening starts. This causes user-visible 409 failures and breaks intentional manual override.
2. Calling the feature "recording" in UX. Use "listening".
3. Blocking upload handlers on transcription. Use background workers/threads where possible.
4. Assuming browser mic works over plain HTTP on non-localhost. It usually will not.
5. Mixing browser sidecar behavior with Pixel edge recording mechanics; they are separate paths.

## Reference Files

- `references/audio-sidecar-integration.md`
- `references/manual-listening-mode.md`
- `references/session-lifecycle-audio.md`
- `references/odin-audio-lifecycle.md`
- `references/browser-audio-capture.md`
- `references/transcript-ingestion-service.md`
- `references/terminology.md`
- `references/web-ui-control.md`

## Verification Checklist

- [ ] Lifecycle starts/stops listening in the expected states.
- [ ] Manual start bypasses active-session requirements.
- [ ] Manual stop always works.
- [ ] User-facing text says "listening".
- [ ] Browser capture uses HTTPS and authenticated uploads.
