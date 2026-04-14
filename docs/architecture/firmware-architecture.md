# **Firmware Architecture**

The firmware architecture defines the deterministic execution model, subsystem boundaries, and interaction rules that govern all sequencing, protection, and relay behavior in the RF‑Sequencer.  
It is designed to provide predictable operation under all conditions while remaining independent of specific relay hardware, station topology, or external integration methods.

This document describes the internal architecture of the firmware.  
For a high‑level conceptual overview, see:

- `overview.md`

For a detailed description of the relay subsystem, see:

- `relay-architecture.md`

---

## **Architectural Goals**

The firmware is built around several core objectives:

- **Deterministic behavior**  
  Every transition, timing step, and protection response follows predictable rules.

- **Hardware abstraction**  
  Relay types, topologies, and electrical characteristics are hidden behind stable interfaces.

- **Safety under all conditions**  
  Faults, invalid states, and unexpected inputs always collapse into a known‑safe configuration.

- **Minimal configuration surface**  
  Only structural and timing parameters are configured; all other behavior is derived.

- **Long‑term maintainability**  
  Subsystems are isolated, with clear responsibilities and minimal coupling.

---

## **Execution Model**

The firmware operates in a deterministic main loop.  
Each cycle evaluates:

- state machine transitions  
- protection conditions  
- relay subsystem actions  
- event generation  
- timing progression  

No asynchronous execution paths modify system state.  
All changes occur within the controlled boundaries of the main loop, ensuring predictability and simplifying reasoning about system behavior.

---

## **State Machine**

The state machine defines the logical sequencing flow for transmit, receive, and SAFE transitions.  
It is the central coordinator of system behavior and the primary consumer of subsystem outputs.

The state machine:

- evaluates input conditions  
- determines the next logical state  
- requests relay transitions  
- coordinates timing windows  
- interacts with the protection engine  

The state machine does **not** interact with hardware directly.  
All physical actions are routed through the relay subsystem and timing model.

For relay subsystem details, see `relay-architecture.md`.

---

## **Relay Subsystem (Summary)**

The relay subsystem abstracts the physical RF switching hardware.  
It models:

- relay topology (single‑relay TX‑only or dual‑relay TX+RX)  
- actuation type (monostable or latching)  
- deterministic transition semantics  

The state machine issues logical relay requests; the subsystem translates them into hardware‑specific actions while guaranteeing:

- ordered transitions  
- bounded switching latency  
- no ambiguous intermediate states  
- consistent SAFE‑mode behavior  

For the full subsystem specification, see `relay-architecture.md`.

---

## **Protection Engine**

The protection engine enforces safe operation by monitoring measurement inputs, timing constraints, and system state.  
It can override the state machine to force SAFE transitions when required.

The protection engine evaluates:

- forward and reflected power  
- timing violations  
- invalid state combinations  
- external PA_RDY conditions  
- calibration validity  

When a protection condition is triggered, the engine:

- forces the system into SAFE mode  
- requests a known‑safe relay configuration  
- emits protection events  
- prevents further transitions until conditions clear  

Protection logic is integrated throughout the architecture rather than layered on top.

---

## **ADC and Measurement Layer**

The ADC subsystem provides sampled measurements used by the protection engine and calibration logic.  
It ensures:

- stable sampling intervals  
- noise‑aware averaging  
- predictable update timing  
- isolation from the main loop’s sequencing logic  

Measurements include:

- forward power  
- reflected power  
- auxiliary analog inputs (installation‑dependent)

The measurement layer does not interpret values; interpretation is handled by the protection engine and calibration subsystem.

---

## **Calibration Layer**

The calibration subsystem converts raw ADC readings into meaningful engineering values.  
It applies:

- scaling factors  
- offsets  
- linearization rules  

Calibration parameters are stored in the configuration layer and validated at startup.  
Invalid calibration data forces SAFE‑mode operation until corrected.

---

## **Event System**

The event system generates structured events that external systems can consume for logging, GUIs, or automation.  
Events are emitted for:

- state transitions  
- protection triggers  
- relay actions  
- configuration changes  
- measurement updates  

Events are strictly informational; they do not influence internal logic.

For integration examples, see `integration-examples.md`.

---

## **Configuration Layer**

The configuration layer stores installation‑specific parameters, including:

- relay topology  
- relay type  
- timing values  
- calibration data  
- protection thresholds  

Configuration is validated at startup.  
Invalid or incomplete configuration forces SAFE‑mode operation.

The configuration layer does not contain operational logic; it only provides structured data to other subsystems.

---

## **Main Execution Loop**

The main loop executes the deterministic firmware cycle:

1. read measurements  
2. evaluate protection conditions  
3. evaluate state machine transitions  
4. compute relay actions  
5. update timing windows  
6. emit events  

No subsystem modifies state outside this loop.  
This ensures predictable timing and simplifies debugging, integration, and long‑term maintenance.

---

## **Subsystem Boundaries**

Each subsystem has a single, well‑defined responsibility:

- **State Machine** — logical sequencing  
- **Relay Subsystem** — hardware abstraction  
- **Protection Engine** — safety enforcement  
- **ADC Layer** — measurement acquisition  
- **Calibration Layer** — measurement interpretation  
- **Event System** — external communication  
- **Configuration Layer** — installation parameters  
- **Main Loop** — deterministic execution  

This separation ensures that changes in one subsystem do not propagate unintended effects into others.

---

## **Design Rationale**

The architecture is intentionally conservative.  
RF systems demand predictable behavior, and the cost of nondeterminism is high.  
By isolating subsystems, enforcing strict sequencing rules, and abstracting hardware differences, the firmware remains stable across:

- new relay types  
- new station architectures  
- new protection requirements  
- future firmware extensions  

The architecture is designed to evolve without compromising safety or determinism.

---

## **Further Reading**

- **Architecture Overview**  
  `overview.md`

- **Relay Subsystem Architecture**  
  `relay-architecture.md`

- **Integration Examples**  
  `integration-examples.md`

---

**File 2 of 4 complete.**  
Say **“next”** and I will deliver **relay-architecture.md**.