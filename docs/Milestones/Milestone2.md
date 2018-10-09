#  Milestone 1

## Overview of Robot Modifications

In this milestone, we aimed to use line tracking (previously developed) to implement a wall-following algorithm 
to navigate an arbitrary set of walls while avoiding decoy robots. This integrates multiple systems explored in previous labs
into a working system on the robot.

### Wall Follow Algorithm

The basic wall following algorithm is a simple set of rules that hold the robot hugging a right-hand wall. If there is a wall on the right
the robot will go straight. If there is a wall on the right, and a wall in front, the robot will turn left. Finally, if there is no wall on the right,
the robot should turn right in order to find the wall again.

Our robot is tuned such that it will only sense a wall at quite close range. This is so that the robot only has to check for a wall at every intersection.
This is why, in the video below, the LED indicators only change their values at intersections. That is where turn decisions are made.

The green light on the right side of the robot indicates that the robot "sees" a wall on the right side.
The yellow light on the front of the robot indicates that the robot"sees" a wall in front of the robot.

[Robot wall follow demonstration video]

Notice that this algorithm can successfully navigate any set of walls until hitting an "edge" of our table.

### Robot Avoidance Algorithm

### Both Wall and Robot Avoidance
