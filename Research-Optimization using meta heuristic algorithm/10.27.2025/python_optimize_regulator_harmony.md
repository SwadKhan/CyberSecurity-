

---

## 🧩 **Section 1: Imports and configuration**

```python
import opendssdirect as dss
import random
import math
import time
import matplotlib.pyplot as plt
```

* **`opendssdirect`**: This is the Python API that directly controls OpenDSS (no COM interface). It’s how we send DSS commands and read results.
* **`random`, `math`, `time`**: Used in the Harmony Search (HS) algorithm for randomness, basic math, and timing the optimization.
* **`matplotlib.pyplot`**: Used for visualization (plots showing voltage distributions).

---

```python
file_path = r'D:\OpenDSS\IEEETestCases\123Bus\IEEE123Master.dss'
VOL_LOWER = 0.95
VOL_UPPER = 1.05
MAX_ITERS = 80
HM_SIZE = 10
HMCR = 0.9
PAR = 0.3
BW = 0.5
SEED = 42
random.seed(SEED)
```

* **`file_path`**: Path to your OpenDSS master file. This is what the program loads into DSS.
* **Voltage bounds (0.95–1.05)**: Define acceptable per-unit voltage range (±5% from nominal).
* **`MAX_ITERS`**: Maximum number of optimization iterations (80 HS loops).
* **`HM_SIZE`**: Number of solutions stored in Harmony Memory (population size).
* **`HMCR`**: *Harmony Memory Consideration Rate* – how often new solutions borrow from past ones (90%).
* **`PAR`**: *Pitch Adjustment Rate* – how often the algorithm tweaks existing solutions slightly (30%).
* **`BW`**: Bandwidth – how big those tweaks can be.
* **`SEED`**: Random seed for reproducibility.

---

## ⚙️ **Section 2: Helper functions for OpenDSS**

### **Load and solve the circuit**

```python
def load_and_solve():
    dss.Text.Command(f'Redirect "{file_path}"')
    dss.Text.Command("Solve")
```

* Sends a DSS command to load your model (`Redirect`) and run the power flow (`Solve`).
* Equivalent to typing those commands inside the OpenDSS console.

---

### **Get all regulators**

```python
def get_regulator_list():
    regs = dss.RegControls.AllNames()
    return regs
```

* Returns a list of all regulator control names from the DSS model.
  (These are devices that control transformer taps.)

---

### **Get info about one regulator**

```python
def get_reg_info(reg):
    dss.RegControls.Name(reg)
    tap = dss.RegControls.TapNumber()
    try:
        vreg = dss.RegControls.ForwardVreg()
    except Exception:
        vreg = None
    band = dss.RegControls.ForwardBand() if 'ForwardBand' in dir(dss.RegControls) else None
    trans = dss.RegControls.Transformer()
    return {"name": reg, "tap": tap, "vreg": vreg, "band": band, "transformer": trans}
```

* Selects a specific regulator (`Name(reg)`).
* Reads:

  * `TapNumber`: current tap position.
  * `ForwardVreg`: voltage regulation target in volts.
  * `ForwardBand`: bandwidth (voltage range where it won’t move taps).
  * `Transformer`: the transformer that regulator controls.
* Returns all as a dictionary — good for inspection or logging.

---

### **Read bus voltage (in p.u.)**

```python
def get_bus_voltage_pu(bus_name):
    ...
    dss.Circuit.SetActiveBus(bus_name)
    mags_and_angles = dss.Bus.puVmagAngle()
    mags = mags_and_angles[::2]
    return sum(mags) / len(mags)
```

* Activates a bus and reads its voltage magnitudes in per-unit (`puVmagAngle()` returns [V1, θ1, V2, θ2, ...]).
* Takes the average across all phases (for a single representative value).

---

### **Get all bus voltages**

```python
def all_bus_voltages_pu():
    buses = dss.Circuit.AllBusNames()
    volt_map = {}
    for b in buses:
        dss.Circuit.SetActiveBus(b)
        mags = dss.Bus.puVmagAngle()[::2]
        if mags:
            volt_map[b] = sum(mags) / len(mags)
    return volt_map
```

* Scans **every bus** in the circuit.
* Records the **average per-unit voltage**.
* Returns a dictionary: `{bus_name: avg_voltage}` → This becomes your “voltage profile.”

---

## 📉 **Section 3: Fitness function**

```python
def fitness_for_profile(vol_map):
    penalty = 0.0
    vals = list(vol_map.values())
    for v in vals:
        if v < VOL_LOWER:
            penalty += (VOL_LOWER - v) ** 2 * 1000
        elif v > VOL_UPPER:
            penalty += (v - VOL_UPPER) ** 2 * 1000
        else:
            penalty += (v - 1.0) ** 2
    mean_v = sum(vals) / len(vals)
    var = sum((x - mean_v) ** 2 for x in vals) / len(vals)
    penalty += var * 10
    return penalty
```

This defines **how “bad” the system’s voltages are**:

* Strongly penalizes voltages **outside** 0.95–1.05 p.u.
* Mildly penalizes voltages **within range** that aren’t near 1.0.
* Adds a variance penalty → encourages flat, even voltages across buses.

The **lower** this fitness score, the **better** your voltage profile.

---

## 🧮 **Section 4: Applying Vreg settings**

```python
def apply_vregs(regs, vreg_vector):
    for reg, v in zip(regs, vreg_vector):
        v_val = round(v, 3)
        dss.Text.Command(f'Edit RegControl.{reg} Vreg={v_val}')
```

* Applies a new list of **Vreg setpoints** to all regulators.
* Uses DSS `Edit` command to set new target voltages (in volts).
* You’ll notice a fallback with lowercase names — OpenDSS isn’t case-sensitive, but this keeps it robust.

---

## ⚡ **Section 5: Disturbance simulation**

```python
def simulate_attack_increase_load(target_load_name, factor=1.5):
    ...
```

and

```python
def set_load_kw(load_name, new_kw):
    dss.Text.Command(f"Edit Load.{load_name} kW={new_kw}")
```

* These create a **voltage disturbance** by increasing one load’s power.
* For example, if you suddenly draw 50% more kW at a feeder end, the voltage there will dip.
* Used to test how well your optimization corrects that disturbance.

---

## 🧠 **Section 6: Harmony Search Algorithm**

```python
def harmony_search_optimize(...):
```

This is the heart of the script.

### Step 1 – Initialize memory

```python
HM = []
for _ in range(hm_size):
    harmony = [random.uniform(lb, ub) for lb, ub in zip(lower_bounds, upper_bounds)]
    HM.append(harmony)
```

Creates `HM_SIZE` random vectors of Vreg values within allowed voltage bounds.
Each vector is a “harmony” (a candidate solution).

---

### Step 2 – Evaluate fitness

```python
def eval_harmony(h):
    apply_vregs(regs, h)
    dss.Text.Command("Solve")
    volt_map = all_bus_voltages_pu()
    return fitness_for_profile(volt_map)
```

Applies that harmony, runs a power flow, and measures the penalty.
Lower score = better voltage regulation.

---

### Step 3 – Main optimization loop

```python
for it in range(iterations):
    new_harmony = []
    for j in range(len(regs)):
        if random.random() < hmcr:
            val = random.choice(HM)[j]
            if random.random() < par:
                delta = bw * (random.random() * 2 - 1)
                val = val + delta
        else:
            val = random.uniform(lower_bounds[j], upper_bounds[j])
        val = max(lower_bounds[j], min(upper_bounds[j], val))
        new_harmony.append(val)
```

* Builds a new candidate solution by:

  * **Borrowing** from existing harmonies (probability HMCR = 0.9)
  * **Tweaking** them slightly (PAR = 0.3)
  * Or randomly generating new values.
* Keeps it within [lower_bound, upper_bound] range.

---

### Step 4 – Replacement and tracking

```python
score = eval_harmony(new_harmony)
worst_idx = max(range(len(HM_scores)), key=lambda i: HM_scores[i])
if score < HM_scores[worst_idx]:
    HM[worst_idx] = new_harmony
    HM_scores[worst_idx] = score
```

* If the new candidate is better than the **worst** one in memory, replace it.
* Keeps population always full of the *best recent ideas*.

---

### Step 5 – Update best

```python
if score < best_score:
    best_score = score
    best_harmony = new_harmony
    print(f"[HS] iter {it+1}: new best score {best_score:.3f}")
```

* Tracks and prints the **best Vreg vector found so far**.

---

## 🔁 **Section 7: Main driver**

This section ties everything together.

1. **Load and solve the baseline**

   ```python
   load_and_solve()
   regs = get_regulator_list()
   ```

   → Starts DSS and gets regulator names.

2. **Read current Vreg settings**

   ```python
   current_vregs = []
   for reg in regs:
       v = dss.RegControls.ForwardVreg() or 120.0
       current_vregs.append(v)
   ```

   → Extracts each regulator’s current target (defaults to 120 V if unreadable).

3. **Get baseline voltages**

   ```python
   baseline_voltages = all_bus_voltages_pu()
   ```

   → Snapshot before disturbance.

4. **Create a disturbance**

   ```python
   loads = dss.Loads.AllNames()
   target_load = loads[0]
   set_load_kw(target_load, 200.0)
   dss.Text.Command("Solve")
   post_attack_volt = all_bus_voltages_pu()
   ```

   → Increases one load (first in list) to cause undervoltage.

5. **Define search bounds**

   ```python
   lower_bounds = [max(110.0, v - 6.0) for v in current_vregs]
   upper_bounds = [min(130.0, v + 6.0) for v in current_vregs]
   ```

   → Limits search for each regulator ±6 V around its starting value.

6. **Run optimization**

   ```python
   best_harmony, best_score = harmony_search_optimize(...)
   apply_vregs(regs, best_harmony)
   dss.Text.Command("Solve")
   final_voltages = all_bus_voltages_pu()
   ```

   → Finds and applies the optimal Vreg setpoints.

7. **Visualize**

   ```python
   plt.hist(baseline, bins=30, ...)
   plt.hist(attacked, bins=30, ...)
   plt.hist(final, bins=30, ...)
   ```

   → Displays how voltage distributions shift:

   * Green (baseline)
   * Red (disturbed)
   * Blue (after optimization)

---

## 🎸 **Analogy: Orchestra and Conductor**

Think of the **Harmony Search** algorithm like a **music band**:

| Concept                  | Analogy                                        |
| ------------------------ | ---------------------------------------------- |
| Each *regulator setting* | A musician’s note                              |
| A *harmony*              | One full performance of all notes              |
| *Harmony Memory*         | The band’s memory of good melodies             |
| *HMCR*                   | Reusing notes from past songs                  |
| *PAR*                    | Slightly tuning an existing note               |
| *BW*                     | How much that tuning can vary                  |
| *Fitness function*       | Audience feedback on how pleasant the sound is |
| *Optimization loop*      | Rehearsals until the song sounds best          |

So your algorithm “listens” (evaluates voltages), tweaks regulators like a conductor, and guides the grid back to harmony (stable voltage).

---
![[voltage distribution.png]]


