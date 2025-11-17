# Electronics Foundations

> **Circuit theory, components, and practical electronics for DIY projects**

---

## Table of Contents

1. [Basic Circuit Theory](#1-basic-circuit-theory)
2. [Passive Components](#2-passive-components)
3. [Active Components](#3-active-components)
4. [Digital Electronics](#4-digital-electronics)
5. [Power Electronics](#5-power-electronics)
6. [Analog Signal Processing](#6-analog-signal-processing)
7. [PCB Design Basics](#7-pcb-design-basics)
8. [Practical Measurement](#8-practical-measurement)

---

## 1. Basic Circuit Theory

### 1.1 Voltage, Current, and Resistance

**Voltage (V)**: Electric potential difference, measured in Volts
**Current (I)**: Flow of electric charge, measured in Amperes
**Resistance (R)**: Opposition to current flow, measured in Ohms (Ω)

**Ohm's Law** - The most important equation in electronics:
```
V = I × R

I = V / R

R = V / I
```

```python
import numpy as np
import matplotlib.pyplot as plt

# Ohm's Law visualization
def ohms_law_demo():
    """Demonstrate Ohm's Law relationships"""

    # Fixed voltage, varying resistance
    V = 5.0  # Volts
    R = np.linspace(1, 1000, 100)  # 1Ω to 1kΩ
    I = V / R  # Current in Amperes

    fig, axes = plt.subplots(2, 2, figsize=(12, 10))

    # V = I × R (fixed voltage)
    axes[0, 0].plot(R, I * 1000)  # Convert to mA
    axes[0, 0].set_xlabel('Resistance (Ω)')
    axes[0, 0].set_ylabel('Current (mA)')
    axes[0, 0].set_title(f'Ohm\'s Law: V={V}V (constant)')
    axes[0, 0].grid(True)
    axes[0, 0].set_xscale('log')

    # Fixed resistance, varying voltage
    R_fixed = 100  # Ohms
    V_range = np.linspace(0, 12, 100)
    I_range = V_range / R_fixed

    axes[0, 1].plot(V_range, I_range * 1000)
    axes[0, 1].set_xlabel('Voltage (V)')
    axes[0, 1].set_ylabel('Current (mA)')
    axes[0, 1].set_title(f'Ohm\'s Law: R={R_fixed}Ω (constant)')
    axes[0, 1].grid(True)

    # Power dissipation: P = V × I = I² × R = V² / R
    P = I * V  # Watts

    axes[1, 0].plot(R, P * 1000)  # Convert to mW
    axes[1, 0].set_xlabel('Resistance (Ω)')
    axes[1, 0].set_ylabel('Power (mW)')
    axes[1, 0].set_title(f'Power Dissipation: V={V}V')
    axes[1, 0].grid(True)
    axes[1, 0].set_xscale('log')

    # I-V characteristic
    V_sweep = np.linspace(0, 10, 100)
    resistors = [10, 100, 1000]

    for R_val in resistors:
        I_sweep = V_sweep / R_val
        axes[1, 1].plot(V_sweep, I_sweep * 1000, label=f'{R_val}Ω')

    axes[1, 1].set_xlabel('Voltage (V)')
    axes[1, 1].set_ylabel('Current (mA)')
    axes[1, 1].set_title('I-V Characteristics')
    axes[1, 1].legend()
    axes[1, 1].grid(True)

    plt.tight_layout()
    plt.savefig('ohms_law_demo.png')

    # Practical examples
    print("Ohm's Law Examples:")
    print("-" * 50)

    examples = [
        ("LED circuit", 5, 20e-3, None),
        ("Quadcopter ESC", 14.8, 30, None),
        ("Sensor pull-up", 3.3, None, 10e3),
    ]

    for name, v, i, r in examples:
        if r is None:
            r = v / i
            print(f"{name}: V={v}V, I={i*1000:.0f}mA → R={r:.1f}Ω")
        elif i is None:
            i = v / r
            print(f"{name}: V={v}V, R={r/1000:.1f}kΩ → I={i*1000:.2f}mA")

ohms_law_demo()
```

### 1.2 Kirchhoff's Laws

**Kirchhoff's Current Law (KCL)**: Sum of currents entering a node = sum leaving
```
ΣI_in = ΣI_out
```

**Kirchhoff's Voltage Law (KVL)**: Sum of voltages around a closed loop = 0
```
ΣV = 0
```

```python
# Example: Series vs Parallel resistors
def series_parallel_demo():
    """Compare series and parallel resistor combinations"""

    R1 = 100  # Ohms
    R2 = 200  # Ohms
    V_supply = 5  # Volts

    # Series: R_total = R1 + R2
    R_series = R1 + R2
    I_series = V_supply / R_series
    V1_series = I_series * R1
    V2_series = I_series * R2

    print("Series Circuit:")
    print(f"Total resistance: {R_series}Ω")
    print(f"Current: {I_series*1000:.2f}mA")
    print(f"Voltage across R1: {V1_series:.2f}V")
    print(f"Voltage across R2: {V2_series:.2f}V")
    print(f"Check KVL: {V1_series} + {V2_series} = {V1_series + V2_series:.2f}V (should be {V_supply}V)")

    # Parallel: 1/R_total = 1/R1 + 1/R2
    R_parallel = 1 / (1/R1 + 1/R2)
    I_total = V_supply / R_parallel
    I1_parallel = V_supply / R1
    I2_parallel = V_supply / R2

    print("\nParallel Circuit:")
    print(f"Total resistance: {R_parallel:.2f}Ω")
    print(f"Total current: {I_total*1000:.2f}mA")
    print(f"Current through R1: {I1_parallel*1000:.2f}mA")
    print(f"Current through R2: {I2_parallel*1000:.2f}mA")
    print(f"Check KCL: {I1_parallel*1000:.2f} + {I2_parallel*1000:.2f} = {(I1_parallel + I2_parallel)*1000:.2f}mA")

series_parallel_demo()
```

### 1.3 Power and Energy

```
Power (W) = V × I = I² × R = V² / R

Energy (J) = Power × Time

1 Wh = 3600 J
```

```python
# Battery capacity and flight time calculator
def battery_flight_time():
    """Calculate quadcopter flight time from battery specs"""

    # Battery specs
    cells = 4  # 4S LiPo
    voltage_per_cell = 3.7  # Nominal
    capacity_mah = 3000

    V_nominal = cells * voltage_per_cell
    capacity_ah = capacity_mah / 1000
    energy_wh = V_nominal * capacity_ah

    print("Battery Specifications:")
    print(f"Configuration: {cells}S")
    print(f"Nominal voltage: {V_nominal}V")
    print(f"Capacity: {capacity_mah}mAh ({capacity_ah}Ah)")
    print(f"Energy: {energy_wh:.1f}Wh ({energy_wh * 3600:.0f}J)")

    # Different flight modes
    flight_modes = {
        'Hover': 150,      # Watts
        'Cruise': 200,
        'Aggressive': 350,
    }

    print("\nEstimated Flight Times:")
    print("-" * 50)

    for mode, power in flight_modes.items():
        # Account for voltage sag and don't discharge below 80%
        usable_energy = energy_wh * 0.8
        time_hours = usable_energy / power
        time_minutes = time_hours * 60

        current = power / V_nominal

        print(f"{mode:12s}: {power:3d}W ({current:5.1f}A) → {time_minutes:5.1f} min")

battery_flight_time()
```

---

## 2. Passive Components

### 2.1 Resistors

**Purpose**: Limit current, divide voltage, pull-up/pull-down

**Types:**
- Carbon film (cheap, ±5%)
- Metal film (precise, ±1%)
- Wire-wound (high power)
- SMD (surface mount)

**Color Code:**
```
Band 1 & 2: Value digits
Band 3: Multiplier
Band 4: Tolerance

Example: Brown-Black-Red-Gold
1-0-×100-±5% = 1000Ω = 1kΩ ±5%
```

**Power Rating**: 1/4W, 1/2W, 1W, etc.
```python
def resistor_power_check(voltage, resistance):
    """Check if resistor power rating is adequate"""
    current = voltage / resistance
    power = voltage * current

    print(f"Voltage: {voltage}V")
    print(f"Resistance: {resistance}Ω")
    print(f"Current: {current*1000:.1f}mA")
    print(f"Power: {power*1000:.1f}mW")

    ratings = [0.125, 0.25, 0.5, 1.0, 2.0]
    for rating in ratings:
        if power < rating * 0.5:  # 50% derating for safety
            print(f"Recommended: {rating}W resistor (50% derated)")
            break

# Example: LED current limiting resistor
V_supply = 5.0
V_led = 2.0  # Red LED forward voltage
I_led = 0.020  # 20mA desired current
R = (V_supply - V_led) / I_led

print("LED Current Limiting Resistor:")
resistor_power_check(V_supply - V_led, R)
```

**Practical Use in Flight Controller:**
```cpp
// Pull-up resistor for I2C bus
// I2C typically uses 4.7kΩ pull-ups

// Signal conditioning for analog sensors
// Voltage divider to scale 5V to 3.3V:
// R1 = 10kΩ (to 5V)
// R2 = 20kΩ (to GND)
// V_out = 5V × 20k/(10k+20k) = 3.33V
```

### 2.2 Capacitors

**Purpose**: Store charge, filter noise, smooth power, AC coupling

**Key Parameters:**
- Capacitance (F, μF, nF, pF)
- Voltage rating (must exceed circuit voltage)
- ESR (Equivalent Series Resistance)
- Type (ceramic, electrolytic, film)

**Capacitor Equation:**
```
Q = C × V  (charge = capacitance × voltage)
I = C × dV/dt  (current charging/discharging)
```

**Types:**

| Type | Range | Use | Polarity |
|------|-------|-----|----------|
| Ceramic | 1pF - 10μF | Decoupling, HF | No |
| Electrolytic | 1μF - 1000μF+ | Power supply, bulk | Yes |
| Tantalum | 1μF - 100μF | Low ESR | Yes |
| Film | 1nF - 1μF | Precision, audio | No |

```python
def capacitor_sizing():
    """Size decoupling and power capacitors"""

    # Decoupling capacitors for digital ICs
    print("Decoupling Capacitor Selection:")
    print("-" * 50)

    capacitors = {
        '100nF ceramic': 'High frequency noise (>1MHz)',
        '10μF ceramic': 'Medium frequency (100kHz-1MHz)',
        '100μF electrolytic': 'Low frequency, bulk storage',
    }

    for cap, purpose in capacitors.items():
        print(f"{cap:20s}: {purpose}")

    # Ripple current calculation for ESC power
    print("\n\nESC Power Capacitor:")
    print("-" * 50)

    V_battery = 14.8  # 4S LiPo
    I_motor_peak = 30  # Amps
    frequency = 20e3  # 20 kHz PWM

    # Ripple voltage target: 0.1V
    dV = 0.1

    # C = I × dt / dV
    # For PWM, dt ≈ 1/(2×f)
    dt = 1 / (2 * frequency)
    C_required = I_motor_peak * dt / dV

    print(f"Peak current: {I_motor_peak}A")
    print(f"PWM frequency: {frequency/1000:.0f}kHz")
    print(f"Target ripple: {dV}V")
    print(f"Required capacitance: {C_required*1e6:.0f}μF")
    print(f"Recommended: {C_required*1e6*2:.0f}μF (2× safety factor)")
    print(f"Suggested: 2× 1000μF 25V low-ESR electrolytics")

capacitor_sizing()
```

**RC Time Constant:**
```
τ = R × C

V(t) = V₀ × (1 - e^(-t/τ))  (charging)
V(t) = V₀ × e^(-t/τ)         (discharging)
```

```python
def rc_circuit_demo():
    """Demonstrate RC charging and discharging"""

    R = 10e3  # 10kΩ
    C = 100e-6  # 100μF
    tau = R * C

    V_supply = 5.0

    t = np.linspace(0, 5*tau, 1000)
    V_charge = V_supply * (1 - np.exp(-t/tau))
    V_discharge = V_supply * np.exp(-t/tau)

    fig, axes = plt.subplots(2, 1, figsize=(10, 8))

    # Charging
    axes[0].plot(t*1000, V_charge)
    axes[0].axhline(y=V_supply*0.632, color='r', linestyle='--',
                    label=f'63.2% at τ={tau*1000:.1f}ms')
    axes[0].axvline(x=tau*1000, color='r', linestyle='--')
    axes[0].set_xlabel('Time (ms)')
    axes[0].set_ylabel('Voltage (V)')
    axes[0].set_title('RC Charging')
    axes[0].legend()
    axes[0].grid(True)

    # Discharging
    axes[1].plot(t*1000, V_discharge)
    axes[1].axhline(y=V_supply*0.368, color='r', linestyle='--',
                    label=f'36.8% at τ={tau*1000:.1f}ms')
    axes[1].axvline(x=tau*1000, color='r', linestyle='--')
    axes[1].set_xlabel('Time (ms)')
    axes[1].set_ylabel('Voltage (V)')
    axes[1].set_title('RC Discharging')
    axes[1].legend()
    axes[1].grid(True)

    plt.tight_layout()
    plt.savefig('rc_circuit.png')

    print(f"RC Time Constant: τ = {tau*1000:.1f}ms")
    print(f"Time to 99% charged: {5*tau*1000:.1f}ms")

rc_circuit_demo()
```

### 2.3 Inductors

**Purpose**: Store energy in magnetic field, filter, DC-DC converters

**Key Parameters:**
- Inductance (H, mH, μH)
- Current rating
- DC resistance (DCR)
- Saturation current

**Inductor Equation:**
```
V = L × dI/dt

E = ½ × L × I²  (energy stored)
```

**Uses:**
- LC filters
- DC-DC converter (buck, boost)
- EMI suppression
- Motor windings

```python
def inductor_energy():
    """Calculate energy stored in inductor"""

    L = 10e-6  # 10μH (typical for buck converter)
    I_peak = 3.0  # Amps

    E = 0.5 * L * I_peak**2

    print(f"Inductance: {L*1e6:.1f}μH")
    print(f"Peak current: {I_peak}A")
    print(f"Energy stored: {E*1e6:.2f}μJ")

    # Buck converter inductor selection
    V_in = 12.0
    V_out = 5.0
    I_out = 2.0
    f_sw = 100e3  # 100kHz switching
    ripple = 0.3  # 30% current ripple

    # L = (V_in - V_out) × D / (f_sw × ΔI)
    # where D = V_out / V_in (duty cycle)
    D = V_out / V_in
    delta_I = ripple * I_out

    L_required = (V_in - V_out) * D / (f_sw * delta_I)

    print("\nBuck Converter Design:")
    print(f"Input: {V_in}V → Output: {V_out}V @ {I_out}A")
    print(f"Switching frequency: {f_sw/1000:.0f}kHz")
    print(f"Required inductance: {L_required*1e6:.1f}μH")
    print(f"Peak current: {I_out + delta_I/2:.2f}A")

inductor_energy()
```

---

## 3. Active Components

### 3.1 Diodes

**Function**: Allow current in one direction only

**I-V Characteristic:**
```
Forward voltage drop:
- Silicon: ~0.7V
- Schottky: ~0.3V
- LED: 1.8-3.3V (color dependent)
```

**Types:**
- Rectifier diode (1N4001-4007)
- Schottky (fast, low Vf)
- Zener (voltage regulation)
- LED (light emission)

```python
def diode_circuit():
    """Diode applications"""

    print("Diode Applications:")
    print("-" * 50)

    # LED current calculation
    V_supply = 5.0
    V_led = 2.0  # Red LED
    I_desired = 0.020  # 20mA

    R_series = (V_supply - V_led) / I_desired
    P_resistor = I_desired**2 * R_series

    print("LED Circuit:")
    print(f"Supply: {V_supply}V")
    print(f"LED forward voltage: {V_led}V")
    print(f"Desired current: {I_desired*1000:.0f}mA")
    print(f"Series resistor: {R_series:.0f}Ω")
    print(f"Power dissipation: {P_resistor*1000:.1f}mW")

    # Flyback diode for motor/relay
    print("\nFlyback Diode (Motor Protection):")
    print("When motor turns off, inductor generates voltage spike")
    print("Diode clamps it to V_supply")
    print("Use: 1N4001 or similar (>1A, >50V)")

    # Reverse polarity protection
    print("\nReverse Polarity Protection:")
    V_battery = 14.8
    I_max = 30
    V_diode = 0.3  # Schottky
    P_loss = V_diode * I_max

    print(f"Battery: {V_battery}V")
    print(f"Max current: {I_max}A")
    print(f"Voltage drop: {V_diode}V")
    print(f"Power loss: {P_loss:.1f}W")
    print(f"Suggested: MBRS340 (40V 3A Schottky)")

diode_circuit()
```

### 3.2 Transistors

**BJT (Bipolar Junction Transistor)**:
- NPN / PNP
- Current amplifier
- Saturation switch

**MOSFET (Metal-Oxide-Semiconductor FET)**:
- N-channel / P-channel
- Voltage controlled
- Low on-resistance

```python
def transistor_switch():
    """Transistor as switch for LED/motor control"""

    print("Transistor Switch Design:")
    print("-" * 50)

    # NPN BJT switch
    print("\nBJT (NPN) Configuration:")
    V_cc = 5.0
    I_load = 0.1  # 100mA load
    V_be = 0.7  # Base-emitter voltage
    hFE = 100  # Current gain
    V_gpio = 3.3  # MCU output

    I_base = I_load / (hFE * 0.5)  # Use half hFE for saturation
    R_base = (V_gpio - V_be) / I_base

    print(f"Load current: {I_load*1000:.0f}mA")
    print(f"Required base current: {I_base*1000:.2f}mA")
    print(f"Base resistor: {R_base:.0f}Ω (use {int(R_base/100)*100}Ω)")

    # MOSFET switch
    print("\n\nMOSFET (N-channel) Configuration:")
    V_motor = 12.0
    I_motor = 30.0  # 30A motor
    V_gs_th = 2.5  # Gate threshold
    R_ds_on = 0.01  # 10mΩ on-resistance

    P_loss = I_motor**2 * R_ds_on

    print(f"Motor voltage: {V_motor}V")
    print(f"Motor current: {I_motor}A")
    print(f"MOSFET R_ds(on): {R_ds_on*1000:.0f}mΩ")
    print(f"Power dissipation: {P_loss:.1f}W")
    print(f"Suggested: IRLB8721 (or similar low R_ds)")

    # Gate resistor
    R_gate = 100  # Ohms
    C_gate = 1000e-12  # 1nF typical
    tau = R_gate * C_gate

    print(f"\nGate drive:")
    print(f"Gate resistor: {R_gate}Ω")
    print(f"Switching time: ~{tau*1e9:.0f}ns")

transistor_switch()
```

### 3.3 Operational Amplifiers (Op-Amps)

**Ideal Op-Amp Rules:**
1. Infinite input impedance (no current into inputs)
2. Zero output impedance
3. Infinite gain
4. Inputs at same voltage (in negative feedback)

**Common Configurations:**

```python
def opamp_circuits():
    """Op-amp circuit calculations"""

    print("Op-Amp Circuits:")
    print("=" * 50)

    # Non-inverting amplifier
    print("\n1. Non-Inverting Amplifier:")
    print("   Gain = 1 + R2/R1")
    R1 = 10e3
    R2 = 100e3
    gain = 1 + R2/R1
    print(f"   R1={R1/1e3:.0f}kΩ, R2={R2/1e3:.0f}kΩ")
    print(f"   Gain = {gain:.1f} ({20*np.log10(gain):.1f}dB)")

    # Inverting amplifier
    print("\n2. Inverting Amplifier:")
    print("   Gain = -R2/R1")
    gain_inv = -R2/R1
    print(f"   R1={R1/1e3:.0f}kΩ, R2={R2/1e3:.0f}kΩ")
    print(f"   Gain = {gain_inv:.1f}")

    # Voltage follower (buffer)
    print("\n3. Voltage Follower (Buffer):")
    print("   Gain = 1, very high input impedance")
    print("   Use: Isolate sensitive circuits")

    # Summing amplifier
    print("\n4. Summing Amplifier:")
    print("   V_out = -Rf × (V1/R1 + V2/R2 + V3/R3)")
    print("   Use: Audio mixing, DAC")

    # Active low-pass filter
    print("\n5. Active Low-Pass Filter:")
    R = 10e3
    C = 10e-9  # 10nF
    fc = 1 / (2 * np.pi * R * C)
    print(f"   R={R/1e3:.0f}kΩ, C={C*1e9:.0f}nF")
    print(f"   Cutoff frequency: {fc:.0f}Hz")
    print("   Use: Anti-aliasing for ADC")

    # Radar IF amplifier
    print("\n6. Radar IF Amplifier Design:")
    V_signal = 0.01  # 10mV from mixer
    V_adc_max = 3.3
    V_desired = 1.0  # Use 1/3 of ADC range

    gain_needed = V_desired / V_signal
    R1_radar = 1e3
    R2_radar = R1_radar * (gain_needed - 1)

    print(f"   Input signal: {V_signal*1000:.1f}mV")
    print(f"   Desired output: {V_desired:.1f}V")
    print(f"   Required gain: {gain_needed:.0f} ({20*np.log10(gain_needed):.1f}dB)")
    print(f"   R1={R1_radar/1e3:.0f}kΩ, R2={R2_radar/1e3:.0f}kΩ")
    print(f"   Op-amp: LM358 (dual, cheap) or OPA2134 (low noise)")

opamp_circuits()
```

---

## 4. Digital Electronics

### 4.1 Logic Levels

```
TTL (Transistor-Transistor Logic):
  Logic 0: 0V - 0.8V
  Logic 1: 2V - 5V

CMOS (3.3V):
  Logic 0: 0V - 0.8V
  Logic 1: 2.4V - 3.3V
```

**Level Shifting:**
- 5V → 3.3V: Voltage divider or level shifter IC
- 3.3V → 5V: Level shifter IC (BSS138-based or dedicated)

```python
def logic_level_shifter():
    """Design voltage divider for 5V→3.3V level shift"""

    V_in = 5.0
    V_out_target = 3.3
    I_max = 0.001  # 1mA max current (low power)

    # Voltage divider: V_out = V_in × R2/(R1+R2)
    # Choose R1+R2 for desired current
    R_total = V_in / I_max

    # R2/R_total = V_out/V_in
    ratio = V_out_target / V_in
    R2 = ratio * R_total
    R1 = R_total - R2

    print("5V → 3.3V Level Shifter (Voltage Divider):")
    print(f"R1 (to 5V): {R1/1000:.1f}kΩ (use {int(R1/1000)}kΩ)")
    print(f"R2 (to GND): {R2/1000:.1f}kΩ (use {int(R2/1000)}kΩ)")
    print(f"Output voltage: {V_in * R2/(R1+R2):.2f}V")
    print(f"Max current: {I_max*1000:.2f}mA")
    print("\nNote: This works for slow signals only!")
    print("For I2C/SPI, use proper bidirectional level shifter")

logic_level_shifter()
```

### 4.2 Pull-up and Pull-down Resistors

```cpp
// Pull-up resistor (common for I2C, buttons)
// When switch open: Pin reads HIGH (pulled to Vcc)
// When switch closed: Pin reads LOW (grounded)

const int BUTTON_PIN = 12;

void setup() {
    pinMode(BUTTON_PIN, INPUT_PULLUP);  // Internal pull-up
}

void loop() {
    if (digitalRead(BUTTON_PIN) == LOW) {
        // Button pressed (pulls to ground)
    }
}
```

**Typical values:**
- I2C: 4.7kΩ - 10kΩ
- General purpose: 10kΩ - 100kΩ
- Stronger pull: 1kΩ - 4.7kΩ

### 4.3 Communication Protocols

**UART (Serial):**
- Asynchronous
- 2 wires: TX, RX
- Common baud rates: 9600, 115200
- Use for: GPS, telemetry

**I2C (Inter-Integrated Circuit):**
- Synchronous, 2 wires: SDA, SCL
- Multi-device (addresses)
- Requires pull-ups (4.7kΩ typical)
- Use for: IMU, barometer, OLED

**SPI (Serial Peripheral Interface):**
- Synchronous, 4 wires: MOSI, MISO, SCK, CS
- Faster than I2C
- One CS per device
- Use for: SD card, high-speed sensors

```cpp
// I2C communication example
#include <Wire.h>

#define MPU6050_ADDR 0x68

void readMPU6050() {
    Wire.beginTransmission(MPU6050_ADDR);
    Wire.write(0x3B);  // Start at register 0x3B (ACCEL_XOUT_H)
    Wire.endTransmission(false);
    Wire.requestFrom(MPU6050_ADDR, 6, true);

    int16_t accel_x = Wire.read() << 8 | Wire.read();
    int16_t accel_y = Wire.read() << 8 | Wire.read();
    int16_t accel_z = Wire.read() << 8 | Wire.read();

    // Convert to m/s²
    float ax = accel_x / 16384.0 * 9.81;  // For ±2g range
    float ay = accel_y / 16384.0 * 9.81;
    float az = accel_z / 16384.0 * 9.81;
}
```

---

## 5. Power Electronics

### 5.1 Voltage Regulators

**Linear Regulators:**
- Simple, cheap
- Dissipate power as heat: P = (V_in - V_out) × I
- Efficiency: V_out / V_in
- Use for: Low current, low noise

**Common ICs:**
- 7805: 5V, 1A
- LM1117: 3.3V, 800mA
- AMS1117: 3.3V, 1A

```python
def linear_regulator_design():
    """Calculate heat dissipation in linear regulator"""

    V_in = 14.8  # 4S LiPo
    V_out = 5.0
    I_load = 0.5  # 500mA

    V_drop = V_in - V_out
    P_dissipated = V_drop * I_load
    efficiency = V_out / V_in

    print("Linear Regulator Analysis:")
    print(f"Input: {V_in}V")
    print(f"Output: {V_out}V @ {I_load*1000:.0f}mA")
    print(f"Voltage drop: {V_drop}V")
    print(f"Power dissipated: {P_dissipated:.2f}W")
    print(f"Efficiency: {efficiency*100:.1f}%")
    print(f"\nHeatsink required: {P_dissipated > 1.0}")

    # Thermal calculation
    T_ambient = 25  # °C
    T_junction_max = 125  # °C
    theta_ja = 50  # °C/W (typical TO-220 without heatsink)

    T_rise = P_dissipated * theta_ja
    T_junction = T_ambient + T_rise

    print(f"\nThermal Analysis:")
    print(f"Temperature rise: {T_rise:.1f}°C")
    print(f"Junction temp: {T_junction:.1f}°C")
    if T_junction > T_junction_max:
        print("WARNING: Exceeds max temperature! Add heatsink.")

linear_regulator_design()
```

**Switching Regulators (DC-DC Converters):**
- Buck (step-down): V_out < V_in
- Boost (step-up): V_out > V_in
- Buck-boost: Either direction
- Efficiency: 85-95%

```python
def switching_regulator():
    """Buck converter design"""

    V_in = 14.8
    V_out = 5.0
    I_out = 3.0  # 3A
    f_sw = 500e3  # 500kHz switching

    # Duty cycle
    D = V_out / V_in

    # Output power
    P_out = V_out * I_out

    # Assume 90% efficiency
    eff = 0.90
    P_in = P_out / eff
    I_in = P_in / V_in

    print("Buck Converter Design:")
    print(f"Input: {V_in}V → Output: {V_out}V @ {I_out}A")
    print(f"Switching frequency: {f_sw/1000:.0f}kHz")
    print(f"Duty cycle: {D*100:.1f}%")
    print(f"Efficiency: {eff*100:.0f}%")
    print(f"Input current: {I_in:.2f}A")
    print(f"Power loss: {P_in - P_out:.2f}W")

    # Component selection
    ripple_current = 0.3  # 30% ripple
    delta_I = ripple_current * I_out

    # Inductor
    L = V_out * (1 - D) / (f_sw * delta_I)

    # Capacitor (output)
    ripple_voltage = 0.05  # 50mV
    ESR = 0.05  # 50mΩ typical
    C_min = delta_I / (8 * f_sw * ripple_voltage)

    print(f"\nComponent Selection:")
    print(f"Inductor: {L*1e6:.1f}μH, {I_out + delta_I/2:.1f}A saturation")
    print(f"Output cap: {C_min*1e6:.0f}μF minimum, low ESR")
    print(f"Suggested IC: LM2596 or XL4015")

switching_regulator()
```

### 5.2 Battery Management

**LiPo Battery Safety:**

```python
def lipo_monitor():
    """LiPo battery monitoring and protection"""

    cells = 4
    V_full = 4.2  # Per cell
    V_nominal = 3.7
    V_empty = 3.0  # Safe cutoff
    V_critical = 3.3  # Warning threshold

    print("LiPo Battery Monitoring:")
    print("=" * 50)
    print(f"Configuration: {cells}S")
    print(f"\nPer-cell voltages:")
    print(f"  Fully charged: {V_full}V")
    print(f"  Nominal: {V_nominal}V")
    print(f"  Warning: {V_critical}V")
    print(f"  Cutoff: {V_empty}V")

    print(f"\nTotal pack voltages:")
    print(f"  Fully charged: {cells * V_full}V")
    print(f"  Nominal: {cells * V_nominal}V")
    print(f"  Warning: {cells * V_critical}V")
    print(f"  Cutoff: {cells * V_empty}V")

    # Voltage divider for monitoring
    V_max = cells * V_full
    V_adc_max = 3.3

    # Design voltage divider
    ratio = V_adc_max / V_max * 0.9  # 90% of ADC range for safety
    R2 = 10e3  # 10kΩ to GND
    R1 = R2 * (1/ratio - 1)

    print(f"\nVoltage Divider for ADC Monitoring:")
    print(f"  R1 (to battery): {R1/1000:.1f}kΩ")
    print(f"  R2 (to GND): {R2/1000:.0f}kΩ")
    print(f"  At full charge: {V_max * ratio:.2f}V → ADC")

    # Warning levels in ADC counts (12-bit)
    adc_full = int(V_max * ratio / V_adc_max * 4095)
    adc_warn = int(cells * V_critical * ratio / V_adc_max * 4095)
    adc_cutoff = int(cells * V_empty * ratio / V_adc_max * 4095)

    print(f"\nADC Threshold Values (12-bit):")
    print(f"  Full: {adc_full}")
    print(f"  Warning: {adc_warn}")
    print(f"  Cutoff: {adc_cutoff}")

lipo_monitor()
```

**Charging:**
- Use dedicated LiPo charger (IMAX B6, etc.)
- CC-CV charging (Constant Current, then Constant Voltage)
- Never exceed 4.2V per cell
- Balance charging for multi-cell packs

---

## 6. Analog Signal Processing

### 6.1 Filters

**Low-Pass Filter (RC):**
```
Cutoff frequency: fc = 1 / (2πRC)
```

**Use:** Anti-aliasing before ADC

```python
def design_lpf():
    """Design low-pass filter for anti-aliasing"""

    f_sample = 100e3  # 100kHz ADC sample rate
    f_cutoff = f_sample / 2.5  # Nyquist / 2.5 for safety

    # Choose C, calculate R
    C = 10e-9  # 10nF (common value)
    R = 1 / (2 * np.pi * f_cutoff * C)

    print("Anti-Aliasing Low-Pass Filter:")
    print(f"ADC sample rate: {f_sample/1000:.0f}kHz")
    print(f"Cutoff frequency: {f_cutoff/1000:.1f}kHz")
    print(f"C: {C*1e9:.0f}nF")
    print(f"R: {R/1000:.1f}kΩ (use {int(R/1000)}kΩ)")

    # Frequency response
    f = np.logspace(2, 6, 1000)  # 100Hz to 1MHz
    H = 1 / np.sqrt(1 + (f / f_cutoff)**2)
    H_dB = 20 * np.log10(H)

    plt.figure(figsize=(10, 6))
    plt.semilogx(f/1000, H_dB)
    plt.axvline(x=f_cutoff/1000, color='r', linestyle='--', label=f'fc={f_cutoff/1000:.1f}kHz')
    plt.axhline(y=-3, color='g', linestyle='--', label='-3dB')
    plt.xlabel('Frequency (kHz)')
    plt.ylabel('Gain (dB)')
    plt.title('Low-Pass Filter Response')
    plt.grid(True, which='both')
    plt.legend()
    plt.savefig('lpf_response.png')

design_lpf()
```

### 6.2 Amplifiers

**Instrumentation Amplifier:**
- High input impedance
- Low noise
- Precise gain
- Use for: Weak sensor signals

**Example: Amplify radar IF signal**

```python
def radar_if_amplifier():
    """Design IF amplifier for radar receiver"""

    # Radar mixer output
    V_if_min = 0.001  # 1mV minimum signal
    V_if_max = 0.1    # 100mV strong signal

    # ADC input range
    V_adc_ref = 3.3
    V_adc_optimal = 1.0  # Use middle range

    # Required gain
    gain = V_adc_optimal / V_if_max
    gain_dB = 20 * np.log10(gain)

    print("Radar IF Amplifier Design:")
    print(f"Input range: {V_if_min*1000:.1f}mV - {V_if_max*1000:.0f}mV")
    print(f"Output range: 0V - {V_adc_optimal}V")
    print(f"Required gain: {gain:.1f} ({gain_dB:.1f}dB)")

    # Op-amp non-inverting configuration
    R1 = 1e3  # 1kΩ
    R2 = R1 * (gain - 1)

    print(f"\nNon-inverting amplifier:")
    print(f"R1 = {R1/1e3:.0f}kΩ")
    print(f"R2 = {R2/1e3:.1f}kΩ (use {int(R2/1000)}kΩ)")

    # Bandwidth requirement
    f_if_max = 10e3  # 10kHz maximum beat frequency
    BW_required = f_if_max * 10  # 10× for safety

    print(f"\nBandwidth requirement: {BW_required/1000:.0f}kHz")
    print(f"Suggested op-amp: LM358 (1MHz BW) or TL072 (3MHz BW)")

    # Noise analysis
    noise_density = 20e-9  # 20nV/√Hz (typical op-amp)
    noise_voltage = noise_density * np.sqrt(BW_required)
    SNR = V_if_min / noise_voltage
    SNR_dB = 20 * np.log10(SNR)

    print(f"\nNoise Analysis:")
    print(f"Op-amp noise density: {noise_density*1e9:.0f}nV/√Hz")
    print(f"Total noise: {noise_voltage*1e9:.1f}nV")
    print(f"SNR (for 1mV signal): {SNR_dB:.1f}dB")

radar_if_amplifier()
```

---

## 7. PCB Design Basics

### 7.1 Design Guidelines

**Trace Width:**
```python
def trace_width_calculator(current, temp_rise=10, copper_thickness=1):
    """
    Calculate PCB trace width
    IPC-2221 formula
    """
    # Constants
    k = 0.048  # External traces
    b = 0.44
    c = 0.725

    # Area in square mils
    A = (current / (k * temp_rise**b))**(1/c)

    # Convert to width (assuming 1oz copper = 1.378 mils thick)
    thickness_mils = copper_thickness * 1.378
    width_mils = A / thickness_mils
    width_mm = width_mils * 0.0254

    print(f"Trace Width Calculator:")
    print(f"Current: {current}A")
    print(f"Temperature rise: {temp_rise}°C")
    print(f"Copper weight: {copper_thickness}oz")
    print(f"Minimum trace width: {width_mm:.2f}mm ({width_mils:.1f}mils)")

    return width_mm

# Examples
print("Power traces:")
trace_width_calculator(1, 10)
print()
trace_width_calculator(5, 10)
print()
trace_width_calculator(30, 20)  # ESC to motor
```

**Ground Plane:**
- Always use ground plane (reduces noise, impedance)
- Pour ground on both sides if possible
- Via stitching between layers

**Decoupling:**
- 100nF ceramic near every IC power pin
- Bulk capacitors (10-100μF) at power entry

**Signal Routing:**
- Keep high-speed traces short
- Differential pairs matched length
- Analog/digital ground separation

### 7.2 Component Placement

```
Power Section (left):
  Battery connector
  ↓
  Power switch
  ↓
  Voltage regulators
  ↓
  Bulk capacitors

MCU Section (center):
  Microcontroller
  Crystal oscillator
  Decoupling caps
  Programming header

Sensor Section (right):
  IMU, barometer
  Pull-up resistors
  Filtering caps

Motor Outputs (edges):
  ESC connectors
  Gate drivers (if needed)
```

---

## 8. Practical Measurement

### 8.1 Multimeter Usage

**Voltage Measurement:**
- Select DC or AC
- Probe: Red=positive, Black=ground
- Measure in parallel

**Current Measurement:**
- Select appropriate range (mA or A)
- Connect in series with load
- Watch for blown fuse!

**Resistance:**
- Power OFF circuit
- Discharge capacitors first
- Measure across component

```python
def measurement_guide():
    """Common measurements for debugging"""

    print("Electronics Measurement Guide")
    print("=" * 50)

    tests = {
        "Power supply voltage": "Measure at battery terminals (expect 11-17V for 3S-4S)",
        "5V regulator output": "Measure after regulator (expect 4.9-5.1V)",
        "3.3V MCU power": "Measure at MCU VCC pin (expect 3.2-3.4V)",
        "ESC signal": "Measure PWM signal (expect 1000-2000μs pulses)",
        "Motor phase": "Measure resistance between motor wires (expect <1Ω)",
        "I2C pull-ups": "Measure SCL/SDA when idle (expect 3.3V or 5V)",
        "Battery current": "Measure in series with battery (expect 0-60A)",
    }

    for test, expected in tests.items():
        print(f"\n{test}:")
        print(f"  {expected}")

    print("\n\nCommon Faults:")
    print("-" * 50)
    faults = {
        "No voltage": "Check connections, fuse, power switch",
        "Low voltage under load": "Weak battery, high resistance connections",
        "Intermittent connection": "Cold solder joint, broken wire",
        "Overcurrent": "Short circuit, damaged component",
        "No I2C response": "Missing pull-ups, wrong address, damaged sensor",
    }

    for fault, diagnosis in faults.items():
        print(f"{fault}: {diagnosis}")

measurement_guide()
```

### 8.2 Oscilloscope Basics

**Key Measurements:**
- Signal amplitude (V)
- Frequency (Hz)
- Duty cycle (%)
- Rise/fall time
- Noise level

**Triggering:**
- Edge trigger: Capture signal at specific voltage
- Single shot: Capture one event
- Auto: Continuous refresh

**Example: Measure PWM signal**
```
Expected PWM for ESC:
- Frequency: 50Hz (20ms period)
- Pulse width: 1000-2000μs
- Amplitude: 3.3V or 5V
- Clean edges (fast rise/fall)
```

---

## Summary

You now understand:

✅ **Circuit theory** - Ohm's law, Kirchhoff's laws, power calculations
✅ **Passive components** - Resistors, capacitors, inductors and their applications
✅ **Active components** - Diodes, transistors, op-amps for signal processing
✅ **Digital electronics** - Logic levels, protocols (I2C, SPI, UART)
✅ **Power electronics** - Regulators, battery management, DC-DC converters
✅ **Signal conditioning** - Filters, amplifiers for sensors and radar
✅ **PCB design** - Layout guidelines, trace sizing, grounding
✅ **Measurement** - Using multimeter and oscilloscope

**Key Takeaways:**

1. Always calculate power dissipation - components overheat!
2. Decouple every IC - 100nF ceramic capacitor
3. Use appropriate wire gauge for current
4. Ground plane is your friend
5. Measure twice, solder once

---

**Next Steps:**

- **[Q4: Motors & ESCs](../quadcopter/Q4-motors-escs.md)** - Apply these concepts to motor control
- **[Q5: Sensors & IMU](../quadcopter/Q5-sensors-imu.md)** - Sensor interface circuits
- **[R4: RF Components](../radar/R4-rf-components.md)** - RF-specific electronics

---

## Practice Problems

1. Calculate current-limiting resistor for 3.3V LED (Vf=2.1V) at 15mA
2. Design RC low-pass filter with fc=1kHz using 10nF capacitor
3. Select MOSFET for 20A motor at 12V (calculate power dissipation)
4. Calculate trace width for 10A at 20°C rise on 1oz copper
5. Design voltage divider to measure 4S LiPo (16.8V max) with 3.3V ADC

**Solutions in:** [exercises/electronics-solutions.md](../exercises/electronics-solutions.md)

---

*Last Updated: November 2025*
