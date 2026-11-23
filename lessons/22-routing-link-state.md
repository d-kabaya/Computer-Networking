# Lesson 22: Link-State Routing and Dijkstra's Algorithm

[← Distance-vector routing](21-routing-distance-vector.md) · [Index](../README.md) · [Next: Bridges and repeaters →](23-bridges-repeaters.md)

*Based on the "Internetworking and Routing" lecture deck, slides 15–28.*

## The big idea

Distance-vector routers only ever hear **second-hand** information from their neighbours, which is why bad news travels slowly (count-to-infinity).

Link-state flips it round: **every router tells every other router about its own neighbourhood**. Then every router has a **complete map** of the network and works out the best routes **itself**, using **Dijkstra's shortest-path algorithm**.

> **Analogy:** distance-vector is asking for directions at each junction and trusting the answer. Link-state is every town publishing its own local roads, so everyone can assemble the whole map and plan their own route with a sat-nav.

It happens in four stages:

```
1. Find out about your neighbours  (ECHO packets)
2. Tell everyone                   (flood a link-state packet)
3. Build the full map              (the link-state database)
4. Work out your routes            (Dijkstra → shortest-path tree → routing table)
```

---

## 1. Link costs (slide 16)

- A link's cost is usually a **weighted sum** of several factors, e.g. traffic level, security level, packet delay.
- **The cost is from a router to a network.** Going from a **network to a router costs 0**.
  - Why? A LAN is usually a **broadcast** network: every node on it, including the router, receives the packet anyway, so there is no extra cost to "reach" the router from the network.

The example network (the same as Lesson 21, now with costs on the router → network links):

| Router | Costs to its networks |
|---|---|
| A | Net 14: **1** · Net 78: **3** · Net 23: **2** |
| B | Net 14: **4** · Net 55: **2** |
| C | Net 55: **5** · Net 66: **2** |
| D | Net 66: **5** · Net 08: **3** |
| E | Net 23: **3** · Net 08: **2** |
| F | Net 78: **2** · Net 92: **3** |

**Watch out:** the same network can have a **different cost from each side**. Net 14 costs 1 from A, but 4 from B.

---

## 2. Link-state packets and flooding (slides 15, 17)

**Stage 1: learn your neighbourhood.** A router sends short **ECHO** packets to its neighbours and watches the replies. A missing reply means something has **changed** (e.g. a neighbour is down).

**Stage 2: advertise it.** Each router builds a **link-state packet** and **floods** it to **every** router in the network:

| Field | Meaning |
|---|---|
| **Advertiser** | ID of the router sending it |
| **Network** | ID of the destination network |
| **Cost** | Link cost to that neighbour |
| **Neighbour** | ID of the neighbour router across that network |

For example, router A's link-state packet:

| Advertiser | Network | Cost | Neighbour |
|---|---|---|---|
| A | 14 | 1 | B |
| A | 78 | 3 | F |
| A | 23 | 2 | E |

---

## 3. The link-state database (slide 18)

Once a router has every router's link-state packet, it puts them together into the **link-state database**: a complete map of the network.

| Advertiser | Network | Cost | Neighbour |
|---|---|---|---|
| A | 14 | 1 | B |
| A | 78 | 3 | F |
| A | 23 | 2 | E |
| B | 14 | 4 | A |
| B | 55 | 2 | C |
| C | 55 | 5 | B |
| C | 66 | 2 | D |
| D | 66 | 5 | C |
| D | 08 | 3 | E |
| E | 23 | 3 | A |
| E | 08 | 2 | D |
| F | 78 | 2 | A |
| F | 92 | 3 | — |

If every router receives the same set of packets (as if they were synchronised), **every router builds the same database**. Each router then **calculates its own routing table** from it.

---

## 4. Dijkstra's shortest-path algorithm (slides 19–20)

Dijkstra builds a **shortest-path spanning tree** for the router running it:
- The router is the **root** of its tree.
- The tree has a route to **every** destination and **no loops**.
- Even with identical databases, **each router's tree is different**, because each has a different root.

**Nodes** are networks or routers, and **arcs** are the links between them. Arc costs: router → network = the cost in the database; network → router = 0.

### The rules

Keep two sets: **Temporary** (candidates) and **Permanent** (final, best route found).

1. **Start:** only the root router is **Permanent**. Its neighbour nodes, and the arcs to them, are **Temporary**.
2. **Pick** the Temporary node with the **lowest cumulative cost from the root**. Make it (and its arc) **Permanent**.
3. **Add** any nodes connected to the new Permanent node that aren't already Temporary, with their arcs, to the Temporary set.
4. **Improve:** if a node already Temporary can be reached **more cheaply through the new Permanent node**, the new route **replaces** the old one.
5. **Repeat** from step 2 until **everything is Permanent**.

> **Key point:** each arc is labelled with the **cumulative cost from the root**, not the cost of that one link.

> **Analogy:** water spreading out from a source. It always reaches the nearest point next. Once a point is wet (Permanent), nothing can reach it any faster.

### Worked example: router A (slides 21–26)

| Step | Made Permanent (cumulative cost) | Added to Temporary | Notes |
|---|---|---|---|
| 1 | **A** (0, the root) | 14 (1), 78 (3), 23 (2) | A's neighbours |
| 2 | **14** (1) | B (1 + 0 = 1) | Network → router costs 0 |
| 3 | **B** (1) | 55 (1 + 2 = 3) | |
| 4 | **23** (2) | E (2) | |
| 5 | **E** (2) | 08 (2 + 2 = 4) | |
| 6 | **78** (3) | F (3) | 78 and 55 **tie** at 3, so one is chosen **arbitrarily** |
| 7 | **F** (3) | 92 (3 + 3 = 6) | |
| 8 | **55** (3) | C (3) | |
| 9 | **C** (3) | 66 (3 + 2 = 5) | |
| 10 | **08** (4) | D (4) | |
| 11 | **D** (4) | 66 via D would cost 4 + 5 = **9** | 9 > 5, so the new arc is **deleted**; 66 keeps its route via C |
| 12 | **66** (5) | — | |
| 13 | **92** (6) | — | Everything Permanent → **STOP** |

**Step 11 is the important one:** D offers a second route to Net 66, but it costs 9 against the 5 already found via C. If it had been **lower**, it would have **replaced** the old route (rule 4).

### The finished tree for A

```
A ─1─ 14 ─0─ B ─2─ 55 ─0─ C ─2─ 66          (cumulative: 14=1, B=1, 55=3, C=3, 66=5)
│
├─3─ 78 ─0─ F ─3─ 92                         (78=3, F=3, 92=6)
│
└─2─ 23 ─0─ E ─2─ 08 ─0─ D                   (23=2, E=2, 08=4, D=4)
```

---

## 5. From tree to routing table (slide 27)

Read the tree: for each network, the **cost** is its cumulative cost, and the **next router** is the **first router** on the path from A.

**Router A's link-state routing table:**

| Net | Cost | Next router |
|---|---|---|
| 08 | 4 | E |
| 14 | 1 | — |
| 23 | 2 | — |
| 55 | 3 | B |
| 66 | 5 | B |
| 78 | 3 | — |
| 92 | 6 | F |

- **14, 23 and 78** have no next router because they are **directly connected** to A.
- **66's next router is B, not C.** C is the last router on the path, but A only needs to know where to send the packet **first**: A → 14 → **B** → 55 → C → 66.
- **Each router's table is (in general) different.**

### Practical problems (slide 27)

- In **large networks**, the **memory** for the database and the **computation time** for the table can be significant.
- In practice, link-state packets **don't all arrive at the same time**, so routers may be working from **different databases** for a while. How wrong the routes are depends on how different the routers' "views" of the network are.

---

## 6. Link-state in practice (slide 28)

**Advantages:** **rapid convergence**, a **small amount of traffic** generated, and a **rapid response to topology changes**.

**Internet examples:** **OSPF** (Open Shortest Path First) and **IS-IS** (Intermediate System to Intermediate System).

Possible **OSPF link costs** (configurable):
- 1 for each link
- **Reliability**: set by the administrator, showing how often the link fails
- Packet delay
- Link bandwidth
- Financial cost of the link

**OSPF's drawbacks:**
- It needs **a lot of memory**: each router holds its routing table **and** the whole link-state database.
- **Dijkstra is processor-intensive.** Older routers may be unable to relay packets while calculating, which could be every time a link-state packet arrives.
- It can use **a lot of bandwidth** if the topology changes often.
- Link-state packets are sent to all routers using **reliable flooding**, which needs a **sequence number** and a **time-to-live (TTL)** field in each packet.

> *Beyond the notes, why these fields?* The **sequence number** lets a router tell a newer packet from an older copy, so it ignores stale information. The **TTL** stops a flooded packet circulating forever.

### Distance-vector vs link-state: the full comparison

| | Distance-vector | Link-state |
|---|---|---|
| Sends | Whole routing table | Own neighbourhood only |
| To | Neighbours | All routers (flooding) |
| When | Regular intervals (~30 s) | When something changes |
| Each router knows | Only "distance and direction" to each network | The **whole map** |
| Algorithm | Distributed **Bellman-Ford** | **Dijkstra** |
| Convergence | Slow (**count-to-infinity**) | **Fast** |
| Memory and CPU | Low | **High** (database plus Dijkstra) |
| Internet examples | **RIP** (and BGP, a variation) | **OSPF**, **IS-IS** |

---

## Quiz

**Q1.** Why is the cost from a network to a router 0?

<details><summary>Show answer</summary>

A LAN is usually a **broadcast** network, so every node on it, including the router, receives the packet anyway. There is no extra cost.
</details>

**Q2.** What four fields are in a link-state packet?

<details><summary>Show answer</summary>

**Advertiser** (sending router) · **Network** (destination network) · **Cost** (link cost to the neighbour) · **Neighbour** (neighbour router)
</details>

**Q3.** How does a router find out about its own neighbourhood?

<details><summary>Show answer</summary>

It sends short **ECHO** packets to its neighbours and monitors the responses.
</details>

**Q4.** If all routers have the same link-state database, do they all build the same routing table? Why?

<details><summary>Show answer</summary>

**No.** Each router runs Dijkstra with **itself as the root**, so each builds a different shortest-path tree, and so a different routing table.
</details>

**Q5.** In A's routing table, why is the next router for Net 66 **B** and not C?

<details><summary>Show answer</summary>

The next router is the **first router** on the path: A → 14 → **B** → 55 → C → 66. A only needs to know where to send the packet first.
</details>

**Q6.** In step 11 of the example, D offers a route to Net 66. Why isn't it used?

<details><summary>Show answer</summary>

Its cumulative cost would be 4 + 5 = **9**, but Net 66 is already Temporary at cost **5** via C. The new route only replaces the old one if it is **cheaper**.
</details>

**Q7.** Using the link-state database from section 3, run Dijkstra for **router E** and write E's routing table.

<details><summary>Show answer</summary>

Order made Permanent (cumulative cost): E 0 · 08 2 · D 2 · 23 3 · A 3 · 14 4 · B 4 · then 55, 78, C and F all at 6 · 66 7 · 92 9

- Net 66: via D = 2 + 5 = **7**; via C = 6 + 2 = 8. So 7, next router D.

| Net | Cost | Next router |
|---|---|---|
| 08 | 2 | — |
| 14 | 4 | A |
| 23 | 3 | — |
| 55 | 6 | A |
| 66 | 7 | D |
| 78 | 6 | A |
| 92 | 9 | A |
</details>

**Q8.** Give three drawbacks of OSPF.

<details><summary>Show answer</summary>

Any three of: needs **a lot of memory** (routing table plus database) · Dijkstra is **processor-intensive** (old routers may stop relaying while calculating) · uses **a lot of bandwidth** if the topology changes often · flooding needs **sequence numbers and TTLs**.
</details>

**Q9.** Why does link-state converge faster than distance-vector?

<details><summary>Show answer</summary>

Every router hears about a change **directly** from the router where it happened (by flooding) and recalculates from a **complete map**. Distance-vector routers only get **second-hand**, possibly out-of-date, information from their neighbours.
</details>

---

## What to remember

- **Link-state:** each router tells **all** routers about its **neighbourhood**, when there's a **change**.
- **Costs:** router → network = the link cost; **network → router = 0**.
- **ECHO** packets find neighbours. **Link-state packets** (advertiser, network, cost, neighbour) are **flooded**.
- All packets → **link-state database** (the same at every router, ideally).
- **Dijkstra:** Temporary and Permanent sets · pick the **lowest cumulative cost** · add its neighbours · replace a route only if **cheaper** · stop when all are Permanent.
- **Routing table:** cost = cumulative cost; next router = the **first** router on the path; "—" for directly connected networks.
- **Pros:** fast convergence, little traffic, quick response. **Cons:** memory, CPU, bandwidth when the topology changes often.
- **OSPF** and **IS-IS** are link-state protocols. Flooding needs **sequence numbers and TTL**.
