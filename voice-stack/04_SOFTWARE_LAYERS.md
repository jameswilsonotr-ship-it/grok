# Five Software Layers — 5 Options Each

For each layer: pick the best fit for the local fleet (Jetson + 2x Galaxy Book4 Edge). Grok Heavy should evaluate all 5, benchmark on actual hardware, and rank.

---

## Layer 1: Capture & VAD (Voice Activity Detection)
Goal: detect speech start/end with minimal latency, support barge-in.

1. **Silero VAD** (ONNX, CPU) — 20ms frames, ~1ms inference, battle-tested, used in LiveKit/Pipecat. Best default.
2. **WebRTC VAD** (Google) — ultra-light, C++, but less accurate on noisy audio.
3. **OpenWakeWord** — for wake-word only ("Hey Grok"), pairs with Silero for continuous VAD.
4. **Porcupine** (Picovoice) — commercial wake word, free tier, custom wake words.
5. **Energy-based VAD** (simple RMS threshold) — fallback, no ML, but noisy.

**Recommendation:** Silero VAD + OpenWakeWord. Local, fast, interruptible.

---

## Layer 2: STT (Speech-to-Text)
Goal: transcribe with <300ms latency on Jetson GPU.

1. **faster-whisper** (CTranslate2) — `small.en` or `base.en`, INT8, GPU. ~1s/utterance on Nano, near-instant on base. Best accuracy/speed tradeoff.
2. **whisper.cpp** — C/C++, Metal/CUDA, `base` or `small`. Slightly faster than faster-whisper on some hardware.
3. **NVIDIA NeMo Parakeet** — streaming, low-latency, but heavier setup on Jetson.
4. **xAI Grok STT** (cloud) — 233ms mean, 4% WER. Use as fallback or for accuracy-critical turns.
5. **Distil-Whisper** — 6x faster than Whisper large, good for CPU fallback on laptops.

**Recommendation:** faster-whisper `small.en` on Jetson GPU for local; Grok STT cloud for hard cases.

---

## Layer 3: Brain / LLM
Goal: reason + personality. Local for speed, cloud for power. LoRA adapters for flavor.

1. **Qwen3-4B Q4** (local) — ~2.5GB, 30-50 tok/s on Orin Nano Super, strong reasoning for size. Best local brain.
2. **Llama 3.2 3B** (local) — smaller, faster, good for simple turns.
3. **Phi-4-mini 3.8B** (local) — Microsoft, efficient, decent quality.
4. **xAI Grok Voice realtime** (cloud) ‐ 0.70s TTFA, full S2S, tools, web search. Best for complex/agentic turns.
5. **Local LoRA-adapted model** — take Qwen3-4B or Llama 3.2, train LoRA on your voice/personality data, merge or load at runtime. This is the "flavor" layer.

**Recommendation:** Local Qwen3-4B + LoRA for fast path; escalate to Grok Voice realtime for hard turns. Hybrid router decides.

---

## Layer 4: TTS (Text-to-Speech)
Goal: natural, non-robotic voice, <100ms TTFA, streaming.

1. **Piper** (ONNX, CPU) — 40ms TTFA, <100MB RAM, real-time on Pi. Fastest, slightly robotic but acceptable.
2. **Kokoro-82M** (GPU) — 90ms TTFA, MOS 4.2, most natural per parameter. Best quality/speed. ~900MB VRAM.
3. **xAI Grok Voice TTS** (cloud) — part of realtime S2S, native quality, 5 voices (Eve, Ara, Rex, Sal, Leo) + custom.
4. **Kitten TTS** — 25MB, very fast, Apache 2.0, newer, good for edge.
5. **XTTS v2** (Coqui) — voice cloning from 6s reference, but 600ms TTFA, 4.5GB VRAM. Only for async/cloned-voice use.

**Recommendation:** Kokoro-82M on Jetson GPU for natural local voice; Piper as ultra-fast fallback; Grok Voice for cloud path.

---

## Layer 5: Orchestration & Transport
Goal: glue the pipeline, stream end-to-end, handle fallback, barge-in.

1. **Pipecat** (Python) — purpose-built voice pipeline framework, supports VAD/STT/LLM/TTS, WebRTC, LiveKit. Best for this use case.
2. **LiveKit Agents** (Python/TS) — SFU + agent framework, used in production voice agents. Excellent for barge-in and multi-turn.
3. **Custom asyncio loop** — zero-copy buffers, full control, minimal overhead. More work but fastest.
4. **xAI Grok Voice WebSocket** — `wss://api.x.ai/v1/realtime?model=grok-voice-latest`. Single connection, streaming audio in/out, tools. Use for cloud path.
5. **Moshi** (Kyutai) — end-to-end S2S model, ~200ms, full-duplex, interruptible. Research-forward but the latency king if it runs on Jetson.

**Recommendation:** Pipecat or LiveKit Agents for the local cascade; xAI WebSocket for the cloud heavy path; custom loop if you want absolute minimal overhead.

---

## Hybrid Routing Logic (the secret sauce)
```
User speaks → VAD (local) → STT (local)
  → Router decides:
     - Simple/fast → Local LLM (Qwen3-4B + LoRA) → Local TTS (Kokoro)
     - Complex/agentic → xAI Grok Voice realtime (WebSocket) → stream audio back
  → Audio out (local speakers or monitor)
```
Barge-in: if VAD detects new speech during TTS playback, cancel current generation immediately.
