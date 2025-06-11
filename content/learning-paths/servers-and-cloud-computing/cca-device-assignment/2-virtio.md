---
title: "Device attach (part 1): virtio"
weight: 2

### FIXED, DO NOT MODIFY
layout: learningpathall
---

This section is a high level overview of VirtIO and Bounce Buffers, and how they
relate to CCA Realms.

## VirtIO

### What is VirtIO ?

VirtIO is an abstraction layer for virtual devices in virtualized environments.
It provides standardized, efficient interfaces between guest virtual machines
(VMs) and host devices, making it easier to develop _paravirtualized_ drivers.
_Paravirtualized_ means that the guest OS is aware it’s running in a virtualized
environment and can use optimized drivers (VirtIO) to communicate with virtual
hardware. Emulating hardware devices (like NICs or disks) for VMs is slow and
inefficient. VirtIO provides a standardized, efficient interface that allows VMs
to bypass full device emulation and instead use optimized drivers.

VirtIO is most commonly used with KVM/QEMU virtualization, example drivers are:
- `virtio-net`: Paravirtualized networking
- `virtio-blk`: Block device (disk) access
- `virtio-fs`: File sharing (host ↔ guest)
- `virtio-balloon`: Dynamic memory management
- `virtio-rng`: Random number source
- `virtio-console`: Simple console interface
- ...

### How VirtIO works in VMs

1. The Host Hypervisor (e.g., QEMU/KVM) exposes VirtIO “backend” devices.
2. The guest OS loads VirtIO _frontend_ drivers (e.g., `virtio_net`,
   `virtio_blk`) that speak the VirtIO protocol.
3. Communication happens via shared memory (`virtqueues`) for I/O operations,
   avoiding full device emulation.
4. Devices are exposed over the PCI or MMIO bus to the guest.

For example, instead of emulating an Intel e1000 NIC, the host exposes a
`virtio-net` interface to the guest OS and the guest OS uses the `virtio-net`
driver to send/receive packets via shared buffers.

## Bounce buffers

### What are bounce buffers?

Bounce buffers are temporary memory buffers used in the Linux kernel to handle
situations where direct memory access (DMA) can’t be performed directly on the
original data buffer. This often happens because:
1. The original buffer is not physically contiguous.
2. The buffer is in high memory or not accessible to the device.
3. The buffer doesn’t meet alignment or boundary requirements of the device.

### Why _Bounce_ buffers?

Data _bounces_ between:
- The original buffer (in user/kernel space) and
- The DMA-capable bounce buffer (used for I/O with the device)

This ensures that data transfers can still happen even when the original memory
is not suitable or accessible for transfers.

## CCA Realms, VirtIO and bounce buffers

The defining feature of a Realm is that its memory (called *Realm memory*) is
cryptographically isolated from both the Normal and Secure Worlds. This means
that:
- Realm memory is encrypted using keys that are unique to each Realm.
- Non-Realm entities (like the host OS or hypervisor) cannot directly read or
  write Realm memory.
- Even Direct Memory Access (DMA) from peripherals or untrusted drivers cannot
  access Realm data.

This design ensures confidentiality but introduces a problem: *how can Realms
interact with untrusted components*, such as:
- Network stacks in the host OS,
- Storage subsystems,
- I/O devices managed by untrusted drivers?

The solution to safely exchange data between a Realm and the outside World is to
use bounce buffers as an intermediary !

### How bounce buffers are used with RME

1. Exporting Data:
   - A Realm application prepares some data (e.g., results of computation).
   - It copies this data from protected Realm memory into a bounce buffer.
   - The Realm notifies the untrusted host or hypervisor that the data is ready.
   - The host retrieves the data from the bounce buffer.

2. Importing Data:
   - The host places data (e.g., input from a file or device) into a bounce buffer.
   - The Realm is notified and validates the source.
   - The Realm copies the data from the bounce buffer into its protected memory.

This pattern preserves confidentiality and integrity of Realm data, since:
- The Realm never allows direct access to its memory.
- It can validate and sanitize any data received via bounce buffers.
- No sensitive data is exposed without explicit copying.

### Confidentiality preserved with bounce buffers, really?

In the previous section, it was mentioned that _bounce buffers preserves
confidentiality_, a sentence which deserves a bit more thoughts. Bounce buffers
are nothing more than an explicitly shared temporary area between the Realm
world and the outside world. This does indeed preserve the confidentiality of
all the rest of the Realm data. On the other hand, for the data being
transferred, it is leaving the Realm world and will only stay confidential if it
is encrypted in some ways, e.g. for network traffic, TLS should be used.

### What now ?



### Seeing a Realm's bounce buffers at work
