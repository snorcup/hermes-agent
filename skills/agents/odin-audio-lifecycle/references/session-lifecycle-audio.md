# Session Lifecycle & Manual Listening

## Problem
When `SIDCAR_REQUIRE_ACTIVE_SESSION=1`, the sidecar rejects audio uploads (`/transcribe`, `/chunk`) with HTTP 409 if no active Odin session exists. Users clicking "Start" in the web UI get frustrated.

## Solution Implemented
- Added `manual_mode` flag to `SessionCapture`.
- `/start` endpoint now sets `manual_mode = True` on explicit user action.
- Transcription permission checks now include `and not capture.manual_mode`.
- `sync_capture_with_odin()` was updated to skip auto-stop when `manual_mode` is active.

## Result
Manual "start listening" always works. Automatic session-based control still applies when the user has not manually overridden it.

## Recommendation
Keep `SIDCAR_REQUIRE_ACTIVE_SESSION=1` for safety, but treat explicit user starts as higher priority.