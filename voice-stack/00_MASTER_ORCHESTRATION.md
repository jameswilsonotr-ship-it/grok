# Voice Stack Orchestration — Master Plan

**Goal:** Silky-smooth, natural voice in/out on the local fleet (2x Galaxy Book4 Edge G9 + touchscreen monitor + Jetson), beating current Android Grok (~600-800 ms) and ideally approaching sub-second with human-like voice. Zero latency is the dream; beating Android is the floor.

**Hardware status:** SOLVED. Projector-clamp mount with rubber grommets, massive air gaps, vibration handled. UPS is not a problem. Power and mounting for all panes-of-glass and Jetson are on board.

**Software status:** THE ONLY REMAINING PROBLEM. This is what Grok Heavy must solve.

## Target Latency Budget (end-of-speech → start-of-audio)
- Floor: ≤ 600 ms (match Android)
- Stretch: ≤ 400 ms
- Dream: ≤ 200 ms (human conversational floor)

## Architecture Overview
Local fleet handles flavor, personality, LoRA adapters, wake word, VAD, and fast-path inference. Cloud (xAI Grok Voice realtime) handles heavy reasoning when needed. Hybrid: local first, escalate to cloud on complex turns.

## Five Software Layers (each gets 5 options)
1. **Capture & VAD** — mic input, voice activity detection, endpointing
2. **STT** — speech-to-text
3. **Brain / LLM** — reasoning, personality, LoRA adapters
4. **TTS** — text-to-speech, natural voice
5. **Orchestration & Transport** — pipeline glue, streaming, WebSocket, fallback logic

## Execution Order for Grok Heavy
1. Read `01_HARDWARE_INVENTORY.md` (pull Gmail receipts for exact models/SKUs)
2. Read `02_JETSON_SPECS.md`
3. Read `03_LATENCY_TARGETS.md`
4. Read `04_SOFTWARE_LAYERS.md` (5 options each)
5. Read `05_SDKS_AND_INTEGRATION.md`
6. Produce a ranked implementation plan with code skeletons
7. Do NOT stop until latency target is met on paper and in a test harness

## Non-negotiables
- Natural voice, not robotic
- Interruptible / barge-in capable
- Runs on the local fleet without constant cloud dependency
- LoRA adapters for personality/flavor
- Streaming end-to-end (no waiting for full response)
