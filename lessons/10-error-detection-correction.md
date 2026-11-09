# Lesson 10: Error Detection and Correction

[← Physical layer and bit errors](09-physical-layer-bit-errors.md) · [Index](../README.md) · [Next: CRC and framing →](11-crc-and-framing.md)

*Based on the "Data Link Errors" lecture deck, slides 15–36.*

## The big idea

The physical layer can flip bits. The **data link layer (L2)** sits on top of it and tries to give **reasonably reliable** communication between **adjacent** devices. Its jobs are:

1. **Framing:** deciding where each frame starts and ends (Lesson 11)
2. **Dealing with transmission errors:** this lesson
3. **Flow control:** making sure the sender doesn't overwhelm the receiver

The trick is **redundancy**: adding extra **check bits** that let the receiver spot, and sometimes fix, errors.

> **Analogy:** reading out a phone number and then repeating the last digit as a "check". If the repeated digit doesn't match, you know you misheard something.

## 1. Simple approaches

### Send every frame 3 times
The receiver takes the **majority** value for each bit.
- p ≈ 3 × L × p_b², which is much smaller. For example, p_b = 10⁻⁷ and L = 10,000 give **3 × 10⁻¹⁰**.
- ❌ The transmission rate drops to **⅓**, and checking 3 copies adds delay.

### Send every frame twice
If the copies match, accept. If they differ, ask for a **retransmission**.
- ❌ The transmission rate is **halved**, and comparing the copies adds delay.

## 2. Codewords

A frame = **k data bits** + **n check (redundancy) bits**, so L = k + n.
The whole (k + n)-bit string is called a **codeword**.

## 3. Single parity check (n = 1)

Add one **parity bit** so the total number of 1s is **even** ("even 1s parity").

- An odd number of 1s in the data → parity bit **1**
- An even number of 1s in the data → parity bit **0**

**Example:** data `10110101` has five 1s, so the parity bit is 1 and the codeword is `101101011`.

| Received | Check | Result |
|---|---|---|
| `101101011` | even ✓ | accepted, correct |
| `101100011` | odd ✗ | rejected, correct (error detected) |
| `101110011` | even ✓ | accepted, **wrong**: 2 errors went undetected |

✅ It detects **any odd number** of bit errors.
❌ It can't tell **which** bit is wrong, and it **misses any even number** of errors.

## 4. Two-dimensional parity

Arrange the data in a grid, add a parity bit to **every row and every column**, then send it row by row.

**Example:** data `1110001 1000111 0011001` (a 3 × 7 grid) becomes a 4 × 8 grid:

```
1 1 1 0 0 0 1 | 0     ← row parity
1 0 0 0 1 1 1 | 0
0 0 1 1 0 0 1 | 1
--------------+--
0 1 0 1 1 1 1 | 1     ← column parity
```

Transmitted: `11100010 10001110 00110011 01011111`

**One bit error** makes exactly **one row and one column** fail. Where they cross is the bad bit, so the receiver flips it back. ✅ It **corrects any single error**.

| Errors | What happens |
|---|---|
| 1 | Detected **and corrected** |
| 2 or 3 | Detected, but **can't be corrected** (3 may even be "corrected" wrongly) |
| 4 in a **rectangle** pattern | **Not detected** at all: every row and column still passes |

## 5. Hamming distance and minimum distance

The **Hamming distance** d(W1, W2) is the **number of bit positions where two words differ**.
Example: `10001001` vs `10110001` → they differ in 3 places, so d = **3**.

The **minimum distance D** of a scheme is the smallest distance between any two valid codewords. The receiver picks the **closest valid codeword** to what it received.

```
If the minimum distance is D:
  detects   any ≤ D − 1 errors
  corrects  any < D/2 errors

To detect B errors  → need D ≥ B + 1
To correct B errors → need D ≥ 2B + 1
```

**Why?** Each bit error moves the word one step. Fewer than D steps can't reach another valid codeword, so the receiver sees an invalid word and the error is **detected**. With fewer than D/2 steps, the original codeword is still the **closest** one, so the error is **corrected**.

| Scheme | D | Detects | Corrects |
|---|---|---|---|
| Single parity | 2 | 1 | 0 |
| 2-D parity | 4 | 3 | 1 |
| {00000 00000, 00000 11111, 11111 00000, 11111 11111} | 5 | 4 | 2 |

## 6. The Hamming code (7 data bits → 11-bit codeword)

**Layout:** positions 11 to 1. Redundancy bits sit at the **powers of 2**: positions 1, 2, 4 and 8. The 7 data bits fill positions 11, 10, 9, 7, 6, 5 and 3, from left to right.

```
Position: 11 10  9  8  7  6  5  4  3  2  1
          d  d   d  r8 d  d  d  r4 d  r2 r1
```

Each r bit makes **even parity** over its group:

| Bit | Checks positions |
|---|---|
| **r1** | 1, 3, 5, 7, 9, 11 (the binary position has bit 1 set) |
| **r2** | 2, 3, 6, 7, 10, 11 |
| **r4** | 4, 5, 6, 7 |
| **r8** | 8, 9, 10, 11 |

### Sending
**Slide example:** data `1001101` → codeword `10011100101` (r1 = 1, r2 = 0, r4 = 1, r8 = 1).

### Receiving
Recompute each parity group, **including** its r bit. Read the four results as a binary number **r8 r4 r2 r1**:
- `0000` → no error
- anything else → **that number is the position of the wrong bit**. Flip it.

**Example:** if bit 7 is received wrong, the checks give `0111` = **7**, so flip position 7.

---

## Quiz

**Q1.** Data `1110010`. What is the codeword with a single (even) parity bit?

<details><summary>Show answer</summary>

There are four 1s (even), so the parity bit is 0: **11100100**.
</details>

**Q2.** True or false: a single parity check can detect two bit errors.

<details><summary>Show answer</summary>

**False.** Two errors keep the number of 1s even, so the check still passes.
</details>

**Q3.** What is the Hamming distance between `1011101` and `1001001`?

<details><summary>Show answer</summary>

They differ in positions 3 and 5 (counting from the left), so **2**.
</details>

**Q4.** A scheme must correct up to 2 bit errors. What minimum distance does it need?

<details><summary>Show answer</summary>

D ≥ 2B + 1 = **5**.
</details>

**Q5.** Generate the Hamming codeword for data `1011101`.

<details><summary>Show answer</summary>

Place the data: positions 11–9 = 1 0 1, 7–5 = 1 1 0, 3 = 1.
r1 (3, 5, 7, 9, 11) = 1, 0, 1, 1, 1 → four 1s → **0**
r2 (3, 6, 7, 10, 11) = 1, 1, 1, 0, 1 → four 1s → **0**
r4 (5, 6, 7) = 0, 1, 1 → two 1s → **0**
r8 (9, 10, 11) = 1, 0, 1 → two 1s → **0**
Codeword = **10101100100**
</details>

**Q6.** The Hamming codeword `11001101101` is received. What data was sent?

<details><summary>Show answer</summary>

Recompute the checks: r1 = 0, r2 = 1, r4 = 1, r8 = 0. Reading r8 r4 r2 r1 gives 0110 = **6**, so flip position 6 from 1 to 0.
Corrected: 11001001101. Data from positions 11, 10, 9, 7, 6, 5, 3 = **1101001**.
</details>

**Q7.** Why can 2-D parity miss 4 errors arranged in a rectangle?

<details><summary>Show answer</summary>

Each affected row has **two** errors, and so does each affected column. Two flips keep the parity even, so every row and column check still passes.
</details>
