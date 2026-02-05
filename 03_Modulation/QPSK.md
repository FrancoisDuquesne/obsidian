# QPSK (Quadrature Phase Shift Keying)

## TL;DR
QPSK encodes **2 bits per symbol** using four constellation points in the complex plane (I/Q).
In SDR systems, signals are processed as **complex samples**:

$$
x = I + jQ
$$

---

## Complex baseband representation
A QPSK symbol is:

$$
s_k \in \left\{ \frac{1}{\sqrt{2}}(\pm 1 \pm j) \right\}
$$

Gray-coded mapping:

| Bits | Symbol |
|---|---|
| 00 | $\frac{1}{\sqrt{2}}(+1 + j)$ |
| 01 | $\frac{1}{\sqrt{2}}(-1 + j)$ |
| 11 | $\frac{1}{\sqrt{2}}(-1 - j)$ |
| 10 | $\frac{1}{\sqrt{2}}(+1 - j)$ |

---

## Phase rotation (why synchronization matters)
The received symbol is typically:

$$
r_k = s_k \cdot e^{j\theta} + n_k
$$

- $\theta$: carrier phase offset
- $n_k$: noise

A non-zero $\theta$ rotates the constellation -> wrong decisions.

---

## Hard decision (ideal case)
If timing and phase are perfect:

$$
\hat{b}_I = \mathbb{1}[\Re(r_k) < 0], \quad
\hat{b}_Q = \mathbb{1}[\Im(r_k) < 0]
$$

If phase is wrong -> this fails.

---

## Receiver intuition
Receivers must estimate:
- carrier frequency offset
- carrier phase offset
- symbol timing

Using known sequences -> see [[04_Synchronization/PRS_and_Preambles]].

---

## Data-plane overview
```mermaid
flowchart LR
  Bits --> Mapper["QPSK mapper"]
  Mapper --> Shaping["Pulse shaping"]
  Shaping --> Channel
  Channel --> Sync["Timing + carrier recovery"]
  Sync --> Demapper
  Demapper --> BitsOut
```
