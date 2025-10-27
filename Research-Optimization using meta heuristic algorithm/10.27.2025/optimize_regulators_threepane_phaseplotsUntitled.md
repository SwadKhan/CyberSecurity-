

---

## 🧠 1. **What this script does overall**

This script connects **Python (via OpenDSSDirect)** to simulate a power distribution feeder model, such as the IEEE 123-bus test system.

Then it runs three scenarios:

1. **Baseline** — Normal system with no problems.
2. **Post-attack** — A “disturbance” (a load demand spike) is introduced.
3. **After Harmony Search (HS)** — Regulator voltage setpoints (`Vreg`) are optimized using the **Harmony Search Algorithm** to restore system voltage balance.

Finally, it plots **three side-by-side graphs** (baseline, post-attack, optimized) for **each phase’s voltage magnitude per bus** — so you can visually compare the effect of the optimization.

---

## 🧩 2. **Imports and Setup**

```python
import opendssdirect as dss
import random, time
import matplotlib.pyplot as plt
```

* `opendssdirect`: Python interface to OpenDSS.
* `random`: For generating random numbers used in Harmony Search.
* `matplotlib`: To plot the voltage profiles for visualization.

---

## ⚙️ 3. **Configuration**

```python
file_path = r'D:\OpenDSS\IEEETestCases\123Bus\IEEE123Master.dss'
```

This is the path to your OpenDSS model file.
→ You can change this to whichever `.dss` model you want to analyze.

```python
TARGET_LOAD_NAME = 'auto'
```

This is the load that will be “attacked” or disturbed (increased demand).

```python
HM_SIZE = 8  # Harmony memory size
MAX_ITERS = 60  # Max iterations for optimization
HMCR = 0.9  # Harmony memory consideration rate
PAR = 0.25  # Pitch adjusting rate
BW = 0.3  # Bandwidth (how big the adjustments are)
```

These are **Harmony Search parameters**, controlling the exploration behavior.

---

## 🔧 4. **Helper Functions**

### `load_and_solve()`

```python
def load_and_solve():
    dss.Text.Command(f'Redirect "{file_path}"')
    dss.Text.Command("Solve")
```

This loads the circuit model and solves it (runs a power flow).

---

### `all_bus_phase_mags()`

Gets voltage magnitudes (in per-unit) for each bus and phase.
If a bus only has 1 or 2 phases, it pads the list to 3 for alignment.

---

### `get_reg_list_and_vregs()`

Retrieves all `RegControl` devices and their target voltage settings (`Vreg`).
If missing, defaults to 120 V.

---

### `apply_vregs(regs, vlist)`

Updates the regulator control voltage setpoints in OpenDSS.

---

### `set_load_kw(load_name, kw_value)`

Artificially increases the load (to simulate an attack/disturbance).

---

### `all_bus_vol_map()`

Creates a mapping `{bus_name: average_voltage}` used for optimization fitness calculation.

---

## 🎯 5. **Harmony Search Optimization**

### `harmony_search_optimize(...)`

This function mimics nature-inspired music improvisation:

* Each “harmony” = a possible set of regulator voltages (`Vreg`s).
* The “fitness” = how good the voltage profile is (less deviation, better).
* Over many iterations, harmonies are “played,” adjusted, and the best one evolves.

Each iteration:

1. Generate a new harmony (candidate regulator settings).
2. Evaluate it via power flow and fitness score.
3. Replace the worst harmony if it’s better.
4. Keep track of the best overall.

---

### `fitness_for_profile(vol_map)`

Computes how “good” a voltage profile is:

* Penalizes voltages outside the range [0.95, 1.05] p.u.
* Adds variance penalty (to smooth voltage differences).
* Rewards voltages close to 1.0 p.u.

---

## 🚀 6. **Main Driver**

### Step 1: Load model

```python
load_and_solve()
bus_names, baseline_phase_mags = all_bus_phase_mags()
```

Run base model and store all bus voltage magnitudes.

---

### Step 2: Simulate the disturbance (attack)

```python
set_load_kw(target_load_name, 200.0)
dss.Text.Command("Solve")
```

→ Forces one load to a high power demand (e.g., 200 kW).
→ This disturbs voltage levels system-wide.

---

### Step 3: Run Harmony Search

```python
best_h, best_score = harmony_search_optimize(regs, cur_vregs, lower, upper)
apply_vregs(regs, best_h)
dss.Text.Command("Solve")
```

→ Adjusts regulator voltages (Vreg) automatically to improve voltages after disturbance.

---

## 📊 7. **Plotting Section**

```python
fig, axs = plt.subplots(1, 3, figsize=(18,6), sharey=True)
```

→ Three subplots (baseline, post-attack, after HS).

Each subplot:

* X-axis: Bus index (1...N)
* Y-axis: Voltage magnitude (p.u.)
* Three color-coded lines: each phase (blue, orange, green)

You’ll see:

* Left: smooth (healthy) voltages.
* Middle: voltage dips and imbalance (disturbed).
* Right: restored profile after HS (closer to baseline).

---

## 📄 8. **Console Output**

Prints voltage magnitudes of the first few buses before, during, and after the disturbance:

```python
print("Sample bus voltage magnitudes (first 8 buses)")
```

So you can compare numerical values alongside the plots.

---

## 🔍 9. **Analogy**

Think of this like a **musical orchestra**:

* Each *bus* is a musical instrument (playing voltage).
* The *feeder* is the conductor’s baton (power flow path).
* *Regulators* are like tuning knobs that adjust tone (voltage).
* The *disturbance* is when one instrument goes out of tune.
* The *Harmony Search algorithm* listens to the whole orchestra and adjusts each knob slightly until the performance sounds balanced again.

---
![[Figure_3.png]]
