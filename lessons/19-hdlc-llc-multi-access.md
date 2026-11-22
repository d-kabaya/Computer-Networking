# Lesson 19: HDLC, LLC, Ethernet and Token Ring

[← Go-back-N and Selective Repeat](18-go-back-n-selective-repeat.md) · [Index](../README.md) · [Next: Wi-Fi (802.11) →](20-wifi-802-11.md)

*Based on the "DataLink – Flow and Error Control" lecture deck, slides 39–52.*

## The big idea

So far, every link has had **two ends**: one sender and one receiver. This lesson does two things:

1. **HDLC:** shows how everything from Lessons 11 and 15–18 (flags, CRC, sequence numbers, Go-back-N) fits together in one **real protocol** for a point-to-point link.
2. **Multi-access:** deals with links **shared by many nodes** (a LAN), where the new problem is **who gets to talk, and when**.

---

## 1. HDLC: High-level Data Link Control (slides 39–40)

### Three types of frame

| Frame | Name | Carries |
|---|---|---|
| **I-frame** | Information | **User data** plus control info about it (e.g. a piggybacked ACK) |
| **S-frame** | Supervisory | **Control info only**, e.g. an ACK or NAK |
| **U-frame** | Unnumbered | **Link management**: setting up and managing the link itself |

### Frame format

```
Bits:    8           8         8        ≥ 0      16         8
     ┌──────────┬─────────┬─────────┬────────┬──────────┬──────────┐
     │ 01111110 │ Address │ Control │  Data  │ Checksum │ 01111110 │
     └──────────┴─────────┴─────────┴────────┴──────────┴──────────┘
        flag                                     (FCS)      flag
```

- **Flag** = `01111110`, the same flag as in bit stuffing (Lesson 11).
- In back-to-back frames, the **end flag of one frame can be the start flag of the next**.
- **FCS** (Frame Check Sequence, also called the checksum) = a **2-byte or 4-byte CRC**.
- The data field can be **0 bits** long. S-frames carry no data at all.

### The control field

The first bits of the control field say which type of frame it is:

| Frame | Starts with | Then carries |
|---|---|---|
| I-frame | `0` | N(S), P/F, N(R) |
| S-frame | `1 0` | a code, P/F, N(R) |
| U-frame | `1 1` | codes and P/F |

- **N(S)** = the number of the frame being sent. **N(R)** = the number of the next frame expected (the ACK).
- **P/F** = the poll/final bit.

**HDLC uses Go-back-N ARQ:**

| Mode | N(S) and N(R) | Numbers | Window size |
|---|---|---|---|
| Normal | **3 bits** | 0–7 (modulo 8) | **7** ("Go-back-7") |
| Extended | **7 bits** | 0–127 (modulo 128) | **127** ("Go-back-127") |

This is the **window size = n − 1** rule from Lesson 15. A k-bit number gives n = 2ᵏ different values, so the window is 2ᵏ − 1.

---

## 2. Splitting the data link layer: LLC and MAC (slides 41–42)

HDLC is **not suitable for broadcast (multi-access) channels**, where many nodes share one medium. Since 1985, **IEEE Project 802** has defined standards for broadcast LANs. It splits the data link layer in two:

```
┌──────────────────────────────┐
│  LLC: Logical Link Control   │  ← the same for every LAN type (802.2)
├──────────────────────────────┤
│  MAC: Medium Access Control  │  ← different for each LAN technology
├──────────────────────────────┤
│  Physical                    │
└──────────────────────────────┘
```

| Sublayer | Job | Examples |
|---|---|---|
| **LLC** | Common to **all** broadcast LANs. Handles **logical addresses**, control information and data. | 802.2 |
| **MAC** | **Specific to the LAN technology.** Resolves **contention** for the channel, and supplies flags, error control and **physical addresses**. | 802.3 (CSMA/CD), 802.4 (Token Bus), 802.5 (Token Ring), 802.11 (Wi-Fi), 802.15 (Bluetooth) |

### The LLC-PDU

```
┌──────┬──────┬─────────┬─────────────┐
│ DSAP │ SSAP │ Control │ Information │
└──────┴──────┴─────────┴─────────────┘
```

- **DSAP** = Destination Service Access Point, **SSAP** = Source Service Access Point. These are **logical addresses**: which upper-layer process the data is for and from.
  - The first bit of DSAP: 0 = an individual address, 1 = a group address.
  - The first bit of SSAP: 0 = a command, 1 = a response.
- The **control field is identical to HDLC's**.
- The LLC-PDU has **no flags, no CRC and no physical (machine) addresses**. The **MAC sublayer supplies** all of those.

---

## 3. Sharing one channel: 3 methods (slide 43)

**The key issue:** which node can use the shared channel at any given time?
- When 2 or more nodes want to transmit at the same time, they **contend** for the channel.
- If 2 or more nodes **do** transmit at the same time, the usual assumption is that **all the transmissions collide and are destroyed**: none succeeds.

| Method | How it works | Example |
|---|---|---|
| **1. Divide the channel** | Split it into independent sub-channels, one per transmission, using **TDM or FDM**. This turns the shared channel into a set of point-to-point links. | Lesson 9 |
| **2. Collision resolution** | Nodes transmit whenever they like but **check for collisions**. After a collision, each node waits a **random** time and tries again. Improvement: **listen before and while** transmitting, and stop immediately on a collision. | Ethernet |
| **3. Reservations** | A node must hold the **token** (permission to transmit) before sending, and passes it on when finished. | Token Ring |

> **Why a random wait?** If both nodes waited the **same** time, they'd retry together and collide again, forever. Random waits make it likely that one goes first.

> **Analogy:** a group conversation.
> 1. **Divide:** everyone gets their own turn, or their own table.
> 2. **Collision resolution:** anyone can speak; if two start at once, both stop, pause for a random moment, and try again.
> 3. **Reservation:** only the person holding the "talking stick" can speak.

---

## 4. Ethernet and CSMA/CD (slides 44–46)

Ethernet was developed at **Xerox PARC in the 1970s** to connect 100 computers on a 1 km cable. IEEE 802.3 is the standard, and it's slightly different from the original Ethernet.

**Channel access method: CSMA/CD** = Carrier Sense Multiple Access with Collision Detection.

| Part | Meaning |
|---|---|
| **CS: Carrier Sense** | **Listen before sending.** If the channel is busy, wait until it's idle, then transmit. |
| **MA: Multiple Access** | Many nodes share one channel. |
| **CD: Collision Detection** | **Listen while sending.** If another transmission is detected, **all nodes involved stop immediately**, each waits a **random** time, then tries again. |

**The basic problem:** in theory, a node may **never** get to transmit. Even with random waits, its attempts could collide every time. The **Maximum Medium Access Time (MMAT) is unbounded (∞)**.

### Why collisions still happen: propagation delay (slide 45)

Carrier sensing doesn't prevent every collision, because a signal takes **time to travel** along the cable. Let **Prop** be the propagation time along the whole channel:

```
time 0     A starts sending.                         A ▶───────────────── B
just < Prop  A's signal hasn't reached B yet, so B
           senses an idle channel and starts sending. A ▶──────────▶◀──── B
≈ Prop     B detects the collision and sends a noise burst.        💥
≈ 2 Prop   The noise burst gets back to A.          A ◀══════════════════ B
```

**Worst-case collision detection time ≈ 2 × Prop.**

### Minimum frame size (slide 46)

In Ethernet, a node must know **while it is still transmitting** whether its frame collided. If it had already finished, it wouldn't know which frame had been hit.

So **every frame must take at least 2 × Prop to transmit**. If the data is too short, the information field is **padded**.

```
Transmission time ≥ 2 × Prop    →    L_min ÷ R ≥ 2 × Prop    →    L_min = 2 × Prop × R
```

**Bonus:** if a whole frame is sent without detecting a collision, it was transmitted correctly and can be assumed received. So **no ACKs are needed** on wired Ethernet.

**Worked example:** a 1 km cable, 2 × 10⁸ m/s, 10 Mb/s.
- Prop = 1,000 ÷ 2 × 10⁸ = 5 µs
- L_min = 2 × 5 µs × 10 × 10⁶ = **100 bits**

**Slide 46 question:** what happens to efficiency if you extend Ethernet's reach (distance)?
> A longer cable means a bigger Prop, so the **minimum frame gets longer**. Short messages need **more padding**, so **efficiency falls**. The same happens if you raise the bit rate without shortening the cable, since L_min = 2 × Prop × R.

### The Ethernet frame (slide 49)

```
  7 B       1 B      6 B            6 B        2 B       46–1500 B          4 B
┌─────────┬─────┬─────────────┬─────────────┬────────┬──────────────────┬─────┐
│Preamble │ SFD │ Destination │   Source    │ Length │  Data (802.2 PDU) │ CRC │
│         │     │   address   │   address   │  PDU   │ SSAP DSAP Ctrl Info│     │
└─────────┴─────┴─────────────┴─────────────┴────────┴──────────────────┴─────┘
                 └──── physical (MAC) addresses ─────┘
```

| Field | Size | Purpose |
|---|---|---|
| **Preamble** | 7 B | 56 bits of alternating 1s and 0s, so the receiver can **synchronise** with the incoming signal |
| **SFD** (Start of Frame Delimiter) | 1 B | Ethernet's **start flag**: `10101011` |
| **Destination / source address** | 6 B each | **Physical (MAC) addresses**, encoded on the node's **NIC**. Every NIC ever made has a **unique** 6-byte address. |
| **Length** | 2 B | Length of the PDU |
| **Data** | **46 to 1,500 B** | The 802.2 LLC-PDU. A minimum of 46 B, so padding is used if needed. |
| **CRC** | 4 B | Error detection |

**Note:** there is **no ACK/NAK** in Ethernet. Any reliability must come from a **higher layer** (e.g. TCP). Ethernet is an **unreliable** medium.

**Frame sizes** *(worked out from the field sizes on the slide)*:
- Smallest frame = 6 + 6 + 2 + 46 + 4 = **64 B** (72 B including the preamble and SFD)
- Largest frame = 6 + 6 + 2 + 1,500 + 4 = **1,518 B**

### Advantages and disadvantages (slide 50)

| ✅ Advantages | ❌ Disadvantages |
|---|---|
| Low average access time when **lightly loaded** | A substantial **analogue** part: the collision detection circuitry |
| Easy to move, upgrade or change nodes (access control is **distributed**) | The minimum frame length is **inefficient for short data** (e.g. 1 character) |
| Performance degrades **gracefully** as nodes are added | **No priorities**, so real-time traffic is hard to support |
| The **most popular** LAN technology, with a huge installed base | Access time is **unpredictable**, and as load rises it grows **without limit** (theoretically unbounded) |

These problems led to alternatives:
- **IBM** chose a ring topology for office automation → **Token Ring (802.5)**
- **General Motors** and others in factory automation chose a bus, matching assembly-line layouts → **Token Bus (802.4)**

**The key point:** both use **reservation**, not contention, so the **MMAT is bounded**, as long as the network is working correctly.

---

## 5. Token Ring (IEEE 802.5) (slides 51–52)

- The **ring** is a set of **point-to-point links forming a circle**. It is logically **one-way**.
- The **token** is a special frame passed from node to node. **Only the node holding the token may transmit.**
- When a node gets the token, it **removes** it from the ring, **transmits one frame**, then puts the token **back** (sends it on to the next node).
  - "Removing the token" means changing it into the **start of a data frame**.
- Each node has a **1-bit buffer**: as each bit passes through, it can be inspected and possibly changed before being sent on. A node is either in **listen** mode or **transmit** mode.

### Token passing step by step (slide 52)

| Step | What happens |
|---|---|
| 1 | The **token** travels round the ring. |
| 2 | Station **A captures the token** and sends its data frame, addressed to D. |
| 3 | Station **D copies the frame** and sends it on round the ring. The frame now carries a mark saying the intended receiver copied it. |
| 4 | The frame gets back to **A**. A removes it and **releases the token** for the next station. |

When the frame returns to A, the **"copied" mark tells A that D received it**. A built-in acknowledgement.

### Ethernet vs Token Ring

| | Ethernet (802.3) | Token Ring (802.5) |
|---|---|---|
| Method | **Contention** (collision resolution) | **Reservation** (token) |
| Topology | Bus | Ring |
| Collisions | Possible | **None**, because only one node can send |
| Max access time (MMAT) | **Unbounded** | **Bounded** |
| Light load | Fast: send straight away | Slower: wait for the token |
| Heavy load | Gets worse and worse | Fair: everyone gets a turn |

---

## Quiz

**Q1.** What are the three HDLC frame types, and what does each carry?

<details><summary>Show answer</summary>

- **I-frame:** user data plus control info
- **S-frame:** control info only (e.g. ACKs)
- **U-frame:** link management
</details>

**Q2.** HDLC uses 3-bit sequence numbers. What ARQ scheme and window size does it use? What about extended mode?

<details><summary>Show answer</summary>

**Go-back-N** with window **7** (Go-back-7). Extended mode uses 7-bit numbers, so window **127**.
</details>

**Q3.** Which three things does an LLC-PDU **not** have, and who supplies them?

<details><summary>Show answer</summary>

**Flags, a CRC and physical (machine) addresses.** The **MAC sublayer** supplies them.
</details>

**Q4.** Name the three ways to share a channel, with an example of each.

<details><summary>Show answer</summary>

1. **Divide the channel** (TDM or FDM)
2. **Collision resolution** (Ethernet, CSMA/CD)
3. **Reservations** (Token Ring)
</details>

**Q5.** Why does each node wait a **random** time after a collision?

<details><summary>Show answer</summary>

If they all waited the **same** time, they'd retry together and **collide again**. Random waits make it likely that one node goes first.
</details>

**Q6.** Why can collisions happen even when every node listens before sending?

<details><summary>Show answer</summary>

**Propagation delay.** A node's signal takes time to reach the others, so another node can sense the channel as idle and start sending before the first signal arrives.
</details>

**Q7.** A cable is 2 km long with a propagation speed of 2 × 10⁸ m/s, and the bit rate is 100 Mb/s. What is the minimum frame size for CSMA/CD, in bytes?

<details><summary>Show answer</summary>

- Prop = 2,000 ÷ 2 × 10⁸ = 10 µs, so 2 × Prop = 20 µs
- L_min = 20 × 10⁻⁶ × 100 × 10⁶ = 2,000 bits = **250 B**
</details>

**Q8.** Why doesn't wired Ethernet use ACKs?

<details><summary>Show answer</summary>

Because every frame lasts at least 2 × Prop, a node always finds out about a collision **while it is still transmitting**. If it finishes with no collision, the frame was sent correctly and is assumed received. Any further reliability is left to a **higher layer**, so Ethernet is an **unreliable** medium.
</details>

**Q9.** A node wants to send 1 byte of data over Ethernet. How big is the frame (without the preamble and SFD), and why is this a disadvantage?

<details><summary>Show answer</summary>

The data field must be at least **46 B**, so it's **padded**: 6 + 6 + 2 + 46 + 4 = **64 B** to carry 1 byte. Most of the frame is overhead, so the **minimum frame length is inefficient for short data**.
</details>

**Q10.** In Token Ring, how does the sender know the receiver got its frame?

<details><summary>Show answer</summary>

The receiver **copies** the frame and marks it as copied, then sends it on round the ring. When it gets **back to the sender**, the mark shows it was received. The sender then removes the frame and releases the token.
</details>

**Q11.** Why might a factory prefer Token Bus or Token Ring over Ethernet?

<details><summary>Show answer</summary>

Their **maximum medium access time is bounded**, so every node is guaranteed a turn within a known time. Ethernet's access time is unpredictable and theoretically unbounded under heavy load.
</details>

---

## What to remember

- **HDLC:** I/S/U frames · flag `01111110` · address, control, data, 2 or 4 B CRC · **Go-back-7** (3-bit numbers) or **Go-back-127** (7-bit).
- **IEEE 802** splits the data link layer into **LLC** (common, 802.2, logical addresses DSAP/SSAP) and **MAC** (per technology, physical addresses, contention).
- **3 ways to share:** divide (TDM/FDM) · collision resolution (Ethernet) · reservation (token).
- **CSMA/CD:** listen before sending, listen while sending, stop and wait a **random** time on a collision. MMAT unbounded.
- **Worst-case collision detection = 2 × Prop.** Minimum frame: **L_min = 2 × Prop × R**.
- **Ethernet frame:** 7 B preamble · 1 B SFD · 6 B + 6 B MAC addresses · 2 B length · **46–1,500 B data** · 4 B CRC. **No ACKs** (unreliable).
- **Token Ring:** only the token holder sends, one frame at a time, and the "copied" mark acts as the ACK. MMAT **bounded**.
