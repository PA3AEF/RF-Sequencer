
# RF‑Sequencer Firmware Documentation

This folder contains all firmware‑related documentation for the **RF‑Sequencer**, including update instructions, firmware packaging details, and notes about how the sequencer firmware interacts with the hardware.

The RF‑Sequencer firmware is built for the **Raspberry Pi Pico / Pico2** platform, chosen for its reliability, speed, and easy UF2‑based recovery mechanism. Even if the bootloader becomes corrupted, the Pico can always be restored by copying a UF2 file to the device.
[1](https://github.com/PA3AEF)


---

## Contents

### **1. Firmware Update Guide**
`firmware-update.md`
Explains how to update the sequencer using the Pico/Pico2 built‑in USB bootloader:

- Entering BOOTSEL mode
- Drag‑and‑drop `.uf2` flashing
- Reboot behavior
- Verifying firmware with the CLI (`STATUS`)
- Preserved settings & factory reset commands

This is the primary resource for performing firmware updates.

---

## About the Firmware

The RF‑Sequencer firmware:

- Implements a clear, deterministic state machine for RX/TX transitions ([statemachine.md](./statemachine.md))
- Monitors interlocks, relay feedback, reflected power, VSWR, and amplifier readiness
- Provides a real‑time CLI (STATUS, CFG, faults, event stream)
([cli.md](./cli.md))
- Enforces Safe Mode when dangerous RF conditions occur
- Runs entirely on the Pico/Pico2 microcontroller platform

The firmware is distributed as **binary‑only** (`.uf2`) and licensed under **0BSD**, according to the project’s firmware license.

---

## Firmware Files

All firmware releases appear as `.uf2` binaries in:

- `firmware/releases/`
- The GitHub Releases page (if provided by the maintainer)

Each UF2 file is flashed directly via BOOTSEL mode.

---

## Configuration Persistence

Configuration values (timeouts, relay behaviors, VSWR limits, calibration values, interlock states) are stored in non‑volatile memory. Firmware updates do **not** erase configuration.

To reset the device to factory defaults:

```
FACTORY_RESET
```

---

## Reporting Issues

If you encounter:

- flashing failures
- unexpected CLI behavior
- relay timing problems
- bootloader issues

please open an Issue in the main RF‑Sequencer repository.

---

## License

The firmware is licensed under **0BSD**, and the hardware under **CERN‑OHL‑P**, as documented in the repository root.˜˜
