# Smart Window Automation System

## Overview
This project introduces an innovative Smart Window Automation System that integrates an Arduino Uno with multiple environmental sensors to intelligently manage window operations and enhance indoor comfort and safety. Leveraging inputs from a rain sensor, smoke sensor, LDR (Light Dependent Resistor), and AC control switch, the system automatically opens or closes the window and deploys a mosquito net based on real-time conditions.

## Key Features
* **Smoke and Gas Alert:** Detects smoke or gas leakage and immediately opens the window to allow safe ventilation.
* **Rain Protection:** Detects rain and ensures the window is closed to prevent water entry.
* **AC Optimization:** Detects AC usage and ensures the window is closed to maintain energy efficiency.
* **Automated Mosquito Net:** Controlled based on light intensity, ensuring it remains deployed at night and retracted during the day.
* **Manual Override:** Allows users to bypass automation and control the system manually.

## Hardware Components
* Arduino Uno
* Rain Sensor
* Smoke Sensor (MQ-2)
* LDR (Light Dependent Resistor)
* Servo Motors (Two motors: one for the window, one for the mosquito net)
* Toggle Switches (for AC status and manual override)

## Circuit Logic & Pin Configuration
* **A0:** LDR (Light Dependent Resistor)
* **D2:** Rain Sensor
* **D3:** Smoke Sensor
* **D4:** AC Status Switch
* **D5:** Window Override Switch
* **D6:** Net Override Switch
* **D9:** Window Servo Motor
* **D10:** Mosquito Net Servo Motor

## Video Demonstration
[Watch the Smart Window System in action here!]( REPLACE_THIS_TEXT_WITH_YOUR_YOUTUBE_LINK )

## Arduino Source Code
```cpp
#include <Servo.h>

// --- PIN DEFINITIONS ---
const int ldrPin = A0;             
const int rainSensorPin = 2;       
const int smokeSensorPin = 3;      
const int acSwitchPin = 4;         
const int overrideWindowPin = 5;   
const int overrideNetPin = 6;      

const int windowServoPin = 9;      
const int netServoPin = 10;        

// --- SERVO OBJECTS ---
Servo windowServo;
Servo netServo;

// --- CONFIGURATION THRESHOLDS ---
const int lightThreshold = 500;    // Threshold for Day/Night detection
const int windowOpenAngle = 90;    
const int windowCloseAngle = 0;    
const int netDeployedAngle = 90;   
const int netRetractedAngle = 0;   

void setup() {
  // Serial monitor active for system monitoring
  Serial.begin(9600);
  Serial.println("Smart Window Automation System Initializing...");

  // Sensor pins
  pinMode(ldrPin, INPUT);
  pinMode(rainSensorPin, INPUT);
  pinMode(smokeSensorPin, INPUT);
  
  // Switches (Using INPUT_PULLUP to avoid external resistors)
  pinMode(acSwitchPin, INPUT_PULLUP);
  pinMode(overrideWindowPin, INPUT_PULLUP);
  pinMode(overrideNetPin, INPUT_PULLUP);

  // Attach servos
  windowServo.attach(windowServoPin);
  netServo.attach(netServoPin);

  // Set initial default positions
  windowServo.write(windowCloseAngle);
  netServo.write(netRetractedAngle);
}

void loop() {
  // Read sensor values
  int ldrValue = analogRead(ldrPin);
  int isRaining = digitalRead(rainSensorPin); 
  int isSmoking = digitalRead(smokeSensorPin); 
  int isAcOn = digitalRead(acSwitchPin); 
  
  int manualWindow = digitalRead(overrideWindowPin); 
  int manualNet = digitalRead(overrideNetPin);       

  // Print to Serial Monitor
  Serial.print("LDR: "); Serial.print(ldrValue);
  Serial.print(" | Rain: "); Serial.print(isRaining);
  Serial.print(" | Smoke: "); Serial.print(isSmoking);
  Serial.print(" | AC ON: "); Serial.println(isAcOn == LOW ? "YES" : "NO");

  // ==========================================
  // 1. WINDOW CONTROL LOGIC
  // ==========================================
  if (manualWindow == LOW) {
    windowServo.write(windowOpenAngle);
    Serial.println("ACTION: Window Manually Opened");
  } 
  else if (isSmoking == HIGH) {
    windowServo.write(windowOpenAngle);
    Serial.println("ACTION: Smoke Detected! Window Opened.");
  } 
  else if (isRaining == LOW) {
    windowServo.write(windowCloseAngle);
    Serial.println("ACTION: Rain Detected. Window Closed.");
  } 
  else if (isAcOn == LOW) {
    windowServo.write(windowCloseAngle);
    Serial.println("ACTION: AC is ON. Window Closed.");
  } 
  else {
    windowServo.write(windowOpenAngle);
  }

  // ==========================================
  // 2. MOSQUITO NET CONTROL LOGIC
  // ==========================================
  if (manualNet == LOW) {
    netServo.write(netDeployedAngle);
    Serial.println("ACTION: Net Manually Deployed");
  }
  else if (ldrValue < lightThreshold) {
    netServo.write(netDeployedAngle);
    Serial.println("ACTION: Low Light. Net Deployed.");
  } 
  else {
    netServo.write(netRetractedAngle);
  }

  delay(500); 
}
