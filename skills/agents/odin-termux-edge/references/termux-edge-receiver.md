# Termux Edge Receiver (Odin + Pixel 9 Pro)

This is the receiver that ran on the Pixel 9 Pro to allow Odin to control audio output.

## Endpoints

- `POST /speak` — Text-to-speech via `termux-tts-speak`
- `POST /play` — Play sound via `termux-media-player`
- `POST /stop` — Stop playback
- `GET /status` — Current speaking/playing state
- `GET /health` — Basic health check

## Boot Script

```bash
~/.termux/boot/start-edge-receiver.sh
```

## Dependencies

- `pkg install python`
- `pip install flask`

## Notes

- Runs on port 8765 by default.
- Uses threading for non-blocking TTS and media playback.
- Designed to be called by the central Odin agent over Tailscale.