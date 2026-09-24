# Jetson Specs & Local Inference Capability

## Candidate Models (verify which one is on board via Gmail)

### Option A: Jetson Orin Nano Super 8GB
- AI Performance: 67 TOPS (INT8)
- GPU: 1024-core Ampere, 32 Tensor Cores
- Memory: 8 GB LPDDR5 unified (CPU+GPU), 102 GB/s bandwidth
- Power: 7-25W (MAXN mode)
- LLM throughput (llama.cpp, 25W): Llama 3.1 8B ~19 tok/s; Qwen2.5 7B ~22 tok/s; SmolLM2 135M ~165 tok/s
- Good for: small/fast models, VAD, STT, LoRA adapters, fast-path replies
- Not enough for: large 70B models, heavy multimodal

### Option B: Jetson Orin NX 16GB
- AI Performance: 157 TOPS
- GPU: 1024-core Ampere, 32 Tensor Cores
- Memory: 16 GB LPDDR5, 102.4 GB/s
- Power: 10-40W
- LLM throughput: significantly higher than Nano; comfortable 8B-13B models at 30+ tok/s
- Good for: running a capable local brain + STT + TTS simultaneously

## What the Jetson Should Own (Local Fast Path)
1. Wake word detection (OpenWakeWord / Porcupine)
2. VAD + endpointing (Silero VAD / WebRTC VAD)
3. STT (faster-whisper small/base, or whisper.cpp)
4. Small LLM for fast replies (Qwen3 4B / Llama 3.2 3B / Phi-4-mini) with LoRA adapters for personality
5. TTS (Piper or Kokoro) for low-latency audio out
6. Orchestration loop (Python + asyncio)

## What the Jetson Should NOT Own
- Heavy reasoning (delegate to xAI Grok Voice realtime over WebSocket)
- Large model inference that exceeds memory
- Anything that would cause thermal throttling under sustained load

## Recommended Local Model Stack (if Orin Nano Super 8GB)
- STT: faster-whisper `small.en` (INT8) — ~1s per utterance on GPU, or `base.en` for near-instant
- LLM: Qwen3-4B Q4_K_M (~2.5 GB) or Llama 3.2 3B — ~30-50 tok/s
- TTS: Piper (CPU, <100ms TTFA) or Kokoro-82M (GPU, ~90ms TTFA, more natural)
- LoRA: train/adapt on personality data, merge or load at runtime via llama.cpp / Ollama

## If Orin NX 16GB
- Can run Llama 3.1 8B comfortably at 30+ tok/s
- Can run STT + LLM + TTS concurrently without swapping
- Better candidate for a fully local, high-quality path
