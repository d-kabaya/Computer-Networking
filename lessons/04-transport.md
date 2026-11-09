# Lesson 4: Transport Layer: UDP and TCP

[← Layers](03-layers.md) · [Index](../README.md) · [Next: TCP congestion control →](05-tcp-congestion.md)

## The big idea

IP gets a packet to the right **computer**. But a computer runs many programs at once: a browser, Spotify, a game. The transport layer gets the data to the right **program**, using a **port number**.

> **Analogy:** the IP address is a block of flats, and the port is the flat number.

## 1. Ports

- Each application process on a host gets its own **port number**.
- Common services use **well-known (reserved) ports**, for example HTTP on 80 and HTTPS on 443.
- User applications choose from the **non-reserved** ports.
- The lecture slides say a process is addressed by a **⟨port, host⟩** pair.

⚠️ **Exam note:** the course answer key marks the statement *"In UDP an IP Host and Port uniquely defines a host's application process"* as **false**. Leave it unticked.

## 2. UDP: User Datagram Protocol

- **Connectionless:** there's **no connection setup**. It just sends.
- **No** reliability, ordering or flow control, so it **can deliver packets out of order** (Test 2: true).
- It has an optional **checksum** for error detection.
- It's fast and simple, so it suits video calls, gaming and DNS lookups.

## 3. TCP: Transmission Control Protocol

- **Connection-oriented:** a connection is set up first.
- **Reliable**, **in-order** delivery.
- A **byte stream**: it does **not** preserve message (packet) boundaries.
- Bytes flow in **both directions** over one connection.

✅ IP is connectionless and unreliable, but **TCP still gives a reliable, connection-oriented service on top of IP**.

## 4. UDP vs TCP

| | UDP | TCP |
|---|---|---|
| Connection | None | Set up first |
| Reliable | No | Yes |
| Order kept | No | Yes |
| Message boundaries | Kept | **Not** kept (byte stream) |
| Speed | Faster | Slower (extra work) |
| Used for | Video, games, DNS | Web, email, file transfer |

## 5. TCP window parameters

| Term | Meaning | Changes? |
|---|---|---|
| **MSS** (Maximum Segment Size) | The largest segment a device can send or receive, *before* the TCP and IP headers are added | Set once, **not changed** |
| **Congestion window** | How much the sender may send right now | Changes **a lot** |
| **Receiver's advertised window** | The most the receiver can accept, a hard ceiling | Can change **at any time** |
| **Threshold** | Where growth switches from doubling to +1 MSS | Changes on a timeout |

Key facts:
- The congestion window **cannot** exceed the receiver's advertised window.
- The threshold **can** be greater than the advertised window.
- The congestion window **can** be greater than the threshold (during the linear phase).

---

## Quiz

**Q1.** True or false: UDP sets up a connection for data transfer at the transport layer.

<details><summary>Show answer</summary>

**False.** UDP is connectionless.
</details>

**Q2.** True or false: TCP preserves message (packet) boundaries.

<details><summary>Show answer</summary>

**False.** TCP is a byte stream.
</details>

**Q3.** True or false: the MSS in TCP usually changes a lot during the connection.

<details><summary>Show answer</summary>

**False.** The MSS is set at the start and not changed. It's the **congestion window** that changes a lot.
</details>

**Q4.** True or false: the receiver's advertised window can be changed at any time in TCP.

<details><summary>Show answer</summary>

**True.**
</details>

**Q5.** True or false: even though IP is connectionless and unreliable, it is still possible to have a reliable, connection-oriented service that uses IP.

<details><summary>Show answer</summary>

**True.** That's exactly what TCP does.
</details>

**Q6.** True or false: the congestion window can be greater than the receiver's advertised window.

<details><summary>Show answer</summary>

**False.** The advertised window is the ceiling.
</details>
