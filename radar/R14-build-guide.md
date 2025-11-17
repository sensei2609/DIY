# R14: Radar Build Guide

> **Build your own object-tracking radar from scratch**

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Parts List](#2-parts-list)
3. [Three Build Options](#3-three-build-options)
4. [Option 1: Doppler Radar (Simplest)](#4-option-1-doppler-radar-simplest)
5. [Option 2: FMCW Radar Module (Best)](#5-option-2-fmcw-radar-module-best)
6. [Option 3: Custom FMCW (Advanced)](#6-option-3-custom-fmcw-advanced)
7. [Software Implementation](#7-software-implementation)
8. [Testing and Calibration](#8-testing-and-calibration)

---

## 1. Project Overview

We'll build THREE different radar systems, from simplest to most advanced:

| Type | Complexity | Cost | Capabilities |
|------|-----------|------|--------------|
| Doppler | ⭐ Easy | $15-30 | Velocity only, no range |
| FMCW Module | ⭐⭐ Medium | $50-100 | Range + Velocity |
| Custom FMCW | ⭐⭐⭐⭐⭐ Hard | $200+ | Full control, research |

**Recommended:** Start with Option 1 (Doppler) to learn basics, then move to Option 2 (FMCW module) for full functionality.

---

## 2. Parts List

### Core Components (All Options)

**Microcontroller:**
- [ ] ESP32 development board - $8-12
  - Or: Raspberry Pi 4 (for more processing power) - $35-55

**Power Supply:**
- [ ] USB power adapter (5V, 2A minimum) - $5-10
- [ ] Breadboard power supply (3.3V/5V) - $3-5

**Display (Optional but helpful):**
- [ ] 0.96" OLED display (I2C) - $4-8
- [ ] Or: Use computer via USB serial

**Miscellaneous:**
- [ ] Breadboard (full size) - $5-8
- [ ] Jumper wires (M-M, M-F) - $5-10
- [ ] USB cable - $3-5
- [ ] Enclosure (optional) - $5-15

---

## 3. Three Build Options

### Option 1: Doppler Radar - HB100 Module

**Best for:** Learning radar basics, motion detection
**Measures:** Velocity only (via Doppler shift)
**Range:** Up to 20m
**Cost:** ~$15-30

**Additional parts:**
- [ ] HB100 or CDM324 Doppler radar module - $8-15
- [ ] LM358 op-amp IC - $0.50
- [ ] Resistors/capacitors (see schematic) - $2-5

### Option 2: FMCW Radar Module

**Best for:** Complete radar system, DIY tracking
**Measures:** Range AND velocity
**Range:** 0.5m - 100m (depending on module)
**Cost:** ~$50-100

**Additional parts:**
- [ ] BGT24MTR11 24GHz FMCW module - $40-60
  - Or: IWR1443 77GHz module - $60-80
- [ ] ADC (if not built-in) - ADS1115 16-bit - $8-12

### Option 3: Custom FMCW from Components

**Best for:** Research, full control, learning RF design
**Requires:** RF engineering knowledge, VNA for tuning
**Cost:** $200-500+

**Components:**
- [ ] VCO (voltage-controlled oscillator) - $20-40
- [ ] Mixer ICs (2x) - $15-30
- [ ] LNA (low-noise amplifier) - $10-20
- [ ] Patch antennas (2x) or horn antennas - $20-50
- [ ] RF PCB (custom design) - $30-100
- [ ] Many passive components

---

## 4. Option 1: Doppler Radar (Simplest)

### 4.1 How It Works

```
HB100 Module:
- Transmits 10.525 GHz continuous wave
- Receives reflected signal
- Outputs IF (difference frequency) = Doppler shift
- Output is LOW FREQUENCY (< 10 kHz)
```

### 4.2 Circuit Schematic

```
HB100 Doppler Radar Interface Circuit:

                    +5V
                     |
                     R1 (10kΩ)
                     |
    HB100       C1   |      C2
    IF Out ----||----+------||---- To ESP32 ADC
            100nF    |    100nF    (GPIO 34)
                     |
                    GND

HB100 Pinout:
Pin 1: +5V
Pin 2: GND
Pin 3: IF Output (connect via caps to ADC)
```

### 4.3 Assembly Steps

```
1. Connect HB100 power:
   - Pin 1 → +5V
   - Pin 2 → GND

2. Connect IF output:
   - Pin 3 → 100nF cap → 10kΩ resistor to +5V
   - Junction → 100nF cap → ESP32 GPIO34 (ADC)

3. Connect ESP32:
   - USB to computer
   - GND to common ground

4. Done! No complex RF needed.
```

### 4.4 Software (Doppler Detection)

```python
# doppler_radar.py - Run on ESP32 with MicroPython

from machine import ADC, Pin
import time
import math

# Setup ADC
adc = ADC(Pin(34))
adc.atten(ADC.ATTN_11DB)  # Full range 0-3.6V
adc.width(ADC.WIDTH_12BIT)  # 12-bit resolution

# Parameters
f_carrier = 10.525e9  # 10.525 GHz
c = 3e8  # Speed of light
sample_rate = 5000  # 5 kHz
samples = 1024

def read_samples(n):
    """Read n ADC samples"""
    data = []
    for i in range(n):
        data.append(adc.read())
        time.sleep_us(1000000 // sample_rate)
    return data

def calculate_velocity(fft_freq, fft_mag):
    """Find peak frequency and convert to velocity"""
    # Find peak (ignore DC component)
    peak_idx = 0
    peak_mag = 0

    for i in range(10, len(fft_mag)//2):  # Skip DC and low freqs
        if fft_mag[i] > peak_mag:
            peak_mag = fft_mag[i]
            peak_idx = i

    if peak_mag < 100:  # Threshold for detection
        return 0, 0

    # Peak frequency
    f_doppler = fft_freq[peak_idx]

    # Calculate velocity: v = f_doppler * c / (2 * f_carrier)
    velocity = f_doppler * c / (2 * f_carrier)

    return velocity, f_doppler

# Main loop
print("Doppler Radar - Motion Detection")
print("Wave hand in front of sensor...")

while True:
    # Read samples
    data = read_samples(samples)

    # Simple FFT (use ufft library or send to PC)
    # For demo, just print raw values
    avg = sum(data) / len(data)
    print(f"Average ADC: {avg:.1f}")

    # Detect motion (variance in signal)
    variance = sum((x - avg)**2 for x in data) / len(data)
    if variance > 1000:
        print("MOTION DETECTED!")

    time.sleep(0.1)
```

**Better approach:** Send data to PC for FFT processing

```python
# doppler_radar_pc.py - Run on PC

import serial
import numpy as np
from scipy import fft
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation

ser = serial.Serial('COM3', 115200)  # Adjust port
sample_rate = 5000
f_carrier = 10.525e9
c = 3e8

fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(10, 8))

def update(frame):
    # Read data from ESP32
    data = []
    for i in range(1024):
        line = ser.readline().decode().strip()
        try:
            data.append(int(line))
        except:
            pass

    if len(data) < 1024:
        return

    # Convert to voltage
    voltage = np.array(data) * 3.3 / 4096

    # Remove DC
    voltage = voltage - np.mean(voltage)

    # Time domain plot
    ax1.clear()
    ax1.plot(voltage[:200])
    ax1.set_ylabel('Voltage (V)')
    ax1.set_title('Time Domain')
    ax1.grid(True)

    # FFT
    fft_result = fft.fft(voltage * np.hanning(len(voltage)))
    fft_mag = np.abs(fft_result[:len(fft_result)//2])
    fft_freq = fft.fftfreq(len(voltage), 1/sample_rate)[:len(voltage)//2]

    # Find peak
    peak_idx = np.argmax(fft_mag[10:]) + 10  # Skip DC
    f_doppler = fft_freq[peak_idx]
    velocity = f_doppler * c / (2 * f_carrier)

    # Frequency domain plot
    ax2.clear()
    ax2.plot(fft_freq, fft_mag)
    ax2.set_xlabel('Frequency (Hz)')
    ax2.set_ylabel('Magnitude')
    ax2.set_title(f'FFT - Velocity: {velocity:.2f} m/s')
    ax2.set_xlim([0, 2000])
    ax2.grid(True)

    if fft_mag[peak_idx] > 50:
        ax2.axvline(x=f_doppler, color='r', linestyle='--',
                    label=f'{f_doppler:.1f} Hz')
        ax2.legend()

ani = FuncAnimation(fig, update, interval=100)
plt.tight_layout()
plt.show()
```

---

## 5. Option 2: FMCW Radar Module (Best)

### 5.1 Recommended Module: BGT24MTR11

**Specifications:**
- Frequency: 24-24.25 GHz (K-band)
- Range: 0.5m - 100m
- Chirp bandwidth: Up to 250 MHz
- Output: IF signal (baseband)

**Pinout:**
```
Pin 1: VCC (+5V)
Pin 2: GND
Pin 3: V_tune (chirp control, 0-5V)
Pin 4: IF1 (channel 1 output)
Pin 5: IF2 (channel 2 output, optional)
```

### 5.2 Circuit Schematic

```
FMCW Radar with BGT24MTR11:

                    +5V
                     |
                  ┌──┴──┐
                  │     │
      ESP32       │ BGT │
  ┌─────────┐     │24MTR│
  │         │     │ 11  │
  │ DAC1────┼─────┤Vtune│  (Chirp control)
  │ GPIO25  │     │     │
  │         │     │ IF1 │─── 100nF cap ─── Low-pass filter ─┐
  │ ADC1────┼─────┤     │                                    │
  │ GPIO36  │     │ IF2 │─── 100nF cap ─── Low-pass filter ─┤
  │ ADC2────┼─────┤     │                                    │
  │ GPIO39  │     └──┬──┘                                    │
  │         │        │                                       │
  │         │       GND                                      │
  │         │                                                │
  └────┬────┘                                                │
       │                                                     │
      GND ←──────────────────────────────────────────────────┘

Low-pass filter (per channel):
    IF Out ──── R (1kΩ) ──── ADC
                      │
                      C (1μF)
                      │
                     GND
```

### 5.3 Assembly Steps

**1. Prepare Module:**
```
- Handle carefully! RF components are sensitive
- Don't touch antenna patch area
- Ground yourself before handling
```

**2. Connect Power:**
```
BGT24 Pin 1 → +5V (from ESP32 VIN or external supply)
BGT24 Pin 2 → GND
```

**3. Connect Control (Chirp Generation):**
```
ESP32 DAC1 (GPIO25) → BGT24 Pin 3 (V_tune)
```

**4. Connect IF Outputs:**
```
BGT24 Pin 4 (IF1) → Low-pass filter → ESP32 ADC1 (GPIO36)
BGT24 Pin 5 (IF2) → Low-pass filter → ESP32 ADC2 (GPIO39)  (optional)
```

**5. Mechanical:**
```
- Mount module on non-metallic surface
- Keep antennas unobstructed
- TX and RX antennas should be separated
```

### 5.4 Chirp Generation

```cpp
// ESP32 code for FMCW chirp generation

#include <Arduino.h>
#include <driver/dac.h>

#define DAC_CHANNEL DAC_CHANNEL_1  // GPIO25

// FMCW parameters
const float T_chirp = 0.001;  // 1 ms chirp
const int chirp_samples = 1000;
const float f_min = 0.0;  // Minimum frequency (V_tune = 0V)
const float f_max = 3.3;  // Maximum frequency (V_tune = 3.3V)

uint8_t chirp_lut[chirp_samples];

void generateChirpLUT() {
    // Pre-calculate chirp waveform
    for (int i = 0; i < chirp_samples; i++) {
        float t = (float)i / chirp_samples;
        float voltage = f_min + (f_max - f_min) * t;  // Linear chirp
        chirp_lut[i] = (uint8_t)(voltage * 255 / 3.3);  // Convert to 8-bit DAC
    }
}

void setup() {
    Serial.begin(115200);
    dac_output_enable(DAC_CHANNEL);
    generateChirpLUT();
}

void loop() {
    // Output chirp
    for (int i = 0; i < chirp_samples; i++) {
        dac_output_voltage(DAC_CHANNEL, chirp_lut[i]);
        delayMicroseconds(1);  // 1 MHz sample rate → 1 ms chirp
    }

    // TODO: During chirp, simultaneously read ADC
    // This requires DMA or interrupts for precise timing
}
```

### 5.5 Data Acquisition and Processing

```python
# fmcw_radar_processing.py

import numpy as np
from scipy import signal, fft
import matplotlib.pyplot as plt

class FMCWRadar:
    def __init__(self, B=250e6, T_chirp=1e-3, fs=100e3):
        self.B = B  # Bandwidth (Hz)
        self.T_chirp = T_chirp  # Chirp duration (s)
        self.fs = fs  # Sampling rate (Hz)
        self.c = 3e8  # Speed of light
        self.f_carrier = 24e9  # Carrier frequency

    def range_from_frequency(self, f_beat):
        """Calculate range from beat frequency"""
        R = (f_beat * self.c * self.T_chirp) / (2 * self.B)
        return R

    def velocity_from_doppler(self, f_doppler):
        """Calculate velocity from Doppler shift"""
        v = (f_doppler * self.c) / (2 * self.f_carrier)
        return v

    def process_chirp(self, adc_data):
        """Process one chirp to extract range"""
        # Remove DC
        adc_data = adc_data - np.mean(adc_data)

        # Window
        window = signal.hann(len(adc_data))
        windowed = adc_data * window

        # FFT
        fft_result = fft.fft(windowed)
        fft_mag = np.abs(fft_result[:len(fft_result)//2])
        fft_freq = fft.fftfreq(len(adc_data), 1/self.fs)[:len(adc_data)//2]

        # Convert frequency to range
        ranges = self.range_from_frequency(fft_freq)

        return ranges, fft_mag

    def process_multiple_chirps(self, chirp_data):
        """
        Process multiple chirps for range-Doppler map
        chirp_data: 2D array [n_chirps, samples_per_chirp]
        """
        n_chirps, n_samples = chirp_data.shape

        # Range FFT for each chirp
        range_fft = np.zeros((n_chirps, n_samples//2), dtype=complex)
        for i in range(n_chirps):
            windowed = chirp_data[i] * signal.hann(n_samples)
            fft_result = fft.fft(windowed)
            range_fft[i] = fft_result[:n_samples//2]

        # Doppler FFT across chirps (for each range bin)
        range_doppler = np.zeros((n_chirps//2, n_samples//2))
        for r in range(n_samples//2):
            doppler_fft = fft.fft(range_fft[:, r])
            range_doppler[:, r] = np.abs(doppler_fft[:n_chirps//2])

        return range_doppler

# Example usage
radar = FMCWRadar(B=200e6, T_chirp=1e-3, fs=100e3)

# Simulate received signal (target at 10m, velocity 5 m/s)
t = np.arange(0, 1e-3, 1/100e3)
R_target = 10  # m
v_target = 5   # m/s

# Beat frequency for range
f_beat = 2 * R_target * radar.B / (radar.c * radar.T_chirp)

# Doppler shift
f_doppler = 2 * v_target * radar.f_carrier / radar.c

# Simulated signal
signal_sim = np.cos(2*np.pi*(f_beat + f_doppler)*t)
signal_sim += 0.1 * np.random.randn(len(t))  # Add noise

# Process
ranges, magnitude = radar.process_chirp(signal_sim)

# Plot
plt.figure(figsize=(12, 5))

plt.subplot(1, 2, 1)
plt.plot(t*1000, signal_sim)
plt.xlabel('Time (ms)')
plt.ylabel('Amplitude')
plt.title('IF Signal (one chirp)')
plt.grid(True)

plt.subplot(1, 2, 2)
plt.plot(ranges, magnitude)
plt.xlabel('Range (m)')
plt.ylabel('Magnitude')
plt.title('Range Profile')
plt.xlim([0, 50])
plt.grid(True)
plt.axvline(x=R_target, color='r', linestyle='--', label=f'Target at {R_target}m')
plt.legend()

plt.tight_layout()
plt.savefig('fmcw_range_profile.png')
plt.show()

print(f"Detected range: {ranges[np.argmax(magnitude)]:.2f} m")
print(f"Actual range: {R_target} m")
```

---

## 6. Option 3: Custom FMCW (Advanced)

**Note:** This is a complex RF design project requiring:
- RF PCB design skills
- Vector Network Analyzer (VNA) for tuning
- Understanding of microwave engineering
- Significant time and budget

**Block Diagram:**

```
┌─────────┐     ┌─────────┐     ┌──────────┐
│ Voltage │ ──► │   VCO   │ ──► │ Power    │ ──► TX Antenna
│ Ramp    │     │ 24 GHz  │     │ Divider  │          ↓
│Generator│     └─────────┘     └────┬─────┘     Transmitted
└─────────┘                          │           Signal
                                     │
                                     ↓ LO               Target
                              ┌──────────┐               ↓
   RX Antenna ──► ┌─────┐ ──►│  Mixer   │           Reflected
                  │ LNA │    └────┬─────┘           Signal
                  └─────┘         │
                                  ↓ IF
                          ┌───────────────┐
                          │  Low-pass     │
                          │  Filter       │
                          └───────┬───────┘
                                  │
                                  ↓
                              ┌───────┐
                              │  ADC  │ ──► Processing
                              └───────┘
```

**Key Components:**
- VCO: Analog Devices HMC512 or similar
- Mixer: Mini-Circuits ZX05-series
- LNA: Mini-Circuits PSA4-series
- Antennas: PCB patch antennas or waveguide horns

**This is beyond scope of this guide.** See advanced RF design resources.

---

## 7. Software Implementation

### Complete Python Application

```python
# radar_tracker.py - Complete object tracking radar

import numpy as np
from scipy import signal, fft
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation
import serial

class RadarTracker:
    def __init__(self, radar_type='fmcw'):
        self.radar_type = radar_type
        self.history = []
        self.max_history = 50

        # Kalman filter for tracking
        self.position_estimate = 0
        self.velocity_estimate = 0
        self.position_variance = 100
        self.velocity_variance = 10

    def kalman_update(self, measured_position, measured_velocity, dt):
        """Simple Kalman filter for smoothing"""
        # Prediction
        predicted_pos = self.position_estimate + self.velocity_estimate * dt
        predicted_vel = self.velocity_estimate

        # Update with measurement
        K_pos = self.position_variance / (self.position_variance + 1)
        K_vel = self.velocity_variance / (self.velocity_variance + 1)

        self.position_estimate = predicted_pos + K_pos * (measured_position - predicted_pos)
        self.velocity_estimate = predicted_vel + K_vel * (measured_velocity - predicted_vel)

        # Update variance
        self.position_variance = (1 - K_pos) * self.position_variance + 0.1
        self.velocity_variance = (1 - K_vel) * self.velocity_variance + 0.1

        return self.position_estimate, self.velocity_estimate

    def track(self, range_m, velocity_ms, dt=0.1):
        """Track object over time"""
        # Kalman filter
        pos_filtered, vel_filtered = self.kalman_update(range_m, velocity_ms, dt)

        # Add to history
        self.history.append({
            'time': len(self.history) * dt,
            'range': range_m,
            'velocity': velocity_ms,
            'range_filtered': pos_filtered,
            'velocity_filtered': vel_filtered
        })

        if len(self.history) > self.max_history:
            self.history.pop(0)

        return pos_filtered, vel_filtered

# Usage example
tracker = RadarTracker('fmcw')

# Simulation: ball moving toward radar
t = np.linspace(0, 5, 50)
true_range = 20 - 5*t  # Starts at 20m, approaches at 5 m/s
true_velocity = -5 * np.ones_like(t)

# Add measurement noise
measured_range = true_range + 0.5*np.random.randn(len(t))
measured_velocity = true_velocity + 0.3*np.random.randn(len(t))

# Track
filtered_range = []
filtered_velocity = []

for i in range(len(t)):
    r_filt, v_filt = tracker.track(measured_range[i], measured_velocity[i], 0.1)
    filtered_range.append(r_filt)
    filtered_velocity.append(v_filt)

# Plot
fig, axes = plt.subplots(2, 1, figsize=(10, 8))

axes[0].plot(t, true_range, 'k-', linewidth=2, label='True')
axes[0].plot(t, measured_range, 'b.', alpha=0.5, label='Measured')
axes[0].plot(t, filtered_range, 'r-', linewidth=2, label='Filtered (Kalman)')
axes[0].set_ylabel('Range (m)')
axes[0].set_title('Object Tracking with Kalman Filter')
axes[0].legend()
axes[0].grid(True)

axes[1].plot(t, true_velocity, 'k-', linewidth=2, label='True')
axes[1].plot(t, measured_velocity, 'b.', alpha=0.5, label='Measured')
axes[1].plot(t, filtered_velocity, 'r-', linewidth=2, label='Filtered (Kalman)')
axes[1].set_xlabel('Time (s)')
axes[1].set_ylabel('Velocity (m/s)')
axes[1].legend()
axes[1].grid(True)

plt.tight_layout()
plt.savefig('radar_tracking_kalman.png')
plt.show()
```

---

## 8. Testing and Calibration

### Initial Tests

**1. Power-On Test:**
```
- Connect power
- Check current draw (should be < 200mA typically)
- Verify no overheating
- Check for RF transmission (use RF detector or another radio)
```

**2. Signal Quality:**
```
- Connect oscilloscope to IF output
- Should see low-frequency signal when target present
- Amplitude should increase with closer target
```

**3. Baseline Measurement:**
```
- Point radar at empty space (no targets)
- Record baseline spectrum
- This is your noise floor
```

### Calibration Steps

**Range Calibration:**
```python
# Place known targets at known distances
calibration_points = [
    (1.0, measured_frequency_1),
    (2.0, measured_frequency_2),
    (5.0, measured_frequency_3),
    (10.0, measured_frequency_4),
]

# Fit calibration curve
distances = [p[0] for p in calibration_points]
frequencies = [p[1] for p in calibration_points]

# Should be linear for FMCW
slope = np.polyfit(frequencies, distances, 1)[0]

print(f"Calibration: Range = {slope:.4f} * frequency")
```

### Performance Tests

**1. Detection Range:**
```
- Test with standard targets (metal sphere, corner reflector)
- Measure maximum reliable detection range
- Document environmental conditions
```

**2. Velocity Accuracy:**
```
- Use moving target (car, person walking)
- Compare with known velocity (GPS, speedometer)
- Calculate error percentage
```

**3. Resolution:**
```
- Place two targets close together
- Find minimum separation for resolving both
```

---

## Safety and Regulations

**FCC Regulations (USA):**
- Part 15 limits for unlicensed operation
- K-band (24 GHz) allowed for low-power operation
- Keep transmit power < 10 mW

**Safety:**
- RF exposure limits (SAR)
- Don't point at people's eyes (especially 77 GHz)
- Proper grounding to avoid ESD damage

**Testing:**
- Test in RF-shielded environment if possible
- Or outdoors, away from interference
- Document test setup

---

## Troubleshooting

**No signal detected:**
- Check power connections
- Verify antenna not blocked
- Check IF amplifier
- Try high-RCS target (metal plate)

**Noise/false detections:**
- Improve shielding
- Add low-pass filtering
- Increase detection threshold
- Check for EMI sources

**Incorrect range:**
- Calibrate with known targets
- Check chirp linearity
- Verify ADC sampling rate
- Temperature can affect VCO!

---

## Next Steps

After building, you can:
1. **Add features:** GUI, data logging, alerts
2. **Improve algorithms:** CFAR detection, multi-target tracking
3. **Integrate:** Mount on quadcopter for obstacle avoidance
4. **Machine learning:** Train classifier for target types

---

**Related Modules:**
- [R7: DSP Fundamentals](./R7-dsp-fundamentals.md)
- [R10: Tracking Algorithms](./R10-tracking-algorithms.md)
- [R12: Signal Processing Code](./R12-signal-processing-code.md)

---

*Last Updated: November 2025*
