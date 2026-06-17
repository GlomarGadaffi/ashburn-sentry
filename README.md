# ashburn-sentry

LoRa RF reverse-engineering toolkit. sweep frequency bands, capture I/Q bursts, analyze bandwidth and spreading factor, decode chirp patterns, and recover modulation parameters from LoRa traffic.

## tools

**aspen_sweep.ino** — spectrum sweeper (ESP32 + SX1262)
- frequency sweep (start, stop, step)
- dwell time per frequency
- RSSI logging
- GPIO trigger on detected carrier above threshold
- outputs JSON scan report

**aspen_sniff.py** — I/Q capture analyzer
- load raw complex64 I/Q file (from SDR or dumped hardware)
- STFT spectrogram analysis
- bandwidth detection (snaps to standard LoRa 125/250/500 kHz)
- spreading factor estimation via chirp-rate analysis
- preamble detection and frame alignment hints

**aspen_analyze.py** — chirp parameter extractor
- decodes LoRa preamble (sync symbols)
- estimates frequency offset and clock skew
- reports modulation parameters (SF, BW, CR, SyncWord)
- confidence scores per estimate

**aspen_trip.py** — real-time decoder (if LoRa frame is unencrypted)
- reads captured frames
- decodes MAC header (to/from, ack, type)
- optional: payload dump if CRC validates

## workflow

```bash
# 1. Sweep for LoRa presence
# (aspen_sweep.ino running on ESP32)

# 2. Capture raw I/Q on detected frequency
# (use rtl-sdr, hackrf, or hardware SPI dump)
sox -t raw -r 2.4M -b 8 -e unsigned-integer -c 1 capture.raw -t wav capture.wav

# 3. Analyze
python3 aspen_sniff.py capture.raw --sample-rate 2400000
# → outputs: BW estimate, SF estimate

# 4. Decode parameters
python3 aspen_analyze.py capture.raw --sample-rate 2400000 --estimated-bw 125000

# 5. Extract payload (if unencrypted)
python3 aspen_trip.py payload.bin --sf 7 --bw 125000
```

## limitations

- SF estimation is heuristic-based; LoRa uses constant-envelope modulation, so SNR must be high
- spreading factors 7-12 supported; SF6 and below are rare
- encrypted payloads decode to raw bytes; no LoRaWAN MIC/FPort parsing
- requires clear, non-overlapping transmissions for chirp analysis
