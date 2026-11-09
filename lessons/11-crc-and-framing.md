# Lesson 11: CRC and Framing

[← Error detection and correction](10-error-detection-correction.md) · [Index](../README.md)

*Based on the "Data Link Errors" lecture deck, slides 37–46.*

## The big idea

Error **correction** is expensive, and real links change over time, so designing for the worst case wastes capacity. In practice, links use error **detection** plus **retransmission**: if a frame is bad, ask for it again. This is more efficient and it adapts to the link. The usual detection tool is the **CRC (Cyclic Redundancy Check)**.

The data link layer also needs to know **where each frame starts and ends**. That's **framing**, done with **flags** and **bit stuffing**.

## 1. CRC: the setup

| Symbol | Meaning |
|---|---|
| **M** | The frame contents to protect (k bits) |
| **G** | The **generator**, **n + 1 bits** long, known to both sides |
| **CRC** | The remainder, **n bits** long |

**XOR rule** (subtraction with no borrowing): 0⊕0 = 0, 1⊕1 = 0, 0⊕1 = 1⊕0 = 1.

## 2. CRC: the sender

1. Append **n zeros** to M (n = length of G − 1).
2. **Divide** by G using XOR: line G up under the first remaining 1, XOR, and repeat.
3. The **remainder** (the last n bits) is the **CRC**.
4. Send **M followed by the CRC**.

**Slide example:** M = `11100110`, G = `11001` (so n = 4). Divide `11100110 0000`:

| Step | Window (under the first 1) | ⊕ G | Result | Bits afterwards |
|---|---|---|---|---|
| 1 | `11100` | `11001` | `00101` | `001011100000` |
| 2 | `10111` | `11001` | `01110` | `000111000000` |
| 3 | `11100` | `11001` | `00101` | `000001010000` |
| 4 | `10100` | `11001` | `01101` | `000000110100` |
| 5 | `11010` | `11001` | `00011` | `000000000110` |

No 1 is left with room for a full window, so the **last 4 bits are the remainder: CRC = `0110`**.
Transmitted = `11100110 0110`.

## 3. CRC: the receiver

1. Divide **everything received** (k + n bits) by G. Don't add zeros this time.
2. Remainder **0** → **no error detected**.
3. Remainder **not 0** → **error detected**, so ask for a retransmission.

**Slide example:** if the CRC bits are hit by a burst and `11100110 1111` arrives, the remainder is `1001` ≠ 0, so the **error is detected** ✓.

⚠️ A CRC isn't 100% perfect. An error is missed only if the error pattern is **identical to G, or has G as a factor**. That's why G is chosen to be **prime** (in modulo-2 arithmetic).

## 4. Standard generators

Generators are written as polynomials. Each 1-bit becomes a power of X.

| Name | Polynomial | Bits |
|---|---|---|
| CRC-16 | X¹⁶ + X¹⁵ + X² + 1 | 17-bit G → 16-bit CRC: `1 1000 0000 0000 0101` |
| CRC-32 | X³² + X²⁶ + … + X + 1 | 33-bit G → 32-bit CRC |
| Others | | 65-bit (64-bit CRC), 9-bit (8-bit CRC) |

## 5. Framing with flags

Frames vary in length, and there are gaps between them, so the receiver needs to know where each frame begins and ends.

- Put a **flag `01111110`** at the **start and end** of each frame.
- **Problem:** the data might contain `01111110` too, which would look like the end of the frame.

## 6. Bit stuffing

**Sender:** after every **five 1s in a row** in the data, **insert a 0**, even if the next bit is already a 0. Then start counting again.
**Receiver:** whenever it sees **five 1s followed by a 0**, it **removes that 0** (destuffing).
The flags themselves are **not** stuffed, so they're the only place six 1s in a row can appear.

**Slide example:**

```
data        = 001111101000
stuffed     = 00111110 0 1000        ← 0 inserted after 11111 (even though a 0 follows)
transmitted = 01111110 0011111001000 01111110
               start flag              end flag
destuffed   = 001111101000           ✓ correct
```

**Why stuff even when the next bit is already 0?** Otherwise the receiver couldn't tell which 0s were stuffed and which were real data.

Other methods also exist: special **character sequences** as flags (with **character stuffing**), or a **length count** in the frame, sometimes combined with flags.

## 7. What the data link layer does (summary)

- Builds on an **unreliable bit pipe** (L1)
- **Error detection:** knows whether a frame has errors
- **Framing:** knows where a frame starts and ends
- Also: what to do after an error, **flow control** on the link, and **controlling access** to a shared medium

---

## Quiz

**Q1.** G = `10101`. How long is the CRC?

<details><summary>Show answer</summary>

G has 5 bits, so the CRC has **4** bits.
</details>

**Q2.** Generate the CRC for M = `1010011001001` with G = `10101`.

<details><summary>Show answer</summary>

Append 0000 and divide by 10101. The remainder is **1001**, so the transmitted bits are 1010011001001 1001.
</details>

**Q3.** Generate the CRC for M = `101001100100101` with G = `101001`.

<details><summary>Show answer</summary>

Append 00000 and divide. The remainder is **00010**.
</details>

**Q4.** `101011100101010011` is received with G = `10011`. Is there an error?

<details><summary>Show answer</summary>

Divide the received bits by G. The remainder is **0000**, so **no error is detected**.
</details>

**Q5.** `101101110101011111` is received with G = `11001`. Is there an error?

<details><summary>Show answer</summary>

The remainder is **1010** ≠ 0, so an **error is detected**.
</details>

**Q6.** Bit stuff the data `0111111101111110` and add flags.

<details><summary>Show answer</summary>

Count the 1s: `0 11111` is five 1s, so stuff a 0. Then `11 0` resets the count. Then `11111` is five more, so stuff a 0. Then `10`.
`0 11111 [0] 11 0 11111 [0] 10` gives stuffed data `011111011011111010` (2 stuffed 0s).
Frame: **01111110 011111011011111010 01111110**
</details>

**Q7.** True or false: the start and end flags are bit-stuffed too.

<details><summary>Show answer</summary>

**False.** They're left alone, so the receiver can find frame boundaries by scanning for `01111110`.
</details>

**Q8.** Why does a CRC system usually just *detect* errors instead of correcting them?

<details><summary>Show answer</summary>

Link conditions vary, and correction designed for the worst case wastes capacity. Detection plus retransmission is **more efficient and adaptive**.
</details>
