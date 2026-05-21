# Odin Audio Lifecycle Integration

How audio listening and edge output were tied to Odin's session states.

## Automatic Behavior

- `standby` or `active` → Start listening + enable edge output
- `idle` or `post_session` → Stop listening + stop edge audio

## Manual Override

Users can still issue:
- `start listening` / `stop listening`
- `edge speak "text"`
- `edge play "file.mp3"`

## Implementation Notes

- Added `set_session_lifecycle()` helper
- Replaced direct `state.lifecycle =` assignments
- Kept manual commands as overrides

This pattern keeps command surface low while retaining user control.