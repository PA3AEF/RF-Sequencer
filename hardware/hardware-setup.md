# **Hardware Setup, Wiring Basics, and First‑Time Preparation**

## **1. Introduction**

The RF‑Sequencer is a modern, safety‑focused, and configurable RF transmit/receive sequencer designed to protect radios, amplifiers, relays, and other station hardware. It is built around the Raspberry Pi Pico/Pico2 platform, chosen for reliability, speed, and ease of firmware management. [\[github.com\]](https://github.com/PA3AEF/RF-Sequencer)

This document guides you through essential hardware wiring, preparation, and the first power‑up process.

***

## **2. Required Components**

*   RF‑Sequencer PCB
*   Raspberry Pi Pico or Pico2 (recommended due to better performance and lower power consumption) [\[github.com\]](https://github.com/PA3AEF/RF-Sequencer)
*   Power supply (5 V USB for the Pico; 12–13.8 V system supply for station devices)
*   TX, RX, PA‑enable lines (depending on station hardware)
*   TR relay
*   Cables for digital inputs (PTT, RFSENSE, PA\_RDY, RELAY\_FB, AUX inputs)
*   Coaxial cabling for RF path (radio ↔ sequencer ↔ amplifier/preamp)
*   A dummy load or antenna *connected before any RF test*

***

## **3. Understanding the Sequencer I/O**

The sequencer monitors and controls:

### **Digital Inputs**

*   **PTT** – primary keying line from your radio.
*   **RFSENSE** – RF‑power‑based keying.
*   **PA\_RDY** – signals power amplifier readiness.
*   **RELAY\_FB** – feedback from TR relay verifying mechanical movement.
*   **AUX1–AUX3** – general‑purpose logic inputs.  
    Digital inputs are *debounced and normalized* internally to ensure stable operation. [\[github.com\]](https://github.com/PA3AEF/RF-Sequencer)

### **Analog Inputs**

*   **Forward Power ADC**
*   **Reflected Power ADC**
*   **Test ADC**

The sequencer derives **VSWR**, detects **fast reflected power spikes**, and evaluates system health continuously. [\[github.com\]](https://github.com/PA3AEF/RF-Sequencer)

### **Outputs**

All outputs (TX, RX, PA\_ENABLE, TR relay, indicator LEDs) are controlled exclusively by the sequencer’s internal state machine. [\[github.com\]](https://github.com/PA3AEF/RF-Sequencer)

***

## **4. Wiring Basics**

### **4.1 Connect Power**

*   Power the Pico via USB (5 V).
*   Power RF devices (PA, relays) from their appropriate supply rails (commonly 12–28 V depending on hardware).

### **4.2 Connect Control Lines**

*   **PTT from radio → PTT input**
*   **TR relay → TR output**
*   **PA control → PA\_ENABLE output**
*   **RX/TX interlock lines → RX\_ENABLE / TX\_ENABLE outputs**

Ensure **common grounds** between the sequencer, radio, PA, and relay systems.

### **4.3 Connect Feedback Lines**

*   **Relay feedback (RELAY\_FB)** is essential for safety — the sequencer *verifies* the relay moved to the commanded position before enabling RF.
*   **PA\_RDY** wiring ensures the PA must signal readiness before transmit is allowed.

These mechanisms directly support Safe Mode activation if something goes wrong. [\[github.com\]](https://github.com/PA3AEF/RF-Sequencer)

***

## **5. First‑Time Preparation Checklist**

1.  Inspect all wiring for:
    *   correct polarity
    *   clean grounds
    *   no shorts
    *   proper connector seating

2.  Verify control cables BEFORE connecting RF power.

3.  Connect a dummy load or antenna.

4.  With the PA powered but unkeyed:
    *   ensure **PA\_RDY is correct**
    *   ensure relay feedback works (TR relay clicks reliably)

5.  Connect the Pico’s USB to your computer.
    *   The Pico exposes USB for power and firmware handling.
    *   If the bootloader is ever corrupted, it can be easily restored by copying a UF2 file onto the Pico’s USB mass‑storage device. [\[github.com\]](https://github.com/PA3AEF/RF-Sequencer)

***
