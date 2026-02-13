# **RF‑Sequencer Documentation Index**

This folder contains all documentation for the RF‑Sequencer project: quickstart guides, firmware architecture, calibration procedures, and background information about the design. It serves as the central entry point for users, developers, and operators who want to understand how the system works and why it was built the way it is.

---

# **1. Background & Motivation**

The RF‑Sequencer was created to solve a practical problem: safely and reliably controlling RF hardware in remote or hard‑to‑reach locations. In SHF systems — especially dish‑feedpoint boxes, tower‑top amplifiers, and remote stations — timing accuracy and protection are critical. A single relay glitch or delayed protection response can damage expensive hardware or interrupt operation during contests or EME sessions.

Commercial solutions exist, but they are often inflexible, opaque, or not designed for remote automation.  
This project aims to provide:

- **Deterministic timing**  
- **Transparent behavior**  
- **Strong protection guarantees**  
- **Remote‑friendly control**  
- **Hardware‑agnostic calibration**  

All in a compact, reliable controller that can live at the feedpoint without supervision.

---

# **2. Why Deterministic Timing Matters**

RF sequencing is fundamentally a timing‑critical problem.  
Relays must switch in the correct order, with precise break‑before‑make windows. Protection must run continuously. ADC sampling must be stable. SAFE mode must engage instantly when needed.

To guarantee this, the firmware uses a **high‑frequency cooperative main loop** instead of interrupts or an RTOS.  
This ensures:

- No hidden execution paths  
- No jitter from ISR pre‑emption  
- No unpredictable delays  
- Perfectly repeatable behavior  

Determinism is the foundation of the entire architecture.

---

# **3. Platform Selection: Pico/Pico 2**

Early prototypes were built on smaller MCUs such as the Arduino NANO and Adafruit ItsyBitsy. While functional, they lacked several qualities required for a safety‑critical RF controller:
- Limited performance headroom  
- Less predictable timing under load  
- More fragile bootloader and recovery process  
- Less comfortable development workflow for rapid iteration  
- Tighter memory constraints for calibration, CLI, and event logging  

The **Raspberry Pi Pico / Pico 2** family offered clear advantages:
- Fast, deterministic dual‑core RP2040  
- Excellent GPIO performance  
- Robust USB bootloader with easy recovery  
- Plenty of flash and RAM  
- Stable ADC subsystem  
- A development experience that encourages experimentation and refinement  

The Pico platform ultimately enabled the clean, modular architecture the project uses today.

---

# **4. Documentation Structure**

```
docs/
│
├── architecture/
│   └── firmware-architecture.md   # Full internal design specification
│
├── quickstart.md                  # First steps and basic usage
│
└── README.md                      # This document
```

Additional sections (hardware, troubleshooting, wiring) will be added in future revisions.

---

# **5. Where to Start**

If you are new to the project:

1. **Quickstart Guide**  
   `docs/quickstart.md`  
   Learn how to flash the firmware, connect to the CLI, and perform your first calibration.

2. **Firmware Architecture**  
   `docs/architecture/firmware-architecture.md`  
   A deep dive into the internal design, timing model, protection logic, and rationale.

3. **Repository README**  
   The top‑level overview of the project, features, and repository layout.

---

# **6. Future Documentation**

Upcoming additions will include:

- Hardware schematics and wiring  
- PCB documentation  
- Detailed troubleshooting guides  
- Advanced automation examples  
- Remote‑operation best practices for SHF systems  
