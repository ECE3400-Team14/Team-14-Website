# Elevation's robot: Tall Boi

<img src="https://user-images.githubusercontent.com/12742304/49548573-21d1cf00-f8b4-11e8-9a2c-1888b377abac.jpeg" width="500" />

**Above:** Tall Boi next to his 3rd place trophy

## Overview:
Tall boi may not be the most elegant robot, but he is very smart! He can perform all of the all of the key features outlined in the [ECE 3400 Course Description, 2018](https://cei-lab.github.io/ece3400-2018/courseDescription.html), listed below.

### Abilities:

- Start signal: Starts when it hears a 660 Hz audio tone.

- Line following: Follows a white line (~2 cm across) on a dark surface. Detects intersections where perpendicular white lines cross.

- Wall detection: Detects walls at intersections in front, to the left, and to the right

- Maze solving: Keeps track of where it has been and uses Dijkstra's algorithm to find the fastest path to unexplored territory.

- Treasure detection: Detects treasures on right hand walls at intersections

- Radio communication: Sends maze data to an Arduino base station to update the [ECE 3400 GUI](https://github.com/backhous/ece3400-maze-gui).

- Robot detection: Avoids collisions with other robots wearing standard ECE 3400 IR Hats.

### Key Components:

## Performance Review:
