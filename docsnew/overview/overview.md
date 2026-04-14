# **RF‑Sequencer Architecture Overview**

The RF‑Sequencer provides a deterministic control layer for managing RF path transitions, amplifier protection, and timing‑critical sequencing in radio systems.  
Its architecture is designed around clarity, predictability, and hardware independence, ensuring consistent behavior across a wide range of station configurations.

This document introduces the major architectural components and explains how they interact at a conceptual level.  
For detailed subsystem specifications, see:

- **Firmware Architecture**  
  `firmware-architecture.md`

- **Relay Subsystem Architecture**  
  `relay-architecture.md`

---

## **System Purpose**

The sequencer coordinates RF transmit and receive transitions while enforcing strict protection rules.  
Its primary goals are:

- ensuring safe amplifier operation  
- preventing hot‑switching of relays  
- providing deterministic timing behavior  
- abstracting hardware differences  
- enabling reliable integration with external systems  

The architecture is intentionally modular, allowing each subsystem to operate independently while contributing to a unified sequencing model.

---

## **Core Architectural Principles**

### **Determinism**
All timing, transitions, and protection responses follow predictable rules.  
No subsystem introduces nondeterministic behavior.

### **Hardware Abstraction**
Relay types, topologies, and station‑specific wiring are abstracted behind stable interfaces.  
The state machine and protection engine operate without knowledge of physical hardware details.

### **Safety First**
Faults, invalid states, and unexpected conditions always collapse into a known‑safe configuration.  
Protection logic is integrated throughout the architecture rather than layered on top.

### **Minimal Configuration Surface**
Only essential structural properties (relay topology, relay type, timing parameters) are configured.  
All other behavior is derived from architecture rules.

---

## **High‑Level Architecture**

The system is composed of several cooperating subsystems:

### **State Machine**
Defines the logical sequencing flow for transmit, receive, and SAFE transitions.  
It is the central coordinator of system behavior.  
See: `firmware-architecture.md`

### **Relay Subsystem**
Abstracts relay topology and actuation model, ensuring deterministic switching independent of hardware.  
See: `relay-architecture.md`

### **Protection Engine**
Monitors RF conditions, timing constraints, and system state to enforce safe operation.  
It can override the state machine to force SAFE transitions.

### **ADC and Measurement Layer**
Provides sampled measurements (e.g., forward/reflected power) used by the protection engine and calibration logic.

### **Event System**
Generates structured events for external systems, enabling integration with logging, GUIs, or automation.

### **Configuration Layer**
Stores structural and timing parameters that define installation‑specific behavior.

### **Main Execution Loop**
Executes the deterministic firmware cycle, evaluating state transitions, protection conditions, and relay actions.

---

## **Interaction Model**

The architecture is built around a clear flow of responsibility:

1. **Measurements** feed into  
2. **Protection logic**, which may override  
3. **State machine transitions**, which drive  
4. **Relay subsystem actions**, which define  
5. **The RF path**, while  
6. **Events** are emitted for external systems.

Each subsystem has a single, well‑defined role, minimizing coupling and simplifying long‑term maintenance.

---

## **Installation Variability**

The sequencer supports a wide range of RF station architectures, including:

- single‑relay TX‑only systems  
- dual‑relay TX+RX systems  
- monostable or latching relays  
- internal or external PA_RDY signaling  
- systems with or without external protection hardware  

These variations are expressed only through configuration and relay topology declarations.  
The architecture itself remains unchanged.

---

## **Design Objectives**

The architecture is designed to:

- provide a stable foundation for long‑term firmware evolution  
- support new relay types and station layouts without redesign  
- ensure predictable behavior under all conditions  
- minimize operator error through clear, deterministic rules  
- maintain separation between logical sequencing and physical hardware  

---

## **Further Reading**

For deeper architectural detail:

- **Firmware Architecture**  
  `firmware-architecture.md`

- **Relay Subsystem Architecture**  
  `relay-architecture.md`

For integration guidance:

- **Integration Examples**  
  `integration-examples.md`

---

