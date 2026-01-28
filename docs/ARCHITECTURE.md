# Architecture: EN->JA Proximity VC Subtitle Overlay (Windows, MVP)

## Summary
Windowsの出力音声（ゲーム音）をWASAPIループバックで取得し、
VADで「人の声らしい区間」だけを切り出して ASR(Whisper) → 翻訳(en->ja) を行い、
WPFの透明最前面ウィンドウに字幕（EN/JA併記）を表示する。

- ゲームプロセスには一切注入しない（DLL injection / DirectX hook などしない）
- 完全ローカル（無料）：クラウド翻訳/音声APIは使わない
- MVPは「発話が終わったら字幕（finalのみ）」、将来「逐次（partial）」へ拡張可能


## Components

### 1) OverlayApp (C# / WPF)
Responsibilities:
- 音声キャプチャ：WASAPI loopback → 16kHz/mono/PCM16 に統一
- 推論サービスへ音声フレーム送信（NamedPipe）
- 推論サービスから字幕JSONを受信（NamedPipe）
- 字幕表示：透明・最前面・任意位置・2〜3発話のスタック表示
- ホットキー：
  - F8: オーバーレイ表示ON/OFF
  - F9: 翻訳/推論ON/OFF（ミュート）
  - F10: クリック透過ON/OFF（位置調整のため）

Non-responsibilities:
- VAD/ASR/翻訳ロジックは持たない（原則Python側）


### 2) InferenceService (Python)
Responsibilities:
- 音声フレーム受信（NamedPipe）
- VAD（Silero等）により発話区間を切り出し
- ASR（faster-whisper）で英語テキスト化
- 翻訳（ローカルモデル）で日本語化
- 字幕JSONをOverlayAppへ送信

Non-responsibilities:
- UI表示はしない


## Data Flow (MVP)

1. OverlayApp が WASAPI loopback で出力音声を取得
2. OverlayApp が 20ms フレーム（16kHz mono PCM16, 640 bytes）として Audio Pipe に送信
3. InferenceService が Audio Pipe を受信し、VADで音声区間をバッファリング
4. 無音が一定時間続いたら「1発話（segment）」として確定
5. segment を ASR → 翻訳
6. 字幕 JSON (final=true) を Captions Pipe に送信
7. OverlayApp が受信し、字幕スタック（最大3件）を更新表示


## IPC

### Named Pipes
Default:
- Audio: `\\.\pipe\tarkov-sub-audio-v1`
- Captions: `\\.\pipe\tarkov-sub-captions-v1`

Audio format:
- PCM signed 16-bit little endian
- mono, 16000 Hz
- fixed frames: 20ms = 320 samples = 640 bytes
- raw stream: receiver reads exactly 640 bytes repeatedly

Captions:
- UTF-8 JSON Lines (1 line = 1 JSON object)

Caption schema (MVP):
- type: "caption"
- start_ms: int (best-effort)
- end_ms: int (best-effort)
- en: string
- ja: string
- final: bool (MVP always true)
- meta: object (optional)

Example:
`{"type":"caption","start_ms":12000,"end_ms":15600,"en":"hold your fire","ja":"撃たないで","final":true}`


## Performance / Latency Strategy

### MVP latency target
- 「発話終了」から字幕表示まで 1〜2秒程度（ベストエフォート）
- 実際の遅延は VAD end_silence + ASR + 翻訳 の合計で決まる

### Future (low latency)
- partial字幕の導入（0.5〜1.0秒のチャンクでASR）
- 翻訳の差分更新、または短いウィンドウ単位で翻訳
- VOIP音声の分離（仮想オーディオでVCのみを別出力にする）で誤爆と精度を改善


## Error Handling & Safety
- 推論プロセスが落ちてもOverlayAppが落ちない（再接続できる設計が望ましい）
- 音声/字幕はデフォルトで保存しない
- デバッグ時のみログを増やせる（ただし個人情報/録音の扱いに注意）


## Configuration
- OverlayApp: `Settings.json` に保存（位置、フォント、ホットキー、スタック数など）
- InferenceService: `config.json` or env vars（VAD閾値、モデル名、デバイス、beam等）

Recommended MVP defaults:
- caption_stack_max = 3
- VAD end_silence_ms = 600〜800
- VAD min_speech_ms = 350〜500
- ASR model: small.en (faster-whisper)
- ASR language="en" fixed
