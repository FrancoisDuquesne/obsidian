# OEM (Orbit Ephemeris Message)

## One-sentence definition
An **OEM (Orbit Ephemeris Message)** is a standardized file that gives the **position (and often velocity) of a spacecraft as a function of time**, in a well-defined reference frame.

---

## What problem OEM solves

OEM answers the question:

> “Where exactly was the spacecraft at time **T**?”

This is critical because many subsystems depend on **space–time geometry**:
- ground station visibility
- antenna pointing
- Doppler correction
- payload data reconstruction
- post-processing of SDR captures

OEM provides a **single, authoritative trajectory** shared across teams and tools.

---

## What is inside an OEM

Conceptually, an OEM is a **time series**:

$$
t_k \;\rightarrow\; \vec{r}(t_k), \;\vec{v}(t_k)
$$

Where:
- $t_k$: timestamp (UTC, TAI, etc.)
- $\vec{r}$: position vector (x, y, z)
- $\vec{v}$: velocity vector (optional but common)

Example (conceptual):

```

## Time (UTC)              X        Y        Z        VX       VY       VZ

2026-02-05T12:00:00   7021.1   -102.4    12.3    -0.001    7.502    1.221
2026-02-05T12:10:00   7019.9   -110.8    14.1    -0.002    7.501    1.219

```

---

## OEM is NOT a formula

Important distinction:

- OEM does **not** describe orbital mechanics
- OEM does **not** contain force models
- OEM is **declarative**

It simply states:
> “At this time, the spacecraft was here, moving like this.”

Any interpolation or prediction is done by the **consumer**.

---

## Reference frames (critical)

OEM always declares a **reference frame**, e.g.:
- ECI (Earth-Centered Inertial)
- ECEF (Earth-Centered Earth-Fixed)

Same numbers + different frame = **completely different physical meaning**.

OEM makes the frame explicit to avoid ambiguity.

---

## How many points does an OEM have?

The number of points is **not fixed**.

$$
N = \frac{\text{time span}}{\text{time step}}
$$

Typical values:

| Time step | Points per day |
|----------|----------------|
| 60 s | 1 440 |
| 30 s | 2 880 |
| 10 s | 8 640 |
| 1 s  | 86 400 |

Most operational OEMs use **10–60 s** steps.

OEMs intended for high-precision payload processing may go down to **1 s or below**.

---

## Interpolation is expected

OEM points are **anchors**, not raw truth samples.

Downstream software usually:
- interpolates position
- interpolates velocity
- computes derived quantities

Velocity in OEM greatly improves:
- Doppler estimation
- phase prediction
- interpolation accuracy

---

## OEM vs TLE (common confusion)

| OEM | TLE |
|---|---|
| High precision | Lower precision |
| Many time points | 2-line format |
| Engineering / ops | Public tracking |
| Explicit metadata | Implicit assumptions |

OEM is used **inside missions**.  
TLE is mostly for **external tracking**.

---

## Why OEM matters for SDR systems

OEM links **signal time** to **space geometry**.

Given:
- SDR samples timestamped using PPS
- OEM providing position & velocity

You can:
- compute Doppler shift:
$$
f_D(t) = \frac{1}{\lambda} \, \vec{v}(t) \cdot \hat{r}(t)
$$
- correct frequency drift
- align multi-pass or multi-SDR data
- reconstruct geometry after the fact

OEM is therefore a **key dependency** for precise SDR processing.

---

## Mental model to keep

> **OEM is the spacecraft’s breadcrumb trail in space–time.**

Everything else (Doppler, pointing, visibility) is derived from it.

---

## Links
- [[02_Signals/SDR]]
- [[04_Synchronization/PPS]]
- [[01_Big_Picture/Multi_SDR_Synchronization]]
