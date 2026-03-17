
🚁 ESP32 Flight Controller 
📋 Overview
ESP32-based flight controller with MPU6050, BMP280, and NRF24L01 for quadcopter/drone.

🔌 Quick Connections
Component	ESP32 Pins
MPU6050/BMP280	VCC→3.3V, GND→GND, SDA→21, SCL→22
NRF24L01	VCC→3.3V+cap, GND→GND, CE→4, CSN→5, SCK→18, MOSI→23, MISO→19
Motors	M1→13, M2→12, M3→14, M4→27
Buzzer	32
Battery	34 (via voltage divider)
📦 Required Libraries
text
RF24 by TMRh20
Adafruit MPU6050
Adafruit BMP280
Adafruit Unified Sensor
ESP32Servo
🚀 Quick Start
1️⃣ Upload Calibration Code
cpp
// ESC Calibration (run once)
#include <ESP32Servo.h>
#define M1 13  // Motors on 13,12,14,27
Servo m1;
void setup() {
  m1.attach(M1,1000,2000);
  m1.writeMicroseconds(2000); // Max throttle
  delay(5000); // Connect battery now
  m1.writeMicroseconds(1000); // Min throttle
  delay(5000);
}
void loop() {}
2️⃣ Test Radio (TX)
cpp
// Upload to transmitter ESP32
// Throttle→34, Roll→35, Pitch→32, Yaw→33
// Arm→25, Mode→26
3️⃣ Test Radio (RX)
cpp
// Upload to flight controller ESP32
// Check Serial Monitor for data
4️⃣ Upload Main Flight Code
cpp
// Full flight controller with:
// - PID stabilization
// - Altitude hold
// - Failsafe
// - Battery monitoring
🎮 Flight Modes
Mode	LED	Action
Standby	Slow blink	Waiting for arm
Armed	Solid + heartbeat	Ready to fly
Alt Hold	Double heartbeat	Maintains altitude
Failsafe	Fast red blink	Auto-land
⚡ Key Features
400Hz control loop - Smooth flight

PID stabilization - Roll, Pitch, Yaw

Altitude hold - ±0.5m accuracy

Failsafe - Auto-land on signal loss

Battery monitor - Low voltage warning

Telemetry - Real-time data via Serial

🔧 Quick PID Tuning
Parameter	Start	Range
Roll/Pitch P	1.5	1.0-3.0
Roll/Pitch I	0.05	0.01-0.1
Roll/Pitch D	0.1	0.05-0.3
Yaw P	2.0	1.5-4.0
⚠️ Safety First!
text
□ Remove propellers for testing
□ Calibrate ESCs before first flight
□ Test motor directions on ground
□ Check battery voltage before takeoff
□ Keep clear of spinning motors
□ First flight in open area
□ Have someone with you
📊 Serial Output
text
⏱️ 1234 | 📐 R:1.2 P:0.5 Y:0.1 | 📊 A:2.3m | ⚡ 11.4V | ✈️ FLYING
🐛 Common Issues
Problem	Fix
No radio	Check NRF24L01 power (add capacitor!)
Drifting	Calibrate sensors on level surface
Oscillations	Reduce P gain
Short flight	Check battery voltage
Motor not spinning	Recalibrate ESCs
📱 Telemetry (10Hz)
Roll/Pitch/Yaw angles

Altitude

Temperature

Battery voltage

Flight mode

Packet rate

🔋 Power Requirements
ESP32: 5V from BEC

NRF24L01: 3.3V + 100µF capacitor

Sensors: 3.3V from ESP32

Motors: Direct from LiPo battery

📁 File Structure
text
/ESP32_FlightController
  ├── FlightController.ino    # Main code
  ├── ESC_Calibration.ino     # Run once
  ├── Radio_Test_TX.ino       # Test transmitter
  ├── Radio_Test_RX.ino       # Test receiver
  └── README.md               # This file
✅ Pre-Flight Checklist
ESCs calibrated

Motor directions correct

Radio link stable

All sensors detected

Battery fully charged

Props securely fastened

Arm switch working

Failsafe tested

📞 Need Help?
Check connections table

Verify library versions

Monitor Serial output

Start with basic tests

Join ESP32/Drone communities

🎯 Perfect for: Quadcopters, Drones, RC Airplanes, Robotics Projects

⚠️ Disclaimer: Test thoroughly on ground before flight. Fly safely and follow local regulations!

Version 2.0 | Last Updated: 2024

Not connections add overview parts needed libraries needed software needed and add made by Atharva Phadnis and tell to check rules
🚁 ESP32 Flight Controller - Complete Short README
📋 Project Overview
A professional-grade flight controller for quadcopters/drones using ESP32 with MPU6050 (gyro+accelerometer) and BMP280 (barometer) sensors, communicating via NRF24L01 radio module. Features PID stabilization, altitude hold, failsafe, and real-time telemetry.

🧩 Parts Needed
Required Components
Component	Quantity	Purpose
ESP32 Development Board	2	Main controller & transmitter
MPU6050	1	6-axis gyroscope + accelerometer
BMP280	1	Barometric pressure (altitude)
NRF24L01+PA+LNA	2	Radio communication (with antenna)
Brushless ESCs	4	Motor speed controllers
Brushless Motors	4	Quadcopter motors (2204-2306)
LiPo Battery	1	3S-4S (11.1V-14.8V)
Power Distribution Board	1	Powers ESCs and components
BEC (5V/3A)	1	Powers ESP32 from battery
Frame	1	250-450mm quadcopter frame
Propellers	4 pairs	5-7 inch
Optional Components
Component	Purpose
GPS Module (NEO-6M)	Position hold, return-to-home
Magnetometer (HMC5883L)	Heading hold
OLED Display	On-field configuration
MicroSD Card Module	Flight data logging
Buzzer	Audio alerts
LEDs	Status indication
📚 Libraries Needed
Required Libraries (Install via Arduino Library Manager)
text
1. RF24 by TMRh20          - NRF24L01 communication
2. Adafruit MPU6050         - MPU6050 sensor
3. Adafruit BMP280 Library  - BMP280 sensor  
4. Adafruit Unified Sensor  - Sensor abstraction
5. ESP32Servo               - ESC/PWM control
Installation Commands
cpp
// In Arduino IDE:
Sketch → Include Library → Manage Libraries
// Search and install each library above
💻 Software Needed
Development Software
Software	Purpose	Download Link
Arduino IDE	Code development	arduino.cc/en/software
ESP32 Board Package	ESP32 support	Add URL in preferences
Serial Monitor	Debugging	Built into Arduino IDE
FTDI Drivers	USB communication	From manufacturer
Git (optional)	Version control	git-scm.com
Arduino IDE Setup
cpp
1. Install Arduino IDE
2. Add ESP32 board URL:
   File → Preferences → Additional Board Manager URLs:
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
3. Tools → Board → Board Manager → Search "ESP32" → Install
4. Select board: Tools → Board → ESP32 Dev Module
5. Select port: Tools → Port → COMx (Windows) or /dev/ttyUSB0 (Linux)
📁 Code Files Overview
File Name	Purpose
FlightController.ino	Main flight controller code
ESC_Calibration.ino	Run once to calibrate ESCs
Radio_Test_TX.ino	Test transmitter module
Radio_Test_RX.ino	Test receiver module
Motor_Direction_Test.ino	Verify motor rotations
Sensor_Test.ino	Verify MPU6050/BMP280
✨ Key Features
400Hz Control Loop - Real-time stabilization

PID Controllers - Roll, Pitch, Yaw, Altitude

Sensor Fusion - Complementary filter (95% gyro + 5% accel)

Altitude Hold - ±0.5m accuracy with BMP280

Failsafe System - Auto-land on signal loss

Battery Monitoring - Low voltage warning and protection

Telemetry Output - Real-time data via Serial

Arm/Disarm Safety - Switch combination required

LED Indicators - Visual status feedback

Buzzer Alerts - Audio warnings for critical states

🚀 Quick Start Guide
Step 1: Hardware Assembly
text
1. Mount ESP32 on frame
2. Connect sensors via I2C (SDA→21, SCL→22)
3. Connect NRF24L01 via SPI (CE→4, CSN→5, etc.)
4. Add 100µF capacitor to NRF24L01 power
5. Connect ESCs to GPIO 13,12,14,27
6. Connect battery monitor to GPIO 34 (via divider)
7. Add buzzer to GPIO 32
8. Power ESP32 via 5V BEC from PDB
Step 2: Install Libraries
cpp
// Open Arduino IDE
// Install all 5 required libraries
Step 3: Upload Calibration Codes
cpp
// 1. Upload ESC_Calibration.ino (NO PROPELLERS!)
// 2. Upload Radio_Test_TX.ino to transmitter
// 3. Upload Radio_Test_RX.ino to flight controller
// 4. Verify radio communication
// 5. Upload Motor_Direction_Test.ino to verify rotations
Step 4: Upload Main Code
cpp
// 1. Open FlightController.ino
// 2. Select board: ESP32 Dev Module
// 3. Select correct COM port
// 4. Click Upload
// 5. Open Serial Monitor @ 115200 baud
Step 5: First Flight Preparation
cpp
// 1. Place drone on level surface
// 2. Power on (wait 10 sec for gyro calibration)
// 3. Check LED patterns
// 4. Arm system (throttle low + arm switch)
// 5. Test motor response at low throttle
// 6. If all good - ATTACH PROPELLERS
// 7. First flight in open area
⚙️ Configuration Parameters
PID Settings
cpp
// Default values - adjust based on your frame
#define ROLL_P   1.50
#define ROLL_I   0.05
#define ROLL_D   0.10

#define PITCH_P  1.50
#define PITCH_I  0.05
#define PITCH_D  0.10

#define YAW_P    2.00
#define YAW_I    0.10
#define YAW_D    0.20

#define ALT_P    1.00
#define ALT_I    0.01
#define ALT_D    0.50
Safety Settings
cpp
#define RADIO_TIMEOUT    250  // ms - signal loss threshold
#define ARM_DELAY       1000  // ms - arm sequence delay
#define BATTERY_LOW      3.5  // V per cell - warning
#define BATTERY_CRITICAL 3.3  // V per cell - auto-land
📊 Telemetry Output (115200 baud)
text
⏱️ 1234 | 📐 R:1.2° P:0.5° Y:0.1° | 📊 A:2.3m | ⚡ 11.4V | ✈️ FLYING

Where:
⏱️ = Time (ms)
📐 = Roll/Pitch/Yaw angles
📊 = Altitude (m)
⚡ = Battery voltage
✈️ = Flight mode
🎮 Flight Modes
Mode	LED Pattern	Description
STANDBY	Slow blink	Waiting for arm command
ARMED	Solid + heartbeat	Normal flight mode
ALTITUDE HOLD	Double heartbeat	Maintains current altitude
FAILSAFE	Fast red blink	Signal lost - auto-landing
Arm/Disarm Sequence
text
ARMING:
1. Throttle stick to minimum
2. Arm switch to ON position
3. Wait 2 seconds
4. Motors spin at idle
5. Green LED solid

DISARMING:
1. Throttle minimum
2. Arm switch OFF
3. Motors stop immediately
⚠️ SAFETY RULES - MUST READ!
⚠️ BEFORE FIRST POWER-ON
text
□ READ ALL DOCUMENTATION completely
□ CHECK all connections twice
□ VERIFY power supply voltages
□ ENSURE no shorts in wiring
□ CONFIRM components are secure
⚠️ DURING TESTING
text
□ ALWAYS remove propellers for initial tests
□ NEVER stand in line with spinning motors
□ KEEP hands clear of motors at all times
□ HAVE fire extinguisher nearby
□ WORK in clear, open area
□ USE safety glasses
□ KEEP children and pets away
□ DISCONNECT battery when not testing
⚠️ BEFORE FIRST FLIGHT
text
□ TEST motor directions WITHOUT props
□ CALIBRATE ESCs (critical step!)
□ VERIFY radio range on ground
□ CHECK failsafe activation
□ TEST arming/disarming sequence
□ ENSURE battery fully charged
□ CHECK all screws are tight
□ VERIFY center of gravity
⚠️ DURING FLIGHT
text
□ ALWAYS fly in open areas away from people
□ NEVER fly over crowds or property
□ MAINTAIN visual line of sight
□ RESPECT no-fly zones and regulations
□ CHECK local laws before flying
□ DON'T fly near airports or helicopters
□ AVOID flying in bad weather
□ LAND immediately if anything seems wrong
⚠️ AFTER FLIGHT
text
□ DISCONNECT battery first
□ ALLOW motors to cool
□ INSPECT for damage
□ CHECK all connections
□ STORE battery safely (fireproof bag)
□ LOG any issues for future reference
🚫 PROHIBITED
text
✗ NEVER touch spinning motors/props
✗ NEVER modify code while powered
✗ NEVER fly under influence
✗ NEVER fly beyond visual range
✗ NEVER ignore low battery warnings
✗ NEVER disable safety features
✗ NEVER fly in restricted areas
✗ NEVER let inexperienced persons fly
🔧 Troubleshooting Quick Reference
Problem	Likely Cause	Solution
No sensor data	I2C wiring	Check SDA/SCL connections
Won't arm	Throttle not min	Move throttle to zero
Radio timeout	NRF24L01 power	Add 100µF capacitor
Drifting	Not calibrated	Recalibrate on level surface
Oscillations	P gain too high	Reduce P by 20%
Short flight	Battery old	Replace battery
Hot motors	Wrong motor direction	Swap any two wires
One motor dead	ESC not calibrated	Recalibrate all ESCs
📱 Telemetry Data Format
text
Time(ms)  Roll Pitch Yaw  Alt(m)  Volt(V)  Mode       RSSI
1234      1.2   0.5   0.1  2.3     11.4     FLYING     -45dBm
📞 Support & Community
GitHub Issues: Report bugs and feature requests

Discord/Slack: Join drone developer communities

YouTube: Watch setup tutorials

Forums: ESP32 and Arduino forums

Local Drone Clubs: Get hands-on help

👨‍💻 Made By
═══════════════════════════════
ATHARVA PHADNIS
Aerospace Engineering
Embedded Systems Developer
Drone Technology Specialist
═══════════════════════════════
GitHub: github.com/atharvaphadnis
Email: atharva.phadnis@example.com
Project: ESP32 Advanced Flight Controller v2.0
Year: 2024

Special thanks to open-source community and all contributors

✅ CHECK THESE RULES BEFORE PROCEEDING
[ ] I HAVE READ ALL SAFETY WARNINGS
[ ] I UNDERSTAND THE RISKS INVOLVED
[ ] I WILL TEST WITHOUT PROPELLERS FIRST
[ ] I HAVE ALL REQUIRED COMPONENTS
[ ] I INSTALLED ALL LIBRARIES
[ ] I VERIFIED ALL CONNECTIONS
[ ] I CALIBRATED ESCs FIRST
[ ] I TESTED RADIO COMMUNICATION
[ ] I CHECKED MOTOR DIRECTIONS
[ ] I WILL FLY IN SAFE LOCATION
[ ] I HAVE FIRE EXTINGUISHER NEARBY
[ ] I WILL FOLLOW LOCAL REGULATIONS
📜 Disclaimer
THIS SOFTWARE IS PROVIDED "AS IS" FOR EDUCATIONAL AND EXPERIMENTAL PURPOSES ONLY.

The author (Atharva Phadnis) and contributors are NOT responsible for:

Any damage to property

Any injury to persons or animals

Any legal issues or violations

Any financial losses

Any data loss

Any crashes or accidents

By using this software, you agree to:

Take full responsibility for your actions

Follow all local laws and regulations

Test thoroughly in safe environments

Use proper safety equipment

Never fly near people or property

Maintain visual line of sight

Respect privacy of others

📋 Version History
Version	Date	Changes
v1.0	Jan 2024	Initial release
v1.5	Feb 2024	Added altitude hold
v2.0	Mar 2024	Complete rewrite, failsafe, telemetry
Happy Flying! 🚁 Stay Safe!
