# Qwen3-ASR Local Setup

This is an optional fallback for Bili Note when public subtitles and browser
AI subtitles are unavailable. It is not required for ordinary metadata, opus,
public-subtitle, comment, and archive workflows.

## Isolation

Use the shared local environment below. It is outside the Git repository and
must not be committed:

%USERPROFILE%\.cache\rimagination-notes\qwen3-asr-venv

The environment used for this setup has include-system-site-packages = false,
so it does not read or alter Codex's Python environment or another project's
virtual environment.

## Windows setup

    $py = "python"
    $venv = "$env:USERPROFILE\.cache\rimagination-notes\qwen3-asr-venv"
    & $py -m venv $venv
    & "$venv\Scripts\python.exe" -m pip install --upgrade pip
    & "$venv\Scripts\python.exe" -m pip install qwen-asr==0.0.6 accelerate==1.12.0 qwen-omni-utils==0.0.9 pandas==3.0.3 torchvision==0.24.0
    & "$venv\Scripts\python.exe" -m pip install --index-url https://download.pytorch.org/whl/cu128 torch==2.9.0 torchvision==0.24.0

Use the cu128 index for the CUDA-enabled PyTorch wheel. A normal PyPI
install can resolve to a +cpu build, which reports torch.cuda.is_available()
as false even when an NVIDIA GPU is installed.

## Verification

    & "$venv\Scripts\python.exe" -c "import torch, qwen_asr; print(torch.__version__, torch.version.cuda, torch.cuda.is_available()); print(torch.cuda.device_count()); print(qwen_asr.__file__)"

The Qwen model is Qwen/Qwen3-ASR-0.6B and is downloaded to the local Hugging
Face cache on first use. Do not commit model weights or the cache.

## ffmpeg

Keep ffmpeg outside Git as a local tool. Bili Note can use a portable install
under %USERPROFILE%\.cache\rimagination-notes\tools\ffmpeg.

## Verified Windows status (2026-07-19)

- RTX 5060 Laptop is visible to `nvidia-smi` with 8 GB VRAM.
- The isolated environment and Python packages pass `pip check`.
- PyPI installation resolves to `torch 2.9.0+cpu`; this is not GPU-enabled.
- The official cu128 wheel is about 2.86 GB. On a slow or filtered network,
  pip may leave a zero-byte temporary file and appear to hang. Stop that run
  and retry only when the PyTorch download endpoint is reachable.
- The Qwen3-ASR-0.6B model must finish downloading before transcription can start.
  An `.incomplete` file in the Hugging Face cache is not a usable model.
- The `hf-mirror.com` endpoint was tested but returned a model layout that the
  current `Qwen3ASRModel` loader could not use; the official model endpoint is
  preferred.

Do not start a multi-hour transcription until both checks pass:

    torch.version.cuda is not None
    torch.cuda.is_available() is True

For CPU fallback, use `dtype=float32` and `device_map=cpu`, but first verify
that the complete Qwen model is present in the local Hugging Face cache.
