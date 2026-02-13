
# ADC System Overview  
**RF‑Sequencer Firmware**

The RF‑Sequencer continuously samples three ADC channels to monitor RF power levels and system health. These ADC measurements feed directly into the protection subsystem and are visible via the `STATUS` command in the CLI.

---

## ADC Channels

### 1. Forward Power (FWD)
Measures power flowing from the transmitter toward the load (antenna, filters, amplifiers). Used for:
- VSWR computation  
- Monitoring transmitter behavior  
- Telemetry via CLI  

### 2. Reflected Power (REF)
Measures power reflected from the load back toward the transmitter. A rise in REF indicates:
- Antenna mismatch  
- Faulty coax or connectors  
- Relay failure  
- Open/shorted transmission lines  

The sequencer uses REF to:
- Compute VSWR  
- Detect fast reflected‑power spikes (fault condition)

### 3. Test / AUX Analog Input
A general‑purpose analog channel used for calibration, diagnostics, or expansion.

---

## Derived Values

### VSWR (Voltage Standing Wave Ratio)
Calculated from forward and reflected power. High VSWR indicates RF mismatch and can damage PA or relays.

The sequencer:
- Calculates VSWR continuously  
- Compares against configured limits  
- Enters Safe Mode when thresholds are exceeded  

### REF_FAST — Fast Spike Detection
Used to detect rapid reflections caused by:
- Relay switching under RF load  
- Arcing or intermittent connections  
- PA instability  
- Antenna flashover  

A REF_FAST event:
- Latches a fault  
- Aborts TX and enters Safe Mode  
- Logs fault reason for inspection  

---

## Protection Logic Integration

All ADC‑derived values feed into the protection engine, which:
- Monitors real‑time RF conditions  
- Validates relay transitions  
- Detects unsafe VSWR  
- Detects fast reflections  
- Forces Safe Mode when necessary  
- Latches last fault for debugging citeturn20search21

This ensures the RF chain remains protected even during transient events.

---

## Viewing ADC Data in the CLI

Use:
```
STATUS
```
This displays:
- Raw ADC values  
- Normalized values  
- Computed VSWR  
- REF_FAST  
- Protection flags  
- Last fault reason  
- Current state (RX, TX, SAFEMODE)  

---

## Calibration

Calibration parameters for ADC channels include:
- Forward‑power scaling  
- Reflected‑power scaling  
- ADC offsets  
- VSWR limit  
- REF_FAST threshold  

Calibration is handled through the CLI configuration interface (`CFG`).

---

## Summary

The ADC subsystem provides:
- Forward power measurement  
- Reflected power measurement  
- AUX/test analog input  
- VSWR calculation  
- Fast‑spike detection  
- Real‑time protection integration  

This subsystem is essential to the RF‑Sequencer’s safety model and provides transparent RF telemetry for debugging and monitoring.
