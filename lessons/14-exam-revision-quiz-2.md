# Lesson 14: Exam Revision Quiz 2

[← Headers, layers and IPv6](13-headers-layers-ipv6.md) · [Index](../README.md)

A mixed exam-style quiz covering **all** the lecture decks. It's built from the course tests (October 2024, October 2026, Test 2) and the quiz solution sets, plus some new variants so you can't just remember the answers.

> Try every question first, then click **Show answer**. Write your working out. That's where most of the marks are.

---

## Part A: Data link errors

**A1.** The BER is 1 in ten thousand and the frame size is 1,000 B. What is the frame error probability p, to two decimal places?

<details><summary>Show answer</summary>

L = 8,000 bits. The approximation gives 0.8, which is too big, so use the exact formula: p = 1 − (1 − 10⁻⁴)^8,000 = **0.55**.
</details>

**A2.** A frame must have an error probability of at most 1 in a thousand, and the BER is 1 in a million. What is the maximum frame size in bytes?

<details><summary>Show answer</summary>

L ≤ 10⁻³ ÷ 10⁻⁶ = 1,000 bits = **125 B**.
</details>

**A3.** A system needs the frame error probability to be no more than 5%. The BER is 40 in a million. What is the maximum frame size in bytes?

<details><summary>Show answer</summary>

5% isn't ≪ 1, so use the exact formula: L ≤ ln(0.95) ÷ ln(1 − 40 × 10⁻⁶) = 1,282 bits → ÷ 8 = **160 B** (the course answer).
⚠️ The approximation gives 0.05 ÷ (4 × 10⁻⁵) = 1,250 bits = 156 B, which would be marked wrong.
</details>

**A4.** A frame must have an error probability of at most 1 in a thousand, and the BER is 1 in twenty thousand. What is the maximum frame size in bytes?

<details><summary>Show answer</summary>

L ≤ 10⁻³ ÷ (5 × 10⁻⁵) = 20 bits = 2.5 → **2 B** (round down).
</details>

**A5.** The BER is 1 in a hundred thousand and the frame size is 1,500 B. What is the frame error probability in percent (rounded)?

<details><summary>Show answer</summary>

L = 12,000 bits. p = 1 − (1 − 10⁻⁵)^12,000 = 0.113 → **11%**. (The approximation would give 12%.)
</details>

**A6.** *(New)* A frame must have an error probability of at most 1 in a hundred, and the BER is 1 in fifty thousand. What is the maximum frame size in bytes?

<details><summary>Show answer</summary>

L ≤ ln(0.99) ÷ ln(1 − 2 × 10⁻⁵) = 502.5 bits → ÷ 8 = 62.8 → **62 B**.
</details>

**A7.** 16 bits of redundancy are used on a data frame of 5 B. How many frames have exactly the same 16-bit redundancy sequence?

<details><summary>Show answer</summary>

5 B = 40 data bits, so there are 2⁴⁰ possible frames but only 2¹⁶ possible check sequences. Frames per sequence = 2⁴⁰ ÷ 2¹⁶ = 2²⁴ = **16,777,216**.
This is why many frames share the same redundancy, so some errors can't be detected.
</details>

**A8.** *(New)* 8 bits of redundancy are used on a 3 B data frame. How many frames share each redundancy sequence?

<details><summary>Show answer</summary>

2²⁴ ÷ 2⁸ = 2¹⁶ = **65,536**.
</details>

**A9.** *(New)* Generate the CRC for data `110101` with generator `1011`.

<details><summary>Show answer</summary>

G has 4 bits, so the CRC is 3 bits. Divide `110101 000` by 1011 using XOR. The remainder is **111**. Transmitted: `110101 111`. (Check: dividing this by G gives remainder 000 ✓.)
</details>

**A10.** *(New)* Generate the Hamming codeword for data `0110011`.

<details><summary>Show answer</summary>

Positions 11, 10, 9, 7, 6, 5, 3 = 0, 1, 1, 0, 0, 1, 1.
r1 (3, 5, 7, 9, 11) = 1, 1, 0, 1, 0 → three 1s → **1**
r2 (3, 6, 7, 10, 11) = 1, 0, 0, 1, 0 → two 1s → **0**
r4 (5, 6, 7) = 1, 0, 0 → one 1 → **1**
r8 (9, 10, 11) = 1, 1, 0 → two 1s → **0**
Codeword = **01100011101**
</details>

**A11.** *(New)* The Hamming codeword `01100001101` is received. Find and correct the error.

<details><summary>Show answer</summary>

Recompute the checks: r8 r4 r2 r1 = 0101 = **5**. Flip position 5: the corrected codeword is `01100011101`, and the data is **0110011**.
</details>

**A12.** *(New)* Bit stuff `0111110111111001` and add the flags.

<details><summary>Show answer</summary>

`0 11111 [0] 0 11111 [0] 1 001` → stuffed data `011111001111101001` (2 stuffed 0s).
Frame: **01111110 011111001111101001 01111110**
</details>

---

## Part B: IP addressing

**B1.** With a mask of 255.255.240.0, how many valid host IPs are available?

<details><summary>Show answer</summary>

/20 leaves 12 host bits, so 2¹² − 2 = **4,094**.
</details>

**B2.** The IP address 141.43.201.17**/13** uses a subnet mask of 255.255.248.0. How many subnets are available?

<details><summary>Show answer</summary>

The mask is /21. Subnet bits = 21 − **13** = 8, so 2⁸ = **256**.
⚠️ When the question gives a **/n prefix**, count from **that**, not from the class.
</details>

**B3.** The IP address 145.43.201.17**/14** uses a subnet mask of 255.255.252.0. How many subnets are available?

<details><summary>Show answer</summary>

The mask is /22. 22 − 14 = 8, so 2⁸ = **256**.
</details>

**B4.** *(New)* The IP address 10.20.30.40/12 uses the mask 255.255.255.0. How many subnets?

<details><summary>Show answer</summary>

24 − 12 = 12, so 2¹² = **4,096**.
</details>

**B5.** 116.143.102.15/13 must be split into 512 subnets. What subnet mask should be used?

<details><summary>Show answer</summary>

512 = 2⁹, so borrow 9 bits: /13 + 9 = /22 = **255.255.252.0**.
</details>

**B6.** Is 145.4.56.127 a valid host with mask 255.255.255.128?

<details><summary>Show answer</summary>

**No.** /25 means blocks of 128: 0–127. .127 is the **broadcast**.
</details>

**B7.** Is 145.4.56.2 a valid host with netmask 255.255.250.0?

<details><summary>Show answer</summary>

**No.** 250 = `11111010`, which isn't a run of 1s followed by 0s, so this isn't a valid mask at all.
</details>

---

## Part C: TCP, delays and switching

**C1.** MSS = 6 KB, threshold = 58 KB, receiver's window = 80 KB. What is the congestion window after the 7th transmission?

<details><summary>Show answer</summary>

Sent: 6, 12, 24, 48, 58 (96 capped at 58), 64, 70. After the 7th: **76 KB**.
</details>

**C2.** RTT = 150 ms, receiver's window = 102 KB, threshold = 81 KB, MSS = 3 KB. How many transmissions does it take to send 345 KB?

<details><summary>Show answer</summary>

Sent: 3, 6, 12, 24, 48, 81 (capped), 84, 87. Running total: 3, 9, 21, 45, 93, 174, 258, **345** → **8 transmissions**.
</details>

**C3.** *(New)* MSS = 4 KB, threshold = 30 KB, receiver's window = 50 KB. There's a timeout on the 6th transmission. What is the window after the 10th?

<details><summary>Show answer</summary>

Sent: 4, 8, 16, 30 (capped), 34. The 6th sends 38 and times out, so threshold = 19 and window = 4. Then 4 → 8, 8 → 16, 16 → 19 (capped), 19 → **23 KB**.
</details>

**C4.** A 38 GB message, 7 intermediate nodes, 84 Mb/s, 30 ms circuit setup. What is the largest packet (KB) for packet switching to be the same or faster than circuit switching?

<details><summary>Show answer</summary>

7 × T ≤ S, so packet size ≤ 0.03 × 84,000,000 ÷ (8 × 7) = **45 KB**.
</details>

**C5.** *(New)* The same question with 4 nodes, 100 Mb/s and a 12 ms setup.

<details><summary>Show answer</summary>

0.012 × 100,000,000 ÷ (8 × 4) = 37,500 B = **37.5 KB**.
</details>

**C6.** A ─L1─ B ─L2─ C. L1: 750 m at 250 × 10⁶ m/s, 25 Mb/s. L2: 2,000 m at 200 × 10⁶ m/s, 70 Mb/s. P1 = 600 B, then P2 = 350 B straight after. When does C receive the last bit of P2?

<details><summary>Show answer</summary>

Propagation: L1 = 3 µs, L2 = 10 µs. P1 on L1 = 192 µs, so it arrives at B at 195 µs. P1 on L2 = 68.6 µs, so B is free at 263.6 µs.
P2 on L1 = 112 µs, from 192 to 304 µs, so it arrives at B at 307 µs (no queueing). P2 on L2 = 40 µs, from 307 to 347 µs. Add 10 µs propagation: **357 µs**.
</details>

**C7.** 1.28 Gb/s, 2.34 km, 260,000 km/s, 240 B packets. How many packets are sent before the first bit arrives?

<details><summary>Show answer</summary>

Propagation = 9 µs. Transmission = 1,920 ÷ 1.28 × 10⁹ = 1.5 µs. 9 ÷ 1.5 = **6**.
</details>

**C8.** *(New)* 3 km at 200,000 km/s, 2 Gb/s, 500 B packets. How many packets are on the link?

<details><summary>Show answer</summary>

Propagation = 15 µs. Transmission = 4,000 ÷ 2 × 10⁹ = 2 µs. 15 ÷ 2 = 7.5 → **7**.
</details>

---

## Part D: True or false?

Click each one to check.

<details><summary>D1. Half duplex can send data in both directions.</summary>

**True.** Both directions, but not at the same time.
</details>

<details><summary>D2. Thermal noise is reduced by heating up the receiver.</summary>

**False.** It's reduced by **cooling** (e.g. liquid nitrogen).
</details>

<details><summary>D3. The probability of frame error is always bigger than the probability of bit error (with more than 1 bit in a frame).</summary>

**True.** A frame fails if any one of its bits fails.
</details>

<details><summary>D4. TDM gives the whole of the resource for some of the time.</summary>

**True.** FDM gives **some** of the resource **all** of the time.
</details>

<details><summary>D5. Impulse noise is caused by a strong signal being close to a weak signal.</summary>

**False.** That's **cross-talk** (NEXT). Impulse noise comes from **external** equipment or activity.
</details>

<details><summary>D6. Crosstalk is caused by lightning strikes.</summary>

**False.** Lightning is a source of **impulse** noise.
</details>

<details><summary>D7. For the same BER, increasing the frame size increases the probability of frame error.</summary>

**True.** More bits means more chances of an error.
</details>

<details><summary>D8. If I halve the frame size, the probability of frame error is also halved.</summary>

**False.** That's only roughly true when p ≪ 1. The exact formula 1 − (1 − p_b)^L isn't linear.
</details>

<details><summary>D9. The physical layer delivers a reliable bit pipe service.</summary>

**False.** It's an **unreliable** bit pipe.
</details>

<details><summary>D10. Statistical multiplexing requires additional overhead to indicate the stream.</summary>

**True.** Each frame has to say which stream it belongs to.
</details>

<details><summary>D11. FDM and TDM are similar in that they split the link into independent parts.</summary>

**True.** FDM splits by frequency and TDM by time, but both create fixed, separate channels.
</details>

<details><summary>D12. Many frames will have the same redundant data sequence used for error detection and correction.</summary>

**True.** See A7: there are far more possible frames than check sequences.
</details>

<details><summary>D13. A two-dimensional parity scheme will always detect the error, even if it sometimes can't correct it.</summary>

**False.** 4 errors in a **rectangle** go undetected.
</details>

<details><summary>D14. A two-dimensional parity scheme will detect all even numbers of errors.</summary>

**False.** For example, the rectangle of 4 errors isn't detected.
</details>

<details><summary>D15. Sending multiple copies of the same frame, and using them to decide if it's correct, reduces the overall probability of frame error.</summary>

**True.** For example, 3 copies with a majority vote gives p ≈ 3L × p_b², but at the cost of capacity.
</details>

<details><summary>D16. IPv6 supports flows, unlike IPv4.</summary>

**True.** It has a flow label and per-flow handling. IPv4 routers ignored the Type Of Service field.
</details>

<details><summary>D17. IPv6 is not compatible with TCP or UDP.</summary>

**False.** IPv6 **is** compatible with TCP, UDP, OSPF, BGP and DNS. It's IPv4 it isn't compatible with.
</details>

<details><summary>D18. IPv6 is compatible with IPv4.</summary>

**False.** The two co-exist, but they aren't compatible.
</details>

<details><summary>D19. IPv6 uses 128-bit addresses.</summary>

**True.**
</details>

<details><summary>D20. Subnetting and CIDR are essentially the same, as they both use netmasks.</summary>

**False.** Both use netmasks, but CIDR allocates blocks on any bit boundary, while subnetting splits one network inside an organisation.
</details>

<details><summary>D21. Modern communication systems use digital transmission.</summary>

**True.**
</details>

<details><summary>D22. There is always noise on any transmission medium.</summary>

**True.** Thermal noise is always present.
</details>

<details><summary>D23. The physical layer is concerned with bit transmission.</summary>

**True.**
</details>

<details><summary>D24. UDP can deliver packets out of order.</summary>

**True.** UDP has no in-order delivery.
</details>

<details><summary>D25. An IPv4 address is resolved by ARP to give a MAC address.</summary>

**True.**
</details>

---

## Scoring

| Score | What to do |
|---|---|
| 45+ / 52 | Exam ready. Redo the ones you missed in a few days. |
| 35–44 | Good. Go back over the lessons for the topics you missed. |
| Below 35 | Work through Lessons 9–11 and 7 again, then retry. |
