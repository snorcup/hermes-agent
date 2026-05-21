# Edge Receiver Implementation

Current receiver running on Pixel 9 Pro (`100.86.74.48`).

## Endpoints

- `POST /speak` — `{ "text": "..." }` → Termux TTS
- `POST /play` — `{ "file": "..." }` or `{ "url": "..." }` → termux-media-player
- `POST /stop` — Stop current playback
- `GET /status` — Returns `{speaking, playing, last_text}`
- `GET /health` — Basic health check

## Boot Script

`~/.termux/boot/start-edge-receiver.sh` uses `nohup` + `disown`.

## Notes

- Requires Termux:API app installed
- Flask must be installed via pip
- Port 8765 (chosen to avoid conflict with other services)