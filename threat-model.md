
[← Back to project overview](README.md)

# Threat Model

This document describes the attack scenarios considered when designing the mitigation architecture.

The goal of this threat model is to identify attack vectors that can exhaust network, kernel, or application resources and to define the mitigation layers responsible for handling them.

The system is primarily designed to protect web services against high packet-per-second attacks and Layer 7 resource exhaustion attacks.

---

# 1. Assets to Protect

The mitigation architecture aims to protect the following critical resources:

- Network interface capacity
- CPU resources
- Kernel networking resources
- Connection handling capacity
- Reverse proxy resources
- Backend application services

Exhaustion of any of these components can lead to service degradation or downtime.

---

# 2. Attack Surface

The exposed attack surface includes:

- Public HTTP/HTTPS services
- TCP connection handling
- Kernel networking stack
- Reverse proxy infrastructure

Attackers may attempt to exploit weaknesses in packet processing or resource management to overwhelm the system.

---

# 3. Attack Scenarios

## 3.1 SYN Flood

### Description

Attackers generate large numbers of TCP SYN packets in order to exhaust the SYN backlog queue.

This can prevent legitimate users from establishing new connections.

### Targeted Resources

- SYN backlog queue
- Kernel TCP connection handling
- CPU resources

### Mitigation Layers

Mitigation occurs through multiple layers:

- upstream anti-DDoS filtering
- early packet filtering using XDP
- kernel TCP tuning
- firewall filtering rules

---

## 3.2 High PPS Packet Flood

### Description

Attackers generate extremely high packet-per-second traffic volumes designed to overwhelm CPU packet processing.

These attacks may include:

- UDP floods
- TCP floods
- random packet floods

### Targeted Resources

- CPU packet processing
- network interrupt handling
- kernel networking stack

### Mitigation Layers

- upstream filtering
- XDP early packet drop
- firewall filtering

XDP plays a critical role by dropping malicious packets before they enter the kernel networking stack.

---

## 3.3 HTTP Flood

### Description

Attackers generate large volumes of HTTP requests designed to exhaust application layer resources.

This attack targets the web service itself rather than the network stack.

### Targeted Resources

- reverse proxy connections
- backend application servers
- worker processes

### Mitigation Layers

- reverse proxy filtering
- connection rate limiting
- request validation

Reverse proxy systems such as HAProxy or Nginx help absorb and filter these requests before they reach backend services.

---

## 3.4 Connection Exhaustion

### Description

Attackers attempt to open large numbers of simultaneous TCP connections in order to exhaust available connection slots.

### Targeted Resources

- connection tracking tables
- socket buffers
- reverse proxy connection pools

### Mitigation Layers

- kernel connection limits
- firewall filtering
- reverse proxy limits

---

## 3.5 Malformed Packet Attacks

### Description

Malformed or intentionally invalid packets may be used to trigger inefficient processing in the networking stack.

These packets may attempt to exploit protocol parsing weaknesses or increase CPU overhead.

### Targeted Resources

- packet parsing logic
- kernel networking stack
- firewall processing

### Mitigation Layers

- XDP validation
- firewall filtering
- protocol validation

---

# 4. Layered Mitigation Strategy

The architecture uses a layered mitigation approach where each layer filters malicious traffic before it reaches deeper parts of the system.

Filtering layers include:

1. Upstream network filtering
2. Early packet filtering using XDP
3. Kernel networking resource protection
4. Firewall filtering via nftables
5. Reverse proxy filtering

This layered strategy significantly reduces the probability that malicious traffic reaches backend services.

---

# 5. Assumptions

The threat model assumes:

- attackers may generate very high packet-per-second traffic
- attacks may originate from distributed sources
- attackers may target multiple layers simultaneously

The architecture is designed to remain resilient under these conditions.

---

# 6. Limitations

This architecture focuses primarily on:

- high PPS network attacks
- connection exhaustion
- HTTP flood scenarios

More advanced application-layer attacks such as complex bot behavior may require additional detection mechanisms.

---

# 7. Summary

The threat model defines the attack scenarios that the mitigation architecture must handle.

By identifying critical resources and potential attack vectors, the system can apply layered filtering techniques to protect network infrastructure and backend services from resource exhaustion attacks.
