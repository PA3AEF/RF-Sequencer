# **Developer Integration Guide**

This guide describes how external systems can integrate with the RF‑Sequencer using its event‑driven communication model.  
It focuses on architectural concepts rather than implementation details, ensuring that integrations remain stable across firmware versions.

For architectural background, see:

- `docs/architecture/firmware-architecture.md`  
- `docs/overview/overview.md`

---

## **Integration Philosophy**

The sequencer exposes a **unidirectional event stream** that external systems can consume to:

- monitor system state  
- coordinate external hardware  
- trigger automation workflows  
- log protection events  
- display real‑time status  

External systems do **not** control the sequencer’s internal logic.  
The sequencer remains the authoritative controller of RF path transitions and protection behavior.

---

## **Event Model**

Events are structured messages representing internal state changes.  
They fall into several categories:

- **State events** — transmit, receive, SAFE transitions  
- **Protection events** — threshold violations, enforced SAFE‑mode  
- **Relay events** — relay configuration changes  
- **Measurement events** — calibrated forward/reflected power  
- **Configuration events** — updates to installation parameters  

Events are:

- **authoritative** — they reflect the sequencer’s actual state  
- **informational** — they do not require acknowledgment  
- **stable** — the format is designed for long‑term compatibility  

---

## **Integration Patterns**

### **Monitoring Systems**
Consume events to display status, log activity, or provide operator feedback.  
No control over the sequencer is required.

### **Supervisory Controllers**
React to sequencer events to coordinate external hardware such as:

- amplifiers  
- filters  
- antenna switches  
- external protection devices  

The sequencer remains the primary controller of RF path transitions.

### **Automation Systems**
Use events to trigger workflows or scripts, enabling:

- remote operation  
- unattended station control  
- automated protection responses  
- scheduled transmit/receive cycles  

---

## **Architectural Guidelines for Developers**

- **Do not infer state from timing or relay behavior**  
  Always rely on explicit state events.

- **Do not attempt to override protection behavior**  
  SAFE‑mode transitions are mandatory.

- **Do not assume relay topology**  
  Relay events reflect the configured installation.

- **Do not synchronize with internal timing**  
  The sequencer’s timing model is internal and deterministic.

- **Do not depend on event ordering beyond architectural guarantees**  
  Events reflect internal sequencing but should not be used to infer timing windows.

- **Do not attempt to control internal logic**  
  External systems should react to events, not drive state.

---

## **Integration Responsibilities**

External systems are responsible for:

- interpreting event data  
- maintaining their own internal state models  
- coordinating external hardware based on sequencer output  
- handling communication errors or disconnections  
- ensuring their behavior does not conflict with sequencer safety rules  

The sequencer does not validate or supervise external system behavior.

---

## **Design Intent**

The integration model is intentionally simple and robust.  
By exposing a stable event stream and avoiding bidirectional control dependencies, the sequencer remains:

- predictable  
- safe  
- easy to integrate  
- resilient to external failures  

This design ensures reliable operation in diverse environments, from simple home stations to complex automated installations.

