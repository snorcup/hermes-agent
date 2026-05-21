---
name: odin-agent
description: "Use when working on Odin, Stephen/Amy's Signal-based constitutional agent: bot behavior, session lifecycle, audio/listening, Pixel/Termux edge transcription, or Odin-specific infrastructure. Do not use for generic Hermes Agent, generic Signal MCP, or unrelated Tailscale/Termux tasks."
version: 2.1.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [odin, signal, agent, lifecycle, audio, termux]
    related_skills: [odin-audio-lifecycle, odin-termux-edge, odin-infrastructure, signal-mcp]
---

# Odin Agent

## Overview

Odin is Stephen/Amy's Signal-first constitutional AI agent. This skill is intentionally small: use it as the Odin router and invariant sheet, then load a narrower Odin skill for subsystem details.

Current architecture:

- Main Odin bot runs server-side and speaks through Signal.
- Session lifecycle states (`idle`, `pre_session`, `standby`, `active`, `post_session`) govern behavior and audio/listening permissions.
- Pixel 9 Pro / Termux edge agent is the primary live audio path: Pixel mic -> `odin_edge_agent.py` -> whisper.cpp -> Odin ingest on the agent host.
- Browser MediaRecorder sidecar remains relevant for desktop/laptop browser mic input.
- still-based Whisper offload is deprecated and only for legacy inspection.

## When to Use

Use this skill when the task explicitly concerns:

- Odin bot behavior, lifecycle, or Signal command handling
- Odin audio/listening behavior
- Pixel/Termux/whisper.cpp edge transcription for Odin
- Odin-specific Tailscale, sidecar, systemd, or deployment infrastructure
- Stephen/Amy Odin consent/session behavior

Do not use for:

- Generic Hermes Agent configuration or troubleshooting
- Generic Signal MCP usage unrelated to Odin
- Generic Termux, Tailscale, or Whisper tasks unrelated to Odin

## Core Invariants

- Say "listening", not "recording", in user-facing Odin audio UX.
- Audio listening starts automatically in `standby` or `active` and stops in `idle` or `post_session`.
- Explicit manual starts (`start listening`, `begin listening`, `start audio`) must bypass active-session requirements by setting manual mode; avoid 409 failures for intentional manual control.
- Pixel edge transcription is the primary path. Do not resurrect still-based Whisper offload unless the user explicitly asks for legacy debugging.
- Prefer pull over push for Amy-facing briefings/info. Odin should not proactively push sensitive interpersonal analysis unless asked.
- Preserve informed consent in structured roleplay/power-dynamic behavior.

## Related Skills

- `odin-audio-lifecycle` — session lifecycle, listening state, browser capture, ingest service, terminology.
- `odin-termux-edge` — Pixel 9 Pro, Termux, whisper.cpp, microphone recording, SSH, boot scripts.
- `odin-infrastructure` — Odin services, deployment, Tailscale certs, deprecated still paths, health checks.
- `signal-mcp` — generic Signal MCP/server operations outside Odin-specific behavior.

## Verification Checklist

- [ ] Loaded the narrow Odin subsystem skill if the task needs implementation details.
- [ ] Did not follow deprecated still-sidecar instructions unless explicitly debugging legacy setup.
- [ ] Preserved manual-listening bypass behavior.
- [ ] Kept Odin-specific assumptions out of generic Hermes/Signal/Tailscale answers.
