# Web UI for Manual Sidecar Control

Added simple responsive UI to odin_sidcar.py on 2026-05-20.

## Trigger
User requested QR code that leads to a page capable of starting audio capture (manual listening).

## Implementation
- Added root `/` route serving inline HTML (no external templates).
- UI includes:
  - Live status box (active/idle, manual_mode, chunk count)
  - Start Listening button → POST /start (sets manual_mode=True)
  - Stop Listening button → POST /stop
  - Refresh Status button
- JavaScript uses fetch with Authorization: Bearer <token>
- Token is currently hardcoded in the page (update from .env on still when rotating).

## Deployment
1. Overwrite `/home/agent/odin-sidecar/odin_sidcar.py` on the target host (still).
2. `systemctl restart odin-sidecar.service`
3. Verify with `systemctl status odin-sidecar.service`

## QR Usage
Generate QR for `http://still.tail8a8496.ts.net:8473/` (or equivalent Tailscale name).
Scanning opens the control page directly.

## Future Improvements
- Pull token from server-side instead of hardcoding.
- Add real Whisper streaming once placeholder transcribe endpoint is replaced.
- Dark theme already applied; mobile-first layout.

This pattern turns the sidecar into a quick manual override tool independent of main Odin session state.