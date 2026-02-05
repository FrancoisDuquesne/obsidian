# PPS (Pulse Per Second)

PPS is a **hardware timing signal** that produces one pulse exactly every second.

## Why it exists
- Gives all devices a shared notion of “time”
- Used to align sample counters and frames

## In multi-SDR systems
PPS is often used to:
- reset or align sample counters
- timestamp frames
- ensure deterministic alignment across radios

PPS is often used together with a **frequency reference** (e.g. 10 MHz).
