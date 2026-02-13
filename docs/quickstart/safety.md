
# **Safety Notes**

The RF‑Sequencer is designed for deterministic timing and continuous protection. These notes highlight important considerations for safe installation and operation.

---

## **Protection Is Always Active**
The protection subsystem runs every loop cycle and can force SAFE mode immediately.

Protection includes:

- VSWR limits  
- Fast reflected‑power protection (REF_FAST)  
- ADC range checks  
- Configuration integrity checks  

SAFE mode cannot be disabled.

---

## **SAFE Mode Behavior**

When a fault is detected:

- All RF paths are forced to a safe state  
- Relays unwind in a controlled sequence  
- Events are emitted to the CLI  
- The heartbeat LED changes pattern  

SAFE mode remains active until cleared by the operator through the
``` > RESET``` command.

---

## **Relay Sequencing**

The relay engine enforces:

- Break‑before‑make timing  
- Deterministic ordering  
- Non‑blocking transitions  

This prevents relay overlap, which is a common cause of T/R hardware damage.

---

## **Remote Operation Notes**
The sequencer is designed for remote installations such as:

- SHF dish‑feedpoint boxes  
- Tower‑top amplifiers  
- Remote shacks  

Recommendations:
- Enable event logging (`EVENTS ON`)  
- Use shielded cabling for coupler signals  
- Ensure stable power and grounding  
- Keep relay wiring short and clean  

---

## **Hardware Notes**
Full hardware documentation (schematics, wiring, PCB) will be added in a future revision.

For now:
- Follow the wiring diagram included with your hardware  
- Ensure coupler outputs stay within ADC limits  
- Verify relay coil polarity and ratings  

---

## **Learn More**

For the full internal architecture, see:

```
../architecture/firmware-architecture.md
```
