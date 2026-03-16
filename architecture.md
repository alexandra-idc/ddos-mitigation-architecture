# High-Performance Layer 7 DDoS Mitigation Architecture

## 1. Overview

This document describes the architecture of a high-performance Layer 7 DDoS mitigation system designed to handle high packet-per-second (PPS) attacks.

The system combines upstream anti-DDoS protection with early packet filtering using XDP/eBPF, kernel-level controls, firewall rules, and reverse proxy filtering to protect backend services from volumetric and application-layer attacks.

---

## 2. Problem Statement

High PPS attacks can overwhelm traditional firewall solutions because packet filtering often occurs too late in the networking stack.

During volumetric attacks this may lead to:

- excessive CPU usage
- connection tracking exhaustion
- SYN backlog saturation
- socket buffer pressure
- application layer overload

To address these issues, packet filtering must occur as early as possible in the packet processing pipeline.

---

## 3. Design Goals

The system is designed with the following objectives:

- Drop malicious packets as early as possible
- Reduce kernel and userspace resource consumption
- Protect application layer services
- Handle high packet-per-second attacks
- Maintain availability for legitimate users
- Use layered mitigation strategies

---

## 4. High-Level Architecture

The mitigation system is deployed on a dedicated bare metal server to maximize packet processing performance and avoid virtualization overhead.

Incoming traffic passes through multiple filtering layers designed to progressively eliminate malicious traffic.

### Architecture Overview

Internet  
↓  
OVH Edge Anti-DDoS  
↓  
Network Interface (NIC)  
↓  
XDP / eBPF Packet Filter  
↓  
Linux Kernel Networking Stack  
 ├ TCP/IP stack  
 ├ SYN backlog  
 └ socket buffers  
↓  
Netfilter Firewall  
 └ nftables  
↓  
Userspace Reverse Proxy  
 ├ HAProxy  
 └ Nginx  
↓  
Origin Backend Services

---

## 5. Layer Description

### OVH Edge Anti-DDoS

Traffic first passes through the upstream anti-DDoS filtering infrastructure provided by OVH.  
This layer mitigates large volumetric attacks before traffic reaches the mitigation server.

### Network Interface (NIC)

Packets arrive at the physical network interface of the bare metal server.

### XDP / eBPF Filtering

An XDP program attached to the NIC performs early packet inspection.

Malicious packets can be dropped before:

- skb allocation
- kernel networking processing
- connection tracking

This significantly reduces CPU overhead during packet floods.

### Linux Kernel Networking Stack

Packets that pass the XDP filter enter the kernel networking stack where TCP/IP processing occurs.

Key kernel resources that must be protected include:

- TCP connection handling
- SYN backlog queues
- socket buffers

Proper tuning of these components helps prevent resource exhaustion during attacks.

### Netfilter / nftables

Packets then pass through the Netfilter framework where nftables rules enforce additional firewall policies.

Examples include:

- packet filtering
- rate limiting
- IP blocking
- protocol validation

### Userspace Reverse Proxy

Application layer filtering occurs in userspace through reverse proxies such as HAProxy or Nginx.

These proxies handle:

- HTTP connection management
- request forwarding
- additional rate limiting
- protection against HTTP floods

### Origin Services

Only validated and filtered requests are forwarded to backend application services.

---

## 6. Packet Processing Flow

1. Traffic arrives from the internet
2. Upstream filtering occurs at the OVH edge anti-DDoS infrastructure
3. Packets reach the mitigation server NIC
4. The XDP program performs early packet inspection
5. Malicious packets are dropped before entering the kernel networking stack
6. Legitimate packets proceed through the Linux TCP/IP stack
7. Netfilter applies firewall rules via nftables
8. Reverse proxy services handle HTTP requests
9. Valid requests are forwarded to backend origin services

---

## 7. Design Rationale

This architecture uses multiple filtering layers to mitigate attacks at different stages of packet processing.

Early filtering using XDP reduces CPU overhead by preventing unnecessary kernel processing.

Kernel-level controls protect core networking resources such as connection queues and socket buffers.

Firewall rules enforce additional traffic policies, while reverse proxies handle application-layer filtering and request management.

This layered approach significantly improves resilience against high PPS and Layer 7 attacks.

---

## 8. Future Implementation

Future work will include:

- implementation of the XDP filtering program
- kernel parameter tuning
- nftables rule optimization
- benchmarking under simulated attack traffic
- observability using metrics and monitoring systems
- performance testing with high PPS workloads

[Packet Processing Pipeline](packet-processing.md)
