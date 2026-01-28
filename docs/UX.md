# UX Spec (MVP)

## Overlay window
- Always-on-top
- Transparent background
- Click-through: optional (toggle)
- Draggable when not click-through
- Position persisted

## Caption rendering
- Two lines: EN (top), JA (bottom)
- Max width: wrap by window width
- Duration: base 4s + (len/15)s up to 10s (tunable)
- Fade-out: last 0.5s
- Queue:
  - If a new caption arrives, replace current or stack (MVP: replace)

## Hotkeys (MVP)
- Toggle overlay visible
- Toggle translation processing (mute)
- Toggle click-through
