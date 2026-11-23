# Lesson 21: Routing Basics and Distance-Vector Routing

[← Wi-Fi (802.11)](20-wifi-802-11.md) · [Index](../README.md) · [Next: Link-state routing →](22-routing-link-state.md)

*Based on the "Internetworking and Routing" lecture deck, slides 1–14.*

## The big idea

Up to now, the data link layer has moved frames between **neighbouring** devices. But a packet from your laptop to a server in another country crosses many networks. At every **router** along the way, something has to decide: **which link should this packet go out on next?** That decision is **routing**, and it happens at the **network layer**.

> **Analogy:** driving across the country with no map, only road signs at each junction. Each junction (router) has a sign (routing table) that says "for Galway, take exit 2". You don't need to know the whole route, only the next turn at each junction.

The deck covers two families of algorithm:

| | Where | Algorithms |
|---|---|---|
| **Routing** | Network layer (routers) | **Distance-vector** (this lesson), **link-state** (Lesson 22) |
| **Bridging** | Data link layer (bridges) | **Transparent**, **source routing** (Lesson 23) |

---

## 1. Broadcast vs point-to-point (slide 2)

| | Broadcast | Point-to-point |
|---|---|---|
| **Channel** | One channel **shared by all** nodes | **Many possible paths** between a pair of nodes |
| **Who receives** | One node, all nodes, or maybe a group. The **address field** says who it's for. | The message may have to pass through **intermediate nodes** |
| **Routing needed?** | No: everyone hears everything | **Usually yes**: to decide whether a path exists and, if there are several, which to use |
| **Shapes (topologies)** | e.g. Ethernet bus, Wi-Fi | Star, ring, tree, mesh, intersecting rings, irregular |

---

## 2. Routers (slides 3–4)

- A router **relays packets between interconnected networks**.
- It is connected to **2 or more networks** at once, and it has an **address on each** of them.
- If no single router connects the sender's network to the receiver's, the packet is passed **router to router**, each time to one that is (hopefully) **"nearer"** the receiver, until it arrives.
- A router works in the **network, data link and physical layers**. It knows **network-layer ("logical") addresses**, i.e. IP addresses.

---

## 3. What a routing algorithm does (slide 5)

A routing algorithm is **the logic a router uses to decide, for each incoming packet, which output link to send it on**.

- **Datagram** packet switching: the decision is made **for every packet**.
- **Virtual circuit** packet switching: the decision is made **only once, at VC set-up** (Lesson 2).

### Desirable properties

| Property | Meaning |
|---|---|
| **Correctness, simplicity, efficiency** | Obviously |
| **Robustness** | Keeps working through failures, because you usually can't "re-boot" the entire network |
| **Stability** | Reaches a steady state (equilibrium) in a reasonable time |
| **Fairness** and **optimality** | Often **in conflict** with each other (see below) |

**Fairness vs optimality (the slide 5 diagram):** flows A→A′, B→B′ and C→C′ each fill one section of a shared line. That gives the **maximum total throughput**, but a flow from **X to X′**, which needs all three sections, gets **nothing**. Optimal for the network, unfair to X.

**"Optimal" with respect to what?** Average packet delay? Total throughput? These conflict too: running a network near full capacity means **long queues** in the routers. The usual **compromise** is to **minimise the number of hops** a packet needs.

---

## 4. Least-cost routing (slide 6)

- Each link is given a **cost** (a value for using it).
- The cost of a route is the **combination** (usually the sum) of its links' costs.
- The **best route** is the one with the **lowest cost**.

| Cost given to each link | The algorithm finds the… |
|---|---|
| **1** for every link | Route with the **fewest hops** |
| Financial cost | **Cheapest** route |
| Packet delay on the link | **Minimum-delay** route |
| Packet transmission time | **Maximum-bandwidth** route (a fast link has a short transmission time, so a low cost) |
| A combination of these | Whatever balance you choose |

### Static vs adaptive

| | Nonadaptive (static) | Adaptive |
|---|---|---|
| Routes decided | **In advance**, not from measurements | **Change** when the topology or traffic changes |
| Information from | — | Neighbours only, or all routers |
| When updated | — | Periodically, on a topology change, or when traffic changes by more than a threshold |
| Extreme case | — | A new route for **every packet** |

**In practice**, routing should be **adaptive and decentralised** (slide 7).

---

## 5. Distance-vector vs link-state: the headline difference (slide 7)

| | Distance-vector | Link-state |
|---|---|---|
| **Tells** | Its **neighbours** only | **All** routers (by flooding) |
| **About** | The **entire network** (its whole routing table) | Only its **own neighbourhood** |
| **When** | At **regular intervals** (e.g. every 30 s) | When there's a **change** |
| **Converges** | Slowly (count-to-infinity) | **Faster**, so it's more widely used |

**Neighbouring routers** = connected by a direct link, e.g. the same LAN.
**Converges** = works out the optimal routes for a given network layout.

> **Memory trick:** distance-vector = "tell your **neighbours** about **everyone**". Link-state = "tell **everyone** about your **neighbours**".

---

## 6. Distance-vector routing (slides 8–13)

### The example network (slide 8)

Cost = **1 per link**, so it finds **minimum-hop** routes. Clouds are LANs (numbered networks), and A–F are routers.

```
     ┌── Net 14 ── B ── Net 55 ──┐
     A                            C
     ├── Net 78 ── F ── Net 92    │
     │                            │
   Net 23                       Net 66
     │                            │
     E ────────── Net 08 ──────── D
```

*(Net 92 connects only to F.)*

### The routing table (slide 10)

| Network ID | Cost | Next hop |
|---|---|---|
| The **final destination** network | **Hops** from this router to it | The **neighbouring router** to send the packet to |

**At the start**, each router knows only the networks it's **directly connected** to, each at cost 1 with no next hop. For example, A starts with:

| Net | Cost | Next hop |
|---|---|---|
| 14 | 1 | — |
| 23 | 1 | — |
| 78 | 1 | — |

Each router then sends its table to its neighbours, and they send theirs. After one exchange A learns about its neighbours' networks, after two about its neighbours' neighbours' networks, and so on, until the information has spread across the whole network.

### The update algorithm: distributed Bellman-Ford (slide 11)

When a router receives a table from a neighbour:

1. **Add 1** to every cost in it, because that neighbour is 1 hop away.
2. **New destination?** Add it to the table, with that neighbour as the next hop.
3. **Destination already in the table?**
   - **Same next hop:** **replace** the entry with the new information, **even if the cost is greater**. ("New information invalidates old": that neighbour is your route, and it's telling you its route got worse.)
   - **Different next hop:** replace it **only if the new cost is lower**.

### Slide 11 example: A receives B's table

```
Received from B:    14 1     55 1
After +1:           14 2 B   55 2 B

A's old table:      14 1 —   23 1 —   78 1 —
A's new table:      14 1 —   23 1 —   55 2 B   78 1 —
```

- **14:** A already has cost 1 (direct). B offers 2 via a different next hop, which isn't lower, so A **keeps 14 1 —**.
- **55:** new destination, so A **adds 55 2 B**.

### Slide 12 example: all the rules at once

Router X receives this message from C, and adds 1 to each cost:

| | Net2 | Net3 | Net6 | Net8 | Net9 |
|---|---|---|---|---|---|
| From C | 4 | 8 | 4 | 3 | 5 |
| After +1 | 5 | 9 | 5 | 4 | 6 |

| Net | Old entry | Offer from C | Rule | New entry |
|---|---|---|---|---|
| Net1 | 7 A | (nothing) | No new information | **7 A** |
| Net2 | 2 C | 5 | **Same** next hop (C) → replace, even though it's worse | **5 C** |
| Net3 | — | 9 | **New** destination → add | **9 C** |
| Net6 | 8 F | 5 | Different next hop, **lower** → replace | **5 C** |
| Net8 | 4 E | 4 | Different next hop, **not lower** (a tie) → keep | **4 E** |
| Net9 | 4 F | 6 | Different next hop, **higher** → keep | **4 F** |

### The converged tables (slide 13)

After enough exchanges every router has a route to every network. For example, **A's final table**:

| Net | Cost | Next hop |
|---|---|---|
| 08 | 2 | E |
| 14 | 1 | — |
| 23 | 1 | — |
| 55 | 2 | B |
| 66 | 3 | E |
| 78 | 1 | — |
| 92 | 2 | F |

**How to read one row:** to reach Net 66, A sends the packet to **E**. The cost is **3 hops**: through Net 23 to E, through Net 08 to D, then onto Net 66.

> **Note (slide 13):** when two routes cost the same, which one a router picks **depends on the exact order the updates arrived in**.

---

## 7. Distance-vector in practice (slide 14)

- The original **ARPANET** (the forerunner of the Internet) used distance-vector routing.
- In the Internet it became **RIP** (Routing Information Protocol).
- A variation is used in **BGP** (Border Gateway Protocol), which finds routes **between Autonomous Systems**. An **AS** is a part of the Internet (e.g. a network) managed by one organisation.
- Link costs don't have to be 1: they can be delay, queue length, and so on.

### The count-to-infinity problem

- Distance-vector **converges slowly** under some conditions, especially after a **link or router fails**.
- **The basic flaw:** a router only hears from its **neighbours**, and their information may be **out of date**: it may not yet reflect the failure.
- Ad-hoc fixes (e.g. **split horizon**) have been tried, but they either don't fully solve it or are hard to implement.
- This slow convergence was a **main reason link-state routing was developed**.

> *Beyond the notes, a simple picture of what goes wrong:* routers X → Y → Z in a line, and the link to Z's network fails. Y knows its route is gone, but X still advertises "I can reach that network in 2 hops". Y believes X and sets its cost to 3, via X. X then hears Y's 3 and moves to 4, Y moves to 5, and so on. They count upwards together, very slowly, towards "infinity" (unreachable), and packets bounce between X and Y the whole time.

---

## Quiz

**Q1.** What is the difference between a broadcast network and a point-to-point network?

<details><summary>Show answer</summary>

**Broadcast:** one channel shared by all nodes; the address field says who should receive it. **Point-to-point:** many possible paths between nodes, through intermediate nodes, so a **routing algorithm** is usually needed.
</details>

**Q2.** In which layers does a router operate, and what kind of address does it use?

<details><summary>Show answer</summary>

**Network, data link and physical.** It uses **network-layer (logical) addresses**, i.e. IP addresses.
</details>

**Q3.** When is the routing decision made in datagram switching, and when in virtual circuit switching?

<details><summary>Show answer</summary>

**Datagram:** for **every packet**. **Virtual circuit:** **once, at VC set-up**.
</details>

**Q4.** If every link has cost 1, what does least-cost routing find? What if the cost is the transmission time on each link?

<details><summary>Show answer</summary>

Cost 1 → the **fewest hops**. Transmission time → the **maximum-bandwidth** route.
</details>

**Q5.** Complete the sentences: distance-vector tells its ______ about the ______; link-state tells ______ about its ______.

<details><summary>Show answer</summary>

Distance-vector tells its **neighbours** about the **entire network**. Link-state tells **all routers** about its **neighbourhood**.
</details>

**Q6.** Router X has this table. It receives the message shown from neighbour C. What is X's new table?

| X's table | | |
|---|---|---|
| Net1 | 3 | B |
| Net2 | 5 | C |
| Net4 | 2 | D |
| Net5 | 6 | C |

Message from C: Net1 1 · Net2 6 · Net3 2 · Net4 4 · Net5 2

<details><summary>Show answer</summary>

After adding 1: Net1 2 · Net2 7 · Net3 3 · Net4 5 · Net5 3

| Net | Rule | New entry |
|---|---|---|
| Net1 | Different next hop (B), 2 < 3, lower → replace | **2 C** |
| Net2 | **Same** next hop (C) → replace even though worse | **7 C** |
| Net3 | New → add | **3 C** |
| Net4 | Different next hop (D), 5 > 2 → keep | **2 D** |
| Net5 | Same next hop (C) → replace | **3 C** |
</details>

**Q7.** Why does a router replace an entry with a **worse** cost when the information comes from the same next hop?

<details><summary>Show answer</summary>

That neighbour **is** the route. If it says the cost has gone up (e.g. because a link failed), the old, lower cost is no longer true. **New information invalidates old.**
</details>

**Q8.** What is the count-to-infinity problem, and why does it happen?

<details><summary>Show answer</summary>

Distance-vector routing **converges very slowly after a failure**. Routers only hear from their **neighbours**, whose information may be **out of date**, so they keep believing in routes that no longer exist and slowly increase their costs.
</details>

**Q9.** Name a protocol that uses distance-vector routing, and one that uses a variation of it.

<details><summary>Show answer</summary>

**RIP** uses distance-vector. **BGP** uses a variation, for routing between Autonomous Systems.
</details>

---

## What to remember

- **Routing** = choosing the output link for each packet. Network layer. Done by **routers** (network, data link and physical layers; logical/IP addresses).
- Datagram: decide **per packet**. Virtual circuit: decide **at set-up**.
- Good algorithm: correct, simple, efficient, **robust**, **stable**, fair and optimal (fairness and optimality conflict). Usual compromise: **minimise hops**.
- **Least-cost:** cost per link, route cost = sum, pick the lowest. Cost 1 → fewest hops.
- **Static** (decided in advance) vs **adaptive** (changes with topology or load). In practice: adaptive and decentralised.
- **Distance-vector:** whole table → neighbours → every ~30 s. Table = network ID, cost (hops), next hop.
- **Update rules:** +1 · new → add · same next hop → always replace · different next hop → replace only if lower.
- Used in **RIP** (and a variation in **BGP**). Weakness: **count-to-infinity** (slow convergence after failures) → led to link-state.
