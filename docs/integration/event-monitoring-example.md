# **Event Monitoring Examples**

The examples here demonstrate how an external system can consume the RF‑Sequencer’s event stream using a simple HTML page. or Python script. It illustrates the structure of emitted events and shows how they can be displayed or processed by a monitoring application.

For architectural integration guidance, see:

- `integration-examples.md`  
- `firmware-architecture.md`  
- `overview.md`

---

## **Purpose**

The purpose of this example is to:

- show the structure of sequencer events  
- demonstrate how an external system can receive and display them  
- provide a minimal reference implementation for developers  

This example does **not** define any sequencing logic, protection behavior, or timing rules.  
It is purely an illustration of event consumption.

---

## **Event Format**

Events are emitted as structured JSON objects.  
Each event contains:

- a **type** identifying the event category  
- a **timestamp**  
- a **payload** containing event‑specific data  

Example event:

```json
{
  "type": "state",
  "timestamp": "2025-01-01T12:00:00.000Z",
  "payload": {
    "from": "receive",
    "to": "transmit"
  }
}
```

Other event types follow the same structure:

- `state`  
- `protection`  
- `relay`  
- `measurement`  
- `configuration`

---

## **Minimal HTML Event Monitor**

The following HTML example connects to the sequencer, receives events, and displays them in a simple log window.  
It is intentionally minimal and suitable as a starting point for GUIs, dashboards, or automation tools.

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>RF‑Sequencer Event Monitor</title>
  <style>
    body { font-family: sans-serif; background: #111; color: #eee; }
    #log { white-space: pre; font-family: monospace; padding: 1rem; }
  </style>
</head>
<body>

<h2>RF‑Sequencer Event Monitor</h2>
<button id="connect">Connect</button>
<div id="log"></div>

<script>
  const log = msg => {
    document.getElementById('log').textContent += msg + "\n";
  };

  document.getElementById('connect').onclick = async () => {
    try {
      const port = await navigator.serial.requestPort();
      await port.open({ baudRate: 115200 });

      const reader = port.readable.getReader();
      log("Connected to RF‑Sequencer");

      while (true) {
        const { value, done } = await reader.read();
        if (done) break;

        const text = new TextDecoder().decode(value);
        text.trim().split("\n").forEach(line => {
          try {
            const event = JSON.parse(line);
            log(JSON.stringify(event, null, 2));
          } catch {
            log("Invalid JSON: " + line);
          }
        });
      }
    } catch (err) {
      log("Connection error: " + err);
    }
  };
</script>

</body>
</html>
```

This example:

- connects to the sequencer  
- reads the event stream  
- parses each line as JSON  
- displays events in a readable format  

It does not implement buffering, reconnection, or filtering.  
Those concerns are left to the integrating system.

---

# **Python Event Monitor**

This example demonstrates how a Python application can consume the RF‑Sequencer’s event stream over a serial connection.  
It illustrates how to read line‑delimited JSON events, parse them, and process them in a deterministic loop.

For architectural integration guidance, see:

- `integration-examples.md`  
- `event-monitoring-example.md`  
- `firmware-architecture.md`

---

## **Purpose**

The purpose of this example is to:

- show how to read the sequencer’s event stream  
- parse structured JSON events  
- provide a minimal reference for Python‑based monitoring systems  

This example does **not** implement buffering, reconnection, or GUI logic.  
It is intentionally minimal and suitable as a foundation for logging, dashboards, or automation tools.

---

## **Python Example**

```python
import json
import serial

def log(msg):
    print(msg)

def main():
    # Adjust the serial port to match your system
    port = serial.Serial('/dev/ttyUSB0', 115200, timeout=1)
    log("Connected to RF‑Sequencer")

    buffer = ""

    while True:
        data = port.read(128).decode(errors='ignore')
        if not data:
            continue

        buffer += data

        # Process complete lines
        while "\n" in buffer:
            line, buffer = buffer.split("\n", 1)
            line = line.strip()
            if not line:
                continue

            try:
                event = json.loads(line)
                log(json.dumps(event, indent=2))
            except json.JSONDecodeError:
                log(f"Invalid JSON: {line}")

if __name__ == "__main__":
    main()
```

---

## **Integration Notes**

- The sequencer emits one JSON event per line.  
- Events are self‑contained and do not require correlation.  
- External systems should not infer timing or sequencing from event order.  
- Protection events should be logged and acted upon immediately.  
- Relay events reflect the configured topology and should not be interpreted as timing signals.  

---

## **Further Reading**

- **Integration Guide**  
  `integration-examples.md`

- **Firmware Architecture**  
  `firmware-architecture.md`

- **Relay Subsystem Architecture**  
  `relay-architecture.md`

