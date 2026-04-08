/**
 * ============================================================
 *  Autonomous Maze Solver Robot - Control Firmware
 * ============================================================
 *  Description:
 *  This firmware implements a left wall following autonomous
 *  navigation strategy for a differential drive robot.
 *
 *  Key Features:
 *  - Ultrasonic-based path detection (front + left)
 *  - IR-based lateral drift correction
 *  - Gyroscope based precise 90° turning (MPU6050)
 *  - Real time decision making for maze traversal
 *
 *  Platform:
 *  - Arduino uno
 *
 *  Author: Pavan Pai
 * ============================================================
 */

#include <Wire.h>
#include <MPU6050.h>

// ================== GLOBAL OBJECTS ===========================

MPU6050 mpu;

//  ================== MOTOR PIN CONFIG  ================== 
const int ENA = 11;  // Right motor PWM
const int IN1 = 10;
const int IN2 = 9;

const int IN3 = 8;
const int IN4 = 7;
const int ENB = 6;   // Left motor PWM

// ================== SENSOR PIN CONFIG  ================== 
// Ultrasonic (Front)
const int F_TRIG = 5;
const int F_ECHO = 4;

// Ultrasonic (Left)
const int L_TRIG = 2;
const int L_ECHO = 3;

// IR Sensors (Active LOW)
const int IR_RIGHT = 13;
const int IR_LEFT  = 12;

//================== FUNCTION DECLARATIONS ====================
void motorControl(int in1, int in2, int in3, int in4, int pwmA, int pwmB);
float readUltrasonic(int trig, int echo);
void rotateBot(char direction);

//================== SETUP FUNCTION ===========================
void setup() {
  Serial.begin(115200);

  // Motor Pins
  pinMode(ENA, OUTPUT);
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);
  pinMode(ENB, OUTPUT);

  // Ultrasonic Sensors
  pinMode(F_TRIG, OUTPUT);
  pinMode(F_ECHO, INPUT);
  pinMode(L_TRIG, OUTPUT);
  pinMode(L_ECHO, INPUT);

  // IR Sensors
  pinMode(IR_RIGHT, INPUT);
  pinMode(IR_LEFT, INPUT);

  // MPU Initialization
  Wire.begin();
  mpu.initialize();

  // Ensure motors are stopped at startup
  motorControl(0, 0, 0, 0, 0, 0);

  delay(10000);  // Startup delay (stabilization / manual placement)
}

//================== MAIN LOOP ================================
void loop() {

  // ----------- Drift Correction using IR Sensors ----------- 

  // Right IR triggered → drift towards right wall → steer left
  if (!digitalRead(IR_RIGHT)) {
    while (!digitalRead(IR_RIGHT)) {
      motorControl(1, 0, 0, 0, 85, 85);
    }
  }

  // Left IR triggered → drift towards left wall → steer right
  if (!digitalRead(IR_LEFT)) {
    while (!digitalRead(IR_LEFT)) {
      motorControl(0, 0, 1, 0, 85, 85);
    }
  }

  /* ----------- Left Wall Priority Check ----------- */

  float leftDist = readUltrasonic(L_TRIG, L_ECHO);

  // If path available on left → take left turn
  if (leftDist > 15) {
    motorControl(0, 0, 0, 0, 0, 0);
    delay(1000);
    rotateBot('L');
  }

  /* ----------- Forward Path Check ----------- */

  float frontDist = readUltrasonic(F_TRIG, F_ECHO);

  if (frontDist > 14) {
    // Move forward
    motorControl(1, 0, 1, 0, 70, 70);
    Serial.println("Moving Forward");
  } else {
    // Obstacle ahead → stop and evaluate
    motorControl(0, 0, 0, 0, 0, 0);
    delay(1000);

    // If no left path → turn right
    if (readUltrasonic(L_TRIG, L_ECHO) <= 15) {
      rotateBot('R');
    }
  }
}

/* ============================================================
   ================== MOTOR CONTROL ============================
   ============================================================ */
/**
 * Controls motor direction and speed.
 * 
 * @param in1, in2 → Right motor direction
 * @param in3, in4 → Left motor direction
 * @param pwmA → Right motor speed
 * @param pwmB → Left motor speed
 */
void motorControl(int in1, int in2, int in3, int in4, int pwmA, int pwmB) {
  digitalWrite(IN1, in1);
  digitalWrite(IN2, in2);
  analogWrite(ENA, pwmA);

  digitalWrite(IN3, in3);
  digitalWrite(IN4, in4);
  analogWrite(ENB, pwmB);

  delay(10);  // Small stabilization delay
}

/* ============================================================
   ================== ULTRASONIC SENSOR ========================
   ============================================================ */
/**
 * Measures distance using ultrasonic sensor.
 * 
 * @return Distance in cm
 */
float readUltrasonic(int trig, int echo) {
  digitalWrite(trig, LOW);
  delayMicroseconds(2);

  digitalWrite(trig, HIGH);
  delayMicroseconds(10);
  digitalWrite(trig, LOW);

  float duration = pulseIn(echo, HIGH);
  float distance = 0.034 * duration / 2;

  Serial.print("Distance: ");
  Serial.println(distance);

  return distance;
}

/* ============================================================
   ================== ROTATION FUNCTION ========================
   ============================================================ */
/**
 * Performs precise 90° rotation using MPU6050 gyroscope.
 * 
 * @param direction → 'L' for left, 'R' for right
 */
void rotateBot(char direction) {

  float gyroBias = 0;
  float yaw = 0;

  unsigned long lastTime = 0;

  // Wake MPU
  mpu.setSleepEnabled(false);
  delay(100);

  /* ----------- Bias Calibration ----------- */
  long sum = 0;
  for (int i = 0; i < 500; i++) {
    sum += mpu.getRotationZ();
    delay(2);
  }
  gyroBias = (sum / 500) / 131.0;

  /* ----------- Rotation Loop ----------- */

  yaw = 0;
  lastTime = millis();

  while (true) {

    unsigned long currentTime = millis();
    float dt = (currentTime - lastTime) / 1000.0;
    lastTime = currentTime;

    int16_t gzRaw = mpu.getRotationZ();
    float gzDps = gzRaw / 131.0;

    float correctedGz = gzDps - gyroBias;

    yaw += correctedGz * dt;

    // Stop at ~90°
    if (abs(yaw) >= 90) {
      mpu.setSleepEnabled(true);
      break;
    }

    // Apply rotation
    if (direction == 'R') {
      motorControl(0, 1, 1, 0, 70, 70);
    } else if (direction == 'L') {
      motorControl(1, 0, 0, 1, 70, 70);
    }

    delay(10);
  }

  /* ----------- Stop after rotation ----------- */
  motorControl(0, 0, 0, 0, 0, 0);
  delay(1000);

  /* ----------- Re-align with wall ----------- */
  while (readUltrasonic(L_TRIG, L_ECHO) > 15) {
    motorControl(1, 0, 1, 0, 70, 70);
  }
}
