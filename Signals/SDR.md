## One-sentence definition
An **SDR (Software Defined Radio)** is a radio system where **most signal processing is done in software on digital samples**, not in fixed analog hardware.

---

## Classical radio vs SDR (mental shift)

### Classical (hardware) radio
```

RF → filters → mixers → demodulator → bits

```

- Each block = physical circuitry
- Modulation type fixed
- Hard to change once built

### SDR
```

RF → ADC → software (DSP) → bits

````

- RF is converted to **digital samples early**
- Everything after is **math + algorithms**
- Modulation, bandwidth, protocols are software-configurable

> **Key shift:** radio becomes a signal-processing problem.

---

## The core idea: complex baseband (I/Q)

SDRs do **not** work directly with "waves" or "bits".

They work with **complex samples**:

$$
x[n] = I[n] + jQ[n]
$$

Where:
- **I** = In-phase component
- **Q** = Quadrature component

These samples represent:
- amplitude
- phase
- frequency content

This is why modulations like QPSK fit naturally in SDRs.

---

## Minimal SDR signal chain

```mermaid
flowchart LR
  Antenna --> RF["RF front-end"]
  RF --> ADC["ADC"]
  ADC --> DSP["DSP (software or FPGA)"]
  DSP --> Demod["Demodulation"]
  Demod --> Bits["Bits / packets"]
````

Where DSP includes:

* filtering
* frequency correction
* timing recovery
* demodulation (QPSK, etc.)

---

## Why SDRs love QPSK (and friends)

QPSK symbols are **points in the complex plane**:

$$
s_k \in \left\{ \frac{1}{\sqrt{2}}(\pm1 \pm j) \right\}
$$

An SDR:

* receives I/Q samples
* estimates phase & timing
* decides which constellation point was sent

This makes SDR + QPSK a very natural pair.

See also: [[Modulation/QPSK]]

---

## What makes an SDR "software" (in practice)

"Software" does not always mean **CPU**.

SDR processing can run on:

* CPU (C/C++, Java bindings, Python)
* FPGA (HDL, parallel DSP)
* DSP cores
* GPU (sometimes)

So SDR is really:

> **reconfigurable signal processing**, not necessarily slow software.

---

## Control plane vs data plane (important distinction)

### Data plane

* I/Q sample streams
* High-rate
* Deterministic
* Often FPGA/DSP

### Control plane

* Configuration
* Mode changes
* Gain, frequency, modulation selection
* Often backend software (Java, Python, etc.)

Most frontend/backend devs interact with the **control plane**.

---

## SDR in space (SSDR)

A **Space SDR (SSDR)**:

* survives radiation
* has deterministic timing
* supports redundancy
* integrates tightly with spacecraft timing (PPS)

Used for:

* telemetry & telecommand
* payload data downlink
* experimental waveforms

---

## Why synchronization is everything in SDR

SDRs assume:

* correct time reference
* correct frequency reference
* correct phase reference

Without synchronization:

* constellations rotate
* symbols drift
* demodulation fails

That's why SDR discussions quickly involve:

* PPS → time alignment
* PRS → frame & phase alignment
* OEM → geometry & Doppler

---

## Mental model to keep

> **An SDR is a programmable measurement instrument for electromagnetic waves.**

Everything else (QPSK, PPS, PRS, OEM) plugs into that idea.

---

## Links

* [[Modulation/QPSK]]
* [[Synchronization/PPS]]
* [[Synchronization/PRS and Preambles]]
* [[Big Picture/Multi SDR Synchronization]]
