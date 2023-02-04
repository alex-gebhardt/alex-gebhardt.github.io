---
layout: page
title: Line-Following Vehicle
short_description: Measurement and Control System II Class Project
long_description: Predesigned chassis with custom line-following code
---

This vehicle was built for Purdue ME 375 - Measurement and Control Systems II. The response of each motor was measured and used to design and implement individual PID controllers for improved straight-line driving. An IR line-following sensor was used to follow a given course. The myRIO hardware was programmed in LabVIEW.

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
    <img src="{{site.baseurl}}/assets/images/LineFollowingRobot1.jpg" alt="overview" style="display: block; margin: auto;" width="91%">
  </div>
  <div class="column">
    <img src="{{site.baseurl}}/assets/images/LineFollowingRobot2.jpg" alt="closeup" style="display: block; margin: auto;">
  </div>
</div> 

<h3>Hardware:</h3>
* NI myRIO
* L298N motor controller
* DC motor (2)
* IR sensor array
* Lithium-ion battery pack


<h3>Electrical Diagram:</h3>
<img src="{{site.baseurl}}/assets/images/LineFollowingRobotElectrical.png" alt="overview" class="responsive" style="display: block; margin: auto;" width="75%">

<h3>LabVIEW Program:</h3>
<img src="{{site.baseurl}}/assets/images/LineFollowingRobotProgram.png" alt="overview" style="display: block; margin: auto;">

