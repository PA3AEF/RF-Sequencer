# **RF Sequencer Event Protocol Specification**  
**Version 1.0 — PA3AEF**

This document defines the machine‑readable event protocol emitted by the RF Sequencer over the SERIAL interface.  
The protocol is designed to be deterministic, lightweight, and easy to parse on any platform, enabling robust remote operation. 

---

# **1. Overview**

The RF Sequencer outputs a continuous stream of ASCII text events over SERIAL.  
Each event is a single comma‑separated line describing:

- ADC measurements  
- Fault conditions  
- State transitions  
- Configuration changes  
- Boot information  

The protocol is:

- **Line‑oriented** (one event per line)  
- **CSV‑style** (comma‑separated fields)  
- **ASCII‑encoded**  
- **Extensible** without breaking existing parsers  

---

# **2. Transport Layer**

## **2.1 Physical Layer**
- SERIAL (UART0 3.3V logic)
- Default baud rate: **115200**
- Format: **8N1** (8 data bits, no parity, 1 stop bit)
- No flow control

## **2.2 Framing**
- Each event ends with a single newline (`\n`)
- No carriage return (`\r`) is emitted

## **2.3 Encoding**
- ASCII text only  
- No binary payloads  

---

# **3. Event Format**

Every event follows this structure:

```
EVENT,<CATEGORY>,<NAME>,<VALUE>
```

Where:

| Field        | Description |
|--------------|-------------|
| `EVENT`      | Literal prefix identifying a protocol line |
| `<CATEGORY>` | Event class (ADC, FAULT, STATE, CONFIG, BOOT) |
| `<NAME>`     | Identifier within the category |
| `<VALUE>`    | Numeric or symbolic value |

No field contains commas.

---

# **4. Event Categories**

## **4.1 ADC Events**
Emitted periodically when ADC sampling is active.

### **Format**
```
EVENT,ADC,<CHANNEL>,<VALUE>
```

### **Channels**
| Channel     | Description |
|-------------|-------------|
| `FWD`       | Forward power ADC reading |
| `REF`       | Reflected power ADC reading |
| `REF_FAST`  | Fast comparator reading |

### **Examples**
```
EVENT,ADC,FWD,741.8
EVENT,ADC,REF,788.2
EVENT,ADC,REF_FAST,809
```

---

## **4.2 FAULT Events**
Emitted when a protection mechanism trips.

### **Format**
```
EVENT,FAULT,<FAULT_NAME>,LATCHED
```

### Supported Fault Names

| Fault Name        | Meaning |
|-------------------|---------|
| `VSWR_LIMIT`      | VSWR exceeded configured limit |
| `REF_FAST_LIMIT`  | Fast reflected-power limit exceeded |
| `PA_RDY_TIMEOUT`  | PA did not assert PA_RDY in time |
| `RELAY_FAILURE`   | Relay dropped unexpectedly during PA_ON or TX |
| `RELAY_TIMEOUT`   | Relay did not reach commanded state in time |
| `ADC_RANGE`       | ADC reading out of valid range (sensor/wiring fault) |
| `CONFIG_ERROR`    | Persistent configuration invalid or corrupted |

### **Examples**
```
EVENT,FAULT,VSWR_LIMIT,LATCHED
EVENT,FAULT,REF_FAST_LIMIT,LATCHED
EVENT,FAULT,PA_RDY_TIMEOUT,LATCHED
```
### **Fault Emission Rules**
- Only the first detected fault emits a FAULT event and triggers SAFE MODE.
- Additional faults detected during the SAFE MODE transition are recorded internally for diagnostics but do not generate additional FAULT events.
- Fault names correspond directly to firmware fault codes.
Fault names correspond to firmware fault codes.  

---

## **4.3 STATE Events**
Emitted when the sequencer changes state.

### **Format**
```
EVENT,STATE,<STATE_NAME>
```

### **Examples**
```
EVENT,STATE,RX_ENABLE
EVENT,STATE,TX_ENABLE
EVENT,STATE,SAFE_MODE
```

---

## **4.4 CONFIG Events**
Emitted when configuration values change.

### **Format**
```
EVENT,CONFIG,<KEY>,<VALUE>
```

### **Examples**
```
EVENT,CONFIG,VSWR_LIMIT,2.50
EVENT,CONFIG,REF_FAST_LIMIT,800
EVENT,CONFIG,RELAY_TIMEOUT,50
```

---

## **4.5 BOOT Events**
Emitted once at startup.

### **Format**
```
EVENT,SYS,BOOT,<VERSION>
```

### **Example**
```
EVENT,SYS,BOOT,1.0
```

---

# **5. Timing and Frequency**

## **5.1 ADC Events**
- Emitted at the firmware’s ADC sampling rate  
- Typically 10–20 Hz  
- Not guaranteed to be periodic  

## **5.2 FAULT Events**
- Emitted once per fault occurrence  
- Never repeated while latched  

## **5.3 STATE Events**
- Emitted only on transitions  

## **5.4 CONFIG Events**
- Emitted only when values change  

---

# **6. Parsing Rules**

## **6.1 Line‑oriented**
Each event is one complete line ending in `\n`.

## **6.2 CSV‑style**
Split on commas:

```python
type, category, name, value = line.split(',')
```

## **6.3 Forward compatibility**
Parsers should:

- Accept new categories  
- Ignore unknown categories  
- Ignore extra fields if added in future versions  

### **Robust parsing pseudocode**
```python
fields = line.split(',')
if fields[0] != "EVENT":
    ignore()

category = fields[1]

if category == "ADC":
    channel = fields[2]
    value = float(fields[3])

elif category == "FAULT":
    fault = fields[2]
    status = fields[3]

elif category == "STATE":
    state = fields[2]

elif category == "CONFIG":
    key = fields[2]
    value = fields[3]
```

---

# **7. Error Handling**

Parsers should ignore:

- Empty lines  
- Lines not starting with `EVENT`  
- Lines with fewer than 3 fields  
- Malformed CSV  

No error events are emitted by the sequencer.

---

# **8. Example Session**

```
EVENT,BOOT,2602062249-1.0
EVENT,STATE,RX_ENABLE
EVENT,ADC,FWD,741.8
EVENT,ADC,REF,788.2
EVENT,ADC,REF_FAST,809
EVENT,FAULT,VSWR_LIMIT,LATCHED
EVENT,STATE,SAFE_MODE
EVENT,CONFIG,VSWR_LIMIT,2.50
```

---

# **9. Versioning**

This protocol is versioned independently of firmware.

- **Major version** increments when breaking changes occur  
- **Minor version** increments when new categories or fields are added  

Current protocol version: **1.0**

---



