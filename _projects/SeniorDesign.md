---
layout: page
title: Lunar Rover Prototype
short_description: Senior Design Project
long_description: Insert long description here
---

This lunar rover prototype was created as part of a team for Purdue ME 463 - Engineering Design. The project goal was to design an Autonomous Lunar Vehicle that is capable of traversing through unexplored lunar lava tubes with the purpose of determining the environment's ability to sustain life.  Lava tubes are believed to exist beneath the surface of the moon, which may be able to shield inhabitants from the lethal radiation and temperature fluctuations that occur at the surface. Very little is known about these lava tubes, or whether they would be able to sustain life at all. 


<https://engineering.purdue.edu/ME/News/malott-innovation-awards-showcase-2019-senior-design-projects>

<img src="{{site.baseurl}}/assets/images/SeniorDesign_Outdoors2.jpg" alt="outdoors2" style="display: block; margin: auto">

<img src="{{site.baseurl}}/assets/images/SeniorDesign_Outdoors1.png" alt="outdoors1" style="display: block; margin: auto">

<h3>Design:</h3>
* Mass: 23.5 kg
* Modified Ackermann ateering
* Infrared Sensors for side obstacle detection
* Retractable lidar for mapping and navigation
* Differential designed for stability
* Pivot rocker–bogie suspension from Mars rovers
* 6 Wheel drive, 4 wheel steering
* U-brackets on wheels for stability


<h3>Electrical:</h3>
<img src="{{site.baseurl}}/assets/images/SeniorDesign_electrical.png" alt="roverBody1" style="display: block; margin: auto">



<h3>Hardware:</h3>
<img src="{{site.baseurl}}/assets/images/SeniorDesign_budget.png" alt="outdoors1" style="display: block; margin: auto">


<h3>Code:</h3>
```cpp
/*
 Main

 This program controls the QUESTINOS lunar rover prototype.
 
 (1) Header files containing important functions and constants are included.
 Movement.h contains all basic movement functions involving the drive, turning, and linear actuator motors.
 Sense.h contains all basic sensing functions involving the accelerometer, IMUs, lidar, and IR distance sensors.
 Nav.h contains all necessary variables and functions for navigation.
 Map.h contains all necessary variables and functions for mapping.
 
 (2) Sensor and motor pins are set based on the 'Pin Assignment.xlsx' document for the Arduino Mega.
 Driving stepper motors are all controlled using the same 4 pins. [+4 DI/AN]
 Turning stepper motors are independently controlled using 4 pins per motor. [+16 DI/AN]
 The stepper motor used to move the lidar sensor mount uses 4 pins. [+4 DI/AN]
 The servo motor used to rotate the lidar sensor uses 1 pin. [+1 DI]
 Each of the linear actuators use 1 pins. [+2 DI/AN]
 The IR distance sensors each use 1 analog pin. [+2 AN].
 The accelerometer uses 2 pins. [+2 DI]
 The IMUs use 2 pins each. [+4 DI]
 The Lidar uses 2 pins. [+2 DI]
 
 (3) The vehicle is controlled using the logic contained in the 'Policy-Driven Autonomous Driving Logic' flowchart.
 
 Created 1 Apr. 2019
 Modified 1 Apr. 2019
 by Team ALV 2.0

 */

// Include Libraries
#include "Movement.h"
#include "Sense.h"
//#include "Nav.h"
//#include "Map.h"


// Set Motor Pins
Stepper drivers(stepsPerRev_drivers, 10, 11, 12, 13); //drive steppers pins 10 through 13
Stepper turningA(stepsPerRev_turningA, 6, 7, 8, 9); //drive steppers pins 6 through 9
Stepper turningB(stepsPerRev_turningB, 2, 3, 4, 5); //drive steppers pins 10 through 13
Stepper turningE(stepsPerRev_turningE, 14, 15, 16, 17); //drive steppers pins 14 through 17
Stepper turningF(stepsPerRev_turningF, 18, 19, 20, 21); //drive steppers pins 18 through 21
int linear1A = 22; // linear actuator 1 pin A
int linear1B = 23; // linear actuator 1 pin B
int linear2A = 24; // linear actuator 2 pin A
int linear2B = 25; // linear actuator 2 pin B
Stepper lidarArm(stepsPerRev_lidarArm, 26, 28, 30, 32); //lidar arm servo pins 26 through 32 (even)
Servo lidarServo; // lidarServo object
int lidarServoPin = 34; // lidarServo pin

// Set Sensor Pins
Adafruit_FXAS21002C gyro = Adafruit_FXAS21002C(0x0021002C); // IMU gyro object
Adafruit_FXOS8700 accelmag = Adafruit_FXOS8700(0x8700A, 0x8700B); // IMU accelerometer and magnetometer object
Adafruit_LIS3DH accel = Adafruit_LIS3DH(); // Accelerometer object
int ir1 = 54; // IR Sensor 1 Pin
int ir2 = 55; // IR Sensor 2 Pin
RPLidar lidar; // Lidar object
int RPLIDAR_MOTOR = 44; // lidar motor pin

void setup() {
  // Set Stepper Motor Driving Speed
  drivers.setSpeed(driving_speed); // set driving speed
  turningA.setSpeed(turning_speed); // set turning speed
  turningB.setSpeed(turning_speed); // set turning speed
  turningE.setSpeed(turning_speed); // set turning speed
  turningF.setSpeed(turning_speed); // set turning speed
  lidarArm.setSpeed(lidarArm_speed); // set lidarArm speed
  pinMode(linear1A, OUTPUT);
  pinMode(linear1B, OUTPUT);
  pinMode(linear2A, OUTPUT);
  pinMode(linear2B, OUTPUT);
  lidarServo.attach(lidarServoPin);
  
  // Set Up Sensors
  gyro.begin();
  accelmag.begin();
  filter.begin(10);
  accel.setRange(LIS3DH_RANGE_2_G);
  pinMode(ir1, INPUT);
  pinMode(ir2, INPUT);
  lidar.begin(Serial1);
  pinMode(RPLIDAR_MOTOR, OUTPUT);
  // Initialize Lidar Data
  for (int i = 0; i < 2; i++) {
    for (int j = 0; j < 360; j++) {
      lidar_data[i][j] = 0;
    }
  }
  
  // Set Baud Rate
  Serial.begin(9600);
}

void loop() {
  //Set Lidar Motor Speed
  analogWrite(RPLIDAR_MOTOR, 200);
  // Get Raw Sensor Values
  /*-----------------------------------------------------------------------*/
  /* IMU - Accelerometer & Magnetometer Included */
  sensors_event_t gyro_event;
  sensors_event_t accel_event;
  sensors_event_t mag_event;
  gyro.getEvent(&gyro_event);
  accelmag.getEvent(&accel_event, &mag_event);
  float x = mag_event.magnetic.x - mag_offsets[0];
  float y = mag_event.magnetic.y - mag_offsets[1];
  float z = mag_event.magnetic.z - mag_offsets[2];
  float mx = x * mag_softiron_matrix[0][0] + y * mag_softiron_matrix[0][1] + z * mag_softiron_matrix[0][2];
  float my = x * mag_softiron_matrix[1][0] + y * mag_softiron_matrix[1][1] + z * mag_softiron_matrix[1][2];
  float mz = x * mag_softiron_matrix[2][0] + y * mag_softiron_matrix[2][1] + z * mag_softiron_matrix[2][2];
  float gx = gyro_event.gyro.x + gyro_zero_offsets[0];
  float gy = gyro_event.gyro.y + gyro_zero_offsets[1];
  float gz = gyro_event.gyro.z + gyro_zero_offsets[2];
  gx *= 57.2958F;
  gy *= 57.2958F;
  gz *= 57.2958F;
  filter.update(gx, gy, gz,
                accel_event.acceleration.x, accel_event.acceleration.y, accel_event.acceleration.z,
                mx, my, mz);
  float roll = filter.getRoll();
  float pitch = filter.getPitch();
  float heading = filter.getYaw();
  Serial.print("Orientation: ");
  Serial.print(heading);
  Serial.print(" ");
  Serial.print(pitch);
  Serial.print(" ");
  Serial.println(roll);
  delay(10);
  
  /* Accelerometer */
  accel.read(); // Get X, Y, and Z accelerometer data
  Serial.print("Raw Data - ");
  Serial.print("X: ");
  Serial.print(accel.x);
  Serial.print(" \tY: ");
  Serial.print(accel.y);
  Serial.print(" \tZ: ");
  Serial.print(accel.z);
  sensors_event_t accelEvent;
  accel.getEvent(&accelEvent);
  Serial.print("\t--> \tX: ");
  Serial.print(accelEvent.acceleration.x);
  Serial.print(" \tY: ");
  Serial.print(accelEvent.acceleration.y);
  Serial.print(" \tZ: ");
  Serial.print(accelEvent.acceleration.z);
  Serial.println(" m/s^2");
  
  /* IR Sensors */
  ir1_data = analogRead(ir1);
  ir2_data = analogRead(ir2);
  ir1_dist1 = 0.0063*pow(ir1_data,2) - 6.5482*ir1_data + 1814.6;  //from IR calibration data plot
  ir1_dist2 = 0.0993*ir1_data - 31.529; // shorter distance from IR calibration data plot
  ir2_dist1 = 0.0063*pow(ir2_data,2) - 6.5482*ir2_data + 1814.6;  //from IR calibration data plot
  ir2_dist2 = 0.0993*ir2_data - 31.529; // shorter distance from IR calibration data plot
  Serial.print("IR Sensor 1: ");
  Serial.print(ir1_data);
  Serial.print(" ");
  Serial.print(ir1_dist1);
  Serial.print(" ");
  Serial.println(ir1_dist2);
  Serial.print("IR Sensor 2: ");
  Serial.print(ir2_data);
  Serial.print(" ");
  Serial.print(ir2_dist1);
  Serial.print(" ");
  Serial.println(ir2_dist2);
  delay(10);
  
  /* Lidar */
  if (IS_OK(lidar.waitPoint())) {
    for(int i = 0; i < 100; i++) {
      lidar.waitPoint();
      float distance = lidar.getCurrentPoint().distance; //distance value in mm unit
      float angle    = lidar.getCurrentPoint().angle; //anglue value in degree
      bool  startBit = lidar.getCurrentPoint().startBit; //whether this point is belong to a new scan
      byte  quality  = lidar.getCurrentPoint().quality; //quality of the current measurement
      if (quality > 10) {
        lidar_data[0][(int)angle] = distance;
        lidar_data[1][(int)angle] = angle;
        Serial.print("dist: ");
        Serial.print(distance);
        Serial.print("\tangle: ");
        Serial.print(angle);
        Serial.print("\tsbit: ");
        Serial.print(startBit);
        Serial.print("\tquality: ");
        Serial.print(quality);
        Serial.print("\t|\t");
      }
    }
  } 
  else {
    analogWrite(RPLIDAR_MOTOR, 0); //stop the rplidar motor
    
    // try to detect RPLIDAR... 
    rplidar_response_device_info_t info;
    if (IS_OK(lidar.getDeviceInfo(info, 100))) {
       // detected...
       lidar.startScan();
       
       // start motor rotating at max allowed speed
       analogWrite(RPLIDAR_MOTOR, 255);
       delay(1000);
    }
  }
  
  delay(10);
  /*-----------------------------------------------------------------------*/
  // Turning/Driving Variables
  rad = .25; // Turning radius of whole rover body in meters
  dist2drive = 0; // Driving distance in meters (negative ==> Forward)
  
  float LAMAX_R = .1; //Max extended distance of Right Lin. Act.    |
  float LAMAX_L = .1; //Max extended distance of Left Lin. Act.     | Linear
  float LAMIN_R = .05; //Min Compressed Distance of Right Lin. Act. | Actuator
  float LAMIN_L = .05; //Min Compressed Distance of Left Lin. Act.  |
  
  float RMAX_R = .1; //Max extended distance of Right Rear Wheel     |
  float RMAX_L = .1; //Max extended distance of Left Rear Wheel      | Rear
  float RMIN_R = .05; //Min Compressed Distance of Right Rear Wheel  | Wheel
  float RMIN_L = .05; //Min Compressed Distance of Left Rear Wheel   |
  
  float R_Lleg_Front = .25; //Length from middle to left front wheel (Fixed) | Front
  float R_Rleg_Front = .25; //Length from middle to left front wheel (Fixed) | Wheel
  
  float x_Lleg = -.2; //X coordinates of Legs  | X-
  float x_Rleg = .2; //X Coordinates of leg    | Coordinates
  
  //rad = .2; //Turning radius (m) +|+|+|+|+|+|+|+|+|+|+|+|+|+|+|+ INPUT HERE
  float LAL = .05; //Characteristic Length of Left Linear Actuator
  float LAR = .05; //Characteristic Length of Right Linear Actuator


  //Angles measured from Straight Forward, CCW (+), CW (-)
  float R_L = -(RMIN_L + (LAMAX_L-LAL)*(RMAX_L-RMIN_L)/(LAMAX_L-LAMIN_L)); 
                //radius to the back leg; Variable by Lin. Act.
  float R_R = -(RMIN_R + (LAMAX_R-LAR)*(RMAX_R-RMIN_R)/(LAMAX_R-LAMIN_R)); 
                //radius to the back leg; Variable by Lin. Act.
  //Lock wheels before moving
  digitalWrite(2,HIGH);
  digitalWrite(3,LOW);
  digitalWrite(4,HIGH);// B
  digitalWrite(5,LOW);

  digitalWrite(6,HIGH);
  digitalWrite(7,LOW);
  digitalWrite(8,HIGH);// A
  digitalWrite(9,LOW);

  digitalWrite(14,HIGH);
  digitalWrite(15,LOW);
  digitalWrite(16,HIGH);// E
  digitalWrite(17,LOW);

  digitalWrite(36,HIGH);
  digitalWrite(38,LOW);
  digitalWrite(40,HIGH);// F
  digitalWrite(42,LOW);
  
  angA = 0; angB = 0; angE = 0; angF = 0; // Initialize angles to zero
  angA = (atan2(R_Lleg_Front,(x_Lleg-rad)))/3.14159*180;
  if (angA > 90) {
    angA = angA - 180;
  }
  else if (angA <-90) {
    angA = angA + 180;
  }
  angB = (atan2(R_Rleg_Front,(x_Rleg-rad)))/3.14159*180;
  if (angB > 90) {
    angB = angB - 180;
  }
  else if (angB <-90) {
    angB = angB + 180;
  }
  angE = (atan2(R_L,(x_Lleg-rad)))/3.14159*180;
  if (angE > 90) {
    angE = angE - 180;
  }
  else if (angE <-90) {
    angE = angE + 180;
  }
  angF = (atan2(R_R,(x_Rleg-rad)))/3.14159*180;
  if (angF > 90) {
    angF = angF - 180;
  }
  else if (angF <-90) {
    angF = angF + 180;
  }
  delay(5000);
  
  // Lock wheels if radius is high
  if (abs(rad) > 6){
    digitalWrite(2,HIGH);
    digitalWrite(3,LOW);
    digitalWrite(4,HIGH);// B
    digitalWrite(5,LOW);

    digitalWrite(6,HIGH);
    digitalWrite(7,LOW);
    digitalWrite(8,HIGH);// A
    digitalWrite(9,LOW);

    digitalWrite(14,HIGH);
    digitalWrite(15,LOW);
    digitalWrite(16,HIGH);// E
    digitalWrite(17,LOW);

    digitalWrite(36,HIGH);
    digitalWrite(38,LOW);
    digitalWrite(40,HIGH);// F
    digitalWrite(42,LOW);
  }

  delay(1000); //1 sec delay
  
  // Call Functions to Autonomously Control Rover
  /*-----------------------------------------------------------------------*/
  rad = 0.5;
  int funcVal = turn(rad, lastRad);
  if (funcVal == 0) {
    Serial.println("Turn: Success.");
  }
  dist2drive = 1;
  funcVal = straight(dist2drive);
  if (funcVal == 0) {
    Serial.println("Drive: Success.");
  }
  funcVal = stop();
  if (funcVal == 0) {
    Serial.println("Stop: Success.");
  }
  
  delay(5000);
  
  lastRad = rad; // Used to determine whether or not the main wheels need to change at all
}


int turn(float rad, float lastRad) {
  if (lastRad != rad){
    Serial.println("BEFORE CONVERSION");
    Serial.print("radius: ");Serial.println(rad);
    Serial.print("A: ");Serial.print(angA);Serial.print("\t\tB: ");Serial.println(angB);
    Serial.print("E: ");Serial.print(angE);Serial.print("\t\tF: ");Serial.println(angF);

    float inc_A = floor(angA*stepsPerRev_turningA/360/100);
    float inc_B = floor(angB*stepsPerRev_turningB/360/100);
    float inc_E = floor(angE*stepsPerRev_turningE/360/100);
    float inc_F = floor(angF*stepsPerRev_turningF/360/100);

    Serial.println("AFTER CONVERSION");
    Serial.print("radius: ");Serial.println(rad);
    Serial.print("A: ");Serial.print(inc_A);Serial.print("\t\tB: ");Serial.println(inc_B);
    Serial.print("E: ");Serial.print(inc_E);Serial.print("\t\tF: ");Serial.println(inc_F);
    drivers.step(stepsPerRev_drivers*dist2drive*5/.15/3.14159); // Rotates all of the Drive wheels forwards set distance.

    delay(1000); //1 sec delay
    
    turningA.step(int(angA*stepsPerRev_turningA/360));                   //Rotates Steering Wheels
    Serial.print(angA*stepsPerRev_turningA/360);Serial.print("\t");
    turningB.step(int(angB*stepsPerRev_turningB/360));
    Serial.print(angB*stepsPerRev_turningB/360);Serial.println();
    turningE.step(int(angE*stepsPerRev_turningE/360));
    Serial.print(angE*stepsPerRev_turningE/360);Serial.print("\t");
    turningF.step(int(angF*stepsPerRev_turningF/360));
    Serial.print(angF*stepsPerRev_turningF/360);Serial.print("\n\n");

    delay(1000); //1 sec delay

    drivers.step(int(stepsPerRev_drivers*dist2drive*5/.15/3.14159)); // Rotates all of the Drive wheels forwards set distance.c

    delay(1000); //1 sec delay

  
    turningA.step(int(-angA*stepsPerRev_turningA/360));// |
    turningB.step(int(-angB*stepsPerRev_turningB/360));// | Returns Rotating Steppers to zero
    turningE.step(int(-angE*stepsPerRev_turningE/360));// | 
    turningF.step(int(-angF*stepsPerRev_turningF/360));// | 
    return 0;
  }
}

int straight(float dist2drive) {
  drivers.step(stepsPerRev_drivers*dist2drive*5/.15/3.14159);
  return 0;
}

int stop() {
    digitalWrite(2,HIGH);
    digitalWrite(3,LOW);
    digitalWrite(4,HIGH);// B
    digitalWrite(5,LOW);

    digitalWrite(6,HIGH);
    digitalWrite(7,LOW);
    digitalWrite(8,HIGH);// A
    digitalWrite(9,LOW);

    digitalWrite(14,HIGH);
    digitalWrite(15,LOW);
    digitalWrite(16,HIGH);// E
    digitalWrite(17,LOW);

    digitalWrite(36,HIGH);
    digitalWrite(38,LOW);
    digitalWrite(40,HIGH);// F
    digitalWrite(42,LOW);
    
    digitalWrite(10,HIGH);
    digitalWrite(11,LOW);
    digitalWrite(12,HIGH);// Drivewheels
    digitalWrite(13,LOW);

    return 0;
}

```

```cpp
// Sense.h

// Include Libraries
#include <Stepper.h>
#include <Servo.h>

// Constants
const int stepsPerRev_drivers = 200;  // drivers steppers steps/rev
const int stepsPerRev_turningA = 200; //turning motors 1 steps/rev    *front
const int stepsPerRev_turningB = 200; //turning motors 2 steps/rev    *front
const int stepsPerRev_turningE = 200; //turning motors 2 steps/rev    *back
const int stepsPerRev_turningF = 200; //turning motors 2 steps/rev    *back
const int stepsPerRev_lidarArm = 200; //lidar rack and pinion steps/rev
const int driving_speed = 60; // [rpm]
const int turning_speed = 10; // [rpm]
const int lidarArm_speed = 30; // [rpm]
float lastRad = 10; // last turning radius [m]
float rad = 0; // turning radius [m]
float angA = 0; float angB = 0; float angE = 0; float angF = 0; // [degrees] wheel angles
float dist2drive = 0;

// Function Declarations
int sign(int num);
int turn(float rad, float lastRad);
int straight(float distance);
int stop();
```

```cpp
// Movement.h

#include <Wire.h>
#include <SPI.h>
#include <Adafruit_Sensor.h>
#include <Mahony.h>
#include <Madgwick.h>
#include <Adafruit_LIS3DH.h>
#include <Adafruit_FXAS21002C.h>
#include <Adafruit_FXOS8700.h>
#include <RPLidar.h>

// Constants
float mag_offsets[3]            = { 0.93F, -7.47F, -35.23F };
float mag_softiron_matrix[3][3] = { {  0.943,  0.011,  0.020 },
                                    {  0.022,  0.918, -0.008 },
                                    {  0.020, -0.008,  1.156 } };
float mag_field_strength        = 50.23F;
float gyro_zero_offsets[3]      = { 0.0F, 0.0F, 0.0F };

// Global Variables
Mahony filter; // Filter for IMU orientation data
int servo_pos = 0; // Servo position
int ir1_data = 0; // IR sensor 1 raw data
int ir2_data = 0; // IR sensor 2 raw data
int ir1_dist1 = 0; //[cm] 100cm to 500cm
int ir1_dist2 = 0; //[cm] 0cm to 50cm
int ir2_dist1 = 0; //[cm] 100cm to 500cm
int ir2_dist2 = 0; //[cm] 0cm to 50cm
float lidar_data[2][360];

// Function Declarations
```



<video src="{{site.baseurl}}/assets/images/SeniorDesign_MotionStudy.mp4" controls="controls" style="display: block; margin: auto";></video>

<img src="{{site.baseurl}}/assets/images/SeniorDesign_RoverBody1.jpg" alt="roverBody1" style="display: block; margin: auto">


<img src="{{site.baseurl}}/assets/images/SeniorDesign_RoverBody2.jpg" alt="roverBody2" style="display: block; margin: auto">

<img src="{{site.baseurl}}/assets/images/SeniorDesign_RoverBody3.jpg" alt="roverBody3" style="display: block; margin: auto">

<img src="{{site.baseurl}}/assets/images/SeniorDesign_RoverBody4.jpg" alt="roverBody4" style="display: block; margin: auto">

<img src="{{site.baseurl}}/assets/images/SeniorDesign_RoverBody5.jpg" alt="roverBody5" style="display: block; margin: auto">

<img src="{{site.baseurl}}/assets/images/SeniorDesign_RoverBody6.jpg" alt="topView1" style="display: block; margin: auto">

<img src="{{site.baseurl}}/assets/images/SeniorDesign_RoverBody7.jpg"" alt="topView2" style="display: block; margin: auto">

<img src="{{site.baseurl}}/assets/images/SeniorDesign_RoverBody8.jpg"" alt="topView3" style="display: block; margin: auto">

<img src="{{site.baseurl}}/assets/images/SeniorDesign_AngleTest1.jpeg" alt="angleTest1" style="display: block; margin: auto">

<img src="{{site.baseurl}}/assets/images/SeniorDesign_AngleTest2.jpeg" alt="angleTest2" style="display: block; margin: auto">

<video src="{{site.baseurl}}/assets/images/SeniorDesign_ObstacleTest.mp4" controls="controls" style="display: block; margin: auto"></video>

<video src="{{site.baseurl}}/assets/images/SeniorDesign_TurningTest1.mp4" controls="controls" style="max-width: 500px;"></video>

<video src="{{site.baseurl}}/assets/images/SeniorDesign_RockerBogieTest.mp4" controls="controls" style="display: block; margin: auto;"></video>

<video src="{{site.baseurl}}/assets/images/SeniorDesign_ObstacleTest.mp4" controls="controls" style="max-width: 500px;"></video>

<img src="{{site.baseurl}}/assets/images/SeniorDesign_mapping1.png" alt="lidarMap" style="display: block; margin: auto">
