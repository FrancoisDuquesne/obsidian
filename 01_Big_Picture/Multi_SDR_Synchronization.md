# Multi-SDR Synchronization

Multi-SSDR systems are **primarily synchronization problems**, not data problems.

## Dimensions of synchronization
- **Time**: sample counters, frame start (PPS)
- **Frequency**: oscillator offsets
- **Phase**: carrier phase alignment
- **Frame**: known sequences (PRS / preambles)

If any of these fail → constellation rotates, frames drift, data breaks.

## Key tools
- PPS → time alignment
- Shared reference clock (e.g. 10 MHz)
- PRS / preambles → frame + phase recovery

Next:
- [[04_Synchronization/PPS]]
- [[04_Synchronization/PRS_and_Preambles]]
