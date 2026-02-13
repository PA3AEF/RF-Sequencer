# **RF‑Sequencer Integration Examples**  
**Practical code samples for automation, monitoring, and visualization**

This supplemental document provides small, self‑contained examples showing how external systems can consume the RF‑Sequencer’s Serial (UART0) event stream.  
These examples are intentionally minimal and framework‑free, making them easy to adapt for dashboards, logging systems, or automated controllers.

---

# **1. Parsing Events in Python (JSON‑Friendly)**

The UART0 event stream uses a simple, stable format:

```
EVENT,<CATEGORY>,<NAME>,<VALUE>
```

The following Python snippet listens to the event stream, parses each line, and converts it into JSON‑friendly dictionaries suitable for dashboards or monitoring tools.

```python
import serial
import json

ser = serial.Serial("/dev/ttyUSB0", 115200, timeout=1)

def parse_event(line):
    # Example input:
    # EVENT,ADC,FWD,123
    parts = line.strip().split(",")
    if len(parts) != 4 or parts[0] != "EVENT":
        return None

    category, name, value = parts[1], parts[2], parts[3]
    return {
        "category": category,
        "name": name,
        "value": float(value) if value.replace('.', '', 1).isdigit() else value
    }

while True:
    raw = ser.readline().decode(errors="ignore")
    evt = parse_event(raw)
    if evt:
        print(json.dumps(evt))
```

### **What you can build with this**

- Real‑time power graphs  
- Fault logging systems  
- State transition dashboards  
- Automated alarms or shutdown logic  
- Integrations with InfluxDB, Grafana, Node‑RED, or MQTT  

---

# **2. Minimal Web Dashboard (HTML + JavaScript)**

This example demonstrates how a browser can directly visualize the event stream using the Web Serial API.  
Save the file as `dashboard.html`, open it in Chrome/Edge, click **Connect**, and select the USB serial device.

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>RF‑Sequencer Dashboard</title>
<style>
  body { font-family: sans-serif; margin: 20px; }
  #log { font-family: monospace; white-space: pre; height: 200px; overflow-y: scroll; border: 1px solid #ccc; padding: 10px; }
  .value { font-size: 2em; margin: 10px 0; }
</style>
</head>
<body>

<h2>RF‑Sequencer Live Dashboard</h2>
<button id="connect">Connect</button>

<div class="value">Forward Power: <span id="fwd">0</span></div>
<div class="value">Reflected Power: <span id="ref">0</span></div>
<div class="value">State: <span id="state">—</span></div>

<h3>Raw Event Log</h3>
<div id="log"></div>

<script>
let port;
let reader;

document.getElementById("connect").onclick = async () => {
  port = await navigator.serial.requestPort();
  await port.open({ baudRate: 115200 });

  reader = port.readable.getReader();
  const decoder = new TextDecoder();

  let buffer = "";

  while (true) {
    const { value, done } = await reader.read();
    if (done) break;

    buffer += decoder.decode(value);
    let lines = buffer.split("\n");
    buffer = lines.pop();

    for (let line of lines) {
      handleEvent(line.trim());
    }
  }
};

function handleEvent(line) {
  if (!line.startsWith("EVENT")) return;

  document.getElementById("log").textContent += line + "\n";

  const parts = line.split(",");
  if (parts.length !== 4) return;

  const [_, category, name, value] = parts;

  if (category === "ADC") {
    if (name === "FWD") document.getElementById("fwd").textContent = value;
    if (name === "REF") document.getElementById("ref").textContent = value;
  }

  if (category === "STATE") {
    document.getElementById("state").textContent = name;
  }
}
</script>

</body>
</html>
```

### **What this dashboard demonstrates**

- Live forward/reflected power display  
- Live state transitions  
- Raw event log for debugging  
- Zero dependencies — just a browser  
- Direct UART0 integration via Web Serial API  
- Instant visualization of the sequencer’s behavior  

---

# **3. Linking From the Architecture Document**

In `firmware-architecture.md`, you can reference this supplemental doc like so:

```
For practical examples of consuming the UART0 event stream, see:

integration-examples.md
```

---

If you want, I can also add a **Node‑RED flow**, a **Grafana dashboard template**, or a **MQTT bridge example** to expand this supplemental doc even further.
