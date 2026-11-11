# Lesson 15: Flow Control, Error Control and ARQ

[← Exam revision quiz 2](14-exam-revision-quiz-2.md) · [Index](../README.md) · [Next: Stop-and-Wait performance →](16-stop-and-wait-performance.md)

*Based on the "DataLink – Flow and Error Control" lecture deck, slides 1–17.*

## The big idea

Lessons 10–11 showed how the receiver can **detect** a damaged frame (with a CRC). But then what? Trying to **correct** everything would need huge overhead, and we usually don't know in advance how bad a link is. So instead:

> **ARQ (Automatic Repeat reQuest):** send the frame, check it, and if it's bad, **ask the sender to send it again**. Forget correction.

- If the frame is correct → pass it up to the higher layers.
- If the frame is in error → get the sender to **retransmit** it.

> **Analogy:** reading a phone number down a bad line. You don't try to guess the missing digits. You say "sorry, can you repeat that?"

## 1. Number everything (slide 3)

| Event | What happens |
|---|---|
| Frame arrives, **CRC OK** | The receiver sends an **ACK** (acknowledgement) |
| Frame arrives, **CRC has a remainder** | The receiver sends a **NAK** (negative ACK) |
| Frame **missing** | The sender's **timer** runs out (a timeout) |

The sequence numbers go in the frame's **control field**:
- **N(S):** the number of the frame being **sent**
- **N(R):** the number of the **next frame requested**

## 2. The key timings (slides 4–5)

```
T_F = time to transmit a frame = frame length L (bits) ÷ bit rate B

T_S = time from starting to send a frame until its ACK is back
    = T_F + propagation + receiver processing
      + ACK transmit time + propagation + sender processing

a = T_S ÷ T_F      ("the size of the link")
```

**a** compares the round-trip time with the frame time. **a = 1** would mean no waiting at all; a large **a** means lots of waiting.

**Slide 5 examples:**

| Link | T_F | a |
|---|---|---|
| Low-speed terrestrial: 1,200 bits at 9.6 kb/s, 100 km | 125 ms | (125 + 0.4 + 125 + 0.4) ÷ 125 ≈ **2** |
| High-speed ATM: 53 B at 620 Mb/s, 100 km | 0.68 µs | (0.68 + 400 + 0.68 + 400) ÷ 0.68 ≈ **1,178** (a ≫ 1) |
| Satellite: 36,000 km up, 120 ms each way, T_F = 125 ms | 125 ms | (125 + 240 + 125 + 240) ÷ 125 = **5.84** |

## 3. Flow control vs error control (slide 6)

| | Flow control | Error control |
|---|---|---|
| **Question** | How much can the sender send before it needs permission? | What happens when frames are damaged or lost? |
| **Why** | A receiver that's too slow has to **drop** frames even when they're error-free | Damaged or lost frames must be **re-sent** |
| **How** | Signal the sender to **slow down** (explicitly, or by delaying ACKs) | **ACK** good frames, **NAK** bad ones. The sender keeps copies of un-ACKed frames. |

They're closely linked, because the same ACKs are used for both. The aim is that packets reach the receiver's **network layer in order**.

## 4. Stop-and-Wait flow control (slide 7)

The sender sends **one frame**, then **waits for its ACK** before sending the next.

```
Sender                Receiver
  |──── Data ────────────>|
  |<──────────── ACK ─────|   ← wait time
  |──── Data ────────────>|
  |<──────────── ACK ─────|
```

- The ACK can be a frame on its own, or a field inside a data frame going the other way. That's called **piggybacking**.
- ✅ **Simple.** ❌ **Inefficient**, because the sender is idle while it waits.

## 5. Sliding window flow control (slides 8–12)

The sender can send **several frames in a row** before it needs an ACK.

- Frames are numbered **modulo n**: 0, 1, …, n − 1, 0, 1, …
- **Window size = n − 1** (one less than the number of different frame numbers).
- **ACK k** means "I've got everything up to k − 1; send **k** next". So **ACK 4** confirms frames up to and including **3**.
- One ACK can confirm **several** frames.

```
Sender's window (n = 8, window size 7):
 6 7 [0 1 2 3 4 5 6] 7 0 1 ...
       ← frames that can be sent without waiting for an ACK
```

**How the window moves:**

| | Left edge moves right… | Right edge moves right… |
|---|---|---|
| **Sender's window** | by one each time a frame is **sent** | by several when an **ACK is received** |
| **Receiver's window** | by one each time a frame is **received** | by several when an **ACK is sent** |

The receiver's window represents the number of un-ACKed frames it can still accept.

## 6. ARQ error control (slides 13–14)

- The **NAK** can be **explicit** (sent by the receiver) or **implicit** (the sender's timer runs out).
- The sender keeps a **copy** of each un-ACKed frame:
  - **ACK** received → discard the copy
  - **NAK** received, or **timeout** → decide how to retransmit
- The sender starts a **timer** for each frame when it's sent. The best timeout is the **expected time for the ACK to come back** (in practice, a little longer).
- A packet isn't considered delivered until the sender **gets its ACK**.

**The 3 types of ARQ:**

| Type | Built on | Key idea |
|---|---|---|
| **1. Stop-and-Wait ARQ** | Stop-and-Wait flow control | One frame at a time |
| **2. Go-back-n ARQ** | Sliding window | The receiver must get frames **in order**. After an error, resend from the bad frame onward. |
| **3. Selective repeat ARQ** | Sliding window | Correct frames that arrive **out of order** are **stored** and put back in order later |

## 7. Stop-and-Wait ARQ in action (slides 15–17)

It's also called the **Alternating Bit Protocol (ABP)**, because frames are just numbered **0, 1, 0, 1, …**

**Damaged frame:** the receiver sends a **NAK**, and the sender resends the same frame.
```
Data 1 ──> OK      <── ACK 0
Data 0 ──> OK      <── ACK 1
Data 1 ──> ✗ error <── NAK
Data 1 ──> OK      <── ACK 0      ← resent
```

**Lost frame:** nothing arrives, so the receiver sends nothing. The sender's **timer runs out**, and it resends.

**Lost ACK:** the frame arrived fine, but the ACK was lost. The sender times out and **resends** the frame. The receiver sees it's a **duplicate**, **discards** the second copy, and sends the ACK again.

> **Slide 17 question:** why does the receiver send **ACK 0** for the copy of data frame 1?
> **Answer:** ACK 0 means "I've got frame 1; send frame **0** next". The receiver already has frame 1, so it's still waiting for frame 0. It repeats the same ACK, so the sender knows frame 1 arrived and can move on.

---

## Quiz

**Q1.** What does ARQ stand for, and what does it do instead of correcting errors?

<details><summary>Show answer</summary>

**Automatic Repeat reQuest.** It **detects** errors (e.g. with a CRC) and asks the sender to **retransmit** the frame.
</details>

**Q2.** What's the difference between flow control and error control?

<details><summary>Show answer</summary>

**Flow control:** stops the sender sending faster than the receiver can handle. **Error control:** gets damaged or lost frames re-sent.
</details>

**Q3.** A 1,200-bit frame on a 9.6 kb/s link. What is T_F?

<details><summary>Show answer</summary>

1,200 ÷ 9,600 = **125 ms**.
</details>

**Q4.** With sliding window, frames are numbered modulo 8. What is the window size?

<details><summary>Show answer</summary>

n − 1 = **7**.
</details>

**Q5.** The sender receives ACK 4. What does it know?

<details><summary>Show answer</summary>

Frames up to and including **3** arrived correctly, and the receiver wants frame **4** next.
</details>

**Q6.** What's the difference between Go-back-n and Selective repeat?

<details><summary>Show answer</summary>

**Go-back-n:** the receiver accepts only **in-order** frames, so after an error everything from that frame onward is resent. **Selective repeat:** correct out-of-order frames are **stored**, and only the bad one is resent.
</details>

**Q7.** In Stop-and-Wait ARQ, an ACK is lost. What happens?

<details><summary>Show answer</summary>

The sender's **timer expires** and it resends the frame. The receiver recognises the **duplicate**, discards it, and re-sends the ACK.
</details>

**Q8.** What is piggybacking?

<details><summary>Show answer</summary>

Putting the ACK inside a **data frame** going the other way, instead of sending a separate ACK frame.
</details>
