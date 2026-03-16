# Packet Processing Pipeline

This document describes the packet processing flow inside the mitigation system.

The goal is to filter malicious traffic as early as possible while allowing legitimate traffic to reach backend services.

---

## 1. Traffic Arrival

Packets arrive from the internet and first pass through the upstream anti-DDoS infrastructure provided by OVH.

This upstream layer absorbs large volumetric attacks before traffic reaches the mitigation server.

After upstream filtering, traffic arrives at the mitigation server network interface.

---

## 2. NIC Reception

Packets are received by the Network Interface Card (NIC).

At this stage the packet has not yet entered the Linux networking stack.

The NIC forwards the packet to the driver where XDP programs can be attached.

---

## 3. XDP Processing

The XDP (eXpress Data Path) program executes directly in the network driver.

This stage allows packet filtering before:

- skb allocation
- kernel networking stack processing
- connection tracking

Typical operations performed in this stage include:

- basic packet validation
- IP filtering
- early packet drops for suspicious traffic
- PPS flood mitigation

Packets identified as malicious can be dropped immediately.

Legitimate packets continue to the next stage.

---

## 4. Kernel Networking Stack

Packets that pass the XDP filter enter the Linux kernel networking stack.

To support high packet-per-second workloads, several kernel networking components must be carefully tuned to avoid resource exhaustion during attacks.

Key components involved include:

### TCP/IP Stack

The TCP/IP stack is responsible for protocol parsing and connection handling.

Kernel parameters can be tuned to improve resilience under high connection rates and large traffic volumes.

### SYN Backlog

Incoming TCP connection requests are temporarily stored in the SYN backlog queue.

During SYN flood attacks this queue may become saturated.

Proper tuning of backlog limits helps maintain availability under high connection rates.

### Socket Buffers

Socket buffers store incoming packets before they are processed by userspace applications.

Adjusting buffer sizes can improve system stability during traffic bursts and high PPS scenarios.

---

## 5. Netfilter Firewall

Packets then pass through the Netfilter framework where firewall rules are applied.

In this architecture, firewall rules are implemented using nftables.

Typical filtering actions include:

- IP filtering
- rate limiting
- protocol validation
- blocking known malicious sources

---

## 6. Userspace Reverse Proxy

Packets that pass firewall filtering reach userspace services.

Reverse proxies such as HAProxy or Nginx handle application-layer traffic.

These proxies provide:

- HTTP connection handling
- request routing
- rate limiting
- additional filtering against Layer 7 attacks

---

## 7. Backend Origin Services

Only validated and filtered traffic is forwarded to backend origin services.

At this stage most malicious traffic has already been filtered by previous layers.

This protects application servers from resource exhaustion.

---

## 8. Summary

The packet processing pipeline applies layered filtering:

1. Upstream anti-DDoS filtering
2. Early packet drop using XDP
3. Kernel networking controls
4. Netfilter firewall filtering
5. Reverse proxy application filtering

This layered approach significantly improves resilience against high PPS and Layer 7 attacks.

[Kernel Networking Tuning](kernel-tuning.md)
