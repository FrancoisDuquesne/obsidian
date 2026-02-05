# PRS and Preambles

PRS = **Pseudo-Random Sequence**

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
