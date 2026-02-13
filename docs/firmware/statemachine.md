
# State Machine Overview
**RF‑Sequencer Firmware**

The RF‑Sequencer is built around a clear, deterministic state machine that controls all transmit/receive transitions and enforces safe sequencing rules. This approach ensures predictable system behavior and guarantees that no unsafe RF condition can occur during operation. All state transitions are driven by validated digital inputs, ADC readings, configuration settings, and protection logic.

---

## State Machine Summary

The firmware defines the following primary states:

### **1. IDLE**
Initial firmware state immediately after boot.
- Hardware initialized
- Inputs normalized
- Protection logic activated
- System prepares to enter RX

### **2. RX (Receive Mode)**
Default operating state.
- TR relay positioned for receive
- PA disabled
- Inputs continuously monitored
- ADC values sampled (FWD, REF, AUX)
- VSWR and REF_FAST checks active

The sequencer remains in RX until a valid transmit request is detected.

### **3. TX_WAIT_PA_RDY**
Intermediate state while preparing for transmit.
Triggered by:
- PTT or RFSENSE input asserting

Actions in this state:
- TR relay commanded to the TX position
- Relay feedback (`RELAY_FB`) must confirm movement
- PA is commanded to enable
- System waits for PA readiness (`PA_RDY`)
- Protections remain active

If the PA fails to assert readiness within the configured timeout, a fault is latched and the sequencer transitions to **Safe Mode**.

### **4. TX_ACTIVE**
Transmit mode is fully engaged when all interlocks and readiness conditions are satisfied.

During TX:
- PA enabled
- TR relay in TX position
- Forward and reflected power continuously monitored
- VSWR and REF_FAST evaluated every cycle
- Any protection fault causes immediate transition to Safe Mode

Leaving TX occurs only when:
- PTT is released
- OR a fault is detected

### **5. SAFEMODE**
A high‑priority emergency state.

Triggered when:
- Relay fails to move
- VSWR exceeds limit
- Fast reflected‑power spike detected
- PA_RDY timeout
- Any unsafe RF condition detected

In Safe Mode:
- PA immediately disabled
- TR relay forced to RX position
- TX outputs inhibited
- Fault latched for CLI inspection
- System remains in Safe Mode until reset or power‑cycled

Safe Mode ensures the RF chain is protected against hardware damage.

---

## State Transition Diagram

```
              +-------+
              | IDLE  |
              +-------+
                  |
                  v
              +-------+
              |  RX   |
              +-------+
          PTT/RFSENSE asserted
                  |
                  v
        +--------------------+
        | TX_WAIT_PA_RDY     |
        +--------------------+
                  | PA_RDY received
                  v
        +--------------------+
        |    TX_ACTIVE       |
        +--------------------+
                  | PTT released
                  v
              +-------+
              |  RX   |
              +-------+

ANY FAULT → SAFEMODE
```


## Inputs That Drive the State Machine

### **Digital Inputs**
- **PTT** — primary transmit trigger
- **RFSENSE** — RF‑based transmit trigger
- **PA_RDY** — amplifier readiness feedback
- **RELAY_FB** — validates relay movement
- **AUX1–AUX3** — general‑purpose logic inputs

All digital inputs are debounced and normalized so that the state machine always operates on clean logical signals.

### **ADC Inputs**
- **Forward Power (FWD)**
- **Reflected Power (REF)**
- **AUX/Test ADC**

Derived values:
- **VSWR**
- **REF_FAST** (fast‑spike detection)

These values drive protection transitions in real time.

---

## **Protection‑Driven Transitions**

The following conditions trigger immediate transition to **Safe Mode**:
- Excessive VSWR
- Rapid reflected‑power spike
- Relay feedback mismatch
- PA_RDY timeout
- Internal consistency errors

The transition is instantaneous and overrides all other logic. The sequencer aborts transmit immediately, restoring the RX path and protecting RF components.

---

## **Visibility in the CLI**

The `STATUS` command displays:
- Current state
- Raw and logical input states
- ADC readings and derived VSWR
- Protection flags
- Last fault and its cause
- Uptime and event counters

The event stream outputs real‑time state changes for integration with logging systems.

---

## **Summary**

The RF‑Sequencer’s state machine provides:
- Deterministic, predictable RF sequencing
- Strict enforcement of interlocks
- Real‑time protection based on ADC and digital inputs
- Transparent state reporting through the CLI
- Immediate fault‑driven recovery via Safe Mode

This architecture ensures safe, reliable operation even in complex RF environments and protects expensive RF components from damage.
