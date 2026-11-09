# Computer Networking: Simple Lessons with Quizzes

Short, beginner-friendly lessons covering the **Basics**, **Internet** and **Data Link Errors** lecture decks. Each lesson explains one topic in plain language, with diagrams, worked examples and a quiz at the end.

> **How the quizzes work:** each answer is hidden inside a collapsible box. Try the question first, then click **"Show answer"** to check yourself.

## Lessons

Work through them in order. Each one builds on the last.

| # | Lesson | What you'll learn |
|---|---|---|
| 1 | [Network performance and delays](lessons/01-delays.md) | Bandwidth, transmission delay, propagation delay, RTT, bandwidth-delay product |
| 2 | [Switching and multiplexing](lessons/02-switching.md) | Circuit vs packet switching, datagram vs virtual circuit, store-and-forward, pipelining, statistical multiplexing |
| 3 | [Layers and encapsulation](lessons/03-layers.md) | OSI (7 layers), TCP/IP (4 layers), SDU / PCI / PDU |
| 4 | [Transport: UDP and TCP](lessons/04-transport.md) | Ports, UDP, TCP, MSS, the windows |
| 5 | [TCP congestion control](lessons/05-tcp-congestion.md) | Slow start, threshold, linear growth, timeouts, throughput |
| 6 | [IPv4 addressing](lessons/06-ipv4-addressing.md) | Binary, classes, special addresses, DNS / ARP, who assigns addresses |
| 7 | [Netmasks, subnetting and CIDR](lessons/07-subnetting.md) | ANDing, the 5-step method, broadcast, valid hosts, counting hosts and subnets |
| 8 | [Exam revision](lessons/08-exam-revision.md) | Every formula on one page, common traps, and a mixed final quiz |
| 9 | [Physical layer and bit errors](lessons/09-physical-layer-bit-errors.md) | Noise, transmission modes, FDM / TDM / statistical multiplexing, frame error probability |
| 10 | [Error detection and correction](lessons/10-error-detection-correction.md) | Repeating frames, single and 2-D parity, Hamming distance, the Hamming code |
| 11 | [CRC and framing](lessons/11-crc-and-framing.md) | Cyclic Redundancy Check, generator polynomials, flags, bit stuffing |

## Study tips

1. **One lesson at a time.** Don't move on until the quiz feels easy.
2. **Write the working out.** Most marks come from method: tables for TCP, the 5 steps for subnetting.
3. **Redo the quizzes** a few days later without looking at the answers.
4. **Check the units:** bits vs bytes, ms vs µs. This is the most common way to lose marks.

## About

Explanations are based on the course lecture slides ("2025-1 Basics", "2023-2 Internet" and "2023-3 Data Link Errors") and the course quiz and test solutions. Anything that goes beyond the slides is marked **Beyond the notes**.

## Printable resources

| File | What it is |
|---|---|
| [Networking_Cheat_Sheet.pdf](resources/Networking_Cheat_Sheet.pdf) | 2-page cheat sheet: formulas, plus definitions and true/false facts |
| [Networking_Practice_Set.pdf](resources/Networking_Practice_Set.pdf) | 18 mixed practice questions with an answer key |
| [IP_Addressing_Practice.pdf](resources/IP_Addressing_Practice.pdf) | 25 addressing and subnetting questions with an answer key |
| [Hops_and_Pipelining.pdf](resources/Hops_and_Pipelining.pdf) | Store-and-forward and pipelining, with diagrams |
| [Valid_Host_Q14.pdf](resources/Valid_Host_Q14.pdf) | How to tell a valid host from a subnet or broadcast address |
| [Errors_1_Worked_Solutions.pdf](resources/Errors_1_Worked_Solutions.pdf) | Problem Set Errors 1: frame and packet error probability |
| [Errors_2_Worked_Solutions.pdf](resources/Errors_2_Worked_Solutions.pdf) | Problem Set Errors 2: Hamming distance, CRC, bit stuffing, Hamming code |
