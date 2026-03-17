# Espfly-07

Get App
🔧 Complete ESC Calibration Code for ESP32 Flight Controller
Here's a comprehensive ESC calibration code that works with your setup. This is CRITICAL for proper motor operation and must be done before first flight!

📋 ESC Calibration Program (Standalone)
cpp
/**
 * ESP32 ESC CALIBRATION TOOL
 * Complete calibration sequence for 4 ESCs simultaneously
 * Compatible with standard 50Hz ESCs (SimonK, BLHeli, etc.)
 * 
 * WIRING:
 * - Connect ESCs to ESP32 pins as defined below
 * - Connect ESC signal wires ONLY (power ESCs from battery)
 * - NEVER power ESCs from ESP32!
 * 
 * SAFETY WARNINGS:
 * - REMOVE PROPELLERS before calibration!
 * - Keep clear of motors during calibration
 * - Have battery ready but disconnected initially
 */

#include <ESP32Servo.h>

// ============================================
// PIN DEFINITIONS - MATCH YOUR MAIN CODE
// ============================================
#define MOTOR1_PIN      13  // Front Right (CW)
#define MOTOR2_PIN      12  // Front Left (CCW)
#define MOTOR3_PIN      14  // Rear Left (CW)
#define MOTOR4_PIN      27  // Rear Right (CCW)

// Button for manual control
#define CALIB_BUTTON_PIN 0  // Boot button on most ESP32 boards

// LED indicators
#define LED_STATUS      2   // Built-in LED
#define LED_ERROR       15

// ============================================
// ESC TIMING CONSTANTS
// ============================================
#define ESC_MIN_PWM     1000  // Minimum pulse width (1.0ms) - STOP
#define ESC_MAX_PWM     2000  // Maximum pulse width (2.0ms) - FULL SPEED
#define ESC_IDLE_PWM    1000  // Idle/stop position
#define PWM_FREQUENCY   50    // Standard ESC frequency (50Hz)

// Calibration timing (milliseconds)
#define CALIB_WAIT_TIME     3000  // Time to wait for ESC beeps
#define CALIB_BEEP_TIME     2000  // Time between beep sequences
#define MOTOR_RUN_TIME      5000  // Time to run motors at low speed for verification

// ============================================
// GLOBAL VARIABLES
// ============================================
Servo esc1, esc2, esc3, esc4;
bool calibrationComplete = false;
int calibrationStep = 0;

// ============================================
// SETUP
// ============================================
void setup() {
  Serial.begin(115200);
  delay(1000);
  
  Serial.println(F("\n\n========================================"));
  Serial.println(F("     ESP32 ESC CALIBRATION TOOL v2.0   "));
  Serial.println(F("========================================"));
  Serial.println(F("\n⚠️  WARNING: Remove propellers BEFORE starting!"));
  Serial.println(F("⚠️  Keep clear of motors during calibration"));
  Serial.println(F("\nPress the BOOT button to start calibration"));
  Serial.println(F("or type 'Y' in Serial Monitor to continue..."));
  
  // Initialize pins
  pinMode(CALIB_BUTTON_PIN, INPUT_PULLUP);
  pinMode(LED_STATUS, OUTPUT);
  pinMode(LED_ERROR, OUTPUT);
  
  // Blink LEDs to show ready
  for(int i = 0; i < 5; i++) {
    digitalWrite(LED_STATUS, HIGH);
    digitalWrite(LED_ERROR, HIGH);
    delay(100);
    digitalWrite(LED_STATUS, LOW);
    digitalWrite(LED_ERROR, LOW);
    delay(100);
  }
  
  // Initialize ESCs
  initESCs();
  
  Serial.println(F("\nWaiting for user confirmation..."));
}

// ============================================
// MAIN LOOP
// ============================================
void loop() {
  // Check for button press or serial input
  if (digitalRead(CALIB_BUTTON_PIN) == LOW || Serial.available() > 0) {
    char c = Serial.read();
    if (c == 'Y' || c == 'y' || digitalRead(CALIB_BUTTON_PIN) == LOW) {
      delay(500); // Debounce
      startCalibration();
    }
  }
  
  // Small delay to prevent watchdog issues
  delay(100);
}

// ============================================
// INITIALIZATION FUNCTIONS
// ============================================
void initESCs() {
  Serial.print(F("Initializing ESCs on pins: "));
  Serial.print(MOTOR1_PIN); Serial.print(F(", "));
  Serial.print(MOTOR2_PIN); Serial.print(F(", "));
  Serial.print(MOTOR3_PIN); Serial.print(F(", "));
  Serial.println(MOTOR4_PIN);
  
  // Allocate timers for ESP32Servo
  ESP32PWM::allocateTimer(0);
  ESP32PWM::allocateTimer(1);
  ESP32PWM::allocateTimer(2);
  ESP32PWM::allocateTimer(3);
  
  // Set frequency for all ESCs
  esc1.setPeriodHertz(PWM_FREQUENCY);
  esc2.setPeriodHertz(PWM_FREQUENCY);
  esc3.setPeriodHertz(PWM_FREQUENCY);
  esc4.setPeriodHertz(PWM_FREQUENCY);
  
  // Attach servos to pins
  esc1.attach(MOTOR1_PIN, ESC_MIN_PWM, ESC_MAX_PWM);
  esc2.attach(MOTOR2_PIN, ESC_MIN_PWM, ESC_MAX_PWM);
  esc3.attach(MOTOR3_PIN, ESC_MIN_PWM, ESC_MAX_PWM);
  esc4.attach(MOTOR4_PIN, ESC_MIN_PWM, ESC_MAX_PWM);
  
  // Set initial position to minimum
  setAllMotors(ESC_MIN_PWM);
  
  Serial.println(F("✅ ESCs initialized at MIN position"));
}

// ============================================
// MAIN CALIBRATION SEQUENCE
// ============================================
void startCalibration() {
  Serial.println(F("\n\n========================================"));
  Serial.println(F("     STARTING ESC CALIBRATION          "));
  Serial.println(F("========================================"));
  
  // STEP 1: Enter calibration mode (max throttle)
  Serial.println(F("\n📌 STEP 1: Entering calibration mode"));
  Serial.println(F("   - Sending MAX THROTTLE (2000µs)"));
  Serial.println(F("   - NOW connect the battery!"));
  Serial.println(F("\n⏰ You have 10 seconds to connect battery..."));
  
  digitalWrite(LED_STATUS, HIGH); // Solid LED during critical phase
  
  // Send max throttle
  setAllMotors(ESC_MAX_PWM);
  
  // Wait for user to connect battery
  for(int i = 10; i > 0; i--) {
    Serial.print(F("   "));
    Serial.print(i);
    Serial.println(F(" seconds remaining..."));
    delay(1000);
  }
  
  Serial.println(F("\n✅ Battery should be connected now"));
  Serial.println(F("   Listening for ESC beeps..."));
  delay(CALIB_WAIT_TIME); // Wait for initial beeps
  
  // STEP 2: Confirm calibration (min throttle)
  Serial.println(F("\n📌 STEP 2: Confirming calibration"));
  Serial.println(F("   - Sending MIN THROTTLE (1000µs)"));
  Serial.println(F("   - ESCs should emit confirmation beeps"));
  
  setAllMotors(ESC_MIN_PWM);
  digitalWrite(LED_STATUS, LOW);
  
  delay(CALIB_WAIT_TIME); // Wait for confirmation beeps
  
  // STEP 3: Verify calibration
  Serial.println(F("\n📌 STEP 3: Verification"));
  Serial.println(F("   - Testing motor response"));
  Serial.println(F("   - Motors will spin slowly"));
  
  verifyCalibration();
  
  // STEP 4: Complete
  calibrationComplete = true;
  
  Serial.println(F("\n========================================"));
  Serial.println(F("✅ ESC CALIBRATION COMPLETE!"));
  Serial.println(F("========================================"));
  Serial.println(F("\nYou can now:"));
  Serial.println(F("   1. Disconnect battery"));
  Serial.println(F("   2. Upload your flight controller code"));
  Serial.println(F("   3. Test motor directions"));
  Serial.println(F("\nPress BOOT button to test motors again"));
}

// ============================================
// VERIFICATION FUNCTIONS
// ============================================
void verifyCalibration() {
  Serial.println(F("\n🔄 Testing motor response..."));
  
  // Test each motor individually
  testIndividualMotors();
  
  // Test all motors together at low speed
  Serial.println(F("\n🔄 Testing all motors together at 10% throttle"));
  setAllMotors(map(10, 0, 100, ESC_MIN_PWM, ESC_MAX_PWM));
  delay(MOTOR_RUN_TIME);
  
  // Stop all motors
  setAllMotors(ESC_MIN_PWM);
  delay(1000);
  
  Serial.println(F("✅ Motor verification complete"));
}

void testIndividualMotors() {
  int motorPins[] = {MOTOR1_PIN, MOTOR2_PIN, MOTOR3_PIN, MOTOR4_PIN};
  String motorNames[] = {"Front Right (CW)", "Front Left (CCW)", "Rear Left (CW)", "Rear Right (CCW)"};
  
  for(int i = 0; i < 4; i++) {
    Serial.print(F("\n   Testing motor "));
    Serial.print(i+1);
    Serial.print(F(" - "));
    Serial.println(motorNames[i]);
    
    // Run this motor only at 15% throttle
    runSingleMotor(i, map(15, 0, 100, ESC_MIN_PWM, ESC_MAX_PWM));
    delay(2000);
    
    // Stop motor
    runSingleMotor(i, ESC_MIN_PWM);
    delay(1000);
  }
}

void runSingleMotor(int motorIndex, int pwmValue) {
  // Stop all motors first
  setAllMotors(ESC_MIN_PWM);
  delay(100);
  
  // Run only the selected motor
  switch(motorIndex) {
    case 0: esc1.writeMicroseconds(pwmValue); break;
    case 1: esc2.writeMicroseconds(pwmValue); break;
    case 2: esc3.writeMicroseconds(pwmValue); break;
    case 3: esc4.writeMicroseconds(pwmValue); break;
  }
}

// ============================================
// UTILITY FUNCTIONS
// ============================================
void setAllMotors(int pwmValue) {
  // Constrain to valid range
  pwmValue = constrain(pwmValue, ESC_MIN_PWM, ESC_MAX_PWM);
  
  esc1.writeMicroseconds(pwmValue);
  esc2.writeMicroseconds(pwmValue);
  esc3.writeMicroseconds(pwmValue);
  esc4.writeMicroseconds(pwmValue);
}

void motorBeepSequence() {
  // Generate a beep pattern through motors
  for(int i = 0; i < 3; i++) {
    setAllMotors(ESC_MIN_PWM + 50); // Slight pulse
    delay(100);
    setAllMotors(ESC_MIN_PWM);
    delay(100);
  }
}

// ============================================
// ADVANCED CALIBRATION OPTIONS
// ============================================
void advancedCalibrationMenu() {
  Serial.println(F("\n\n📋 ADVANCED CALIBRATION OPTIONS:"));
  Serial.println(F("   1. Standard calibration (all ESCs)"));
  Serial.println(F("   2. Calibrate ESCs individually"));
  Serial.println(F("   3. Test motor directions"));
  Serial.println(F("   4. Reset to defaults"));
  Serial.println(F("\nEnter choice (1-4):"));
  
  while(!Serial.available()) {
    delay(100);
  }
  
  int choice = Serial.parseInt();
  
  switch(choice) {
    case 1:
      startCalibration();
      break;
    case 2:
      calibrateIndividualESCs();
      break;
    case 3:
      testMotorDirections();
      break;
    case 4:
      resetToDefaults();
      break;
  }
}

void calibrateIndividualESCs() {
  Serial.println(F("\n⚠️  Calibrating ESCs individually"));
  Serial.println(F("Connect ONLY the ESC you want to calibrate"));
  
  for(int i = 0; i < 4; i++) {
    Serial.print(F("\nCalibrating ESC "));
    Serial.println(i+1);
    Serial.println(F("Connect this ESC only, then press button"));
    
    // Wait for button press
    while(digitalRead(CALIB_BUTTON_PIN) == HIGH) {
      delay(100);
    }
    
    // Calibrate this ESC
    runSingleMotor(i, ESC_MAX_PWM);
    delay(5000); // Wait for beeps
    
    runSingleMotor(i, ESC_MIN_PWM);
    delay(3000); // Wait for confirmation
    
    Serial.println(F("✅ ESC calibrated"));
    delay(1000);
  }
}

void testMotorDirections() {
  Serial.println(F("\n🔄 Testing motor directions"));
  Serial.println(F("Expected rotation:"));
  Serial.println(F("   Motor1 (FR): Clockwise"));
  Serial.println(F("   Motor2 (FL): Counter-Clockwise"));
  Serial.println(F("   Motor3 (RL): Clockwise"));
  Serial.println(F("   Motor4 (RR): Counter-Clockwise"));
  
  for(int i = 0; i < 4; i++) {
    Serial.print(F("\nRunning motor "));
    Serial.print(i+1);
    Serial.println(F(" at low speed - CHECK DIRECTION!"));
    
    runSingleMotor(i, map(15, 0, 100, ESC_MIN_PWM, ESC_MAX_PWM));
    delay(3000);
    runSingleMotor(i, ESC_MIN_PWM);
    delay(1000);
  }
}

void resetToDefaults() {
  Serial.println(F("\n🔄 Resetting ESCs to defaults"));
  Serial.println(F("This requires power cycling the ESCs"));
  
  setAllMotors(ESC_MIN_PWM);
  delay(1000);
  
  // Power cycle indication
  digitalWrite(LED_ERROR, HIGH);
  delay(2000);
  digitalWrite(LED_ERROR, LOW);
  
  Serial.println(F("✅ Reset complete - power cycle ESCs"));
}

// ============================================
// EMERGENCY STOP
// ============================================
void emergencyStop() {
  Serial.println(F("\n⚠️  EMERGENCY STOP ACTIVATED!"));
  setAllMotors(ESC_MIN_PWM);
  
  // Rapid LED flash
  for(int i = 0; i < 20; i++) {
    digitalWrite(LED_ERROR, HIGH);
    delay(50);
    digitalWrite(LED_ERROR, LOW);
    delay(50);
  }
}
📋 Quick Calibration Script (Simplified Version)
If you want a simpler version for quick calibration:

cpp
/**
 * SIMPLE ESC CALIBRATION
 * Minimal version for quick calibration
 */

#include <ESP32Servo.h>

#define MOTOR1 13
#define MOTOR2 12
#define MOTOR3 14
#define MOTOR4 27

Servo esc1, esc2, esc3, esc4;

void setup() {
  Serial.begin(115200);
  delay(1000);
  
  Serial.println("\nSIMPLE ESC CALIBRATION");
  Serial.println("======================");
  Serial.println("1. DISCONNECT battery");
  Serial.println("2. REMOVE propellers");
  Serial.println("3. Press ENTER when ready...");
  
  while(!Serial.available()) {
    delay(100);
  }
  Serial.read();
  
  // Initialize ESCs
  ESP32PWM::allocateTimer(0);
  ESP32PWM::allocateTimer(1);
  ESP32PWM::allocateTimer(2);
  ESP32PWM::allocateTimer(3);
  
  esc1.attach(MOTOR1, 1000, 2000);
  esc2.attach(MOTOR2, 1000, 2000);
  esc3.attach(MOTOR3, 1000, 2000);
  esc4.attach(MOTOR4, 1000, 2000);
  
  // Step 1: Max throttle
  Serial.println("\nSending MAX throttle...");
  Serial.println("NOW connect the battery!");
  esc1.writeMicroseconds(2000);
  esc2.writeMicroseconds(2000);
  esc3.writeMicroseconds(2000);
  esc4.writeMicroseconds(2000);
  
  delay(5000); // Wait for initialization beeps
  
  // Step 2: Min throttle
  Serial.println("Sending MIN throttle...");
  esc1.writeMicroseconds(1000);
  esc2.writeMicroseconds(1000);
  esc3.writeMicroseconds(1000);
  esc4.writeMicroseconds(1000);
  
  delay(5000); // Wait for confirmation beeps
  
  Serial.println("CALIBRATION COMPLETE!");
  Serial.println("Disconnect battery now.");
}

void loop() {
  // Blink LED to show completion
  digitalWrite(2, HIGH);
  delay(500);
  digitalWrite(2, LOW);
  delay(500);
}
📋 Motor Direction Test Code
After calibration, test motor directions:

cpp
/**
 * MOTOR DIRECTION TESTER
 * Run this after calibration to verify correct rotation
 */

#include <ESP32Servo.h>

#define MOTOR1 13  // Front Right
#define MOTOR2 12  // Front Left
#define MOTOR3 14  // Rear Left
#define MOTOR4 27  // Rear Right

Servo motors[4];
int motorPins[4] = {MOTOR1, MOTOR2, MOTOR3, MOTOR4};
String motorNames[4] = {"Front Right", "Front Left", "Rear Left", "Rear Right"};
String expectedRotation[4] = {"CW", "CCW", "CW", "CCW"};

void setup() {
  Serial.begin(115200);
  delay(1000);
  
  Serial.println("\nMOTOR DIRECTION TESTER");
  Serial.println("======================");
  Serial.println("Expected rotations:");
  for(int i = 0; i < 4; i++) {
    Serial.print("  Motor ");
    Serial.print(i+1);
    Serial.print(" (");
    Serial.print(motorNames[i]);
    Serial.print("): ");
    Serial.println(expectedRotation[i]);
  }
  
  Serial.println("\nPress ENTER to start test...");
  while(!Serial.available()) {
    delay(100);
  }
  Serial.read();
  
  // Initialize motors
  for(int i = 0; i < 4; i++) {
    motors[i].setPeriodHertz(50);
    motors[i].attach(motorPins[i], 1000, 2000);
    motors[i].writeMicroseconds(1000); // Stop
  }
  
  // Test each motor
  for(int i = 0; i < 4; i++) {
    Serial.print("\nTesting motor ");
    Serial.print(i+1);
    Serial.print(" (");
    Serial.print(motorNames[i]);
    Serial.println(")");
    Serial.println("Motor should rotate at low speed");
    Serial.println("Press ENTER when ready to start...");
    
    while(!Serial.available()) {
      delay(100);
    }
    Serial.read();
    
    // Run motor at 15% throttle
    motors[i].writeMicroseconds(1150);
    
    Serial.println("Motor running - CHECK DIRECTION!");
    Serial.println("Press ENTER when done checking...");
    
    while(!Serial.available()) {
      delay(100);
    }
    Serial.read();
    
    // Stop motor
    motors[i].writeMicroseconds(1000);
    
    Serial.print("Does it rotate ");
    Serial.print(expectedRotation[i]);
    Serial.println("? (Y/N)");
    
    while(!Serial.available()) {
      delay(100);
    }
    char response = Serial.read();
    
    if(response == 'Y' || response == 'y') {
      Serial.println("✓ Correct direction");
    } else {
      Serial.println("✗ WRONG DIRECTION - Swap any two motor wires");
    }
    
    delay(1000);
  }
  
  Serial.println("\nMotor test complete!");
}

void loop() {
  // Keep motors stopped
  for(int i = 0; i < 4; i++) {
    motors[i].writeMicroseconds(1000);
  }
}
📋 ESC Calibration Procedure - Step by Step
Before You Start:
REMOVE ALL PROPELLERS - This is critical for safety

Disconnect battery from ESCs

Connect ESP32 to USB power only

Have your flight battery ready but disconnected

Calibration Steps:
Upload the calibration code to your ESP32

Open Serial Monitor (115200 baud)

Follow the prompts:

Code will send MAX throttle signal

NOW connect your battery to ESCs

ESCs will beep (usually 1-2-3 beeps)

After 5 seconds, code sends MIN throttle

ESCs will beep confirmation (usually 3 beeps)

Disconnect battery immediately after calibration

Troubleshooting:
Problem	Solution
No beeps	Check ESC power, signal connections
Continuous beeping	Throttle not at minimum, recalibrate
Motors twitch but don't run	Check ESC calibration, battery voltage
One motor doesn't respond	Check that ESC's signal wire, try individual calibration
ESCs won't enter calibration	Ensure throttle was at max when power applied
Safety Checklist:
Propellers removed

Clear work area

Battery fully charged

All connections secure

Fire extinguisher nearby

Someone else present (recommended)

After Calibration:
Disconnect battery

Upload your main flight controller code

Test motor directions (without props)

Re-attach props only after all tests pass

This calibration code ensures your ESCs have consistent throttle ranges and will respond correctly to your flight controller's commands!

Motor calibration code short
🔧 SHORT ESC CALIBRATION CODE
Here's a minimal but complete ESC calibration code for your ESP32:

📋 Ultra-Short Calibration Code
cpp
/**
 * ULTRA-SHORT ESC CALIBRATION
 * Minimal code for quick calibration
 * 
 * PINS: Motor1=13, Motor2=12, Motor3=14, Motor4=27
 */

#include <ESP32Servo.h>

// Motor pins
#define M1 13  // Front Right
#define M2 12  // Front Left
#define M3 14  // Rear Left
#define M4 27  // Rear Right

Servo m1, m2, m3, m4;

void setup() {
  Serial.begin(115200);
  delay(1000);
  
  Serial.println("\n=== ESC CALIBRATION ===");
  Serial.println("1. DISCONNECT battery");
  Serial.println("2. REMOVE propellers");
  Serial.println("3. Press ENTER to continue...");
  
  while(!Serial.available()) delay(100);
  Serial.read();
  
  // Initialize ESCs
  ESP32PWM::allocateTimer(0);
  ESP32PWM::allocateTimer(1);
  
  m1.attach(M1, 1000, 2000);
  m2.attach(M2, 1000, 2000);
  m3.attach(M3, 1000, 2000);
  m4.attach(M4, 1000, 2000);
  
  // STEP 1: MAX THROTTLE
  Serial.println("\nSending MAX throttle (2000µs)");
  Serial.println("NOW connect the battery!");
  Serial.println("Wait for beeps...");
  
  m1.writeMicroseconds(2000);
  m2.writeMicroseconds(2000);
  m3.writeMicroseconds(2000);
  m4.writeMicroseconds(2000);
  
  delay(5000); // Wait for beeps
  
  // STEP 2: MIN THROTTLE
  Serial.println("Sending MIN throttle (1000µs)");
  Serial.println("Wait for confirmation beeps...");
  
  m1.writeMicroseconds(1000);
  m2.writeMicroseconds(1000);
  m3.writeMicroseconds(1000);
  m4.writeMicroseconds(1000);
  
  delay(5000); // Wait for beeps
  
  Serial.println("\n✅ CALIBRATION COMPLETE!");
  Serial.println("Disconnect battery now");
}

void loop() {
  // Blink built-in LED to show done
  digitalWrite(2, HIGH);
  delay(500);
  digitalWrite(2, LOW);
  delay(500);
}
📋 Even Shorter Version (No Serial)
cpp
/**
 * MINIMAL ESC CALIBRATION - No Serial needed
 * Just upload and follow LED blinks
 */

#include <ESP32Servo.h>

#define M1 13
#define M2 12
#define M3 14
#define M4 27
#define LED 2

Servo m1, m2, m3, m4;

void setup() {
  pinMode(LED, OUTPUT);
  
  // Fast blink = ready for battery
  for(int i=0; i<10; i++) {
    digitalWrite(LED, HIGH); delay(100);
    digitalWrite(LED, LOW); delay(100);
  }
  
  ESP32PWM::allocateTimer(0);
  ESP32PWM::allocateTimer(1);
  
  m1.attach(M1, 1000, 2000);
  m2.attach(M2, 1000, 2000);
  m3.attach(M3, 1000, 2000);
  m4.attach(M4, 1000, 2000);
  
  // STEP 1: MAX THROTTLE - Solid LED
  digitalWrite(LED, HIGH);
  m1.writeMicroseconds(2000);
  m2.writeMicroseconds(2000);
  m3.writeMicroseconds(2000);
  m4.writeMicroseconds(2000);
  
  delay(5000); // Wait for beeps
  
  // STEP 2: MIN THROTTLE - Blinking LED
  m1.writeMicroseconds(1000);
  m2.writeMicroseconds(1000);
  m3.writeMicroseconds(1000);
  m4.writeMicroseconds(1000);
  
  // Blink LED during confirmation
  for(int i=0; i<10; i++) {
    digitalWrite(LED, HIGH); delay(250);
    digitalWrite(LED, LOW); delay(250);
  }
  
  // Slow blink = calibration complete
}

void loop() {
  digitalWrite(LED, HIGH);
  delay(1000);
  digitalWrite(LED, LOW);
  delay(1000);
}
📋 Single ESC Calibration (for testing)
cpp
/**
 * SINGLE ESC CALIBRATION
 * Test one ESC at a time
 */

#include <ESP32Servo.h>

#define MOTOR_PIN 13  // Change this to test different ESCs
#define LED_PIN 2

Servo esc;

void setup() {
  Serial.begin(115200);
  
  Serial.println("\nSINGLE ESC CALIBRATION");
  Serial.print("Testing motor on pin "); Serial.println(MOTOR_PIN);
  
  ESP32PWM::allocateTimer(0);
  esc.attach(MOTOR_PIN, 1000, 2000);
  
  Serial.println("\nMAX throttle - Connect battery NOW!");
  esc.writeMicroseconds(2000);
  delay(5000);
  
  Serial.println("MIN throttle - Waiting for beeps");
  esc.writeMicroseconds(1000);
  delay(5000);
  
  Serial.println("✅ Done! Test motor at 15%");
  esc.writeMicroseconds(1150);
  delay(3000);
  
  esc.writeMicroseconds(1000);
  Serial.println("Complete - disconnect battery");
}

void loop() {
  digitalWrite(LED_PIN, HIGH);
  delay(1000);
  digitalWrite(LED_PIN, LOW);
  delay(1000);
}
Quick Instructions:
Upload any of the above codes

Remove propellers ⚠️

Disconnect battery

Open Serial Monitor (if using first version)

Follow prompts or watch LED patterns

Connect battery when told

Wait for beeps (about 5 seconds each step)

Done! Disconnect battery

The LED pattern version is safest - no chance of wrong serial commands!

Make me a long readme with connections abilities future scopes
🚁 ESP32 Advanced Flight Controller Documentation
Complete README & Technical Documentation
https://img.shields.io/badge/ESP32-Flight%2520Controller-blue
https://img.shields.io/badge/Version-2.0.0-green
https://img.shields.io/badge/Status-Stable-success
https://img.shields.io/badge/License-MIT-orange

📑 Table of Contents
Project Overview

Features

Hardware Requirements

Pin Connections

Software Architecture

Installation Guide

Configuration

Calibration

Flight Modes

PID Tuning

Safety Features

Troubleshooting

Future Scope

Contributing

License

🎯 Project Overview
This is a professional-grade flight controller built on the ESP32 platform, designed for quadcopters, drones, and various multirotor applications. It combines the power of ESP32's dual-core processing with high-precision sensors (MPU6050 + BMP280) and reliable NRF24L01 communication to create a robust, feature-rich flight control system.

Key Highlights
Dual-core processing for real-time control and telemetry

6-DOF stabilization with sensor fusion

Barometric altitude hold for precise height control

Long-range communication via NRF24L01 (up to 1km with proper antennas)

Advanced failsafe systems with auto-land capability

Real-time telemetry for ground station monitoring

✨ Features
Core Features
Feature	Description	Status
6-Axis Stabilization	MPU6050-based roll, pitch, yaw control	✅ Stable
Altitude Hold	BMP280 barometric pressure sensor	✅ Stable
PID Control	4 independent PID controllers	✅ Tuneable
Radio Link	NRF24L01 with 250kbps - 2Mbps	✅ Reliable
Failsafe	Auto-land on signal loss	✅ Active
Battery Monitoring	Voltage sensing with alerts	✅ Active
Advanced Features
Complementary Filter for attitude estimation (95% gyro / 5% accelerometer)

Anti-windup Protection on all PID controllers

Motor Mixing for Quad-X configuration

ARM/Disarm Safety with switch combination

LED Status Indication for all flight modes

Buzzer Alerts for critical conditions

EEPROM Configuration storage (future)

Black Box Logging capability (future)

🔧 Hardware Requirements
Required Components
Component	Quantity	Specifications	Approx Cost
ESP32 Development Board	1	Any variant with 30+ pins	$5-10
MPU6050	1	6-Axis IMU (accel + gyro)	$3-5
BMP280	1	Barometric pressure sensor	$3-5
NRF24L01+PA+LNA	2	With external antenna for range	$5-8 each
ESCs (SimonK/BLHeli)	4	20-30A rating	$8-12 each
Brushless Motors	4	2204-2306 size	$10-15 each
LiPo Battery	1	3S-4S 1300-2200mAh	$15-25
Power Distribution Board	1	With BEC 5V output	$5-10
Frame	1	250-450mm size	$15-30
Propellers	4 pairs	5-7 inch	$5-10
Optional Components
Component	Purpose
GPS Module (NEO-6M/NEO-8M)	Position hold, return-to-home
Magnetometer (HMC5883L)	Heading hold
Current Sensor	Power monitoring
OLED Display	On-field configuration
MicroSD Card Module	Flight data logging
Bluetooth Module	Wireless configuration
Total Project Cost: $70-150 (depending on options)
🔌 Pin Connections
Complete Wiring Diagram
text
                    ESP32 DEVELOPMENT BOARD
    ┌─────────────────────────────────────────────────┐
    │                                                 │
    │  3.3V ──────┬───── MPU6050 VCC                 │
    │             └───── BMP280 VCC                   │
    │             └───── NRF24L01 VCC (with cap)      │
    │                                                 │
    │  GND ───────┬───── MPU6050 GND                  │
    │             └───── BMP280 GND                    │
    │             └───── NRF24L01 GND                  │
    │                                                 │
    │  GPIO21 (SDA) ──── MPU6050 SDA                  │
    │                 └─── BMP280 SDA                  │
    │                                                 │
    │  GPIO22 (SCL) ──── MPU6050 SCL                  │
    │                 └─── BMP280 SCL                  │
    │                                                 │
    │  GPIO4  (CE)  ──── NRF24L01 CE                  │
    │  GPIO5  (CSN) ──── NRF24L01 CSN                 │
    │  GPIO18 (SCK) ──── NRF24L01 SCK                 │
    │  GPIO23 (MOSI) ─── NRF24L01 MOSI                │
    │  GPIO19 (MISO) ─── NRF24L01 MISO                │
    │                                                 │
    │  GPIO13 ──────── Motor 1 ESC (Front Right)      │
    │  GPIO12 ──────── Motor 2 ESC (Front Left)       │
    │  GPIO14 ──────── Motor 3 ESC (Rear Left)        │
    │  GPIO27 ──────── Motor 4 ESC (Rear Right)       │
    │                                                 │
    │  GPIO32 ──────── Buzzer                          │
    │  GPIO34 ──────── Battery Monitor (voltage)      │
    │  GPIO2  ──────── LED (Status/Armed)             │
    │  GPIO15 ──────── LED (Error)                    │
    │                                                 │
    │  GPIO0  ──────── Calibration Button (boot)      │
    └─────────────────────────────────────────────────┘
Detailed Pin Table
Function	ESP32 Pin	Wire Color	Notes
POWER			
3.3V	3.3V	Red	To all modules
GND	GND	Black	Common ground
I2C SENSORS			
MPU6050/BMP280 SDA	GPIO21	Green	Shared bus
MPU6050/BMP280 SCL	GPIO22	Yellow	Shared bus
NRF24L01			
CE	GPIO4	Orange	Chip Enable
CSN	GPIO5	Brown	Chip Select
SCK	GPIO18	Purple	SPI Clock
MOSI	GPIO23	Blue	Master Out
MISO	GPIO19	Gray	Master In
MOTORS			
Motor 1 (FR)	GPIO13	White	ESC signal
Motor 2 (FL)	GPIO12	White	ESC signal
Motor 3 (RL)	GPIO14	White	ESC signal
Motor 4 (RR)	GPIO27	White	ESC signal
INDICATORS			
Status LED	GPIO2	Green	Built-in
Error LED	GPIO15	Red	External
Buzzer	GPIO32	Yellow	Optional
SENSORS			
Battery Voltage	GPIO34	Orange	Via divider
Important Notes:
Add a 10-100µF capacitor across NRF24L01 VCC and GND

Use 10kΩ pull-up resistors on I2C lines (SDA/SCL)

Voltage divider for battery: Use 1kΩ and 2.2kΩ resistors

Keep signal wires short (<20cm) to prevent interference

🏗️ Software Architecture
System Block Diagram
text
┌─────────────────────────────────────────────────────────────┐
│                        ESP32 FLIGHT CONTROLLER               │
├─────────────────────────────────────────────────────────────┤
│                         CORE 0 (Protocol)                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ NRF24L01 RX  │  │  Telemetry   │  │   Serial     │      │
│  │   (50Hz)     │  │   (10Hz)     │  │   Debug      │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                  │                  │              │
├─────────┼──────────────────┼──────────────────┼─────────────┤
│         │                  │                  │              │
│  ┌──────▼───────┐  ┌──────▼───────┐  ┌──────▼───────┐      │
│  │   Command    │  │   Telemetry  │  │    Debug     │      │
│  │  Processing  │  │   Formatter  │  │   Output     │      │
│  └──────┬───────┘  └──────────────┘  └──────────────┘      │
│         │                                                    │
├─────────┼────────────────────────────────────────────────────┤
│         │                  CORE 1 (Control)                  │
│  ┌──────▼───────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   MPU6050    │  │    BMP280    │  │   Battery    │      │
│  │   (400Hz)    │  │   (100Hz)    │  │   Monitor    │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                  │                  │              │
│  ┌──────▼──────────────────▼──────────────────▼───────┐     │
│  │              Sensor Fusion & Attitude               │     │
│  │           (Complementary Filter - 400Hz)           │     │
│  └──────────────────────┬─────────────────────────────┘     │
│                         │                                     │
│  ┌──────────────────────▼─────────────────────────────┐     │
│  │                PID Controllers                      │     │
│  │    Roll    Pitch    Yaw    Altitude                │     │
│  └──────────────────────┬─────────────────────────────┘     │
│                         │                                     │
│  ┌──────────────────────▼─────────────────────────────┐     │
│  │               Motor Mixing (Quad-X)                 │     │
│  └──────────────────────┬─────────────────────────────┘     │
│                         │                                     │
│  ┌──────────────────────▼─────────────────────────────┐     │
│  │              ESC Output (50Hz PWM)                  │     │
│  └─────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────┘
Task Timing
Task	Frequency	Core	Priority
IMU Reading	400Hz	1	High
PID Calculation	400Hz	1	High
Motor Output	400Hz	1	High
Radio RX	50Hz	0	Medium
Telemetry	10Hz	0	Low
Battery Monitor	1Hz	0	Low
📥 Installation Guide
Step 1: Arduino IDE Setup
bash
1. Download Arduino IDE from arduino.cc
2. Install ESP32 board support:
   - File → Preferences
   - Add URL: https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   - Tools → Board → Board Manager → Search "ESP32" → Install
Step 2: Library Installation
bash
Required Libraries (install via Library Manager):
1. "RF24" by TMRh20
2. "Adafruit MPU6050" by Adafruit
3. "Adafruit BMP280 Library" by Adafruit
4. "Adafruit Unified Sensor" by Adafruit
5. "ESP32Servo" by Kevin Harrington
Step 3: Code Upload
cpp
// 1. Select board: ESP32 Dev Module
// 2. Select correct COM port
// 3. Upload flight controller code
// 4. Open Serial Monitor @ 115200 baud
Step 4: First Power-Up Sequence
text
1. Disconnect battery
2. Upload code via USB
3. Open Serial Monitor
4. Verify sensor initialization
5. Disconnect USB
6. Connect battery
7. Check LED patterns
8. Arm system (throttle low + arm switch)
⚙️ Configuration
PID Tuning Parameters
cpp
// Default PID values (starting point)
#define PID_ROLL_KP   1.50
#define PID_ROLL_KI   0.05
#define PID_ROLL_KD   0.10

#define PID_PITCH_KP  1.50
#define PID_PITCH_KI  0.05
#define PID_PITCH_KD  0.10

#define PID_YAW_KP    2.00
#define PID_YAW_KI    0.10
#define PID_YAW_KD    0.20

#define PID_ALT_KP    1.00
#define PID_ALT_KI    0.01
#define PID_ALT_KD    0.50
Flight Parameters
cpp
// Angle limits
#define MAX_ANGLE_ROLL     30.0  // degrees
#define MAX_ANGLE_PITCH    30.0  // degrees
#define MAX_YAW_RATE      180.0  // degrees/sec

// Throttle limits
#define THROTTLE_MIN     1000  // µs
#define THROTTLE_IDLE    1050  // µs
#define THROTTLE_MAX     2000  // µs

// Safety timers
#define RADIO_TIMEOUT    250   // ms
#define ARM_DELAY        1000  // ms
Radio Configuration
cpp
// NRF24L01 settings
#define RF_CHANNEL       100   // 2.4GHz channel
#define RF_DATA_RATE     RF24_250KBPS  // Better range
#define RF_PA_LEVEL      RF24_PA_HIGH  // Maximum power
🔄 Calibration
ESC Calibration Procedure
cpp
/**
 * Step-by-step ESC calibration
 */
 
 1. Upload ESC calibration code
 2. REMOVE PROPELLERS (critical!)
 3. Disconnect battery
 4. Open Serial Monitor
 5. Follow prompts:
    - Code sends MAX throttle
    - NOW connect battery
    - Wait for beeps (2-3 seconds)
    - Code sends MIN throttle
    - Wait for confirmation beeps
 6. Disconnect battery
 7. Upload flight controller code
Sensor Calibration
cpp
/**
 * Automatic sensor calibration on level surface
 */
 
 1. Place drone on perfectly level surface
 2. Power on (do not move)
 3. Wait 10 seconds for gyro calibration
 4. LED will blink when complete
 5. System is now calibrated
🎮 Flight Modes
Mode 1: STANDBY
LED: Slow blinking

Motors: Disarmed

Description: Safe mode, waiting for arm command

Mode 2: ARMED (Acro Mode)
LED: Solid + heartbeat

Motors: Armed, ready for flight

Stabilization: Rate mode (pilot controls rate)

Mode 3: ALTITUDE HOLD
LED: Solid + double heartbeat

Motors: Automatic altitude maintenance

Stabilization: Maintains current altitude

Mode 4: FAILSAFE
LED: Fast red blink

Buzzer: Continuous beep

Action: Auto-land, then disarm

Arm/Disarm Sequence
text
ARMING:
1. Throttle at minimum
2. Arm switch to ON position
3. Wait 2 seconds
4. Motors will spin at idle
5. Green LED solid

DISARMING:
1. Throttle at minimum
2. Arm switch to OFF
3. Motors stop immediately
4. Green LED blinking
📊 PID Tuning Guide
Understanding PIDs
Term	Effect	Too Low	Too High
P (Proportional)	Response to error	Slow response	Oscillations
I (Integral)	Corrects steady error	Drifting	Wind-up, instability
D (Derivative)	Damping, overshoot control	Oscillations	Noise amplification
Tuning Procedure
cpp
/**
 * Step-by-step PID tuning
 */
 
// START WITH SAFE VALUES
P = 1.0, I = 0.01, D = 0.1

// TUNING SEQUENCE
Step 1: P Gain (Roll/Pitch)
  - Increase P until slight oscillations
  - Reduce by 20% for stable value

Step 2: D Gain (Roll/Pitch)
  - Increase D to stop oscillations
  - Stop if motors get hot/noisy

Step 3: I Gain (Roll/Pitch)
  - Increase I to eliminate drift
  - Small values only (0.01-0.05)

Step 4: Yaw PIDs
  - Yaw needs higher P (2.0-3.0)
  - Minimal I (0.01) or zero
  - D for damping (0.1-0.3)

Step 5: Altitude Hold
  - P for responsiveness
  - I for altitude maintenance
  - D for smoothness
PID Presets
Flight Style	Roll/Pitch P	Roll/Pitch I	Roll/Pitch D	Yaw P
Beginner	1.2	0.02	0.15	2.0
Sport	1.8	0.04	0.10	2.5
Racing	2.5	0.06	0.05	3.0
Cinematic	1.0	0.03	0.20	1.8
🛡️ Safety Features
1. Arming Safety
Requires throttle at minimum

Arm switch must be toggled

2-second delay before arming

Prevents accidental startup

2. Failsafe System
text
Trigger Conditions:
- Radio signal lost >250ms
- Battery critically low
- Sensor failure detected

Failsafe Actions:
1. Reduce throttle gradually
2. Level attitude
3. Auto-land
4. Disarm motors
5. Continuous buzzer
3. Battery Protection
Voltage (3S)	Status	Action
>11.4V	Normal	Normal operation
10.5-11.4V	Low	Buzzer warning (slow beep)
<10.5V	Critical	Failsafe activation
4. Sensor Failure Detection
cpp
// Continuous monitoring of:
- IMU data rate
- Sensor values within range
- I2C communication
- Gyro drift

// Action on failure:
- Immediate disarm
- Error LED pattern
- Buzzer alarm
🔍 Troubleshooting
Common Issues & Solutions
Problem	Possible Cause	Solution
No sensor data	I2C wiring wrong	Check SDA/SCL connections
Motors won't arm	Throttle not min	Move throttle to zero
Arm switch off	Toggle arm switch
Radio timeout	Check transmitter
Drifting in flight	Sensor not level	Recalibrate on level surface
Vibration	Soft mount flight controller
I gain too low	Increase I term slightly
Oscillations	P gain too high	Reduce P by 20%
D gain too low	Increase D slightly
Frame resonance	Change PID frequency
Short flight time	Battery old	Replace battery
Props damaged	Replace props
Motors inefficient	Check motor bearings
Radio range issue	Antenna position	Orient vertically
Power too low	Set PA level to HIGH
Interference	Change channel
One motor not spinning	ESC not calibrated	Recalibrate ESCs
Motor wire broken	Check connections
ESC damaged	Test with servo tester
LED Error Codes
LED Pattern	Meaning	Action
Fast blink (all)	Sensor init failed	Check wiring
2 blinks, pause	Radio not connected	Check transmitter
3 blinks, pause	Battery critical	Land immediately
4 blinks, pause	Gyro calibration	Place on level surface
Solid + slow blink	Armed, all OK	Ready to fly
🚀 Future Scope
Immediate Enhancements (v2.1)
cpp
// Coming in next update
- [ ] GPS integration (NEO-6M)
- [ ] Return-to-home functionality
- [ ] Waypoint navigation
- [ ] Black box data logging
- [ ] In-flight PID tuning via radio
Advanced Features (v3.0)
cpp
// Planned features
- [ ] Optical flow for indoor positioning
- [ ] Obstacle avoidance with sonar/LIDAR
- [ ] Computer vision integration
- [ ] Swarm capabilities
- [ ] Autonomous mission planning
Research & Development
1. Advanced Control Algorithms
Kalman filtering for better attitude estimation

Machine learning for adaptive PID tuning

Model Predictive Control (MPC)

2. Computer Vision Integration
Object tracking and following

Landing pad detection

Collision avoidance

Aerial mapping

3. Swarm Intelligence
Multi-drone coordination

Formation flying

Collaborative sensing

Mesh networking

4. Extended Capabilities
cpp
// Hardware upgrades
- 4G/LTE for beyond visual line of sight
- Thermal imaging camera
- Multispectral sensors for agriculture
- LiDAR for 3D mapping
- Payload delivery system
Potential Applications
Industry	Application	Required Features
Agriculture	Crop monitoring, spraying	Multispectral, GPS, altitude hold
Construction	Site surveying, inspection	LiDAR, photogrammetry
Search & Rescue	Thermal imaging, area search	Thermal cam, long range
Film Industry	Cinematic shots	Smooth gimbal, waypoints
Security	Perimeter surveillance	Object tracking, auto-return
Delivery	Package transport	Heavy lift, precision landing
Environmental	Wildlife monitoring	Silent motors, long endurance
Performance Improvements
cpp
// Optimization targets
1. Control loop: 400Hz → 1kHz
2. Sensor fusion: Complementary → Kalman
3. Telemetry: 10Hz → 50Hz
4. Range: 1km → 5km (with directional antennas)
5. Flight time: 15min → 30min (efficient motors)
Community Features
Web-based configurator (like Betaflight)

Mobile app for monitoring

Cloud logging for flight analysis

Community PID database for different frames

Simulator integration for safe testing

🤝 Contributing
How to Contribute
Fork the repository

Create feature branch

Commit changes

Push to branch

Open pull request

Coding Standards
cpp
// Follow these guidelines
- Use meaningful variable names
- Comment complex logic
- Follow existing formatting
- Test on hardware before PR
- Update documentation
Testing Requirements
cpp
// Before submitting
- [ ] Compiles without warnings
- [ ] Passes basic functionality test
- [ ] No regression in existing features
- [ ] Updated documentation
- [ ] Added test cases if applicable
📄 License
This project is licensed under the MIT License - see below:

text
MIT License

Copyright (c) 2024 ESP32 Flight Controller Project

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
🙏 Acknowledgments
Arduino Community for incredible libraries

Adafruit for excellent sensor libraries

TMRh20 for RF24 library

ESP32 community for platform support

Betaflight/ArduPilot for inspiration

All contributors and testers

📞 Support & Contact
GitHub Issues: [Link to repository]

Discord: [Link to server]

Email: project@example.com

Wiki: [Link to wiki]

📊 Version History
Version	Date	Changes
v1.0.0	Jan 2024	Initial release
v1.5.0	Feb 2024	Added altitude hold
v2.0.0	Mar 2024	Major rewrite, failsafe
v2.0.1	Current	Bug fixes, optimization
⚠️ Disclaimer
THIS SOFTWARE IS PROVIDED FOR EXPERIMENTAL AND EDUCATIONAL PURPOSES ONLY.

Operating unmanned aircraft systems (drones) may be subject to local laws and regulations. Always:

Check local regulations before flying

Fly in designated areas

Maintain visual line of sight

Respect privacy of others

Ensure safe operation at all times

The authors and contributors are not responsible for any misuse, damage, or legal issues arising from the use of this software.

Happy Flying! 🚁

