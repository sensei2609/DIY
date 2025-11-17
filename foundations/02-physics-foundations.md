# Physics Foundations

> **Core physics principles for flight dynamics and electromagnetic wave propagation**

---

## Table of Contents

1. [Classical Mechanics & Dynamics](#1-classical-mechanics--dynamics)
2. [Rotational Dynamics & Kinematics](#2-rotational-dynamics--kinematics)
3. [Fluid Dynamics & Aerodynamics](#3-fluid-dynamics--aerodynamics)
4. [Electromagnetism & Wave Theory](#4-electromagnetism--wave-theory)
5. [Thermodynamics (Battery Chemistry)](#5-thermodynamics-battery-chemistry)
6. [Measurement & Units](#6-measurement--units)

---

## 1. Classical Mechanics & Dynamics

### 1.1 Newton's Laws of Motion

#### First Law: Inertia
**An object at rest stays at rest, an object in motion stays in motion unless acted upon by an external force.**

For a quadcopter in flight:
- Without thrust, it falls due to gravity
- Without drag, it would continue at constant velocity forever
- Stabilization requires continuous force adjustments

#### Second Law: F = ma

This is THE fundamental equation for quadcopter control:

```
F = ma
∑F = ma

For multiple forces:
F_net = F_thrust + F_gravity + F_drag = ma
```

**Example: Hovering Quadcopter**

```python
import numpy as np
import matplotlib.pyplot as plt

# Quadcopter parameters
m = 1.5  # kg (mass of quadcopter)
g = 9.81  # m/s² (gravitational acceleration)

# Forces in hover
F_gravity = -m * g  # Downward (negative)
F_thrust_required = m * g  # Upward to balance

print(f"Mass: {m} kg")
print(f"Weight (gravitational force): {-F_gravity:.2f} N downward")
print(f"Thrust required for hover: {F_thrust_required:.2f} N upward")
print(f"Net force in hover: {F_gravity + F_thrust_required:.2f} N")

# What if thrust is not exactly balanced?
thrust_values = np.linspace(0, 25, 100)
accelerations = (thrust_values - m*g) / m

plt.figure(figsize=(10, 6))
plt.plot(thrust_values, accelerations)
plt.axhline(y=0, color='r', linestyle='--', label='Hover (a=0)')
plt.axvline(x=F_thrust_required, color='g', linestyle='--',
            label=f'Required thrust = {F_thrust_required:.1f} N')
plt.fill_between(thrust_values, 0, accelerations,
                 where=(thrust_values < F_thrust_required),
                 alpha=0.3, color='red', label='Descending')
plt.fill_between(thrust_values, 0, accelerations,
                 where=(thrust_values > F_thrust_required),
                 alpha=0.3, color='green', label='Ascending')
plt.xlabel('Thrust (N)')
plt.ylabel('Acceleration (m/s²)')
plt.title('Quadcopter Acceleration vs Thrust')
plt.legend()
plt.grid(True)
plt.savefig('thrust_acceleration.png')
```

#### Third Law: Action-Reaction

**For every action, there is an equal and opposite reaction.**

For quadcopters:
- Propellers push air down → Air pushes quadcopter up
- This is how thrust is generated!

```python
# Momentum change of air = thrust force on quadcopter

# Simplified propeller model
rho_air = 1.225  # kg/m³ (air density at sea level)
A_prop = 0.02  # m² (propeller disk area)
v_downwash = 10  # m/s (air velocity through propeller)

# Mass flow rate of air
m_dot_air = rho_air * A_prop * v_downwash

# Momentum thrust
F_thrust_momentum = m_dot_air * v_downwash

print(f"Air mass flow rate: {m_dot_air:.3f} kg/s")
print(f"Thrust from momentum theory: {F_thrust_momentum:.2f} N")
```

### 1.2 Energy and Work

#### Kinetic Energy
```
KE = ½mv²
```

```python
# Calculate energy required to accelerate quadcopter
m = 1.5  # kg
v_initial = 0  # m/s (starting from rest)
v_final = 10  # m/s (target velocity)

KE_initial = 0.5 * m * v_initial**2
KE_final = 0.5 * m * v_final**2
work_required = KE_final - KE_initial

print(f"Kinetic energy gained: {work_required:.1f} J")

# If we apply constant thrust for time t
F_thrust = 20  # N
t_accel = m * (v_final - v_initial) / F_thrust

print(f"Acceleration time: {t_accel:.2f} s")
print(f"Distance covered: {0.5 * (v_final/t_accel) * t_accel**2:.2f} m")
```

#### Potential Energy
```
PE = mgh
```

```python
# Energy cost of climbing
h_climb = 100  # m (climb to 100m altitude)
PE_gained = m * g * h_climb

print(f"Potential energy gained: {PE_gained:.1f} J")

# Battery capacity needed (assuming 50% efficiency)
efficiency = 0.5
energy_from_battery = PE_gained / efficiency

battery_voltage = 14.8  # V (4S LiPo)
charge_needed = energy_from_battery / battery_voltage

print(f"Energy from battery: {energy_from_battery:.1f} J")
print(f"Charge needed: {charge_needed:.1f} As = {charge_needed/3600:.4f} Ah")
```

#### Power
```
Power = Energy / Time = Force × Velocity
```

```python
# Power required for vertical climb
v_climb = 5  # m/s (climbing at 5 m/s)
F_total = m * g + m * 2  # Thrust for hover + acceleration

P_mechanical = F_total * v_climb
P_electrical = P_mechanical / efficiency

print(f"Mechanical power: {P_mechanical:.1f} W")
print(f"Electrical power: {P_electrical:.1f} W")
print(f"Current draw at {battery_voltage}V: {P_electrical/battery_voltage:.1f} A")
```

### 1.3 Momentum and Impulse

```
p = mv (momentum)
Impulse = Δp = FΔt
```

**Application: ESC/Motor Response Time**

```python
# When you command a thrust change, motors take time to respond
# This affects control loop performance

motor_response_time = 0.1  # seconds (typical for hobby motors)

# Commanded thrust change
F_delta = 5  # N (sudden 5N increase command)

# Time to reach 63.2% of commanded change (first-order system)
t_63 = motor_response_time

time = np.linspace(0, 0.5, 1000)
thrust_response = F_delta * (1 - np.exp(-time/motor_response_time))

plt.figure(figsize=(10, 6))
plt.plot(time, thrust_response)
plt.axhline(y=F_delta, color='r', linestyle='--', label='Commanded')
plt.axhline(y=F_delta*0.632, color='g', linestyle='--',
            label=f'63.2% at τ={motor_response_time}s')
plt.axvline(x=motor_response_time, color='g', linestyle='--')
plt.xlabel('Time (s)')
plt.ylabel('Thrust (N)')
plt.title('Motor/ESC First-Order Response')
plt.legend()
plt.grid(True)
plt.savefig('motor_response.png')

print(f"Time to 95% response: {3*motor_response_time:.3f} s")
```

This is why PID derivative time constants must be tuned relative to motor response!

---

## 2. Rotational Dynamics & Kinematics

### 2.1 Angular Quantities

#### Angular Position, Velocity, Acceleration

```
θ = angle (radians)
ω = dθ/dt (angular velocity, rad/s)
α = dω/dt (angular acceleration, rad/s²)
```

**Conversion to linear motion:**
```
v = ωr (linear velocity)
a = αr (tangential acceleration)
a_c = ω²r (centripetal acceleration)
```

```python
# Propeller spinning
rpm = 6000  # revolutions per minute
omega = rpm * 2 * np.pi / 60  # Convert to rad/s

print(f"Angular velocity: {omega:.1f} rad/s")

# Tip of a 10-inch (0.254m) propeller
r_tip = 0.254 / 2  # m (radius)
v_tip = omega * r_tip

print(f"Tip speed: {v_tip:.1f} m/s")
print(f"Tip speed: {v_tip * 3.6:.1f} km/h")

# Centripetal acceleration at tip
a_centripetal = omega**2 * r_tip
g_force = a_centripetal / 9.81

print(f"Centripetal acceleration: {a_centripetal:.1f} m/s²")
print(f"That's {g_force:.0f}g!")
```

### 2.2 Moment of Inertia

The rotational equivalent of mass:

```
I = ∫ r² dm
```

For common shapes:
```
Point mass:           I = mr²
Thin rod (center):    I = (1/12)mL²
Thin disk (center):   I = (1/2)mr²
Solid sphere:         I = (2/5)mr²
```

**For a quadcopter:**

```python
# Approximate quadcopter as:
# - Central body (cylinder)
# - Four arms with motors/props

# Central body
m_body = 0.8  # kg
r_body = 0.05  # m (radius)
I_body = 0.5 * m_body * r_body**2

print(f"Body moment of inertia: {I_body:.6f} kg·m²")

# Four arms (approximate as point masses)
m_motor = 0.06  # kg each motor+prop
r_arm = 0.15  # m (motor distance from center)
I_arms = 4 * m_motor * r_arm**2

print(f"Arms moment of inertia: {I_arms:.6f} kg·m²")

# Total (for rotation about vertical axis)
I_total = I_body + I_arms

print(f"Total I_z: {I_total:.6f} kg·m²")

# This affects how quickly the quadcopter can change yaw!
```

### 2.3 Torque and Angular Momentum

```
τ = I α (Rotational Newton's second law)
τ = r × F (Torque from force)
L = I ω (Angular momentum)
```

**Quadcopter Roll Moment:**

```python
# When motors on one side spin faster, they create a moment

# Force difference between left and right motors
F_right = 10  # N
F_left = 8   # N
delta_F = F_right - F_left

# Arm length
L_arm = 0.15  # m

# Torque about center
tau_roll = delta_F * L_arm

print(f"Roll torque: {tau_roll:.3f} N·m")

# Angular acceleration (using I from above)
I_roll = I_total  # Simplified
alpha = tau_roll / I_roll

print(f"Angular acceleration: {alpha:.2f} rad/s²")
print(f"Angular acceleration: {np.degrees(alpha):.1f} °/s²")

# Time to rotate 45 degrees from rest
theta_target = np.radians(45)
# θ = ½αt²
t_rotate = np.sqrt(2 * theta_target / alpha)

print(f"Time to rotate 45°: {t_rotate:.3f} s")
```

### 2.4 Gyroscopic Effect

A spinning rotor resists changes in its orientation:

```
L = I ω (angular momentum)
τ = dL/dt
```

**Precession:**

```python
# When you try to tilt a spinning propeller, it creates torque
# in a perpendicular direction!

# Propeller parameters
I_prop = 0.0001  # kg·m² (moment of inertia)
omega_spin = 628  # rad/s (6000 RPM)
L_prop = I_prop * omega_spin

print(f"Propeller angular momentum: {L_prop:.4f} kg·m²/s")

# If quadcopter pitches forward at ω_pitch
omega_pitch = 1  # rad/s

# Gyroscopic torque
tau_gyro = L_prop * omega_pitch

print(f"Gyroscopic torque per prop: {tau_gyro:.4f} N·m")
print(f"Four props total: {4*tau_gyro:.4f} N·m")
```

This is why quadcopter control is challenging - gyroscopic effects couple pitch/roll/yaw!

### 2.5 Euler Angles and Rotation Matrices

**Already covered in Mathematics**, but physically:

- **Roll (φ)**: Rotation about X-axis (forward)
- **Pitch (θ)**: Rotation about Y-axis (right)
- **Yaw (ψ)**: Rotation about Z-axis (up)

```python
# Example: Calculate thrust components in earth frame

def thrust_components(total_thrust, roll, pitch, yaw):
    """
    Given thrust in body frame, calculate components in earth frame
    """
    # Rotation matrix (ZYX Euler)
    cr, sr = np.cos(roll), np.sin(roll)
    cp, sp = np.cos(pitch), np.sin(pitch)
    cy, sy = np.cos(yaw), np.sin(yaw)

    # Full rotation matrix
    R = np.array([
        [cy*cp, cy*sp*sr - sy*cr, cy*sp*cr + sy*sr],
        [sy*cp, sy*sp*sr + cy*cr, sy*sp*cr - cy*sr],
        [-sp,   cp*sr,            cp*cr          ]
    ])

    # Thrust vector in body frame (pointing up in body)
    T_body = np.array([0, 0, total_thrust])

    # Transform to earth frame
    T_earth = R @ T_body

    return T_earth

# Example: 15N thrust, tilted 10° roll, 20° pitch
T_earth = thrust_components(15, np.radians(10), np.radians(20), 0)

print(f"Thrust components in earth frame:")
print(f"  X (forward): {T_earth[0]:.2f} N")
print(f"  Y (right):   {T_earth[1]:.2f} N")
print(f"  Z (up):      {T_earth[2]:.2f} N")

# This is how the quadcopter moves horizontally!
horizontal_thrust = np.sqrt(T_earth[0]**2 + T_earth[1]**2)
vertical_thrust = T_earth[2]

print(f"Horizontal thrust: {horizontal_thrust:.2f} N")
print(f"Vertical thrust: {vertical_thrust:.2f} N")

# Horizontal acceleration
m = 1.5  # kg
a_horizontal = horizontal_thrust / m
print(f"Horizontal acceleration: {a_horizontal:.2f} m/s²")
```

---

## 3. Fluid Dynamics & Aerodynamics

### 3.1 Properties of Air

```python
# Standard atmosphere at sea level
rho = 1.225   # kg/m³ (air density)
P = 101325    # Pa (pressure)
T = 288.15    # K (temperature = 15°C)
mu = 1.81e-5  # Pa·s (dynamic viscosity)

print(f"Air density: {rho:.3f} kg/m³")
print(f"Air pressure: {P/1000:.1f} kPa")
print(f"Temperature: {T-273.15:.1f} °C")

# Density varies with altitude!
def air_density_altitude(h):
    """
    Air density vs altitude (simplified)
    h in meters
    """
    return rho * np.exp(-h / 8500)  # 8500m scale height

altitudes = np.linspace(0, 5000, 100)
densities = air_density_altitude(altitudes)

plt.figure(figsize=(10, 6))
plt.plot(altitudes, densities)
plt.xlabel('Altitude (m)')
plt.ylabel('Air Density (kg/m³)')
plt.title('Air Density vs Altitude')
plt.grid(True)
plt.savefig('air_density_altitude.png')

# At 1000m altitude
rho_1000m = air_density_altitude(1000)
print(f"\nAt 1000m: ρ = {rho_1000m:.3f} kg/m³")
print(f"Density ratio: {rho_1000m/rho:.3f}")
print(f"Thrust reduction: {(1-rho_1000m/rho)*100:.1f}%")
```

**Your quadcopter will perform worse at altitude!**

### 3.2 Bernoulli's Equation

For steady, incompressible flow:

```
P + ½ρv² + ρgh = constant

P₁ + ½ρv₁² = P₂ + ½ρv₂²  (same height)
```

**Application: Pressure on a moving quadcopter**

```python
# Stagnation pressure on nose of quadcopter
v_forward = 15  # m/s
P_static = 101325  # Pa

P_dynamic = 0.5 * rho * v_forward**2
P_stagnation = P_static + P_dynamic

print(f"Forward velocity: {v_forward} m/s")
print(f"Dynamic pressure: {P_dynamic:.1f} Pa")
print(f"Stagnation pressure: {P_stagnation:.1f} Pa")
print(f"Pressure increase: {P_dynamic/1000:.2f} kPa")
```

### 3.3 Drag Force

```
F_drag = ½ρv²C_d A
```

Where:
- C_d = drag coefficient (shape dependent)
- A = frontal area

```python
# Quadcopter drag
C_d = 1.0  # Approximate (complex shape)
A = 0.04   # m² (frontal area, rough estimate)

velocities = np.linspace(0, 25, 100)
drag_forces = 0.5 * rho * velocities**2 * C_d * A

plt.figure(figsize=(10, 6))
plt.plot(velocities, drag_forces)
plt.xlabel('Velocity (m/s)')
plt.ylabel('Drag Force (N)')
plt.title('Quadcopter Drag vs Velocity')
plt.grid(True)
plt.savefig('drag_force.png')

# Power required to overcome drag
power_drag = drag_forces * velocities

plt.figure(figsize=(10, 6))
plt.plot(velocities, power_drag)
plt.xlabel('Velocity (m/s)')
plt.ylabel('Power (W)')
plt.title('Power Required to Overcome Drag')
plt.grid(True)
plt.savefig('drag_power.png')

# At 10 m/s
v_cruise = 10
F_drag_cruise = 0.5 * rho * v_cruise**2 * C_d * A
P_drag_cruise = F_drag_cruise * v_cruise

print(f"\nAt {v_cruise} m/s:")
print(f"Drag force: {F_drag_cruise:.2f} N")
print(f"Power for drag: {P_drag_cruise:.1f} W")
```

### 3.4 Propeller Aerodynamics

#### Momentum Theory

Simple model: propeller is a disk that accelerates air

```python
# Momentum theory for hover
T = thrust  # N
v_hover = np.sqrt(T / (2 * rho * A_prop))

# For our 1.5kg quadcopter
T_total = 1.5 * 9.81
T_per_prop = T_total / 4

A_prop = np.pi * (0.254/2)**2  # 10-inch prop area

v_hover = np.sqrt(T_per_prop / (2 * rho * A_prop))

print(f"Thrust per propeller: {T_per_prop:.2f} N")
print(f"Propeller area: {A_prop:.4f} m²")
print(f"Induced velocity (hover): {v_hover:.2f} m/s")

# Power required (ideal, momentum theory)
P_ideal = T_per_prop * v_hover
P_total_ideal = 4 * P_ideal

print(f"Ideal power per prop: {P_ideal:.1f} W")
print(f"Total ideal power: {P_total_ideal:.1f} W")

# Real power (with efficiency ~60%)
efficiency = 0.6
P_real = P_total_ideal / efficiency

print(f"Estimated real power: {P_real:.1f} W")
```

#### Blade Element Theory (More Complex)

Each blade element sees a different velocity and angle of attack.

```python
# Simplified blade element analysis
# Propeller: 10 inch, 2-blade, 4.5 inch pitch

diameter = 0.254  # m
pitch = 0.1143    # m (4.5 inches)
RPM = 6000

# At different radial positions
r_positions = np.linspace(0.02, diameter/2, 20)

omega = RPM * 2*np.pi / 60

# Blade velocity at each position
v_blade = omega * r_positions

# Advance ratio components
v_axial = v_hover  # Upward flow through prop
v_tangential = v_blade

# Resultant velocity and angle
v_resultant = np.sqrt(v_axial**2 + v_tangential**2)
phi = np.arctan2(v_axial, v_tangential)  # Inflow angle

# Geometric pitch angle
theta_geom = np.arctan(pitch / (2*np.pi*r_positions))

# Angle of attack (simplified)
alpha = theta_geom - phi

plt.figure(figsize=(12, 10))

plt.subplot(3, 1, 1)
plt.plot(r_positions*100, np.degrees(phi), label='Inflow angle φ')
plt.plot(r_positions*100, np.degrees(theta_geom), label='Pitch angle θ')
plt.xlabel('Radius (cm)')
plt.ylabel('Angle (degrees)')
plt.title('Blade Angles vs Radius')
plt.legend()
plt.grid(True)

plt.subplot(3, 1, 2)
plt.plot(r_positions*100, np.degrees(alpha))
plt.xlabel('Radius (cm)')
plt.ylabel('Angle of Attack (degrees)')
plt.title('Angle of Attack Distribution')
plt.grid(True)
plt.axhline(y=0, color='r', linestyle='--')

plt.subplot(3, 1, 3)
plt.plot(r_positions*100, v_resultant)
plt.xlabel('Radius (cm)')
plt.ylabel('Resultant Velocity (m/s)')
plt.title('Velocity Seen by Blade Element')
plt.grid(True)

plt.tight_layout()
plt.savefig('blade_element_analysis.png')
```

### 3.5 Reynolds Number

Characterizes flow regime:

```
Re = ρvL/μ
```

```python
# Reynolds number for propeller blade
v_typical = 50  # m/s (mid-span velocity)
L_chord = 0.02  # m (blade chord length)

Re = rho * v_typical * L_chord / mu

print(f"Reynolds number: {Re:.0f}")

if Re < 2300:
    print("Laminar flow")
elif Re < 4000:
    print("Transitional flow")
else:
    print("Turbulent flow")

# Most quadcopter props operate in turbulent regime
```

---

## 4. Electromagnetism & Wave Theory

### 4.1 Electric Fields and Forces

```
F = qE (Force on charge)
E = F/q (Electric field)
```

Coulomb's Law:
```
F = k(q₁q₂/r²)
k = 8.99 × 10⁹ N·m²/C²
```

### 4.2 Magnetic Fields

```
F = qv × B (Lorentz force)
```

**This is how motors work!**

```python
# Simplified motor force calculation
# Current-carrying wire in magnetic field

I = 10  # A (current through motor winding)
L_wire = 0.05  # m (effective length of conductor)
B = 0.5  # T (magnetic field strength)

F_motor = I * L_wire * B

print(f"Force on conductor: {F_motor:.3f} N")

# Torque (with moment arm)
r_motor = 0.01  # m (moment arm)
tau_motor = F_motor * r_motor

print(f"Torque: {tau_motor:.4f} N·m")
```

### 4.3 Electromagnetic Waves

Maxwell's equations lead to wave equation:

```
∇²E = μ₀ε₀ ∂²E/∂t²
```

**Wave speed:**
```
c = 1/√(μ₀ε₀) = 3 × 10⁸ m/s
```

**Wave properties:**
```
λ = c/f (wavelength)
f = frequency
```

```python
# Radar operating frequencies
c = 3e8  # m/s

frequencies = {
    '24 GHz (K-band)': 24e9,
    '77 GHz (W-band)': 77e9,
    '10 GHz (X-band)': 10e9,
    '5.8 GHz (C-band)': 5.8e9,
}

print("Radar Frequency Analysis:")
print("-" * 50)

for band, freq in frequencies.items():
    wavelength = c / freq
    print(f"{band}:")
    print(f"  Frequency: {freq/1e9:.1f} GHz")
    print(f"  Wavelength: {wavelength*1000:.2f} mm")
    print(f"  Photon energy: {6.626e-34 * freq / 1.602e-19:.2e} eV")
    print()
```

### 4.4 Electromagnetic Spectrum

```python
import matplotlib.pyplot as plt
import numpy as np

fig, ax = plt.subplots(figsize=(14, 6))

# Frequency ranges (Hz)
bands = {
    'Radio': (3e3, 3e9),
    'Microwave': (3e9, 3e11),
    'Infrared': (3e11, 4.3e14),
    'Visible': (4.3e14, 7.5e14),
    'UV': (7.5e14, 3e17),
    'X-Ray': (3e17, 3e19),
    'Gamma': (3e19, 3e24)
}

colors = ['red', 'orange', 'yellow', 'green', 'blue', 'indigo', 'violet']

for i, (band, (f_min, f_max)) in enumerate(bands.items()):
    ax.barh(i, f_max - f_min, left=f_min, height=0.8,
            color=colors[i], alpha=0.7, label=band)
    # Add wavelength info
    lambda_min = c / f_max
    lambda_max = c / f_min
    ax.text(np.sqrt(f_min*f_max), i,
            f'{band}\n{lambda_min*1e9:.1e} - {lambda_max*1e9:.1e} nm',
            ha='center', va='center', fontsize=8)

# Mark our radar frequencies
radar_freqs = [5.8e9, 10e9, 24e9, 77e9]
for f in radar_freqs:
    ax.axvline(x=f, color='black', linestyle='--', alpha=0.5, linewidth=1)
    ax.text(f, 7, f'{f/1e9:.0f}GHz', rotation=90, fontsize=8)

ax.set_xscale('log')
ax.set_xlabel('Frequency (Hz)')
ax.set_xlim(1e3, 1e24)
ax.set_yticks(range(len(bands)))
ax.set_yticklabels([])
ax.set_title('Electromagnetic Spectrum (Radar frequencies marked)')
ax.legend(loc='upper right', fontsize=8)

plt.tight_layout()
plt.savefig('em_spectrum.png')
```

### 4.5 Wave Propagation

**Free space path loss:**

```
FSPL (dB) = 20·log₁₀(d) + 20·log₁₀(f) + 20·log₁₀(4π/c)
```

```python
# Calculate path loss for radar
def free_space_path_loss_db(distance_m, frequency_hz):
    """Calculate free space path loss in dB"""
    fspl = 20*np.log10(distance_m) + 20*np.log10(frequency_hz) + 20*np.log10(4*np.pi/c)
    return fspl

distances = np.linspace(1, 100, 100)

frequencies_radar = [5.8e9, 10e9, 24e9, 77e9]
labels = ['5.8 GHz', '10 GHz', '24 GHz', '77 GHz']

plt.figure(figsize=(10, 6))

for freq, label in zip(frequencies_radar, labels):
    losses = free_space_path_loss_db(distances, freq)
    plt.plot(distances, losses, label=label)

plt.xlabel('Distance (m)')
plt.ylabel('Path Loss (dB)')
plt.title('Free Space Path Loss vs Distance')
plt.legend()
plt.grid(True)
plt.savefig('path_loss.png')

# For 24 GHz at 10m
loss_10m = free_space_path_loss_db(10, 24e9)
print(f"\nPath loss at 10m, 24 GHz: {loss_10m:.1f} dB")

# Note: For radar, signal goes out AND back, so double this!
radar_path_loss = 2 * loss_10m
print(f"Two-way radar path loss: {radar_path_loss:.1f} dB")
```

### 4.6 Doppler Effect

```
f_observed = f_source × (c ± v_observer)/(c ± v_source)

For radar (simplified):
f_doppler = 2v·f_carrier/c
```

```python
# Doppler shift calculation
def doppler_shift(velocity_ms, carrier_freq_hz):
    """
    Calculate Doppler shift for radar
    velocity_ms: target velocity (m/s, positive = approaching)
    """
    return 2 * velocity_ms * carrier_freq_hz / c

# Example: tracking a ball
f_carrier = 24e9  # 24 GHz radar

velocities = np.linspace(0, 50, 100)  # 0 to 50 m/s
doppler_shifts = doppler_shift(velocities, f_carrier)

plt.figure(figsize=(10, 6))
plt.plot(velocities, doppler_shifts/1e3)  # Convert to kHz
plt.xlabel('Target Velocity (m/s)')
plt.ylabel('Doppler Shift (kHz)')
plt.title('Doppler Shift vs Target Velocity (24 GHz Radar)')
plt.grid(True)
plt.savefig('doppler_shift.png')

# For a tennis ball at 30 m/s
v_ball = 30
f_d = doppler_shift(v_ball, f_carrier)
print(f"\nTennis ball at {v_ball} m/s:")
print(f"Doppler shift: {f_d/1e3:.2f} kHz")
print(f"Doppler shift: {f_d:.0f} Hz")

# Velocity resolution (depends on observation time)
T_observe = 0.1  # seconds
delta_f = 1 / T_observe  # Frequency resolution
delta_v = delta_f * c / (2 * f_carrier)

print(f"\nWith {T_observe*1000:.0f}ms observation:")
print(f"Frequency resolution: {delta_f:.1f} Hz")
print(f"Velocity resolution: {delta_v:.3f} m/s")
```

---

## 5. Thermodynamics (Battery Chemistry)

### 5.1 Energy and Power

```
Energy = Power × Time
1 Wh = 3600 J
```

**LiPo Battery Basics:**

```python
# Typical 4S LiPo battery
cells_series = 4
voltage_per_cell_nominal = 3.7  # V
voltage_per_cell_max = 4.2      # V
voltage_per_cell_min = 3.0      # V (safe cutoff)

V_nominal = cells_series * voltage_per_cell_nominal
V_max = cells_series * voltage_per_cell_max
V_min = cells_series * voltage_per_cell_min

print(f"4S LiPo Battery:")
print(f"Nominal voltage: {V_nominal:.1f} V")
print(f"Full charge: {V_max:.1f} V")
print(f"Cutoff voltage: {V_min:.1f} V")

# Capacity example
capacity_mah = 3000  # mAh
capacity_ah = capacity_mah / 1000

energy_wh = capacity_ah * V_nominal
energy_j = energy_wh * 3600

print(f"\nCapacity: {capacity_mah} mAh = {capacity_ah:.1f} Ah")
print(f"Energy: {energy_wh:.1f} Wh = {energy_j/1000:.1f} kJ")

# Discharge rate (C rating)
c_rating = 25  # 25C battery
max_current = capacity_ah * c_rating
max_power = max_current * V_nominal

print(f"\nC-rating: {c_rating}C")
print(f"Max continuous current: {max_current:.0f} A")
print(f"Max power: {max_power:.0f} W")
```

### 5.2 Flight Time Estimation

```python
# Estimate flight time
capacity_ah = 3.0
average_current = 15  # A

flight_time_hours = capacity_ah / average_current
flight_time_minutes = flight_time_hours * 60

print(f"Estimated flight time: {flight_time_minutes:.1f} minutes")

# More detailed model
hover_power = 150  # W
cruise_power = 200  # W
aggressive_power = 350  # W

flight_times = {
    'Hover': capacity_ah * V_nominal / hover_power * 60,
    'Cruise': capacity_ah * V_nominal / cruise_power * 60,
    'Aggressive': capacity_ah * V_nominal / aggressive_power * 60
}

print("\nFlight time by mode:")
for mode, time_min in flight_times.items():
    print(f"  {mode}: {time_min:.1f} minutes")

# Battery discharge curve
discharge_times = np.linspace(0, flight_times['Hover'], 100)
# Simplified voltage sag model
voltages = V_max - (V_max - V_min) * (discharge_times / flight_times['Hover'])**1.2

plt.figure(figsize=(10, 6))
plt.plot(discharge_times, voltages)
plt.axhline(y=V_min, color='r', linestyle='--', label='Cutoff voltage')
plt.axhline(y=V_nominal, color='g', linestyle='--', label='Nominal voltage')
plt.xlabel('Time (minutes)')
plt.ylabel('Battery Voltage (V)')
plt.title('Battery Voltage During Discharge (Hover)')
plt.legend()
plt.grid(True)
plt.savefig('battery_discharge.png')
```

### 5.3 Heat Generation

```
Power_loss = I²R (resistive heating)
```

```python
# Heat generation in ESC/motor
I_motor = 20  # A
R_total = 0.1  # Ohms (ESC + motor + wiring resistance)

P_loss = I_motor**2 * R_total

print(f"Current: {I_motor} A")
print(f"Resistance: {R_total} Ω")
print(f"Power loss as heat: {P_loss:.1f} W")

# Temperature rise (simplified)
# ΔT = P / (m·c·rate)
# This is why cooling is important!

# Different operating currents
currents = np.linspace(0, 40, 100)
heat_power = currents**2 * R_total

plt.figure(figsize=(10, 6))
plt.plot(currents, heat_power)
plt.xlabel('Current (A)')
plt.ylabel('Heat Generation (W)')
plt.title('Resistive Heating in Motor/ESC System')
plt.grid(True)
plt.savefig('heat_generation.png')

print(f"\nAt {I_motor}A, heat = {P_loss:.1f}W")
print("This requires adequate cooling (airflow, heatsinks)")
```

---

## 6. Measurement & Units

### 6.1 SI Units

**Base Units:**
- Length: meter (m)
- Mass: kilogram (kg)
- Time: second (s)
- Current: ampere (A)
- Temperature: kelvin (K)

**Derived Units:**
- Force: newton (N) = kg·m/s²
- Energy: joule (J) = N·m
- Power: watt (W) = J/s
- Voltage: volt (V) = J/C
- Frequency: hertz (Hz) = 1/s

### 6.2 Measurement Uncertainty

Every sensor has:
- **Accuracy**: How close to true value
- **Precision**: Repeatability
- **Resolution**: Smallest detectable change
- **Noise**: Random variation

```python
# Example: Accelerometer specifications
# (typical MEMS sensor like MPU6050)

accel_range = 2  # ±2g
accel_resolution_bits = 16
accel_sensitivity = accel_range * 2 / (2**accel_resolution_bits)

print("Accelerometer Specifications:")
print(f"Range: ±{accel_range}g")
print(f"Resolution: {accel_resolution_bits} bits")
print(f"Sensitivity: {accel_sensitivity*1000:.4f} mg/LSB")

# Noise density (from datasheet)
noise_density = 400  # μg/√Hz

# At 100 Hz bandwidth
bandwidth = 100  # Hz
noise_rms = noise_density * np.sqrt(bandwidth)

print(f"\nNoise density: {noise_density} μg/√Hz")
print(f"RMS noise at {bandwidth}Hz: {noise_rms:.0f} μg = {noise_rms/1e6:.4f} g")

# What this means for position estimation
dt = 0.01  # 100 Hz sampling
noise_g = noise_rms / 1e6
noise_ms2 = noise_g * 9.81

# Integrate twice to get position error
# Error accumulates as t^2
time = np.linspace(0, 10, 1000)
position_error = 0.5 * noise_ms2 * time**2

plt.figure(figsize=(10, 6))
plt.plot(time, position_error)
plt.xlabel('Time (s)')
plt.ylabel('Position Error (m)')
plt.title('Position Error from Accelerometer Noise (without correction)')
plt.grid(True)
plt.yscale('log')
plt.savefig('sensor_drift.png')

print(f"Position error after 10s: {position_error[-1]:.1f} m")
print("This is why we need sensor fusion!")
```

### 6.3 Dimensional Analysis

Always check that equations are dimensionally consistent!

```python
# Example: Check hover thrust equation
# T = m * g

m_kg = 1.5  # kg
g_ms2 = 9.81  # m/s²

T_N = m_kg * g_ms2

print("Dimensional analysis:")
print(f"m: [{m_kg}] kg")
print(f"g: [{g_ms2}] m/s²")
print(f"T = m × g: [{T_N}] kg·m/s² = N ✓")

# Example: Propeller thrust equation (momentum theory)
# T = 2 * rho * A * v^2

rho_kgm3 = 1.225
A_m2 = 0.02
v_ms = 5

T_calc = 2 * rho_kgm3 * A_m2 * v_ms**2

print(f"\nT = 2·ρ·A·v²")
print(f"ρ: [kg/m³] × A: [m²] × v²: [(m/s)²]")
print(f"= [kg/m³ · m² · m²/s²] = [kg·m/s²] = [N] ✓")
print(f"Thrust: {T_calc:.2f} N")
```

---

## Summary and Applications

### For Quadcopter Project:

You now understand:
✅ Forces and moments that control flight
✅ Rotational dynamics for attitude control
✅ Aerodynamics of propellers
✅ Energy and power for battery sizing
✅ Sensor measurement principles

### For Radar Project:

You now understand:
✅ Electromagnetic wave propagation
✅ Doppler effect for velocity measurement
✅ Path loss and range calculations
✅ Power and energy in RF systems

### Key Equations to Remember:

**Mechanics:**
- F = ma
- τ = Iα
- E = ½mv² + mgh

**Aerodynamics:**
- F_drag = ½ρv²C_dA
- Bernoulli: P + ½ρv² + ρgh = const

**Electromagnetism:**
- λ = c/f
- f_doppler = 2vf/c
- FSPL = 20log₁₀(d) + 20log₁₀(f) + K

**Energy:**
- P = VI
- E = Pt
- P_loss = I²R

---

**Next Steps:**

1. **[Electronics Foundations](./04-electronics-foundations.md)** - Circuits, components, and practical electronics
2. **[Programming Foundations](./03-programming-foundations.md)** - Code the control systems
3. **[Quadcopter Q1: Flight Physics](../quadcopter/Q1-flight-physics.md)** - Apply these concepts to quadcopter design
4. **[Radar R1: EM Wave Theory](../radar/R1-em-wave-theory.md)** - Deep dive into radar principles

---

*Last Updated: November 2025*
