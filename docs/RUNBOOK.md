# RUNBOOK: Setup / Run / Troubleshoot (Windows)

## Prerequisites
- Windows 10/11
- NVIDIA GPU (RTX 4060 Ti 16GB)
- 最新のGPUドライバ（推奨）
- .NET SDK 8.x（OverlayAppビルド用）
- Python 3.10〜3.12（InferenceService用）
- Git

> Note: 本リポジトリはクラウドAPI不要（無料・ローカル完結）を前提。
> 翻訳/ASRモデルは大きいため、原則repoには同梱しない（初回起動時にDL or 手動DL）。


## Repository layout
- `src/OverlayApp` : C# WPF app
- `src/InferenceService` : Python inference service
- `docs/` : specs
- `tasks/` : task files


## Quick Start (MVP)
### 1) Python環境（InferenceService）
PowerShell:
```powershell
cd src\InferenceService
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
