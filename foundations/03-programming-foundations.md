# Programming Foundations

> **Essential programming skills for embedded systems and signal processing**

---

## Table of Contents

1. [C/C++ for Embedded Systems](#1-cc-for-embedded-systems)
2. [Python for Signal Processing](#2-python-for-signal-processing)
3. [Real-Time Programming Concepts](#3-real-time-programming-concepts)
4. [Development Tools](#4-development-tools)
5. [Debugging Techniques](#5-debugging-techniques)

---

## 1. C/C++ for Embedded Systems

### 1.1 Why C/C++ for Quadcopters?

- **Fast execution** - Critical for control loops running at 1000+ Hz
- **Direct hardware access** - Control timers, interrupts, peripherals
- **Predictable performance** - No garbage collection pauses
- **Small memory footprint** - Fits on microcontrollers

### 1.2 Basic Structure of Flight Controller Code

```cpp
// main.cpp - Simplified flight controller structure
#include <Arduino.h>
#include "sensors.h"
#include "control.h"
#include "motors.h"

// Global variables (state)
struct FlightState {
    float roll, pitch, yaw;          // Current angles (degrees)
    float roll_rate, pitch_rate, yaw_rate;  // Angular velocities (deg/s)
    float altitude;                   // Height (meters)
    uint32_t loop_timer;             // For timing
} state;

struct ControlInputs {
    float throttle;  // 0.0 to 1.0
    float roll_cmd;  // -1.0 to 1.0
    float pitch_cmd;
    float yaw_cmd;
} inputs;

// Constants
const float LOOP_FREQUENCY = 1000.0;  // 1000 Hz
const float DT = 1.0 / LOOP_FREQUENCY;  // 1 ms

void setup() {
    Serial.begin(115200);

    // Initialize hardware
    initSensors();    // I2C sensors (IMU, etc.)
    initMotors();     // PWM outputs for ESCs
    initReceiver();   // RC receiver input

    Serial.println("Flight Controller Initialized");

    // Start loop timer
    state.loop_timer = micros();
}

void loop() {
    // 1. READ SENSORS (fastest changing data first)
    readIMU(&state.roll_rate, &state.pitch_rate, &state.yaw_rate,
            &state.roll, &state.pitch, &state.yaw);

    // 2. READ RC INPUTS
    readRCInputs(&inputs.throttle, &inputs.roll_cmd,
                 &inputs.pitch_cmd, &inputs.yaw_cmd);

    // 3. CONTROL ALGORITHMS
    float motor_commands[4];
    calculateMotorCommands(&state, &inputs, motor_commands);

    // 4. OUTPUT TO MOTORS
    writeMotors(motor_commands);

    // 5. TIMING - Wait for next loop iteration
    maintainLoopTiming(&state.loop_timer, DT);
}
```

### 1.3 Memory Management

**Stack vs Heap:**

```cpp
// STACK - Fast, automatic cleanup
void stackExample() {
    float sensor_data[6];  // Allocated on stack
    int counter = 0;
    // Automatically freed when function returns
}

// HEAP - Manual management, use sparingly in embedded systems
void heapExample() {
    float* big_buffer = (float*)malloc(1000 * sizeof(float));
    // ... use buffer ...
    free(big_buffer);  // Must manually free!
}

// AVOID heap in interrupt routines!
// Prefer static allocation for embedded systems
```

**Best Practices for Embedded:**

```cpp
// GOOD - Fixed-size buffers
#define IMU_BUFFER_SIZE 10
float imu_buffer[IMU_BUFFER_SIZE];

// GOOD - Stack variables
void processData() {
    float temp_value;  // Fast, automatic
    // Process...
}

// BAD - Dynamic allocation in real-time code
void processDataBad() {
    float* data = new float[100];  // SLOW! Avoid in control loops
    // ...
    delete[] data;
}
```

### 1.4 Fixed-Point vs Floating-Point

Some microcontrollers don't have FPU (Floating-Point Unit):

```cpp
// Floating-point (easy, but slow without FPU)
float acceleration = 9.81;
float velocity = acceleration * 0.001;  // *dt

// Fixed-point (faster on processors without FPU)
// Represent 9.81 as integer: 9.81 * 1000 = 9810
int32_t acceleration_fp = 9810;  // In milli-g units
int32_t velocity_fp = (acceleration_fp * 1) / 1000;  // *dt, keep scale

// Modern ARM Cortex-M4F+ have FPU, so use float!
// Check your microcontroller specs
```

### 1.5 Practical Example: PID Controller

```cpp
// pid.h
#ifndef PID_H
#define PID_H

class PIDController {
private:
    float kp, ki, kd;
    float integral;
    float previous_error;
    float integral_limit;

public:
    PIDController(float p, float i, float d, float i_limit = 100.0)
        : kp(p), ki(i), kd(d), integral(0), previous_error(0),
          integral_limit(i_limit) {}

    float compute(float setpoint, float measured, float dt) {
        float error = setpoint - measured;

        // Proportional term
        float P = kp * error;

        // Integral term (with anti-windup)
        integral += error * dt;
        integral = constrain(integral, -integral_limit, integral_limit);
        float I = ki * integral;

        // Derivative term
        float derivative = (error - previous_error) / dt;
        float D = kd * derivative;

        previous_error = error;

        // PID output
        return P + I + D;
    }

    void reset() {
        integral = 0;
        previous_error = 0;
    }

    float constrain(float value, float min_val, float max_val) {
        if (value < min_val) return min_val;
        if (value > max_val) return max_val;
        return value;
    }
};

#endif
```

**Usage:**

```cpp
// Create PID controllers for each axis
PIDController roll_rate_pid(1.5, 0.1, 0.05);
PIDController pitch_rate_pid(1.5, 0.1, 0.05);
PIDController yaw_rate_pid(2.0, 0.15, 0.0);

void loop() {
    // Read sensors
    float roll_rate_measured = readGyroRoll();

    // Compute control output
    float roll_command = 0.0;  // From RC input
    float roll_output = roll_rate_pid.compute(roll_command,
                                               roll_rate_measured,
                                               0.001);  // 1ms dt

    // roll_output is torque command -> motor mixing
}
```

---

## 2. Python for Signal Processing

### 2.1 Why Python for Radar?

- **NumPy/SciPy** - Fast array operations, FFT, filters
- **Matplotlib** - Visualization
- **Easy prototyping** - Test algorithms before implementing in C/C++
- **Serial communication** - Interface with hardware

### 2.2 Essential Libraries

```python
import numpy as np              # Numerical computing
import matplotlib.pyplot as plt # Plotting
from scipy import signal        # Signal processing
from scipy import fft           # FFT functions
import serial                   # Serial port communication
```

### 2.3 Reading Radar Data from Serial Port

```python
import serial
import numpy as np
import struct

class RadarInterface:
    def __init__(self, port='/dev/ttyUSB0', baudrate=115200):
        self.ser = serial.Serial(port, baudrate, timeout=1)
        self.buffer_size = 1024

    def read_samples(self, n_samples):
        """Read n samples from ADC"""
        # Send command
        self.ser.write(b'READ\n')

        # Read binary data
        data = self.ser.read(n_samples * 2)  # 2 bytes per sample

        # Convert to numpy array
        samples = np.frombuffer(data, dtype=np.uint16)

        # Convert to voltage (assuming 12-bit ADC, 3.3V reference)
        voltage = samples * (3.3 / 4096.0)

        return voltage

    def close(self):
        self.ser.close()

# Usage
radar = RadarInterface('/dev/ttyACM0')
data = radar.read_samples(1024)
radar.close()
```

### 2.4 FFT Processing

```python
def process_radar_fft(time_domain_signal, sample_rate):
    """
    Process radar signal using FFT
    Returns frequencies and magnitude spectrum
    """
    # Apply window to reduce spectral leakage
    window = signal.hann(len(time_domain_signal))
    windowed_signal = time_domain_signal * window

    # Compute FFT
    fft_result = fft.fft(windowed_signal)

    # Frequency axis
    freqs = fft.fftfreq(len(time_domain_signal), 1/sample_rate)

    # Magnitude spectrum
    magnitude = np.abs(fft_result)

    # Only positive frequencies
    positive_mask = freqs >= 0
    freqs_positive = freqs[positive_mask]
    magnitude_positive = magnitude[positive_mask]

    return freqs_positive, magnitude_positive

# Example usage
fs = 100e3  # 100 kHz sampling rate
t = np.arange(0, 0.1, 1/fs)

# Simulate signal: target at 5 kHz (Doppler or beat frequency)
signal_sim = np.cos(2*np.pi*5000*t) + 0.5*np.random.randn(len(t))

# Process
freqs, magnitude = process_radar_fft(signal_sim, fs)

# Plot
plt.figure(figsize=(12, 5))

plt.subplot(1, 2, 1)
plt.plot(t[:1000]*1000, signal_sim[:1000])
plt.xlabel('Time (ms)')
plt.ylabel('Amplitude')
plt.title('Time Domain Signal')
plt.grid(True)

plt.subplot(1, 2, 2)
plt.plot(freqs/1000, magnitude)
plt.xlabel('Frequency (kHz)')
plt.ylabel('Magnitude')
plt.title('Frequency Domain (FFT)')
plt.grid(True)
plt.xlim([0, 10])

plt.tight_layout()
plt.savefig('radar_fft_processing.png')
plt.show()
```

### 2.5 Real-Time Plotting

```python
import matplotlib.pyplot as plt
import matplotlib.animation as animation
import numpy as np

class RadarPlot:
    def __init__(self, radar_interface):
        self.radar = radar_interface
        self.fig, (self.ax1, self.ax2) = plt.subplots(2, 1, figsize=(10, 8))

        # Time domain plot
        self.line1, = self.ax1.plot([], [])
        self.ax1.set_xlim(0, 1024)
        self.ax1.set_ylim(0, 3.3)
        self.ax1.set_xlabel('Sample')
        self.ax1.set_ylabel('Voltage (V)')
        self.ax1.set_title('Time Domain')
        self.ax1.grid(True)

        # Frequency domain plot
        self.line2, = self.ax2.plot([], [])
        self.ax2.set_xlim(0, 10)  # 0-10 kHz
        self.ax2.set_ylim(0, 100)
        self.ax2.set_xlabel('Frequency (kHz)')
        self.ax2.set_ylabel('Magnitude')
        self.ax2.set_title('Frequency Domain')
        self.ax2.grid(True)

    def update(self, frame):
        # Read new data
        data = self.radar.read_samples(1024)

        # Update time domain
        self.line1.set_data(range(len(data)), data)

        # Update frequency domain
        freqs, mag = process_radar_fft(data, 100e3)
        self.line2.set_data(freqs/1000, mag)

        return self.line1, self.line2

    def run(self):
        ani = animation.FuncAnimation(
            self.fig, self.update, interval=50, blit=True)
        plt.show()

# Usage
# radar = RadarInterface()
# plotter = RadarPlot(radar)
# plotter.run()
```

---

## 3. Real-Time Programming Concepts

### 3.1 Interrupts

**What are interrupts?**
Hardware events that pause normal execution to handle urgent tasks.

```cpp
// Example: Timer interrupt for precise loop timing
#include <Arduino.h>

volatile bool time_for_loop = false;
volatile uint32_t loop_counter = 0;

// Interrupt Service Routine (ISR)
void IRAM_ATTR timerISR() {
    time_for_loop = true;
    loop_counter++;
}

void setup() {
    // Setup timer interrupt at 1000 Hz (1 ms period)
    hw_timer_t* timer = timerBegin(0, 80, true);  // 80 MHz / 80 = 1 MHz
    timerAttachInterrupt(timer, &timerISR, true);
    timerAlarmWrite(timer, 1000, true);  // 1000 microseconds = 1 ms
    timerAlarmEnable(timer);
}

void loop() {
    if (time_for_loop) {
        time_for_loop = false;

        // DO CONTROL LOOP WORK HERE
        // This runs at exactly 1000 Hz
        readSensors();
        calculateControl();
        updateMotors();
    }

    // Other low-priority tasks can run here
    if (loop_counter % 1000 == 0) {  // Every second
        Serial.println("Heartbeat");
    }
}
```

**ISR Rules:**
- Keep ISRs SHORT and FAST
- No blocking operations (no delays, prints, etc.)
- Use `volatile` for shared variables
- Minimize floating-point operations

### 3.2 Timing and Scheduling

```cpp
// Precise timing for control loops
void maintainLoopTiming(uint32_t* last_time, float dt_seconds) {
    uint32_t dt_micros = dt_seconds * 1e6;

    // Calculate time until next loop
    uint32_t current_time = micros();
    uint32_t elapsed = current_time - *last_time;

    if (elapsed < dt_micros) {
        delayMicroseconds(dt_micros - elapsed);
    } else {
        // Loop overrun! Log warning
        Serial.println("WARNING: Loop time exceeded!");
    }

    *last_time = micros();
}

// Multi-rate scheduling
void loop() {
    static uint32_t fast_loop_timer = 0;
    static uint32_t slow_loop_timer = 0;
    static uint16_t loop_counter = 0;

    // Fast loop: 1000 Hz (1 ms)
    if (micros() - fast_loop_timer >= 1000) {
        fast_loop_timer = micros();
        loop_counter++;

        // Critical tasks every loop
        readIMU();
        calculateControl();
        updateMotors();

        // Medium-rate task: 100 Hz (every 10 loops)
        if (loop_counter % 10 == 0) {
            readBarometer();
            updateAltitudeControl();
        }

        // Slow task: 10 Hz (every 100 loops)
        if (loop_counter % 100 == 0) {
            readGPS();
            updatePositionControl();
            sendTelemetry();
        }

        // Reset counter
        if (loop_counter >= 1000) loop_counter = 0;
    }

    // Background tasks (non-critical)
    checkSerialCommands();
}
```

### 3.3 Data Structures for Real-Time

**Ring Buffers** - Essential for sensor data buffering:

```cpp
template<typename T, size_t SIZE>
class RingBuffer {
private:
    T buffer[SIZE];
    size_t head;
    size_t tail;
    size_t count;

public:
    RingBuffer() : head(0), tail(0), count(0) {}

    bool push(const T& item) {
        if (count >= SIZE) return false;  // Buffer full

        buffer[head] = item;
        head = (head + 1) % SIZE;
        count++;
        return true;
    }

    bool pop(T& item) {
        if (count == 0) return false;  // Buffer empty

        item = buffer[tail];
        tail = (tail + 1) % SIZE;
        count--;
        return true;
    }

    size_t size() const { return count; }
    bool is_full() const { return count >= SIZE; }
    bool is_empty() const { return count == 0; }
};

// Usage
RingBuffer<float, 100> imu_data_buffer;

// In ISR or fast loop
void readSensor() {
    float accel_x = readAccelerometer();
    imu_data_buffer.push(accel_x);
}

// In processing task
void processBuffer() {
    float value;
    while (imu_data_buffer.pop(value)) {
        // Process value
        applyFilter(value);
    }
}
```

---

## 4. Development Tools

### 4.1 Version Control with Git

```bash
# Initialize repository
git init

# Add files
git add main.cpp sensors.h control.cpp
git commit -m "Initial flight controller implementation"

# Create feature branch
git checkout -b feature/altitude-hold

# Make changes, commit
git add altitude_control.cpp
git commit -m "Add altitude hold PID controller"

# Merge back to main
git checkout main
git merge feature/altitude-hold

# Push to remote
git remote add origin https://github.com/user/quadcopter.git
git push -u origin main
```

### 4.2 PlatformIO (Better than Arduino IDE)

**platformio.ini:**
```ini
[env:esp32]
platform = espressif32
board = esp32dev
framework = arduino
monitor_speed = 115200
lib_deps =
    Wire
    SPI
    adafruit/Adafruit MPU6050 @ ^2.2.4
    adafruit/Adafruit BMP280 Library @ ^2.6.6

build_flags =
    -DCORE_DEBUG_LEVEL=3
    -O2
```

**Commands:**
```bash
# Build
pio run

# Upload
pio run --target upload

# Monitor serial
pio device monitor

# Run unit tests
pio test
```

### 4.3 Debugging with Serial Monitor

```cpp
// Structured logging
#define DEBUG_LEVEL 2  // 0=none, 1=error, 2=info, 3=debug

#if DEBUG_LEVEL >= 1
    #define LOG_ERROR(msg) Serial.print("[ERROR] "); Serial.println(msg)
#else
    #define LOG_ERROR(msg)
#endif

#if DEBUG_LEVEL >= 2
    #define LOG_INFO(msg) Serial.print("[INFO] "); Serial.println(msg)
#else
    #define LOG_INFO(msg)
#endif

#if DEBUG_LEVEL >= 3
    #define LOG_DEBUG(msg) Serial.print("[DEBUG] "); Serial.println(msg)
#else
    #define LOG_DEBUG(msg)
#endif

// Usage
void initSensors() {
    LOG_INFO("Initializing IMU...");

    if (!imu.begin()) {
        LOG_ERROR("IMU initialization failed!");
        while(1);  // Halt
    }

    LOG_INFO("IMU initialized successfully");
    LOG_DEBUG("IMU Address: 0x68");
}
```

**Data Logging for Analysis:**

```cpp
void logFlightData() {
    // CSV format for easy import to Excel/Python
    Serial.print(millis());           Serial.print(",");
    Serial.print(state.roll, 3);      Serial.print(",");
    Serial.print(state.pitch, 3);     Serial.print(",");
    Serial.print(state.yaw, 3);       Serial.print(",");
    Serial.print(inputs.throttle, 3); Serial.print(",");
    Serial.print(motor_cmd[0], 3);    Serial.print(",");
    Serial.print(motor_cmd[1], 3);    Serial.print(",");
    Serial.print(motor_cmd[2], 3);    Serial.print(",");
    Serial.println(motor_cmd[3], 3);
}

// Python script to read and plot
"""
import serial
import pandas as pd
import matplotlib.pyplot as plt

ser = serial.Serial('/dev/ttyUSB0', 115200)
data = []

for i in range(10000):  # Read 10000 samples
    line = ser.readline().decode().strip()
    values = [float(x) for x in line.split(',')]
    data.append(values)

df = pd.DataFrame(data, columns=['time', 'roll', 'pitch', 'yaw',
                                  'throttle', 'm1', 'm2', 'm3', 'm4'])

df.plot(x='time', y=['roll', 'pitch', 'yaw'])
plt.show()
"""
```

---

## 5. Debugging Techniques

### 5.1 Logic Analyzer

For digital signals (I2C, SPI, PWM):

```cpp
// Toggle a debug pin to measure timing
#define DEBUG_PIN 13

void loop() {
    digitalWrite(DEBUG_PIN, HIGH);  // Mark start

    // Your code here
    readSensors();
    calculateControl();

    digitalWrite(DEBUG_PIN, LOW);   // Mark end

    // Use logic analyzer to measure pulse width = execution time
}
```

### 5.2 Unit Testing

```cpp
// test_pid.cpp
#include <unity.h>
#include "pid.h"

void test_pid_proportional() {
    PIDController pid(1.0, 0.0, 0.0);

    float output = pid.compute(10.0, 5.0, 0.01);  // Error = 5.0
    TEST_ASSERT_FLOAT_WITHIN(0.01, 5.0, output);  // Expect P = 1.0 * 5.0
}

void test_pid_integral() {
    PIDController pid(0.0, 1.0, 0.0);

    pid.compute(10.0, 5.0, 0.01);  // Error = 5, integral = 0.05
    float output = pid.compute(10.0, 5.0, 0.01);  // Integral = 0.10

    TEST_ASSERT_FLOAT_WITHIN(0.01, 0.10, output);
}

void setup() {
    UNITY_BEGIN();
    RUN_TEST(test_pid_proportional);
    RUN_TEST(test_pid_integral);
    UNITY_END();
}

void loop() {}
```

### 5.3 Simulation Testing

Test algorithms before flying!

```python
# simulate_quadcopter.py
import numpy as np
import matplotlib.pyplot as plt

class QuadcopterSimulator:
    def __init__(self, mass=1.5, I_xx=0.01):
        self.m = mass
        self.I = I_xx
        self.g = 9.81

        # State: [roll, roll_rate]
        self.state = np.array([0.0, 0.0])

    def update(self, torque, dt):
        roll, roll_rate = self.state

        # Physics: I * alpha = torque
        alpha = torque / self.I

        # Integrate
        roll_rate += alpha * dt
        roll += roll_rate * dt

        self.state = np.array([roll, roll_rate])
        return roll, roll_rate

class PIDController:
    def __init__(self, kp, ki, kd):
        self.kp, self.ki, self.kd = kp, ki, kd
        self.integral = 0
        self.prev_error = 0

    def compute(self, setpoint, measured, dt):
        error = setpoint - measured
        self.integral += error * dt
        derivative = (error - self.prev_error) / dt
        self.prev_error = error
        return self.kp * error + self.ki * self.integral + self.kd * derivative

# Simulate
quad = QuadcopterSimulator()
pid = PIDController(kp=2.0, ki=0.1, kd=0.3)

dt = 0.001  # 1 ms
time = np.arange(0, 5, dt)
roll_history = []
roll_cmd = 30  # Command 30 degree roll

for t in time:
    roll, roll_rate = quad.state

    # PID control
    torque = pid.compute(np.radians(roll_cmd), roll, dt)
    torque = np.clip(torque, -1.0, 1.0)  # Limit

    # Update quadcopter
    quad.update(torque, dt)

    roll_history.append(np.degrees(roll))

# Plot
plt.figure(figsize=(10, 6))
plt.plot(time, roll_history, label='Actual')
plt.axhline(y=roll_cmd, color='r', linestyle='--', label='Command')
plt.xlabel('Time (s)')
plt.ylabel('Roll Angle (degrees)')
plt.title('Simulated PID Response')
plt.legend()
plt.grid(True)
plt.savefig('pid_simulation.png')
plt.show()
```

---

## Summary

You now know:

✅ **C/C++ for embedded** - Memory management, interrupts, real-time code
✅ **Python for signal processing** - NumPy, SciPy, FFT, serial communication
✅ **Real-time concepts** - Interrupts, timing, scheduling
✅ **Development tools** - Git, PlatformIO, debugging
✅ **Testing** - Unit tests, simulation

**Best Practices:**

1. **Profile before optimizing** - Measure execution time
2. **Test on hardware early** - Simulations aren't perfect
3. **Use version control** - Git from day one
4. **Document as you go** - Comment complex algorithms
5. **Plan for debugging** - Add logging, test points

---

**Next Steps:**

- **[Electronics Foundations](./04-electronics-foundations.md)** - Understanding hardware
- **[Q8: Flight Controller Code](../quadcopter/Q8-flight-controller-code.md)** - Complete implementation
- **[R12: Radar Signal Processing Code](../radar/R12-signal-processing-code.md)** - DSP implementation

---

*Last Updated: November 2025*
