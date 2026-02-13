# **RF‑Sequencer Firmware**

This folder contains the **precompiled firmware binaries** for the RF‑Sequencer.  
The firmware is distributed as UF2 files and can be flashed directly to the microcontroller without installing any development tools.

If you only want to operate the sequencer, this is the only folder you need.

---

## **Firmware Releases**

Precompiled firmware files are located in: firmware/releases


Each release contains:

- `RF-Sequencer-vX.Y.Z.uf2` — main firmware image  
- `CHANGELOG.md` entry describing what changed  

You can also download firmware from the GitHub Releases page.

---

## **Flashing the Firmware**

The RF‑Sequencer uses a USB mass‑storage bootloader.  
To flash the firmware:

1. Connect the device to your computer via USB.
2. Enter bootloader mode:
   - Press and hold the BOOTSEL button  
   - Plug in the USB cable  
   - Release the button  
3. A new drive will appear (e.g., `RPI-RP2`).
4. Drag‑and‑drop the `.uf2` file onto the drive.
5. The device will reboot automatically.

Detailed instructions are available in: docs/quickstart/firmware-update.md


---

## **Configuration & Operation**

After flashing, connect to the device using any serial terminal:

- **115200 baud**
- **8N1**
- Local echo off

You will see the CLI prompt: >



Documentation for all commands is available in: docs/firmware/cli.md


---

## **Notes**

- Firmware source code is not included in this repository.
- All configuration values are stored in non‑volatile memory.
- You can safely update firmware without losing your settings.

---

If you need help updating or recovering the device, refer to the troubleshooting section:
`docs/troubleshooting/`
