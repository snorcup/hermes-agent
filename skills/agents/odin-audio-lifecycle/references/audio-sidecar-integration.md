# Audio Sidecar Integration (2026-05-20)

## Pattern Established

- Automatic audio start/stop tied to session lifecycle via `set_session_lifecycle()` helper.
- Manual override commands remain available for Stephen/Amy.
- Terminology standardized to "listening" only.

## Key Functions Added

- `start_audio_capture(session_id=None)`
- `stop_audio_capture()`
- `set_session_lifecycle(state, new_lifecycle, session_id=None)`

## Command Surface (Minimized)

Only these phrases trigger manual control:

**Start listening:**
- start listening
- begin listening
- start audio
- begin audio

**Stop listening:**
- stop listening
- end listening
- stop audio
- end audio

## Lifecycle → Audio Mapping

| Lifecycle       | Audio Behavior          |
|-----------------|-------------------------|
| standby / active | Listening starts       |
| idle / post_session | Listening stops     |

## Future Extension Points

- Add `/status` endpoint call for "listening status" command.
- Allow per-session audio settings stored in `active_session` dict.