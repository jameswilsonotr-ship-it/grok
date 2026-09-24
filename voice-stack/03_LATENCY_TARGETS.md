# Latency Targets & Benchmarks

## The Bar We Must Beat
- Current Android Grok: ~600-800 ms end-to-end (user-reported)
- Target: ≤ 600 ms (match), ideally ≤ 400 ms, dream ≤ 200 ms

## Where Latency Lives (breakdown of a typical cascade)
| Stage | Typical Cost | How to Cut It |
|---|---|---|
| Mic capture + VAD | 50-200 ms | Local Silero VAD, 20ms frames, no network |
| STT | 100-500 ms | faster-whisper on Jetson GPU, small model, streaming |
| LLM TTFT | 200-1000 ms | Local small model OR xAI Grok Voice realtime (0.70s TTFA) |
| TTS TTFA | 40-600 ms | Piper (40ms) or Kokoro (90ms), streaming chunks |
| Network RTT | 50-300 ms | Eliminate by going local; or use xAI WebSocket (107ms p50 US) |
| Orchestration overhead | 20-100 ms | Single asyncio event loop, zero-copy buffers |

## Reference Numbers (from research, Sept 2026)
- xAI Grok Voice Think Fast 2.0: 0.70s time-to-first-audio (sub-second, only top-5 under 1s)
- xAI API TTFB: 107 ms p50 from US Central
- Grok STT: 233 ms mean time-to-final-segment
- Local cascade (RTX 3060 12GB): 1-2s end-to-end with Whisper small + Llama 8B + Piper
- Local cascade (Jetson Orin Nano Super): ~2-4s with small models; faster with tiny models
- Moshi (Kyutai, end-to-end S2S): ~160-200 ms theoretical, full-duplex, interruptible

## Strategy to Beat Android
1. **Eliminate network for the fast path.** Local VAD + STT + small LLM + TTS on Jetson = no RTT.
2. **Use xAI Grok Voice realtime as the heavy path** when local model can't handle it. WebSocket, 0.70s TTFA, streaming audio out.
3. **Stream everything.** Don't wait for full STT transcript or full LLM response. Pipe tokens to TTS as they arrive.
4. **LoRA adapters** for personality so the local model sounds like *you*, not generic.
5. **Barge-in / full-duplex** so it feels alive, not turn-based.

## Measurement Plan
- Instrument every stage with timestamps (Python `time.perf_counter` or Rust `Instant`)
- Log: VAD end → STT done → LLM first token → TTS first audio → audio playback start
- Target p50 and p95 for each stage
- Run 100-turn test harness, report distribution
