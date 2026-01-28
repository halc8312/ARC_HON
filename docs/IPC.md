# IPC Spec (MVP)

## Overview
C#が音声を取得し、Python推論サービスへPCMをストリーミング送信。
PythonはVADで発話区間を切り出し、ASR→翻訳後、字幕JSONをC#へ送る。

## Pipe Names (default)
- Audio:  \\.\pipe\tarkov-sub-audio-v1
- Captions: \\.\pipe\tarkov-sub-captions-v1

## Audio Stream Format
- PCM signed 16-bit little endian
- Mono
- Sample rate: 16000 Hz
- Framing: 20 ms frames
  - samples_per_frame = 320
  - bytes_per_frame = 640
- Transport:
  - Raw bytes only (no headers). Receiver reads exactly 640 bytes per frame.

## Captions Messages (JSON Lines)
- Pipe sends UTF-8 text, one JSON object per line (`\n`).
- Schema:
  - type: "caption"
  - start_ms, end_ms: int (timeline in receiver, best-effort)
  - en: string
  - ja: string
  - final: bool (MVP always true)
  - confidence: float? (optional)
  - meta: object? (optional)

Example:
{"type":"caption","start_ms":12000,"end_ms":15600,"en":"hold your fire","ja":"撃たないで","final":true}
