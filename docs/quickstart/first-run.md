
# **First Run & CLI Basics**

This guide helps you verify that the RF‑Sequencer is running correctly and perform the initial configuration. It applies equally to bench setups and remote SHF feedpoint installations.

---

## **Connect to the CLI**

Open your terminal program with (USB or SERIAL):

- **115200 baud**  
- **8N1**  
- **Local echo off**

When the connection is established, you should see:

```
>
```

Try a few basic commands:

```
> STATUS
> CFG
> HELP
```

If you see valid responses, the firmware is running correctly.

---

## **Run the Calibration Wizard (Optional)**

The RF‑Sequencer includes a guided calibration mode that adapts to any directional coupler or detector, which is ideal for remote SHF feedpoint boxes where hardware varies.

Start calibration with:

```
> CALIBRATE
```

The wizard guides you through:

- Zero‑power baseline  
- Forward‑power scaling  
- Reflected‑power scaling  
- VSWR verification  

Calibration is stored in non‑volatile memory.

### **Calibration Is Optional**

If your installation uses **VSWR‑only protection**, or if you have disabled certain inputs, running the full calibration wizard is optional.

You can inspect and adjust input settings with:

```
> GET CONFIG
> SET INPUT <name> ON|OFF
```

Each input (PTT, RFSENSE, PA_RDY, RELAY_FB, VSWR, REF, AUX1-3) can be individually enabled or disabled.  The sequencer automatically adapts its behavior based on which inputs are active.

This flexibility is useful for:

- Minimal bench setups  
- Remote SHF feedpoint boxes  
- Systems without PA_RDY or RFSENSE  
- Installations relying solely on VSWR protection  

---

Ah, perfect — thanks for the clarification.  
Given your actual **HELP** output, your commands are **exactly correct**, and we should reflect them *precisely* in `first-run.md` without inventing or simplifying anything.

Here is a **polished, accurate, and consistent** version of your “Basic Operation” section, rewritten to:

- match your real command syntax  
- keep the quickstart tone clean and beginner‑friendly  
- avoid ambiguity  
- present the commands in a way that’s easy to scan  

You can paste this directly into `first-run.md`.

---
## **Basic Operation**

The following commands are available through both the USB interface and the hardware serial port:

```
> HELP                      # Show all available commands
> STATUS                    # Current state, power levels, protection status
> GET CONFIG                # Show all configuration values
> SET EVENTS ON|OFF         # Enable/disable machine event stream on the serial line
> SET SAFEMODE ON|OFF       # Force or clear SAFE mode
```

Additional useful commands include:

```
> RESET                     # Clear faults and return to RX
> LOGLEVEL <L>              # Set USB log level: DEBUG|INFO|WARN|ERROR
> SET INPUT <NAME> ON|OFF   # Enable/disable individual inputs
```

The system automatically:

- Monitors forward and reflected power  
- Enforces VSWR and REF_FAST protection  
- Manages relay sequencing  
- Enters SAFE mode on any fault  

---
## **Next Steps**

For safety and protection considerations, continue with:

```
safety.md
```

For internal design details, see:

```
../architecture/firmware-architecture.md
```
