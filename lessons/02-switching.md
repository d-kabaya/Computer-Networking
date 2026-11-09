# Lesson 2: Switching and Multiplexing

[← Delays](01-delays.md) · [Index](../README.md) · [Next: Layers →](03-layers.md)

## The big idea

A network has to get data from A to B through many nodes in between. There are two main ways to share the links between users:

- **Circuit switching:** reserve a path first, then send. This is like booking a private road.
- **Packet switching:** chop the data into packets and send them with no reservation. This is like posting letters.

## 1. Circuit switching

1. **Setup:** reserve a path (a circuit) through the network.
2. **Transfer:** the data flows straight through. The link is **held for the whole connection**.
3. **Release:** free the circuit.

```
Circuit delay = setup + message size ÷ rate + hops × propagation (+ release)
```

**Lecture example:** a 4.6 MB file at 250 kb/s, with 0.85 s setup and 0.05 s release:
4.6 × 8,000,000 ÷ 250,000 = 147.2 s → total = 0.85 + 147.2 + 0.05 = **148.1 s**

👍 Good for smooth, constant-rate traffic (e.g. voice). Performance is predictable.
👎 Wastes capacity for bursty traffic, and setup takes time.

## 2. Packet switching

The data is split into packets, which share the links on demand. There are two types:

| | Datagram | Virtual circuit |
|---|---|---|
| Setup needed? | No | Yes (and release afterwards) |
| Path | Each packet routed separately | All packets follow the same path |
| Order | May arrive **out of order** | Arrive **in order** |
| Links shared? | Yes | Yes |
| Same as circuit switching? | No | **No** (links are still shared) |

✅ IP uses **datagram** packet switching.
✅ Datagram switching suits **bursty** traffic better than circuit switching.
❌ Neither type is "always better".

## 3. Store-and-forward and hops

A **hop** is one link. **Hops = intermediate nodes + 1.**

```
Src ──hop 1──> R1 ──hop 2──> R2 ──hop 3──> Dst      (2 nodes → 3 hops)
```

**Store-and-forward:** each node must receive the **whole packet** before sending it on. So the transmission delay is paid **on every hop**.

### One packet

```
Delay = hops × (transmission + propagation)
```

**Example:** 6 intermediate nodes, 1,500 B, 150 Mb/s, propagation ignored
- Transmission = 12,000 ÷ 150,000,000 = 80 µs
- Hops = 7, so the delay = 7 × 80 = **560 µs**

If the links run at **different rates**, add up each link's transmission time:
200 B over 10 Mb/s then 20 Mb/s = 160 µs + 80 µs = **240 µs**.

### Many packets: pipelining

While R1 forwards packet 1, the source is already sending packet 2. All the links work at the same time, like an assembly line:

```
time slot →      1    2    3    4    5
hop 1 (Src→R1):  P1   P2   P3
hop 2 (R1→R2):        P1   P2   P3
hop 3 (R2→Dst):            P1   P2   P3
```

```
Delay = (N + k) × T_packet + hops × T_prop
        N = packets, k = intermediate nodes
```

Why it works: the **last** packet waits for all N packets to leave the source, then is re-sent by each of the k nodes, then propagates along every hop.

## 4. Two-link questions (A ─L1─ B ─L2─ C)

A sends P1, then P2 straight after. **There's no queueing at B if P2 finishes arriving no earlier than B finishes sending P1:**

```
P2 ÷ R1  ≥  P1 ÷ R2          (propagation cancels out)
Min P2 = P1 × R1 ÷ R2       Max P1 = P2 × R2 ÷ R1
```

**Example:** P1 = 1,200 B, R1 = 200 Mb/s, R2 = 500 Mb/s → min P2 = 1,200 × 200 ÷ 500 = **480 B**

⚠️ Check whether the answer should be in **bits** or **bytes**.

**"When does C receive the last bit of P2?"** P2 starts on L2 at the **later** of (P2 fully at B) and (B finished with P1). Then add P2's transmission on L2 and the propagation on L2.

## 5. Circuit vs packet comparison

**Example:** a 0.2 MB message, 7 nodes, 916 ms setup, 10 ms per hop, 8,750 B packets, 10 Mb/s

| | Packet switching | Circuit switching |
|---|---|---|
| Hops | 8 | 8 |
| Sending | 23 packets × 7 ms + 7 nodes × 7 ms = 210 ms | 0.2 MB ÷ 10 Mb/s = 160 ms |
| Propagation | 8 × 10 = 80 ms | 80 ms |
| Setup | none | 916 ms |
| **Total** | **290 ms** | **1,156 ms** |

So circuit switching is about **4×** slower here.

**When does packet switching win (one packet, no propagation)?** When **setup > k × T_packet**.
Example: 5 nodes, 1,500 B, 10 Mb/s → T = 1.2 ms, so setup > **6 ms**.

This holds for **any message size**: the message time and the propagation are the same in both, so they cancel. Only k × T_packet vs setup matters.
**Largest packet** for packet switching to be the same or faster = setup × R ÷ (8k) bytes.
Example (Test 2): 7 nodes, 84 Mb/s, 30 ms setup → 0.03 × 84,000,000 ÷ 56 = **45,000 B = 45 KB**.

## 6. Statistical multiplexing

Traffic is **bursty**, so flows rarely all peak at the same time. Instead of reserving for the worst case, packet switching **shares** the link.

| Term | Meaning |
|---|---|
| **Aggregate of peaks** | Add up each flow's peak: the worst case |
| **Peak of aggregate** | The highest value of the combined flow |

✅ Peak of aggregate ≤ aggregate of peaks (usually **much** smaller)
✅ Many bursty flows together → the peak of the aggregate gets close to the **average**
❌ The peak of the aggregate can't be smaller than the sum of the averages

**Quiz formulas (given averages and peaks):**
- Worst case = Σ peaks
- Best case = max(Σ averages, largest single peak)

## 7. Queueing

A packet waits at a node if the node is still busy sending an earlier packet.

- A **smaller packet after a larger one** (same input and output rate) **will** queue.
- A **bigger packet after a smaller one:** "there can be no queueing" is marked **false**.
- Queueing **can** happen even with only one input and one output.

---

## Quiz

**Q1.** 3 intermediate nodes, 1,500 B packets, 12 Mb/s links, 2 ms propagation per link. What is the delay for one packet?

<details><summary>Show answer</summary>

T = 12,000 ÷ 12,000,000 = 1 ms. Hops = 4. Delay = 4 × (1 + 2) = **12 ms**.
</details>

**Q2.** The same links, but now **10 packets**. What is the total delay?

<details><summary>Show answer</summary>

(N + k) × T + hops × P = (10 + 3) × 1 + 4 × 2 = **21 ms**.
</details>

**Q3.** 6 intermediate nodes, 150 Mb/s, propagation negligible. The total delay must not exceed 350 µs. What is the maximum packet size in bytes?

<details><summary>Show answer</summary>

7 hops, so each hop gets 350 ÷ 7 = 50 µs. Size = 50 µs × 150 Mb/s = 7,500 bits = 937.5 → **937 B**.
</details>

**Q4.** Three flows: averages 2, 1 and 4 Mb/s, peaks 9, 12 and 6 Mb/s. What is the required bit rate (a) in the worst case and (b) in the best case?

<details><summary>Show answer</summary>

(a) Σ peaks = 9 + 12 + 6 = **27 Mb/s**. (b) max(Σ averages = 7, largest peak = 12) = **12 Mb/s**.
</details>

**Q5.** True or false: virtual circuit packet switching is the same as circuit switching.

<details><summary>Show answer</summary>

**False.** In a virtual circuit the links are still **shared**. It only fixes the path, and it still needs setup and release.
</details>

**Q6.** A ─L1─ B ─L2─ C. L1 = 160 Mb/s and L2 = 100 Mb/s. P2 is 1,200 B. What is the **biggest** P1 so that there's no queueing at B?

<details><summary>Show answer</summary>

Max P1 = P2 × R2 ÷ R1 = 1,200 × 100 ÷ 160 = **750 B**.
</details>

**Q7.** True or false: in datagram packet switching, packets always arrive in order.

<details><summary>Show answer</summary>

**False.** Each datagram is routed separately, so they can arrive out of order.
</details>
