# R2: Radar Fundamentals

> **Understanding how radar detects and tracks objects - from EM waves to target detection**

---

## Table of Contents

1. [What is Radar?](#1-what-is-radar)
2. [The Radar Equation](#2-the-radar-equation)
3. [Pulse vs Continuous Wave Radar](#3-pulse-vs-continuous-wave-radar)
4. [Doppler Radar](#4-doppler-radar)
5. [FMCW Radar](#5-fmcw-radar)
6. [Resolution and Accuracy](#6-resolution-and-accuracy)
7. [Target Detection](#7-target-detection)

---

## 1. What is Radar?

**RADAR = RAdio Detection And Ranging**

### 1.1 Basic Principle

1. **Transmit** electromagnetic wave
2. **Reflect** off target
3. **Receive** echo
4. **Measure** time delay and frequency shift
5. **Calculate** range and velocity

```python
import numpy as np
import matplotlib.pyplot as plt

# Visualize basic radar operation
fig, ax = plt.subplots(figsize=(12, 6))

# Timeline
time_points = {
    'Transmit': 0,
    'Echo from 10m': 0.067,   # 2*10m / 3e8 m/s
    'Echo from 50m': 0.333,   # 2*50m / 3e8 m/s
    'Echo from 100m': 0.667,  # 2*100m / 3e8 m/s
}

# Plot transmitted pulse
t_tx = np.linspace(0, 0.01, 100)
signal_tx = np.sin(2*np.pi*24e9*t_tx)  # 24 GHz carrier (conceptual)

colors = ['red', 'blue', 'green', 'orange']
for (label, t_delay), color in zip(time_points.items(), colors):
    if label == 'Transmit':
        ax.plot(t_tx*1e6, signal_tx, color=color, linewidth=2, label=label)
    else:
        # Echo (attenuated)
        t_echo = t_tx + t_delay*1e-6
        signal_echo = 0.3 * np.sin(2*np.pi*24e9*t_tx)
        ax.plot(t_echo*1e6, signal_echo, color=color, linewidth=1.5, alpha=0.7, label=label)

ax.set_xlabel('Time (μs)')
ax.set_ylabel('Signal Amplitude')
ax.set_title('Radar Pulse and Echoes from Different Distances')
ax.legend()
ax.grid(True, alpha=0.3)
ax.set_xlim([0, 1])
plt.savefig('radar_basic_principle.png')

print("Radar Operating Principle:")
print("-" * 50)
for label, t_delay in time_points.items():
    if label != 'Transmit':
        distance = t_delay * 1e-6 * 3e8 / 2
        print(f"{label}: delay = {t_delay:.3f} μs, distance = {distance:.1f} m")
```

### 1.2 Key Radar Parameters

```python
# Fundamental radar parameters
c = 3e8  # Speed of light (m/s)

# Frequency bands commonly used
radar_bands = {
    'HF': (3e6, 30e6),
    'VHF': (30e6, 300e6),
    'UHF': (300e6, 1e9),
    'L-band': (1e9, 2e9),
    'S-band': (2e9, 4e9),
    'C-band': (4e9, 8e9),
    'X-band': (8e9, 12e9),
    'Ku-band': (12e9, 18e9),
    'K-band': (18e9, 27e9),
    'Ka-band': (27e9, 40e9),
    'V-band': (40e9, 75e9),
    'W-band': (75e9, 110e9),
}

print("\nRadar Frequency Bands:")
print("-" * 70)
print(f"{'Band':<10} {'Frequency Range':<25} {'Wavelength Range':<30}")
print("-" * 70)

for band, (f_min, f_max) in radar_bands.items():
    lambda_max = c / f_min
    lambda_min = c / f_max
    print(f"{band:<10} {f_min/1e9:>6.1f} - {f_max/1e9:<6.1f} GHz    "
          f"{lambda_min*100:>7.2f} - {lambda_max*100:<7.2f} cm")

# For our DIY project
f_diy = 24e9  # 24 GHz (K-band)
lambda_diy = c / f_diy

print(f"\n** Our DIY Radar: {f_diy/1e9:.0f} GHz (K-band), λ = {lambda_diy*1000:.2f} mm **")
```

---

## 2. The Radar Equation

### 2.1 Basic Radar Equation

The fundamental equation relating all radar parameters:

```
P_r = (P_t × G_t × G_r × λ² × σ) / ((4π)³ × R⁴ × L)

Where:
P_r = received power (W)
P_t = transmitted power (W)
G_t = transmit antenna gain
G_r = receive antenna gain
λ = wavelength (m)
σ = radar cross section (m²)
R = range to target (m)
L = system losses
```

**Key insight:** Received power ∝ 1/R⁴ (goes out and comes back!)

```python
def radar_equation(P_t, G_t, G_r, wavelength, sigma, R, L=1):
    """
    Calculate received power using radar equation
    All SI units
    """
    P_r = (P_t * G_t * G_r * wavelength**2 * sigma) / ((4*np.pi)**3 * R**4 * L)
    return P_r

# Example parameters for DIY 24 GHz radar
P_t = 0.01  # 10 mW (10 dBm) - typical for low-power modules
G_t = 10    # 10 dBi antenna gain
G_r = 10    # Same antenna for receive
f = 24e9
wavelength = c / f
sigma = 0.01  # Small ball, RCS ~ 0.01 m²
L = 2       # 3 dB system losses

# Calculate vs distance
distances = np.linspace(1, 50, 100)
received_powers = [radar_equation(P_t, G_t, G_r, wavelength, sigma, R, L) for R in distances]
received_powers_dBm = [10*np.log10(P*1000) for P in received_powers]

plt.figure(figsize=(10, 6))
plt.plot(distances, received_powers_dBm)
plt.xlabel('Range (m)')
plt.ylabel('Received Power (dBm)')
plt.title('Received Power vs Range (24 GHz Radar)')
plt.grid(True)
plt.axhline(y=-90, color='r', linestyle='--', label='Typical noise floor')
plt.legend()
plt.savefig('radar_equation_range.png')

# Print powers at specific ranges
test_ranges = [5, 10, 20, 30]
print("\nReceived Power at Different Ranges:")
print("-" * 50)
for R in test_ranges:
    P_r = radar_equation(P_t, G_t, G_r, wavelength, sigma, R, L)
    P_r_dBm = 10*np.log10(P_r*1000)
    print(f"Range: {R:>3} m  →  P_r = {P_r_dBm:>7.1f} dBm  ({P_r*1e12:.2f} pW)")
```

### 2.2 Maximum Detectable Range

```
R_max = [(P_t × G_t × G_r × λ² × σ) / ((4π)³ × P_min × L)]^(1/4)

Where P_min is the minimum detectable signal power.
```

```python
def max_range(P_t, G_t, G_r, wavelength, sigma, P_min, L=1):
    """Calculate maximum detection range"""
    numerator = P_t * G_t * G_r * wavelength**2 * sigma
    denominator = (4*np.pi)**3 * P_min * L
    R_max = (numerator / denominator) ** 0.25
    return R_max

# System parameters
P_t = 0.01  # 10 mW
G_t = G_r = 10
wavelength = c / 24e9
L = 2

# Minimum detectable signal (limited by noise)
# Noise floor ≈ -174 dBm/Hz + 10*log10(bandwidth) + noise figure
bandwidth = 100e3  # 100 kHz
noise_figure_dB = 10  # 10 dB noise figure
SNR_required_dB = 10  # Need 10 dB SNR for reliable detection

noise_floor_dBm = -174 + 10*np.log10(bandwidth) + noise_figure_dB
P_min_dBm = noise_floor_dBm + SNR_required_dB
P_min = 10**((P_min_dBm - 30)/10)  # Convert dBm to W

print(f"Noise floor: {noise_floor_dBm:.1f} dBm")
print(f"Minimum detectable signal: {P_min_dBm:.1f} dBm")

# Max range for different target sizes
target_sizes = {
    'Small ball (5cm)': 0.001,     # ~0.001 m² RCS
    'Tennis ball': 0.003,           # ~0.003 m²
    'Basketball': 0.02,             # ~0.02 m²
    'Person': 1.0,                  # ~1 m²
    'Car': 10,                      # ~10 m²
}

print("\nMaximum Detection Range:")
print("-" * 50)
for target, sigma in target_sizes.items():
    R = max_range(P_t, G_t, G_r, wavelength, sigma, P_min, L)
    print(f"{target:<20}: {R:>6.1f} m")
```

### 2.3 Radar Cross Section (RCS)

The RCS (σ) determines how much energy a target reflects back.

**Simple shapes (approximate):**
```
Sphere (radius r): σ = π × r²  (for r >> λ)
Flat plate (area A): σ = (4π × A²) / λ²  (when perpendicular)
Corner reflector: σ = (12π × L⁴) / λ²
```

```python
# Calculate RCS for different objects
def rcs_sphere(radius, wavelength):
    """RCS of sphere (Rayleigh or optical region)"""
    if radius < wavelength / 10:  # Rayleigh region
        return np.pi**5 * radius**6 / wavelength**4
    else:  # Optical region
        return np.pi * radius**2

def rcs_flat_plate(width, height, wavelength):
    """RCS of flat plate perpendicular to radar"""
    A = width * height
    return (4 * np.pi * A**2) / wavelength**2

wavelength = c / 24e9

objects = {
    'Small ball (r=2.5cm)': rcs_sphere(0.025, wavelength),
    'Tennis ball (r=3.3cm)': rcs_sphere(0.033, wavelength),
    'Basketball (r=12cm)': rcs_sphere(0.12, wavelength),
    'Ping pong ball (r=2cm)': rcs_sphere(0.02, wavelength),
    '10cm square plate': rcs_flat_plate(0.1, 0.1, wavelength),
}

print("\nRadar Cross Sections at 24 GHz:")
print("-" * 60)
print(f"{'Object':<30} {'RCS (m²)':<15} {'RCS (dBsm)':<15}")
print("-" * 60)

for obj, sigma in objects.items():
    sigma_dBsm = 10 * np.log10(sigma)
    print(f"{obj:<30} {sigma:>10.6f}     {sigma_dBsm:>10.1f}")

# Visualize RCS vs size
radii = np.linspace(0.01, 0.2, 100)
rcs_values = [rcs_sphere(r, wavelength) for r in radii]

plt.figure(figsize=(10, 6))
plt.loglog(radii*100, rcs_values)
plt.xlabel('Sphere Radius (cm)')
plt.ylabel('RCS (m²)')
plt.title('Radar Cross Section vs Sphere Size (24 GHz)')
plt.grid(True, which='both')
plt.axvline(x=3.3, color='r', linestyle='--', alpha=0.5, label='Tennis ball')
plt.axvline(x=12, color='g', linestyle='--', alpha=0.5, label='Basketball')
plt.legend()
plt.savefig('rcs_vs_size.png')
```

---

## 3. Pulse vs Continuous Wave Radar

### 3.1 Pulsed Radar

**Operation:**
1. Transmit short pulse
2. Wait for echo
3. Repeat

**Range determination:**
```
R = (c × Δt) / 2

Where Δt is round-trip time
```

```python
# Pulsed radar simulation
def pulsed_radar_simulation():
    """Simulate pulsed radar operation"""
    # Parameters
    pulse_width = 0.1e-6  # 0.1 μs pulse
    PRF = 10e3  # 10 kHz pulse repetition frequency
    PRI = 1/PRF  # Pulse repetition interval

    # Target at 50m
    R_target = 50  # m
    t_delay = 2 * R_target / c

    # Time axis
    t = np.linspace(0, 3*PRI, 10000)

    # Transmitted pulses
    tx_signal = np.zeros_like(t)
    for i in range(3):
        pulse_start = i * PRI
        pulse_end = pulse_start + pulse_width
        tx_signal[(t >= pulse_start) & (t < pulse_end)] = 1

    # Received echo
    rx_signal = np.zeros_like(t)
    for i in range(3):
        echo_start = i * PRI + t_delay
        echo_end = echo_start + pulse_width
        rx_signal[(t >= echo_start) & (t < echo_end)] = 0.3  # Attenuated

    # Plot
    fig, axes = plt.subplots(2, 1, figsize=(12, 8))

    axes[0].plot(t*1e6, tx_signal, 'b-', linewidth=2)
    axes[0].set_ylabel('Transmitted')
    axes[0].set_title(f'Pulsed Radar (PRF={PRF/1e3:.0f} kHz, Pulse Width={pulse_width*1e6:.1f} μs)')
    axes[0].set_ylim([-0.1, 1.2])
    axes[0].grid(True)

    axes[1].plot(t*1e6, rx_signal, 'r-', linewidth=2)
    axes[1].set_xlabel('Time (μs)')
    axes[1].set_ylabel('Received')
    axes[1].set_ylim([-0.1, 0.5])
    axes[1].grid(True)

    # Mark delay
    axes[1].annotate(f'Delay = {t_delay*1e6:.2f} μs\nRange = {R_target} m',
                     xy=(t_delay*1e6, 0.3), xytext=(t_delay*1e6+20, 0.4),
                     arrowprops=dict(arrowstyle='->', color='red'))

    plt.tight_layout()
    plt.savefig('pulsed_radar_waveform.png')

    # Range resolution
    range_resolution = (c * pulse_width) / 2
    print(f"\nPulsed Radar Parameters:")
    print(f"Pulse width: {pulse_width*1e6:.2f} μs")
    print(f"PRF: {PRF/1e3:.1f} kHz")
    print(f"Maximum unambiguous range: {c/(2*PRF):.1f} m")
    print(f"Range resolution: {range_resolution:.2f} m")

pulsed_radar_simulation()
```

### 3.2 Continuous Wave (CW) Radar

**Operation:**
- Transmits continuously
- Cannot measure range directly (no time reference)
- Can measure velocity via Doppler shift

**Advantage:** Simpler, cheaper
**Disadvantage:** No range information

```python
# CW Doppler radar simulation
def cw_radar_simulation():
    """Simulate CW Doppler radar"""
    f_carrier = 24e9  # Hz
    v_target = 10  # m/s (approaching)

    # Doppler shift
    f_doppler = 2 * v_target * f_carrier / c

    # Time axis (we'll simulate at baseband)
    t = np.linspace(0, 0.01, 10000)  # 10 ms

    # Transmitted signal (constant frequency)
    tx_signal = np.cos(2*np.pi * 1000 * t)  # 1 kHz reference

    # Received signal (Doppler shifted)
    rx_signal = 0.3 * np.cos(2*np.pi * (1000 + f_doppler) * t)

    # Beat frequency (after mixing)
    beat_signal = tx_signal * rx_signal

    # Plot
    fig, axes = plt.subplots(3, 1, figsize=(12, 10))

    axes[0].plot(t*1000, tx_signal, 'b-')
    axes[0].set_ylabel('TX Signal')
    axes[0].set_title('CW Radar - Doppler Detection')
    axes[0].set_xlim([0, 3])
    axes[0].grid(True)

    axes[1].plot(t*1000, rx_signal, 'r-')
    axes[1].set_ylabel('RX Signal')
    axes[1].set_xlim([0, 3])
    axes[1].grid(True)

    axes[2].plot(t*1000, beat_signal, 'g-')
    axes[2].set_xlabel('Time (ms)')
    axes[2].set_ylabel('Beat Signal')
    axes[2].set_xlim([0, 3])
    axes[2].grid(True)

    plt.tight_layout()
    plt.savefig('cw_radar_waveform.png')

    print(f"\nCW Doppler Radar:")
    print(f"Carrier frequency: {f_carrier/1e9:.1f} GHz")
    print(f"Target velocity: {v_target} m/s")
    print(f"Doppler shift: {f_doppler:.1f} Hz")
    print(f"Can detect velocity: YES")
    print(f"Can detect range: NO (need FMCW)")

cw_radar_simulation()
```

---

## 4. Doppler Radar

### 4.1 Doppler Effect

**Frequency shift for moving target:**

```
f_doppler = (2 × v × f_carrier) / c

Where:
v = relative velocity (positive if approaching)
f_carrier = transmitted frequency
c = speed of light
```

**Factor of 2 because:** Signal travels to target AND back

```python
# Doppler shift calculator
def doppler_shift(velocity, f_carrier):
    """Calculate Doppler shift"""
    return 2 * velocity * f_carrier / c

def velocity_from_doppler(f_doppler, f_carrier):
    """Calculate velocity from Doppler shift"""
    return (f_doppler * c) / (2 * f_carrier)

# Example calculations
f_carrier = 24e9
velocities = np.linspace(-50, 50, 100)  # -50 to +50 m/s
doppler_shifts = doppler_shift(velocities, f_carrier)

plt.figure(figsize=(10, 6))
plt.plot(velocities, doppler_shifts/1e3)
plt.xlabel('Velocity (m/s)')
plt.ylabel('Doppler Shift (kHz)')
plt.title('Doppler Shift vs Velocity (24 GHz Radar)')
plt.grid(True)
plt.axhline(y=0, color='k', linestyle='-', linewidth=0.5)
plt.axvline(x=0, color='k', linestyle='-', linewidth=0.5)
plt.savefig('doppler_vs_velocity.png')

# Specific examples
examples = {
    'Walking (1.5 m/s)': 1.5,
    'Running (5 m/s)': 5,
    'Cycling (8 m/s)': 8,
    'Tennis ball (30 m/s)': 30,
    'Baseball (45 m/s)': 45,
    'Car (30 m/s = 108 km/h)': 30,
}

print("\nDoppler Shifts for Common Velocities:")
print("-" * 60)
print(f"{'Target':<30} {'Velocity':<15} {'Doppler Shift':<15}")
print("-" * 60)

for target, v in examples.items():
    f_d = doppler_shift(v, f_carrier)
    print(f"{target:<30} {v:>8.1f} m/s     {f_d:>10.1f} Hz")
```

### 4.2 Velocity Resolution

```
Δv = (c × Δf) / (2 × f_carrier)

Where Δf is frequency resolution = 1/T_observation
```

```python
# Velocity resolution vs observation time
f_carrier = 24e9

observation_times = np.logspace(-3, 1, 100)  # 1 ms to 10 s
freq_resolutions = 1 / observation_times
velocity_resolutions = (c * freq_resolutions) / (2 * f_carrier)

plt.figure(figsize=(10, 6))
plt.loglog(observation_times*1000, velocity_resolutions)
plt.xlabel('Observation Time (ms)')
plt.ylabel('Velocity Resolution (m/s)')
plt.title('Velocity Resolution vs Observation Time (24 GHz)')
plt.grid(True, which='both')

# Mark practical points
practical_times = [0.01, 0.1, 1.0]  # 10ms, 100ms, 1s
for T in practical_times:
    Δv = c / (2 * f_carrier * T)
    plt.plot(T*1000, Δv, 'ro', markersize=8)
    plt.annotate(f'{T*1000:.0f}ms: {Δv:.3f} m/s',
                 xy=(T*1000, Δv), xytext=(T*1000*2, Δv*2),
                 arrowprops=dict(arrowstyle='->', color='red'))

plt.savefig('velocity_resolution.png')

print("\nVelocity Resolution:")
print("-" * 50)
for T in [0.001, 0.01, 0.1, 1.0]:
    Δv = c / (2 * f_carrier * T)
    print(f"Observation time: {T*1000:>8.1f} ms  →  Δv = {Δv:>7.4f} m/s")
```

---

## 5. FMCW Radar

**Frequency Modulated Continuous Wave** - The best choice for DIY!

### 5.1 FMCW Principle

**Operation:**
1. Sweep frequency linearly (chirp)
2. Mix transmitted and received signals
3. Measure beat frequency
4. Calculate range from beat frequency

```
f_beat = (2 × R × B) / (c × T_chirp)

Where:
R = range
B = bandwidth (frequency sweep)
T_chirp = chirp duration
```

```python
# FMCW radar simulation
def fmcw_simulation():
    """Simulate FMCW radar operation"""
    # Parameters
    f_start = 24.0e9  # 24.0 GHz
    f_stop = 24.2e9   # 24.2 GHz
    B = f_stop - f_start  # 200 MHz bandwidth
    T_chirp = 1e-3  # 1 ms chirp duration
    chirp_rate = B / T_chirp

    # Target parameters
    R_target = 10  # m
    v_target = 5   # m/s

    # Time delay
    tau = 2 * R_target / c

    # Beat frequency (range)
    f_beat_range = chirp_rate * tau

    # Doppler shift
    f_doppler = 2 * v_target * f_start / c

    print("FMCW Radar Simulation:")
    print("-" * 50)
    print(f"Bandwidth: {B/1e6:.0f} MHz")
    print(f"Chirp duration: {T_chirp*1000:.1f} ms")
    print(f"Chirp rate: {chirp_rate/1e12:.1f} THz/s")
    print(f"\nTarget:")
    print(f"Range: {R_target} m")
    print(f"Velocity: {v_target} m/s")
    print(f"\nMeasured:")
    print(f"Beat frequency (range): {f_beat_range/1e3:.2f} kHz")
    print(f"Doppler shift: {f_doppler:.1f} Hz")
    print(f"Total beat frequency: {(f_beat_range + f_doppler)/1e3:.2f} kHz")

    # Time axis
    t = np.linspace(0, T_chirp, 10000)

    # Transmitted frequency
    f_tx = f_start + chirp_rate * t

    # Received frequency (delayed and Doppler shifted)
    t_rx = t - tau
    t_rx[t_rx < 0] = 0
    f_rx = f_start + chirp_rate * t_rx + f_doppler

    # Plot frequency vs time
    plt.figure(figsize=(12, 8))

    plt.subplot(2, 1, 1)
    plt.plot(t*1000, (f_tx-f_start)/1e6, 'b-', label='TX', linewidth=2)
    plt.plot(t*1000, (f_rx-f_start)/1e6, 'r--', label='RX (delayed)', linewidth=2)
    plt.xlabel('Time (ms)')
    plt.ylabel('Frequency - 24 GHz (MHz)')
    plt.title('FMCW Chirp: Transmitted and Received Signals')
    plt.legend()
    plt.grid(True)

    # Beat frequency (difference)
    f_beat = f_tx - f_rx

    plt.subplot(2, 1, 2)
    plt.plot(t*1000, f_beat/1e3, 'g-', linewidth=2)
    plt.xlabel('Time (ms)')
    plt.ylabel('Beat Frequency (kHz)')
    plt.title(f'Beat Frequency (constant) = {f_beat_range/1e3:.2f} kHz')
    plt.grid(True)
    plt.ylim([0, f_beat_range/1e3 * 1.5])

    plt.tight_layout()
    plt.savefig('fmcw_chirp.png')

fmcw_simulation()
```

### 5.2 Range Resolution

```
ΔR = c / (2 × B)

Where B is the bandwidth
```

**Larger bandwidth = Better range resolution!**

```python
# Range resolution vs bandwidth
bandwidths = np.logspace(6, 9, 100)  # 1 MHz to 1 GHz
range_resolutions = c / (2 * bandwidths)

plt.figure(figsize=(10, 6))
plt.loglog(bandwidths/1e6, range_resolutions)
plt.xlabel('Bandwidth (MHz)')
plt.ylabel('Range Resolution (m)')
plt.title('Range Resolution vs Bandwidth')
plt.grid(True, which='both')

# Mark common bandwidths
common_bw = [50e6, 200e6, 500e6, 1e9]
for bw in common_bw:
    res = c / (2 * bw)
    plt.plot(bw/1e6, res, 'ro', markersize=8)
    plt.annotate(f'{bw/1e6:.0f} MHz\n{res:.2f} m',
                 xy=(bw/1e6, res), xytext=(bw/1e6*0.5, res*2),
                 arrowprops=dict(arrowstyle='->', color='red'),
                 fontsize=8)

plt.savefig('range_resolution_vs_bandwidth.png')

print("\nRange Resolution:")
print("-" * 50)
for bw in [10e6, 50e6, 100e6, 200e6, 500e6, 1e9]:
    res = c / (2 * bw)
    print(f"Bandwidth: {bw/1e6:>6.0f} MHz  →  ΔR = {res:>6.3f} m")
```

### 5.3 Maximum Range

```
R_max = (c × T_chirp × f_s) / (4 × B)

Where f_s is ADC sampling rate
```

```python
# Maximum unambiguous range
def fmcw_max_range(B, T_chirp, f_sample):
    """Calculate FMCW maximum unambiguous range"""
    # Max beat frequency = f_sample / 2 (Nyquist)
    f_beat_max = f_sample / 2
    R_max = (c * T_chirp * f_sample) / (4 * B)
    return R_max

# Example system
B = 200e6  # 200 MHz bandwidth
T_chirp = 1e-3  # 1 ms
f_sample = 1e6  # 1 MHz ADC

R_max = fmcw_max_range(B, T_chirp, f_sample)

print(f"\nFMCW System Parameters:")
print(f"Bandwidth: {B/1e6:.0f} MHz")
print(f"Chirp time: {T_chirp*1000:.1f} ms")
print(f"Sample rate: {f_sample/1e6:.1f} MHz")
print(f"Maximum range: {R_max:.1f} m")
print(f"Range resolution: {c/(2*B):.2f} m")
```

---

## 6. Resolution and Accuracy

### 6.1 Range Resolution vs Range Accuracy

**Resolution:** Ability to distinguish two close targets
**Accuracy:** How close measurement is to true value

```python
# Demonstrate resolution vs accuracy
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Resolution demonstration
ax1 = axes[0]
targets_close = [10.0, 10.5]  # 0.5m apart
targets_far = [10.0, 12.0]    # 2m apart

# With poor resolution (1.5m)
resolution_poor = 1.5
ax1.axvspan(targets_close[0]-resolution_poor/2, targets_close[0]+resolution_poor/2,
            alpha=0.3, color='red', label=f'Resolution bin ({resolution_poor}m)')
ax1.plot(targets_close, [1, 1], 'ro', markersize=10, label='Two targets (0.5m apart)')
ax1.text(targets_close[0], 1.2, 'Cannot\nresolve', ha='center', fontsize=10, color='red')

# With good resolution (0.3m)
resolution_good = 0.3
ax1.axvspan(targets_far[0]-resolution_good/2, targets_far[0]+resolution_good/2,
            alpha=0.3, color='green')
ax1.axvspan(targets_far[1]-resolution_good/2, targets_far[1]+resolution_good/2,
            alpha=0.3, color='green')
ax1.plot(targets_far, [0.5, 0.5], 'go', markersize=10, label='Two targets (2m apart)')
ax1.text(targets_far[0], 0.7, 'Resolved!', ha='center', fontsize=10, color='green')

ax1.set_xlim([8, 14])
ax1.set_ylim([0, 1.5])
ax1.set_xlabel('Range (m)')
ax1.set_title('Range Resolution')
ax1.legend()
ax1.grid(True, alpha=0.3)

# Accuracy demonstration
ax2 = axes[1]
true_range = 10.0  # m
measurements = np.random.normal(true_range, 0.1, 100)  # 0.1m std dev

ax2.hist(measurements, bins=20, alpha=0.7, edgecolor='black')
ax2.axvline(x=true_range, color='r', linestyle='--', linewidth=2, label='True range')
ax2.axvline(x=np.mean(measurements), color='g', linestyle='--', linewidth=2, label='Mean measurement')
ax2.set_xlabel('Measured Range (m)')
ax2.set_ylabel('Count')
ax2.set_title(f'Range Accuracy (σ = 0.1m)')
ax2.legend()
ax2.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('resolution_vs_accuracy.png')

print("\nResolution vs Accuracy:")
print("-" * 50)
print(f"True range: {true_range:.2f} m")
print(f"Measured mean: {np.mean(measurements):.3f} m")
print(f"Std deviation: {np.std(measurements):.3f} m")
print(f"Accuracy (error): {abs(np.mean(measurements) - true_range):.3f} m")
```

### 6.2 Angular Resolution

For antenna array or scanning radar:

```
θ_resolution ≈ λ / D

Where D is antenna aperture size
```

```python
# Angular resolution
def angular_resolution(wavelength, aperture):
    """Calculate angular resolution in radians"""
    return wavelength / aperture

wavelength = c / 24e9

apertures = np.linspace(0.01, 0.5, 100)  # 1cm to 50cm
angular_res = [np.degrees(angular_resolution(wavelength, D)) for D in apertures]

plt.figure(figsize=(10, 6))
plt.plot(apertures*100, angular_res)
plt.xlabel('Antenna Aperture (cm)')
plt.ylabel('Angular Resolution (degrees)')
plt.title('Angular Resolution vs Antenna Size (24 GHz)')
plt.grid(True)
plt.savefig('angular_resolution.png')

# Examples
antenna_sizes = [0.05, 0.1, 0.2, 0.5]
print("\nAngular Resolution:")
print("-" * 50)
for D in antenna_sizes:
    theta_rad = angular_resolution(wavelength, D)
    theta_deg = np.degrees(theta_rad)
    # At 10m range, what's the cross-range resolution?
    cross_range_10m = 10 * theta_rad
    print(f"Aperture: {D*100:>4.0f} cm  →  θ = {theta_deg:>5.2f}°  "
          f"(at 10m: {cross_range_10m:>5.2f} m cross-range)")
```

---

## 7. Target Detection

### 7.1 Signal-to-Noise Ratio (SNR)

```
SNR = P_signal / P_noise

SNR (dB) = 10 × log₁₀(SNR)
```

**Typical requirement:** SNR > 10 dB for reliable detection

```python
# SNR calculation
def calculate_snr(P_signal, P_noise):
    """Calculate SNR in linear and dB"""
    snr_linear = P_signal / P_noise
    snr_db = 10 * np.log10(snr_linear)
    return snr_linear, snr_db

# Example
P_signal = 1e-9  # 1 nW
P_noise = 1e-11  # 10 pW

snr_lin, snr_db = calculate_snr(P_signal, P_noise)

print(f"Signal power: {P_signal*1e9:.1f} nW")
print(f"Noise power: {P_noise*1e12:.1f} pW")
print(f"SNR: {snr_lin:.0f} (linear) = {snr_db:.1f} dB")

# Detection probability vs SNR
snr_db_range = np.linspace(-10, 30, 100)

# Simplified detection probability (assumes Gaussian noise)
def detection_probability(snr_db, false_alarm_rate=1e-6):
    """Approximate detection probability"""
    from scipy import stats
    snr_linear = 10**(snr_db/10)
    # Threshold for given false alarm rate
    threshold = stats.norm.ppf(1 - false_alarm_rate)
    # Detection probability
    P_d = 1 - stats.norm.cdf(threshold - np.sqrt(snr_linear))
    return P_d

P_d = [detection_probability(snr) for snr in snr_db_range]

plt.figure(figsize=(10, 6))
plt.plot(snr_db_range, P_d)
plt.xlabel('SNR (dB)')
plt.ylabel('Detection Probability')
plt.title('Detection Probability vs SNR')
plt.grid(True)
plt.axhline(y=0.9, color='r', linestyle='--', alpha=0.5, label='90% detection')
plt.axvline(x=13, color='r', linestyle='--', alpha=0.5, label='~13 dB needed')
plt.legend()
plt.savefig('detection_probability_vs_snr.png')
```

### 7.2 Detection Threshold

**CFAR (Constant False Alarm Rate)** - Adaptive threshold

```python
# Simple CFAR detector
def cfar_detection(signal, guard_cells=2, training_cells=10, Pfa=1e-3):
    """
    Cell-Averaging CFAR detector
    """
    alpha = training_cells * (Pfa**(-1/training_cells) - 1)
    detected = []

    for i in range(len(signal)):
        # Training cells (avoid guard cells)
        left_start = max(0, i - guard_cells - training_cells)
        left_end = max(0, i - guard_cells)
        right_start = min(len(signal), i + guard_cells + 1)
        right_end = min(len(signal), i + guard_cells + training_cells + 1)

        training = np.concatenate([signal[left_start:left_end],
                                   signal[right_start:right_end]])

        if len(training) > 0:
            noise_level = np.mean(training)
            threshold = alpha * noise_level
            detected.append(signal[i] > threshold)
        else:
            detected.append(False)

    return np.array(detected)

# Simulate radar return with target
n_samples = 200
noise = np.random.rayleigh(1, n_samples)  # Rayleigh noise (typical for radar)

# Add target at range bin 100
signal = noise.copy()
signal[95:105] += 15  # Strong target

# Apply CFAR
detections = cfar_detection(signal)

plt.figure(figsize=(12, 6))
plt.plot(signal, 'b-', label='Radar return', alpha=0.7)
plt.plot(np.where(detections)[0], signal[detections], 'ro',
         markersize=10, label='Detections')
plt.xlabel('Range Bin')
plt.ylabel('Amplitude')
plt.title('CFAR Target Detection')
plt.legend()
plt.grid(True)
plt.savefig('cfar_detection.png')

print(f"\nCFAR Detection:")
print(f"Total samples: {n_samples}")
print(f"Detections: {np.sum(detections)}")
print(f"Detection at bins: {np.where(detections)[0]}")
```

---

## Summary

You now understand:

✅ **Radar basics** - transmit, reflect, receive, analyze
✅ **Radar equation** - how power, range, and RCS relate
✅ **Pulse vs CW vs FMCW** - different radar types
✅ **Doppler effect** - measuring velocity
✅ **FMCW operation** - best for DIY projects
✅ **Resolution** - range, velocity, and angular
✅ **Detection** - SNR, thresholds, CFAR

**Key Formulas:**

```
Range (pulsed):      R = c×Δt/2
Doppler shift:       f_d = 2×v×f/c
FMCW beat frequency: f_beat = 2×R×B/(c×T_chirp)
Range resolution:    ΔR = c/(2×B)
Velocity resolution: Δv = c/(2×f×T_obs)
Radar equation:      P_r ∝ 1/R⁴
```

---

**Next Steps:**

1. **[R3: FMCW Radar Deep Dive](./R3-fmcw-radar.md)** - Detailed FMCW implementation
2. **[R7: DSP Fundamentals](./R7-dsp-fundamentals.md)** - Signal processing for radar
3. **[R14: Build Guide](./R14-build-guide.md)** - Build your own radar

---

## Practice Problems

1. A 24 GHz radar detects a target at 50m. Calculate the round-trip time.

2. Calculate the Doppler shift for a car moving at 100 km/h (use 10 GHz radar).

3. Design an FMCW radar with 1.5m range resolution. What bandwidth is needed?

4. A radar has 10 dBi antenna gain and transmits 100 mW. Calculate received power from a 1 m² target at 100m.

5. What observation time gives 0.1 m/s velocity resolution at 77 GHz?

---

*Last Updated: November 2025*
