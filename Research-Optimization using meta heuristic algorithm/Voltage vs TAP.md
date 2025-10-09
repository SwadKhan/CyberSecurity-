
Perfect — this is *exactly* what you should be doing now 👏
Let’s go through this script **line-by-line and concept-by-concept**, so you understand *every single step* — what it’s doing in OpenDSS, how it gets the data, and why it matters for your smart grid + cybersecurity research.

---

## 🧠 Overall Goal of the Script

The script:

1. Loads the IEEE 123-bus test feeder in OpenDSS.
2. Runs a **power flow** simulation.
3. Extracts **regulator data** (like tap position, target voltage, regulated bus).
4. Calculates **actual voltages** at those regulated buses.
5. Visualizes the results (tap vs voltage).

You’ll use this to see **how voltage regulators are behaving** — which is vital for identifying **critical nodes** and **cyber-vulnerable points** in a smart grid.

---

## ⚙️ Step-by-Step Explanation

---

### 🧩 STEP 1: Load the IEEE 123-bus Feeder

```python
file_path = r"C:\Program Files\OpenDSS\IEEETestCases\123Bus\IEEE123Master.dss"
dss.Text.Command(f'Redirect "{file_path}"')
```

* `file_path` — location of the main OpenDSS circuit file.
  It contains all buses, loads, lines, and equipment.

* `dss.Text.Command("Redirect ...")` — tells OpenDSS to **load** that circuit model.

🧠 Think of this as:
“Open the 123-bus power system model.”

---

### ⚡ STEP 2: Solve the Circuit

```python
dss.Text.Command("Solve")
```

* Runs a **load flow analysis**, which computes:

  * Voltage magnitudes and angles at every bus,
  * Current flow in each line,
  * Power at loads, etc.

🧠 This is like pressing **“Run Simulation”** in a power systems tool.

---

### 🧾 STEP 3: Get All Regulator Names

```python
regulators = dss.RegControls.AllNames()
print("Regulators found:", regulators)
```

* `dss.RegControls.AllNames()` → retrieves a list of **all voltage regulators** in the system.
* In IEEE 123-bus, you’ll get names like:

  ```
  ['creg1a', 'creg2a', 'creg3a', 'creg3c', 'creg4a', 'creg4b', 'creg4c']
  ```

Each one controls the voltage on a transformer tap.

🧠 Regulators = “Automatic tap changers” that adjust transformer turns ratio to keep voltage within a band.

---

### ⚙️ STEP 4: Loop Through Each Regulator

```python
for reg in regulators:
    dss.RegControls.Name(reg)
```

This tells OpenDSS:

> “Make this regulator active — I want to read its properties.”

---

#### ⚡ Get Regulator Properties

```python
tap = dss.RegControls.TapNumber()
vreg = dss.RegControls.ForwardVreg()
band = dss.RegControls.ForwardBand()
trans_name = dss.RegControls.Transformer()
```

| Property        | Meaning                                  | Example      |
| --------------- | ---------------------------------------- | ------------ |
| `TapNumber()`   | Current tap position (integer)           | 6, 4, -2     |
| `ForwardVreg()` | Target regulated voltage (in volts, L-N) | 120.0, 124.0 |
| `ForwardBand()` | Voltage deadband range (in volts)        | 2.0          |
| `Transformer()` | The name of the transformer it controls  | `creg4`      |

🧠 Each regulator *controls one transformer winding* to maintain voltage within ±band of the target.

---

#### 🔍 Find Connected Buses

```python
dss.Transformers.Name(trans_name)
primary_bus = dss.Transformers.Bus(1)
secondary_bus = dss.Transformers.Bus(2)
```

Every transformer (and hence every regulator) connects two buses:

* **Primary bus** (incoming high voltage side)
* **Secondary bus** (outgoing lower voltage side, where regulation happens)

🧠 This gives you the **exact buses** where the regulator is active — that’s where you’ll later measure voltage.

---

#### 📈 Measure Voltage at the Regulated Bus

```python
dss.Circuit.SetActiveBus(secondary_bus)
voltages = dss.Bus.puVmagAngle()[::2]  # Magnitude only
avg_voltage = sum(voltages) / len(voltages)
```

Here’s what happens:

1. You tell OpenDSS to “activate” the secondary bus.
2. `dss.Bus.puVmagAngle()` returns `[V1_pu, angle1, V2_pu, angle2, ...]`.
   So `[::2]` takes only the magnitudes (ignores the angles).
3. Then you average the voltage of all phases at that bus.

💡 Example:
If phase A = 1.02 p.u., B = 1.00 p.u., C = 0.99 p.u.,
then `avg_voltage = (1.02 + 1.00 + 0.99) / 3 ≈ 1.003 p.u.`

---

#### 📋 Store Results for Plotting

```python
regulator_labels.append(f"{reg}\n({secondary_bus})")
tap_data.append(tap)
voltage_data.append(avg_voltage)
```

* Save regulator name + bus for labeling.
* Keep track of tap and voltage values to visualize later.

---

### 🧾 STEP 5: Print the Summary

```python
print("\nRegulator Details:")
for i, reg in enumerate(regulator_labels):
    print(f"{reg}: Tap={tap_data[i]}, Voltage={voltage_data[i]:.3f} p.u.")
```

Output example:

```
creg1a (Bus 150): Tap=6, Voltage=1.018 p.u.
creg4a (Bus 83): Tap=10, Voltage=1.043 p.u.
```

🧠 This tells you:

* Which regulator controls which bus,
* How far each regulator is boosting or reducing voltage.

---

### 📊 STEP 6: Plot Tap and Voltage Levels

```python
fig, ax1 = plt.subplots(figsize=(10, 6))
ax1.bar(regulator_labels, tap_data, color='teal', alpha=0.6, label='Tap Position')
ax1.set_ylabel("Tap Position", color='teal')
ax1.set_xlabel("Regulator (Regulated Bus)")
```

Creates a **bar chart** (blue bars = tap positions).

---

#### Overlay Voltage Line on Secondary Axis

```python
ax2 = ax1.twinx()
ax2.plot(regulator_labels, voltage_data, color='darkred', marker='o', label='Regulated Voltage (p.u.)')
ax2.set_ylabel("Voltage (p.u.)", color='darkred')
```

Adds a **red line plot** for voltage on a separate y-axis.
So you can visually compare how tap changes relate to voltage output.

---

#### Final Formatting

```python
plt.title("Voltage Regulators: Tap Positions and Regulated Bus Voltages")
plt.xticks(rotation=30, ha='right')
plt.grid(axis='x', linestyle='--', alpha=0.4)
fig.tight_layout()
plt.show()
```

Makes the plot clear and readable.

---

## 🧩 Summary of What You’re Seeing

| Element         | Meaning                                                                    |
| --------------- | -------------------------------------------------------------------------- |
| **Bars (blue)** | Current tap position — how much the regulator is boosting/reducing voltage |
| **Line (red)**  | Actual voltage (in per-unit) at the regulated bus                          |
| **X-axis**      | Each regulator and the bus it controls                                     |

🧠 This visual helps you:

* Identify over- or under-voltage conditions.
* See which regulators are most active (high tap values).
* Find critical buses — where small cyber manipulations could destabilize the system.

---

## 🔐 Why This Matters for Cybersecurity + Optimization

Once you can identify how regulators behave:

* You can simulate **false voltage data attacks** (spoofing regulator sensors).
* You can use **metaheuristic algorithms (GA, PSO, etc.)** to:

  * Optimize tap settings for stability.
  * Detect abnormal regulator behavior (sign of cyber intrusion).
  * Reinforce control logic against compromised measurements.

---
Code :


import opendssdirect as dss
import matplotlib.pyplot as plt

# -------------------------------------------------------
# STEP 1: Load the IEEE 123-bus feeder
# -------------------------------------------------------
file_path = r'D:\OpenDSS\IEEETestCases\123Bus\IEEE123Master.dss'
dss.Text.Command(f'Redirect "{file_path}"')

# -------------------------------------------------------
# STEP 2: Solve the circuit
# -------------------------------------------------------
dss.Text.Command("Solve")

# -------------------------------------------------------
# STEP 3: Get all regulator names
# -------------------------------------------------------
regulators = dss.RegControls.AllNames()
print("Regulators found:", regulators)

# -------------------------------------------------------
# STEP 4: Collect regulator details
# -------------------------------------------------------
tap_data = []
voltage_data = []
regulator_labels = []

for reg in regulators:
    dss.RegControls.Name(reg)
    tap = dss.RegControls.TapNumber()
    vreg = dss.RegControls.ForwardVreg()
    band = dss.RegControls.ForwardBand()
    trans_name = dss.RegControls.Transformer()

    # Set active transformer
    dss.Transformers.Name(trans_name)

    # Get bus names safely depending on version
    try:
        # Most recent versions
        bus_names = dss.CktElement.BusNames()
    except Exception:
        # Fallback for older builds
        bus_names = []

    primary_bus = bus_names[0] if len(bus_names) > 0 else "N/A"
    secondary_bus = bus_names[1] if len(bus_names) > 1 else "N/A"

    # Get voltage magnitude (per unit) at secondary (regulated) bus
    if secondary_bus != "N/A":
        dss.Circuit.SetActiveBus(secondary_bus)
        voltages = dss.Bus.puVmagAngle()[::2]  # Only magnitudes (skip angles)
        if voltages:
            avg_voltage = sum(voltages) / len(voltages)
        else:
            avg_voltage = 0
    else:
        avg_voltage = 0

    # Store values for plotting
    regulator_labels.append(f"{reg}\n({secondary_bus})")
    tap_data.append(tap)
    voltage_data.append(avg_voltage)

# -------------------------------------------------------
# STEP 5: Print summary
# -------------------------------------------------------
print("\nRegulator Details:")
for i, reg in enumerate(regulator_labels):
    print(f"{reg}: Tap={tap_data[i]}, Voltage={voltage_data[i]:.3f} p.u.")

# -------------------------------------------------------
# STEP 6: Plot results
# -------------------------------------------------------
fig, ax1 = plt.subplots(figsize=(10, 6))

# Bar chart for tap positions
ax1.bar(regulator_labels, tap_data, color='teal', alpha=0.6, label='Tap Position')
ax1.set_ylabel("Tap Position", color='teal')
ax1.set_xlabel("Regulator (Regulated Bus)")
ax1.tick_params(axis='y', labelcolor='teal')

# Line plot for regulated voltages
ax2 = ax1.twinx()
ax2.plot(regulator_labels, voltage_data, color='darkred', marker='o', label='Regulated Voltage (p.u.)')
ax2.set_ylabel("Voltage (p.u.)", color='darkred')
ax2.tick_params(axis='y', labelcolor='darkred')

plt.title("Voltage Regulators: Tap Positions and Regulated Bus Voltages")
plt.xticks(rotation=30, ha='right')
plt.grid(axis='x', linestyle='--', alpha=0.4)
fig.tight_layout()
plt.show()


![[Regulator_Tap_Positions.png]]

