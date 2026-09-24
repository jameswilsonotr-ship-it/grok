# Hardware Inventory

## Confirmed (from conversation)
- 2x Samsung Galaxy Book4 Edge ("G9") laptops — 16 GB RAM, 250 GB storage each (verify exact SKU via Gmail receipts)
- 1x touchscreen monitor (on board, powered)
- 1x NVIDIA Jetson (model TBD — check receipts; likely Orin Nano Super 8GB or Orin NX 16GB)
- Projector mount (heavy-duty) used as clamp base for all minis — fixed, vibration-isolated via rubber grommets
- Massive air gaps between devices for cooling
- UPS — confirmed not a problem
- On-board power for all panes-of-glass and Jetson

## Step 1 for Grok Heavy: Pull Gmail Receipts
Search Gmail for:
- `from:samsung OR from:amazon OR from:bestbuy OR from:nvidia OR from:jetson` + keywords: Galaxy Book4 Edge, Jetson, Orin, monitor, UPS, projector mount, grommets
- Extract: exact model numbers, RAM, storage, purchase dates, prices, serials
- Build a table: Device | Model | RAM | Storage | GPU/NPU | Power Draw | Role

## Likely Specs (to verify)
### Galaxy Book4 Edge (16")
- CPU: Qualcomm Snapdragon X Elite (X1E-80-100 or X1E-84-100), 12-core, up to 4.2 GHz
- GPU: Qualcomm Adreno (integrated)
- NPU: ~45 TOPS (Copilot+ PC)
- RAM: 16 GB LPDDR5X
- Storage: 512 GB or 1 TB eUFS/SSD (user reports 250 GB — verify)
- Display: 16" 3K AMOLED 2880x1800, 120 Hz, touchscreen
- OS: Windows 11 Home
- AI: Galaxy AI

### Jetson (TBD — check receipts)
- If Orin Nano Super 8GB: 67 TOPS, 8 GB LPDDR5, 102 GB/s bandwidth, 25W
- If Orin NX 16GB: 157 TOPS, 16 GB LPDDR5, 102.4 GB/s, 10-40W
- Runs Llama 3.1 8B at ~19 tok/s (Nano Super) to higher on NX

### Touchscreen Monitor
- TBD from receipts. Likely USB-C or HDMI input, powered from on-board supply.

## Mounting
- Heavy projector clamp → all minis mounted fixed
- Rubber grommets isolate vibration
- Air gaps for passive/active cooling
- No thermal throttling expected under normal voice-assistant load
