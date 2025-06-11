---
title: "About CCA Realms"
weight: 2

### FIXED, DO NOT MODIFY
layout: learningpathall
---

This section is a high level overview of RME and Realms.

## About RME and Realms

Arm’s *Realm Management Extension (RME)* is a key security feature introduced in
the Armv9-A architecture. It enables a new form of hardware-enforced isolation
designed to support _confidential computing_. It defines the set of hardware
features and properties that are required to comply with the Arm *Confidential
Computing Architecture (CCA)* architecture.

At the heart of RME is the concept of a *Realm*, a protected execution
environment that operates independently from the conventional *Normal World*
(used by operating systems and applications) and the *Secure World* (used by
trusted firmware or TEE). Realms are managed by a new privileged entity called
the *Realm Management Monitor (RMM)* and are enforced by the hardware via the
*Granule Protection Table (GPT)* and *Granule Transitioning* mechanism.

## CCA Realm

Realms allow lower-privileged software, such as an application or a virtual
machine, to protect its content and execution from attacks by higher-privileged
software, such as an OS or a hypervisor. Realms provide an environment for
confidential computing, without requiring the Realm owner to trust the software
components that manage the resources that the Realm uses.
