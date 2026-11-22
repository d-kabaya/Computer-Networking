# Lesson 18: Go-back-N and Selective Repeat

[← Quiz practice](17-quiz-practice-data-link-flow.md) · [Index](../README.md) · [Next: HDLC, LLC and shared links →](19-hdlc-llc-multi-access.md)

*Based on the "DataLink – Flow and Error Control" lecture deck, slides 26–35.*

## The big idea

Stop-and-Wait wastes time: after every frame, the sender sits idle waiting for the ACK. On a link with a large **a**, that's most of the time (Lesson 16).

The fix is the **sliding window** from Lesson 15: let the sender keep sending frames **while** it waits for ACKs. That raises a new question: **if one frame goes wrong, what happens to the frames sent after it?** The two schemes in this lesson answer it differently.

| | Go-back-N | Selective Repeat |
|---|---|---|
| **After an error, resend…** | The bad frame **and every frame after it** | **Only** the bad frame |
| **Receiver accepts out-of-order frames?** | ❌ No, it discards them | ✅ Yes, it stores them and reorders later |
| **Receiver needs a buffer?** | No | Yes |
| **Complexity** | Simpler | Much more complex sender and receiver |

> **Analogy:** you're reading out a list of 10 phone numbers and number 3 is misheard.
> - **Go-back-N:** the listener says "I lost you at number 3" and you read 3, 4, 5, … 10 all over again.
> - **Selective Repeat:** the listener writes down 4–10 anyway and just says "repeat number 3".

---

## 1. Go-back-N (slide 26)

- The sender **transmits frames continuously** if it can.
- At most **N frames are outstanding** (sent but not yet ACKed) at any time. That's the window.
- Frames are numbered **modulo N**, so the numbers must be big enough for good performance. If they're too small, you get **sequence number starvation**: the sender runs out of numbers to put on new frames and has to stop and wait.
- An error is spotted when a frame arrives **with a CRC remainder**, or when **N(S) skips a number** (a frame went missing). The sender then "rolls back the clock" and resends from that frame.

**The key rule:** the receiver only accepts correct frames **in the right order**. Anything else is discarded, so the receiver never has to buffer and reorder frames.

### Case 1: damaged frame (slide 27)

```
Sender                         Receiver
Data 0 ───────────────────────> 0 ✓
Data 1 ───────────────────────> 1 ✓
Data 2 ───────────────────────> 2 ✓        ──> ACK 3
Data 3 ───────────────────────> 3 ✗ error  ──> NAK 3
Data 4 ───────────────────────> discarded (expected 3)
Data 5 ───────────────────────> discarded (expected 3)
Data 3 ───────────────────────> 3 ✓        ← sender went back to 3
Data 4 ───────────────────────> 4 ✓
Data 5 ───────────────────────> 5 ✓
```

**NAK 3** means two things at once: "resend frame 3", and "everything **before** 3 arrived correctly".

Frames 4 and 5 were probably fine, but they're thrown away anyway, because the receiver is still waiting for 3.

### Case 2: lost frame (slide 28)

Frame 2 is lost. Frames 3 and 4 arrive correctly, but the receiver sees **N(S) skip** from 1 to 3, so it sends **NAK 2** and discards 3 and 4. The sender resends 2, 3, 4.

> Frame 3 is discarded **even though it arrived correctly**, because the receiver was expecting frame 2.

### Case 3: lost ACK (slide 29)

Frames 0, 1, 2 arrive fine and the receiver sends **ACK 3**, but the ACK is lost. The sender's **timer runs out**, so it resends 0, 1, 2. The receiver recognises them as **copies, discards** them, and sends ACK 3 again.

---

## 2. Go-back-N performance (slides 30–31)

The slide assumes:
- the **optimal timeout** = T_A + 2 × (Prop + Proc), as in Lesson 16
- a window big enough that the sender **never has to stop** when there are no errors

With **r = p + (1 − p) × q** (the chance that a frame or its ACK fails, as before):

```
Average frame delay   D = [T_F + r × Timeout] ÷ (1 − r)
Average throughput      = (1 − r) ÷ [T_F + r × Timeout]      frames per second
Efficiency              = (1 − r) × T_F ÷ [T_F + r × Timeout]
```

**What the formula is saying:** each frame costs T_F to send. A failure (probability r) costs an extra **timeout** while the sender waits to find out. With no errors (r = 0), the efficiency is T_F ÷ T_F = **1, or 100%**. Go-back-N can deliver frames continuously.

**Compare Stop-and-Wait** (slide 30) under the same conditions:
```
Stop-and-Wait efficiency = (1 − r) × T_F ÷ [T_F + Timeout]
```
The whole timeout is in the bottom **every time**, not just r of the time. As r → 0 it only reaches T_F ÷ (T_F + Timeout) = T_F ÷ T_S = 1 ÷ a. That's the "built-in inefficiency" of Stop-and-Wait.

> **Link to Lesson 16:** 1 − r = (1 − p)(1 − q). With q = 0 that's (1 − p), and with p = q it's (1 − p)², so this is the same Stop-and-Wait formula you already know, written a different way.

### A shortcut using a *(Beyond the notes: an algebra step, not on the slide)*

With the optimal timeout, T_F + Timeout = T_S, so **Timeout ÷ T_F = a − 1**. Dividing the top and bottom of the efficiency by T_F:

```
Go-back-N efficiency = (1 − r) ÷ [1 + r × (a − 1)]
Stop-and-Wait        = (1 − r) ÷ a
```

### Worked example

Use the link from Problem Set Stop and Wait Q1–Q2: a = 2.524, frame error p = 0.0208, and q = p (symmetrical).

1. r = p + (1 − p) × q = 0.0208 + 0.9792 × 0.0208 = **0.0411**
2. Go-back-N = (1 − 0.0411) ÷ [1 + 0.0411 × 1.524] = 0.9589 ÷ 1.0627 = **90.2%**
3. Stop-and-Wait = 0.9589 ÷ 2.524 = **38.0%** (the same as Q2 ✓)

Go-back-N is more than twice as efficient on the same link.

---

## 3. Selective Repeat (slides 32–33)

**Why go back and resend everything? Some of those frames were fine.** Selective Repeat resends **only the bad frames**.

- It's an **improvement over Go-back-N**, but the improvement only matters when:
  - **a is large** (many frames are in flight at once), **and**
  - there is likely to be a frame in error on the link because of the BER.
- It **attains the theoretical maximum throughput**.
- Frames arrive **out of order**, so the receiver must **store and reorder** them.
- The sender and receiver are **more complex**, so use it only when needed.

### Damaged frame (slide 33)

```
Sender                         Receiver
Data 0 ───────────────────────> 0 ✓
Data 1 ───────────────────────> 1 ✓
Data 2 ───────────────────────> 2 ✗ error   ──> NAK 2
Data 3 ───────────────────────> 3 ✓ stored ┐
Data 4 ───────────────────────> 4 ✓ stored ├ can't be ACKed yet
Data 5 ───────────────────────> 5 ✓ stored ┘
Data 2 ───────────────────────> 2 ✓  (only frame 2 is resent)
                                  ──> ACK 6: deliver 2, 3, 4, 5 to the network layer
```

- Frames 3–5 are **stored** but **can't be ACKed yet**, because an ACK means "everything before this number arrived" and frame 2 is still missing.
- Once frame 2 arrives, the receiver delivers 2–5 **in order** to its network layer and sends **ACK 6**.
- The sender must be able to **pick out individual frames** to resend.
- Lost or damaged ACKs and NAKs are handled the same way as in Go-back-N.

---

## 4. Comparing all three (slides 34–35)

The slides plot **throughput against BER** (a log scale, so −6 means 10⁻⁶) for 10,000-bit packets:

```
throughput
 1 ┤━━━━━━━━━━━━━━━━━━━╮╮
   │                    ╲╲  ← Selective Repeat (highest)
   │                     ╲ ╲
   │                      ╲  ╲ ← Go-back-N
   │━━━━━━━━━━━━━━━━━━━━━━━╲━━╲ ← Stop-and-Wait (flat and low)
 0 ┼────────────────────────────────── BER →
   10⁻⁷     10⁻⁶     10⁻⁵     10⁻⁴
```

What to read from the graph:
- **Low BER:** Go-back-N and Selective Repeat are both close to **100%**. Stop-and-Wait is stuck at about **1 ÷ a** (around 0.3 on slide 34, and 0.1 on slide 35), no matter how clean the link is.
- **As the BER rises:** Go-back-N falls first, because each error throws away a whole window of frames. Selective Repeat holds up longer.
- **Very high BER:** all three collapse towards 0, because almost every frame is damaged.

**Order of efficiency:** Stop-and-Wait < Go-back-N ≤ Selective Repeat

---

## Quiz

**Q1.** In Go-back-N, frame 3 arrives with a CRC error, and frames 4 and 5 arrive correctly. What happens to 4 and 5, and why?

<details><summary>Show answer</summary>

They're **discarded**. The receiver only accepts frames **in order**, and it's still waiting for frame 3. The sender then resends 3, 4 and 5.
</details>

**Q2.** What two things does NAK 3 tell the sender?

<details><summary>Show answer</summary>

1. **Resend frame 3.**
2. Every frame **before 3** (0, 1, 2) arrived correctly.
</details>

**Q3.** How does a Go-back-N receiver notice a **lost** frame, when no damaged frame ever arrives?

<details><summary>Show answer</summary>

**N(S) skips a number.** For example, it receives frame 1 and then frame 3, so it knows frame 2 is missing.
</details>

**Q4.** What is sequence number starvation?

<details><summary>Show answer</summary>

The sender **runs out of sequence numbers** to put on new frames (all of them are used by outstanding, un-ACKed frames), so it has to stop and wait. It happens when the modulo N is too small.
</details>

**Q5.** In Selective Repeat, frame 2 is damaged and frames 3, 4, 5 arrive correctly. Why can't the receiver ACK frame 5 straight away?

<details><summary>Show answer</summary>

An ACK means "**everything before this number** has arrived". ACK 6 would claim frame 2 arrived, which isn't true. Frames 3–5 are stored, and ACK 6 is sent once frame 2 arrives.
</details>

**Q6.** When is Selective Repeat worth its extra complexity?

<details><summary>Show answer</summary>

When **a is large** (lots of frames in flight) **and** errors are likely enough that a frame on the link is often in error. Otherwise Go-back-N does nearly as well.
</details>

**Q7.** With no errors at all (r = 0), what is the efficiency of Go-back-N? Of Stop-and-Wait?

<details><summary>Show answer</summary>

- Go-back-N: (1 − 0) × T_F ÷ (T_F + 0) = **100%**
- Stop-and-Wait: T_F ÷ (T_F + Timeout) = T_F ÷ T_S = **1 ÷ a**
</details>

**Q8.** a = 10, p = 0.1, q = 0, optimal timeout. Find the efficiency of Go-back-N and of Stop-and-Wait.

<details><summary>Show answer</summary>

- r = 0.1 + 0.9 × 0 = 0.1
- Go-back-N = (1 − r) ÷ [1 + r(a − 1)] = 0.9 ÷ (1 + 0.1 × 9) = 0.9 ÷ 1.9 = **47.4%**
- Stop-and-Wait = (1 − r) ÷ a = 0.9 ÷ 10 = **9%**
</details>

**Q9.** T_F = 1 ms, the timeout is 2 ms and r = 0.1. What is the average frame delay D for Go-back-N?

<details><summary>Show answer</summary>

D = [T_F + r × Timeout] ÷ (1 − r) = (1 + 0.1 × 2) ÷ 0.9 = 1.2 ÷ 0.9 = **1.33 ms**
</details>

---

## What to remember

- **Go-back-N:** in-order only, no receiver buffer, resend the bad frame **and everything after it**.
- **Selective Repeat:** store out-of-order frames, resend **only the bad one**, and ACK once the gap is filled. More complex, and it reaches the theoretical maximum throughput.
- **NAK k:** resend k, and everything before k is fine.
- **Errors spotted by:** a CRC remainder, or N(S) skipping a number.
- **Go-back-N efficiency** = (1 − r) × T_F ÷ [T_F + r × Timeout] → 100% when r = 0.
- **Stop-and-Wait** = (1 − r) × T_F ÷ [T_F + Timeout] → 1 ÷ a when r = 0.
- **r = p + (1 − p) × q.**
- **Order:** Stop-and-Wait < Go-back-N ≤ Selective Repeat.
