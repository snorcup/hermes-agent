---
name: odin-termux-edge
description: "Use when operating Odin's Pixel/Termux edge agent: odin_edge_agent.py, Termux microphone capture, whisper.cpp, opus/ogg chunks, wake locks, SSH, boot scripts, and edge health checks."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [odin, termux, android, pixel, whispercpp, audio, tailscale]
    related_skills: [odin-agent, odin-audio-lifecycle, odin-infrastructure]
---

# Odin Termux Edge

## Overview

Odin's primary live audio path runs on the Pixel 9 Pro in Termux. The edge agent records mic audio locally, transcribes with whisper.cpp, then POSTs transcripts to Odin ingest on the agent host.

Pipeline:

```text
Pixel mic -> odin_edge_agent.py -> whisper.cpp -> POST /ingest to agent host
```

## When to Use

Use this skill for:

- Pixel 9 Pro / Termux edge-agent operation
- `odin_edge_agent.py` health, start/stop, or deployment
- `termux-microphone-record` behavior
- whisper.cpp model/runtime issues
- Opus/Ogg audio format choices
- Termux SSH, boot scripts, wake locks, and Tailscale access

Use `odin-audio-lifecycle` instead for high-level Odin listening state and manual command semantics.

## Current Edge Defaults

From the known Odin setup:

- Edge IP: `100.86.74.48`
- Edge service port: `8765`
- Backend: whisper.cpp
- Recommended encode flag: `-e opus`, producing `.ogg`
- Ingest target: agent host `/ingest` endpoint, commonly port `8473`
- Medium/tiny whisper.cpp models are viable on CPU; large-v3 is too slow for routine edge use.

## Correct Recording Pattern

Do not rely on `os.setsid` / `os.killpg` on Termux/Android. Process groups are unreliable there. Use the Termux API stop command:

```python
proc = subprocess.Popen(
    f"termux-microphone-record -f '{path}' -e opus",
    shell=True,
    stdout=subprocess.DEVNULL,
    stderr=subprocess.DEVNULL,
)

time.sleep(duration)

subprocess.run(["termux-microphone-record", "-q"], capture_output=True, timeout=10)
proc.wait(timeout=5)
time.sleep(1.5)  # filesystem sync before validating file size
```

Key rules:

- Use `-e opus`; whisper.cpp handles `.ogg` well.
- The `-l N` flag does not reliably make the command return; explicitly stop with `-q`.
- Clean up stale recordings with `termux-microphone-record -q` before starting a new agent.
- Use `termux-wake-lock` if listening must survive screen-off behavior.

## SSH / Process Pitfalls

- Default Termux sshd port is `8022`.
- Termux username follows `u0_aXXX`.
- Host keys may rotate after reinstall; clear `[ip]:8022` from known_hosts.
- Avoid broad `pkill -f` over SSH; it can kill the SSH session itself. Prefer a remote restart script launched with `nohup` and exit cleanly.

## Performance Notes

- Medium model on Pixel CPU can take roughly as long to transcribe as the recorded chunk duration.
- First chunk may be slower due to model loading.
- Short chunks reduce latency but increase overhead. Around 30 seconds is a known stable tradeoff.

## Reference Files

- `references/termux-audio-recording.md`
- `references/termux-edge-receiver.md`
- `references/edge-receiver-implementation.md`
- `references/termux-ssh-keyauth-boot.md`
- `references/ssh-key-setup.md`

## Verification Checklist

- [ ] Edge health endpoint responds.
- [ ] Stale microphone capture was stopped before restart.
- [ ] Recording uses `-e opus` and explicit `termux-microphone-record -q` stop.
- [ ] Transcripts POST successfully to Odin ingest.
- [ ] Wake lock/boot behavior matches the requested persistence level.
