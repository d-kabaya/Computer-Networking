# Lesson 16: Stop-and-Wait ARQ Performance

[← Flow control and ARQ](15-flow-error-control-arq.md) · [Index](../README.md)

*Based on the "DataLink – Flow and Error Control" lecture deck, slides 18–25, and Problem Set Stop and Wait.*

## The big idea

With Stop-and-Wait, the sender spends a lot of its time **waiting**. **Efficiency** measures how much of the time is spent actually delivering **new** frames:

> **Efficiency = T_F ÷ D**, where D is the average time to get one frame through successfully.

It's a fraction from 0 to 1. Multiply by 100 for a percentage.

## 1. The symbols (slide 18)

| Symbol | Meaning |
|---|---|
| **T_F** | Time to transmit a data frame = L ÷ B |
| **T_A** | Time to transmit the ACK/NAK. If the ACK is **piggybacked**, T_A = T_F |
| **Prop** | Propagation delay = distance ÷ speed |
| **Proc** | Processing time at the sender or the receiver |
| **p** | Probability of a frame error, sender → receiver |
| **q** | Probability of a frame error, receiver → sender (the ACK) |
| **T_S** | Time for one **successful** round: frame out, ACK back |
| **a** | T_S ÷ T_F |

## 2. Success and failure times (slides 18–22)

**Success** (the frame and its ACK both OK):
```
T_S = T_F + T_A + 2 × (Prop + Proc)
```
If the two ends have **different** processing times, add each one: T_S = T_F + Prop + Proc_receiver + T_A + Prop + Proc_sender.

It happens with probability (1 − p)(1 − q).

**Failure** (anything goes wrong) happens with probability **r = p + (1 − p) × q**.

| What goes wrong | Time wasted T_Fail |
|---|---|
| Frame damaged, so a **NAK** comes back | T_Fail = T_S |
| Frame **lost** | T_Fail = T_F + Timeout |
| ACK or NAK **lost** | T_Fail = T_F + Timeout |

## 3. The general formulas (slide 23)

```
E = average transmissions per frame = 1 ÷ (1 − r)
D = average delay per frame        = (E − 1) × T_Fail + T_S
Efficiency                          = T_F ÷ D
```

**Why?** On average, E − 1 attempts fail (each wasting T_Fail), and then one succeeds (taking T_S).

## 4. The special cases you'll actually use (slides 24–25)

With the **optimal timeout** = T_A + 2 × (Prop + Proc), a failure wastes exactly as long as a success, so **T_Fail = T_S** and **D = E × T_S**.

| Case | E | Efficiency |
|---|---|---|
| **No errors** at all | 1 | **T_F ÷ T_S = 1 ÷ a** (the maximum) |
| **No errors in ACKs** (q = 0) | 1 ÷ (1 − p) | **(1 − p) ÷ a** |
| **Symmetrical** (p = q, e.g. piggybacked ACKs) | 1 ÷ (1 − p)² | **(1 − p)² ÷ a** |

where p = 1 − (1 − p_b)^L, the frame error formula from Lesson 9.

**Slide 25** plots efficiency against BER: efficiency falls as the BER rises, and a larger **a** means a lower efficiency curve overall.

## 5. The method

1. Work out **T_F** = L ÷ B (bits!).
2. Work out **T_S**: add up every part of the round trip.
3. **a** = T_S ÷ T_F.
4. Work out **p** from the BER: p = 1 − (1 − p_b)^L.
5. Pick the right case: no errors, q = 0, or symmetrical.
6. Efficiency = (1 − p)^(1 or 2) ÷ a.

**Working backwards (max BER for a target efficiency η):**
```
q = 0:         (1 − p) ≥ η × a    → (1 − p_b)^L ≥ η × a    → p_b ≤ 1 − (η × a)^(1/L)
symmetrical:   (1 − p)² ≥ η × a   → (1 − p_b)^(2L) ≥ η × a → p_b ≤ 1 − (η × a)^(1/(2L))
```

---

## Problem Set Stop and Wait (with worked answers)

Full worked solutions are also in [`resources/Stop_and_Wait_Worked_Solutions.pdf`](../resources/Stop_and_Wait_Worked_Solutions.pdf).

**Q1.** A Stop-and-Wait scheme has a frame length of 21,000 bits on a 100 Mb/s link that is 2 km long, with a propagation speed of 200,000 km/s. The ACK is sent back in a frame of similar length, and the return link is also 2 km at 200,000 km/s. The receiver processing time is 30 µs and the sender processing time is 60 µs. What is the maximum efficiency?

<details><summary>Show answer</summary>

- T_F = 21,000 ÷ 100 × 10⁶ = **210 µs**. T_A = T_F = 210 µs (similar length).
- Prop = 2 km ÷ 200,000 km/s = **10 µs** each way.
- T_S = 210 + 10 + 30 + 210 + 10 + 60 = **530 µs**.
- a = 530 ÷ 210 = **2.524**.
- Maximum efficiency (no errors) = 1 ÷ a = 210 ÷ 530 = **0.396 ≈ 39.6%**.
</details>

**Q2.** The same scheme as Q1, with a bit error probability of 1 in a million in both directions. What is the efficiency now? (Assume the optimal timeout.)

<details><summary>Show answer</summary>

- p = 1 − (1 − 10⁻⁶)^21,000 = **0.0208**. The ACK frame is the same length, so q = p (the **symmetrical** case).
- Efficiency = (1 − p)² ÷ a = 0.9792² ÷ 2.524 = **0.380 ≈ 38.0%**.
</details>

**Q3.** As in Q1, but now the ACK is sent **by itself**, not piggybacked. Assume the same value of a as in Q1, and that the probability of an ACK error is zero. With the optimal timeout, what is the maximum efficiency?

<details><summary>Show answer</summary>

- Now q = 0, so only the data frame can fail. With the BER of 1 in a million from Q2, p = 0.0208.
- Efficiency = (1 − p) ÷ a = 0.9792 ÷ 2.524 = **0.388 ≈ 38.8%**.
- (If you assume no bit errors at all, it's 1 ÷ a = 39.6%, the same as Q1. The question builds on Q2, so 38.8% is the intended answer.)
</details>

**Q4.** As in Q3, the goal is at least 20% efficiency. What is the maximum bit error rate allowed?

<details><summary>Show answer</summary>

- (1 − p) ÷ a ≥ 0.2, so (1 − p) ≥ 0.2 × 2.524 = **0.5048**.
- (1 − p_b)^21,000 ≥ 0.5048, so p_b ≤ 1 − 0.5048^(1/21,000) = **3.26 × 10⁻⁵**.
</details>

**Q5.** A Stop-and-Wait scheme has a = 1.75 and a frame length of 680 bytes. What is the efficiency when the probability of bit error is 1 in five hundred thousand? (Optimal timeout, and the ACKs are sent on their own, so q = 0.)

<details><summary>Show answer</summary>

- L = 680 × 8 = **5,440 bits**. p_b = 1 ÷ 500,000 = 2 × 10⁻⁶.
- p = 1 − (1 − 2 × 10⁻⁶)^5,440 = **0.0108**.
- Efficiency = (1 − p) ÷ a = 0.9892 ÷ 1.75 = **0.565 ≈ 56.5%**.
</details>

**Q6.** For the scheme in Q5, what is the maximum bit error rate for an efficiency of at least 40%?

<details><summary>Show answer</summary>

- (1 − p) ≥ 0.4 × 1.75 = **0.7**.
- (1 − p_b)^5,440 ≥ 0.7, so p_b ≤ 1 − 0.7^(1/5,440) = **6.56 × 10⁻⁵**.
</details>

---

## What to remember

- **Efficiency = T_F ÷ D**, the fraction of time spent delivering new frames.
- **a = T_S ÷ T_F**, and the best possible efficiency is **1 ÷ a**.
- **Optimal timeout** means T_Fail = T_S.
- **ACKs on their own (q = 0):** efficiency = (1 − p) ÷ a.
- **Piggybacked ACKs (p = q):** efficiency = (1 − p)² ÷ a.
- Always: **L in bits**, p = 1 − (1 − p_b)^L, and include **both** processing times in T_S.
