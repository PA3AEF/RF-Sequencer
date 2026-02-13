# 📘 **Calibration Wizard**

The RF Sequencer includes a built‑in **Guided Calibration Wizard** that allows the user to calibrate any external directional coupler or detector without modifying hardware or firmware. This ensures accurate forward and reflected voltage readings regardless of coupler type, frequency, or coupling factor.

The wizard automatically computes the correct calibration factors based on two simple measurements.

---

## **When to Use the Calibration Wizard**

Use the wizard whenever:

- You connect a new coupler or detector  
- You change frequency bands  
- You want to improve measurement accuracy  
- You suspect the coupler output has drifted  

The wizard calibrates **FWD** and **REF** channels independently.

---

# **How the Calibration Wizard Works**

The sequencer measures:

- The **ADC voltage** at the microcontroller (after the resistor divider)  
- The **actual coupler voltage** you enter (measured with a multimeter)

It then computes:

$$
\text{scale} = \frac{V_{\text{coupler}}}{V_{\text{adc}}}
$$

This scale factor is stored in configuration and applied automatically to all future readings.

VSWR calculations continue to use **raw ADC values**, ensuring correct physics.

---

# **Running the Calibration Wizard**

Start the wizard:

```
> CALIBRATE
```

The sequencer enters calibration mode and guides you step‑by‑step.

### **Step 1 — Apply RF Power**
Apply a stable RF signal at the power level you want to calibrate.

The display shows:

```
=== Guided Calibration Mode ===
Step 1: Apply RF power now.
Measure the FORWARD coupler voltage with a multimeter.
Enter the measured FWD voltage (in volts):
```

Enter the measured forward voltage, for example:

```
2.90
```

### **Step 2 — Measure REF Voltage**
The wizard continues:

```
OK.
Now measure the REFLECTED coupler voltage.
Enter the measured REF voltage (in volts):
```

Enter the reflected voltage, for example:

```
0.60
```

### **Step 3 — Review Calibration Results**

The sequencer computes the new scale factors and displays:

```
=== Calibration Results ===
FWD scale: 15.0000
REF scale: 12.3456

Type YES to apply these values, or NO to cancel:
```

Confirm:

```
YES
```

The sequencer stores the calibration in non‑volatile memory.

---

# **STATUS Output After Calibration**

A typical STATUS output looks like:

```
ANALOG INPUTS (RAW ADC)
  FWD_ADC:           3968
  REF_ADC:           1015
  VTEST_ADC:         824

COUPLER VOLTAGES (CALIBRATED)
  FWD_VOLTS:         3.10 V
  REF_VOLTS:         0.78 V

DERIVED VALUES
  VSWR:              1.69
  REF_FAST:          ACTIVE
```

- **RAW ADC** values are integers (0–4095)  
- **CALIBRATED VOLTS** reflect your coupler’s real output  
- **VSWR** uses raw ADC values for correct physics  
- **REF_FAST** indicates fast protection status  

---

# **Notes and Best Practices**

- Calibration is **frequency‑dependent** if your detector is frequency‑sensitive  
- You can recalibrate as often as needed  
- Calibration does **not** affect VSWR protection thresholds  
- Calibration does **not** affect REF_FAST thresholds  



