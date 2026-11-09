# Lesson 8: Exam Revision

[← Subnetting](07-subnetting.md) · [Index](../README.md) · [Next: Physical layer and bit errors →](09-physical-layer-bit-errors.md)

A printable two-page version of this is in [`resources/Networking_Cheat_Sheet.pdf`](../resources/Networking_Cheat_Sheet.pdf).

## All the formulas

### Delays
| What | Formula |
|---|---|
| Transmission | size (bits) ÷ bandwidth |
| Propagation | length ÷ speed |
| Packet delay | transmission + propagation (+ queueing) |
| RTT | 2 × (transmission + propagation) |
| Bandwidth-delay product | bandwidth × propagation |
| Packets on the link | ⌊propagation ÷ transmission⌋ |
| Packet error probability | ≈ bits × bit-error probability |

### Paths and switching
| What | Formula |
|---|---|
| Hops | intermediate nodes + 1 |
| One packet, store-and-forward | hops × (T_trans + T_prop) |
| N packets, pipelined | (N + k) × T_packet + hops × T_prop |
| Circuit switching | setup + message ÷ rate + hops × T_prop |
| No queueing at B | P2 ÷ R1 ≥ P1 ÷ R2 |
| Worst / best multiplexing | Σ peaks / max(Σ averages, largest peak) |

### TCP
| What | Rule |
|---|---|
| Below the threshold | double, capped at the threshold |
| At or above the threshold | +1 MSS |
| Ceiling | the receiver's advertised window |
| Timeout | threshold = ⌊½ × window⌋, window = 1 MSS |
| Throughput | window ÷ RTT |

### Subnetting
| What | Formula |
|---|---|
| Usable hosts | 2^(32−n) − 2 |
| Subnets | 2^(n − class bits) |
| Block | 256 − mask octet |
| Subnet | copy · start · 0s |
| Broadcast | copy · next start − 1 · 255s |

### Data link errors
| What | Formula |
|---|---|
| Frame error | p = 1 − (1 − p_b)^L ≈ L × p_b (only if p ≪ 1) |
| Over k hops | P(good) = (1 − p_b)^(L × k) |
| Largest frame | L < ln(1 − p) ÷ ln(1 − p_b) |
| Detect / correct | ≤ D − 1 errors / < D/2 errors |
| CRC | G has n + 1 bits → n-bit CRC · append n zeros, XOR-divide, remainder = CRC |
| Bit stuffing | a 0 after every five 1s · flag 01111110 at both ends |

## Top 10 exam traps

1. **Bytes vs bits:** multiply by 8.
2. **Nodes vs hops:** hops = nodes + 1.
3. **TCP threshold cap:** doubling stops **at** the threshold (12 → 22, not 24, if the threshold is 22).
4. **Timeout halving** uses the **current window**, not the old threshold or the advertised window.
5. **"After the n-th transmission"** means the value **after** row n.
6. **Time to first full window:** count the bursts **before** the full one.
7. **The cut in the 3rd octet:** everything after it becomes **0** (subnet) or **255** (broadcast).
8. **.0 and .255 can be valid hosts** when the host bits span more than one octet.
9. **Class A = 1–126:** 127 is loopback.
10. **OSI = 7 layers, TCP/IP = 4.**

---

## Final mixed quiz

**Q1.** A ─L1─ B ─L2─ C, with L1 = 200 Mb/s and L2 = 500 Mb/s. P1 is 1,200 B. What is the smallest P2 (in bytes) so that there's no queueing at B?

<details><summary>Show answer</summary>

P2 = 1,200 × 200 ÷ 500 = **480 B**.
</details>

**Q2.** A 0.5 MB message crosses 4 intermediate nodes. Setup = 500 ms, propagation = 5 ms per hop, packets = 10,000 B, rate = 8 Mb/s. What is the circuit ÷ packet delay ratio?

<details><summary>Show answer</summary>

Circuit = 500 + 500 + 25 = 1,025 ms. Packet = (50 + 4) × 10 + 25 = 565 ms. Ratio ≈ **1.81**.
</details>

**Q3.** MSS 3 KB, threshold 30 KB, receiver's window 45 KB, timeout on the 9th transmission. What is the window after the 14th?

<details><summary>Show answer</summary>

3, 6, 12, 24, 30, 33, 36, 39, then the 9th sends 42 and times out, so threshold = 21 and window = 3. Then 3 → 6, 6 → 12, 12 → 21, 21 → 24, 24 → **27 KB**.
</details>

**Q4.** 145.32.180.5 / 255.255.224.0: what are the subnet and broadcast addresses?

<details><summary>Show answer</summary>

Block 32 in the 3rd octet: 180 is in 160–191. Subnet = **145.32.160.0**, broadcast = **145.32.191.255**.
</details>

**Q5.** 201.15.7.99 / 255.255.255.248: what is the class, the usable hosts per subnet, the number of subnets and the broadcast?

<details><summary>Show answer</summary>

**Class C**. /29 leaves 3 host bits, so 2³ − 2 = **6** hosts. Subnet bits = 29 − 24 = 5, so **32** subnets. Block 8: 99 is in 96–103, so the broadcast is **201.15.7.103**.
</details>

**Q6.** Three flows have averages of 6, 4 and 4 Mb/s and peaks of 9, 12 and 6 Mb/s. What is the required bit rate in the best case?

<details><summary>Show answer</summary>

max(Σ averages = 14, largest peak = 12) = **14 Mb/s**.
</details>

**Q7.** True or false: a domain name can be resolved to many MAC addresses.

<details><summary>Show answer</summary>

**True.** DNS gives several IPs, and ARP gives a MAC for each.
</details>

**Q8.** True or false: subnetting and CIDR are really the same.

<details><summary>Show answer</summary>

**False.** CIDR allocates blocks on any bit boundary, while subnetting splits one network into smaller ones.
</details>

**Q9.** A TCP machine sends full windows of 131,072 bytes over a 100 Mb/s channel with a 10 ms one-way delay. What is the maximum throughput?

<details><summary>Show answer</summary>

131,072 × 8 ÷ 0.020 s ≈ **52 Mb/s**.
</details>

**Q10.** True or false: a smaller packet that follows a larger packet through an intermediate node, where the input and output bit rates are the same, will experience queueing delay.

<details><summary>Show answer</summary>

**True.** The small packet arrives before the node has finished sending the big one.
</details>
