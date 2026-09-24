# Voice Stack v3 — Local-First Orchestration
**Stamp:** 2026-09-24  
**Repo path:** `jameswilsonotr-ship-it/grok/voice-stack-v3-20260924/`  
**Does NOT overwrite** `voice-stack/` (v1, xAI drafts) or `voice-stack-v2-20260924/` (v2 lock).  
**Supersedes cloud-voice lines only.** House 09-15 fullstack-wireup stays.

## Hard constraints
- **No xAI.** Not voice. Not brain. Not fallback. Pipecat lists xAI STT at 2.14 s P99 — another reason.
- **Voice is local.** STT, TTS, VAD run on steel. Groq is optional **text-only** failover for hard turns. Never the mouth.
- **Orpheus is out.** Do not spend tokens on it.
- **As much brain as possible is local.** Letta + small local LLM is the default path. Databases count.
- **Beat Android Grok:** 600–800 ms E2E is the floor to match. Stretch ≤400 ms is Phase-3 (OCuLink / OpenVoiceStream TRT path), not week-1 cascade promise.
- **Hardware mount is solved.** Projector clamp, rubber grommets, air gaps, UPS. This tree is software.

## What this freeze is
When the truck stops: Grafana pane is up AND a spoken turn comes back in under a second on the cab path. Not Moshi. Not PersonaPlex on an 8 GB Nano. Cascade first. Duplex later when the OCuLink GPU lands.

## Who owns what
| Box | Role | Does NOT do |
|---|---|---|
| K15 (Ultra5 125U, 32 GB, OCuLink, 3× M.2, 21 TOPS NPU) | Foreman. Compose host. Letta :8283. Postgres+pgvector. Qdrant (lake). Prometheus/Grafana. Piper CPU. Zenoh router. nomic-embed. Hourly recap cron. | Chat mouth. NPU is not 4B decode. |
| Jetson Orin Nano Super A (8 GB, 67 TOPS, 102 GB/s) | Chat brain. llama.cpp Qwen3-4B or Qwen3.5-4B Q4_K_M + one LoRA. Short KV (2–4k). | Whisper + Kokoro on the same board. |
| Jetson Orin Nano Super B (8 GB) | Ears/mouth. whisper.cpp CUDA tiny/base.en **or** WhisperTRT base.en **or** Wyoming-whisper-trt. Kokoro-ONNX or Wyoming-Piper. | 8B LLM. |
| 2× Elite Mini G9 (16 GB / ~250 GB stated) | Glass. Grafana pane. PNGTuber. Lake mount. | Model weights. Disks fill. |
| Touchscreen + extra laptops | Same UI, different seat. | Inference. |
| Pixel 9a / 11 Pro | Walkaway ears/mouth. sherpa-onnx or Pocket TTS. Posts PCM/text over Tailscale. | Letta. |
| Hailo-10H (upgrade, USB still preorder) | Phase 1: Whisper HEF + Qwen2-1.5B HEF router/memory. Piper stays host CPU. Shared VDevice. | LoRA hot-swap. Kokoro HEF. |
| Future OCuLink 18–24 GB GPU | Phase 3: same LoRAs on F16/Q8. Optional CSM / PersonaPlex + audio shim. | Week-1 work. |

## v0 steel path (the one that can beat 600–800 ms)
```
Mic
  → Silero VAD (K15 or Nano B)          stop_secs=0.2  ≈ 200 ms commit
  → whisper.cpp tiny.en CUDA | WhisperTRT | Wyoming-whisper-trt   (Nano B)
  → Letta on K15 (core blocks + tools)
       ├─ simple turn → llama.cpp on Nano A (Qwen3-4B Q4 + ONE LoRA)
       └─ hard turn   → Groq OpenAI-compat (text only) OR stay local and refuse
  → Kokoro-ONNX or Piper streamed sentence-by-sentence (K15 / Nano B / Wyoming)
  → Speakers + PNGTuber overlay + Prom histograms
```

Wyoming is the **stable ASR/TTS socket** even if Pipecat owns frames later. Do not invent a third protocol.

## Latency budget (honest)
```
Silero stop_secs=0.2              ~200 ms commit
whisper.cpp tiny.en CUDA          ~150–300 ms final on short turn (stream partials)
3B/4B Q4 TTFT Nano                ~150–450 ms warm (stream tokens NOW)
Kokoro/Piper first audio          ~40–180 ms (Piper CPU often wins TTFA; Kokoro wins MOS)
Overlap: TTS starts on first sentence while LLM still generating
Warm local E2E first-audio        500–800 ms     ← freeze target
Cold / Letta-tool / recall turn   1.5–3 s        ← slow path, cover with filler bank
OpenVoiceStream published         251 ms V2V p50 on Orin Nano highperf
                                  EXISTENCE PROOF. Not our number until we measure.
```

## Recommended v0 pick line
Silero v5/v6 → whisper.cpp tiny.en CUDA **or** Wyoming-whisper-trt (Nano B) → Letta+PG on K15 talking to llama.cpp Qwen3-4B Q4 (Nano A, one LoRA) → Piper (cheap TTFA) or Kokoro-ONNX (quality), streamed → PNGTuber + Grafana on G9. Zenoh between boxes. Wyoming sockets stay. Groq only if Letta marks the turn hard.

## File map
- `01_HARDWARE.md` — fleet, what fits, receipts nag
- `02_LAYERS.md` — five OSS options per layer, scored + citations
- `03_LETTA_AND_MEMORY.md` — 10-hour math, DB backends, agency block
- `04_DOCKER_MICROSERVICES.md` — compose, Wyoming twin, ports
- `05_OBSERVABILITY.md` — Prom/Grafana histograms
- `06_LORA_VTUBER_SHIMS.md` — adapters, glass, session tear-down
- `07_BLIND_SPOTS.md` — traps, debt rank, freeze checklist
- `08_INSTALL_ORDER.md` — ranked commands, pin tags, who-runs-where
- `09_TEST_HARNESS.md` — 20-turn script that prints the histograms
- `10_SOURCES.md` — URLs and measured numbers used in this tree
