---
title: Quick Start Guide
parent: Quickstart
nav_order: 1
---

# Overview

# **RF‑Sequencer – Quick Start Guide**

This guide helps you get the RF‑Sequencer wired, configured, and operational in just a few minutes.  
It assumes basic familiarity with radios, PTT control, and RF hardware.

---

## **1. Overview**

The RF‑Sequencer ensures safe and deterministic switching between **RX** and **TX** in RF systems.  
It protects your PA, relays, and front‑end by enforcing correct timing and monitoring critical signals:

- PTT  
- PA_RDY  
- Relay feedback  
- VSWR / Reflected power  
- RF‑sense  
- Auxiliary inputs  

The sequencer drives **TX_ENABLE**, **RX_ENABLE**, **PA_ENABLE**, and dual‑coil relays (TX and RX coils).

---

## **2. Hardware Connections**

### **Required connections**
| Signal | Direction | Description |
|--------|-----------|-------------|
| **PTT_N** | Input | Active‑low PTT from radio or footswitch |
| **PA_RDY** | Input | PA ready / interlock input |
| **RELAY_FB** | Input | Relay position feedback (optional) |
| **FWD / REF ADC** | Input | Coupler forward/reflected power |
| **TX_RELAY** | Output | TX coil drive (monostable or latching) |
| **RX_RELAY** | Output | RX coil drive |
| **TX_ENABLE** | Output | Enables TX path |
| **RX_ENABLE** | Output | Enables RX path |
| **PA_ENABLE** | Output | Enables PA |

### **Relay wiring**
The firmware always supports **two coils**:

- Connect **TX coil** → `PIN_TX_RELAY`
- Connect **RX coil** → `PIN_RX_RELAY`  
- If you use a TX‑only relay, simply leave RX unconnected.

Overlap protection remains active but harmless if RX is not wired.

---

## **3. Power‑Up Behavior**

On boot:

1. Sequencer initializes all GPIOs  
2. Relay outputs default to **RX**  
3. PA is disabled  
4. Faults are cleared  
5. Status is printed over USB (if logging enabled)

---

## **4. USB CLI**

Open a serial terminal at:

```
115200 baud, 8N1
```

You’ll see:

```
RF Sequencer - PA3AEF <build>
> 
```

### **Useful commands**

```
> HELP
> STATUS
> RESET
> GET CONFIG
> SET STATE_DELAY <ms>
> SET ACTUATION MONOSTABLE|LATCHING
> SET INPUT <NAME> ON|OFF
> SET VSWR_LIMIT <float>
> SET REF_FAST_LIMIT <int>
> SET LOGGING ON|OFF
> SET EVENTS ON|OFF
```
Commands are not case senssitive. 
Use **TAB / arrow keys** for command history.

---

## **5. Basic Configuration**

### **Check current settings**
```
> GET CONFIG
```

### **Enable/disable inputs**
```
> SET INPUT PTT ON
> SET INPUT PA_RDY ON
> SET INPUT RELAY_FB ON
> SET INPUT VSWR ON
> SET INPUT REF ON
```

### **Set relay actuation**
```
> SET ACTUATION MONOSTABLE
```
or
```
> SET ACTUATION LATCHING
```

### **Set timing**
```
> SET STATE_DELAY 0        (normal operation)
> SET STATE_DELAY 200      (slow transitions for debugging or slow hardware)
> SET RELAY_TIMEOUT 50
> SET PA_TIMEOUT 200
```

---

## **6. Operating the Sequencer**

### **RX → TX transition**
Triggered by:

- PTT pressed  
- RF‑sense (if enabled)  
- External AUX input (if configured)

Sequence:

1. Disable RX path  
2. Switch relay to TX  
3. Wait for relay feedback (if enabled)  
4. Enable TX path  
5. Enable PA  
6. Monitor VSWR / REF_FAST continuously  

### **TX → RX transition**
Triggered by:

- PTT released  
- Fault condition  
- PA_RDY lost  
- VSWR/REF protection  

Sequence:

1. Disable PA  
2. Disable TX path  
3. Switch relay to RX  
4. Enable RX path  

---

## **7. Protection System**

The sequencer continuously monitors:

- **VSWR**  
- **Fast reflected power**  
- **Relay overlap**  
- **Relay feedback mismatch**  
- **PA_RDY timeout**  
- **Relay timeout**  

On fault:

- PA is disabled  
- TX is disabled  
- Relay returns to RX  
- Fault is latched  
- SAFE MODE is entered  
- Event is logged and emitted over serial  

Clear faults with:

```
> RESET
```

---

## **8. VSWR Calibration**

Start guided VSWR calibration:

```
CALIBRATE
```

Follow on‑screen instructions.

---

## **9. Debugging Tools**

### **State transition delay**
Slows down the sequencer so you can visually observe relay and LED behavior.

```
> SET STATE_DELAY 200
```

Set to zero for normal operation:

```
> SET STATE_DELAY 0
```

### **Event stream**
Machine‑readable events over Serial1:

```
> SET EVENTS ON
```

### **USB logging**
Human‑readable logs:

```
> SET LOGGING ON
```

---

## **10. Firmware Updates**
Firmeare can be downloaded from the firmware section. Follow its instructions to update. 
1. Flash via USB or SWD   
2. Config is preserved across updates  
3. Factory reset if needed:

```
> FACTORY_RESET
```

---

## **11. Safety Notes**

- Always verify relay wiring before connecting RF power  
- Use relay feedback if available  
- Ensure PA_RDY is correctly wired for your amplifier  
- Never bypass VSWR protection unless testing at low power  

---

## **12. Support & Documentation**

- GitHub: `https://github.com/PA3AEF/RF-Sequencer`
- Issues: use GitHub issue tracker  
- Hardware schematics: `/hardware`  
- Firmware docs: `/docs`  

---
