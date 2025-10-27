

Nice — I’ll walk you through the **whole script** as if I’m sitting next to you in a lab session. I’ll keep language friendly, avoid jargon where possible, and explain *why* each part exists and *what it does to the power system / your experiment*. I’ll group lines into logical chunks (imports, configuration, helpers, HS, main flow, plotting) and explain the important lines inside each chunk **one-by-one** so it’s easy to follow.

If anything still looks unclear, tell me which exact line number or variable you want me to expand on and I’ll zoom in further.

# 1. Top-of-file: purpose comment

```python
"""
final_optimization_sensitive_attack.py
...
"""
```

* Short docstring describing the script’s goals. Good practice: tells a reader what the file does without opening it.

# 2. Imports and random seed

```python
import opendssdirect as dss
import random, time
import matplotlib.pyplot as plt
import numpy as np
```

* `opendssdirect as dss` — the Python API that controls OpenDSS (runs power flows, returns voltages, etc.). All interaction with the grid model goes through this.
* `random` — used by Harmony Search to create random candidate solutions and small random tweaks.
* `time` — used for timing how long optimization takes.
* `matplotlib.pyplot as plt` — plotting library to draw histograms, line plots, bar charts.
* `numpy as np` — numeric library for arrays and statistics.

```python
# Harmony Search params
HM_SIZE = 8
MAX_ITERS = 70
HMCR = 0.9
PAR = 0.25
BW = 0.3
SEED = 42
random.seed(SEED)
```

* These lines set the algorithm’s hyperparameters:

  * `HM_SIZE`: how many candidate solutions (harmonies) we keep in memory.
  * `MAX_ITERS`: how many iterations the algorithm will run.
  * `HMCR` (harmony memory consideration rate): probability to use a value from memory when constructing a new candidate.
  * `PAR` (pitch adjust rate): probability to tweak a chosen memory value slightly.
  * `BW`: the maximum size of that tweak.
  * `SEED`: seeds Python RNG so runs are reproducible.

# 3. Configuration constants

```python
file_path = r"D:\OpenDSS\IEEETestCases\123Bus\IEEE123Master.dss"
VOL_LOWER, VOL_UPPER = 0.95, 1.05
```

* `file_path`: path to your OpenDSS master file. Must point to the IEEE 123-bus model.
* `VOL_LOWER`/`VOL_UPPER`: acceptable voltage per-unit bounds (±5%). Used to compute penalties and “out-of-range” statistics.

# 4. Utility helpers — loading and reading voltages

```python
def load_and_solve():
    dss.Text.Command(f'Redirect "{file_path}"')
    dss.Text.Command("Solve")
```

* `Redirect` loads the DSS file into OpenDSS; `Solve` runs a power-flow solution (computes voltages/currents).
* You call this first to create the baseline network state.

```python
def all_bus_avg_pu():
    buses = dss.Circuit.AllBusNames()
    names, vals = [], []
    for b in buses:
        try:
            dss.Circuit.SetActiveBus(b)
            mags_ang = dss.Bus.puVmagAngle()
            if not mags_ang:
                continue
            mags = mags_ang[::2]
            if len(mags) > 0:
                names.append(b)
                vals.append(sum(mags) / len(mags))
        except Exception:
            continue
    return names, vals
```

* `dss.Circuit.AllBusNames()` returns all bus names in the circuit (e.g., `'150'`, `'160r'`, etc.).
* For each bus:

  * `SetActiveBus(b)` tells OpenDSS: “I want to query this bus now.”
  * `dss.Bus.puVmagAngle()` returns a list `[V1, θ1, V2, θ2, ...]` — magnitudes and angles for bus phases.
  * `mags = mags_ang[::2]` picks the magnitude values (every other element).
  * We average magnitudes across phases at that bus and append it. So the function returns `([bus1, bus2,...], [avg_pu1, avg_pu2,...])`.
* The averaging gives one number per bus, which is easier to plot as a bus-index vs voltage profile.

```python
def all_bus_vol_map():
    names, vals = all_bus_avg_pu()
    return {n: v for n, v in zip(names, vals)}
```

* Builds a dictionary mapping bus name → average p.u. voltage. Handy for the fitness function.

# 5. Regulator helpers (read/modify regulator settings)

```python
def get_reg_list_and_vregs_taps():
    regs = dss.RegControls.AllNames()
    vregs = []
    taps = []
    for r in regs:
        dss.RegControls.Name(r)
        try:
            vregs.append(dss.RegControls.ForwardVreg())
        except Exception:
            vregs.append(120.0)
        try:
            taps.append(dss.RegControls.TapNumber())
        except Exception:
            taps.append(0)
    return regs, vregs, taps
```

* `dss.RegControls.AllNames()` returns the RegControl elements (e.g., `['creg1a','creg3c', ...]`).
* For each regulator:

  * `ForwardVreg()` returns the target voltage (e.g., 120 V) the RegControl tries to maintain at its monitored bus.
  * `TapNumber()` returns the regulator’s current tap position (discrete integer).
* We store current vregs and taps to compare before/after optimization.

```python
def apply_vregs(regs, vlist):
    for name, v in zip(regs, vlist):
        vval = round(float(v), 3)
        dss.Text.Command(f'Edit RegControl.{name} Vreg={vval}')
```

* To change a RegControl setpoint, we call OpenDSS text command `Edit RegControl.<name> Vreg=<value>`.
* We change `Vreg` (continuous target) rather than taps because:

  * `Vreg` is robust across OpenDSS builds,
  * RegControl itself will update taps when `Solve()` runs, so changing `Vreg` indirectly adjusts taps.

# 6. Load manipulation helper

```python
def set_load_kw(load_name, kw_value):
    dss.Text.Command(f'Edit Load.{load_name} kW={kw_value}')
```

* Simple wrapper to edit a load’s kW in the DSS model. Used to create the attack (increase kW).

# 7. Metrics / fitness

```python
def voltage_quality_metrics(vals):
    vals = np.array(vals)
    mean_abs_dev = np.mean(np.abs(vals - 1.0))
    out_pct = np.sum((vals < VOL_LOWER) | (vals > VOL_UPPER)) / len(vals) * 100
    return mean_abs_dev, out_pct
```

* `mean_abs_dev`: average absolute deviation from 1.0 p.u. — a simple measure of how “far” voltages are from nominal on average.
* `out_pct`: percentage of buses outside the acceptable window (0.95–1.05) — an easy-to-interpret reliability measure.

```python
def fitness_for_profile(vol_map):
    vals = list(vol_map.values())
    # in_range_pen, low_viol, high_viol, penalty calculation...
    return float(penalty)
```

* This function returns a **scalar fitness** used by Harmony Search:

  * small squared deviations inside the allowed band,
  * **very large penalty** (×1000) for violations outside bounds,
  * plus a term proportional to variance to penalize unbalanced voltage spread.
* HS **minimizes** this score — lower is better.

# 8. Harmony Search algorithm (core)

```python
def harmony_search_optimize(...):
    HM = [[random.uniform(lb, ub) for lb, ub in zip(lower_bounds, upper_bounds)] for _ in range(hm_size)]
```

* `HM` (harmony memory) is a list of candidate Vreg vectors (one vector per harmony). Each harmony is a list with length = number of regulators.

```python
def eval_harmony(h):
    apply_vregs(regs, h)
    dss.Text.Command("Solve")
    volt_map = all_bus_vol_map()
    return fitness_for_profile(volt_map)
```

* To evaluate a candidate harmony:

  1. Apply the Vreg vector to the RegControls.
  2. Run `Solve()` to let regulators change taps and compute the new power flow.
  3. Read bus voltages and compute the fitness.

```python
HM_scores = [eval_harmony(h) for h in HM]
best_idx = int(np.argmin(HM_scores))
best_h = HM[best_idx][:]
best_score = HM_scores[best_idx]
```

* Evaluate the initial memory and find the best starting harmony and score.

Main loop:

```python
for it in range(iterations):
    new = []
    for j in range(len(regs)):
        if random.random() < hmcr:
            val = random.choice(HM)[j]
            if random.random() < par:
                delta = bw * (random.random()*2 - 1)
                val += delta
        else:
            val = random.uniform(lower_bounds[j], upper_bounds[j])
        val = max(lower_bounds[j], min(upper_bounds[j], val))
        new.append(val)
    new_score = eval_harmony(new)
    worst_idx = int(np.argmax(HM_scores))
    if new_score < HM_scores[worst_idx]:
        HM[worst_idx] = new
        HM_scores[worst_idx] = new_score
    if new_score < best_score:
        best_score = new_score
        best_h = new[:]
        best_history.append(best_score)
```

* For each new candidate:

  * For each regulator dimension:

    * With probability `HMCR` pick an existing value from memory (re-use); with `PAR` tweak it slightly (pitch adjust).
    * Else pick a random new value (explore).
  * Clamp to the allowed bounds.
  * Evaluate new candidate.
  * Replace the worst memory entry if new is better.
  * Update global best if improved.
* `best_history` collects the best score improvements for plotting/logging.

**Why HS here?**

* It balances exploitation (re-using good solutions) and exploration (random new values + pitch adjust) and works well on continuous search spaces (Vreg). It’s simple and effective for moderate dimensionality (number of regulators).

# 9. Sensitivity scan (select most impactful load)

```python
loads = dss.Loads.AllNames()
for load in loads:
    dss.Loads.Name(load)
    orig_kw = dss.Loads.kW()
    original_kw_map[load] = orig_kw
    set_load_kw(load, orig_kw * 3.0)
    dss.Text.Command("Solve")
    _, vals = all_bus_avg_pu()
    mean_abs_dev = float(np.mean(np.abs(np.array(vals) - 1.0)))
    sensitivity[load] = mean_abs_dev
    set_load_kw(load, orig_kw)
```

* This loop:

  1. Iterates every load in the system.
  2. Reads its original kW (`dss.Loads.kW()`).
  3. Temporarily multiplies the load by 3.0 (strong disturbance).
  4. Runs a solve and measures the global mean absolute voltage deviation.
  5. Stores that deviation as the load’s sensitivity metric.
  6. Restores the load to its original value.
* After the loop the script picks the load with the **largest mean deviation**. That is the “most sensitive” load — the one whose manipulation creates the worst voltage disturbance system-wide.
* This is research-wise important: red-team style — test worst-case attack.

# 10. Attack, optimize, restore

```python
target_load = max(sensitivity.keys(), key=lambda k: sensitivity[k])
dss.Loads.Name(target_load)
orig_kw = dss.Loads.kW()
attack_kw = orig_kw * 3.0
set_load_kw(target_load, attack_kw)
dss.Text.Command("Solve")
_, post_attack_vals = all_bus_avg_pu()
```

* Choose the worst load and apply the severe attack (3× kW). Re-solve and capture post-attack voltages.

```python
best_harmony, best_score, best_history = harmony_search_optimize(...)
apply_vregs(regs, best_harmony)
dss.Text.Command("Solve")
_, final_vals = all_bus_avg_pu()
```

* Run the HS optimizer to find Vreg targets that minimize the fitness under the attacked condition.
* Apply the found Vreg vector and re-solve to see final voltages.

# 11. Metrics reporting

```python
base_dev, base_out = voltage_quality_metrics(baseline_vals)
atk_dev, atk_out = voltage_quality_metrics(post_attack_vals)
hs_dev, hs_out = voltage_quality_metrics(final_vals)
```

* Compute:

  * mean absolute deviation (`mean|V-1|`) for baseline, post-attack, after-HS,
  * % of buses out-of-range for each stage.
* Print these to summarize system health numerically (easy to include in results tables).

# 12. Plotting & visualization

Three-panel figure:

1. **Histogram** of voltage distributions (baseline, attack, after HS). Useful to see how many buses moved away from 1.0 and how HS tightened the distribution.
2. **Bus-index voltage profile** lines (baseline / attack / after HS) — you can visually locate where drops occurred along the feeder. The script marks the attacked bus with a red dashed line and label.
3. **Bar chart** of regulator taps before vs after — shows the physical control actions (how many taps changed and by how much).

```python
axes[1].axvline(att_idx, color='red', ls='--', lw=2, label=f'Attacked bus: {att_bus_name}')
```

* This marks the attacked bus’s index so viewers can quickly see the disturbance origin.

# 13. Clean-up and restore

```python
set_load_kw(target_load, orig_kw)
dss.Text.Command("Solve")
```

* Restores the attacked load to original value after the experiment — good practice so the model isn’t left modified.

# 14. Return value

```python
return {
  "target_load": target_load,
  ...
}
```

* Optionally returns a dictionary with summary data for further programmatic analysis.

---

## Quick conceptual recap (in plain words)

* You load the grid, measure baseline voltages.
* You find the single load that, if pushed hard, causes the worst voltage disturbance.
* You push that load (attack), then run an optimizer that changes regulator *targets* (Vreg) so regulation devices change taps and improve voltages.
* You measure and plot how the network’s voltages changed and how regulators physically reacted (tap positions).
* You quantify the improvement with two simple metrics: mean absolute deviation and percent of buses outside acceptable voltage window.

---

## Why this is a good research setup (for an undergrad)

* **Reproducible**: all steps are scripted and deterministic (seeded random).
* **Measurable**: you compute metrics that can be compared across scenarios or algorithms.
* **Explainable**: plots and tap charts show physical cause–effect (regulator taps cause voltage changes).
* **Extendable**: you can swap Harmony Search with GA or WOA, or add a detection layer (ML anomaly detector) on top.
![[Figure_1 1.png]]






