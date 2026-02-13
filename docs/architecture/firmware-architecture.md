# **Firmware Architecture**
**RF‑Sequencer – PA3AEF**

**A deterministic, safety‑driven firmware architecture for reliable RF sequencing and remote operation.**

This document describes the internal firmware architecture of the RF‑Sequencer. It explains how the system achieves deterministic timing, continuous protection, and reliable remote operation, even when deployed in demanding RF environments such as tower‑top amplifiers or dish feedpoint enclosures. Rather than focusing on source code, the document outlines the design principles, subsystem responsibilities, and architectural decisions that shape the firmware. The goal is to provide a clear, maintainable, and future‑proof understanding of how the sequencer operates, why it behaves the way it does, and how its components interact to ensure safety and predictability in all operating conditions.

---

# **1. Architectural Principles**
The firmware is built around a set of principles that ensure predictable behavior,
robust protection, and reliable remote operation, even when the sequencer is deployed in demanding
environments such as tower‑top enclosures or dish feedpoint RF boxes.

**Determinism** -
All timing, sequencing, and protection logic must behave identically on every cycle. The system never
relies on chance, timing quirks, or implicit ordering. Determinism ensures that relay transitions,
protection thresholds, and state changes remain stable and predictable under all conditions.

**Non‑blocking execution** -
All firmware tasks run cooperatively without ever pausing the system. There are no delays, sleeps, or
blocking loops, pitfalls that undermine many embedded Arduino-ish designs. Every subsystem receives frequent,
predictable execution time, ensuring relay timing, protection checks, and remote control remain responsive
even over long serial links or IP‑serial bridges.

**Safety first** -
Protection logic always takes precedence over user commands, sequencing requests, or external control.
Faults are detected continuously and acted upon immediately, guaranteeing that no command or timing
sequence can compromise the safety of connected RF equipment.

**Transparency** -
Every state change, fault, and configuration update is emitted as a machine‑friendly event. This provides
complete visibility for operators, logging systems, and remote controllers, making the sequencer easy to
integrate into larger RF infrastructures.

**Hardware abstraction** -
All GPIO, ADC, and relay operations are isolated behind clear interfaces. This Hardware Abstraction Layer keeps the
firmware portable, maintainable, and resilient to hardware revisions, while ensuring that higher‑level logic remains clean and
hardware‑agnostic.

**Configurability** -
All thresholds, timings, calibration factors, and feature toggles are stored in non‑volatile memory.
The system adapts to different couplers, relays, and station architectures without requiring firmware
changes, making it suitable for both standalone use and embedded integration

---

# **2. Main Execution Loop**
The RF‑Sequencer firmware is built around a fast, cooperative main execution loop rather than an interrupt‑driven
architecture. This approach provides deterministic timing, predictable behavior, and continuous protection, all
essential for a safety‑critical RF controller.

## **Execution Model and Timing**
On a Raspberry Pi Pico or Pico 2 the main loop typically runs at 2–5 kHz, giving every subsystem frequent,
consistent execution time. This ensures:
- sub‑millisecond protection responsiveness
- precise relay sequencing
- stable ADC sampling
- smooth CLI interaction
- reliable event emission

The RP2040’s deterministic execution, fast GPIO, and ample CPU headroom make the Pico family an excellent fit
for this cooperative design.

## **Why Not Interrupts?**
An early interrupt‑driven prototype exposed subtle timing jitter and unpredictable execution paths.
Interrupt‑based designs can pre‑empt critical tasks at arbitrary moments, introducing delays and hidden
behavior that are unacceptable in a safety‑critical RF sequencer. Interrupts can:

- delay protection checks
- distort ADC sampling cadence
- interrupt relay transitions mid‑sequence
- create race conditions in state logic
- make remote control feel inconsistent

By avoiding interrupts entirely, the sequencer ensures that every subsystem runs predictably and that no
unexpected preemption can compromise safety or timing accuracy..

## **Cooperative Subsystem Scheduling**
Each subsystem exposes a lightweight tick() function that runs every loop cycle in a fixed,
intentional order:

- ADC sampling & filtering
- Calibration layer
- Protection evaluation
- State machine update
- Relay sequencing engine
- Event generator
- CLI processing
- Configuration persistence checks
- Heartbeat update

This structure ensures that protection always sees fresh data, relay timing remains precise, and the
system stays responsive even when deployed in remote RF enclosures such as dish feedpoints or
tower‑top amplifier boxes.

---

# **3. ADC Subsystem**
The ADC subsystem provides the firmware with continuous, high‑integrity measurements of the RF environment.
It is responsible for sampling the forward, reflected, and test voltages, filtering them, validating their
ranges, and delivering stable data to the protection and calibration layers.

Architecturally, the ADC subsystem is designed to be:
- non‑blocking - sampling and filtering occur cooperatively
- predictable - sampling intervals remain consistent across Pico and Pico 2
- robust - out‑of‑range conditions immediately trigger ADC_RANGE faults
- hardware‑agnostic - differences between ADC implementations are abstracted
- cleanly layered - raw values feed protection; calibrated values feed reporting

The subsystem performs oversampling and averaging to reduce noise, ensuring that protection decisions are based on stable, meaningful data. All raw ADC values remain in integer form until passed to the calibration layer, preserving precision and avoiding floating‑point drift.

This design ensures that the sequencer maintains accurate RF measurements even in electrically noisy environments or when installed at remote RF locations where reliability is paramount.

---

# **4. Calibration Layer**

The calibration layer transforms raw ADC readings into meaningful, coupler‑specific forward and reflected power
values. It applies user‑defined scaling factors and ensures that the sequencer can adapt to any directional
coupler, detector, or RF sensing hardware without requiring firmware changes.

Architectural goals:
- universality - supports any coupler or detector through simple scaling
- safety isolation - calibration never affects protection thresholds
- precision - scaling is applied after filtering but before reporting
- persistence - calibration factors are stored in non‑volatile memory
- guided workflow - the Calibration Wizard provides a structured, foolproof process

The calibration layer sits between the ADC subsystem and all user‑visible reporting. Protection logic always uses raw, uncalibrated values to ensure that safety decisions remain independent of user configuration.
This separation guarantees that calibration enhances usability without ever compromising protection integrity.

---

# **5. Protection Subsystem**
The protection subsystem continuously evaluates the health of the RF path and enforces immediate,
deterministic responses to unsafe conditions. It operates independently of user commands, sequencing
requests, or external control, ensuring that no operational scenario can compromise connected RF equipment.

Protection monitors:
- VSWR
- reflected power
- REF_FAST thresholds
- ADC range validity
- relay feedback (optional)
- configuration integrity

Faults are detected in real time and acted upon without delay. When a fault is triggered, the system
transitions into a safe state, unwinds relay positions, and emits a clear, machine‑friendly event describing
the condition.

Architecturally, the protection subsystem is designed to be:

- continuous - evaluated on every loop cycle
- deterministic - identical behavior under all conditions
- non‑blocking - never delays other subsystems
- authoritative - overrides all user or automation requests
- transparent - every fault is logged and emitted immediately

This ensures the sequencer remains trustworthy even when deployed in remote or harsh RF environments.

---

# **6. State Machine**

The state machine governs the operational mode of the sequencer and provides the backbone for predictable
system behavior. It defines a small set of well‑understood states:
- **IDLE**
- **TX**
- **FAULT**
- **RECOVERY**
- **SAFEMODE**

Each transition is explicit, validated, and logged. There are no implicit or hidden states, and no transitions
occur without a clear architectural reason.

The state machine does not manipulate hardware directly. Instead, it issues high‑level intent (“enter TX”, “enter SAFE”, “recover from fault”), and the relay sequencing engine executes the transition safely and non‑blocking.

Architectural characteristics:
- **Deterministic** - transitions always follow the same rules
- **Safe** - no state allows unsafe relay combinations
- **Observable** - every transition emits a machine‑friendly event
- **Authoritative** - user commands request transitions, but protection logic decides
- **Modular** - relay control and protection logic remain separate

This separation of intent (state machine) and execution (relay engine) keeps the system predictable,
maintainable, and easy to reason about.

---

# **7. Relay Sequencing Engine**
The relay sequencing engine is responsible for executing state transitions safely and precisely. It handles
all timing, ordering, and validation required to move between relay configurations without ever blocking the
main loop.

Responsibilities include:
- break‑before‑make transitions
- configurable timing delays
- safe unwind during faults
- optional relay feedback validation
- non‑blocking timing (no sleeps or delays)

The engine receives high‑level intent from the state machine and converts it into a sequence of micro‑operations,
each executed cooperatively over multiple loop cycles. This ensures that relay timing remains accurate even when
the system is under load or being controlled remotely.

Architectural goals:
- **Precision** - timing is accurate and repeatable
- **Safety** - no relay combination can damage RF hardware
- **Responsiveness** - transitions never block protection checks
- **Robustness** - feedback validation detects stuck or failed relays
- **Transparency** - sequencing steps and outcomes are logged

This design allows the sequencer to operate reliably in demanding RF installations, including tower‑top amplifiers, dish feedpoint boxes, and remote stations where relay behavior must be both predictable and safe.

---

# **8. Configuration Storage**

The configuration subsystem provides persistent, fault‑tolerant storage for all user‑defined parameters,
including timing values, protection thresholds, calibration factors, and feature toggles. It is built on top
of the Pico/Pico 2’s LittleFS‑backed flash file system, ensuring reliable operation even in environments where
power may be interrupted or the device is deployed remotely.

Architectural characteristics:

- **File‑system based** - configuration is stored as a compact, versioned structure on LittleFS, avoiding the
wear‑leveling and corruption issues common with raw flash writes.
- **Validated at boot** - the configuration block is checked for structural integrity and version compatibility.
Any inconsistency triggers a CONFIG fault and forces safe defaults.
- **Atomic updates** - writes are performed in a way that ensures the system never ends up with a partially written
configuration, even during unexpected resets.
- **Event‑driven transparency** - every configuration change emits a machine‑friendly CONFIG event, allowing remote
controllers to track updates in real time.
- **Decoupled from firmware** - all operational parameters are user‑modifiable without requiring firmware changes,
making the sequencer adaptable to any RF station architecture.

This design ensures that configuration remains reliable, portable, and safe - a critical requirement for
installations where the sequencer may be mounted at a dish feedpoint, tower‑top amplifier, or other inaccessible
location.

---

# **9. CLI Subsystem**

The Command Line Interface (CLI) is the primary human‑ and machine‑interaction layer of the RF‑Sequencer.
It is intentionally simple, robust, and text‑based so it can operate reliably over any transport: USB serial, remote serial bridges, IP‑serial converters, or embedded controllers inside larger RF systems.

The CLI is designed with two complementary goals:
1. **Friendly and discoverable for humans** -
  -  Operators can connect with any terminal program and immediately understand the system through:
  - readable, self‑describing commands
  - aligned help output
  - clear status reports
  - guided calibration
  - intuitive history navigation (UP/DOWN)

  This makes the sequencer approachable even when installed in difficult‑to‑reach locations.

2. **Deterministic and machine‑friendly for automation** - Every command produces predictable, line‑oriented output suitable for:
  - remote control of RF boxes at a dish feedpoint
  - integration with station automation
  - monitoring systems
  - logging frameworks
  - custom GUIs
  - embedded controllers in amplifiers or transverters

  The event stream is intentionally minimalistic and stable, making it easy to parse even on low‑resource microcontrollers.

Architectural notes:

- CLI runs cooperatively
- never blocks the main loop
- all commands are stateless and idempotent
- output is line‑oriented for easy parsing

---

# **10. Event Generator**

The event generator provides a transparent, machine‑friendly stream of system activity. Every significant action, state transitions, faults, configuration updates, calibration progress, and periodic measurements is emitted as a clean, line‑oriented event suitable for both human operators and automated systems.

Architectural characteristics:
- **deterministic** - events follow a stable, documented format
- **non‑blocking** - event emission never delays protection or sequencing
- **minimalistic** - designed for easy parsing by scripts, GUIs, or embedded controllers
- **comprehensive** - reflects all meaningful internal activity
- **transport‑agnostic** - works equally wel over USB serial, IP‑serial bridges, or remote links

This event stream is the foundation for remote monitoring and control. Whether the sequencer is installed at a dish feedpoint, tower‑top amplifier, or remote station, external systems can reliably track its behavior, log its activity, and react to faults in real time.
The event generator turns the sequencer into a transparent, observable component of a larger RF infrastructure.

For practical examples of consuming the UART0 event stream, see:
```
integration-examples.md
```

---

# **11. Heartbeat System**

The heartbeat LED provides immediate, at‑a‑glance insight into the sequencer’s operational state. It is
intentionally simple, non‑blocking, and expressive, serving as a universal status indicator even when no
terminal or remote connection is available.

The heartbeat operates in several distinct modes:
- **Normal operation** - a steady, rhythmic pulse indicating the system is healthy and responsive.
- **SAFE mode** - a slow, deliberate pattern signaling that the sequencer has entered a protective state due
  to a fault or unsafe condition.
- **Fault indication** - rapid or distinctive patterns that highlight active fault conditions, aiding quick
  diagnosis in the field.
- **Calibration mode** - a unique pattern that clearly distinguishes calibration from normal operation,
  preventing accidental RF activation during setup.

Architectural goals:
- **Non‑blocking** - heartbeat updates never interfere with relay timing, protection checks, or CLI
  responsiveness.
- **Multipurpose** - a single LED conveys operational, fault, and calibration states without requiring
  additional hardware.
- **Field‑friendly** - patterns are designed to be recognizable even in bright outdoor environments or
  when the sequencer is mounted in a remote RF enclosure.
- **Deterministic** - timing and patterns remain consistent across all hardware variants.

The heartbeat system turns the sequencer into a self‑reporting device, offering immediate visual feedback that
complements the machine‑friendly event stream.

---

# **13. Design Philosophy Summary**

The firmware is designed to be:

- **safe**
- **predictable**
- **transparent**
- **modular**
- **non‑blocking**
- **hardware‑agnostic**
- **user‑friendly**

It is intentionally simple in structure but robust in behavior.

# **14. Design Rationale**
The RF‑Sequencer firmware architecture is the result of deliberate engineering choices shaped by real‑world RF operating conditions, safety requirements, and the need for reliable remote operation. This section summarizes the motivations behind the major architectural decisions and the trade‑offs that guided the design.

## **Deterministic Cooperative Loop Instead of Interrupts**
A cooperative main loop was chosen over an interrupt‑driven design to guarantee predictable timing and eliminate hidden execution paths. An early interrupt‑based prototype exposed subtle jitter and pre‑emption effects that could delay protection checks, distort ADC sampling, or interrupt relay transitions mid‑sequence. In a safety‑critical RF controller, such unpredictability is unacceptable. The cooperative loop ensures that every subsystem runs at a stable, high frequency with no unexpected preemption.

## **Clear Separation of Intent and Execution**
The architecture separates what the system should do from how it is done.
The state machine expresses intent, the relay engine executes transitions safely, and the protection subsystem can override both at any time. This separation keeps the system maintainable, testable, and robust, while preventing complex interactions between timing‑critical components.

## **Protection as the Highest Authority**
RF hardware is expensive and often deployed in inaccessible locations such as tower‑top amplifiers or dish feedpoints. The protection subsystem therefore operates continuously and independently, evaluating fresh ADC data every loop cycle. It can force SAFE mode immediately, regardless of user commands or automation. This ensures that no external control can compromise the safety of connected equipment.

## **Transparent, Machine‑Friendly Event Stream**
The sequencer emits a clean, line‑oriented event stream that reflects all meaningful internal activity. This transparency is essential for remote installations where operators rely on logs, GUIs, or automation systems to understand system behavior. Events are deterministic, minimalistic, and easy to parse, making the sequencer a reliable component in larger RF infrastructures.

## **Hardware Abstraction for Portability and Longevity**
All GPIO, ADC, and relay operations are isolated behind hardware abstraction layers. This keeps the firmware portable across Pico and Pico 2 hardware and ensures that future hardware revisions or relay configurations can be supported without rewriting core logic. The architecture is designed to outlive any single hardware generation.

## **File‑System‑Backed Configuration for Reliability**
Configuration is stored on a LittleFS‑backed flash file system to ensure durability, atomic updates, and safe recovery from corruption. This approach avoids the wear‑leveling issues of raw flash writes and guarantees that user settings survive power interruptions — a critical requirement for remote RF installations.

## **Minimal, Robust CLI for Human and Machine Control**
The CLI is intentionally simple and resilient. It avoids complex terminal features and instead focuses on predictable, stateless commands that work reliably over USB serial, IP‑serial bridges, or embedded controllers. This makes the sequencer equally suitable for hands‑on operation and fully automated remote control.

## **Multipurpose Heartbeat for Immediate Visual Feedback**
A single LED conveys operational, fault, SAFE‑mode, and calibration states through distinct patterns. This provides immediate, hardware‑level feedback even when no terminal connection is available, making field diagnostics faster and more intuitive.

# **15. Closing Note**
The RF‑Sequencer firmware architecture reflects a clear philosophy: predictable behavior, uncompromising safety, and transparent operation are more valuable than complexity or cleverness. Every subsystem — from the cooperative main loop to the protection engine, relay sequencing, calibration, configuration storage, CLI, and heartbeat — is designed to behave consistently, even in remote or harsh RF environments where reliability matters most. By combining deterministic timing, clean layering, and robust fault handling, the sequencer becomes a dependable component in any RF chain, whether mounted at a dish feedpoint, integrated into an amplifier, or operated from a remote station. This architecture is built to be understood, trusted, and extended, ensuring the system remains maintainable and future‑proof for years to come
