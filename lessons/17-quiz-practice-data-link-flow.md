# Lesson 17: Quiz Practice – Data Link Errors and Flow Control

[← Stop-and-Wait performance](16-stop-and-wait-performance.md) · [Index](../README.md)

*Questions from the course quizzes of 4, 5 and 8 November 2026, with full worked answers. They revise Lessons 9–11 and 15–16.*

> **How to use this:** try each question on paper first, then open the answer.

---

## Part A: Hamming code (Lesson 10)

**Reminder:** positions run 11 → 1 from left to right. Check bits sit at positions 8, 4, 2, 1. Data fills positions 11, 10, 9, 7, 6, 5, 3.

| Check bit | Positions it covers |
|---|---|
| r1 | 1, 3, 5, 7, 9, 11 |
| r2 | 2, 3, 6, 7, 10, 11 |
| r4 | 4, 5, 6, 7 |
| r8 | 8, 9, 10, 11 |

**A1.** What is the 11-bit sequence sent using a 7-4 Hamming code for this data: 1 1 1 0 0 0 1?

<details><summary>Show answer</summary>

Place the data:

```
Position: 11 10  9  8  7  6  5  4  3  2  1
Bit:       1  1  1 r8  0  0  0 r4  1 r2 r1
```

- r1 (3, 5, 7, 9, 11) = 1, 0, 0, 1, 1 → three 1s → **r1 = 1**
- r2 (3, 6, 7, 10, 11) = 1, 0, 0, 1, 1 → three 1s → **r2 = 1**
- r4 (5, 6, 7) = 0, 0, 0 → no 1s → **r4 = 0**
- r8 (9, 10, 11) = 1, 1, 1 → three 1s → **r8 = 1**

Sent: **11110000111**
</details>

**A2.** You receive this from a Hamming 7-4 coding: 10011110110. One bit is in error. Which bit is it?

<details><summary>Show answer</summary>

```
Position: 11 10  9  8  7  6  5  4  3  2  1
Bit:       1  0  0  1  1  1  1  0  1  1  0
```

Count the 1s in each group (including the check bit itself). Even = 0, odd = 1.

- r1 (1, 3, 5, 7, 9, 11) = 0, 1, 1, 1, 0, 1 → four 1s → **0**
- r2 (2, 3, 6, 7, 10, 11) = 1, 1, 1, 1, 0, 1 → five 1s → **1**
- r4 (4, 5, 6, 7) = 0, 1, 1, 1 → three 1s → **1**
- r8 (8, 9, 10, 11) = 1, 0, 0, 1 → two 1s → **0**

r8 r4 r2 r1 = 0110 = **bit 6**.
</details>

**A3.** You receive this from a Hamming 7-4 coding: 01101011110. What is the correct data that was sent?

<details><summary>Show answer</summary>

```
Position: 11 10  9  8  7  6  5  4  3  2  1
Bit:       0  1  1  0  1  0  1  1  1  1  0
```

- r1 (1, 3, 5, 7, 9, 11) = 0, 1, 1, 1, 1, 0 → four 1s → **0**
- r2 (2, 3, 6, 7, 10, 11) = 1, 1, 0, 1, 1, 0 → four 1s → **0**
- r4 (4, 5, 6, 7) = 1, 1, 0, 1 → three 1s → **1**
- r8 (8, 9, 10, 11) = 0, 1, 1, 0 → two 1s → **0**

r8 r4 r2 r1 = 0100 = **bit 4** is wrong. Bit 4 is a **check bit** (r4), so the data bits are all fine.

Data = positions 11, 10, 9, 7, 6, 5, 3 = **0111011**

> **Trap:** the question asks for the **data** (7 bits), not the whole corrected code word (11 bits).
</details>

---

## Part B: CRC (Lesson 11)

**Reminder:** a generator with n + 1 bits gives an **n-bit CRC**. Add n zeros to the data, divide using XOR, and the remainder is the CRC.

**B1.** Data 11010011, generator 10110. What is the 4-bit CRC?

<details><summary>Show answer</summary>

Add 4 zeros: 110100110000. Divide by 10110 (XOR whenever the leading bit is 1):

```
110100110000
10110
 11000
 10110
  11101
  10110
   10111
   10110
       10000
       10110
        0110   ← remainder
```

CRC = **0110**
</details>

**B2.** Generator 10101, data 101001. What is the 4-bit CRC?

<details><summary>Show answer</summary>

Add 4 zeros: 1010010000.

```
1010010000
10101
    110000
    10101
     11010
     10101
      1111   ← remainder
```

CRC = **1111**
</details>

---

## Part C: Frame error over a path (Lesson 9)

**C1.** A path has **four intermediate nodes**. Each link has a BER of 1 in a million. The frame size is 1,000 bits. What is the probability of frame error on the path, as a percentage to two decimal places?

<details><summary>Show answer</summary>

- 4 intermediate nodes → **5 links** (hops = nodes + 1).
- The frame crosses 5 links, so it's like 5 × 1,000 = 5,000 bits exposed to errors.
- p = 1 − (1 − 10⁻⁶)^5,000 = 0.0049875

= **0.50%**

> **Trap:** using 4 links gives 0.40%, which is wrong.
</details>

---

## Part D: The size of the link, a (Lessons 15–16)

**Reminder:** a = T_S ÷ T_F, where T_S = T_F + Prop + T_A + Prop (+ any processing).

**D1.** Link length 231 m, propagation speed 300,000 km/s, frame size 500 bits, rate 1 Gb/s, negligible processing, and the ACK comes back in another frame of equal size. Find a to two decimal places.

<details><summary>Show answer</summary>

- T_F = 500 ÷ 10⁹ = **0.5 µs**. T_A = 0.5 µs (equal size).
- Prop = 0.231 km ÷ 300,000 km/s = **0.77 µs**.
- T_S = 0.5 + 0.77 + 0.5 + 0.77 = **2.54 µs**.
- a = 2.54 ÷ 0.5 = **5.08**
</details>

**D2.** Link length 18.48 km, propagation speed 300,000 km/s, frame size 600 B, rate 2 Gb/s, negligible processing, and the ACK is in a control frame so small we can neglect it. Find a to two decimal places.

<details><summary>Show answer</summary>

- L = 600 × 8 = **4,800 bits**. T_F = 4,800 ÷ (2 × 10⁹) = **2.4 µs**.
- T_A ≈ 0 (neglected).
- Prop = 18.48 ÷ 300,000 = **61.6 µs**.
- T_S = 2.4 + 61.6 + 0 + 61.6 = **125.6 µs**.
- a = 125.6 ÷ 2.4 = **52.33**

> **Watch out:** if you accidentally count the ACK as a full frame (T_A = 2.4 µs) you get 53.33. The question says to neglect the ACK, so the working above gives 52.33.
</details>

---

## Part E: True or false

### Errors, CRC and framing

<details><summary>E1. A six-bit generator polynomial has a CRC that is six bits long.</summary>

**False.** The CRC is one bit shorter than the generator: **5 bits**.
</details>

<details><summary>E2. A sixteen-bit generator polynomial has a CRC that is fifteen bits long.</summary>

**True.** CRC length = generator length − 1.
</details>

<details><summary>E3. The generator for a CRC-8 could be represented by 10110010.</summary>

**False.** CRC-8 means an 8-bit CRC, so the generator needs **9 bits**. This one has only 8.
</details>

<details><summary>E4. A generator for a CRC-32 can be represented as X^32 + X^26 + 1.</summary>

**True.** The highest power is 32, so the generator has 33 bits and the CRC has 32.
</details>

<details><summary>E5. Larger frames have a higher probability of error p than smaller frames with the same BER.</summary>

**True.** p = 1 − (1 − p_b)^L grows as L grows.
</details>

<details><summary>E6. When I divide the incoming bit sequence by the generator polynomial and there is no remainder, I assume the frame has not been corrupted.</summary>

**True.** Remainder 0 = accept the frame.
</details>

<details><summary>E7. It is not possible to have an error in a frame when the remainder is zero.</summary>

**False.** It's very unlikely but possible: some error patterns happen to be divisible by the generator. That's why E6 says "**assume**".
</details>

<details><summary>E8. The data link layer is only concerned with the framing of bits and transmission errors.</summary>

**False.** It also does **flow control** (and error control using ARQ).
</details>

<details><summary>E9. A two-dimensional parity scheme can detect up to three bits in error.</summary>

**True.** It detects all 1-, 2- and 3-bit errors.
</details>

<details><summary>E10. If there are five 1s in a row followed by a 0, there is no need to use bit stuffing.</summary>

**False.** You **always** stuff a 0 after five 1s, whatever comes next. Otherwise the receiver couldn't tell a real 0 from a stuffed one.
</details>

<details><summary>E11. When we know in advance the size of a block of data to be bit stuffed, we can work out how many extra bits we will get.</summary>

**False.** The number of stuffed bits depends on the **content** (how many runs of five 1s), not the size.
</details>

<details><summary>E12. Bit stuffing ensures that the flags (01111110) at the start and end of the frame are unique.</summary>

**True.** After stuffing, six 1s in a row can never appear in the data.
</details>

<details><summary>E13. If the Hamming distance between code words is 7, we can correct 4 bit errors.</summary>

**False.** Correct fewer than D ÷ 2 = 3.5, so up to **3**. (It can **detect** up to 6.)
</details>

<details><summary>E14. When handing over in a mobile network, to make it seamless we have to make a connection before we break the prior connection.</summary>

**True** (the quiz answer). This is "make before break". *Note: this one isn't in the lecture text used for these lessons, so just learn it as a fact.*
</details>

### The size of the link, a

<details><summary>E15. The parameter a is the number of frames that could be sent back to back until we get a response from the receiver.</summary>

**True.** a = T_S ÷ T_F: how many frame-times fit into one round trip.
</details>

<details><summary>E16. If we double the length of the link, a increases.</summary>

**True.** A longer link means more propagation delay, so T_S grows while T_F stays the same.
</details>

<details><summary>E17. If we increase the bit rate on a link, a will decrease.</summary>

**False.** A higher bit rate makes T_F **smaller**, and the propagation delay doesn't change, so a = T_S ÷ T_F **increases**.
</details>

### Stop-and-Wait, Go-back-N and Selective Repeat

<details><summary>E18. If a is ten, the maximum efficiency of Stop-and-Wait is 10%.</summary>

**True.** Maximum efficiency = 1 ÷ a = 1 ÷ 10.
</details>

<details><summary>E19. When I increase the bit rate in Stop-and-Wait, the efficiency increases.</summary>

**False.** A higher bit rate increases a (see E17), so the efficiency 1 ÷ a **falls**.
</details>

<details><summary>E20. When the propagation delay in Stop-and-Wait increases, the efficiency also increases.</summary>

**False.** More propagation delay means a bigger a and a **lower** efficiency.
</details>

<details><summary>E21. When I increase the frame size, I increase the probability of frame error.</summary>

**True.** Same reason as E5.
</details>

<details><summary>E22. In Stop-and-Wait, as there is only one frame outstanding, we do not have to number the frames.</summary>

**False.** Frames are numbered 0, 1, 0, 1 (the Alternating Bit Protocol) so the receiver can spot a **duplicate** after a lost ACK.
</details>

<details><summary>E23. The efficiency of Go-back-N is better than Stop-and-Wait.</summary>

**True.** With no errors Go-back-N approaches 100%, but Stop-and-Wait is stuck at 1 ÷ a.
</details>

<details><summary>E24. Selective Repeat is not always more efficient than Go-back-N.</summary>

**False** (the quiz answer). The slides call Selective Repeat an "improvement over Go Back N" that "attains the theoretical maximum throughput", so it is treated as always at least as good. Its cost is **complexity**, not efficiency.
</details>

---

## What to remember

- **Hamming:** the syndrome r8 r4 r2 r1 gives the position. If that position is 1, 2, 4 or 8, only a check bit was wrong and the **data is unchanged**.
- **CRC length = generator length − 1.** CRC-n has a generator with n + 1 bits and a highest power of X^n.
- **Path error:** intermediate nodes + 1 = links. Use L × links in the exponent.
- **a:** grows with link length and with bit rate. A bigger a means a lower Stop-and-Wait efficiency (max = 1 ÷ a).
- **Neglect the ACK** means T_A = 0. **Equal-size ACK** means T_A = T_F.
- **Efficiency order:** Stop-and-Wait < Go-back-N ≤ Selective Repeat.
