# **Installation & Firmware Flashing**

This guide explains how to install the RF‑Sequencer firmware on a Raspberry Pi Pico or Pico 2.

## **Requirements**

- Raspberry Pi Pico or Pico 2  
- USB cable  
- Latest `.uf2` firmware file  
- Terminal program (minicom, PuTTY, screen, etc.)

Firmware binaries are located in:

```
../../firmware/
```

or in GitHub Releases.

---

## **Enter Bootloader Mode**

1. Hold the **BOOTSEL** button on the Pico.  
2. Plug in the USB cable.  
3. Release the button.

The device appears as a USB mass‑storage drive.

---

## **Flash the Firmware**

Copy the `.uf2` firmware file onto the bootloader drive (RPI-RP2).  
The Pico will reboot automatically and start the RF‑Sequencer firmware.

The Raspberry Pi Pico and other RP2350‑based boards are extremely difficult to brick. Their USB bootloader resides in a **32 KB read‑only ROM**, so even if the flash becomes corrupted, the device can always be recovered.

If you ever need to fully reset the board, you can restore it by copying **`flash_nuke.uf2`** to the bootloader drive. This erases all flash contents (including configuration data) but returns the board to a clean state, ready for flashing the Sequencer firmware..

More information and the official recovery image are available at:  
[Resetting Flash Memory](https://www.raspberrypi.com/documentation/microcontrollers/pico-series.html#resetting-flash-memory)

---

## **After Flashing**

The RF‑Sequencer firmware starts immediately.  
Continue with:

```
first-run.md
```
