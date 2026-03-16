# Kernel Networking Tuning

This document describes kernel-level tuning considerations for supporting high packet-per-second workloads in the mitigation architecture.

High PPS attacks can stress multiple kernel networking components including connection queues, socket buffers, and network processing paths.

Proper kernel tuning helps maintain system stability and prevents resource exhaustion during attack scenarios.

---

## 1. SYN Backlog Configuration

The SYN backlog queue stores incoming TCP connection requests before they are fully established.

During SYN flood attacks this queue can become saturated.

Increasing backlog limits helps maintain availability during large bursts of connection attempts.

Key parameters typically tuned include:

- tcp_max_syn_backlog
- somaxconn

---

## 2. Socket Buffer Limits

Socket buffers temporarily store packets before they are processed by userspace applications.

If buffers are too small, packets may be dropped during traffic bursts.

Increasing buffer limits can improve resilience during high PPS workloads.

Relevant parameters include:

- rmem_max
- wmem_max
- netdev_max_backlog

---

## 3. Network Queue Handling

High PPS traffic requires efficient packet queue handling inside the kernel.

Network queues must be able to absorb traffic bursts without dropping legitimate packets.

Proper tuning of queue sizes helps maintain packet processing stability.

---

## 4. Connection Tracking Limits

The Netfilter connection tracking table keeps track of active network connections.

Under large traffic volumes this table may become exhausted.

Increasing connection tracking limits helps prevent legitimate traffic from being dropped.

Relevant parameters include:

- nf_conntrack_max
- nf_conntrack_buckets

---

## 5. TCP Behavior Adjustments

Additional TCP parameters can improve system behavior under heavy connection load.

Examples include tuning retransmission handling and connection timeouts.

These adjustments help the system recover faster during attack conditions.

---

## 6. Summary

Kernel tuning plays a critical role in supporting high packet-per-second workloads.

Combined with early packet filtering and firewall rules, proper kernel configuration significantly improves the resilience of the mitigation system.
