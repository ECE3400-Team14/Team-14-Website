[Return Home](https://ece3400-team14.github.io/Team-14-Website/)
#  Milestone 1

## Overview of Robot Modifications

In order to implement line-following functionality, we added four analog line sensors to our robot. These sensors contain photo-diodes that output a varying voltage based on the light entering the sensor (voltage decreases as light increases). Since the grid is made of white tape on a dark surface, we used the voltage from these sensors to distinguish between when the robot is over the white line (low voltage) and when it is on the black surface (high voltage). 

To gather accurate readings, we mounted the sensors close to the ground using the bent stilt 3D-printed pieces provided in the lab. Because the wheels we used were too small, we had to mount the stilts above the chassis using spacers. We also added spacers to the ground-facing end of the stilts to lower the sensor to the ideal height. We go into more detail on the optimal calbiration of these sensors in later sections. 

<img src="https://user-images.githubusercontent.com/12742304/45529003-f954a000-b7b0-11e8-8332-0f714e077335.jpeg" width="400" />

While we were able to get this design to perform successful line detection and grid navigation, we were unsatisfied with the disorganized wiring and instability of the sensor stilts. We are in the process of performing a redesign of the robot using bigger wheels to raise the chassis and clean up the sensor wiring. Updates on the modified design can be found [here](../OtherUpdates/newWheels.md).

### Circuit Diagram:

<img src="https://user-images.githubusercontent.com/12742304/45528981-e2ae4900-b7b0-11e8-82cb-600af7f6c201.png" width="500" />

## Line Detection

After reading anolog values from the line sensors, we have to determine what values correspond to the white line and what values correspond to the dark surface by setting a threshold. The readings vary across different sensors due to different setup and installations. Thus, we have to calibrate our threashold for each sensor. Here's one of the four functions:

```cpp
/* returns 0 if white detected, 1 if black */
int readLeftSensor() {
  int val = analogRead(0);
  return val > 800? 1:0;
}
```

### Algorithm

In order for the robot the detect the white line for the robot to follow, we decided to use two line sensors close together on the front side of the chassis. These two sensors can provide the robot with the necessary information to orient itself on a line:

* If both sensors detect the line, the robot is aligned with the white line and should move forward.

```cpp
    if (readLeftSensor() == 0 && readRightSensor() == 0)
      forward();
```

* If one sensor detects the line while the other detects the ground, the robot should turn towards the sensor that detects the line until both sensors detect the line.

```cpp
    else if (readLeftSensor() == 0 && readRightSensor() == 1){
      writeRight(180);
      writeLeft(90);
    }
    else if (readLeftSensor() == 1 && readRightSensor() == 0){
      writeLeft(180);
      writeRight(90);
    }
```

* If both sensors do not detect the line, the robot should move backwards (this prevents the robot from leaving the grid, which is bounded by black tape).

### Demonstration of Line Navitation
<iframe width="560" height="315" src="https://www.youtube.com/embed/uoAQjQ9QIC0?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
## Intersection Detection

Once we finished the line detection part, we moved on to the having our robot detect intersections. In addition to the two line sensors already installed, we added two more sensors to achieve this task, mounting to the sides of the othe two line sensors and farther back under the chassis.


### Algorithm

Our robot makes a turn as follows:

* If both sensors in the middle detect the line, but neither of the sensors on the outside detects the line, the robot is between two intersections and keeps tracking the line while moving forward

```cpp
  while (readLeftmostSensor() == 1 || readRightmostSensor() == 1){
    trackLine();
  }
```

* If all four sensors detect a line, the robot hits an intersection, and starts turning left or right, depending on the function called

* The robot is stuck in a while loop until one of the middle sensors moves out of the line

```cpp
  while (readLeftSensor() == 0 && readRightSensor() == 0);
```

* The robot is stuck in another while loop until both middle sensors detect the line

```cpp
  while (readLeftSensor() == 1 || readRightSensor() == 1);
```

* The robot has finished the turn and freezes all motor functions until further instructions

### Demonstration of Figure Eight
At first, we had the turn executed with only one wheel turning, and the other stationary to act as a pivot:
<iframe width="560" height="315" src="https://www.youtube.com/embed/9SsG4Rz07zQ?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

Then, we implemented a faster turn using both wheels. We called this a "dime turn":
<iframe width="560" height="315" src="https://www.youtube.com/embed/m2-dYcXzxF4?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

[Return Home](https://ece3400-team14.github.io/Team-14-Website/)
