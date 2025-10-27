
---

## 🧭 OVERVIEW

The code simulates a **three-step experiment**:

1. **Baseline:** Normal feeder operation (no disturbance).
2. **Post-attack:** A load is suddenly increased (a “voltage disturbance”).
3. **After Harmony Search (HS):** The regulator voltages are automatically retuned to restore system balance.

It then plots **three side-by-side graphs**—one per scenario—showing **per-phase voltage magnitudes** for every bus.

---

## ⚙️ SETUP & CONFIGURATION

```python
file_path = r'D:\OpenDSS\IEEETestCases\123Bus\IEEE123Master.dss'
TARGET_LOAD_NAME = 'auto'
```

Loads the 123-bus OpenDSS feeder file.
If `auto`, the code picks the first load automatically.

Harmony Search parameters:

```python
HM_SIZE = 8   # number of harmonies stored
MAX_ITERS = 60
HMCR = 0.9    # harmony memory consideration rate
PAR = 0.25    # pitch adjusting rate
BW = 0.3      # adjustment bandwidth
```

These control how widely the optimizer explores new regulator setpoints.

---

## 🧩 HELPER FUNCTIONS

**`load_and_solve()`** — opens the feeder and runs a power flow.
**`all_bus_phase_mags()`** — collects each bus’s per-phase voltage magnitudes (p.u.).
**`get_reg_list_and_vregs()`** — lists all regulators and their current `Vreg` setpoints.
**`apply_vregs()`** — updates each regulator’s `Vreg`.
**`set_load_kw()`** — increases a load to simulate an attack.
**`all_bus_vol_map()`** — computes average voltage per bus, used for fitness evaluation.

---

## 🎼 HARMONY SEARCH (HS)

The optimizer treats each **regulator setting vector** as a “harmony.”
At each iteration:

1. Generate a new candidate set of `Vreg`s (either from memory or random).
2. Apply it in OpenDSS and re-solve.
3. Compute **fitness** = how far bus voltages deviate from 1.0 p.u.
4. Keep better harmonies and discard worse ones.

The **fitness function** penalizes:

* Voltages outside [0.95, 1.05] p.u.
* Large variance across buses.

Goal → minimize voltage deviation and imbalance.

---

## 🚀 MAIN SIMULATION STEPS

1. **Baseline:**

   ```python
   load_and_solve()
   bus_names, baseline_phase_mags = all_bus_phase_mags()
   ```

   Stores normal voltages.

2. **Post-attack:**

   ```python
   set_load_kw(target_load_name, 200.0)
   dss.Text.Command("Solve")
   ```

   Raises one load to 200 kW to disturb the system.

3. **Optimization:**

   ```python
   best_h, best_score = harmony_search_optimize(...)
   apply_vregs(regs, best_h)
   dss.Text.Command("Solve")
   ```

   Runs HS and applies the best regulator settings.

---

## 📊 PLOTTING

Three subplots (baseline, disturbed, optimized).
Each shows three lines—Phase 1 (blue), Phase 2 (orange), Phase 3 (green)—of voltage magnitude vs. bus index.

Left → healthy voltages
Middle → disturbed profile
Right → restored profile

This visually demonstrates how HS brings the voltages back near 1.0 p.u.

---

## 🧮 OUTPUT SAMPLE

At the end, the script prints the first few buses’ voltage triplets:

```
Bus1: [1.00, 0.99, 1.01] | [0.92, 0.93, 0.95] | [0.99, 0.99, 1.00]
```

→ baseline | post-attack | after HS

---

## 🎻 ANALOGY

Think of your feeder as an **orchestra**:

* Each **bus** is a musician (voltage level).
* Each **regulator** is a tuning knob.
* The **attack** throws one instrument out of tune.
* The **Harmony Search** listens and adjusts every knob until the orchestra sounds balanced again.

---

If you’d like, I can rewrite this same script with **inline comments** (each step explained beside the code) so that when you open it in VS Code you can read it like a tutorial.
Would you like me to format it that way next?


Sure — here’s what that full script is doing, broken down so you can follow it line by line while you work in VS Code:

---

### 🧭 Goal

This program loads an **OpenDSS feeder**, creates a **voltage disturbance** by increasing one load, and then uses the **Harmony Search algorithm** to retune every voltage regulator so the feeder voltages come back toward 1.0 p.u.
It plots three side-by-side graphs so you can see what happens *before*, *during*, and *after* the optimization.

---

### ⚙️ 1.  Imports and constants

```python
import opendssdirect as dss
import random, time
import matplotlib.pyplot as plt
```

* `opendssdirect` – lets Python control OpenDSS directly.
* `random` – random numbers for the Harmony Search.
* `matplotlib` – draws the voltage graphs.

Then you set:

```python
file_path = r'D:\OpenDSS\IEEETestCases\123Bus\IEEE123Master.dss'
TARGET_LOAD_NAME = 'auto'
```

and the HS tuning constants (`HM_SIZE`, `MAX_ITERS`, etc.).
Those control how many candidate “harmonies” are stored and how widely they’re adjusted.

---

### 🧩 2.  Helper functions

| Function                   | What it does                                                                    |
| -------------------------- | ------------------------------------------------------------------------------- |
| `load_and_solve()`         | Loads the feeder file and runs a power flow once.                               |
| `all_bus_phase_mags()`     | Returns every bus’s phase voltages in p.u. (pads to 3 phases so plots line up). |
| `get_reg_list_and_vregs()` | Lists all `RegControl` objects and reads their `Vreg` settings.                 |
| `apply_vregs()`            | Writes new `Vreg` values back into OpenDSS.                                     |
| `set_load_kw()`            | Changes the kW of a chosen load → this is the “attack.”                         |
| `all_bus_vol_map()`        | Builds a `{bus: average voltage}` dictionary for use in the fitness function.   |

---

### 🎼 3.  Harmony Search optimizer

`harmony_search_optimize()`:

1. Creates a “harmony memory” of random regulator settings within bounds.
2. Evaluates each by:

   * applying it,
   * solving the circuit,
   * scoring it with `fitness_for_profile()`.
3. Repeats for `MAX_ITERS` iterations, always keeping the best-scoring harmonies.
4. Returns the best set of regulator voltages.

`fitness_for_profile()` gives a low score to profiles that:

* stay inside 0.95 – 1.05 p.u.,
* are close to 1.0 p.u.,
* and have small variance across buses.

---

### 🚀 4.  Main simulation steps

```python
load_and_solve()                      # 1️⃣  baseline
bus_names, baseline_phase_mags = all_bus_phase_mags()

set_load_kw(target_load_name, 200.0)  # 2️⃣  simulate attack
dss.Text.Command("Solve")
bus_names2, post_attack_phase_mags = all_bus_phase_mags()

best_h, best_score = harmony_search_optimize(...)  # 3️⃣  run HS
apply_vregs(regs, best_h)
dss.Text.Command("Solve")
bus_names3, final_phase_mags = all_bus_phase_mags()
```

So you end up with three datasets of bus-by-phase voltages.

---

### 📊 5.  Plotting

Creates three subplots, one for each scenario, sharing the same Y-axis.
Each plot shows Phase 1 (blue), Phase 2 (orange), Phase 3 (green) voltage magnitude vs. bus index.

You’ll see:

* **Left:** baseline—nearly flat around 1.0 p.u.
* **Middle:** disturbance—voltage dips on several buses.
* **Right:** after HS—voltages mostly restored.

---

### 🧮 6.  Printed results

At the end:

```python
for i in range(min(8, len(bus_names))):
    print(f"{bus_names[i]}: {baseline_phase_mags[i]} | "
          f"{post_attack_phase_mags[i]} | {final_phase_mags[i]}")
```

so you can compare numeric voltage magnitudes for the first few buses.

---

### 🎻 Analogy

Imagine a 3-phase feeder as a **symphony orchestra**:

* Each **bus** = a musician’s note (voltage).
* Each **regulator** = a tuning knob.
* The **attack** = one player suddenly out of tune (a heavy load).
* The **Harmony Search** = a sound engineer turning knobs until the music sounds balanced again.

---


![[Figure_4.png]]

