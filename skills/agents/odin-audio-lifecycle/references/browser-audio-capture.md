# Browser Audio Capture + HTTPS on Tailscale

## Pattern
- Flask sidecar + MediaRecorder in browser
- HTTPS required for getUserMedia on Tailscale hosts
- Chunks posted to `/upload` with Bearer token
- In-memory buffer until transcription pipeline is attached

## Tailscale Cert Setup
```bash
tailscale cert still.tail8a8496.ts.net
cp /var/lib/tailscale/certs/still.* /home/agent/odin-sidecar/
chown agent:agent *.key
chmod 600 *.key
```

## Common Failure
PermissionError on `load_cert_chain` when using paths under `/var/lib/tailscale/certs/`.

## LM Studio Fallback
Once a Whisper model is loaded in LM Studio, call:
`POST http://127.0.0.1:1234/v1/audio/transcriptions`

This keeps heavy inference off the sidecar host.