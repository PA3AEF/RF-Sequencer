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

> 
> help
RF Sequencer - PA3AEF 2604111322.1.2
Available commands:
  HELP                             Show this help text
  STATUS                           Show system status
  PERF                             Show performance timing statistics
  RESET                            Clear faults and return to RX
  FACTORY_RESET                    Restore defaults and save config
  CALIBRATE                        Starts guided VSWR calibration
  GET CONFIG                       Show all configuration values
  LOGLEVEL <L>                     Set USB log level: INFO|WARN|ERROR
  SET SAFEMODE ON|OFF              Force or clear safemode
  SET VSWR_LIMIT <float>           Set VSWR protection threshold
  SET REF_FAST_LIMIT <int>         Set fast reflected-power limit
  SET RELAY_MODE <NORMAL|LATCHED>  Set relay switching type
  SET HAS_RX_RELAY ON|OFF          Enable/disable seperate RX relay
  SET RELAY_TIMEOUT <ms>           Set relay transition timeout
  SET PA_TIMEOUT <ms>              Set PA_RDY wait timeout
  SET INPUT <NAME> ON|OFF          Enable/disable input
                                   <PTT|RFSENSE|PA_RDY|RELAY_FB|VSWR|REF|AUX1-2>
  SET EVENTS ON|OFF                Enable/disable Serial machine event stream
  SET LOGGING ON|OFF               Enable/disable USB human-readable logs

> 
```
---
### **STATUS**
Show system status.

The `STATUS` command includes a `FAULT_HISTORY` section listing up to the five most recent
faults detected by the firmware. Entries are shown in reverse chronological order (most
recent first).

```
>

> status
RF SEQUENCER STATUS (LIVE READINGS AND DERIVATIVES)
STATE
  Sequencer State:   RX_ENABLE
  PTT:               Released
  PA Ready:          No
  Relay Position:    RX
  RX Relay Present:  No
  Relay Mode:        TX-only (normal relay)
  Uptime:            00:55:22
DIGITAL INPUTS
  PTT:               RAW=1  LOGICAL=0  (enabled)
  RFSENSE:           RAW=1  LOGICAL=0  (disabled)
  PA_RDY:            RAW=0  LOGICAL=0  (disabled)
  RELAY_FB:          RAW=0  LOGICAL=0  (disabled)
  REF_FAST:          RAW=0  LOGICAL=1  (disabled)
  AUX1:              RAW=0  LOGICAL=1  (disabled)
  AUX2:              RAW=1  LOGICAL=0  (disabled)
ANALOG INPUTS (RAW ADC)
  FWD_ADC:           885
  REF_ADC:           908
  VTEST_ADC:         822
COUPLER VOLTAGES (CALIBRATED)
  FWD_VOLTS:         0.71 V
  REF_VOLTS:         0.73 V
DERIVED VALUES
  VSWR:              9998.67
  REF_FAST:          OK
OUTPUTS
  TX_ENABLE:         OFF
  RX_ENABLE:         ON
  PA_ENABLE:         OFF
  TX_RELAY:          OFF
  RX_RELAY:          OFF
PROTECTIONS
  Fault Latched:     NO
  VSWR Protect:      DISABLED
  REF_FAST Protect:  DISABLED
  RFSENSE:           No RF
  PA_RDY:            Ignored
  RELAY_FB:          Ignored
  Temperature:       Normal
FAULT_HISTORY:
1. VSWR_LIMIT — VSWR limit exceeded
2. PA_RDY_TIMEOUT — PA did not become ready in time
3. RELAY_TIMEOUT — Relay did not reach expected state in time
>
```

---
### **RESET**
Clear faults, exit SAFE MODE, and return to RX state.

```
> reset
STATE=RX_ENABLE
RESET
OK RESET_DONE
> 
```

---

### **FACTORY_RESET**
Restore all configuration values to defaults and save them.

```
> factory_reset
CONFIG SAVED
CONFIG FACTORY_RESET
OK FACTORY_RESET
> 
```

---

### **GET CONFIG**
Display all configuration values.

```
> get config
CONFIG:
  VSWR_LIMIT = 2.50
  REF_FAST_LIMIT = 1000
  FWD_CALIBRATION: 1.0000
  REF_CALIBRATION: 1.0000
  RELAY_MODE = 0
  RELAY_TIMEOUT = 150
  PA_TIMEOUT = 200
  INPUT PTT = ON
  INPUT RFSENSE = OFF
  INPUT PA_RDY = OFF
  INPUT RELAY_FB = OFF
  INPUT VSWR = OFF
  INPUT REF = OFF
  INPUT AUX1 = OFF
  INPUT AUX2 = OFF
  SERIAL EVENTS = OFF
  LOGGING USB = ON
  LOGLEVEL = INFO
> 
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

Example:

```
SET INPUT PTT ON
```

---

### **SET EVENTS ON|OFF**
Enable or disable the UART0 machine event stream.

```
SET EVENTS ON
SET EVENTS OFF
```

---

### **SET LOGGING ON|OFF**
Enable or disable human-readable USB logging.

```
SET LOGGING ON
```


