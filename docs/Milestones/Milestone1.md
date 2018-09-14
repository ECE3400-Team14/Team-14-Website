#  Milestone 1

## Overview of Robot Modifications

In order to implement line-following functionality into our robot, we added four analog line sensors to our robot. These sensors contain photo-diodes that output a varying voltage based on the light entering the sensor (voltage decreases as light increases). Since the grid is made of white tape on a dark surface, we used the voltage from these sensors to distinguish when the robot is over the white line (low voltage) and when it is on the black surface (high voltage). 

To gather accurate readings, we mounted the sensors close to the ground using the bent stilt 3-D-printed pieces provided in the lab. Because the wheels we used were too small, we had to mount the stilts above the chassis using spacers. We also added spacers to the ground-facing end of the stilts to lower the sensor to the ideal height. We go into more detail on the optimal calbiration of these sensors in later sections. 

![Robot with Sensors Image Here]()
![Sensor Diagram Here]()

While we were able to get this design to perform successful line detection and grid navigation, we were unsatisfied with the disorganized wiring and instability of the sensor stilts. We are in the process of performing a redesign of the robot using bigger wheels to raise the chassis and clean up the sensor wiring. Updates will be added with the modified design. 

## Line Detection

### Algorithm
In order for the robot the detect the white line for the robot to follow, we decided to use two line sensors close together on the front side of the chassis. These two sensors can provide the robot with the necessary information to orient itself on a line:

* If both sensors detect the line, the robot is aligned with the white line and should move forward
* If one sensor detects the line while the other detects the ground, the robot should turn towards the sensor that detects the line until both sensors detect the line
* If both sensors do not detect the line, the robot should move backwards (this prevents the robot from leaving the grid, which is bounded by black tape)


### Demonstration of Line Navitation
[insert video of line tracking]
## Intersection Detection

### Algorithm
Once we finished the line detection part, we moved on to the having our robot detect intersections. In addition to the two line sensors already installed, we added two more sensors to achieve this task.

[insert close up shot on all four sensors, if available]

Our robot makes a turn as follows:

* If both sensors in the middle detect the line, but neither of the sensors on the outside detects the line, the robot is between two intersections and keeps tracking the line while moving forward

```cpp
  while (readLeftmostSensor() == 1 || readRightmostSensor() == 1){
    trackLine();
  }
```

* If all four sensors detect a line, the robot hits an intersection, and starts turning left or right, depending on the function called
* The robot is stuck in an infinite loop until one of the middle sensors moves out of the line

```cpp
  while (readLeftSensor() == 0 && readRightSensor() == 0);
```

* The robot is stuck in another infinite loop until both middle sensors detect the line

```cpp
  while (readLeftSensor() == 1 || readRightSensor() == 1);
```

* The robot has finished the turn and freezes all motor functions until further instructions
### Demonstration of Figure Eight
[insert video of figure eight]
