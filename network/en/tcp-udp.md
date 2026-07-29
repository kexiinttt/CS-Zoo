# TCP vs. UDP

| | TCP (Transmission Control Protocol) | UDP (User Datagram Protocol) |
|---|---|---|
| Connection | Connection-oriented; requires a three-way handshake | Connectionless; sends directly |
| Reliability | Reliable; retransmits lost packets | Unreliable; does not retransmit lost packets |
| Ordering | Guarantees that data arrives in order | Does not guarantee ordering |
| Data model | Byte stream | Datagram |
| Message boundaries | Does not preserve message boundaries | Preserves message boundaries |
| Flow control | Yes | No |
| Congestion control | Yes | No |
| Overhead | Higher | Lower |
| Latency | Relatively higher | Relatively lower |
| Speed | Relatively slower but stable | Fast but may lose packets |
| Typical problems | Head-of-line blocking | Packet loss, reordering, and duplicate packets |
| Suitable scenarios | Data must be complete, correct, and ordered | Low latency is important and a small amount of loss is acceptable |

---

# Flow Control

The sender cannot transmit too quickly, or the receiver's buffer may be unable to process the data.

In TCP, the receiver tells the sender how much data it can currently accept: the **receive window (`rwnd`)**.

---

# Congestion Control

Even if the receiver is powerful enough, routers, switches, and links in the network may not be able to handle too much traffic. This causes packet loss and increased latency.

TCP dynamically adjusts its sending speed according to network conditions using the **congestion window (`cwnd`)**.

> If the network appears smooth, TCP gradually increases the sending volume. If packet loss or a timeout occurs, TCP assumes the network is congested and reduces the sending volume.
