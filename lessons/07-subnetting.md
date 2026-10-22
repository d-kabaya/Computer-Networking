# Lesson 7: Netmasks, Subnetting and CIDR

[← IPv4 addressing](06-ipv4-addressing.md) · [Index](../README.md) · [Next: Exam revision →](08-exam-revision.md)

## The big idea

Classes were too rigid: a Class B was too big for most organisations, and a Class C too small. So we use a **netmask** to put the network/host dividing line **anywhere**, and **subnetting** to split one network into smaller ones.

> **Analogy:** one long street with 256 houses (one network). We cut it into equal parts (subnets). The first house in each part is the part's "name" (the subnet address), and the last house is the loudspeaker (the broadcast).

## 1. The netmask

- 32 bits: **1s** for the network part, then **0s** for the host part.
- **/n** ("slash notation") = the number of 1s. `255.255.255.240` = 28 ones = **/28**.
- The 1s must be **contiguous**, so `255.255.255.170` (`10101010`) is **not** a valid mask.

| Mask octet | Binary | Block size |
|---|---|---|
| 128 | 10000000 | 128 |
| 192 | 11000000 | 64 |
| 224 | 11100000 | 32 |
| 240 | 11110000 | 16 |
| 248 | 11111000 | 8 |
| 252 | 11111100 | 4 |
| 254 | 11111110 | 2 |
| 255 | 11111111 | 1 |

## 2. Why AND?

To find which network an address is on, the computer **ANDs** the IP with the mask:
- 1 AND x = x (the network bits are **kept**)
- 0 AND x = 0 (the host bits are **wiped**)

The result is the **subnet address**. If two addresses give the same result, they're on the same subnet, so the packet can be delivered directly instead of going to the router.

```
IP:    192.168.5.13   = 11000000.10101000.00000101.00001101
Mask:  255.255.255.0  = 11111111.11111111.11111111.00000000
AND:   192.168.5.0    = 11000000.10101000.00000101.00000000
```

## 3. The 5-step method

1. **Cutting octet:** the mask octet that is **not 255 and not 0**.
2. **Block size** = 256 − that octet.
3. **Start** = ⌊IP octet ÷ block⌋ × block (the multiple of the block at or below the IP).
4. **Subnet address:** copy the octets before the cut, write the start, then **0** for everything after.
5. **Broadcast:** copy the octets before the cut, write next start − 1, then **255** for everything after.

**Usable hosts** = subnet + 1 up to broadcast − 1.

### Example A: cut in the 4th octet

**107.73.14.224 / 255.255.255.192**
1. The cut is at 192 (4th octet). 2. Block = 64. 3. 224 ÷ 64 = 3.5 → 3 × 64 = **192**.
4. Subnet = **107.73.14.192**. 5. Broadcast = 256 − 1 = 255 → **107.73.14.255**. Hosts .193 to .254.

### Example B: cut in the 3rd octet

**52.92.172.187 / 255.255.224.0**
1. The cut is at 224 (3rd octet). 2. Block = 32. 3. 172 ÷ 32 = 5.4 → 5 × 32 = **160**.
4. Subnet = **52.92.160.0**. 5. Broadcast = 192 − 1 = 191 → **52.92.191.255**.

## 4. Valid host, subnet address or broadcast?

Find the block, then check where the address sits:
- Equals the **first** address → subnet address ❌
- Equals the **last** address → broadcast ❌
- In between → **valid host** ✅

When the cut is in the **3rd octet** (/17 to /23), addresses ending in **.0 or .255 can be valid hosts**. For example, 172.16.9.0/22 sits inside the block 172.16.8.0 to 172.16.11.255.

**Broadcast endings:** with a cut in the 4th octet the last octet is (a multiple of the block) − 1. For /26 that's 63, 127, 191 or 255, and for /28 it's always odd. With a cut in the 3rd octet, the broadcast always ends in **.255**.

## 5. Counting

```
Host bits        = 32 − n
Usable hosts     = 2^(32−n) − 2
Total addresses  = 2^(32−n)
Subnet bits      = n − class bits     (A = 8, B = 16, C = 24)
Number of subnets = 2^(subnet bits)
```

**Example: 151.8.201.217 / 255.255.255.240 (Class B)**

```
|<- 16 bits: Class B network ->|<- 12 bits: subnet ->|<- 4: host ->|
```

- Hosts = 2⁴ − 2 = **14**
- Subnet bits = 28 − 16 = 12, so subnets = 2¹² = **4,096**
- Block 16 → 217 is in 208–223 → broadcast **151.8.201.223**

## 6. Host ID when the cut is inside an octet

**Host ID = IP − subnet address**

151.8.201.217/28 → the subnet is .208, so 217 − 208 = **9**.

## 7. CIDR vs subnetting

- **CIDR** (Classless Inter-Domain Routing) allocates address blocks on **any bit boundary**, e.g. ICANN → RIPE → ISP → UCD.
- **Subnetting** splits **one** network into smaller sub-networks inside an organisation. From outside, it still looks like one network.
- They are **not** the same thing.

## 8. How to write answers

- Addresses (subnet, broadcast, host) are written in **full**: `107.73.14.192`, not `.192`.
- A host ID is written as only the host octets: `14.10`.
- Counts are just numbers: `4,096`.

---

## Quiz

**Q1.** How many usable hosts are there with mask 255.255.192.0?

<details><summary>Show answer</summary>

192 = 2 ones, so the mask is /18. 2¹⁴ − 2 = **16,382**.
</details>

**Q2.** 128.96.134.15 / 255.255.128.0: what is the subnet address? Is 128.96.255.139 on the same subnet? What is the broadcast?

<details><summary>Show answer</summary>

Block 128 in the 3rd octet: 134 is in 128–255, so the subnet is **128.96.128.0**. 255 is also in 128–255, so **yes**, it's the same subnet. Broadcast = **128.96.255.255**.
</details>

**Q3.** 172.16.45.9 / 255.255.240.0: what are the subnet and broadcast addresses?

<details><summary>Show answer</summary>

Block 16: 45 ÷ 16 → 2 × 16 = 32, so the subnet is **172.16.32.0**. The next start is 48, so the broadcast is **172.16.47.255**.
</details>

**Q4.** Is 10.0.255.255 a valid host with /16? With /8?

<details><summary>Show answer</summary>

With /16 the host part is 255.255, all 1s, so it's the **broadcast** ❌. With /8 the host part is 0.255.255, which isn't all 1s, so it's a **valid host** ✅.
</details>

**Q5.** IP 102.45.68.201 with netmask 255.255.240.0. How many subnets are there?

<details><summary>Show answer</summary>

102 is Class A (8 bits). The mask is /20, so the subnet bits = 20 − 8 = 12, giving 2¹² = **4,096**.
</details>

**Q6.** Are 156.0.214.17/22 and 156.0.214.255/22 in the same subnet?

<details><summary>Show answer</summary>

Block 4 in the 3rd octet: 214 is in 212–215, so both are in 156.0.212.0. **Yes.** (.214.255 is an ordinary host here.)
</details>

**Q7.** What is the broadcast address of 192.168.153.33/21?

<details><summary>Show answer</summary>

/21 means 5 ones in the 3rd octet (248), so the block is 8. 153 is in 152–159, so the broadcast is **192.168.159.255**.
</details>

**Q8.** True or false: 153.182.201.187/30 has a broadcast address of 153.182.201.191.

<details><summary>Show answer</summary>

**False.** /30 means block 4: 187 is in 184–187, so .187 is itself the broadcast.
</details>
