# **RF‑Sequencer Firmware Overview**  
**PA3AEF - High‑Reliability RF Safety Controller**

This document provides a high‑level overview of the RF‑Sequencer firmware. It explains the system’s purpose, the major subsystems, and how they work together to deliver deterministic, safety‑critical RF control.

For a full technical deep dive, see:

```
firmware-architecture.md
```

---

# **1. Purpose of the Firmware**

The RF‑Sequencer is designed for installations where RF hardware must be protected with absolute reliability, especially **remote SHF feedpoint boxes**, tower‑top amplifiers, and any system where physical access is limited.

The firmware ensures:

- **Deterministic relay sequencing**  
- **Fast and predictable protection response**  
- **Clear operator feedback**  
- **Safe behavior under all conditions**  

It is built to be transparent, maintainable, and robust in field deployments.

---

# **2. Core Design Philosophy**

The firmware follows several guiding principles:

- **Determinism** - identical inputs always produce identical behavior  
- **Safety first** - protection logic overrides all other functions  
- **Simplicity** - no interrupts, no hidden execution paths  
- **Transparency** - clear logs, clear events, clear state transitions  
- **Extensibility** - modular subsystems with clean boundaries  

These principles shape every subsystem described here.

---

# **3. High‑Level Architecture**

The firmware is composed of several cooperating modules:

---

### **State Machine (Deterministic Core)**

Coordinates all RF sequencing, relay transitions, PA control, and SAFE-MODE entry.  
All transitions are explicit, logged, and executed in a predictable order.

---

### **Protection Engine**

The protection engine continuously evaluates several independent fault domains.  
Each domain can trigger a latched fault, forcing an immediate transition into SAFE-MODE:

- **VSWR** - calculated from forward and reflected power, with a configurable limit.  
- **Fast reflected‑power (REF_FAST)** - a high‑speed protection path that reacts to
sudden spikes in reflected power *before* they appear in the averaged VSWR calculation.
This provides sub‑millisecond protection against abrupt mismatches, arcing, or relay failures.  
- **ADC range validation** - detects sensor faults, wiring issues, or invalid readings.  
- **Relay feedback** - ensures the T/R relay remains engaged during PA_ON and TX; any drop triggers a fault.  
- **PA_RDY timing** - enforces a strict timeout for PA warm‑up and readiness signaling.

Any violation in these domains immediately raises a fault, latches it, and forces the system into SAFE-MODE with deterministic behavior.

---

### **REF_FAST vs. VSWR - What’s the Difference?**

| Mechanism | Purpose | Reaction Speed | Strengths | When It Triggers |
|----------|---------|----------------|-----------|------------------|
| **VSWR** | Measures the *ratio* of reflected to forward power using averaged ADC samples. | Fast, but averaged over several samples. | Stable, accurate, ideal for continuous mismatch detection. | Gradual or sustained mismatch conditions. |
| **REF_FAST** | Monitors *instantaneous* reflected‑power spikes using a high‑speed comparator path. | **Sub‑millisecond** (fastest protection path). | Catches sudden events before they appear in VSWR. | Abrupt mismatches, arcing, relay bounce, connector faults. |

**In practice:**  
VSWR protects against *steady‑state* problems, while **REF_FAST** protects against *sudden, dangerous spikes* that occur too quickly for averaged VSWR to detect. Both work together to provide a complete protection envelope.

---

### **ADC Subsystem**

The ADC subsystem continuously samples the forward and reflected power channels, forming the real‑time measurement backbone of the sequencer. Sampling runs in a tight, deterministic loop, ensuring the firmware always works with the most recent values available. These readings feed directly into the **protection engine** and the **event generator**, enabling fast fault detection and continuous telemetry.  
The subsystem is designed to behave predictably across both Pico and Pico 2 platforms, without relying on interrupts or DMA.

---

### **Relay & PA Control**

Relay and PA control form the hardware‑driving layer of the sequencer. The firmware enforces strict **break‑before‑make** sequencing to prevent relay overlap, validates relay activation using optional feedback, and applies configurable timeouts to detect mechanical or electrical failures. The PA enable line is asserted only after the RF path is confirmed safe, and the PA_RDY signal is monitored with a deterministic timeout to ensure the amplifier is ready before RF is applied.  
During shutdown, the system performs a controlled **unwind sequence** (PA_OFF → TR_RELAY_OFF) to guarantee safe return to receive mode under all conditions.

---


### **Event Generator (SERIAL)**

The event generator produces a continuous stream of machine‑readable status updates over the Serial line (UART0):

```
EVENT,<CATEGORY>,<NAME>,<VALUE>
```

These events are designed for automation, logging, and remote monitoring. Because the format is simple and structured, external systems can easily parse the stream. For example, a JSON‑based dashboard can subscribe to the SERIAL output and visualize forward/reflected power, state transitions, and protection activity in real time.

---

### **CLI (USB CDC and SERIAL)**

The sequencer exposes a human‑friendly command‑line interface on **both USB and SERIAL**, allowing operators to interact with the firmware using any standard serial terminal. The CLI provides full access to the system’s command set, including:

- Status inspection  
- Configuration management  
- Calibration control  
- Fault history and diagnostics  

While both interfaces support the same CLI commands, their roles differ:

- **USB** also carries **human‑readable logs** (INFO/WARN/ERROR).  
- **SERIAL** is primarily used for **machine‑readable events**, but still accepts CLI commands when needed.

The interface includes a lightweight line editor with history recall, and all commands execute synchronously within the deterministic main loop, ensuring that CLI interaction never interferes with protection timing or state‑machine behavior.

---


### **Configuration System**

All settings are stored persistently by the configuration sub-system:

- VSWR limits  
- REF_FAST limits  
- Relay timing  
- Input enables  
- Logging settings  

Changes are saved automatically and emitted as CONFIG events. Saved settings are restored at power-up after the selftest.

---

### **Logging Subsystem**

The logging subsystem provides **human‑readable diagnostics** over the USB interface, using familiar severity levels:

- **INFO** — normal operational messages  
- **WARN** — non‑fatal anomalies or boundary conditions  
- **ERROR** — faults, protection triggers, and critical conditions  

These logs give operators immediate insight into what the sequencer is doing and why. They complement the **machine‑readable SERIAL event stream**, which is intended for automation, dashboards, and remote monitoring. Together, the two channels provide both a clear human narrative and a structured data feed without interfering with the deterministic main loop.

---

# **4. System Behavior Summary**

At runtime, the firmware continuously:

1. Samples ADC values  
2. Evaluates protection thresholds  
3. Updates the state machine  
4. Drives relays and PA lines  
5. Emits events and logs  
6. Responds to CLI commands  

All of this occurs in a **single deterministic main loop** with no interrupts or asynchronous callbacks.

---

# **5. SAFE-MODE**

SAFE-MODE is the system’s protective terminal state:

- Entered on the **first fault**  
- Hardware is forced into a safe configuration  
- Heartbeat displays the fault code  
- Additional faults are recorded but not emitted  
- Manual reset is required to exit  

This ensures predictable, fail‑safe behavior.

---

# **6. Startup Behavior**

On power‑up, the firmware performs:

- Configuration integrity checks (solid heartbeat LED)  
- ADC readiness checks  
- Input sanity checks  
- Internal initialization  

If successful, the system enters `RX_ENABLE`. If not, it enters SAFE-MODE immediately.

---

# **7. Where to Go Next**

For the full technical specification - including timing, fault domains, ADC characteristics, relay sequencing, and detailed subsystem behavior - continue with:

```
firmware-architecture.md
```
