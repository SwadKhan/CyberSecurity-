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