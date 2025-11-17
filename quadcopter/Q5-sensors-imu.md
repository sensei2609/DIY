# Q5: Sensors & IMU

> **Understanding and implementing inertial measurement units and other sensors**

---

## Table of Contents

1. [IMU Overview](#1-imu-overview)
2. [Gyroscopes](#2-gyroscopes)
3. [Accelerometers](#3-accelerometers)
4. [Magnetometers](#4-magnetometers)
5. [Barometers](#5-barometers)
6. [GPS](#6-gps)
7. [Sensor Calibration](#7-sensor-calibration)
8. [Practical Implementation](#8-practical-implementation)

---

## 1. IMU Overview

### 1.1 What is an IMU?

**Inertial Measurement Unit** = Combination of sensors measuring motion:

```
IMU (6-DOF) = 3-axis Gyroscope + 3-axis Accelerometer
IMU (9-DOF) = 6-DOF + 3-axis Magnetometer
IMU (10-DOF) = 9-DOF + Barometer
```

**Common IMU chips for DIY:**

| Chip | Axes | Interface | Cost | Notes |
|------|------|-----------|------|-------|
| MPU6050 | 6-DOF | I2C | $2-5 | Most popular, good for learning |
| MPU9250 | 9-DOF | I2C/SPI | $5-10 | Includes magnetometer |
| BMI088 | 6-DOF | I2C/SPI | $8-15 | High performance, separate gyro/accel |
| ICM-20602 | 6-DOF | I2C/SPI | $5-8 | Low noise, fast |
| LSM6DS3 | 6-DOF | I2C/SPI | $3-6 | Low power |

**For flight controllers, 6-DOF is sufficient. 9-DOF adds heading reference.**

### 1.2 Coordinate Systems

```
IMU Body Frame:
  X = Forward (nose)
  Y = Right (right wing)
  Z = Down

Standard orientation:
  - IMU arrow points forward
  - IMU flat on frame
  - Z-axis points down (important!)
```

```python
import numpy as np
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D

# Visualize IMU axes
fig = plt.figure(figsize=(10, 8))
ax = fig.add_subplot(111, projection='3d')

# Origin
origin = [0, 0, 0]

# Axes
X_axis = [1, 0, 0]
Y_axis = [0, 1, 0]
Z_axis = [0, 0, -1]  # Down!

# Plot
ax.quiver(*origin, *X_axis, color='r', arrow_length_ratio=0.15, linewidth=3, label='X (Forward)')
ax.quiver(*origin, *Y_axis, color='g', arrow_length_ratio=0.15, linewidth=3, label='Y (Right)')
ax.quiver(*origin, *Z_axis, color='b', arrow_length_ratio=0.15, linewidth=3, label='Z (Down)')

# Label
ax.text(1.2, 0, 0, 'Forward', fontsize=12)
ax.text(0, 1.2, 0, 'Right', fontsize=12)
ax.text(0, 0, -1.2, 'Down', fontsize=12)

ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_zlabel('Z')
ax.set_title('IMU Body Frame Coordinate System')
ax.legend()
plt.savefig('imu_coordinates.png')
```

---

## 2. Gyroscopes

### 2.1 What Gyroscopes Measure

**Output:** Angular velocity (rotation rate) in °/s or rad/s

```
ωx = Roll rate (about X-axis)
ωy = Pitch rate (about Y-axis)
ωz = Yaw rate (about Z-axis)
```

**NOT angles!** Must integrate to get angles.

### 2.2 MEMS Gyroscope Technology

**How it works (simplified):**
- Vibrating proof mass
- Coriolis force when rotated
- Capacitive sensing of displacement

**Key Specifications:**

```python
# MPU6050 Specifications
specs_mpu6050 = {
    'Range': '±250, ±500, ±1000, ±2000 °/s',
    'Sensitivity': {
        250: 131,    # LSB/(°/s)
        500: 65.5,
        1000: 32.8,
        2000: 16.4
    },
    'Noise': '0.005 °/s/√Hz',
    'Sample Rate': 'Up to 8 kHz',
    'Bandwidth': '5-256 Hz (configurable)',
}

# Example: Convert raw ADC to °/s
def raw_to_degrees_per_sec(raw_value, range_dps=250):
    sensitivity = specs_mpu6050['Sensitivity'][range_dps]
    return raw_value / sensitivity

# If ADC reads 1310 at ±250°/s range:
raw = 1310
rate = raw_to_degrees_per_sec(raw, 250)
print(f"Raw ADC: {raw}")
print(f"Angular rate: {rate:.2f} °/s")
```

### 2.3 Gyroscope Errors

**1. Bias (Offset)**
- Constant offset when stationary
- Temperature dependent
- Must calibrate at startup

```python
def calibrate_gyro(num_samples=1000):
    """Collect samples when stationary, calculate bias"""
    gyro_x_samples = []
    gyro_y_samples = []
    gyro_z_samples = []

    for i in range(num_samples):
        gx, gy, gz = read_gyro_raw()  # Your read function
        gyro_x_samples.append(gx)
        gyro_y_samples.append(gy)
        gyro_z_samples.append(gz)
        time.sleep(0.001)  # 1ms between samples

    # Calculate average bias
    bias_x = np.mean(gyro_x_samples)
    bias_y = np.mean(gyro_y_samples)
    bias_z = np.mean(gyro_z_samples)

    print(f"Gyro bias: X={bias_x:.3f}, Y={bias_y:.3f}, Z={bias_z:.3f} °/s")

    return bias_x, bias_y, bias_z

# Use bias
def get_corrected_gyro(bias_x, bias_y, bias_z):
    gx_raw, gy_raw, gz_raw = read_gyro_raw()

    gx = gx_raw - bias_x
    gy = gy_raw - bias_y
    gz = gz_raw - bias_z

    return gx, gy, gz
```

**2. Drift**
- Integration error accumulates
- Temperature changes cause bias shift
- Typical: 0.1-1 °/s bias → 360° drift in 6-60 minutes

**Solution:** Sensor fusion with accelerometer!

**3. Noise**
- White noise, always present
- Can be filtered

```python
# Low-pass filter for gyro
class LowPassFilter:
    def __init__(self, alpha=0.1):
        self.alpha = alpha
        self.value = 0

    def update(self, new_value):
        self.value = self.alpha * new_value + (1 - self.alpha) * self.value
        return self.value

gyro_x_filter = LowPassFilter(alpha=0.2)

# In loop:
gx_raw = read_gyro_x()
gx_filtered = gyro_x_filter.update(gx_raw)
```

### 2.4 Integration to Angles

```cpp
// C++ integration example
float roll_angle = 0;
float pitch_angle = 0;
float yaw_angle = 0;

float dt = 0.001;  // 1ms loop time

void updateAnglesFromGyro(float gx, float gy, float gz) {
    // Simple Euler integration
    roll_angle += gx * dt;
    pitch_angle += gy * dt;
    yaw_angle += gz * dt;

    // Wrap yaw to ±180°
    if (yaw_angle > 180) yaw_angle -= 360;
    if (yaw_angle < -180) yaw_angle += 360;
}
```

**Warning:** This drifts! Use complementary filter (see Q2).

---

## 3. Accelerometers

### 3.1 What Accelerometers Measure

**Output:** Specific force (acceleration minus gravity) in g or m/s²

**Key insight:** When stationary, measures gravity!

```
On flat surface:
  ax = 0
  ay = 0
  az = 1g (measures gravity pulling down)

Tilted 45° in roll:
  ax = 0
  ay = 0.707g
  az = 0.707g
```

This is how we get tilt angles!

### 3.2 Calculating Tilt from Accelerometer

```python
def accel_to_angles(ax, ay, az):
    """
    Calculate roll and pitch from accelerometer
    Assumes: Not accelerating (or slow acceleration)
    """
    # Roll (rotation about X)
    roll = np.arctan2(ay, az)

    # Pitch (rotation about Y)
    pitch = np.arctan2(-ax, np.sqrt(ay**2 + az**2))

    # Convert to degrees
    roll_deg = np.degrees(roll)
    pitch_deg = np.degrees(pitch)

    return roll_deg, pitch_deg

# Example: Tilted 30° in roll
ax = 0
ay = 0.5  # sin(30°) ≈ 0.5g
az = 0.866  # cos(30°) ≈ 0.866g

roll, pitch = accel_to_angles(ax, ay, az)
print(f"Roll: {roll:.1f}°, Pitch: {pitch:.1f}°")
```

**Limitation:** Doesn't work during acceleration!

```python
# During forward acceleration
ax_forward = -0.2  # 0.2g forward acceleration
ay = 0
az = 1.0

roll, pitch = accel_to_angles(ax_forward, ay, az)
print(f"Pitch (false): {pitch:.1f}°")  # Will show ~11° pitch up (wrong!)
```

### 3.3 Accelerometer Specifications

```python
# MPU6050 Accelerometer
accel_specs = {
    'Range': '±2g, ±4g, ±8g, ±16g',
    'Sensitivity': {
        2: 16384,   # LSB/g
        4: 8192,
        8: 4096,
        16: 2048
    },
    'Noise': '400 μg/√Hz',
    'Bandwidth': '5-260 Hz',
}

def raw_to_g(raw_value, range_g=2):
    sensitivity = accel_specs['Sensitivity'][range_g]
    return raw_value / sensitivity

# Convert to m/s²
def raw_to_ms2(raw_value, range_g=2):
    g_value = raw_to_g(raw_value, range_g)
    return g_value * 9.81
```

### 3.4 Accelerometer Calibration

**Offset calibration:**
```python
def calibrate_accel():
    """
    Place on flat surface, measure offsets
    """
    samples = 1000
    ax_sum = 0
    ay_sum = 0
    az_sum = 0

    for i in range(samples):
        ax, ay, az = read_accel_raw()
        ax_sum += ax
        ay_sum += ay
        az_sum += az
        time.sleep(0.001)

    # Average
    ax_offset = ax_sum / samples
    ay_offset = ay_sum / samples
    az_offset = az_sum / samples - 16384  # Subtract 1g (for ±2g range)

    print(f"Accel offsets: X={ax_offset}, Y={ay_offset}, Z={az_offset}")

    return ax_offset, ay_offset, az_offset
```

**Scale calibration (6-point):**
```
1. Place IMU with each axis parallel to gravity (6 orientations)
2. Record readings
3. Calculate scale factors

+X up: ax should read +1g
-X up: ax should read -1g
... repeat for Y and Z
```

---

## 4. Magnetometers

### 4.1 What Magnetometers Measure

**Output:** Earth's magnetic field in 3 axes (Gauss or μT)

**Purpose:** Determine heading (yaw angle) relative to magnetic north

```
Earth's magnetic field:
  Strength: ~0.25-0.65 Gauss (25-65 μT)
  Direction: Points north (and down in northern hemisphere)

Heading (yaw) = arctan2(my, mx)  (when level)
```

### 4.2 Magnetic Declination

Magnetic north ≠ True north!

```python
def apply_declination(magnetic_heading_deg, declination_deg):
    """
    Convert magnetic heading to true heading

    Declination examples:
      New York: -13° (magnetic north is 13° west of true north)
      Los Angeles: +12° (magnetic north is 12° east)

    Find yours at: www.magnetic-declination.com
    """
    true_heading = magnetic_heading_deg + declination_deg

    # Wrap to 0-360
    if true_heading < 0:
        true_heading += 360
    if true_heading >= 360:
        true_heading -= 360

    return true_heading

# Example: NYC
mag_heading = 45  # NE magnetic
declination = -13  # NYC
true_heading = apply_declination(mag_heading, declination)
print(f"Magnetic: {mag_heading}°, True: {true_heading}°")
```

### 4.3 Hard and Soft Iron Calibration

**Hard iron:** Permanent magnets on quadcopter (motors, magnets)
**Soft iron:** Ferromagnetic materials that distort field

**Calibration procedure:**
```python
def calibrate_magnetometer():
    """
    Rotate quadcopter in all directions for 30 seconds
    Record min/max on each axis
    """
    print("Rotate quadcopter in figure-8 pattern...")

    mx_min = mx_max = my_min = my_max = mz_min = mz_max = None

    start_time = time.time()
    samples = []

    while time.time() - start_time < 30:
        mx, my, mz = read_magnetometer()

        if mx_min is None or mx < mx_min: mx_min = mx
        if mx_max is None or mx > mx_max: mx_max = mx
        if my_min is None or my < my_min: my_min = my
        if my_max is None or my > my_max: my_max = my
        if mz_min is None or mz < mz_min: mz_min = mz
        if mz_max is None or mz > mz_max: mz_max = mz

        samples.append([mx, my, mz])
        time.sleep(0.01)

    # Calculate offsets (hard iron)
    offset_x = (mx_max + mx_min) / 2
    offset_y = (my_max + my_min) / 2
    offset_z = (mz_max + mz_min) / 2

    # Calculate scale (soft iron - simplified)
    range_x = mx_max - mx_min
    range_y = my_max - my_min
    range_z = mz_max - mz_min

    avg_range = (range_x + range_y + range_z) / 3

    scale_x = avg_range / range_x
    scale_y = avg_range / range_y
    scale_z = avg_range / range_z

    print(f"Hard iron offsets: X={offset_x:.1f}, Y={offset_y:.1f}, Z={offset_z:.1f}")
    print(f"Soft iron scales: X={scale_x:.3f}, Y={scale_y:.3f}, Z={scale_z:.3f}")

    return offset_x, offset_y, offset_z, scale_x, scale_y, scale_z

def apply_mag_calibration(mx, my, mz, cal):
    """Apply calibration"""
    offset_x, offset_y, offset_z, scale_x, scale_y, scale_z = cal

    mx_cal = (mx - offset_x) * scale_x
    my_cal = (my - offset_y) * scale_y
    mz_cal = (mz - offset_z) * scale_z

    return mx_cal, my_cal, mz_cal
```

**Warning:** Keep magnetometer away from motors and wires!

---

## 5. Barometers

### 5.1 Altitude from Air Pressure

**Barometric formula:**
```
h = 44330 × (1 - (P/P₀)^0.1903)

Where:
  h = altitude (meters)
  P = current pressure (Pa)
  P₀ = sea level pressure (101325 Pa)
```

```python
def pressure_to_altitude(pressure_pa, sea_level_pressure=101325):
    """Calculate altitude from pressure"""
    altitude = 44330 * (1 - (pressure_pa / sea_level_pressure)**0.1903)
    return altitude

# Example
P_sea_level = 101325  # Pa
P_at_1000m = 89875  # Pa (typical)

h = pressure_to_altitude(P_at_1000m)
print(f"Pressure: {P_at_1000m} Pa → Altitude: {h:.1f} m")
```

### 5.2 Common Barometer Chips

**BMP280:**
```python
bmp280_specs = {
    'Range': '300-1100 hPa',
    'Resolution': '0.16 Pa (1.3 cm altitude)',
    'Accuracy': '±1 hPa (±8m at sea level)',
    'Sample Rate': 'Up to 157 Hz',
    'Noise': '0.2 Pa RMS',
}
```

**BMP388 (better):**
- Higher accuracy: ±0.4 hPa
- Lower noise: 0.06 Pa RMS
- Better resolution: 0.02 Pa

### 5.3 Relative Altitude

For control, use **relative** altitude:

```cpp
float ground_pressure = 0;
bool is_armed = false;

void setup() {
    // Calibrate ground pressure
    float sum = 0;
    for (int i = 0; i < 100; i++) {
        sum += read_pressure();
        delay(10);
    }
    ground_pressure = sum / 100;
}

float get_relative_altitude() {
    float current_pressure = read_pressure();

    // Relative altitude
    float altitude = 44330 * (1 - pow(current_pressure / ground_pressure, 0.1903));

    return altitude;
}
```

### 5.4 Vertical Velocity from Barometer

```python
class BaroVelocityEstimator:
    def __init__(self, lpf_alpha=0.1):
        self.prev_altitude = None
        self.prev_time = None
        self.lpf_alpha = lpf_alpha
        self.velocity = 0

    def update(self, altitude, timestamp):
        if self.prev_altitude is None:
            self.prev_altitude = altitude
            self.prev_time = timestamp
            return 0

        dt = timestamp - self.prev_time

        if dt > 0:
            # Raw velocity
            raw_velocity = (altitude - self.prev_altitude) / dt

            # Low-pass filter
            self.velocity = (self.lpf_alpha * raw_velocity +
                           (1 - self.lpf_alpha) * self.velocity)

        self.prev_altitude = altitude
        self.prev_time = timestamp

        return self.velocity

# Usage
baro_vel = BaroVelocityEstimator(lpf_alpha=0.2)

altitude = get_altitude()
timestamp = time.time()
vertical_velocity = baro_vel.update(altitude, timestamp)

print(f"Altitude: {altitude:.2f}m, Climb rate: {vertical_velocity:.2f}m/s")
```

---

## 6. GPS

### 6.1 GPS Basics

**What GPS provides:**
- Latitude, Longitude (position)
- Altitude (MSL - mean sea level)
- Velocity (ground speed)
- Course (direction of travel)
- Time (very accurate!)
- Fix quality and satellite count

**Update rate:**
- Typical: 1-10 Hz
- High-performance: Up to 25 Hz

**Accuracy:**
- Horizontal: 2-5m (with WAAS/EGNOS)
- Vertical: 5-10m (less accurate than horizontal)
- Velocity: 0.1 m/s

### 6.2 Common GPS Modules

**u-blox NEO-M8N:**
```python
gps_specs = {
    'Channels': 72,
    'Update Rate': '10 Hz max',
    'Accuracy': '2.5m CEP',
    'Cold Start': '26s',
    'Hot Start': '1s',
    'Interface': 'UART (9600-460800 baud)',
}
```

**Protocol:** NMEA 0183 (text) or UBX (binary)

### 6.3 Parsing NMEA

```python
import serial

def parse_nmea_gga(sentence):
    """
    Parse GGA sentence (position)
    $GPGGA,123519,4807.038,N,01131.000,E,1,08,0.9,545.4,M,46.9,M,,*47
    """
    parts = sentence.split(',')

    if parts[0] != '$GPGGA':
        return None

    # Extract data
    time_utc = parts[1]
    lat_raw = parts[2]
    lat_dir = parts[3]
    lon_raw = parts[4]
    lon_dir = parts[5]
    fix_quality = int(parts[6])
    num_satellites = int(parts[7])
    hdop = float(parts[8]) if parts[8] else 0
    altitude = float(parts[9]) if parts[9] else 0

    # Convert lat/lon to decimal degrees
    lat_deg = int(lat_raw[:2])
    lat_min = float(lat_raw[2:])
    latitude = lat_deg + lat_min / 60
    if lat_dir == 'S':
        latitude = -latitude

    lon_deg = int(lon_raw[:3])
    lon_min = float(lon_raw[3:])
    longitude = lon_deg + lon_min / 60
    if lon_dir == 'W':
        longitude = -longitude

    return {
        'latitude': latitude,
        'longitude': longitude,
        'altitude': altitude,
        'fix_quality': fix_quality,
        'satellites': num_satellites,
        'hdop': hdop
    }

# Usage
ser = serial.Serial('/dev/ttyAMA0', 9600)

while True:
    line = ser.readline().decode('ascii', errors='ignore')

    if line.startswith('$GPGGA'):
        data = parse_nmea_gga(line)
        if data and data['fix_quality'] > 0:
            print(f"Position: {data['latitude']:.6f}, {data['longitude']:.6f}")
            print(f"Altitude: {data['altitude']:.1f}m, Sats: {data['satellites']}")
```

### 6.4 GPS Velocity

**Never differentiate position!** Too noisy.

Use GPS velocity output directly (from Doppler shift).

---

## 7. Sensor Calibration

### 7.1 Complete Calibration Procedure

**Step-by-step:**

```cpp
// calibration.h
struct CalibrationData {
    // Gyro
    float gyro_offset_x, gyro_offset_y, gyro_offset_z;

    // Accel
    float accel_offset_x, accel_offset_y, accel_offset_z;
    float accel_scale_x, accel_scale_y, accel_scale_z;

    // Mag
    float mag_offset_x, mag_offset_y, mag_offset_z;
    float mag_scale_x, mag_scale_y, mag_scale_z;

    // Baro
    float ground_pressure;
};

CalibrationData calibration;

void performCalibration() {
    Serial.println("=== CALIBRATION START ===");

    // 1. Gyro calibration
    Serial.println("Gyro: Keep STILL...");
    calibrateGyro();
    delay(1000);

    // 2. Accelerometer calibration
    Serial.println("Accel: Place on level surface...");
    calibrateAccel();
    delay(1000);

    // 3. Magnetometer calibration
    Serial.println("Mag: Rotate in figure-8 for 30s...");
    calibrateMag();
    delay(1000);

    // 4. Barometer calibration
    Serial.println("Baro: Recording ground pressure...");
    calibrateBaro();

    // Save to EEPROM
    saveCalibration();

    Serial.println("=== CALIBRATION COMPLETE ===");
}

void calibrateGyro() {
    const int samples = 1000;
    float sum_x = 0, sum_y = 0, sum_z = 0;

    for (int i = 0; i < samples; i++) {
        int16_t gx, gy, gz;
        mpu.getRotation(&gx, &gy, &gz);

        sum_x += gx;
        sum_y += gy;
        sum_z += gz;

        delay(1);
    }

    calibration.gyro_offset_x = sum_x / samples;
    calibration.gyro_offset_y = sum_y / samples;
    calibration.gyro_offset_z = sum_z / samples;

    Serial.print("Gyro offsets: ");
    Serial.print(calibration.gyro_offset_x);
    Serial.print(", ");
    Serial.print(calibration.gyro_offset_y);
    Serial.print(", ");
    Serial.println(calibration.gyro_offset_z);
}
```

### 7.2 Storage in EEPROM

```cpp
#include <EEPROM.h>

#define EEPROM_CAL_ADDR 0
#define CAL_MAGIC 0xCAFE

void saveCalibration() {
    EEPROM.put(EEPROM_CAL_ADDR, CAL_MAGIC);
    EEPROM.put(EEPROM_CAL_ADDR + 2, calibration);

    Serial.println("Calibration saved to EEPROM");
}

bool loadCalibration() {
    uint16_t magic;
    EEPROM.get(EEPROM_CAL_ADDR, magic);

    if (magic == CAL_MAGIC) {
        EEPROM.get(EEPROM_CAL_ADDR + 2, calibration);
        Serial.println("Calibration loaded from EEPROM");
        return true;
    }

    Serial.println("No calibration found - using defaults");
    return false;
}
```

---

## 8. Practical Implementation

### 8.1 Complete IMU Reading Function

```cpp
// imu.cpp
#include <Wire.h>
#include <Adafruit_MPU6050.h>
#include <Adafruit_BMP280.h>

Adafruit_MPU6050 mpu;
Adafruit_BMP280 bmp;

struct IMUData {
    float gyro_x, gyro_y, gyro_z;      // °/s
    float accel_x, accel_y, accel_z;   // m/s²
    float roll, pitch, yaw;             // degrees
    float altitude;                     // meters (relative)
    float temperature;                  // °C
};

IMUData imu_data;

bool initIMU() {
    // Initialize I2C
    Wire.begin();
    Wire.setClock(400000);  // 400kHz fast mode

    // Initialize MPU6050
    if (!mpu.begin()) {
        Serial.println("MPU6050 not found!");
        return false;
    }

    // Configure MPU6050
    mpu.setAccelerometerRange(MPU6050_RANGE_8_G);
    mpu.setGyroRange(MPU6050_RANGE_500_DEG);
    mpu.setFilterBandwidth(MPU6050_BAND_21_HZ);

    // Initialize BMP280
    if (!bmp.begin()) {
        Serial.println("BMP280 not found!");
        return false;
    }

    // Configure BMP280
    bmp.setSampling(Adafruit_BMP280::MODE_NORMAL,
                    Adafruit_BMP280::SAMPLING_X2,
                    Adafruit_BMP280::SAMPLING_X16,
                    Adafruit_BMP280::FILTER_X16,
                    Adafruit_BMP280::STANDBY_MS_1);

    Serial.println("IMU initialized successfully");
    return true;
}

void readIMU() {
    // Read MPU6050
    sensors_event_t a, g, temp;
    mpu.getEvent(&a, &g, &temp);

    // Gyro (rad/s to °/s, with calibration)
    imu_data.gyro_x = (g.gyro.x * 57.2958) - calibration.gyro_offset_x;
    imu_data.gyro_y = (g.gyro.y * 57.2958) - calibration.gyro_offset_y;
    imu_data.gyro_z = (g.gyro.z * 57.2958) - calibration.gyro_offset_z;

    // Accel (m/s², with calibration)
    imu_data.accel_x = (a.acceleration.x - calibration.accel_offset_x) * calibration.accel_scale_x;
    imu_data.accel_y = (a.acceleration.y - calibration.accel_offset_y) * calibration.accel_scale_y;
    imu_data.accel_z = (a.acceleration.z - calibration.accel_offset_z) * calibration.accel_scale_z;

    // Read BMP280
    float pressure = bmp.readPressure();
    imu_data.altitude = 44330 * (1 - pow(pressure / calibration.ground_pressure, 0.1903));
    imu_data.temperature = bmp.readTemperature();
}

void updateAttitude(float dt) {
    // Complementary filter (see Q2-control-systems.md)

    // Accel angles
    float accel_roll = atan2(imu_data.accel_y, imu_data.accel_z) * 57.2958;
    float accel_pitch = atan2(-imu_data.accel_x,
                               sqrt(imu_data.accel_y*imu_data.accel_y +
                                    imu_data.accel_z*imu_data.accel_z)) * 57.2958;

    // Gyro integration
    imu_data.roll += imu_data.gyro_x * dt;
    imu_data.pitch += imu_data.gyro_y * dt;
    imu_data.yaw += imu_data.gyro_z * dt;

    // Complementary filter (98% gyro, 2% accel)
    float alpha = 0.98;
    imu_data.roll = alpha * imu_data.roll + (1 - alpha) * accel_roll;
    imu_data.pitch = alpha * imu_data.pitch + (1 - alpha) * accel_pitch;

    // Yaw from gyro only (no correction without mag)
}
```

### 8.2 Main Loop Integration

```cpp
void loop() {
    static unsigned long last_time = micros();
    unsigned long current_time = micros();
    float dt = (current_time - last_time) / 1000000.0;
    last_time = current_time;

    // Read sensors
    readIMU();

    // Update attitude estimate
    updateAttitude(dt);

    // Log data (every 100ms for debugging)
    static unsigned long last_log = 0;
    if (millis() - last_log > 100) {
        Serial.print("Roll: "); Serial.print(imu_data.roll);
        Serial.print(" Pitch: "); Serial.print(imu_data.pitch);
        Serial.print(" Yaw: "); Serial.print(imu_data.yaw);
        Serial.print(" Alt: "); Serial.println(imu_data.altitude);

        last_log = millis();
    }

    // Control loop (see Q8-flight-controller-code.md)
    // ...

    // Maintain loop timing
    while (micros() - current_time < 1000) {
        // Wait for 1ms loop time
    }
}
```

---

## Summary

You now understand:

✅ **IMU components** - Gyro, accel, mag, baro, GPS
✅ **Sensor physics** - What each measures and limitations
✅ **Calibration** - Offset, scale, hard/soft iron
✅ **Data fusion** - Complementary filter basics
✅ **Implementation** - Complete working code

**Key Takeaways:**

1. Gyros drift - must fuse with accel
2. Accels measure gravity + acceleration
3. Magnetometers need careful calibration
4. Barometers for relative altitude
5. GPS for position, but use its velocity output

---

**Next Steps:**

- **[Q8: Flight Controller Code](./Q8-flight-controller-code.md)** - Put it all together
- **[Q12: Testing & Tuning](./Q12-testing-tuning.md)** - Calibration in practice

---

*Last Updated: November 2025*
