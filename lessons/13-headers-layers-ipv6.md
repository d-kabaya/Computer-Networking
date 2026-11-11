# Lesson 13: Headers, Layers Revisited and IPv6

[← Naming, DNS and ARP](12-naming-dns-arp.md) · [Index](../README.md)

*Based on the "Internet" lecture deck, slides 20, 27, 30 and 65–78.*

## 1. Why IP is "best effort" (slide 20)

The Internet was designed for **robust computer-to-computer data transfer over long distances**. Best effort suits that: there were no real-time requirements, and the end points can adapt to network conditions if they need to.

**Why connectionless packet switching was chosen over circuit switching:**

| Advantage | Meaning |
|---|---|
| **No set-up delay** | Just start sending |
| **No blocking** | "Fire and forget, forward if possible, deliver as received". ⚠️ But there's **no guarantee** any data arrives |
| **Flexible bit rates** | Circuit switching is tied to a few fixed rates |
| **No fixed "path"** | **More reliable**, because traffic can route around problems |
| **Efficient for bursty traffic** | Statistical multiplexing (Lesson 2) |

## 2. The TCP header (slide 27)

- It's **20 to 60 bytes**, so it varies in size.
- The **sequence number (SN)** in a segment is the number of its **first byte**. Every byte gets its own SN.
- The **acknowledgement** field gives the **SN of the next byte expected** at the receiver.
- The **checksum** covers the TCP header, the TCP data and the IP **pseudoheader** (the source and destination address fields from the IP header).

## 3. TCP window management example (slide 30)

The receiver has a **4K buffer** and uses the window field (WIN) to tell the sender how much room it has left.

| Step | What happens | Receiver buffer |
|---|---|---|
| 1 | The sender sends 2K (SEQ=0) | 2K used |
| 2 | The receiver replies ACK=2048, **WIN=2048** ("room for 2K more") | |
| 3 | The sender sends 2K (SEQ=2048) | **Full** |
| 4 | The receiver replies ACK=4096, **WIN=0**, so the **sender is blocked** | |
| 5 | The application reads 2K, and the receiver sends ACK=4096, WIN=2048 | 2K free |
| 6 | The sender may now send up to 2K. It sends 1K (SEQ=4096) | |

**Rules from the slide:**
- While WIN = 0, the sender may only send **urgent data**, or a **1-byte "probe"** segment after a **persistence timer** expires, to check whether WIN has changed.
- The receiver **shouldn't send a window update** until it has room for a maximum-size segment, or its buffer is half empty (whichever is smaller).
- Otherwise you get the **"silly window" problem**: window updates sent for tiny window sizes, with the sender responding with tiny segments. That's very inefficient.

## 4. Layers again (slides 65–72)

### What the network implements (slide 65)

| Layer | Job | Where |
|---|---|---|
| **L1 Physical** | Bits arrive on the wire | Everywhere |
| **L2 Data link** | Packets delivered across **links and local networks** | Everywhere |
| **L3 Network** | Packets delivered **between networks** (global delivery) | Everywhere |
| **L4 Transport and above** | Reliable delivery | **Only at the end hosts** |

- The network itself does **not** support reliable delivery. That job belongs to the end hosts (TCP).
- Switches used to implement L1–L2 and routers L1–L3. Now most switches also do L3, so the terms **switch and router are used interchangeably**.

### Protocols (slide 66)

A **protocol** is an **agreement between parties on how to communicate**. It defines the **syntax** (the format) and the **semantics** (the meaning). Protocols are defined by standards bodies such as **IETF, IEEE and ITU**. Because the layers are separate, **innovation can happen in parallel**.

### Logical communication (slide 67)

- Each layer **logically** talks to its **peer**, the same layer on the other device.
- The **lower three layers** are implemented **everywhere**, including in routers.
- The **top two layers** are implemented **only at the hosts**.

### Local and global (slide 68)

- **Local** delivery is **within** an **Autonomous System (AS)**, for example Ethernet.
- **Global** delivery is **between** ASes, for example optical links (or radio).

### Who thinks about what (slide 69)

- The **application** thinks about **data**.
- The **NIC and its driver** think about **packets**.
- The **network stack** sits in between and translates between the two.

**The letter example (slide 70):** Bob (UCD) writes to Alice (UCC). The office worker bags it, the courier delivers it, and the process is reversed at the other end. The information flows **down** to the physical layer, across, and back **up**.

### Encapsulation (slide 72)

A packet carries **multiple headers**, one from each layer above. Or it might carry only **parts** of them, spread over several packets.

## 5. The IPv4 header: fragmentation and TTL (slide 73)

| Field | What it's for |
|---|---|
| **Identification** | All **fragments** of the same datagram carry the **same** ID value |
| **Fragmentation** | Needed when an intermediate network's **maximum frame size** is too small to carry the datagram |
| **TTL** (Time To Live) | **Decreased by 1 at each hop** |
| **Options** | **Padded to a multiple of 4 bytes** (32 bits). They can control routing, timing, management and security |

## 6. IPv6 (slides 74–77)

**Why it exists:** IPv4 addresses are running out, even with subnetting and CIDR.

| | IPv4 | IPv6 |
|---|---|---|
| Address size | **32 bits** (≈ 4.3 billion addresses) | **128 bits** (2¹²⁸ ≈ 3.4 × 10³⁸) |
| Header checksum | Yes | **None** |
| Fragmentation | By routers along the way | **At the source only** |
| Addressing | Classful (old), then CIDR | **Classless**, divided by prefixes |

**Key facts:**
- It was designed by the **IETF** in the early **1990s**.
- It was motivated by IPv4's **limited address space**, **no handling of real-time "flows"** (routers ignored the Type Of Service field), and **little use of security features**.
- IPv6 is **not compatible with IPv4**, but it **is** compatible with TCP, UDP, OSPF, BGP and DNS.
- IPv4 and IPv6 **co-exist**, and probably will for a long time.
- There's so much address space that even pessimistic estimates give **16,000 addresses per square metre** of the Earth's surface. The **Internet of Things (IoT)** will use a lot of them.

**Major features:**
- **128-bit addresses**
- A **simplified header**: fragmentation at the source only, and no header checksum
- Better support for **options**, using **extension headers** that are only present when needed
- **Per-flow handling** and traffic classes (a flow = source + destination + flow label)
- Support for **authentication and security**

**IPv6 header fields:** version · traffic class · flow label · payload length · next header · hop limit · source address (16 bytes) · destination address (16 bytes) · then any extension headers, each with its own next header field.

---

## Quiz

**Q1.** Give three reasons connectionless packet switching was preferred for the Internet.

<details><summary>Show answer</summary>

Any three of: no set-up delay · no blocking · flexible bit rates · no fixed path (it can route around problems) · more efficient for bursty traffic.
</details>

**Q2.** What does the acknowledgement number in a TCP header mean?

<details><summary>Show answer</summary>

The **sequence number of the next byte the receiver expects**.
</details>

**Q3.** The receiver advertises WIN = 0. What can the sender do?

<details><summary>Show answer</summary>

It's **blocked**. It can only send **urgent data**, or a **1-byte probe** after the persistence timer expires, to check whether the window has opened.
</details>

**Q4.** Which layers are implemented in routers, and which only at hosts?

<details><summary>Show answer</summary>

The **lower three** (physical, data link, network) are implemented everywhere. The **top two** (transport and application, in the slide's model) are only at the hosts.
</details>

**Q5.** Why does an IPv4 datagram need fragmentation, and what links the fragments together?

<details><summary>Show answer</summary>

It's needed when an intermediate network's **maximum frame size** is too small for the datagram. All fragments share the **same Identification value**.
</details>

**Q6.** What happens to the TTL field at each hop?

<details><summary>Show answer</summary>

It's **decreased by 1**.
</details>

**Q7.** Give three differences between IPv4 and IPv6.

<details><summary>Show answer</summary>

IPv6 has **128-bit** addresses (IPv4 has 32) · **no header checksum** · fragmentation at the **source only** · better options through **extension headers** · per-flow handling.
</details>

**Q8.** True or false: IPv6 is compatible with IPv4.

<details><summary>Show answer</summary>

**False.** IPv6 isn't compatible with IPv4, but it is compatible with TCP, UDP, OSPF, BGP and DNS. The two versions co-exist.
</details>
