# Synchronization Signals

## PRS — Pseudo-Random Sequence

A deterministic sequence that looks random but is known to both transmitter and receiver.

## Uses
- Frame detection (correlation peak)
- Timing estimation
- Carrier phase estimation
- Frequency offset estimation

## Intuition
Receiver correlates incoming samples with known PRS:
- Peak -> frame start
- Phase of peak -> carrier phase offset

This directly enables QPSK demodulation.

---

## PPS — Pulse Per Second

PPS is a **hardware timing signal** that produces one pulse exactly every second.

### Why it exists
- Gives all devices a shared notion of "time"
- Used to align sample counters and frames

### In multi-SDR systems
PPS is often used to:
- reset or align sample counters
- timestamp frames
- ensure deterministic alignment across radios

PPS is often used together with a **frequency reference** (e.g. 10 MHz).

> **Note:** In other parts of this project, PPS refers to the **Pass Processor Service** (a backend service). See [[PPS PRS Backend Services]].
