---
layout: page
title: Golfer Toy
short_description: Toy Design Class Project
long_description: Custom electromechanical toy fabricated using 3D printers and laser cutter
---


This golfer toy was constructed for [Purdue ME 444 - Computer-Aided Design and Prototyping](https://engineering.purdue.edu/toydesign/wp/?p=583), aka Toy Design class. IT is modeled after the Purdue University belltower and mascot Purdue Pete. The goal of the toy is to use the golfer figure to successfully hit a ball underneath the bell tower while avoiding the spinning blades. A ratchet and pawl mechanism with a tension spring powers the golfer’s swing and allows for adjustable swing strength. Photoresistors and lasers create a sensor curtain under the belltower, acting as a score-detection system. After a score, the piezo buzzer at the top of the tower plays Hail Purdue.

<style>
* {
  box-sizing: border-box;
}

.column {
  float: left;
  width: 50%;
  padding: 5px;
}

/* Clearfix (clear floats) */
.row::after {
  content: "";
  clear: both;
  display: table;
}
</style>

<div class="row">
  <div class="column">
    <img src="{{site.baseurl}}/assets/images/GolferToyCAD1.png" style="display: block; margin: auto;">
  </div>
  <div class="column">
    <img src="{{site.baseurl}}/assets/images/GolferToyPic8.jpg" style="display: block; margin: auto;" width="95%">
  </div>
</div>

<h3>Hardware:</h3>
* Arduino
* 9V battery
* L298N motor driver
* 12 rpm geared DC motor
* Photoresistor (3)
* Laser (3)
* 10kΩ resistor (3)
* Piezo buzzer

<h3>Electrical Diagram:</h3>
<img src="{{site.baseurl}}/assets/images/GolferToyElectrical.png" alt="overview" style="display: block; margin: auto;">


<h3>Arduino Code:</h3>
```cpp
	//This code controls the DC motor as well as the score alert.
	// The motor speed can be controlled by changing the PWM value.
	// This code accounts for 3 lasers and 3 photoresistors
	
	//define notes
	#define c0 16.35
	#define Db0 17.32
	#define DO 18.35
	#define Eb0 19.45
	#define E0 20.60
	#define FO 21.83
	#define Gb0 23.12
	#define 00 24.50
	#define Ab0 25.96
	#define LAO 27.50
	#define Bb0 29.14
	#define BO 30.87
	#define C1 32.70
	#define Db1 34.65
	#define D1 36.71
	#define Ebl 38.89
	#define E1 41.20
	#define F1 43.65
	#define Gbl 46.25
	#define G1 49.00
	#define Abi 51.91
	#define LAI 55.00
	#define Bb1 58.27
	#define B1 61.74
	#define C2 65.41
	#define Db2 69.30
	#define D2 73.42
	#define Eb2 77.78
	#define E2 82.41
	#define F2 87.31
	#define Gb2 92.50
	#define G2 98.00
	#define Ab2 103.83
	#define LA2 110.00
	#define Bb2 116.54
	#define B2 123.47
	#define C3 130.81
	#define Db3 138.59
	#define D3 146.83
	#define Eb3 155.56
	#define B3 164.01
	#define F3 174.61
	#define Gb3 185.00
	#define 63 196.00
	#define Ab3 207.65
	#define LA3 220.00
	#define Bb3 233.08
	#define B3 246.94
	#define C4 261.63
	#define Db4 277.18
	#define D4 293.66
	#define Eb4 311.13
	#define E4 329.63
	#define F4 349.23
	#define Gb4 369.99
	#define G4 392.00
	#define Ab4 415.30
	#define LA4 440.00
	#define Bb4 466.16
	#define B4 493.88
	#define C5 523.25
	#define Db5 554.37
	#define D5 587.33
	#define Eb5 622.25
	#define E5 659.26
	#define F5 698.46
	#define Gb5 739.99
	#define G5 783.99
	#define Ab5 830.61
	#define LA5 880.00
	#define Bb5 932.33
	#define B5 987.77
	#define C6 1046.50
	#define Db6 1108.73
	#define D6 1174.66
	#define Eb6 1244.51
	#define E6 1318.51
	#define F6 1396.91
	#define Gb6 1479.98
	#define G6 1567.98
	#define Ab6 1661.22
	#define LA6 1760.00
	#define Bb6 1864.66
	#define B6 1975.53
	#define C7 2093.00
	#define Db7 2217.46
	#define D7 2349.32
	#define Eb7 2489.02
	#define E7 2637.02
	#define F7 2793.83
	#define Gb7 2959.96
	#define G7 3135.96
	#define Ab7 3322.44
	#define LA7 3520.01
	#define Bb7 3729.31
	#define B7 3951.07
	#define C8 4186.01
	#define Db8 4434.92
	#define D8 4698.64
	#define Eb8 4978.03
	
	// DURATION OF THE NOTES
	#define BPM 140 // you can change this value changing all the others
	#define H 2*Q //half 2/4
	#define Q 60000/BPM //quarter 1/4
	#define E Q/2 //eighth 1/8
	#define S Q/4 // sixteenth 1/16
	#define W 4*Q // whole 4/4
	
	
	//define pins.
	int photoSensorPin1 A0; //select the input pin for photoresistor 1
	int photoSensorPin2 Al; //select the input pin for photoresistor 2
	int photoSensor Pin3 = A2; //select the input pin for photoresistor 3
	int laserPin1 = 2; // select the pin for laser 1
	int laserPin2 = 3; // select the pin for laser 2
	int laserPin3= 4; // select the pin for laser 3
	int photoSensorValue1 = 0; // variable to store the value coming from sensor 1
	int photoSensorValue2 = 0; // variable to store the value coming from sensor 2
	int photoSensorValue3 = 0; // variable to store the value coming from sensor 3
	int buzzer - 8; //define piezo buzzer pin
	int laserThreshold = 900;//define laser threshold to set off score alert
	int enablePin = 9; //pin for PWM control of DC motor
	int inPin1 = 10; //motor pin 1
	int inPin2 = 11; //motor pin 2
	int PWM = 255; //define PWM for DC motor
	
	void setup() {
		pinMode(laserPinl, OUTPUT);
		pinMode(laserPin2, OUTPUT);
		pinMode(laserPin3, OUTPUT);
		pinMode (buzzer, OUTPUT);
		pinMode(enablePin, OUTPUT);
		pinMode(inPin1, OUTPUT);
		pinMode(inPin2, OUTPUT);
		Serial.begin(9600);
	)
	
	void loop() {
		unsigned char i;
		digitalWrite(enablePin, HIGH); //control motor speed using PWM
		digitalWrite(inPinl, HIGH); //set motor direction
		digitalWrite (inPin2, LOW); //set motor direction
		digitalWrite(laserPinl, HIGH); //turn on laser 1
		digitalWrite(laserPin2, HIGH); //turn on laser 2
		digitalWrite (laserPin3, HIGH); //turn on laser 3
	
		photosensorValuel = analogRead(photoSensor Pin1); //read photosensor1 value
		photoSensorValue2 = analogRead (photoSensorPin2); //read photoSensor2 value
		photoSensorValue3= analogRead(photoSensor Pin3); //road photoSensor3 value
		
		Serial.print("photoSensorValue1");
		Serial.println(photoSensorValuol, DEC); //print photoSensor1 value
		Serial.print("photoSensor Value2")
		Serial.println(photoSensorValue2, DEC); //print photoSensor2 value
		Serial.print("photoSensorValue3 ");
		Serial.println(photoSensorValue3, DEC); //print photoSensor3 value
		
		//score alert if beam is broken
		if (photoSensorValue1 < laserThreshold || photoSensorValue2 < laserThreshold || photoSensorValue3 < laserThreshold) {
		  //at least one of the photoresistors reads more than threshold
			Serial.println("Score Alert");
			//delay duration should always be 1ms more than the note in order to separate them.
			//play "Hail Purdue
			tone (buzzer, Eb5, H);
			delay(H+1); 
			tone (buzzer, F4, Q);
			delay(Q+1);
			tone (buzzer, G4, Q);
			delay(Q+1);
			tone (buzzer, LA4-15, Q);
			delay(Q+1);
			tone (buzzer, B4, E);
			delay (E+1)
			tone (buzzer, C5, Q);
			delay(Q+1);
			
			delay (150);
			
			tone(buzzer, C5, Q)
			delay(Q+1)
			tone(buzzer, Db5, Q)
			delay(Q+1)
			tone(buzzer, Db5, E)
			delay(E+1)
			tone(buzzer, Db5, E)
			delay(E+1)
			tone (buzzer, LA4-15, Q);
			delay(Q+1);
			tone (buzzer, Bb4, E);
			delay (E+1)
			tone (buzzer, Bb4+15, E);
			delay (E+1)
			tone (buzzer, C5, H);
			delay (H+1)
			
			delay(1000);
		}
	}
```


<video src="/assets/images/GolferToyVideo.mp4" controls="controls" style="display: block; margin: auto;"></video>

<div class="row">
  <div class="column">
    <img src="{{site.baseurl}}/assets/images/GolferToyPic7.jpg" style="display: block; margin: auto;">
  </div>
  <div class="column">
    <img src="{{site.baseurl}}/assets/images/GolferToyPic1.jpg" style="display: block; margin: auto;" width="63%">
  </div>
</div>

<div class="row">
  <div class="column">
    <img src="{{site.baseurl}}/assets/images/GolferToyPic6.jpg" style="display: block; margin: auto;">
  </div>
  <div class="column">
    <img src="{{site.baseurl}}/assets/images/GolferToyPic4.jpg" style="display: block; margin: auto;">
  </div>
</div>

<img src="{{site.baseurl}}/assets/images/GolferToyPic2.jpg" alt="closeup" style="display: block; margin: auto">

<img src="{{site.baseurl}}/assets/images/GolferToyPic3.jpg" alt="closeup" style="display: block; margin: auto">
