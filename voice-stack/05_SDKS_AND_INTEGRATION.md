# SDKs, APIs & Integration Map

## Local SDKs (install on Jetson + laptops)
| Component | SDK / Library | Install | Notes |
|---|---|---|---|
| VAD | `silero-vad` (ONNX) | `pip install silero-vad` | CPU, 1ms/frame |
| Wake word | `openwakeword` | `pip install openwakeword` | ONNX, customizable |
| STT | `faster-whisper` | `pip install faster-whisper` | CTranslate2, GPU via CUDA |
| STT (alt) | `whisper.cpp` | build from source | C/C++, CUDA support |
| LLM | `llama.cpp` (Python bindings) or `ollama` | `pip install llama-cpp-python` / Ollama | LoRA support, GGUF models |
| LLM (alt) | `transformers` + `peft` | `pip install transformers peft` | For LoRA training/inference |
| TTS | `kokoro` or `piper-tts` | `pip install kokoro` / Piper binary | Kokoro needs ~1GB VRAM |
| Orchestration | `pipecat-ai` or `livekit-agents` | `pip install pipecat-ai` / `pip install livekit-agents` | Voice pipeline frameworks |
| Audio I/O | `sounddevice` or `pyaudio` | `pip install sounddevice` | Cross-platform, low-latency |
| LoRA training | `unsloth` or `peft` + `trl` | `pip install unsloth` | Fast LoRA fine-tuning on consumer GPU |

## Cloud APIs (xAI / SpaceXAI)
| API | Endpoint | Model | Latency | Use |
|---|---|---|---|---|
| Grok Voice realtime (S2S) | `wss://api.x.ai/v1/realtime?model=grok-voice-latest` | grok-voice-think-fast-2.0 | 0.70s TTFA | Heavy reasoning, agentic turns |
| Grok STT (streaming) | REST/streaming STT endpoint | grok-voice-transcribe-2.0 | 233ms mean | Accuracy-critical transcription |
| Grok TTS (streaming) | `wss://api.x.ai/v1/tts` | Various voices | Low | Cloud TTS fallback |
| Grok LLM (text) | `https://api.x.ai/v1/chat/completions` | grok-4.x | 107ms p50 TTFB | Text-only reasoning if needed |

## LoRA Adapter Strategy
1. Collect personality/voice data (your past conversations, preferred phrasing, tone)
2. Fine-tune LoRA on Qwen3-4B or Llama 3.2 3B using `unsloth` (fast, low VRAM)
3. Export as GGUF or merge into base
4. Load at runtime on Jetson via `llama.cpp` or Ollama
5. Swap adapters per context (casual, technical, playful) without reloading base model

## Integration Flow
```
[Mic] → sounddevice → Silero VAD → faster-whisper STT → Router
                                                      ├→ Local: Qwen3-4B+LoRA → Kokoro TTS → [Speakers]
                                                      └→ Cloud: xAI Grok Voice WebSocket → [Speakers]
```
All streaming. All interruptible. All instrumented for latency logging.

## Gmail Receipt Search (Step 1)
When Grok Heavy runs, search Gmail for:
- `Galaxy Book4 Edge` OR `Jetson` OR `Orin` OR `projector mount` OR `UPS` OR `touchscreen monitor`
- Extract exact SKUs, confirm RAM/storage, confirm Jetson model
- Update `01_HARDWARE_INVENTORY.md` with verified specs

## Next Action
Drop the Grok Heavy prompt. It reads these 5 files, pulls Gmail receipts, benchmarks each layer option on the actual hardware, and returns a ranked, code-ready implementation plan targeting ≤ 400 ms end-to-end.
