# Mathematics Exercises

> **Practice problems with detailed solutions**

---

## Contents

1. [Calculus Problems](#calculus-problems)
2. [Linear Algebra Problems](#linear-algebra-problems)
3. [Fourier Analysis Problems](#fourier-analysis-problems)
4. [Statistics Problems](#statistics-problems)
5. [Complex Numbers Problems](#complex-numbers-problems)

---

## Calculus Problems

### Problem 1: Velocity and Position

A quadcopter starts from rest and accelerates upward with a(t) = 2 + 0.5t m/s².

**a)** Find the velocity v(t)
**b)** Find the position s(t)
**c)** What is the altitude after 5 seconds?

**Solution:**

a) Velocity is the integral of acceleration:
```
v(t) = ∫ a(t) dt = ∫ (2 + 0.5t) dt
     = 2t + 0.25t² + C

Initial condition: v(0) = 0 → C = 0
Therefore: v(t) = 2t + 0.25t²
```

b) Position is the integral of velocity:
```
s(t) = ∫ v(t) dt = ∫ (2t + 0.25t²) dt
     = t² + 0.0833t³ + C

Initial condition: s(0) = 0 → C = 0
Therefore: s(t) = t² + 0.0833t³
```

c) At t = 5s:
```
s(5) = 5² + 0.0833(5³)
     = 25 + 0.0833(125)
     = 25 + 10.41
     = 35.41 m
```

**Check with Python:**
```python
import numpy as np

def acceleration(t):
    return 2 + 0.5*t

t = 5
# Numerical integration
from scipy.integrate import quad
velocity_at_5, _ = quad(acceleration, 0, t)
position_at_5, _ = quad(lambda x: 2*x + 0.25*x**2, 0, t)

print(f"Velocity at t=5s: {velocity_at_5:.2f} m/s")
print(f"Position at t=5s: {position_at_5:.2f} m")
```

---

### Problem 2: Motor Thrust

Thrust from a propeller is T = k·ω², where k = 1.2×10⁻⁶ N/(rad/s)² and ω is angular velocity.

**a)** Find dT/dω (rate of thrust change with speed)
**b)** At what angular velocity is the thrust 5N?
**c)** How sensitive is thrust to small speed changes at ω = 500 rad/s?

**Solution:**

a) Derivative of T with respect to ω:
```
dT/dω = d/dω(k·ω²) = 2k·ω
```

b) Solve for ω when T = 5N:
```
T = k·ω²
5 = 1.2×10⁻⁶ · ω²
ω² = 5 / (1.2×10⁻⁶)
ω² = 4.167×10⁶
ω = 2041 rad/s ≈ 19,500 RPM
```

c) Sensitivity at ω = 500 rad/s:
```
dT/dω = 2k·ω = 2(1.2×10⁻⁶)(500)
      = 0.0012 N/(rad/s)

A 1 rad/s change causes 0.0012N thrust change
A 10 rad/s change causes 0.012N thrust change
```

---

### Problem 3: RC Time Constant

An RC low-pass filter has R = 10kΩ, C = 100nF. Voltage starts at 5V and discharges.

**a)** Write the equation for V(t)
**b)** What is the voltage after one time constant?
**c)** When does voltage reach 1V?

**Solution:**

a) Discharge equation:
```
τ = RC = 10,000 × 100×10⁻⁹ = 1×10⁻³ s = 1 ms

V(t) = V₀ · e^(-t/τ)
V(t) = 5 · e^(-t/0.001)
```

b) After one time constant (t = τ):
```
V(τ) = 5 · e^(-1)
     = 5 × 0.368
     = 1.84 V (36.8% of initial)
```

c) Solve for t when V = 1V:
```
1 = 5 · e^(-t/0.001)
e^(-t/0.001) = 0.2
-t/0.001 = ln(0.2)
-t/0.001 = -1.609
t = 1.609 ms
```

```python
import numpy as np
import matplotlib.pyplot as plt

R = 10e3
C = 100e-9
tau = R * C
V0 = 5

t = np.linspace(0, 5*tau, 1000)
V = V0 * np.exp(-t/tau)

# Find when V = 1V
t_1V = -tau * np.log(1/V0)

plt.figure(figsize=(10, 6))
plt.plot(t*1000, V)
plt.axhline(y=1, color='r', linestyle='--', label=f'V=1V at t={t_1V*1000:.2f}ms')
plt.axvline(x=tau*1000, color='g', linestyle='--', label=f'τ={tau*1000:.0f}ms')
plt.xlabel('Time (ms)')
plt.ylabel('Voltage (V)')
plt.title('RC Discharge')
plt.legend()
plt.grid(True)
plt.savefig('rc_discharge_problem.png')
```

---

## Linear Algebra Problems

### Problem 4: Rotation Matrix

A quadcopter is rolled 30° and pitched 20°. A vector [1, 0, 0] (forward in body frame) needs to be transformed to earth frame.

**a)** Calculate the rotation matrix
**b)** Transform the vector
**c)** What are the earth-frame components?

**Solution:**

a) Roll (about X) and pitch (about Y) rotation matrices:

```python
import numpy as np

roll = np.radians(30)
pitch = np.radians(20)

# Roll rotation matrix (X-axis)
Rx = np.array([
    [1, 0, 0],
    [0, np.cos(roll), -np.sin(roll)],
    [0, np.sin(roll), np.cos(roll)]
])

# Pitch rotation matrix (Y-axis)
Ry = np.array([
    [np.cos(pitch), 0, np.sin(pitch)],
    [0, 1, 0],
    [-np.sin(pitch), 0, np.cos(pitch)]
])

# Combined rotation (pitch then roll)
R = Ry @ Rx

print("Rotation matrix R:")
print(R)
```

b) Transform vector:
```python
v_body = np.array([1, 0, 0])
v_earth = R @ v_body

print(f"\nBody frame vector: {v_body}")
print(f"Earth frame vector: {v_earth}")
```

c) Components:
```
v_earth = [0.940, 0.171, -0.296]

X (forward): 0.940 - pointing mostly forward
Y (right): 0.171 - small right component due to roll
Z (down): -0.296 - upward component due to pitch
```

This shows the quadcopter is tilted forward (causing forward motion) and slightly right.

---

### Problem 5: Motor Mixing Matrix

Given motor thrusts T1, T2, T3, T4 in a X-configuration, derive the matrix that relates them to roll, pitch, yaw moments and total thrust.

**Solution:**

For X-configuration with arm length L = 0.15m:

```python
L = 0.15  # meters
k = 0.015  # torque coefficient

# Motor positions (from center)
# M1: front-right (+L, -L)
# M2: back-left  (-L, +L)
# M3: front-left (+L, +L)
# M4: back-right (-L, -L)

# Mixing matrix: [Total, Roll, Pitch, Yaw]ᵀ = M × [T1, T2, T3, T4]ᵀ
M = np.array([
    [1,    1,    1,    1],      # Total thrust
    [L,   -L,   -L,    L],      # Roll moment
    [-L,   L,   -L,    L],      # Pitch moment
    [k,   -k,    k,   -k]       # Yaw moment
])

print("Mixing matrix M:")
print(M)

# Example: Hover with slight roll right
thrusts = np.array([4.0, 3.5, 3.5, 4.0])  # N
outputs = M @ thrusts

print(f"\nMotor thrusts: {thrusts}")
print(f"Total thrust: {outputs[0]:.2f} N")
print(f"Roll moment: {outputs[1]:.3f} N·m")
print(f"Pitch moment: {outputs[2]:.3f} N·m")
print(f"Yaw moment: {outputs[3]:.4f} N·m")
```

**Inverse (control to motors):**
```python
M_inv = np.linalg.inv(M)

# Desired: 15N total, 0.1 N·m roll right, 0 pitch, 0 yaw
desired = np.array([15, 0.1, 0, 0])
required_thrusts = M_inv @ desired

print(f"\nDesired control: {desired}")
print(f"Required motor thrusts: {required_thrusts}")
```

---

## Fourier Analysis Problems

### Problem 6: Radar Doppler Shift

A 24 GHz radar detects a ball moving at 25 m/s toward the radar.

**a)** Calculate the Doppler frequency shift
**b)** If sampled at 100 kHz, what frequency bin will the peak appear in (1024-point FFT)?
**c)** What velocity resolution is achieved with 10ms observation?

**Solution:**

a) Doppler shift:
```
f_d = 2·v·f_c / c
    = 2 × 25 × 24×10⁹ / (3×10⁸)
    = 4000 Hz = 4 kHz
```

b) FFT frequency resolution:
```python
f_sample = 100e3  # 100 kHz
N = 1024
f_doppler = 4000  # Hz

# Frequency resolution
df = f_sample / N
print(f"Frequency resolution: {df:.2f} Hz/bin")

# Bin number
bin_num = int(f_doppler / df)
print(f"Doppler appears in bin: {bin_num}")
```

c) Velocity resolution:
```
T_obs = 10e-3  # 10 ms
Δf = 1 / T_obs = 100 Hz

Δv = Δf × c / (2 × f_c)
   = 100 × 3×10⁸ / (2 × 24×10⁹)
   = 0.625 m/s
```

So we can resolve velocities to ±0.625 m/s with 10ms observation.

---

### Problem 7: Aliasing

An IMU samples at 1000 Hz. A vibration occurs at 800 Hz.

**a)** What is the Nyquist frequency?
**b)** Will 800 Hz be aliased?
**c)** What frequency will it appear as?
**d)** Design an anti-aliasing filter.

**Solution:**

a) Nyquist frequency = f_sample / 2 = 500 Hz

b) Yes! 800 Hz > 500 Hz → aliased

c) Aliased frequency:
```
f_alias = |f_sample - f_actual|
        = |1000 - 800|
        = 200 Hz
```

The 800 Hz vibration appears as 200 Hz!

d) Anti-aliasing low-pass filter:
```python
# Design RC filter with fc = 400 Hz (below Nyquist)
fc = 400  # Hz
C = 10e-9  # 10nF (chosen)
R = 1 / (2 * np.pi * fc * C)

print(f"Anti-aliasing filter:")
print(f"R = {R/1000:.1f} kΩ")
print(f"C = {C*1e9:.0f} nF")
print(f"fc = {fc} Hz")

# Attenuation at 800 Hz
f_vib = 800
attenuation_dB = -20 * np.log10(np.sqrt(1 + (f_vib/fc)**2))
print(f"Attenuation at {f_vib}Hz: {attenuation_dB:.1f} dB")
```

---

## Statistics Problems

### Problem 8: Sensor Noise

An accelerometer has noise density of 400 μg/√Hz. Bandwidth is 100 Hz.

**a)** Calculate RMS noise
**b)** If 1000 samples are averaged, what is the noise reduction?
**c)** How does this affect position uncertainty after 10 seconds?

**Solution:**

a) RMS noise:
```
Noise_RMS = noise_density × √bandwidth
          = 400×10⁻⁶ g × √100
          = 400×10⁻⁶ × 10
          = 4×10⁻³ g = 4 mg

In m/s²: 4×10⁻³ × 9.81 = 0.039 m/s²
```

b) Averaging N samples reduces noise by √N:
```
Noise_reduced = Noise_RMS / √N
              = 0.039 / √1000
              = 0.039 / 31.62
              = 0.001 m/s²
```

c) Position uncertainty from acceleration noise:
```
σ_position = σ_accel × t² / 2
           = 0.039 × 10² / 2
           = 1.95 m

With averaging:
σ_position_averaged = 0.001 × 100 / 2 = 0.05 m
```

This is why sensor fusion is critical!

```python
import numpy as np

# Simulate
dt = 0.01  # 100 Hz
time = np.arange(0, 10, dt)
noise_std = 0.039  # m/s²

# Generate noisy acceleration measurements
accel_noise = np.random.normal(0, noise_std, len(time))

# Integrate twice (velocity then position)
velocity = np.cumsum(accel_noise) * dt
position = np.cumsum(velocity) * dt

print(f"Position drift after 10s: {position[-1]:.2f} m")
print(f"Position std dev: {np.std(position):.2f} m")
```

---

### Problem 9: Kalman Filter Update

A simple 1D Kalman filter has:
- Prior estimate: x̂ = 5.0 m, P = 2.0 m²
- Measurement: z = 5.5 m with R = 1.0 m²

Calculate the updated estimate and covariance.

**Solution:**

Kalman update equations:
```
K = P / (P + R)
x̂_new = x̂ + K(z - x̂)
P_new = (1 - K) × P
```

Calculation:
```python
# Prior
x_prior = 5.0
P_prior = 2.0

# Measurement
z = 5.5
R = 1.0

# Kalman gain
K = P_prior / (P_prior + R)
print(f"Kalman gain K = {K:.3f}")

# Update estimate
innovation = z - x_prior
x_posterior = x_prior + K * innovation
print(f"Innovation = {innovation}")
print(f"Updated estimate = {x_posterior:.3f} m")

# Update covariance
P_posterior = (1 - K) * P_prior
print(f"Updated covariance = {P_posterior:.3f} m²")
print(f"Uncertainty reduced by {(1-P_posterior/P_prior)*100:.1f}%")
```

**Result:**
- K ≈ 0.667 (trusts prior more than measurement, since P < R)
- x̂ = 5.33 m (weighted average of 5.0 and 5.5)
- P = 0.67 m² (uncertainty reduced)

---

## Complex Numbers Problems

### Problem 10: FMCW Radar Beat Frequency

An FMCW radar transmits: TX(t) = cos(2π(f₀ + αt)t) where f₀ = 24 GHz, α = 200 MHz/ms.
Target at 10m creates delay τ = 2R/c.

**a)** Express TX and RX as complex exponentials
**b)** Calculate beat frequency
**c)** After mixing (TX × RX*), what frequency remains?

**Solution:**

a) Complex exponential form:
```
TX(t) = e^(j2π(f₀t + αt²/2))

RX(t) = e^(j2π(f₀(t-τ) + α(t-τ)²/2))
```

b) Beat frequency from time delay:
```python
f0 = 24e9  # Hz
alpha = 200e6 / 1e-3  # Hz/s
R = 10  # m
c = 3e8
tau = 2*R/c

f_beat = alpha * tau
print(f"Time delay τ = {tau*1e9:.3f} ns")
print(f"Beat frequency = {f_beat/1e3:.2f} kHz")

# Verify range calculation
R_calculated = f_beat * c / (2 * alpha)
print(f"Calculated range = {R_calculated:.2f} m")
```

c) After mixing (multiply and take low-pass):
```
IF(t) = TX(t) × RX*(t)
      ≈ e^(j2π·f_beat·t)
```

The intermediate frequency (IF) is just the beat frequency (13.33 kHz for 10m).

---

## Challenge Problems

### Challenge 1: PID Gains from Ziegler-Nichols

A quadcopter's roll rate loop shows sustained oscillation at Kp = 2.5 with period Tu = 0.4s.
Calculate recommended PID gains and simulate the response.

**Challenge 2: Sensor Fusion Comparison

Implement both complementary and Kalman filters for IMU data. Compare:
- Computational cost
- Accuracy
- Response time
- Noise rejection

**Challenge 3: Range-Doppler Processing

Simulate an FMCW radar with two targets:
- Target 1: 15m, stationary
- Target 2: 25m, approaching at 10 m/s

Implement 2D FFT to create range-Doppler map.

---

## Summary

**Key Concepts Practiced:**

✅ Calculus - Integration, differentiation for dynamics
✅ Linear algebra - Rotations, transformations, matrices
✅ Fourier analysis - FFT, Doppler, aliasing
✅ Statistics - Noise, filtering, Kalman updates
✅ Complex numbers - Radar signal processing

**Next Steps:**

1. Attempt all problems without looking at solutions
2. Implement solutions in Python
3. Verify with simulations
4. Apply to your actual projects

---

**Solutions Code Repository:** [See /code/exercises/](../code/exercises/)

*Last Updated: November 2025*
