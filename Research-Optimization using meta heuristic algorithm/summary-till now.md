


# Analogy (water plumbing)

Think of the feeder like a water distribution pipe that starts at the water plant (substation) and runs through streets (lines) to neighborhood taps (buses). Customers (loads) are faucets along the pipe that take water (current).

- The **bus** is like a node or junction in the pipe network (a T-junction).
    
- A **load** is like a house faucet drawing water — the more faucets open, the more pressure drop down the line.
    
- A **regulator (transformer with taps)** is like a pressure booster/valve installed on the pipe between two street segments. The valve can be turned a little to increase or decrease downstream pressure.
    
- **vreg** is the _desired pressure_ at a particular house downstream. A pressure sensor (RegControl) sits at that house and tells the valve to open or close so the pressure stays at the target.
    
- **Tap position** is how much you open/close that valve (tiny increments). Each click of the valve knob changes downstairs pressure by a small percent (tap step).
    
- **Range covered by the regulator** = how far the valve can open or close (e.g., ±10% pressure change). If demand increases a lot (many faucets open) and the valve reaches full open, houses downstream will still see low pressure — meaning the regulator’s range was insufficient.
    
So regulators + RegControl = sensor (vreg) + small valve adjustments (tap positions) to keep downstream “pressure” (voltage) where customers need it.



Why taps differ without any modifications


creg3c, creg4a, etc. — these are RegControl names (regulator control elements).

(25r.3), (160r.1) — these are bus names (the transformer/regulator’s associated bus).

25r or 160r is the bus name (often numbering convention in IEEE 123).

.1, .2, .3 are phase suffixes (phase 1 = A, phase 2 = B, phase 3 = C). So 160r.1 means bus 160r, phase A.

Tap=10 — the regulator’s tap position (discrete mechanical step). Taps change the transformer turn ratio slightly. Higher tap = boosting voltage at that location.

Voltage=1.042 p.u. — the actual measured voltage at the regulated bus (per unit of nominal).





Regulators automatically choose tap positions to reach their target voltage (Vreg) given the network topology and loads.

Different regulators sit at different locations and see different load downstream; therefore they often use different tap settings even in the baseline case.

Some regulators are single-phase (creg1a) and will behave differently per phase; some banks may intentionally have higher Vreg (e.g., 124 V) to compensate for long feeders. So different taps are expected.





