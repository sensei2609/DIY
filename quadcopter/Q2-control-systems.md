# Q2: Control Systems Theory

> **Deep dive into PID control, cascaded loops, and state estimation for quadcopter flight**

---

## Table of Contents

1. [Control System Fundamentals](#1-control-system-fundamentals)
2. [PID Control Theory](#2-pid-control-theory)
3. [PID Tuning Methods](#3-pid-tuning-methods)
4. [Cascaded Control Loops](#4-cascaded-control-loops)
5. [Complementary Filter](#5-complementary-filter)
6. [Kalman Filter](#6-kalman-filter)
7. [Advanced Control](#7-advanced-control)

---

## 1. Control System Fundamentals

### 1.1 Open-Loop vs Closed-Loop

**Open-Loop (No Feedback):**
```
Input → Controller → Actuator → Output
```
- Simple
- No correction for errors
- Example: Toaster (just timer, no temperature feedback)

**Closed-Loop (Feedback):**
```
Reference → (+) → Controller → Actuator → Output
              ↑                              ↓
              └──────── Sensor ─────────────┘
           (-)
```
- Measures output
- Corrects errors automatically
- Example: Thermostat, quadcopter flight

```python
import numpy as np
import matplotlib.pyplot as plt

def compare_open_closed_loop():
    """Demonstrate open-loop vs closed-loop control"""

    # System parameters
    dt = 0.01
    time = np.arange(0, 10, dt)

    # Desired trajectory
    setpoint = np.ones_like(time) * 45  # 45° roll angle

    # Disturbance at t=5s
    disturbance = np.zeros_like(time)
    disturbance[time > 5] = -10  # Wind gust

    # Open-loop
    output_openloop = np.zeros_like(time)
    control_input = 1.0  # Fixed control input

    for i in range(1, len(time)):
        # No feedback - just applies constant input
        output_openloop[i] = output_openloop[i-1] + control_input * dt
        # Apply disturbance
        output_openloop[i] += disturbance[i]

    # Closed-loop (simple proportional)
    output_closedloop = np.zeros_like(time)
    Kp = 2.0

    for i in range(1, len(time)):
        # Calculate error and control
        error = setpoint[i] - output_closedloop[i-1]
        control = Kp * error

        # Update output
        output_closedloop[i] = output_closedloop[i-1] + control * dt

        # Apply disturbance
        output_closedloop[i] += disturbance[i]

    # Plot
    fig, axes = plt.subplots(2, 1, figsize=(12, 10))

    axes[0].plot(time, setpoint, 'k--', linewidth=2, label='Setpoint')
    axes[0].plot(time, output_openloop, 'r-', label='Open-loop')
    axes[0].axvline(x=5, color='gray', linestyle=':', alpha=0.5)
    axes[0].text(5.1, 30, 'Disturbance', fontsize=10)
    axes[0].set_ylabel('Angle (degrees)')
    axes[0].set_title('Open-Loop Control (No Feedback)')
    axes[0].legend()
    axes[0].grid(True)
    axes[0].set_ylim([0, 80])

    axes[1].plot(time, setpoint, 'k--', linewidth=2, label='Setpoint')
    axes[1].plot(time, output_closedloop, 'g-', label='Closed-loop')
    axes[1].axvline(x=5, color='gray', linestyle=':', alpha=0.5)
    axes[1].text(5.1, 30, 'Disturbance\n(rejected!)', fontsize=10)
    axes[1].set_xlabel('Time (s)')
    axes[1].set_ylabel('Angle (degrees)')
    axes[1].set_title('Closed-Loop Control (With Feedback)')
    axes[1].legend()
    axes[1].grid(True)
    axes[1].set_ylim([0, 80])

    plt.tight_layout()
    plt.savefig('open_vs_closed_loop.png')

    print("Open-Loop vs Closed-Loop Comparison:")
    print("-" * 50)
    print("Open-Loop:")
    print("  + Simple, no sensors needed")
    print("  - No disturbance rejection")
    print("  - Accumulates errors")
    print("\nClosed-Loop:")
    print("  + Automatically corrects errors")
    print("  + Rejects disturbances")
    print("  - Requires sensors")
    print("  - Can be unstable if poorly tuned")

compare_open_closed_loop()
```

### 1.2 System Response Characteristics

**Key Metrics:**

```
Rise Time (tr): Time to go from 10% to 90% of final value
Settling Time (ts): Time to stay within ±2% of final value
Overshoot: Peak value - final value (percentage)
Steady-State Error: Final error after settling
```

```python
def analyze_step_response():
    """Analyze step response characteristics"""

    from scipy import signal as sig

    # Second-order system: typical for mechanical systems
    # Transfer function: ωn² / (s² + 2ζωn·s + ωn²)

    wn = 10  # Natural frequency (rad/s)
    zeta_values = [0.3, 0.5, 0.7, 1.0, 1.5]  # Damping ratios

    fig, axes = plt.subplots(2, 2, figsize=(12, 10))
    axes = axes.flatten()

    colors = ['red', 'orange', 'green', 'blue', 'purple']

    for ax_idx, zeta in enumerate(zeta_values):
        # Create transfer function
        num = [wn**2]
        den = [1, 2*zeta*wn, wn**2]
        sys = sig.TransferFunction(num, den)

        # Step response
        t, y = sig.step(sys, T=np.linspace(0, 2, 1000))

        # Plot
        if ax_idx < 4:
            axes[ax_idx].plot(t, y, color=colors[ax_idx], linewidth=2)
            axes[ax_idx].axhline(y=1, color='k', linestyle='--', alpha=0.3)
            axes[ax_idx].set_title(f'ζ = {zeta} ({["Underdamped", "Underdamped", "Underdamped", "Critically Damped", "Overdamped"][ax_idx]})')
            axes[ax_idx].set_xlabel('Time (s)')
            axes[ax_idx].set_ylabel('Response')
            axes[ax_idx].grid(True)
            axes[ax_idx].set_ylim([0, 1.5])

            # Calculate metrics
            final_value = y[-1]
            peak_value = np.max(y)
            overshoot = (peak_value - final_value) / final_value * 100

            # Rise time (10% to 90%)
            idx_10 = np.where(y >= 0.1 * final_value)[0][0]
            idx_90 = np.where(y >= 0.9 * final_value)[0][0]
            rise_time = t[idx_90] - t[idx_10]

            # Settling time (±2%)
            tolerance = 0.02
            settled = np.abs(y - final_value) <= tolerance * final_value
            if np.any(settled):
                settling_time = t[np.where(settled)[0][0]]
            else:
                settling_time = t[-1]

            axes[ax_idx].text(0.5, 1.3,
                            f'Overshoot: {overshoot:.1f}%\n'
                            f'Rise time: {rise_time:.3f}s\n'
                            f'Settling: {settling_time:.3f}s',
                            fontsize=8)

    plt.tight_layout()
    plt.savefig('system_response_characteristics.png')

    print("\nDamping Ratio Effects:")
    print("-" * 50)
    print("ζ < 1: Underdamped - Oscillatory, fast but overshoots")
    print("ζ = 1: Critically damped - Fastest without overshoot")
    print("ζ > 1: Overdamped - Slow, no overshoot")
    print("\nFor quadcopters: Typically want ζ ≈ 0.7-1.0")

analyze_step_response()
```

---

## 2. PID Control Theory

### 2.1 The PID Equation

```
u(t) = Kp·e(t) + Ki·∫e(t)dt + Kd·de(t)/dt

Where:
u(t) = control output
e(t) = error = setpoint - measured_value
Kp = proportional gain
Ki = integral gain
Kd = derivative gain
```

**Discrete (Digital) Form:**
```
u[k] = Kp·e[k] + Ki·Σe[k]·dt + Kd·(e[k] - e[k-1])/dt
```

### 2.2 Each Term Explained

**Proportional (P):**
- Output proportional to current error
- Fast response
- Steady-state error remains
- Higher Kp → faster, but can overshoot

**Integral (I):**
- Accumulates past errors
- Eliminates steady-state error
- Too high → oscillation, windup
- Anti-windup needed

**Derivative (D):**
- Predicts future error (rate of change)
- Dampens oscillations
- Reduces overshoot
- Sensitive to noise

```python
class PID:
    """Complete PID controller with anti-windup and filtering"""

    def __init__(self, kp, ki, kd, dt, output_limits=None, integral_limits=None):
        self.kp = kp
        self.ki = ki
        self.kd = kd
        self.dt = dt

        self.output_limits = output_limits or (-np.inf, np.inf)
        self.integral_limits = integral_limits or (-np.inf, np.inf)

        self.integral = 0.0
        self.previous_error = 0.0
        self.previous_output = 0.0

        # Low-pass filter for derivative (reduce noise sensitivity)
        self.derivative_lpf_alpha = 0.1
        self.filtered_derivative = 0.0

    def compute(self, setpoint, measured_value):
        """Compute PID output"""

        # Error
        error = setpoint - measured_value

        # Proportional term
        p_term = self.kp * error

        # Integral term with anti-windup
        self.integral += error * self.dt

        # Clamp integral
        self.integral = np.clip(self.integral,
                                self.integral_limits[0],
                                self.integral_limits[1])

        i_term = self.ki * self.integral

        # Derivative term with filtering
        derivative = (error - self.previous_error) / self.dt

        # Low-pass filter on derivative
        self.filtered_derivative = (self.derivative_lpf_alpha * derivative +
                                   (1 - self.derivative_lpf_alpha) * self.filtered_derivative)

        d_term = self.kd * self.filtered_derivative

        # Total output
        output = p_term + i_term + d_term

        # Clamp output
        output = np.clip(output, self.output_limits[0], self.output_limits[1])

        # Anti-windup: If output is saturated, don't integrate
        if output != p_term + i_term + d_term:
            # Output was clamped - back-calculate integral
            self.integral -= error * self.dt

        # Store for next iteration
        self.previous_error = error
        self.previous_output = output

        return output

    def reset(self):
        """Reset controller state"""
        self.integral = 0.0
        self.previous_error = 0.0
        self.filtered_derivative = 0.0


def demonstrate_pid_terms():
    """Show effect of each PID term individually"""

    dt = 0.01
    time = np.arange(0, 10, dt)
    setpoint = 45.0  # degrees

    # Simple system model (integrator + inertia)
    def plant_model(control_input, state, dt):
        """Simple second-order system"""
        # state = [position, velocity]
        position, velocity = state

        # Dynamics: acceleration = control_input - damping
        damping = 0.5
        acceleration = control_input - damping * velocity

        # Update
        velocity += acceleration * dt
        position += velocity * dt

        return np.array([position, velocity])

    # Test different controller configurations
    configs = [
        ("P only", 0.5, 0, 0),
        ("PI", 0.5, 0.1, 0),
        ("PD", 0.5, 0, 0.1),
        ("PID", 0.5, 0.1, 0.1),
    ]

    fig, axes = plt.subplots(2, 2, figsize=(12, 10))
    axes = axes.flatten()

    for ax, (name, kp, ki, kd) in zip(axes, configs):
        pid = PID(kp, ki, kd, dt, output_limits=(-10, 10))

        state = np.array([0.0, 0.0])  # [position, velocity]
        output = []

        for t in time:
            # Measure current position
            measured = state[0]

            # Compute control
            control = pid.compute(setpoint, measured)

            # Apply to plant
            state = plant_model(control, state, dt)

            output.append(state[0])

        # Plot
        ax.plot(time, setpoint * np.ones_like(time), 'k--', linewidth=2, label='Setpoint')
        ax.plot(time, output, linewidth=2, label='Output')
        ax.set_xlabel('Time (s)')
        ax.set_ylabel('Angle (degrees)')
        ax.set_title(f'{name}: Kp={kp}, Ki={ki}, Kd={kd}')
        ax.legend()
        ax.grid(True)
        ax.set_ylim([0, 60])

    plt.tight_layout()
    plt.savefig('pid_terms_comparison.png')

    print("PID Term Effects:")
    print("-" * 50)
    print("P only:")
    print("  Fast response, but steady-state error")
    print("  Oscillates if Kp too high")
    print("\nPI:")
    print("  Eliminates steady-state error")
    print("  Can oscillate, slower response")
    print("\nPD:")
    print("  Smooth response, reduced overshoot")
    print("  Still has steady-state error")
    print("\nPID:")
    print("  Fast, smooth, no steady-state error")
    print("  Requires proper tuning")

demonstrate_pid_terms()
```

### 2.3 Practical PID Implementation for Quadcopter

```cpp
// pid_controller.h
#ifndef PID_CONTROLLER_H
#define PID_CONTROLLER_H

class PIDController {
private:
    // Gains
    float kp, ki, kd;

    // State
    float integral;
    float previous_error;
    float previous_derivative;

    // Limits
    float output_min, output_max;
    float integral_min, integral_max;

    // Derivative filtering
    float derivative_lpf_alpha;

    // Time
    float dt;

public:
    PIDController(float p, float i, float d, float sample_time)
        : kp(p), ki(i), kd(d), dt(sample_time)
    {
        reset();

        // Default limits
        output_min = -1000.0f;
        output_max = 1000.0f;
        integral_min = -100.0f;
        integral_max = 100.0f;

        // Derivative low-pass filter (20Hz cutoff for 1kHz loop)
        derivative_lpf_alpha = 0.1f;
    }

    void setOutputLimits(float min_val, float max_val) {
        output_min = min_val;
        output_max = max_val;
    }

    void setIntegralLimits(float min_val, float max_val) {
        integral_min = min_val;
        integral_max = max_val;
    }

    void setGains(float p, float i, float d) {
        kp = p;
        ki = i;
        kd = d;
    }

    float compute(float setpoint, float measured_value) {
        // Calculate error
        float error = setpoint - measured_value;

        // Proportional term
        float p_term = kp * error;

        // Integral term
        integral += error * dt;

        // Anti-windup: clamp integral
        if (integral > integral_max) integral = integral_max;
        if (integral < integral_min) integral = integral_min;

        float i_term = ki * integral;

        // Derivative term (with low-pass filter)
        float derivative = (error - previous_error) / dt;

        // Filter derivative to reduce noise
        float filtered_derivative = derivative_lpf_alpha * derivative +
                                   (1.0f - derivative_lpf_alpha) * previous_derivative;

        float d_term = kd * filtered_derivative;

        // Total output
        float output = p_term + i_term + d_term;

        // Clamp output
        if (output > output_max) output = output_max;
        if (output < output_min) output = output_min;

        // Anti-windup: if output saturated, back off integral
        if (output == output_max || output == output_min) {
            integral -= error * dt;
        }

        // Save for next iteration
        previous_error = error;
        previous_derivative = filtered_derivative;

        return output;
    }

    void reset() {
        integral = 0.0f;
        previous_error = 0.0f;
        previous_derivative = 0.0f;
    }

    float getP() { return kp * previous_error; }
    float getI() { return ki * integral; }
    float getD() { return kd * previous_derivative; }
};

#endif
```

---

## 3. PID Tuning Methods

### 3.1 Manual Tuning (Most Common for Quads)

**Step-by-step procedure:**

```
1. Set all gains to zero: Kp=0, Ki=0, Kd=0

2. Increase Kp until oscillation starts
   - Start with small value (0.1)
   - Increase gradually
   - Stop when sustained oscillation occurs
   - Reduce Kp by 50%

3. Increase Kd to dampen oscillations
   - Start with Kd = Kp/10
   - Increase until overshoot acceptable
   - Too much Kd = sluggish, noise amplification

4. Add Ki to remove steady-state error
   - Start with Ki = Kp/10
   - Increase slowly
   - Watch for windup and oscillation
   - Keep Ki small!

5. Fine-tune
   - Fly and observe
   - Adjust based on feel
   - P: responsiveness
   - I: drift correction
   - D: smoothness
```

```python
def tuning_simulator():
    """Interactive PID tuning simulator"""

    dt = 0.01
    time = np.arange(0, 5, dt)
    setpoint = 45.0

    # Tuning sequence
    tuning_steps = [
        ("Step 1: Kp only (too low)", 0.2, 0, 0),
        ("Step 2: Kp only (too high)", 2.0, 0, 0),
        ("Step 3: Kp good", 0.8, 0, 0),
        ("Step 4: Kp + Kd", 0.8, 0, 0.2),
        ("Step 5: Full PID", 0.8, 0.15, 0.2),
    ]

    fig, axes = plt.subplots(3, 2, figsize=(14, 12))
    axes = axes.flatten()

    def plant(control, state, dt):
        pos, vel = state
        acc = control - 0.3*vel
        vel += acc * dt
        pos += vel * dt
        return np.array([pos, vel])

    for idx, (title, kp, ki, kd) in enumerate(tuning_steps):
        pid = PID(kp, ki, kd, dt, output_limits=(-15, 15))
        state = np.array([0.0, 0.0])
        trajectory = []

        for t in time:
            control = pid.compute(setpoint, state[0])
            state = plant(control, state, dt)
            trajectory.append(state[0])

        axes[idx].plot(time, setpoint * np.ones_like(time), 'k--', linewidth=2, label='Setpoint')
        axes[idx].plot(time, trajectory, linewidth=2)
        axes[idx].set_title(title)
        axes[idx].set_xlabel('Time (s)')
        axes[idx].set_ylabel('Angle (deg)')
        axes[idx].legend()
        axes[idx].grid(True)
        axes[idx].set_ylim([0, 70])

    # Hide last subplot
    axes[5].axis('off')

    plt.tight_layout()
    plt.savefig('pid_tuning_steps.png')

tuning_simulator()
```

### 3.2 Ziegler-Nichols Method

**Ultimate Gain Method:**

```
1. Set Ki=0, Kd=0
2. Increase Kp until sustained oscillation
3. Record:
   - Ku = ultimate gain (Kp at oscillation)
   - Tu = oscillation period

Then:
   Kp = 0.6 × Ku
   Ki = 2 × Kp / Tu
   Kd = Kp × Tu / 8
```

```python
def ziegler_nichols_calculator():
    """Calculate PID gains using Ziegler-Nichols"""

    print("Ziegler-Nichols PID Tuning Calculator")
    print("=" * 50)

    # Example measurements
    Ku = 2.0  # Ultimate gain
    Tu = 0.5  # Oscillation period (seconds)

    print(f"Measured values:")
    print(f"  Ultimate gain (Ku): {Ku}")
    print(f"  Oscillation period (Tu): {Tu}s")

    # Calculate gains
    Kp = 0.6 * Ku
    Ki = 2 * Kp / Tu
    Kd = Kp * Tu / 8

    print(f"\nRecommended PID gains:")
    print(f"  Kp = {Kp:.3f}")
    print(f"  Ki = {Ki:.3f}")
    print(f"  Kd = {Kd:.3f}")

    print(f"\nNote: These are starting points!")
    print(f"Fine-tune based on actual performance.")

    # Alternative tunings for different response
    print(f"\nAlternative tunings:")
    print(f"\nP-only (no overshoot):")
    print(f"  Kp = {0.5 * Ku:.3f}")

    print(f"\nPI (some overshoot):")
    print(f"  Kp = {0.45 * Ku:.3f}")
    print(f"  Ki = {0.54 * Ku / Tu:.3f}")

    print(f"\nPID (minimal overshoot):")
    print(f"  Kp = {Kp:.3f}")
    print(f"  Ki = {Ki:.3f}")
    print(f"  Kd = {Kd:.3f}")

ziegler_nichols_calculator()
```

### 3.3 Tuning Tips for Quadcopters

**Roll/Pitch PID (Rate Loop):**
```
Start with: Kp = 1.0-2.0, Ki = 0.05-0.15, Kd = 0.02-0.08

Symptoms:
- Oscillates at low throttle: Reduce P
- Oscillates at high throttle: Reduce D
- Drifts slowly: Increase I
- Wobbles: Increase D
- Sluggish: Increase P
```

**Yaw PID:**
```
Start with: Kp = 2.0-4.0, Ki = 0.1-0.3, Kd = 0.0

Yaw usually needs:
- Higher P (less mechanical resistance)
- More I (to hold heading)
- Little or no D (yaw is already damped)
```

**Altitude PID:**
```
Start with: Kp = 1.0-3.0, Ki = 0.5-1.0, Kd = 0.5-1.5

- Bounces: Too much D
- Drops slowly: Increase P and I
- Oscillates: Reduce P
```

---

## 4. Cascaded Control Loops

### 4.1 Why Cascade?

Quadcopters use **nested control loops**:

```
Position Loop (Slow, 10-50 Hz)
    ↓ commands velocity
Velocity Loop (Medium, 50-100 Hz)
    ↓ commands angle
Angle Loop (Fast, 100-500 Hz)
    ↓ commands rate
Rate Loop (Fastest, 500-2000 Hz)
    ↓ commands motors
Motors
```

**Benefits:**
- Modularity (tune each loop separately)
- Stability (fast inner loop stabilizes before outer acts)
- Flexibility (can disable outer loops for different modes)

### 4.2 Rate Control Loop (Innermost)

**Input:** Desired angular rate (°/s)
**Sensor:** Gyroscope
**Output:** Motor commands

```cpp
// Rate PID loop (runs at 1000 Hz)
void rateControl() {
    // Read gyro (°/s)
    float roll_rate = gyro.x;
    float pitch_rate = gyro.y;
    float yaw_rate = gyro.z;

    // Desired rates from angle loop (or pilot in acro mode)
    float roll_rate_desired = ...;
    float pitch_rate_desired = ...;
    float yaw_rate_desired = ...;

    // PID control
    float roll_output = roll_rate_pid.compute(roll_rate_desired, roll_rate);
    float pitch_output = pitch_rate_pid.compute(pitch_rate_desired, pitch_rate);
    float yaw_output = yaw_rate_pid.compute(yaw_rate_desired, yaw_rate);

    // Mix to motors
    motorMixing(throttle, roll_output, pitch_output, yaw_output);
}
```

### 4.3 Angle Control Loop (Middle)

**Input:** Desired angle (°)
**Sensor:** Accelerometer + Gyro (fused)
**Output:** Desired rate (commands rate loop)

```cpp
// Angle PID loop (runs at 250 Hz - slower than rate)
void angleControl() {
    // Read attitude from sensor fusion
    float roll_angle = attitude.roll;
    float pitch_angle = attitude.pitch;

    // Desired angles from velocity loop (or pilot in angle mode)
    float roll_angle_desired = ...;
    float pitch_angle_desired = ...;

    // PID control → outputs desired rates
    float roll_rate_desired = roll_angle_pid.compute(roll_angle_desired, roll_angle);
    float pitch_rate_desired = pitch_angle_pid.compute(pitch_angle_desired, pitch_angle);

    // Limit rate commands
    roll_rate_desired = constrain(roll_rate_desired, -200, 200);  // ±200°/s max
    pitch_rate_desired = constrain(pitch_rate_desired, -200, 200);

    // Pass to rate loop
    // (rate loop runs faster and will execute multiple times)
}
```

### 4.4 Complete Cascade Example

```python
def cascaded_control_simulation():
    """Simulate cascaded angle + rate control"""

    dt = 0.001  # 1kHz rate loop
    time = np.arange(0, 5, dt)

    # Setpoint: 30° roll angle
    angle_setpoint = 30.0

    # PIDs
    # Rate loop (inner): fast, responsive
    rate_pid = PID(kp=1.5, ki=0.1, kd=0.05, dt=dt, output_limits=(-10, 10))

    # Angle loop (outer): slower, smoother
    angle_pid = PID(kp=3.0, ki=0.0, kd=0.0, dt=dt*4, output_limits=(-200, 200))  # Runs at 250Hz

    # System state
    angle = 0.0
    rate = 0.0

    # Storage
    angles = []
    rates = []
    angle_loop_counter = 0

    for i, t in enumerate(time):
        # Angle loop (runs every 4th iteration = 250 Hz)
        if i % 4 == 0:
            # Compute desired rate from angle error
            rate_setpoint = angle_pid.compute(angle_setpoint, angle)
        else:
            # Keep previous rate setpoint
            pass

        # Rate loop (runs every iteration = 1000 Hz)
        control_torque = rate_pid.compute(rate_setpoint, rate)

        # Apply to plant (simple model)
        I = 0.01  # Moment of inertia
        damping = 0.5

        angular_accel = (control_torque - damping * rate) / I

        rate += angular_accel * dt
        angle += rate * dt

        angles.append(angle)
        rates.append(rate)

    # Plot
    fig, axes = plt.subplots(2, 1, figsize=(12, 10))

    axes[0].plot(time, angle_setpoint * np.ones_like(time), 'k--', linewidth=2, label='Setpoint')
    axes[0].plot(time, angles, 'b-', linewidth=1.5, label='Actual angle')
    axes[0].set_ylabel('Roll Angle (°)')
    axes[0].set_title('Cascaded Control: Angle Loop (Outer)')
    axes[0].legend()
    axes[0].grid(True)

    axes[1].plot(time, rates, 'r-', linewidth=1, label='Roll rate')
    axes[1].set_xlabel('Time (s)')
    axes[1].set_ylabel('Roll Rate (°/s)')
    axes[1].set_title('Rate Loop (Inner)')
    axes[1].legend()
    axes[1].grid(True)

    plt.tight_layout()
    plt.savefig('cascaded_control.png')

    print("Cascaded Control Architecture:")
    print("-" * 50)
    print("Outer loop (Angle): Slow (250 Hz)")
    print("  - Uses fused sensor data")
    print("  - Commands desired rate")
    print("  - Provides stability")
    print("\nInner loop (Rate): Fast (1000 Hz)")
    print("  - Uses raw gyro")
    print("  - Direct control of motors")
    print("  - Provides responsiveness")

cascaded_control_simulation()
```

---

## 5. Complementary Filter

### 5.1 The Problem: Sensor Fusion

**Gyroscope:**
- ✅ Accurate short-term, no lag
- ❌ Drifts over time (integration error)

**Accelerometer:**
- ✅ Accurate long-term (gravity reference)
- ❌ Noisy, affected by acceleration

**Solution:** Combine both!

### 5.2 Complementary Filter Equation

```
angle = α × (angle + gyro_rate × dt) + (1 - α) × accel_angle

Where:
α = time constant / (time constant + dt)
Typical α = 0.98 (98% gyro, 2% accel)
```

**Why it works:**
- High-pass filter on gyro (trust short-term)
- Low-pass filter on accel (trust long-term)
- Together = complementary (sum = 1)

```python
class ComplementaryFilter:
    """Simple and effective sensor fusion"""

    def __init__(self, alpha=0.98):
        self.alpha = alpha
        self.angle = 0.0

    def update(self, gyro_rate, accel_angle, dt):
        """
        gyro_rate: Angular velocity from gyro (°/s)
        accel_angle: Angle from accelerometer (°)
        dt: Time step (s)
        """
        # Integrate gyro
        gyro_angle = self.angle + gyro_rate * dt

        # Complementary filter
        self.angle = self.alpha * gyro_angle + (1 - self.alpha) * accel_angle

        return self.angle


def demonstrate_complementary_filter():
    """Show complementary filter fusing gyro and accel"""

    dt = 0.01
    time = np.arange(0, 20, dt)

    # True angle (sinusoidal)
    true_angle = 30 * np.sin(2*np.pi*0.1*time)

    # Gyro: accurate but drifts
    gyro_rate = np.gradient(true_angle, dt)  # True rate
    gyro_drift = 0.5 * time  # Linear drift
    gyro_rate += 0.5  # Add constant bias → causes drift

    # Accelerometer: noisy but no drift
    accel_angle = true_angle + 5*np.random.randn(len(time))  # 5° noise

    # Integrate gyro (dead reckoning)
    gyro_integrated = np.zeros_like(time)
    for i in range(1, len(time)):
        gyro_integrated[i] = gyro_integrated[i-1] + gyro_rate[i] * dt

    # Apply complementary filter
    comp_filter = ComplementaryFilter(alpha=0.98)
    comp_filtered = []

    for i in range(len(time)):
        angle = comp_filter.update(gyro_rate[i], accel_angle[i], dt)
        comp_filtered.append(angle)

    # Plot
    fig, axes = plt.subplots(3, 1, figsize=(12, 12))

    axes[0].plot(time, true_angle, 'k-', linewidth=2, label='True angle')
    axes[0].plot(time, gyro_integrated, 'r--', label='Gyro only (drifts!)')
    axes[0].set_ylabel('Angle (°)')
    axes[0].set_title('Gyro Integration - Drift Problem')
    axes[0].legend()
    axes[0].grid(True)

    axes[1].plot(time, true_angle, 'k-', linewidth=2, label='True angle')
    axes[1].plot(time, accel_angle, 'b.', alpha=0.3, markersize=2, label='Accel (noisy!)')
    axes[1].set_ylabel('Angle (°)')
    axes[1].set_title('Accelerometer - Noise Problem')
    axes[1].legend()
    axes[1].grid(True)

    axes[2].plot(time, true_angle, 'k-', linewidth=2, label='True angle')
    axes[2].plot(time, comp_filtered, 'g-', label='Complementary filter')
    axes[2].set_xlabel('Time (s)')
    axes[2].set_ylabel('Angle (°)')
    axes[2].set_title('Complementary Filter - Best of Both!')
    axes[2].legend()
    axes[2].grid(True)

    plt.tight_layout()
    plt.savefig('complementary_filter_demo.png')

    print("Complementary Filter Performance:")
    print("-" * 50)

    # Calculate errors
    error_gyro = np.mean(np.abs(gyro_integrated - true_angle))
    error_accel = np.mean(np.abs(accel_angle - true_angle))
    error_comp = np.mean(np.abs(comp_filtered - true_angle))

    print(f"Mean absolute error:")
    print(f"  Gyro only: {error_gyro:.2f}°")
    print(f"  Accel only: {error_accel:.2f}°")
    print(f"  Complementary: {error_comp:.2f}°")
    print(f"\nImprovement: {(1 - error_comp/error_accel)*100:.1f}% over accel alone")

demonstrate_complementary_filter()
```

### 5.3 Implementation for Quadcopter

```cpp
// complementary_filter.cpp
class ComplementaryFilter {
private:
    float alpha;
    float roll;
    float pitch;

public:
    ComplementaryFilter(float time_constant, float dt) {
        // Calculate alpha from time constant
        alpha = time_constant / (time_constant + dt);
        roll = 0.0;
        pitch = 0.0;
    }

    void update(float gyro_x, float gyro_y, float gyro_z,
                float accel_x, float accel_y, float accel_z,
                float dt) {

        // Calculate angle from accelerometer
        float accel_roll = atan2(accel_y, accel_z) * 57.2958;  // Convert to degrees
        float accel_pitch = atan2(-accel_x, sqrt(accel_y*accel_y + accel_z*accel_z)) * 57.2958;

        // Integrate gyro
        roll += gyro_x * dt;
        pitch += gyro_y * dt;

        // Complementary filter
        roll = alpha * roll + (1.0 - alpha) * accel_roll;
        pitch = alpha * pitch + (1.0 - alpha) * accel_pitch;

        // Yaw from gyro only (no accel reference for yaw)
        // Yaw drift is unavoidable without magnetometer
    }

    float getRoll() { return roll; }
    float getPitch() { return pitch; }
};

// Usage in main loop
ComplementaryFilter filter(0.98, 0.001);  // 98% gyro, 1ms dt

void loop() {
    // Read IMU
    float ax, ay, az, gx, gy, gz;
    readIMU(&ax, &ay, &az, &gx, &gy, &gz);

    // Update filter
    filter.update(gx, gy, gz, ax, ay, az, 0.001);

    // Get fused angles
    float roll = filter.getRoll();
    float pitch = filter.getPitch();

    // Use in control loop...
}
```

---

## 6. Kalman Filter

### 6.1 Introduction

**More sophisticated than complementary filter:**
- Statistically optimal
- Adapts to changing noise
- Can estimate unmeasured states

**Trade-off:**
- More computationally expensive
- Requires noise covariance tuning
- Can be overkill for simple attitude estimation

### 6.2 Kalman Filter Algorithm

```
Prediction step:
  x̂⁻ = A·x̂ + B·u
  P⁻ = A·P·Aᵀ + Q

Update step:
  K = P⁻·Hᵀ·(H·P⁻·Hᵀ + R)⁻¹
  x̂ = x̂⁻ + K·(z - H·x̂⁻)
  P = (I - K·H)·P⁻

Where:
x̂ = state estimate
P = estimation error covariance
K = Kalman gain
Q = process noise covariance
R = measurement noise covariance
```

**Don't worry if this looks complex - the code explains it!**

```python
class SimpleKalmanFilter:
    """1D Kalman filter for altitude estimation"""

    def __init__(self, process_noise, measurement_noise):
        self.Q = process_noise  # Process noise (how much we trust model)
        self.R = measurement_noise  # Measurement noise (sensor accuracy)

        self.x = 0.0  # State estimate (altitude)
        self.P = 1.0  # Estimation error covariance

    def predict(self, u, dt):
        """
        Prediction step
        u = control input (acceleration)
        """
        # State update: x = x + u*dt
        self.x = self.x + u * dt

        # Covariance update
        self.P = self.P + self.Q

    def update(self, measurement):
        """
        Update step with measurement
        """
        # Kalman gain
        K = self.P / (self.P + self.R)

        # Update estimate
        self.x = self.x + K * (measurement - self.x)

        # Update covariance
        self.P = (1 - K) * self.P

    def get_state(self):
        return self.x


def demonstrate_kalman_filter():
    """Kalman filter for altitude estimation"""

    dt = 0.01
    time = np.arange(0, 20, dt)

    # True altitude (smoothly varying)
    true_altitude = 10 + 5*np.sin(2*np.pi*0.1*time)

    # Barometer: slow but accurate on average
    baro_noise = 0.5  # 0.5m noise
    baro_measurement = true_altitude + baro_noise * np.random.randn(len(time))

    # Accelerometer: fast but drifts when integrated
    true_accel = np.gradient(np.gradient(true_altitude, dt), dt)
    accel_bias = 0.1  # Constant bias
    accel_measurement = true_accel + accel_bias + 0.3*np.random.randn(len(time))

    # Integrate accelerometer (dead reckoning)
    velocity = 0
    position_accel = 0
    accel_integrated = []

    for acc in accel_measurement:
        velocity += acc * dt
        position_accel += velocity * dt
        accel_integrated.append(position_accel)

    # Kalman filter
    kf = SimpleKalmanFilter(process_noise=0.01, measurement_noise=0.25)

    kalman_estimates = []

    for i in range(len(time)):
        # Predict with accelerometer
        kf.predict(accel_measurement[i], dt)

        # Update with barometer (every 10 samples = 100ms update rate)
        if i % 10 == 0:
            kf.update(baro_measurement[i])

        kalman_estimates.append(kf.get_state())

    # Plot
    fig, axes = plt.subplots(2, 1, figsize=(12, 10))

    axes[0].plot(time, true_altitude, 'k-', linewidth=2, label='True altitude')
    axes[0].plot(time, baro_measurement, 'b.', alpha=0.3, markersize=1, label='Barometer (noisy)')
    axes[0].plot(time, accel_integrated, 'r--', alpha=0.7, label='Accel integrated (drifts)')
    axes[0].set_ylabel('Altitude (m)')
    axes[0].set_title('Sensor Data')
    axes[0].legend()
    axes[0].grid(True)

    axes[1].plot(time, true_altitude, 'k-', linewidth=2, label='True altitude')
    axes[1].plot(time, kalman_estimates, 'g-', linewidth=1.5, label='Kalman estimate')
    axes[1].set_xlabel('Time (s)')
    axes[1].set_ylabel('Altitude (m)')
    axes[1].set_title('Kalman Filter Fusion')
    axes[1].legend()
    axes[1].grid(True)

    plt.tight_layout()
    plt.savefig('kalman_filter_demo.png')

    # Calculate errors
    error_baro = np.mean(np.abs(baro_measurement - true_altitude))
    error_accel = np.mean(np.abs(accel_integrated - true_altitude))
    error_kalman = np.mean(np.abs(kalman_estimates - true_altitude))

    print("Kalman Filter Performance:")
    print("-" * 50)
    print(f"Mean absolute error:")
    print(f"  Barometer: {error_baro:.2f}m")
    print(f"  Accel integrated: {error_accel:.2f}m")
    print(f"  Kalman filter: {error_kalman:.2f}m")

demonstrate_kalman_filter()
```

---

## 7. Advanced Control

### 7.1 Feed-Forward Control

Add expected control output based on model:

```cpp
float pid_output = pid.compute(setpoint, measured);
float feedforward = model_based_estimate(setpoint);
float total_control = pid_output + feedforward;
```

**Benefits:**
- Faster response
- Reduces steady-state error
- Less reliance on integral term

### 7.2 LQR (Linear Quadratic Regulator)

Optimal control for linear systems:
- Mathematically optimal
- Requires state-space model
- Used in advanced flight controllers

*Beyond scope of DIY build, but worth knowing!*

### 7.3 Adaptive Control

PID gains change based on flight conditions:
```cpp
// Adjust gains based on throttle (different dynamics at high/low throttle)
float throttle_factor = throttle / 1000.0;  // 0-1
float kp_adjusted = kp_base * (1 + 0.2 * throttle_factor);
```

---

## Summary

You now understand:

✅ **Control fundamentals** - Open vs closed loop, system response
✅ **PID theory** - Each term's effect and interaction
✅ **Tuning methods** - Manual, Ziegler-Nichols, practical tips
✅ **Cascaded loops** - Rate, angle, velocity, position hierarchy
✅ **Sensor fusion** - Complementary filter (simple), Kalman filter (optimal)
✅ **Implementation** - Production-ready C++ code

**Key Takeaways:**

1. PID is simple but powerful when tuned correctly
2. Cascade control provides stability and modularity
3. Sensor fusion is essential for stable flight
4. Start conservative, tune iteratively
5. Each quadcopter is unique - expect to tune!

---

**Next Steps:**

- **[Q5: Sensors & IMU](./Q5-sensors-imu.md)** - Understanding your sensors
- **[Q8: Flight Controller Code](./Q8-flight-controller-code.md)** - Complete implementation
- **[Q12: Testing & Tuning](./Q12-testing-tuning.md)** - Practical tuning guide

---

## Practice Problems

1. Calculate PID gains using Ziegler-Nichols if Ku=3.5, Tu=0.8s
2. Implement complementary filter with 95% gyro weight
3. Explain why rate loop must run faster than angle loop
4. Design cascaded controller for position hold (add velocity loop)
5. Compare complementary vs Kalman filter for your application

---

*Last Updated: November 2025*
