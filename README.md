# **RF‑Sequencer – High‑Reliability RF Safety & Relay Controller**
**PA3AEF – Amateur Radio / RF Engineering Project**

A deterministic, safety‑driven controller for RF sequencing, protection, and relay management. Designed for amateur radio, RF labs, and high‑power systems where timing accuracy and hardware protection are critical.

This RF‑Sequencer combines robust hardware, carefully engineered firmware, and clear documentation to deliver a professional‑grade solution for T/R control, VSWR protection, and remote operation.

---

# **Features**

- **Deterministic relay sequencing** for safe T/R switching  
- **High‑speed ADC sampling** for forward and reflected power  
- **VSWR and fast reflected‑spike protection (REF_FAST)**  
- **Continuous safety monitoring** with immediate SAFE‑mode entry  
- **Guided Calibration Wizard** for any coupler or detector  
- **Human‑friendly CLI** with history navigation  
- **Machine‑friendly event stream** for automation and logging  
- **Non‑volatile configuration storage (LittleFS)**  
- **Serial interface** for local or remote control  
- **Modular, maintainable firmware architecture**  

---

# **Why Another Sequencer?**

Existing RF sequencers, including the influential **W1GHZ Mark 5**, proved that microcontroller‑based protection can outperform analog timing chains. But modern stations demand more: deterministic firmware behavior, structured telemetry, remote monitoring, fault latching, and safe operation in unattended or tower‑top installations.

This project was created to provide:

- A **fully deterministic** main‑loop architecture  
- **Sub‑millisecond protection paths** (REF_FAST)  
- A **machine‑readable event stream** for dashboards and automation  
- A **human‑readable logging channel** for operators  
- A **calibration system** that adapts to any coupler  
- A **robust SAFE‑mode model** suitable for remote sites  

In short: it’s a **next‑generation MCU sequencer**, built on the ideas that came before but engineered for today’s RF environments.

---

# **Design Lineage & Acknowledgments**

The architectural foundation of this project is inspired in part by the pioneering work of **Paul Wade, W1GHZ**, whose **Mark 5 sequencer** introduced the idea of a microcontroller‑based RF sequencer with a dedicated **SAFE MODE**. His design demonstrated that deterministic firmware control could outperform analog protection chains, especially in microwave and remote installations.

This RF‑Sequencer builds on that lineage with modern timing guarantees, multi‑fault capture, structured event output, and a more comprehensive protection model. Portions of the RF measurement and protection approach, particularly the handling of forward/reflected power and the protection envelope, trace their roots to the engineering principles documented in W1GHZ’s sequencer work.

---

# **Hardware Philosophy**

The hardware is designed with the same priorities as the firmware: **determinism, robustness, and field reliability**. Every component choice and signal path reflects the needs of remote, high‑power, and microwave‑band installations where failure is not an option.

Key principles include:

- **Fail‑safe defaults**  
  All control lines, relay drivers, and protection inputs are biased toward safe states. A reboot, brown‑out, or cable failure must never energize the PA or leave the RF path undefined.

- **Clean, unambiguous relay control**  
  Relay drivers are designed for predictable switching with proper flyback handling, ensuring repeatable timing and long relay life, even with large, slow, or high‑current RF relays.

- **Transparent RF sensing**  
  The forward/reflected power interface is intentionally simple and compatible with a wide range of directional couplers and detectors. The firmware’s calibration layer handles scaling, allowing the hardware to remain universal.

- **Noise‑resilient signal paths**  
  Protection inputs, relay feedback, and PA-ready lines are conditioned for long cable runs and electrically noisy environments typical of tower‑top or outdoor enclosures.

- **No hidden behavior**  
  There are no analog timing chains, RC delays, or “mystery” interactions. All sequencing and protection logic is implemented in firmware, where it is deterministic, inspectable, and fully documented.

This philosophy ensures that the hardware and firmware form a coherent system: predictable, maintainable, and suitable for unattended or remote operation.

[Hardware Philosophy](./img/hw-philosophy.png)
---

# **Repository Structure**

```
RF-Sequencer/
│
├── docs/
│   ├── architecture/      # Architecture overview and details
│   ├── firmware/          # Info in calibration, cli, protocol, ...
│   ├── quickstart/        # First steps and basic usage
│   ├── troubleshooting/   # Chasing ADC, faults and VSWR issues
│   ├── img                # Image helper
│   └── README.md          # Documentation index
│
├── firmware/                     # Precompiled UF2 firmware binaries
│   ├── README.md                 # Flashing instructions
|   └── RFSequencer_Pico_TYPE.uf2 # Firmware for Pico, PicoW, Pico2, Pico2W
│
├── hardware/              # Wiring, schematics, PCB notes, gerbers
│   └── README.md          # Index, setup, ...
│
├── LICENSE_FIRMWARE.md    # Firmware license (binary-only)
├── LICENSE_HARDWARE.md    # Hardware license (CERN OHL-P)
├── LICENSE_NOTICE.md      # Human-readable licensing summary
└── CHANGELOG.md
```

Hardware documentation (schematics, PCB, wiring) will be added in a future revision.

---

# **Getting Started**

### **1. Download the Firmware**

Precompiled UF2 binaries are available in:

```
firmware/
```

or via GitHub Releases.

### **2. Flash the Device**

The Pico/Pico 2 appears as a USB mass‑storage bootloader.
Drag‑and‑drop the `.uf2` file, or use the helper script in:

```
upload_firmware/
```

### **3. Connect via Serial or USB**

The RF‑Sequencer exposes two communication interfaces: the **USB virtual serial port** and the **hardware UART**. Most users will use the USB connector, which provides both command input and event logging.

Any terminal program (PuTTY, KiTTY, minicom, screen, etc.) can connect using:

- 115200 baud
- 8N1
- Local echo off

The hardware UART is primarily intended for remote or embedded installations. It outputs the continuous event stream but also accepts commands, making it suitable for integration with external controllers.

When the connection is established, the CLI prompt appears:

```
>
```

---

# **Calibration Wizard**

The sequencer includes a guided calibration mode that adapts to any directional coupler or detector.

Start it with:

```
> CALIBRATE
```

A walkthrough is available in:

```
docs/quickstart.md
```

---

# **Documentation**

All documentation is located under:

```
docs/
```

Start with:

```
docs/README.md
```

For a complete description of the internal firmware design, see:

```
docs/architecture/firmware-architecture.md
```

This includes:

- ADC subsystem
- Calibration layer
- Protection engine
- State machine
- Relay sequencing
- Event generator
- Configuration storage
- Main execution loop
- Design rationale

---

# **Firmware**

The firmware is modular, deterministic, and designed for reliability.
It implements:

- Continuous ADC sampling and filtering
- Calibration and scaling
- Protection logic (VSWR, REF_FAST, ADC range, configuration integrity)
- A deterministic state machine
- A non‑blocking relay sequencing engine
- A simple, robust CLI
- A machine‑friendly event stream
- Persistent configuration storage on LittleFS

The firmware is distributed as **precompiled UF2 binaries**.
Source code is **not included** in this repository.

---


# **License**

This project uses a dual‑license model:

- **Firmware:** binary‑only, see `LICENSE_FIRMWARE.md`
- **Hardware:** CERN OHL‑P, see `LICENSE_HARDWARE.md`

A human‑readable summary is provided in:

```
LICENSE_NOTICE.md
```
