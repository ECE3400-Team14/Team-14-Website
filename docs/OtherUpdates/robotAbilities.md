[Return Home](https://ece3400-team14.github.io/Team-14-Website/)
# Elevation's Robot

<img src="https://user-images.githubusercontent.com/12742304/49548573-21d1cf00-f8b4-11e8-9a2c-1888b377abac.jpeg" width="500" />

**Above:** Tall Boi next to his 3rd place trophy

## Overview:
Tall boi may not be the most elegant robot, but he is very smart! Tall Boi is outfitted with almost entirely default materials provided by ECE 3400, and is very capable with just those pieces! You can see where our robot, Tall Boi, gets his name! Three tiers of electrical engineering effectiveness brings his stature to quite a height. This was so he could perform all of the all of the key features outlined in the [ECE 3400 Course Description, 2018](https://cei-lab.github.io/ece3400-2018/courseDescription.html), listed below.


### Capabilities

- Start signal: Starts when it hears a 660 Hz audio tone (using FFT).

- Line following: Follows a white line (~2 cm across) on a dark surface. Detects intersections where perpendicular white lines cross.

- Wall detection: Detects walls at intersections in front, to the left, and to the right

- Maze solving: Keeps track of where it has been and uses Dijkstra's algorithm to find the fastest path to unexplored territory.

- Treasure detection: Detects treasures on right hand walls at intersections.

- Radio communication: Sends maze data to an Arduino base station to update the [ECE 3400 GUI](https://github.com/backhous/ece3400-maze-gui).

- Robot detection: Avoids collisions with other robots wearing standard ECE 3400 IR Hats (detected using FFT).

To see our robot's abilities in action, check out our robot's performance during the [final competition](../finalCompetition.md). Unfortunately, the competition does not show off our robot's treasure detection abilities very well, but we have demos of treasure detection in action [here]().

### Hardware Included

* Larger wheels with rubber band treads as grips.
* A low sitting chassis for consistent line tracking.
* Ball bearings in the rear and just behind the line sensors in the front for stability.
* Four photosensitive line sensors, three short range IR distance sensors (wall sensing), attached to a MUX for easy analog reading.
* A microphone and attached amplifier circuit.
* IR phototransistor for robot detection (MUX'd with the audio circuit since audio is only required for the starting command).
* An Arduino Genuino Uno for all processing except for treasure detection and for I/O.
* An FPGA and accompanying CMOS OV7670 camera for imaging, image processing, and transmission of shape/color data to the Arduino.
* A radio [brand? make? model?] for base station data transmission.

The base station has its own radio and Arduino Uno for updating the GUI (an interface provided by the 3400 staff).

Everything is mounted to three main chassis plates, supported by struts, nuts, and bolts, hot glue, tape, and anything else to hold it together (including the sweat and tears of the invested Elevation Team).


## Performance Review:

Tall Boi can track lines and intersections with great accuracy (if batteries are propery fueled), sense walls on three sides
for maze mapping, detect a 660Hz audio start signal, detect other robots' IR signals, and detect treasure color and location.
In terms of software, it uses wall sensor information in a search algorithm for navigation the maze,
performs fast Fourier transform algorithms to detect audio and IR signal frequencies, transmits maze data to a base station over radio frequency communication, and adapts to detected robots in its vicinity.

[Return Home](https://ece3400-team14.github.io/Team-14-Website/)
