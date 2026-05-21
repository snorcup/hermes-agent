# Termux Audio Recording — Correct Patterns

## The Problem

`os.setsid()` + `os.killpg()` does **not work** on Termux/Android. The kernel does not reliably create new process groups via `preexec_fn=os.setsid`, so `os.killpg()` fails silently or kills nothing. The recording process hangs indefinitely.

`termux-microphone-record -l N` sets a bitrate/quality limit but does **not** cause the process to auto-terminate after N seconds. The process blocks forever.

## Correct Recording Pattern (v2.2)

```python
def record_chunk(duration: int = 30) -> str | None:
    path = f"/data/data/com.termux/files/usr/tmp/odin_chunk_{int(time.time())}.ogg"
    try:
        proc = subprocess.Popen(
            f"termux-microphone-record -f '{path}' -e opus",
            shell=True,
            stdout=subprocess.DEVNULL,
            stderr=subprocess.DEVNULL,
        )
        state._recording_proc = proc

        time.sleep(duration)

        # STOP using the official Termux API — NOT killpg
        subprocess.run(["termux-microphone-record", "-q"],
                        capture_output=True, timeout=10)
        try:
            proc.wait(timeout=5)
        except subprocess.TimeoutExpired:
            proc.kill()  # fallback only

        state._recording_proc = None
        time.sleep(1.5)  # filesystem sync

        if os.path.exists(path) and os.path.getsize(path) > 1000:
            return path
        else:
            _safe_unlink(path)
            return None
    except Exception as e:
        state._recording_proc = None
        try:
            subprocess.run(["termux-microphone-record", "-q"],
                            capture_output=True, timeout=5)
        except Exception:
            pass
        _safe_unlink(path)
        return None
```

## Key Rules

1. **Always use `-q` to stop** — it's the only reliable way to stop `termux-microphone-record`
2. **Always clean up stale recordings** before starting a new agent session:
   ```bash
   termux-microphone-record -q 2>/dev/null
   ```
3. **Use `-e opus`** for `.ogg` output — whisper.cpp supports it natively, no ffmpeg conversion needed
4. **Sleep 1.5s after stop** — filesystem needs sync time before file validation
5. **Minimum file size check** — reject files < 1000 bytes (typically silence or failed recordings)
6. **Wakelock required** — use `termux-wake-lock -p audio` to prevent Android from killing the process during screen-off

## SSH Gotcha: pkill Kills SSH Sessions

Running `pkill -f "pattern"` over SSH on Termux can kill the SSH session itself if the pattern matches the SSH command. Workaround:

```bash
# DON'T: pkill -f "odin_edge" (kills SSH too)
# DO: write a restart script, run it detached
nohup bash ~/.odin/restart.sh > /dev/null 2>&1 &
```

The restart script handles its own cleanup:
```bash
#!/data/data/com.termux/files/usr/bin/bash
kill $(pgrep -f "python3 odin_edge") 2>/dev/null
kill $(pgrep -f "termux-microphone-record") 2>/dev/null
sleep 2
cd ~/.odin && source env.sh && exec python3 odin_edge_agent.py
```

## Whisper Timing (Pixel 9 Pro, CPU only)

| Chunk Duration | Whisper Transcription | Total Cycle |
|---|---|---|
| 12s | ~15-20s | ~30-35s |
| 30s | ~30-35s | ~60-65s |
| 60s | ~60-90s | ~120-150s |

First chunk after model load takes 3-4 minutes (model loading into memory).
Subsequent chunks stabilize to consistent cycle times.
`large-v3` is 3-4x slower than `medium` on CPU — not recommended for edge devices.

## Soak Test Receiver Pattern

Create a disposable Flask app on a non-production port for testing:

```python
# /tmp/test_audio_receiver.py — runs on :9473
# Logs all POSTs to /tmp/odin_audio_test.jsonl with timestamps and intervals
# GET /health returns uptime, chunk counts, last interval
# GET /results returns full log entries
```

Steps:
1. Start test receiver on agent host
2. Set `ODIN_INGEST_URL` on Pixel to `http://agent:9473/ingest`
3. Restart edge agent, trigger `/start`
4. Monitor `GET /health` on both sides every 5 minutes
5. After soak test, restore production URL and kill receiver
