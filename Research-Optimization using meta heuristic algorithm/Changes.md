
Driver code-

import opendssdirect as dss

import matplotlib.pyplot as plt

  

# Specify the file path for the IEEE 123-bus test feeder

file_path = r'D:\OpenDSS\IEEETestCases\123Bus\IEEE123Master.dss'

  

# Redirect to the DSS file (load the circuit)

dss.Text.Command(f'Redirect "{file_path}"')
  

# Solve the load flow

dss.Text.Command("Solve")

  

# Initialize a dictionary to store voltages by phase

voltages_by_phase = {1: [], 2: [], 3: []}

  

# Retrieve voltage magnitudes for each phase

for phase in [1, 2, 3]:

    voltages = dss.Circuit.AllNodeVmagPUByPhase(phase)

    voltages_by_phase[phase] = voltages

  

# Plotting

plt.figure(figsize=(10, 6))

for phase, voltages in voltages_by_phase.items():

    plt.plot(voltages, label=f"Phase {phase}", marker="o")

plt.title("Voltage Magnitudes by Phase")

plt.xlabel("Node Index")

plt.ylabel("Voltage Magnitude (V)")

plt.legend()

plt.grid(True)

plt.show()

the figure we get-
![[Figure_1.png]]



after disabling the all regulators this( using this command ->dss.Text.Command('disable Regulators')


![[Figure_1_no_regulators.png]]

using (  print(dir(dss.RegControls))   ) this command I found all the controls-
which are 
['AllNames', 'CTPrimary', 'Count', 'Delay', 'First', 'ForwardBand', 'ForwardR', 'ForwardVreg', 'ForwardX', 'Idx', 'IsInverseTime', 'IsReversible', 'MaxTapChange', 'MonitoredBus', 'Name', 'Next', 'PTRatio', 'Reset', 'ReverseBand', 'ReverseR', 'ReverseVreg', 'ReverseX', 'TapDelay', 'TapNumber', 'TapWinding', 'Transformer', 'VoltageLimit', 'Winding', '_Get_AllNames', '_Get_Count', '_Get_First', '_Get_Name', '_Get_Next', '_Get_idx', '_Set_Name', '_Set_idx', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getstate__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__module__', '__name__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__slots__', '__str__', '__subclasshook__', '__weakref__', '_api_prefix', '_api_util', '_check_for_error', '_columns', '_decode_and_free_string', '_dss_attributes', '_dss_original_attributes', '_enable_exceptions', '_errorPtr', '_frozen_attrs', '_get_complex128_array', '_get_complex128_gr_array', '_get_complex128_gr_simple', '_get_complex128_simple', '_get_fcomplex128_array', '_get_fcomplex128_gr_array', '_get_fcomplex128_gr_simple', '_get_fcomplex128_simple', '_get_float64_array', '_get_float64_gr_array', '_get_int32_array', '_get_int32_gr_array', '_get_int8_array', '_get_int8_gr_array', '_get_string', '_get_string_array', '_getattr', '_getattr_case_check', '_lib', '_prepare_complex128_array', '_prepare_complex128_simple', '_prepare_float64_array', '_prepare_int32_array', '_prepare_string_array', '_set_string_array', '_setattr', '_setattr_case_check', '_use_exceptions']

There is no VReg() or vreg()
Instead, OpenDSSDirect uses ForwardVreg() and ForwardBand() for the forward direction of power flow.
If the regulator is reversible (it can operate in both directions), there are also ReverseVreg() and ReverseBand().

And using this we found the regulators-   ##regulators = dss.RegControls.AllNames()
Regulators found: ['creg1a', 'creg2a', 'creg3a', 'creg3c', 'creg4a', 'creg4b', 'creg4c']


##print("\nRegulator Details:")
for info in regulator_info:
    print(
        f"{info['Name']:>6} | MonBus: {info['MonitoredBus']:<15} "
        f"| Target V: {info['TargetVoltage(Vreg)']} "
        f"| Band: {info['Bandwidth']} "
        f"| Tap: {info['TapPosition']}"
    )
For this command : 

Regulator Details:
creg1a | MonBus:                 | Target V: 120.0 | Band: 2.0 | Tap: 6
creg2a | MonBus:                 | Target V: 120.0 | Band: 2.0 | Tap: 0
creg3a | MonBus:                 | Target V: 120.0 | Band: 1.0 | Tap: 2
creg3c | MonBus:                 | Target V: 120.0 | Band: 1.0 | Tap: 0
creg4a | MonBus:                 | Target V: 124.0 | Band: 2.0 | Tap: 10
creg4b | MonBus:                 | Target V: 124.0 | Band: 2.0 | Tap: 4
creg4c | MonBus:                 | Target V: 124.0 | Band: 2.0 | Tap: 6


the whole code :

import opendssdirect as dss
import matplotlib.pyplot as plt

# -------------------------------------------------------
# STEP 1: Load the IEEE 123-bus test feeder
# -------------------------------------------------------
file_path = r'D:\OpenDSS\IEEETestCases\123Bus\IEEE123Master.dss'
dss.Text.Command(f'Redirect "{file_path}"')
print(dir(dss.RegControls))
# -------------------------------------------------------
# STEP 2: Solve the circuit
# -------------------------------------------------------
dss.Text.Command("Solve")

# -------------------------------------------------------
regulators = dss.RegControls.AllNames()
print("Regulators found:", regulators)

# -------------------------------------------------------
# STEP 4: Gather data about each regulator
# -------------------------------------------------------
tap_data = []
regulator_info = []

for reg in regulators:
    dss.RegControls.Name(reg)  # Activate regulator
    tap = dss.RegControls.TapNumber()
    vreg = dss.RegControls.ForwardVreg()
    band = dss.RegControls.ForwardBand()
    mon_bus = dss.RegControls.MonitoredBus()
    
    regulator_info.append({
        "Name": reg,
        "MonitoredBus": mon_bus,
        "TargetVoltage(Vreg)": vreg,
        "Bandwidth": band,
        "TapPosition": tap
    })
    
    tap_data.append(tap)

# -------------------------------------------------------
# STEP 5: Display regulator data in console
# -------------------------------------------------------
print("\nRegulator Details:")
for info in regulator_info:
    print(
        f"{info['Name']:>6} | MonBus: {info['MonitoredBus']:<15} "
        f"| Target V: {info['TargetVoltage(Vreg)']} "
        f"| Band: {info['Bandwidth']} "
        f"| Tap: {info['TapPosition']}"
    )

# -------------------------------------------------------
# STEP 6: Plot regulator tap positions
# -------------------------------------------------------
plt.figure(figsize=(8, 5))
plt.bar(regulators, tap_data, color='teal')
plt.title("Tap Positions of Voltage Regulators (IEEE 123-bus)")
plt.xlabel("Regulator Name")
plt.ylabel("Tap Position")
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()


Plot Result:![[Figure_2_tap_positions.png]]



⚙️ 1. Why do you see 6 regulators (creg1a, creg2a, etc.)?

In the IEEE 123-bus feeder, some regulators are per-phase rather than three-phase combined.

Here’s what that means 👇

🧩 Types of Regulators
Type	Description	Example Names
3-phase regulator	One regulator controls all three phases (A, B, C) together	Reg1
Single-phase regulators	Each phase (A, B, or C) has its own independent control	creg1a, creg4b, etc.

So your system uses single-phase regulators, each controlling one phase on a certain line.

✅ That’s why you see:

creg1a  → Phase A regulator for feeder section 1  
creg2a  → Phase A regulator for section 2  
creg3a, creg3c → Regulators for phases A and C on section 3  
creg4a, creg4b, creg4c → Regulators for all 3 phases on section 4


Each one has its own settings (Target V, Band, and Tap).

⚡ 2. Why target voltages differ (120 V vs. 124 V)

These values (ForwardVreg) are the target regulated voltages, specified in line-to-neutral volts.

120 V ≈ 1.0 p.u. (per unit) at a base of 120 V.

124 V ≈ 1.033 p.u. — slightly boosted to compensate for voltage drops along the line.

So different regulators may have different targets depending on:

Distance from substation,

Expected load downstream,

Local design requirements.

💡 In other words:

Closer regulators → 120 V target (keep it nominal)

Farther regulators → 122 – 124 V (slightly boost to offset downstream voltage drops)

That’s why you see variation — it’s intentional, not an error.

🧩 Why this works

Each RegControl is tied to one Transformer.

The transformer’s taps define the actual mechanical tap range.

The RegControl only adjusts within that transformer’s tap range.