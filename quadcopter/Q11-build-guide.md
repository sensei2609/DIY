# Q11: Quadcopter Build Guide

> **Complete step-by-step guide to building your DIY quadcopter**

---

## Table of Contents

1. [Parts List](#1-parts-list)
2. [Tools Required](#2-tools-required)
3. [Frame Assembly](#3-frame-assembly)
4. [Electronic Assembly](#4-electronic-assembly)
5. [Wiring Diagram](#5-wiring-diagram)
6. [Software Setup](#6-software-setup)
7. [Pre-Flight Checklist](#7-pre-flight-checklist)

---

## 1. Parts List

### Option A: Budget Build (~$150-200)

**Frame:**
- [ ] 250mm quadcopter frame kit (carbon fiber or fiberglass) - $25-40
  - Recommended: ZMR250, QAV250, or similar X-frame

**Motors:**
- [ ] 4x brushless motors (2204-2206 size, 2300-2600KV) - $40-60
  - Recommended: Emax RS2205 2300KV or similar

**ESCs:**
- [ ] 4x ESC (20-30A, BLHeli firmware) - $30-50
  - Or: 4-in-1 ESC board - $35-45

**Flight Controller:**
- [ ] Flight controller board - $20-40
  - **Recommended for DIY:** Arduino compatible boards:
    - ESP32 DevKit ($8-12) + MPU6050 ($3-5)
    - STM32F4 development board ($15-25) + MPU6050
  - **Alternative (easier):** Pre-made flight controllers:
    - Matek F405 or F722
    - SpeedyBee F4/F7

**Propellers:**
- [ ] 4x propellers (5-6 inch, matching motor specs) - $5-10
  - Buy extras! They will break.
  - 2x CW, 2x CCW

**Battery:**
- [ ] LiPo battery (3S or 4S, 1300-1800mAh, 45C+) - $20-35
  - 3S = 11.1V nominal
  - 4S = 14.8V nominal (more power, shorter flight time)
  - Buy 2-3 batteries for longer flight sessions

**Charger:**
- [ ] LiPo balance charger - $15-30
  - IMAX B6 or similar

**Receiver:**
- [ ] RC receiver (compatible with your transmitter) - $15-25
  - FlySky FS-iA6B (budget)
  - FrSky XM+ (better)

**Transmitter:**
- [ ] RC transmitter (if you don't have one) - $40-80
  - FlySky FS-i6X (budget, good for beginners)
  - Radiomaster TX16S (advanced)

**Power Distribution:**
- [ ] Power distribution board (PDB) - $5-10
  - Or use 4-in-1 ESC with integrated PDB

**Miscellaneous:**
- [ ] XT60 connectors (battery connector) - $2-5
- [ ] Battery strap - $3-5
- [ ] M3 screws/standoffs kit - $5-10
- [ ] Heat shrink tubing - $3-5
- [ ] Zip ties - $2-3
- [ ] Double-sided tape (for mounting FC) - $2-3

### Option B: DIY from Scratch Build (~$80-120)

Build your own flight controller!

**Core Components:**
- [ ] ESP32 or STM32F4 development board - $10-20
- [ ] MPU6050 (gyro + accel) - $3-5
- [ ] BMP280 (barometer) - $3-5
- [ ] HMC5883L or QMC5883L (compass) - $3-5
- [ ] Prototype PCB or custom PCB - $5-15
- [ ] Voltage regulator (5V BEC) - $3-5
- [ ] Headers, wire, connectors - $10-15

**Plus all other parts from Option A** (motors, ESCs, frame, etc.)

### Budget Breakdown

```
Frame:           $30
Motors (4x):     $50
ESCs (4x):       $40
Flight Controller: $15 (DIY) or $30 (pre-made)
Props:           $10
Battery (2x):    $50
Charger:         $20
Receiver:        $20
Transmitter:     $60
Misc:            $20
─────────────────────
Total:           $315 (complete kit)
                 $255 (if you have TX)
                 $155 (if DIY FC + have TX)
```

---

## 2. Tools Required

### Essential:
- [ ] Soldering iron (adjustable temp, 60-80W recommended)
- [ ] Solder (60/40 or lead-free)
- [ ] Wire cutters/strippers
- [ ] Small Phillips screwdrivers
- [ ] Hex key set (metric)
- [ ] Multimeter
- [ ] Heat gun or lighter (for heat shrink)

### Helpful:
- [ ] Helping hands / PCB holder
- [ ] Desoldering pump/braid
- [ ] Wire crimpers
- [ ] Hot glue gun
- [ ] Electrical tape
- [ ] Isopropyl alcohol (cleaning flux)

### Safety:
- [ ] Safety glasses (required when soldering and flying!)
- [ ] Fire-resistant work surface
- [ ] LiPo safety bag
- [ ] Fire extinguisher nearby

---

## 3. Frame Assembly

### Step 1: Identify Parts

```
Typical frame kit contains:
- 2x carbon fiber plates (top and bottom)
- 4x arms (carbon fiber tubes or plates)
- 4x motor mounts
- Standoffs and screws
- Power distribution board (sometimes included)
```

### Step 2: Assemble Frame

```
1. Attach arms to bottom plate
   - Use thread locker on screws (blue Loctite)
   - Don't overtighten! Carbon fiber can crack

2. Install standoffs
   - 4 corners for mounting top plate
   - Additional for mounting flight controller

3. Mount motors to arms
   - Motor shaft should point UP
   - Check rotation direction markings
   - Motor 1 (front-right): CW rotation
   - Motor 2 (back-left): CW rotation
   - Motor 3 (front-left): CCW rotation
   - Motor 4 (back-right): CCW rotation

4. Test fit all components before final assembly
   - Flight controller location
   - ESC placement
   - Battery mounting
   - Receiver antenna routing
```

### Motor Positions (X-Configuration)

```
         FRONT
           ↑

     M3 ⊙      ⊗ M1
        \  ╬  /
         \ | /
      CCW \|/ CW

         / | \
        /  ╬  \
     M4 ⊗      ⊙ M2
      CW        CCW
```

---

## 4. Electronic Assembly

### Step 1: ESC to Motor Connections

```
Each ESC has 3 wires to motor:
- Connect any 3 ESC wires to 3 motor wires
- Test motor direction (connect battery, apply throttle)
- If motor spins WRONG direction:
  → Swap ANY TWO of the three wires
```

**Testing Motor Direction:**

```cpp
// Arduino sketch to test motors one at a time
#include <Servo.h>

Servo esc1, esc2, esc3, esc4;

void setup() {
    Serial.begin(115200);
    esc1.attach(5);  // GPIO pins for ESC signal
    esc2.attach(18);
    esc3.attach(19);
    esc4.attach(21);

    // Send minimum throttle (ESC arming)
    esc1.writeMicroseconds(1000);
    esc2.writeMicroseconds(1000);
    esc3.writeMicroseconds(1000);
    esc4.writeMicroseconds(1000);

    delay(3000);  // Wait 3 seconds
    Serial.println("Ready. Type 1-4 to test motor");
}

void loop() {
    if (Serial.available()) {
        char cmd = Serial.read();
        int throttle = 1200;  // Low throttle for testing

        // Test motors individually
        if (cmd == '1') {
            Serial.println("Testing Motor 1 (should spin CW)");
            esc1.writeMicroseconds(throttle);
            delay(2000);
            esc1.writeMicroseconds(1000);
        }
        // ... similar for motors 2, 3, 4
    }
}
```

### Step 2: Power Distribution

**Wiring Diagram:**

```
                    BATTERY
                    |
                 XT60 CONNECTOR
                    |
        ┌───────────┴───────────┐
        |    POWER DISTRO BOARD  |
        |                        |
        |  +12V (or 4S)    GND   |
        └─┬────┬────┬────┬───────┘
          |    |    |    |
        ESC1 ESC2 ESC3 ESC4
          |    |    |    |
         M1   M2   M3   M4

    5V Regulator (BEC)
          |
    Flight Controller + Receiver
```

**Current Draw:**

```
Hover: ~10-15A total
Max:   ~60-80A total (4x 20A ESCs)

Use wire gauge:
- 12-14 AWG for battery to PDB
- 18-20 AWG for PDB to ESCs
- 22-24 AWG for signal wires
```

### Step 3: Flight Controller Connections

**Pinout (Example for ESP32-based DIY controller):**

```
ESP32 Pin  | Connection
-----------+---------------------------
GPIO 5     | ESC 1 signal (PWM)
GPIO 18    | ESC 2 signal
GPIO 19    | ESC 3 signal
GPIO 21    | ESC 4 signal
GPIO 22    | I2C SCL (IMU)
GPIO 21    | I2C SDA (IMU)
GPIO 16    | RX from receiver CH1
GPIO 17    | RX from receiver CH2
(etc)      | ... more RX channels
3V3        | IMU VCC, Receiver VCC
GND        | Common ground
VIN (5V)   | From BEC
```

**IMU Mounting:**
- Mount IMU flat, aligned with frame
- Arrow on IMU should point FORWARD
- Use foam tape to reduce vibrations
- Keep away from motors (magnetic interference)

### Step 4: Receiver Wiring

**PWM Receiver (Traditional):**
```
Receiver          Flight Controller
─────────         ─────────────────
CH1 (Roll)     → GPIO 16
CH2 (Pitch)    → GPIO 17
CH3 (Throttle) → GPIO 4
CH4 (Yaw)      → GPIO 15
CH5 (Aux)      → GPIO 13
GND            → GND
VCC (5V)       → 5V
```

**Serial Receiver (SBUS/iBUS):**
```
Receiver          Flight Controller
─────────         ─────────────────
Signal         → RX pin (UART)
GND            → GND
VCC (5V)       → 5V
```

---

## 5. Wiring Diagram

```
                    Battery (3S/4S LiPo)
                          |
                       XT60 Connector
                          |
           ┌──────────────┴──────────────┐
           |                             |
           |    Power Distribution       |
           |         Board               |
           |    (12V/14.8V + GND)        |
           └─┬────┬────┬────┬────────────┘
             |    |    |    |
           ┌─┴─┐┌─┴─┐┌─┴─┐┌─┴─┐
           |ESC||ESC||ESC||ESC|
           | 1 || 2 || 3 || 4 |
           └─┬─┘└─┬─┘└─┬─┘└─┬─┘
             |    |    |    |
          ┌──┴┐┌──┴┐┌──┴┐┌──┴┐
          | M1|| M2|| M3|| M4|  Motors
          └───┘└───┘└───┘└───┘

    5V Regulator (BEC)
          |
    ┌─────┴─────┐
    |           |
    |   Flight  |     I2C Bus
    |Controller |────────┬──────────┐
    |  (ESP32)  |        |          |
    |           |     ┌──┴───┐  ┌───┴────┐
    └─────┬─────┘     | MPU  |  | BMP280 |
          |           | 6050 |  | (Baro) |
          |           └──────┘  └────────┘
          |
    ┌─────┴──────┐
    |  Receiver  |
    |  (FlySky)  |
    └────────────┘

ESC Signal Wires → GPIO 5, 18, 19, 21
RX Channels      → GPIO 16, 17, 4, 15, 13
```

---

## 6. Software Setup

### Option 1: Custom Firmware (Full DIY)

**Development Environment:**

```bash
# Install PlatformIO
pip install platformio

# Create new project
pio init --board esp32dev

# Edit platformio.ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
lib_deps =
    adafruit/Adafruit MPU6050 @ ^2.2.4
    adafruit/Adafruit BMP280 Library @ ^2.6.6
    ESP32Servo
```

**Basic Flight Controller Code Structure:**

```cpp
// main.cpp
#include <Arduino.h>
#include <Wire.h>
#include <Adafruit_MPU6050.h>
#include <ESP32Servo.h>

// Hardware objects
Adafruit_MPU6050 mpu;
Servo esc1, esc2, esc3, esc4;

// State
struct {
    float roll, pitch, yaw;
    float roll_rate, pitch_rate, yaw_rate;
} state;

// PID controllers
class PID {
    float kp, ki, kd, integral, prev_error;
public:
    PID(float p, float i, float d) : kp(p), ki(i), kd(d), integral(0), prev_error(0) {}

    float compute(float setpoint, float measured, float dt) {
        float error = setpoint - measured;
        integral += error * dt;
        float derivative = (error - prev_error) / dt;
        prev_error = error;
        return kp*error + ki*integral + kd*derivative;
    }
};

PID roll_pid(1.5, 0.1, 0.05);
PID pitch_pid(1.5, 0.1, 0.05);
PID yaw_pid(2.0, 0.1, 0.0);

void setup() {
    Serial.begin(115200);

    // Initialize IMU
    if (!mpu.begin()) {
        Serial.println("MPU6050 not found!");
        while (1) delay(10);
    }

    // Configure IMU
    mpu.setAccelerometerRange(MPU6050_RANGE_8_G);
    mpu.setGyroRange(MPU6050_RANGE_500_DEG);
    mpu.setFilterBandwidth(MPU6050_BAND_21_HZ);

    // Initialize ESCs
    esc1.attach(5, 1000, 2000);
    esc2.attach(18, 1000, 2000);
    esc3.attach(19, 1000, 2000);
    esc4.attach(21, 1000, 2000);

    // Arm ESCs
    esc1.writeMicroseconds(1000);
    esc2.writeMicroseconds(1000);
    esc3.writeMicroseconds(1000);
    esc4.writeMicroseconds(1000);

    delay(3000);
    Serial.println("Ready!");
}

void loop() {
    static unsigned long last_time = millis();
    unsigned long current_time = millis();
    float dt = (current_time - last_time) / 1000.0;
    last_time = current_time;

    // Read IMU
    sensors_event_t a, g, temp;
    mpu.getEvent(&a, &g, &temp);

    state.roll_rate = g.gyro.x * 57.3;  // Convert to deg/s
    state.pitch_rate = g.gyro.y * 57.3;
    state.yaw_rate = g.gyro.z * 57.3;

    // Integrate for angles (simplified - use complementary filter in real code)
    state.roll += state.roll_rate * dt;
    state.pitch += state.pitch_rate * dt;

    // Read RC inputs (simplified - implement your receiver protocol)
    float throttle = 0.5;  // 0.0 to 1.0
    float roll_cmd = 0.0;  // -1.0 to 1.0
    float pitch_cmd = 0.0;
    float yaw_cmd = 0.0;

    // PID control
    float roll_output = roll_pid.compute(roll_cmd * 45, state.roll, dt);
    float pitch_output = pitch_pid.compute(pitch_cmd * 45, state.pitch, dt);
    float yaw_output = yaw_pid.compute(yaw_cmd * 180, state.yaw_rate, dt);

    // Motor mixing
    int m1 = 1000 + throttle * 1000 + roll_output - pitch_output + yaw_output;
    int m2 = 1000 + throttle * 1000 - roll_output + pitch_output - yaw_output;
    int m3 = 1000 + throttle * 1000 - roll_output - pitch_output + yaw_output;
    int m4 = 1000 + throttle * 1000 + roll_output + pitch_output - yaw_output;

    // Constrain and output
    m1 = constrain(m1, 1000, 2000);
    m2 = constrain(m2, 1000, 2000);
    m3 = constrain(m3, 1000, 2000);
    m4 = constrain(m4, 1000, 2000);

    esc1.writeMicroseconds(m1);
    esc2.writeMicroseconds(m2);
    esc3.writeMicroseconds(m3);
    esc4.writeMicroseconds(m4);

    delay(10);  // ~100 Hz loop (increase to 1000 Hz for production)
}
```

### Option 2: Betaflight (Pre-made Controllers)

1. Download Betaflight Configurator
2. Connect FC via USB
3. Flash latest Betaflight firmware
4. Configure in GUI:
   - Board orientation
   - Receiver type
   - Motor order
   - PID tuning
   - Failsafe

---

## 7. Pre-Flight Checklist

### Before First Power-On:

- [ ] Visual inspection of all solder joints
- [ ] Check for shorts with multimeter
- [ ] Verify polarity (+ and -)
- [ ] Propellers OFF!
- [ ] Battery charged and balanced

### First Power-On (Bench Test):

- [ ] Connect battery (expect ESC beeps)
- [ ] Check LED indicators
- [ ] Verify FC boots up
- [ ] Check sensor readings
- [ ] Test receiver connection
- [ ] Calibrate ESCs if needed

### Motor Test (Props OFF!):

- [ ] Arm system
- [ ] Test each motor direction
- [ ] Verify motor mapping
- [ ] Check motor direction matches:
  - M1 (FR): CW
  - M2 (BL): CW
  - M3 (FL): CCW
  - M4 (BR): CCW

### IMU Calibration:

- [ ] Place quadcopter on level surface
- [ ] Run accelerometer calibration
- [ ] Run gyro calibration
- [ ] Verify orientation in software

### RC Transmitter Setup:

- [ ] Bind receiver
- [ ] Check all channels
- [ ] Set up arming switch
- [ ] Configure flight modes
- [ ] Test failsafe

### Props-On Ground Test:

- [ ] Install propellers (correct rotation!)
- [ ] Clear area of people/objects
- [ ] Tether quadcopter (string to table)
- [ ] Slowly increase throttle
- [ ] Check for vibrations
- [ ] Verify stable hover possible

### First Flight:

- [ ] Fly in large, open area
- [ ] Calm weather (no wind)
- [ ] Have a spotter
- [ ] Start with short hover
- [ ] Test basic controls
- [ ] Land immediately if unstable

---

## Safety Rules

**ALWAYS:**
✅ Remove propellers when testing on bench
✅ Use LiPo safety bag for charging
✅ Never leave LiPo charging unattended
✅ Wear safety glasses
✅ Follow local drone laws
✅ Maintain line of sight
✅ Stay away from people/property

**NEVER:**
❌ Fly indoors (until experienced)
❌ Fly near airports
❌ Fly over people
❌ Fly in bad weather
❌ Use damaged batteries
❌ Touch spinning propellers (obviously!)

---

## Troubleshooting

**Motor won't spin:**
- Check ESC power
- Check signal wire connection
- Verify ESC calibration

**Quadcopter flips immediately:**
- Check motor directions
- Verify motor mapping
- Check propeller orientation

**Unstable hover:**
- Check CG (center of gravity) - should be centered
- Reduce PID gains
- Check for loose parts
- Verify IMU orientation

**Won't arm:**
- Check transmitter connection
- Verify failsafe settings
- Check battery voltage
- Review error messages

---

**Next:** [Q12: Testing and Tuning](./Q12-testing-tuning.md)

---

*Last Updated: November 2025*
