# Lesson 12: Naming, DNS, ARP and MAC Addresses

[← CRC and framing](11-crc-and-framing.md) · [Index](../README.md) · [Next: Headers, layers and IPv6 →](13-headers-layers-ipv6.md)

*Based on the "Internet" lecture deck, slides 22 and 40–44.*

## The big idea

When you type `www.ucd.ie`, three different kinds of "address" are involved, and each one is turned into the next:

```
Host name  ──DNS──>  IP address  ──ARP──>  MAC address
www.ucd.ie            137.43.155.237         8:0:20:72:93:18
(for people)          (for routing)          (for the local link)
```

> **Analogy:** a friend's **name** (host name) → look it up in your contacts to get their **home address** (IP address) → when you reach their street, find the **actual front door** (MAC address).

## 1. The questions the Internet has to answer (slide 40)

| Question | Topic |
|---|---|
| How do we **name** end hosts? | Naming |
| How do we **address** end hosts? | Addressing (Lessons 6–7) |
| How do we **map names to addresses**? | DNS and ARP (this lesson) |
| How do we compute forwarding tables and forward packets? | Routing (later in the course) |
| How do hosts communicate reliably? | Reliable delivery (later) |
| How fast can sources send? | Congestion control (Lesson 5) |

## 2. Host names (domain names)

- These are easy names like `mail.google.com`, `irishtimes.com` or `ucd.ie`.
- Sites can make up more names that end in their root name: `www.ucd.ie`, `docs.ucd.ie`, `mail.ucd.ie`.
- There are many top-level endings: `.com`, `.org`, `.net`, `.ie` (Ireland) and so on.
- **Big sites** may use **many computers** for one name, so **one name → many IP addresses**.
- **Small sites** may put **many names** on **one computer**, so **one IP → many names**.

## 3. IP addresses and the NIC

- A **NIC** (Network Interface Card) is the hardware that connects a device to the network.
- **Each NIC on a device must have an IP address.** It's the device's **globally unique** address.
- The IP address gets the packet to the right **device**. The **port** number then gets it to the right **application** on that device (slide 22). Don't confuse this kind of port with the physical ports on a router.

## 4. DNS: Domain Name System (slide 43)

DNS translates **user-friendly names into IP addresses**.

How it works:

1. Every host is set up with the address of a **local name server**.
2. Name servers form a **hierarchy**: a **root** name server at the top, then servers below it (for example Arizona, then CS within Arizona).
3. The addresses of the root servers are **well-known** and stored in the local name server's configuration file.
4. If a name server **can't resolve the whole name**, it answers with **as much as it can**, and the lookup carries on further down the hierarchy.

```
                 Root name server
                /                \
     Arizona name server     NASA name server
        /           \
  CS name server   ECE name server   → resolves cs.arizona.edu
```

## 5. MAC addresses

- A **MAC** (Medium Access Control) address is the **physical** address of a NIC, for example `8:0:20:72:93:18`.
- It's used to deliver frames on the **local** network (the data link layer, L2).
- *(Beyond the notes)* A MAC address is **48 bits** long. That's probably why "IPv4 is 48 bits" appears as a false statement in the quizzes: IPv4 is 32 bits.

## 6. ARP: Address Resolution Protocol (slide 44)

ARP finds the **MAC address** that belongs to an **IP address** on the local network.

1. Each host keeps an **ARP cache**: a table of IP → MAC mappings. Entries **time out** after about **15 minutes**.
2. If the destination IP isn't in the cache, the host **broadcasts an ARP query** to the whole local network: "who has this IP? Tell me your MAC."
3. The query includes the **sender's own IP and MAC**, so every other host can add the sender to its cache.
4. **If the destination is on the local network**, it replies with its MAC address (an **ARP reply**).
5. **If the destination isn't on the local network**, an **ARP server** replies with **its own** MAC address. This is called **proxy ARP**, and the packet then goes to that server to be forwarded.

## 7. RARP: Reverse ARP

The opposite direction: **MAC → IP**.

- It's used when a host **doesn't know its own IP address**, for example a **diskless workstation** that is booting up.
- The host **broadcasts its physical (MAC) address** and asks for its IP, and a **RARP server** replies.

## 8. Summary table

| Protocol | Converts | Direction |
|---|---|---|
| **DNS** | host name → IP address(es) | name → address |
| **ARP** | IP address → MAC address | local network |
| **RARP** | MAC address → IP address | "what's my IP?" |

## 9. True/false facts from the course answer keys

| Statement | Verdict |
|---|---|
| A hostname can have many IP addresses | ✅ True |
| An IP address can support many domain names | ✅ True |
| A domain name can be resolved to many MAC addresses (using DNS and ARP) | ✅ True |
| A domain name can resolve to many IPs, each with a single MAC | ✅ True |
| An IPv4 address is resolved by ARP to give a MAC address | ✅ True |
| An IP address can have multiple MAC addresses | ❌ False |
| A MAC address can support many IP addresses | ❌ False |
| A MAC address is resolved by DNS to give a single IPv4 address | ❌ False (DNS works on **names**; MAC → IP is RARP) |

**How to remember it:** the "many" links happen at the **name** level (DNS). Once you're down at **IP ↔ MAC** (ARP), it's **one to one**.

---

## Quiz

**Q1.** What does DNS translate, and what does ARP translate?

<details><summary>Show answer</summary>

**DNS:** host name → IP address. **ARP:** IP address → MAC address.
</details>

**Q2.** A host wants to send to an IP that isn't in its ARP cache. What does it do?

<details><summary>Show answer</summary>

It **broadcasts an ARP query** on the local network, asking for the MAC address of that IP. The query includes its own IP and MAC.
</details>

**Q3.** What is proxy ARP?

<details><summary>Show answer</summary>

When the destination is **not** on the local network, an ARP server replies with **its own** MAC address, so the packet is sent to it to be forwarded on.
</details>

**Q4.** When would a host use RARP?

<details><summary>Show answer</summary>

When it **doesn't know its own IP address**, for example a diskless workstation booting up. It broadcasts its MAC and a RARP server replies with its IP.
</details>

**Q5.** If a name server can't resolve the whole name, what does it do?

<details><summary>Show answer</summary>

It returns an answer for **as much of the name as it can**, and the lookup carries on through the hierarchy.
</details>

**Q6.** True or false: an IP address can have multiple MAC addresses.

<details><summary>Show answer</summary>

**False** (course answer key). IP ↔ MAC is one to one. The many-to-many links are at the name level.
</details>

**Q7.** What is the difference between an IP address and a port number?

<details><summary>Show answer</summary>

The **IP address** identifies the **device** (its NIC). The **port number** identifies the **application** on that device.
</details>
