# **Relay Subsystem Architecture**
The relay subsystem provides a deterministic and hardware‑agnostic switching layer for the RF‑Sequencer.
Its purpose is to abstract the physical characteristics of RF relays while ensuring predictable behavior across different station topologies and relay technologies.

## **Architectural Role**
Within the overall system architecture, the relay subsystem forms the boundary between:

the state machine (which defines sequencing intent), and

the physical RF path (which enforces transmit/receive routing).

The subsystem guarantees that relay transitions occur in a controlled, deterministic manner, independent of relay type or hardware configuration.
This allows the protection engine, timing model, and SAFE‑mode logic to operate without assumptions about the underlying switching hardware.

## **Hardware Topology Abstraction**
Installations vary widely in how RF switching is implemented.
To accommodate this, the subsystem models two abstract topologies:

TX‑only topology
A single relay defines the RF path.
Typical in systems where the receive path is internally bypassed or externally switched.

TX+RX topology
Two independent relays define transmit and receive routing.
Common in systems with external LNAs, tower‑top equipment, or microwave front‑ends.

The firmware treats these as architectural declarations rather than operational modes.
Once the topology is known, all sequencing and protection logic derives from it automatically.

## **Relay Actuation Model**
RF installations may use different relay technologies.
The subsystem abstracts these differences through a unified actuation model supporting:

monostable relays (continuous‑drive)

latching relays (pulse‑driven state changes)

The abstraction ensures that:

timing guarantees remain identical

SAFE‑mode transitions behave consistently

the state machine does not depend on relay physics

the protection engine can assume deterministic switching latency

This separation allows the firmware to support a wide range of relay hardware without altering the sequencing logic.

## ** Deterministic Transition Semantics**
All relay transitions follow a deterministic sequence defined by the state machine.
The relay subsystem enforces:

ordered transitions

bounded switching latency

monotonic state progression

no ambiguous intermediate states

This ensures that the RF path is always in a well‑defined condition, even during rapid transitions or fault conditions.

## **Integration with Protection and SAFE‑Mode**
The relay subsystem is tightly integrated with the protection engine.
During fault conditions:

relay transitions are forced into a known‑safe configuration

switching is performed deterministically

no assumptions are made about previous relay states

This guarantees that the RF path is always driven to a safe configuration, regardless of the fault origin.

## **Design Goals**
The relay subsystem is designed around the following architectural goals:

Hardware independence  
The sequencing logic must not depend on relay type or topology.

Deterministic behavior  
All transitions must be predictable and bounded in time.

Safety under all conditions  
Faults, resets, and brown‑outs must result in a defined RF path.

Minimal configuration surface  
Only the hardware topology and relay type need to be declared; all other behavior is derived.

Long‑term maintainability  
The subsystem must remain stable even as new relay types or station architectures are introduced.

## **Summary**
The relay subsystem provides a stable architectural foundation for RF path control.
By abstracting hardware topology and relay technology, it enables the sequencer to deliver deterministic timing, robust protection behavior, and consistent SAFE‑mode semantics across a wide range of RF installations.

It is not a user‑facing component but an internal architectural layer that ensures the system behaves predictably regardless of the physical relays connected to it.