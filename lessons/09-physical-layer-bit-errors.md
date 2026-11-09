# Lesson 9: The Physical Layer, Noise and Bit Errors

[← Exam revision](08-exam-revision.md) · [Index](../README.md) · [Next: Error detection and correction →](10-error-detection-correction.md)

*Based on the "Data Link Errors" lecture deck, slides 1–14.*

## The big idea

The **physical layer (L1)** turns 0s and 1s into real signals (voltages, light pulses, radio waves) and sends them to the **adjacent** device, the one directly connected. But the real world is noisy, so sometimes a bit arrives flipped. The slides call this an **"unreliable bit pipe"**: bits go in one end and come out the other, but some may be wrong.

## 1. The three causes of noise

| Cause | What it is | Everyday example |
|---|---|---|
| **Cross-talk** | A signal on one wire leaks onto a nearby wire | Hearing another call faintly on an old phone line |
| **Impulse noise** | Short, large spikes caused by **external** equipment or activity | A crackle on the radio when a motor starts nearby |
| **Thermal noise** | Random movement of electrons. It's **always** present, even with no data. | The hiss from speakers turned up with nothing playing |

- The worst cross-talk is **NEXT (Near-End Cross-Talk)**. At your end of the cable, your own **powerful** outgoing signal sits next to a **tiny** incoming signal that has weakened over the distance, and the strong one drowns out the weak one.
- Impulse noise causes **large distortion for a short time**, so it often damages several bits in a row. That's a **burst error**.
- Thermal noise is also called **white noise**. It can only be reduced by **cooling**, for example with liquid nitrogen at −196 °C (77 K).

## 2. Why digital transmission helps

Send a 0 and a 1 as **two very different signals**, for example +3 V for a 0 and −3 V for a 1. Noise moves the signal a little, but the receiver can still tell which side it's closer to. An error only happens when the noise pushes the signal past the middle.

## 3. Transmission modes

| Mode | Directions | Example |
|---|---|---|
| **Simplex** | One way only, ever | A TV broadcast |
| **Half-duplex** | Both ways, but **not at the same time** | A walkie-talkie |
| **Full-duplex** | Both ways **at the same time** | A phone call |

## 4. Multiplexing: sharing one link

Two digital signals can't share a wire at the same moment, because the bits would interfere. There are three ways to share a link:

| Method | How it shares | Analogy |
|---|---|---|
| **FDM** (Frequency Division) | Each user gets its own **frequency band**, all the time. It needs a modulator and a demodulator. | Radio stations on different frequencies |
| **TDM** (Time Division) | Each user gets the **whole link** in its own **time slot**, in turn | Taking turns to speak |
| **Statistical** | Send whoever has data. The link is **never idle** while there's data waiting. | A queue at a single shop till |

- FDM gives **some of the resource all of the time**: a 100 Mb/s pipe shared by 50 users gives each one 2 Mb/s.
- TDM gives **all of the resource some of the time**.
- Both FDM and TDM fix the channels, which is **inefficient for bursty traffic**.
- Statistical multiplexing costs extra **overhead** (each frame must say which stream it belongs to), the **queueing delay varies**, and it needs a method to decide which packet goes next.

## 5. Bit errors

There are only **two** ways to make a bit error: a 1 read as a 0, or a 0 read as a 1.

**p_b** is the probability of a single bit being in error. It's also called the **BER** (Bit Error Rate), even though it isn't really a rate.

| Medium | Typical p_b |
|---|---|
| Optical fibre | 10⁻⁹ or less (10⁻¹⁰ on slide 17) |
| Copper / coaxial | around 10⁻⁶ (10⁻⁷ on slide 17) |
| Radio (wireless) | anywhere from 10⁻³ to 10⁻⁶ |

## 6. From bit errors to frame errors

A frame of **L bits** is only good if **every** bit is good:

```
P(frame good) = (1 − p_b)^L
p (frame error) = 1 − (1 − p_b)^L          ← exact
p ≈ L × p_b                                ← only when p ≪ 1
```

⚠️ **L is in bits**, so multiply bytes by 8.

**Slide 8 examples:**

| p_b | Frame | Approximation | Use it? |
|---|---|---|---|
| 10⁻⁴ | 1,000 B | 8,000 × 10⁻⁴ = 0.8 | ❌ too big, so use the exact formula: **0.55** |
| 10⁻⁶ | 1,000 B | 0.008 | ✅ |
| 10⁻⁸ | 64,000 B | 0.005 | ✅ |

### Over several hops

The packet has to survive **every** link:

```
P(good over k hops) = (1 − p_b)^(L × k)       hops = intermediate nodes + 1
```

### Working backwards

```
Largest frame:  L < ln(1 − p) ÷ ln(1 − p_b)     (round down; ÷ 8 for bytes)
Highest BER:    p_b ≤ 1 − (1 − p)^(1/(L × k))
```

Quick version: L < p ÷ p_b (only when the target p is small).

---

## Quiz

**Q1.** Which kind of noise is present even when no data is being sent?

<details><summary>Show answer</summary>

**Thermal (white) noise.** It comes from electron movement and can only be reduced by cooling.
</details>

**Q2.** What's the difference between half-duplex and full-duplex?

<details><summary>Show answer</summary>

Half-duplex can go both ways, but only **one way at a time**. Full-duplex goes both ways **at the same time**.
</details>

**Q3.** Why is statistical multiplexing better than TDM for bursty traffic?

<details><summary>Show answer</summary>

TDM gives every user a fixed slot even when they have nothing to send, so slots go to waste. Statistical multiplexing sends whoever has data, so the link is never idle while data is waiting.
</details>

**Q4.** A frame is 12,000 bits and p_b = 10⁻⁶. What is the frame error probability?

<details><summary>Show answer</summary>

p ≈ 12,000 × 10⁻⁶ = **0.012**. That's small, so the approximation is fine.
</details>

**Q5.** p_b = 10⁻⁹ and you want p < 10⁻⁶. What is the largest frame in bytes?

<details><summary>Show answer</summary>

L < 10⁻⁶ ÷ 10⁻⁹ = 1,000 bits = **125 bytes**.
</details>

**Q6.** A path has 5 hops, BER 10⁻⁵ and 1,200-byte packets. What is the probability the packet is in error?

<details><summary>Show answer</summary>

9,600 × 5 = 48,000 bits. The approximation gives 0.48, which is too big, so use the exact formula: 1 − (1 − 10⁻⁵)^48,000 ≈ **0.38**.
</details>

**Q7.** 8 intermediate nodes and 1,250 B packets. What is the highest BER so that 99.9% of packets arrive correctly?

<details><summary>Show answer</summary>

9 hops × 10,000 bits = 90,000. p_b ≤ 1 − 0.999^(1/90,000) ≈ **1.1 × 10⁻⁸**.
</details>
