# Mathematics Foundations

> **Essential mathematical concepts for quadcopter flight control and radar signal processing**

---

## Table of Contents

1. [Calculus](#1-calculus)
2. [Linear Algebra](#2-linear-algebra)
3. [Differential Equations](#3-differential-equations)
4. [Fourier Analysis & Transforms](#4-fourier-analysis--transforms)
5. [Probability & Statistics](#5-probability--statistics)
6. [Complex Numbers & Phasors](#6-complex-numbers--phasors)
7. [Numerical Methods](#7-numerical-methods)

---

## 1. Calculus

### 1.1 Why You Need Calculus

**For Quadcopters:**
- Derivatives describe rates of change (velocity from position, acceleration from velocity)
- Integrals accumulate sensor data over time
- PID controllers use derivatives and integrals explicitly

**For Radar:**
- Signal analysis involves continuous functions
- Doppler shift calculations use derivatives
- Power and energy calculations use integration

### 1.2 Derivatives - Rate of Change

#### The Fundamental Concept

A derivative measures how a function changes as its input changes.

**Definition:**
```
f'(x) = lim[h→0] (f(x+h) - f(x))/h
```

**Physical Interpretation:**
- Position → Velocity (first derivative)
- Velocity → Acceleration (second derivative)
- Acceleration → Jerk (third derivative)

#### Essential Derivative Rules

```
Power Rule:        d/dx(x^n) = n·x^(n-1)
Product Rule:      d/dx(f·g) = f'·g + f·g'
Quotient Rule:     d/dx(f/g) = (f'·g - f·g')/g²
Chain Rule:        d/dx(f(g(x))) = f'(g(x))·g'(x)

Common Functions:
d/dx(sin(x)) = cos(x)
d/dx(cos(x)) = -sin(x)
d/dx(e^x) = e^x
d/dx(ln(x)) = 1/x
```

#### Practical Example: Quadcopter Motion

```python
import numpy as np
import matplotlib.pyplot as plt

# Position as a function of time: s(t) = 5t² + 2t + 1
t = np.linspace(0, 5, 100)
position = 5*t**2 + 2*t + 1

# Velocity is the derivative: v(t) = ds/dt = 10t + 2
velocity = 10*t + 2

# Acceleration is the second derivative: a(t) = d²s/dt² = 10
acceleration = 10 * np.ones_like(t)

# Plot all three
fig, (ax1, ax2, ax3) = plt.subplots(3, 1, figsize=(10, 8))

ax1.plot(t, position)
ax1.set_ylabel('Position (m)')
ax1.set_title('Position, Velocity, and Acceleration')
ax1.grid(True)

ax2.plot(t, velocity, color='orange')
ax2.set_ylabel('Velocity (m/s)')
ax2.grid(True)

ax3.plot(t, acceleration, color='red')
ax3.set_ylabel('Acceleration (m/s²)')
ax3.set_xlabel('Time (s)')
ax3.grid(True)

plt.tight_layout()
plt.savefig('motion_derivatives.png')
```

#### Partial Derivatives

When functions have multiple variables, we use partial derivatives.

**Example:** Temperature field T(x, y, t)
```
∂T/∂x  = rate of change in x-direction
∂T/∂y  = rate of change in y-direction
∂T/∂t  = rate of change over time
```

**For Quadcopters:** Forces depend on multiple variables:
```
Lift = f(velocity, angle_of_attack, air_density)

∂Lift/∂velocity = how lift changes with speed
∂Lift/∂angle = how lift changes with angle
```

### 1.3 Integrals - Accumulation

#### The Fundamental Concept

An integral accumulates a quantity over a range.

**Definite Integral:**
```
∫[a to b] f(x)dx = Area under curve from a to b
```

**Physical Interpretations:**
- Velocity over time → Distance traveled
- Acceleration over time → Velocity change
- Power over time → Energy consumed

#### Essential Integration Rules

```
Power Rule:        ∫ x^n dx = x^(n+1)/(n+1) + C  (n ≠ -1)
Exponential:       ∫ e^x dx = e^x + C
Trigonometric:     ∫ sin(x)dx = -cos(x) + C
                   ∫ cos(x)dx = sin(x) + C

Substitution:      ∫ f(g(x))g'(x)dx = ∫ f(u)du  where u = g(x)
Integration by Parts: ∫ u dv = uv - ∫ v du
```

#### Practical Example: IMU Integration

```python
# Accelerometer data integration to get velocity and position
# This is actually done in quadcopter flight controllers!

import numpy as np

# Simulated accelerometer data (m/s²)
dt = 0.01  # 100 Hz sampling rate
time = np.arange(0, 5, dt)
# Constant acceleration of 2 m/s²
accel = 2 * np.ones_like(time)

# Integrate acceleration to get velocity (Trapezoidal rule)
velocity = np.zeros_like(accel)
for i in range(1, len(accel)):
    velocity[i] = velocity[i-1] + accel[i] * dt

# Integrate velocity to get position
position = np.zeros_like(velocity)
for i in range(1, len(velocity)):
    position[i] = position[i-1] + velocity[i] * dt

print(f"Final velocity: {velocity[-1]:.2f} m/s")
print(f"Final position: {position[-1]:.2f} m")

# Compare with analytical solution
# v = at = 2 * 5 = 10 m/s
# s = 0.5at² = 0.5 * 2 * 25 = 25 m
print(f"Analytical velocity: 10 m/s")
print(f"Analytical position: 25 m")
```

**Problem with IMU Integration:** Sensor noise and bias accumulate, causing **drift**. This is why we need sensor fusion!

### 1.4 Multivariable Calculus

#### Gradient Vector

The gradient points in the direction of steepest increase:

```
∇f = [∂f/∂x, ∂f/∂y, ∂f/∂z]
```

**Application:** Gradient descent optimization (used in PID tuning, ML)

#### Divergence

Measures how much a vector field spreads out:

```
div F = ∇·F = ∂Fx/∂x + ∂Fy/∂y + ∂Fz/∂z
```

**Application:** Fluid flow, electromagnetic field analysis

#### Curl

Measures rotation of a vector field:

```
curl F = ∇×F
```

**Application:** Rotational dynamics, electromagnetic fields

---

## 2. Linear Algebra

### 2.1 Why Linear Algebra Matters

**For Quadcopters:**
- Rotation matrices transform between coordinate systems
- State-space representation of dynamics
- Kalman filters use matrix operations
- Quaternion math (alternative to Euler angles)

**For Radar:**
- Signal processing algorithms
- Multiple antenna arrays (beamforming)
- Matrix inversions in calibration

### 2.2 Vectors

#### Vector Basics

A vector is a quantity with magnitude and direction.

**Representation:**
```
v = [v₁, v₂, v₃]ᵀ  (column vector)
```

**Operations:**
```python
import numpy as np

# Define vectors
v1 = np.array([1, 2, 3])
v2 = np.array([4, 5, 6])

# Addition
v_sum = v1 + v2
print(f"v1 + v2 = {v_sum}")  # [5, 7, 9]

# Scalar multiplication
v_scaled = 2 * v1
print(f"2*v1 = {v_scaled}")  # [2, 4, 6]

# Magnitude (norm)
magnitude = np.linalg.norm(v1)
print(f"|v1| = {magnitude:.3f}")  # √(1²+2²+3²) = √14

# Unit vector
v1_unit = v1 / magnitude
print(f"v1_unit = {v1_unit}")
```

#### Dot Product (Inner Product)

```
v₁ · v₂ = |v₁||v₂|cos(θ)
       = v₁ₓv₂ₓ + v₁ᵧv₂ᵧ + v₁ᵤv₂ᵤ
```

**Geometric Meaning:** Projection of one vector onto another

**Applications:**
- Check if vectors are perpendicular (dot product = 0)
- Find angle between vectors
- Calculate work done by a force

```python
# Dot product
dot_product = np.dot(v1, v2)
print(f"v1 · v2 = {dot_product}")  # 1*4 + 2*5 + 3*6 = 32

# Angle between vectors
cos_theta = dot_product / (np.linalg.norm(v1) * np.linalg.norm(v2))
theta_rad = np.arccos(cos_theta)
theta_deg = np.degrees(theta_rad)
print(f"Angle between v1 and v2: {theta_deg:.2f}°")
```

#### Cross Product

```
v₁ × v₂ = |v₁||v₂|sin(θ) n̂
```

Where n̂ is perpendicular to both v₁ and v₂ (right-hand rule).

**Component Form:**
```
v₁ × v₂ = [v₁ᵧv₂ᵤ - v₁ᵤv₂ᵧ]
          [v₁ᵤv₂ₓ - v₁ₓv₂ᵤ]
          [v₁ₓv₂ᵧ - v₁ᵧv₂ₓ]
```

**Applications:**
- Torque = r × F
- Angular momentum
- Normal to a plane

```python
# Cross product
cross_product = np.cross(v1, v2)
print(f"v1 × v2 = {cross_product}")  # [-3, 6, -3]

# The result is perpendicular to both v1 and v2
print(f"(v1 × v2) · v1 = {np.dot(cross_product, v1)}")  # Should be 0
print(f"(v1 × v2) · v2 = {np.dot(cross_product, v2)}")  # Should be 0
```

### 2.3 Matrices

#### Matrix as Linear Transformation

A matrix transforms vectors from one space to another.

```python
import numpy as np

# 2D rotation matrix (rotate by 45°)
theta = np.pi/4
R = np.array([
    [np.cos(theta), -np.sin(theta)],
    [np.sin(theta),  np.cos(theta)]
])

# Original vector
v = np.array([1, 0])

# Rotated vector
v_rotated = R @ v  # @ is matrix multiplication in Python
print(f"Original: {v}")
print(f"Rotated 45°: {v_rotated}")
```

#### Essential Matrix Operations

**Transpose:**
```python
A = np.array([[1, 2, 3],
              [4, 5, 6]])
A_T = A.T  # Swap rows and columns
print(f"A:\n{A}")
print(f"A^T:\n{A_T}")
```

**Matrix Multiplication:**
```python
# Matrix-vector multiplication
A = np.array([[1, 2],
              [3, 4]])
v = np.array([5, 6])
result = A @ v
print(f"Av = {result}")  # [1*5+2*6, 3*5+4*6] = [17, 39]

# Matrix-matrix multiplication
B = np.array([[7, 8],
              [9, 10]])
C = A @ B
print(f"AB =\n{C}")
```

**Important:** Matrix multiplication is NOT commutative: AB ≠ BA

**Inverse:**
```python
# For square matrix A, inverse A⁻¹ satisfies: A @ A⁻¹ = I
A = np.array([[4, 7],
              [2, 6]])
A_inv = np.linalg.inv(A)
print(f"A:\n{A}")
print(f"A⁻¹:\n{A_inv}")
print(f"A @ A⁻¹ =\n{A @ A_inv}")  # Should be identity matrix
```

**Determinant:**
```python
det_A = np.linalg.det(A)
print(f"det(A) = {det_A}")
```

- If det(A) = 0, matrix is singular (no inverse exists)
- |det(A)| = scaling factor of transformation

### 2.4 Rotation Matrices (Critical for Quadcopters!)

#### 2D Rotation

```python
def rotation_matrix_2d(theta):
    """Rotate by angle theta (radians) counterclockwise"""
    return np.array([
        [np.cos(theta), -np.sin(theta)],
        [np.sin(theta),  np.cos(theta)]
    ])

# Example: Rotate point (1, 0) by 90°
R = rotation_matrix_2d(np.pi/2)
point = np.array([1, 0])
rotated = R @ point
print(f"(1,0) rotated 90°: {rotated}")  # Should be (0, 1)
```

#### 3D Rotations

**Roll (rotation about X-axis):**
```python
def rotation_x(phi):
    return np.array([
        [1, 0, 0],
        [0, np.cos(phi), -np.sin(phi)],
        [0, np.sin(phi),  np.cos(phi)]
    ])
```

**Pitch (rotation about Y-axis):**
```python
def rotation_y(theta):
    return np.array([
        [np.cos(theta), 0, np.sin(theta)],
        [0, 1, 0],
        [-np.sin(theta), 0, np.cos(theta)]
    ])
```

**Yaw (rotation about Z-axis):**
```python
def rotation_z(psi):
    return np.array([
        [np.cos(psi), -np.sin(psi), 0],
        [np.sin(psi),  np.cos(psi), 0],
        [0, 0, 1]
    ])
```

**Combined Rotation (Euler Angles - ZYX convention):**
```python
def euler_to_rotation_matrix(roll, pitch, yaw):
    """
    Convert Euler angles to rotation matrix.
    This transforms from body frame to earth frame.
    """
    R_x = rotation_x(roll)
    R_y = rotation_y(pitch)
    R_z = rotation_z(yaw)

    # Order matters! ZYX convention (yaw, then pitch, then roll)
    R = R_z @ R_y @ R_x
    return R

# Example
roll = np.radians(10)   # 10° roll
pitch = np.radians(20)  # 20° pitch
yaw = np.radians(30)    # 30° yaw

R = euler_to_rotation_matrix(roll, pitch, yaw)
print(f"Rotation matrix:\n{R}")

# Transform a vector from body frame to earth frame
body_vector = np.array([1, 0, 0])  # Pointing forward in body frame
earth_vector = R @ body_vector
print(f"Body [1,0,0] → Earth {earth_vector}")
```

### 2.5 Eigenvalues and Eigenvectors

#### Concept

For matrix A, if Av = λv, then:
- v is an eigenvector
- λ is the corresponding eigenvalue

**Physical Meaning:** Eigenvectors are special directions that only get scaled (not rotated) by the transformation.

```python
# Find eigenvalues and eigenvectors
A = np.array([[4, -2],
              [1,  1]])

eigenvalues, eigenvectors = np.linalg.eig(A)

print(f"Eigenvalues: {eigenvalues}")
print(f"Eigenvectors:\n{eigenvectors}")

# Verify: Av = λv for first eigenvector
v1 = eigenvectors[:, 0]
lambda1 = eigenvalues[0]
print(f"\nVerification:")
print(f"Av1 = {A @ v1}")
print(f"λ₁v1 = {lambda1 * v1}")
```

**Applications:**
- Stability analysis of control systems
- Principal component analysis (PCA)
- Vibration modes of structures

---

## 3. Differential Equations

### 3.1 Why Differential Equations?

Physical systems are described by differential equations:
- Newton's laws: F = ma → d²x/dt² = F/m
- RC circuits: τ(dV/dt) + V = V_in
- PID controllers: u(t) = K_p·e + K_i·∫e dt + K_d·de/dt

### 3.2 First-Order ODEs

#### General Form
```
dy/dt = f(t, y)
```

#### Example: RC Circuit

```
τ(dV/dt) + V = V_in
```

Where τ = RC (time constant).

**Solution for step input:**
```
V(t) = V_in(1 - e^(-t/τ))
```

```python
import numpy as np
import matplotlib.pyplot as plt

# RC circuit parameters
R = 1000  # 1 kΩ
C = 1e-6  # 1 μF
tau = R * C  # Time constant = 1 ms
V_in = 5  # 5V step input

# Time array
t = np.linspace(0, 5*tau, 1000)

# Analytical solution
V = V_in * (1 - np.exp(-t/tau))

# Plot
plt.figure(figsize=(10, 6))
plt.plot(t*1000, V)  # Convert to milliseconds
plt.axhline(y=V_in*0.632, color='r', linestyle='--',
            label=f'63.2% at t=τ={tau*1000:.2f}ms')
plt.axvline(x=tau*1000, color='r', linestyle='--')
plt.xlabel('Time (ms)')
plt.ylabel('Voltage (V)')
plt.title('RC Circuit Step Response')
plt.grid(True)
plt.legend()
plt.savefig('rc_step_response.png')
```

### 3.3 Second-Order ODEs

#### General Form (Linear)
```
a(d²y/dt²) + b(dy/dt) + cy = f(t)
```

#### Example: Mass-Spring-Damper System

This is fundamental for understanding quadcopter dynamics!

```
m(d²x/dt²) + c(dx/dt) + kx = F(t)
```

Where:
- m = mass
- c = damping coefficient
- k = spring constant
- F(t) = external force

**Characteristic Equation:**
```
mλ² + cλ + k = 0
```

**Solutions depend on discriminant:**

1. **Overdamped** (c² > 4mk): Slow return, no oscillation
2. **Critically damped** (c² = 4mk): Fastest return without oscillation
3. **Underdamped** (c² < 4mk): Oscillates before settling

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.integrate import odeint

def mass_spring_damper(state, t, m, c, k, F):
    """
    State: [position, velocity]
    Returns: [velocity, acceleration]
    """
    x, v = state
    dxdt = v
    dvdt = (F - c*v - k*x) / m
    return [dxdt, dvdt]

# System parameters
m = 1.0  # kg
k = 10.0  # N/m
F = 0    # No external force (free response)

# Different damping values
c_underdamped = 1.0     # ζ < 1
c_critical = 2*np.sqrt(k*m)  # ζ = 1
c_overdamped = 10.0     # ζ > 1

# Initial conditions: displaced 1m, zero velocity
initial_state = [1.0, 0.0]
t = np.linspace(0, 10, 1000)

# Solve for each damping case
sol_under = odeint(mass_spring_damper, initial_state, t,
                   args=(m, c_underdamped, k, F))
sol_critical = odeint(mass_spring_damper, initial_state, t,
                      args=(m, c_critical, k, F))
sol_over = odeint(mass_spring_damper, initial_state, t,
                  args=(m, c_overdamped, k, F))

# Plot
plt.figure(figsize=(12, 6))
plt.plot(t, sol_under[:, 0], label='Underdamped (ζ < 1)')
plt.plot(t, sol_critical[:, 0], label='Critically damped (ζ = 1)')
plt.plot(t, sol_over[:, 0], label='Overdamped (ζ > 1)')
plt.xlabel('Time (s)')
plt.ylabel('Position (m)')
plt.title('Mass-Spring-Damper System Response')
plt.legend()
plt.grid(True)
plt.savefig('damping_comparison.png')
```

**Relevance to Quadcopters:**
- PID tuning is about achieving critically damped or slightly underdamped response
- Too little gain → slow response (overdamped)
- Too much gain → oscillations (underdamped)

### 3.4 Numerical Solution Methods

For complex equations without analytical solutions, we use numerical methods.

#### Euler's Method (Simple but inaccurate)

```python
def euler_method(f, y0, t):
    """
    Solve dy/dt = f(t, y) using Euler's method
    """
    y = np.zeros(len(t))
    y[0] = y0
    for i in range(len(t)-1):
        dt = t[i+1] - t[i]
        y[i+1] = y[i] + f(t[i], y[i]) * dt
    return y

# Example: dy/dt = -y (exponential decay)
f = lambda t, y: -y
t = np.linspace(0, 5, 100)
y_euler = euler_method(f, 1.0, t)
y_exact = np.exp(-t)

plt.figure(figsize=(10, 6))
plt.plot(t, y_exact, 'k-', label='Exact solution', linewidth=2)
plt.plot(t, y_euler, 'r--', label='Euler method')
plt.xlabel('Time')
plt.ylabel('y')
plt.title('Euler Method vs Exact Solution')
plt.legend()
plt.grid(True)
plt.savefig('euler_method.png')
```

#### Runge-Kutta Methods (More accurate)

**RK4 (4th order Runge-Kutta)** is commonly used in flight controllers:

```python
def rk4_step(f, t, y, dt):
    """Single RK4 step"""
    k1 = f(t, y)
    k2 = f(t + dt/2, y + k1*dt/2)
    k3 = f(t + dt/2, y + k2*dt/2)
    k4 = f(t + dt, y + k3*dt)
    return y + (k1 + 2*k2 + 2*k3 + k4) * dt/6

def rk4_method(f, y0, t):
    """Solve ODE using RK4"""
    y = np.zeros(len(t))
    y[0] = y0
    for i in range(len(t)-1):
        dt = t[i+1] - t[i]
        y[i+1] = rk4_step(f, t[i], y[i], dt)
    return y

y_rk4 = rk4_method(f, 1.0, t)

plt.figure(figsize=(10, 6))
plt.plot(t, y_exact, 'k-', label='Exact', linewidth=2)
plt.plot(t, y_euler, 'r--', label='Euler', alpha=0.7)
plt.plot(t, y_rk4, 'b:', label='RK4', linewidth=2)
plt.xlabel('Time')
plt.ylabel('y')
plt.title('Comparison of Numerical Methods')
plt.legend()
plt.grid(True)
plt.savefig('numerical_methods_comparison.png')
```

---

## 4. Fourier Analysis & Transforms

### 4.1 Why Fourier Analysis?

**For Radar:**
- Convert time-domain signals to frequency domain
- Identify target velocity from Doppler shift
- Filter unwanted frequencies

**For Quadcopters:**
- Analyze vibrations and resonances
- Design digital filters
- Process IMU data

### 4.2 Fourier Series

Any periodic function can be represented as a sum of sines and cosines:

```
f(t) = a₀/2 + Σ[aₙcos(nωt) + bₙsin(nωt)]
```

**Example: Square Wave**

```python
import numpy as np
import matplotlib.pyplot as plt

def fourier_square_wave(t, n_terms):
    """
    Approximate square wave using Fourier series
    f(t) = (4/π) * Σ[sin((2k-1)t)/(2k-1)] for k=1,2,3,...
    """
    result = np.zeros_like(t)
    for k in range(1, n_terms+1):
        result += np.sin((2*k-1)*t) / (2*k-1)
    return (4/np.pi) * result

t = np.linspace(0, 4*np.pi, 1000)

fig, axes = plt.subplots(2, 2, figsize=(12, 10))
terms_list = [1, 3, 10, 50]

for ax, n in zip(axes.flat, terms_list):
    approx = fourier_square_wave(t, n)
    ax.plot(t, approx)
    ax.set_title(f'{n} terms')
    ax.set_ylim(-1.5, 1.5)
    ax.grid(True)
    ax.axhline(y=1, color='r', linestyle='--', alpha=0.3)
    ax.axhline(y=-1, color='r', linestyle='--', alpha=0.3)

plt.suptitle('Fourier Series Approximation of Square Wave')
plt.tight_layout()
plt.savefig('fourier_series_square_wave.png')
```

**Key Insight:** Complex periodic signals are composed of simple sinusoids!

### 4.3 Fourier Transform

Extends Fourier series to non-periodic functions:

```
F(ω) = ∫[-∞ to ∞] f(t)e^(-jωt) dt

f(t) = (1/2π) ∫[-∞ to ∞] F(ω)e^(jωt) dω
```

**Important Transform Pairs:**

```
Time Domain          ↔  Frequency Domain
────────────────────────────────────────
δ(t) (impulse)       ↔  1 (constant)
1 (constant)         ↔  2πδ(ω)
e^(-at)u(t)          ↔  1/(a + jω)
cos(ω₀t)             ↔  π[δ(ω-ω₀) + δ(ω+ω₀)]
e^(-t²/2σ²) (Gauss)  ↔  σ√(2π)e^(-ω²σ²/2)
```

**Properties:**

```
Linearity:      F{af(t) + bg(t)} = aF(ω) + bG(ω)
Time Shift:     F{f(t-t₀)} = e^(-jωt₀)F(ω)
Frequency Shift: F{e^(jω₀t)f(t)} = F(ω-ω₀)
Scaling:        F{f(at)} = (1/|a|)F(ω/a)
Convolution:    F{f*g} = F(ω)·G(ω)
```

### 4.4 Discrete Fourier Transform (DFT)

For sampled data (what we actually use in computers):

```
X[k] = Σ[n=0 to N-1] x[n]e^(-j2πkn/N)

x[n] = (1/N) Σ[k=0 to N-1] X[k]e^(j2πkn/N)
```

**Fast Fourier Transform (FFT)** is an efficient algorithm for computing DFT.

```python
import numpy as np
import matplotlib.pyplot as plt

# Create a signal: 50 Hz + 120 Hz sine waves
fs = 1000  # Sampling frequency (Hz)
t = np.linspace(0, 1, fs, endpoint=False)
signal = np.sin(2*np.pi*50*t) + 0.5*np.sin(2*np.pi*120*t)

# Add some noise
np.random.seed(42)
signal_noisy = signal + 0.2*np.random.randn(len(signal))

# Compute FFT
fft_result = np.fft.fft(signal_noisy)
fft_freq = np.fft.fftfreq(len(signal_noisy), 1/fs)

# Take only positive frequencies
positive_freq_idx = fft_freq > 0
fft_magnitude = np.abs(fft_result[positive_freq_idx])
fft_freq_positive = fft_freq[positive_freq_idx]

# Plot
fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(12, 10))

# Time domain
ax1.plot(t[:200], signal_noisy[:200])  # First 200 samples
ax1.set_xlabel('Time (s)')
ax1.set_ylabel('Amplitude')
ax1.set_title('Time Domain Signal (50 Hz + 120 Hz + Noise)')
ax1.grid(True)

# Frequency domain
ax2.plot(fft_freq_positive, fft_magnitude)
ax2.set_xlabel('Frequency (Hz)')
ax2.set_ylabel('Magnitude')
ax2.set_title('FFT - Frequency Domain')
ax2.set_xlim(0, 200)
ax2.grid(True)
ax2.axvline(x=50, color='r', linestyle='--', alpha=0.5, label='50 Hz')
ax2.axvline(x=120, color='g', linestyle='--', alpha=0.5, label='120 Hz')
ax2.legend()

plt.tight_layout()
plt.savefig('fft_example.png')
```

### 4.5 Practical Application: Doppler Radar

```python
# Simulate a Doppler radar return signal
fs = 10000  # 10 kHz sampling
t = np.linspace(0, 0.1, int(fs*0.1))

# Transmitted frequency: 24 GHz (typical for automotive radar)
# We'll simulate the IF (intermediate frequency) signal
f_if = 100  # IF frequency in Hz
v_target = 10  # Target velocity: 10 m/s

# Doppler shift: f_d = 2*v*f_carrier/c
# For 24 GHz and 10 m/s: f_d ≈ 1600 Hz
c = 3e8  # Speed of light
f_carrier = 24e9
f_doppler = 2 * v_target * f_carrier / c

print(f"Doppler shift: {f_doppler:.1f} Hz")

# Simulated received signal (downconverted)
received_signal = np.cos(2*np.pi*(f_if + f_doppler)*t)

# Apply FFT to find Doppler shift
fft_doppler = np.fft.fft(received_signal)
freq_doppler = np.fft.fftfreq(len(received_signal), 1/fs)

# Find peak frequency
positive_idx = freq_doppler > 0
peak_idx = np.argmax(np.abs(fft_doppler[positive_idx]))
detected_frequency = freq_doppler[positive_idx][peak_idx]

print(f"Detected IF + Doppler frequency: {detected_frequency:.1f} Hz")
print(f"Extracted Doppler: {detected_frequency - f_if:.1f} Hz")

# Calculate velocity from Doppler
v_calculated = (detected_frequency - f_if) * c / (2 * f_carrier)
print(f"Calculated target velocity: {v_calculated:.2f} m/s")
```

### 4.6 Windowing

Applying a window reduces spectral leakage in FFT:

```python
from scipy import signal

# Common window functions
windows = {
    'Rectangular': np.ones(512),
    'Hamming': signal.hamming(512),
    'Hanning': signal.hann(512),
    'Blackman': signal.blackman(512)
}

fig, axes = plt.subplots(2, 2, figsize=(12, 10))

for ax, (name, window) in zip(axes.flat, windows.items()):
    # Time domain
    ax_time = ax
    ax_time.plot(window)
    ax_time.set_title(f'{name} Window')
    ax_time.set_xlabel('Sample')
    ax_time.set_ylabel('Amplitude')
    ax_time.grid(True)

plt.tight_layout()
plt.savefig('window_functions.png')

# Compare FFT with and without windowing
fs = 1000
t = np.linspace(0, 1, fs, endpoint=False)
# Signal at 100.5 Hz (not exactly a bin center)
signal_test = np.sin(2*np.pi*100.5*t)

fft_no_window = np.abs(np.fft.fft(signal_test))
fft_with_hamming = np.abs(np.fft.fft(signal_test * signal.hamming(len(signal_test))))

freq = np.fft.fftfreq(len(signal_test), 1/fs)
positive_idx = (freq > 0) & (freq < 200)

plt.figure(figsize=(12, 6))
plt.semilogy(freq[positive_idx], fft_no_window[positive_idx], label='No window')
plt.semilogy(freq[positive_idx], fft_with_hamming[positive_idx], label='Hamming window')
plt.xlabel('Frequency (Hz)')
plt.ylabel('Magnitude (log scale)')
plt.title('Effect of Windowing on Spectral Leakage')
plt.legend()
plt.grid(True)
plt.savefig('windowing_comparison.png')
```

---

## 5. Probability & Statistics

### 5.1 Why Statistics for Hardware Projects?

**For Both Projects:**
- Sensor measurements contain noise
- Kalman filters are based on probability theory
- Understanding measurement uncertainty
- Signal detection in noise

### 5.2 Random Variables and Distributions

#### Gaussian (Normal) Distribution

Most sensor noise follows a Gaussian distribution:

```
p(x) = (1/σ√(2π)) exp(-(x-μ)²/(2σ²))
```

Where:
- μ = mean
- σ = standard deviation
- σ² = variance

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy import stats

# Generate Gaussian random variables
mu = 0
sigma = 1
x = np.linspace(-4, 4, 1000)
pdf = stats.norm.pdf(x, mu, sigma)

# Plot PDF
plt.figure(figsize=(12, 5))

plt.subplot(1, 2, 1)
plt.plot(x, pdf, 'b-', linewidth=2)
plt.fill_between(x, pdf, alpha=0.3)
plt.axvline(x=mu, color='r', linestyle='--', label=f'Mean μ={mu}')
plt.axvline(x=mu+sigma, color='g', linestyle='--', label=f'μ+σ={mu+sigma}')
plt.axvline(x=mu-sigma, color='g', linestyle='--', label=f'μ-σ={mu-sigma}')
plt.xlabel('x')
plt.ylabel('Probability Density')
plt.title('Gaussian Distribution')
plt.legend()
plt.grid(True)

# Generate samples and histogram
samples = np.random.normal(mu, sigma, 10000)
plt.subplot(1, 2, 2)
plt.hist(samples, bins=50, density=True, alpha=0.7, label='Samples')
plt.plot(x, pdf, 'r-', linewidth=2, label='True PDF')
plt.xlabel('x')
plt.ylabel('Probability Density')
plt.title('10,000 Random Samples')
plt.legend()
plt.grid(True)

plt.tight_layout()
plt.savefig('gaussian_distribution.png')

# Key property: 68-95-99.7 rule
print("68% of data within μ ± σ")
print("95% of data within μ ± 2σ")
print("99.7% of data within μ ± 3σ")
```

#### Simulating Noisy Sensor Data

```python
# Simulate accelerometer readings with noise
true_acceleration = 9.81  # m/s² (gravity)
sensor_noise_std = 0.1    # Standard deviation of noise

num_samples = 1000
measurements = np.random.normal(true_acceleration, sensor_noise_std, num_samples)

# Statistics of measurements
measured_mean = np.mean(measurements)
measured_std = np.std(measurements)

print(f"True value: {true_acceleration} m/s²")
print(f"Measured mean: {measured_mean:.3f} m/s²")
print(f"Measured std dev: {measured_std:.3f} m/s²")
print(f"Error: {abs(measured_mean - true_acceleration):.3f} m/s²")

# Plot
plt.figure(figsize=(12, 5))

plt.subplot(1, 2, 1)
plt.plot(measurements[:100])  # First 100 samples
plt.axhline(y=true_acceleration, color='r', linestyle='--', label='True value')
plt.axhline(y=measured_mean, color='g', linestyle='--', label='Measured mean')
plt.fill_between(range(100), measured_mean-measured_std,
                 measured_mean+measured_std, alpha=0.3, label='±1σ')
plt.xlabel('Sample number')
plt.ylabel('Acceleration (m/s²)')
plt.title('Simulated Accelerometer Data')
plt.legend()
plt.grid(True)

plt.subplot(1, 2, 2)
plt.hist(measurements, bins=30, density=True, alpha=0.7)
x_hist = np.linspace(measurements.min(), measurements.max(), 100)
plt.plot(x_hist, stats.norm.pdf(x_hist, true_acceleration, sensor_noise_std),
         'r-', linewidth=2, label='True distribution')
plt.xlabel('Acceleration (m/s²)')
plt.ylabel('Probability Density')
plt.title('Distribution of Measurements')
plt.legend()
plt.grid(True)

plt.tight_layout()
plt.savefig('noisy_sensor_data.png')
```

### 5.3 Filtering Noisy Data

#### Simple Moving Average

```python
def moving_average(data, window_size):
    """Simple moving average filter"""
    return np.convolve(data, np.ones(window_size)/window_size, mode='valid')

# Apply moving average to noisy data
window_sizes = [5, 10, 20, 50]

plt.figure(figsize=(12, 8))
plt.plot(measurements[:200], alpha=0.3, label='Raw data')
plt.axhline(y=true_acceleration, color='r', linestyle='--', linewidth=2, label='True value')

for window in window_sizes:
    filtered = moving_average(measurements[:200], window)
    plt.plot(range(window-1, 200), filtered, label=f'MA window={window}')

plt.xlabel('Sample')
plt.ylabel('Acceleration (m/s²)')
plt.title('Moving Average Filtering')
plt.legend()
plt.grid(True)
plt.savefig('moving_average_filter.png')
```

**Trade-off:** Larger window → smoother but more lag

#### Exponential Moving Average (Better for real-time)

```python
def exponential_moving_average(data, alpha):
    """
    Exponential moving average
    alpha: smoothing factor (0 < alpha < 1)
    Smaller alpha = more smoothing
    """
    ema = np.zeros_like(data)
    ema[0] = data[0]
    for i in range(1, len(data)):
        ema[i] = alpha * data[i] + (1 - alpha) * ema[i-1]
    return ema

# Try different alpha values
alphas = [0.1, 0.2, 0.5, 0.8]

plt.figure(figsize=(12, 8))
plt.plot(measurements[:200], alpha=0.3, label='Raw data')
plt.axhline(y=true_acceleration, color='r', linestyle='--', linewidth=2, label='True value')

for alpha in alphas:
    filtered = exponential_moving_average(measurements[:200], alpha)
    plt.plot(filtered, label=f'EMA α={alpha}')

plt.xlabel('Sample')
plt.ylabel('Acceleration (m/s²)')
plt.title('Exponential Moving Average Filtering')
plt.legend()
plt.grid(True)
plt.savefig('ema_filter.png')
```

This is commonly used in flight controllers for gyro/accel filtering!

### 5.4 Covariance and Correlation

#### Covariance

Measures how two variables vary together:

```
Cov(X,Y) = E[(X-μₓ)(Y-μᵧ)]
```

```python
# Example: Accelerometer X and Y axes might be correlated due to vibration
np.random.seed(42)
n = 1000

# Create correlated data
accel_x = np.random.normal(0, 1, n)
accel_y = 0.7 * accel_x + np.random.normal(0, 0.5, n)  # Correlated with X

# Calculate covariance
cov_matrix = np.cov(accel_x, accel_y)
print(f"Covariance matrix:\n{cov_matrix}")
print(f"Cov(X,Y) = {cov_matrix[0,1]:.3f}")

# Correlation coefficient (normalized covariance)
corr_coefficient = np.corrcoef(accel_x, accel_y)[0,1]
print(f"Correlation coefficient: {corr_coefficient:.3f}")

# Visualize
plt.figure(figsize=(12, 5))

plt.subplot(1, 2, 1)
plt.scatter(accel_x, accel_y, alpha=0.5)
plt.xlabel('Accel X')
plt.ylabel('Accel Y')
plt.title(f'Correlated Data (ρ={corr_coefficient:.2f})')
plt.grid(True)
plt.axis('equal')

# Compare with uncorrelated data
accel_y_uncorr = np.random.normal(0, 1, n)
corr_uncorr = np.corrcoef(accel_x, accel_y_uncorr)[0,1]

plt.subplot(1, 2, 2)
plt.scatter(accel_x, accel_y_uncorr, alpha=0.5)
plt.xlabel('Accel X')
plt.ylabel('Accel Y (uncorrelated)')
plt.title(f'Uncorrelated Data (ρ={corr_uncorr:.2f})')
plt.grid(True)
plt.axis('equal')

plt.tight_layout()
plt.savefig('covariance_example.png')
```

**Used in Kalman Filters:** Covariance matrices describe uncertainty!

---

## 6. Complex Numbers & Phasors

### 6.1 Complex Number Basics

```
z = a + jb = r∠θ = re^(jθ)
```

Where:
- a = real part
- b = imaginary part
- r = magnitude = √(a² + b²)
- θ = phase = arctan(b/a)
- j = √(-1)

**Euler's Formula (Most important!):**
```
e^(jθ) = cos(θ) + j·sin(θ)
```

```python
import numpy as np
import matplotlib.pyplot as plt

# Define complex numbers
z1 = 3 + 4j
z2 = 2 - 1j

print(f"z1 = {z1}")
print(f"z2 = {z2}")

# Operations
print(f"z1 + z2 = {z1 + z2}")
print(f"z1 * z2 = {z1 * z2}")
print(f"z1 / z2 = {z1 / z2}")

# Magnitude and phase
r1 = np.abs(z1)
theta1 = np.angle(z1)
print(f"|z1| = {r1:.3f}")
print(f"∠z1 = {np.degrees(theta1):.1f}°")

# Visualize on complex plane
fig, ax = plt.subplots(figsize=(8, 8))
ax.quiver(0, 0, z1.real, z1.imag, angles='xy', scale_units='xy', scale=1,
          color='b', width=0.006, label='z1')
ax.quiver(0, 0, z2.real, z2.imag, angles='xy', scale_units='xy', scale=1,
          color='r', width=0.006, label='z2')
ax.quiver(0, 0, (z1+z2).real, (z1+z2).imag, angles='xy', scale_units='xy', scale=1,
          color='g', width=0.006, label='z1+z2')

ax.set_xlim(-2, 6)
ax.set_ylim(-2, 6)
ax.set_xlabel('Real')
ax.set_ylabel('Imaginary')
ax.set_title('Complex Number Arithmetic')
ax.grid(True)
ax.axhline(y=0, color='k', linewidth=0.5)
ax.axvline(x=0, color='k', linewidth=0.5)
ax.legend()
ax.set_aspect('equal')
plt.savefig('complex_plane.png')
```

### 6.2 Phasors for AC Circuits and Signals

A phasor is a complex number representing sinusoidal signals:

```
v(t) = Vₘcos(ωt + φ)  ↔  V = Vₘ∠φ
```

```python
# AC circuit analysis with phasors
# RLC series circuit

# Circuit parameters
R = 100  # Ohms
L = 0.01  # Henries
C = 1e-6  # Farads
f = 1000  # Frequency (Hz)
omega = 2 * np.pi * f

# Impedances (complex!)
Z_R = R
Z_L = 1j * omega * L
Z_C = 1 / (1j * omega * C)

Z_total = Z_R + Z_L + Z_C

print(f"Z_R = {Z_R:.2f} Ω")
print(f"Z_L = {Z_L:.2f} Ω")
print(f"Z_C = {Z_C:.2f} Ω")
print(f"Z_total = {Z_total:.2f} Ω")
print(f"|Z_total| = {np.abs(Z_total):.2f} Ω")
print(f"∠Z_total = {np.degrees(np.angle(Z_total)):.1f}°")

# Applied voltage
V_source = 10 + 0j  # 10V at 0° phase

# Current (Ohm's law with phasors)
I = V_source / Z_total
print(f"\nCurrent: {I:.4f} A")
print(f"|I| = {np.abs(I):.4f} A")
print(f"∠I = {np.degrees(np.angle(I)):.1f}°")

# Frequency response
frequencies = np.logspace(1, 5, 500)  # 10 Hz to 100 kHz
Z_mag = np.zeros_like(frequencies)

for i, f in enumerate(frequencies):
    omega = 2 * np.pi * f
    Z = R + 1j*omega*L + 1/(1j*omega*C)
    Z_mag[i] = np.abs(Z)

# Resonant frequency
f_resonant = 1 / (2 * np.pi * np.sqrt(L * C))

plt.figure(figsize=(10, 6))
plt.semilogx(frequencies, Z_mag)
plt.axvline(x=f_resonant, color='r', linestyle='--',
            label=f'Resonance at {f_resonant:.1f} Hz')
plt.xlabel('Frequency (Hz)')
plt.ylabel('|Z| (Ω)')
plt.title('RLC Circuit Impedance vs Frequency')
plt.grid(True, which='both')
plt.legend()
plt.savefig('rlc_frequency_response.png')
```

### 6.3 Application: Radar Signal Processing

```python
# FMCW Radar: Mix transmitted and received signals

# Transmitted signal (upchirp)
t = np.linspace(0, 1e-3, 10000)  # 1 ms
f_start = 24e9  # 24 GHz
f_stop = 24.2e9  # 24.2 GHz
chirp_rate = (f_stop - f_start) / t[-1]

# For simulation, work with intermediate frequency
f_tx = 1000 + 500000 * t  # Linear chirp

# Received signal (delayed by round-trip time)
target_distance = 10  # meters
c = 3e8
delay = 2 * target_distance / c
f_rx = 1000 + 500000 * (t - delay)

# Complex representation
tx_complex = np.exp(1j * 2 * np.pi * np.cumsum(f_tx) * (t[1]-t[0]))
rx_complex = np.exp(1j * 2 * np.pi * np.cumsum(f_rx) * (t[1]-t[0]))

# Mixing (multiplication of complex signals)
if_signal = tx_complex * np.conj(rx_complex)

# Beat frequency (constant for FMCW)
beat_freq = chirp_rate * delay

print(f"Target distance: {target_distance} m")
print(f"Round-trip delay: {delay*1e9:.3f} ns")
print(f"Beat frequency: {beat_freq/1e3:.1f} kHz")

# FFT to extract beat frequency
fft_result = np.fft.fft(if_signal)
freq = np.fft.fftfreq(len(if_signal), t[1]-t[0])

positive_idx = freq > 0
plt.figure(figsize=(12, 5))
plt.plot(freq[positive_idx]/1e3, np.abs(fft_result[positive_idx]))
plt.xlabel('Frequency (kHz)')
plt.ylabel('Magnitude')
plt.title('FMCW Radar Beat Frequency')
plt.xlim(0, 20)
plt.grid(True)
plt.axvline(x=beat_freq/1e3, color='r', linestyle='--',
            label=f'Beat freq = {beat_freq/1e3:.1f} kHz')
plt.legend()
plt.savefig('fmcw_beat_frequency.png')

# Calculate range from beat frequency
detected_beat_freq = freq[positive_idx][np.argmax(np.abs(fft_result[positive_idx]))]
calculated_range = detected_beat_freq * c / (2 * chirp_rate)
print(f"Calculated range: {calculated_range:.2f} m")
```

---

## 7. Numerical Methods

### 7.1 Root Finding

#### Newton-Raphson Method

Find x where f(x) = 0:

```
x_{n+1} = x_n - f(x_n)/f'(x_n)
```

```python
def newton_raphson(f, df, x0, tol=1e-6, max_iter=100):
    """
    Find root of f(x) = 0 using Newton-Raphson method
    f: function
    df: derivative of function
    x0: initial guess
    """
    x = x0
    for i in range(max_iter):
        fx = f(x)
        if abs(fx) < tol:
            print(f"Converged in {i} iterations")
            return x
        x = x - fx / df(x)
    print("Warning: Did not converge")
    return x

# Example: Find √2 by solving x² - 2 = 0
f = lambda x: x**2 - 2
df = lambda x: 2*x

root = newton_raphson(f, df, x0=1.0)
print(f"√2 ≈ {root:.10f}")
print(f"Actual: {np.sqrt(2):.10f}")
print(f"Error: {abs(root - np.sqrt(2)):.2e}")
```

### 7.2 Numerical Integration

#### Trapezoidal Rule

```python
def trapezoidal_integration(f, a, b, n):
    """
    Integrate f from a to b using n trapezoids
    """
    x = np.linspace(a, b, n+1)
    y = f(x)
    h = (b - a) / n
    integral = h * (0.5*y[0] + np.sum(y[1:-1]) + 0.5*y[-1])
    return integral

# Example: ∫[0 to π] sin(x) dx = 2
f = np.sin
integral_numerical = trapezoidal_integration(f, 0, np.pi, 100)
integral_exact = 2.0

print(f"Numerical integral: {integral_numerical:.6f}")
print(f"Exact integral: {integral_exact:.6f}")
print(f"Error: {abs(integral_numerical - integral_exact):.2e}")
```

#### Simpson's Rule (More accurate)

```python
from scipy import integrate

# Using scipy's quad function (adaptive integration)
integral_scipy, error = integrate.quad(f, 0, np.pi)
print(f"Scipy integral: {integral_scipy:.10f}")
print(f"Estimated error: {error:.2e}")
```

### 7.3 Interpolation

```python
from scipy import interpolate

# Example: Calibration table for a sensor
# Voltage -> Temperature mapping
voltage_table = np.array([0.5, 1.0, 1.5, 2.0, 2.5, 3.0])
temp_table = np.array([0, 20, 35, 48, 59, 68])

# Create interpolation function
f_interp_linear = interpolate.interp1d(voltage_table, temp_table, kind='linear')
f_interp_cubic = interpolate.interp1d(voltage_table, temp_table, kind='cubic')

# Interpolate at new points
voltage_new = np.linspace(0.5, 3.0, 100)
temp_linear = f_interp_linear(voltage_new)
temp_cubic = f_interp_cubic(voltage_new)

plt.figure(figsize=(10, 6))
plt.plot(voltage_table, temp_table, 'ko', markersize=8, label='Calibration points')
plt.plot(voltage_new, temp_linear, 'b-', label='Linear interpolation')
plt.plot(voltage_new, temp_cubic, 'r--', label='Cubic interpolation')
plt.xlabel('Voltage (V)')
plt.ylabel('Temperature (°C)')
plt.title('Sensor Calibration Curve')
plt.legend()
plt.grid(True)
plt.savefig('interpolation_example.png')
```

---

## Summary and Next Steps

You now have the mathematical foundation for:

✅ **Calculus** - Understanding rates of change and accumulation
✅ **Linear Algebra** - Transformations, rotations, and state-space models
✅ **Differential Equations** - Modeling physical systems
✅ **Fourier Analysis** - Signal processing and frequency analysis
✅ **Statistics** - Handling noisy sensor data
✅ **Complex Numbers** - AC circuits and signal representation
✅ **Numerical Methods** - Practical computation

**Practice Problems:**

1. Derive the equation of motion for a quadcopter in one dimension
2. Implement a complementary filter for IMU data (uses simple calculus)
3. Write an FFT-based Doppler velocity calculator
4. Create a Kalman filter for 1D position estimation (uses linear algebra and statistics)

**Next:** Move on to [Physics Foundations](./02-physics-foundations.md) to see how these mathematical concepts apply to real physical systems.

---

## Additional Resources

**Books:**
- "Calculus" by James Stewart
- "Linear Algebra and Its Applications" by Gilbert Strang
- "The Scientist and Engineer's Guide to Digital Signal Processing" (free online)

**Online:**
- 3Blue1Brown YouTube channel (excellent visual explanations)
- MIT OpenCourseWare - 18.01, 18.02, 18.03 (Calculus)
- Khan Academy - Linear Algebra

**Practice:**
- Work through exercises in each section
- Implement algorithms in Python
- Apply to real sensor data when you build your projects

---

*Last Updated: November 2025*
