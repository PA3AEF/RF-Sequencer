---
title: Relay Subsystem Architecture
parent: Architecture
nav_order: 3
---

# Overview

# **Relay Subsystem Architecture**

The relay subsystem provides the abstraction layer between the firmware’s logical sequencing model and the physical RF switching hardware.  
Its purpose is to ensure deterministic, safe, and hardware‑independent relay behavior across all supported station configurations.

This document describes the relay subsystem at an architectural level.  
For the full firmware architecture, see:

- `firmware-architecture.md`

For a high‑level conceptual overview, see:

- `overview.md`

---

## **Purpose and Scope**

The relay subsystem defines how the firmware models, controls, and validates RF switching hardware.  
It isolates the state machine and protection engine from:

- relay topology  
- relay actuation type  
- electrical characteristics  
- timing requirements  
- installation‑specific wiring  

By abstracting these details, the subsystem ensures consistent sequencing behavior regardless of the underlying hardware.

---

## **Topology Model**

The subsystem supports two structural relay configurations:

### **Single‑Relay (TX‑Only)**
A single switching element defines the RF path.  
This topology is used in simple installations where receive path switching is not required.

### **Dual‑Relay (TX+RX)**
Independent transmit and receive paths are controlled by separate relays.  
This topology supports more complex station architectures and provides full control over both RF paths.

Topology is a **structural property** of the installation.  
Once declared, all sequencing and SAFE‑mode behavior is derived automatically.

---

## **Actuation Model**

Relays are modeled using an abstract actuation interface that supports:

- **Monostable relays**  
  Require continuous drive to maintain state.

- **Latching relays**  
  Use pulse‑driven actuation and retain state without continuous power.

The firmware does not expose electrical characteristics to higher layers.  
The state machine interacts only with **logical relay states**, while the subsystem handles the required actuation behavior.

---

## **Deterministic Transition Semantics**

All relay transitions follow strict architectural rules:

- transitions occur in a defined order  
- switching latency is bounded  
- intermediate states are never exposed  
- SAFE‑mode transitions override all other requests  
- relay actions are synchronized with the main execution loop  

These guarantees ensure that the RF path is always in a well‑defined state, even during rapid transitions or fault conditions.

---

## **SAFE‑Mode Behavior**

The relay subsystem integrates tightly with the protection engine.  
When a protection condition is triggered:

- the subsystem forces a known‑safe relay configuration  
- previous relay states are ignored  
- transitions occur deterministically  
- the subsystem remains in SAFE configuration until cleared  

SAFE‑mode behavior is derived from topology and actuation type, ensuring consistent protection across installations.

---

## **Interaction with the State Machine**

The state machine issues **logical relay requests** such as:

- “set TX path active”  
- “set RX path active”  
- “enter SAFE configuration”  

The relay subsystem translates these requests into hardware‑specific actions while enforcing:

- timing constraints  
- topology rules  
- protection overrides  
- deterministic sequencing  

The state machine never interacts with relay hardware directly.

---

## **Interaction with the Protection Engine**

The protection engine may override relay requests at any time.  
When this occurs:

- the subsystem immediately transitions to SAFE configuration  
- pending relay actions are discarded  
- the state machine is informed through protection events  

This ensures that relay behavior always reflects the current safety requirements.

---

## **Configuration Parameters**

The relay subsystem relies on configuration values that define:

- relay topology  
- relay type (monostable or latching)  
- actuation timing  
- SAFE‑mode configuration  

These parameters are validated at startup.  
Invalid configuration forces the system into SAFE mode until corrected.

---

## **Design Objectives**

The relay subsystem is designed to:

- provide a stable abstraction for all relay hardware  
- ensure deterministic switching behavior  
- maintain safety under all conditions  
- minimize configuration complexity  
- support future relay types without architectural changes  

Its design allows the firmware to evolve while preserving predictable RF path control.

---

## **Further Reading**

- **Firmware Architecture**  
  `firmware-architecture.md`

- **Architecture Overview**  
  `overview.md`

- **Integration Examples**  
  `integration-examples.md`

