# Command-Line Interface (CLI)

The RF Sequencer provides a simple, robust command-line interface over USB CDC.
It is designed for human interaction using any serial terminal (Arduino IDE
Serial Monitor, PuTTY, CoolTerm, VSCode Serial Monitor, etc.).

The CLI supports:

- Minimal line editing
- Backspace
- UP/DOWN history recall
- One command per line
- Case-insensitive commands

No ANSI escape sequences are used, ensuring compatibility with all terminals.

---

## Prompt

The CLI prompt is:

```
> 
```

Commands are executed when the user presses Enter.

---

## Command Reference

### **HELP**
Show help text.

```
HELP
```

---

### **STATUS**
Show system status.
#### Fault History
The `STATUS` command includes a `FAULT_HISTORY` section listing up to the five most recent
faults detected by the firmware. Entries are shown in reverse chronological order (most
recent first).

Example:
```
FAULT_HISTORY:
1. VSWR_LIMIT — VSWR limit exceeded
2. PA_RDY_TIMEOUT — PA did not become ready in time
3. RELAY_TIMEOUT — Relay did not reach expected state in time


```
STATUS
```

---

### **RESET**
Clear faults, exit SAFE MODE, and return to RX state.

```
RESET
```

---

### **FACTORY_RESET**
Restore all configuration values to defaults and save them.

```
FACTORY_RESET
```

---

### **GET CONFIG**
Display all configuration values.

```
GET CONFIG
```

---

## SET Commands

### **SET SAFEMODE ON|OFF**
Force or clear SAFE MODE.

```
SET SAFEMODE ON
SET SAFEMODE OFF
```

---

### **SET VSWR_LIMIT <float>**
Set the VSWR protection threshold.

```
SET VSWR_LIMIT 2.5
```

---

### **SET REF_FAST_LIMIT <int>**
Set the fast reflected-power limit.

```
SET REF_FAST_LIMIT 800
```

---

### **SET RELAY_TIMEOUT <ms>**
Set the relay transition timeout in milliseconds.

```
SET RELAY_TIMEOUT 50
```

---

### **SET PA_TIMEOUT <ms>**
Set the PA_RDY wait timeout in milliseconds.

```
SET PA_TIMEOUT 200
```

---

### **SET INPUT <NAME> ON|OFF**
Enable or disable an input.

Valid names:

- PTT  
- RFSENSE  
- PA_RDY  
- RELAY_FB  
- VSWR  
- REF  
- AUX1  
- AUX2  
- AUX3  

Example:

```
SET INPUT PTT ON
```

---

### **SET SERIAL ON|OFF**
Enable or disable the UART0 machine event stream.

```
SET SERIAL ON
SET SERIAL OFF
```

---

### **SET LOGGING ON|OFF**
Enable or disable human-readable USB logging.

```
SET LOGGING ON
```

---

## Notes

- Commands are case-insensitive.
- Unknown commands return `ERR UNKNOWN_CMD`.
- Invalid arguments return `ERR BAD_ARGS` or `ERR BAD_STATE`.

```
