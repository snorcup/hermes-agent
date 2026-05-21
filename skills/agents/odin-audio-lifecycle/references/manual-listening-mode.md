# Manual Listening Mode (2026-05-20)

## Problem
Uploads to `/transcribe` and `/chunk` returned 409 even after user clicked "Start" because `SIDCAR_REQUIRE_ACTIVE_SESSION` blocked transcription when no Odin session was active.

## Solution
Introduced `capture.manual_mode` flag:

- `/start` sets `capture.manual_mode = True`
- Transcription checks now allow uploads when `manual_mode` is active:
  ```python
  if (SIDCAR_REQUIRE_ACTIVE_SESSION and not ctx.get("allowed") and not capture.manual_mode):
      return 409
  ```
- `sync_capture_with_odin()` skips auto-stop when `manual_mode` is set.

## Result
Users can now explicitly enable listening via the web UI or Signal commands ("start listening") without requiring an active Odin session. The sidecar respects the user's intent while still protecting automatic/background capture.