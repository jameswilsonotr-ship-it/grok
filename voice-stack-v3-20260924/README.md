# voice-stack-v3-20260924

Local-first cab voice companion. No xAI. Groq text-only failover. Orpheus out.

Does not overwrite:
- `voice-stack/` (v1 xAI drafts)
- `voice-stack-v2-20260924/` (v2 lock)

Start here: `00_MASTER.md`
Install: `08_INSTALL_ORDER.md`
Prove it: `09_TEST_HARNESS.md`

v0 pick line:
Silero -> whisper.cpp tiny.en CUDA or Wyoming-whisper-trt (Nano B)
-> Letta+PG on K15 talking to llama.cpp Qwen3-4B Q4 (Nano A, one LoRA)
-> Piper or Kokoro streamed
-> PNGTuber + Grafana on G9
Wyoming sockets stay. Zenoh between boxes.
