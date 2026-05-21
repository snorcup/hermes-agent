# Transcript Ingestion Service Pattern (2026-05-20)

## Context
When extending the Odin sidecar with live transcription, we chose to create a separate lightweight Flask microservice (`odin_ingest.py`) rather than adding the ingestion logic directly into `odin_bot.py`.

## Why separate service
- Keeps the main constitutional bot focused on Signal + session lifecycle
- Easier to restart / scale the ingestion path independently
- Reduces risk of breaking the main bot when modifying audio handling
- Simple surface: one POST endpoint + health check

## Recommended Structure
- Port: 8765 (internal / Tailscale only)
- Endpoint: `POST /ingest_transcript`
- Auth: Bearer token (reuse `ODIN_INGEST_TOKEN` or `ODIN_SIDCAR_TOKEN`)
- Payload: `{ "transcript": "...", "session_id": "..." }`
- Side effects: insert into `session_transcripts` + update `audio_observations` in active session state

## Deployment
```bash
# Run via the Odin venv
/root/.local/share/odin/venv/bin/python /root/.local/share/odin/odin_ingest.py
```

Consider adding a systemd unit (`odin-ingest.service`) for production.

## Sidecar Integration
The sidecar should POST new transcript segments to the ingest service as soon as they are produced (live mode) or at the end of a listening session.

This pattern is preferred over direct DB writes from the sidecar.