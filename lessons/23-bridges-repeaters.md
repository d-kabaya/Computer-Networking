# Lesson 23: Bridges, Repeaters and the Module Summary

[← Link-state routing](22-routing-link-state.md) · [Index](../README.md)

*Based on the "Internetworking and Routing" lecture deck, slides 29–52. This is the last lesson of the module.*

## The big idea

Routers connect **different networks** using **IP addresses** (Lessons 21–22). Inside one network, LAN segments can also be joined, at the **data link layer**, using **MAC addresses**. The device for that is a **bridge**.

The three internetworking devices in the module, from the bottom layer up:

| Device | Layers it works in | Address it understands | What it does |
|---|---|---|---|
| **Repeater** | Physical only | None | **Regenerates** ("cleans up") the signal to extend a network's reach |
| **Bridge** | Physical + data link | **Physical (MAC)** | Joins LAN segments and **filters** frames by MAC address |
| **Router** | Physical + data link + network | **Logical (IP)** | Relays packets **between networks** using routing tables |

> **Analogy:** delivering post.
> - **Repeater:** a loudspeaker. It makes the message louder so it reaches further, but has no idea who it's for.
> - **Bridge:** a receptionist for one building. It knows which floor each person sits on and only sends post to that floor.
> - **Router:** the national postal service. It uses the **address** (town, street) to route post between buildings and cities.

---

## 1. Repeaters (slide 50)

- An electronic device that **regenerates** ("cleans up") incoming signals.
- It lets you **extend the physical reach** of a network.
- **Physical layer only:** it copies **everything**, including errors and collisions, to the other side. It has no logic to decide what to forward.

---

## 2. Bridges (slides 29–32)

- A bridge works in the **physical and data link layers**.
- It **knows the physical (MAC) addresses** of the connected nodes.
- It can **divide a large network into smaller segments**, or **join two LANs** that weren't connected before.

**Unlike a repeater**, a bridge contains **logic** that keeps each segment's traffic separate, so a bridge can **filter or route** traffic. This helps with:
- controlling **traffic congestion**
- **isolating problems** (a fault on one segment doesn't spread)
- **security** (frames aren't sent to segments that don't need them)

### Traffic filtering (slide 32)

When a frame arrives, the bridge **regenerates the signal** and also **checks the destination address**. It forwards the frame **only to the segment where that address is**.

```
Upper segment:  A ─────── D ─────────────
                         │
                      [Bridge]
                         │
Lower segment:  F ─────── G ─────────────

A → D: both on the upper segment → the bridge does NOT forward it.
       The frame is relayed only on the upper segment.
A → G: G is on the lower segment → the bridge forwards it there.
```

---

## 3. Transparent bridges (slides 33–37)

The first type. **The bridges do the routing**, using a table called the **Forwarding Database (FDB)**.

### How it forwards a frame

1. Read the frame's **source** and **destination** addresses.
2. Look up the **destination** in the FDB to find which **output port** to use.
3. **If that's the same port the frame arrived on**, the destination is on the same segment, so **discard** the frame and move on to the next one. This is called **frame filtering**.
4. Otherwise, forward it on that port.

> ⚠️ **Port here means a physical connection on the bridge**, not a TCP/UDP port number (Lesson 4).

### Bridge learning

- The FDB **isn't set up by hand**. It is **built and kept up to date automatically** during normal operation. This is called **bridge learning**.
- The bridge **notes the source address** of every frame, and records it against the port the frame **came in on**. ("A frame from station 5 arrived on port 3, so station 5 must be through port 3.")
- **If the destination isn't in the FDB yet**, the bridge sends a copy of the frame out of **all its other ports**. Each bridge repeats it, so the frame spreads through the network. This is **flooding**.

> **Analogy:** a new receptionist who doesn't know anyone yet. They take post for an unknown name to every floor. Each time someone comes down to collect post, the receptionist notes which floor they came from. Soon they can deliver straight to the right floor.

### FDB example (slide 34)

```
           Bridge 1                       Bridge 2
     Port 1   Port 2   Port 3 ─────── (to Bridge 2)
       │        │                       Port 1     Port 2
     1  2     3  4                       5  6       7  8
```

**Bridge 1's FDB:**

| Station | Port |
|---|---|
| 1 | 1 |
| 2 | 1 |
| 3 | 2 |
| 4 | 2 |
| 5 | 3 |
| 6 | 3 |
| 7 | 3 |
| 8 | 3 |

Stations 5–8 are all **port 3** for Bridge 1. It doesn't care that they're behind another bridge, only which of its own ports leads to them.

| Frame | Lookup | Action |
|---|---|---|
| 1 → 2 (arrives on port 1) | 2 is on port 1, the **same** port | **Discard** (filtered) |
| 1 → 4 (arrives on port 1) | 4 is on port 2 | Forward on **port 2** |
| 3 → 7 (arrives on port 2) | 7 is on port 3 | Forward on **port 3**, then Bridge 2 sends it on its port 2 |

### Why "transparent" (slide 37)

- The bridges make all the routing decisions, so the **stations don't even know the bridges are there**: they're **transparent** to them.
- Also called a **learning bridge**, because it builds its tables automatically by noting source addresses.

### Loops and the spanning tree (slides 37–38)

- If **more than one bridge connects two LANs** (e.g. a spare bridge added for reliability), a **loop** can form. Frames, especially flooded ones, could then **circulate forever**.
- **Fix:** the bridges learn the topology and build a **loop-free spanning tree**. Some bridges (or ports) are left **unused**.
- The **alternative bridges** added for reliability **aren't used** while the tree is in place.
- The spanning tree algorithm **runs at regular intervals**, so it **dynamically picks** a set of bridges from those currently working. If an active bridge fails, a spare can be brought into the new tree.

---

## 4. Source routing bridges (slides 39–40)

The second type. **The sending station (the source) does the routing.**

- **Before sending any frames**, the end station picks the **exact route**: which segments and bridges to go through.
- It learns the topology using **discovery frames**.
- The route is put at the **head of the frame** in a **routing information field**:

```
Routing information field:
┌─────────────────┬──────────────────────┬──────────────────────┬─────┬──────────────────────┐
│ Routing control │ Routing designator 1 │ Routing designator 2 │ ... │ Routing designator N │
└─────────────────┴──────────────────────┴──────────────────────┴─────┴──────────────────────┘
Routing control  = frame type · maximum frame size · routing field length
Each designator  = segment identifier + bridge identifier
```

- A bridge only has to **search the routing field for its own bridge ID**. It forwards the frame **only if** its ID is there **and** is followed by the ID of a segment connected to one of its ports.
- Bridges just **follow the instructions** in the frame.

### Example (slide 40)

```
     A    B                                     C
  ───┴────┴── Segment 1 ──[B1]── Segment 2 ──[B2]── Segment 3 ───┴───
                                    │
                                    D
```

**A's routing table:**

| Destination | Route |
|---|---|
| B | Seg 1 |
| C | Seg 1, **B1**, Seg 2, **B2**, Seg 3 |
| D | Seg 1, **B1**, Seg 2 |

> **Analogy:** transparent bridging is like posting a letter: you just write the address and the post office works out the route. Source routing is like giving a taxi driver turn-by-turn directions: the driver just follows them.

---

## 5. Transparent vs source routing (slide 42)

| | Transparent bridges | Source routing bridges |
|---|---|---|
| **Who routes** | The **bridges** co-operate; invisible to the stations | The **source station** puts the route in the frame |
| **Quality of routes** | A spanning tree prevents loops, but routes are **unlikely to be optimal** for every pair of stations | **All** routes are found, so the source can choose the **optimum** one |
| **Use of bandwidth** | Some ports are **blocked** to keep a tree, so **not all** bandwidth is used | All segments are used in route-finding, so in theory the **full bandwidth** can be used |
| **Stations need to…** | Nothing special | Discover routes and keep routing tables |

---

## 6. Module summary (slide 52)

| Part | Topics | Lessons |
|---|---|---|
| **Network basics** | Links, switches and paths · asynchronous operation · circuit vs packet switching (VC and datagram) | 1–2 |
| **Internet** | Layered communication · TCP congestion control · names (DNS, ARP), IP, CIDR, subnetting | 3–8, 12–14 |
| **Physical and data link** | Errors, noise, parity, Hamming, CRC · flow and error control · ARQ: Stop-and-Wait, Go-back-N, Selective Repeat · HDLC, Ethernet, Token Ring, Wi-Fi | 9–11, 15–20 |
| **Routing** | Distance-vector and link-state · transparent and source routing bridges | 21–23 |

---

## Quiz

**Q1.** Which layers do a repeater, a bridge and a router each work in?

<details><summary>Show answer</summary>

**Repeater:** physical only · **Bridge:** physical and data link · **Router:** physical, data link and network
</details>

**Q2.** What can a bridge do that a repeater can't?

<details><summary>Show answer</summary>

**Filter** traffic. It reads the **destination MAC address** and forwards a frame only to the segment where that station is. A repeater just regenerates and copies everything.
</details>

**Q3.** A transparent bridge receives a frame on port 2, and the FDB says the destination is on port 2. What happens, and what is this called?

<details><summary>Show answer</summary>

The frame is **discarded**, because the destination is on the same segment it came from. This is **frame filtering**.
</details>

**Q4.** How does a transparent bridge learn where stations are?

<details><summary>Show answer</summary>

It notes the **source address** of each frame and the **port it arrived on**. This is **bridge learning**.
</details>

**Q5.** What does a transparent bridge do with a frame whose destination isn't in its FDB?

<details><summary>Show answer</summary>

It **floods** it: it sends a copy out of **every other port**.
</details>

**Q6.** Why do transparent bridges build a spanning tree?

<details><summary>Show answer</summary>

If more than one bridge connects two LANs, a **loop** can form and frames could **circulate forever**. A spanning tree has **no loops**.
</details>

**Q7.** Using slide 34's FDB, station 3 sends a frame to station 6. What does Bridge 1 do?

<details><summary>Show answer</summary>

The frame arrives on **port 2**. The FDB says 6 is on **port 3**, a different port, so Bridge 1 **forwards it on port 3** towards Bridge 2, which delivers it on its port 1.
</details>

**Q8.** In source routing, who decides the route, and where is it stored?

<details><summary>Show answer</summary>

The **source station** decides, using discovery frames to learn the topology. The route is stored in the **routing information field at the head of the frame**.
</details>

**Q9.** Give one advantage of source routing over transparent bridging.

<details><summary>Show answer</summary>

Either: the source can pick the **optimum** route (all routes are known), or **all segments** can be used, so in theory the **full bandwidth** is available (transparent bridges block some ports).
</details>

**Q10.** A router and a bridge both forward traffic. What is the key difference?

<details><summary>Show answer</summary>

A **bridge** uses **physical (MAC) addresses** at the data link layer to join segments of a LAN. A **router** uses **logical (IP) addresses** at the network layer to relay packets **between different networks**.
</details>

---

## What to remember

- **Repeater** (physical): regenerates signals, extends reach, no filtering.
- **Bridge** (physical + data link): knows **MAC** addresses, splits or joins LANs, **filters** traffic (helps congestion, fault isolation and security).
- **Router** (up to network): knows **IP** addresses, relays between networks.
- **Transparent bridge:** bridges route using the **FDB** · same port → **discard** (filtering) · unknown → **flood** · learns from **source addresses** · builds a **loop-free spanning tree**, leaving spare bridges unused.
- **Source routing bridge:** the **source** picks the full route with **discovery frames** and puts it in the frame header · bridges look for **their own ID**.
- **Transparent:** simple for stations, but routes may not be optimal and some bandwidth is unused. **Source routing:** optimum routes, full bandwidth, but more work for stations.
