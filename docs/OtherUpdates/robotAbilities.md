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

To see our robot's abilities in action, check out our robot's performance during the [final competition](../finalCompetition.md). Unfortunately, the competition does not show off our robot's treasure detection abilities very well, but we have demos of treasure detection in action [here](preCompetitionUpdates.md).

### Hardware Included

* Larger wheels with rubber band treads as grips.
* A low sitting chassis for consistent line tracking.
* Ball bearings in the rear and just behind the line sensors in the front for stability.
* Four photosensitive analog line sensors for line following.
* Three short range IR distance sensors for wall sensing.
* Two analog [CD74HC4051 multiplexers](http://www.ti.com/lit/ds/symlink/cd74hc4051.pdf) for easy analog reading. One is used for line sensor and wall sensor inputs, the other is used to switch between audio and IR signal inputs. 
* A microphone and attached amplifier circuit using an [LM258 op amp](http://www.ti.com/lit/ds/symlink/lm158-n.pdf).
* [OP598A](http://www.mouser.com/ds/2/414/OP593-598-6739.pdf) IR phototransistor for robot detection. 
* An [Arduino Genuino Uno](https://store.arduino.cc/usa/arduino-uno-rev3) for all processing except for treasure detection and for I/O.
* A [CMOS OV7670 camera](http://web.mit.edu/6.111/www/f2016/tools/OV7670_2006.pdf) for seeing treasures. 
* A [DE0-NANO FPGA](http://www.ti.com/lit/ug/tidu737/tidu737.pdf) for image processing.
* A [Nordic nRF24L01+ radio tranceiver](https://www.sparkfun.com/datasheets/Components/SMD/nRF24L01Pluss_Preliminary_Product_Specification_v1_0.pdf) for transmission to the Arduino base station.

The base station has its own Nordic nRF24L01+ radio and Arduino Genuino Uno for updating the GUI (an interface provided by the 3400 staff).

Everything is mounted to three main chassis plates, supported by struts, nuts, and bolts, hot glue, tape, and anything else to hold it together (including the sweat and tears of the invested Elevation Team).

## Performance Review:

Tall Boi can track lines and intersections with great accuracy (if batteries are propery fueled), sense walls on three sides
for maze mapping, detect a 660Hz audio start signal, detect other robots' IR signals, and detect treasure color and location.
In terms of software, it uses wall sensor information in a search algorithm for navigation the maze,
performs fast Fourier transform algorithms to detect audio and IR signal frequencies, transmits maze data to a base station over radio frequency communication, and adapts to detected robots in its vicinity. 

After building the robot throughout the semester and having it compete in the [competition](../finalCompetition.md), some of the main flaws in the design became clear, some more significant than others:
- The audio start signal was slightly too sensitive, causing the robot to have multiple false starts in the competition environment. This could easily have been fixed in software by setting a higher threshold for the input signal from the microphone circuit to trigger a detection.
- Our robot suffered from movement problems in the competition environment. This included violent shaking on rough or dirty surfaces as well as getting stuck on elevated surfaces. This was because the lower chassis on our robot was positioned very close to the ground, causing the robot to either continuoulsy run it's front line sensors into the ground or get stuck with it's wheels above the ground. Since we were mainly focusing on getting all of the required features onto the robot by the end of the semester, we did not give ourselves a lot of time for signficant hardware changes. Our robot may have benefitted from an effort to create custom hardware through 3D printing or PCB milling to make a more stable and compact robot. 
- While our robot was mostly good at detection intersections, it missed them enough that the robot could never get through a full 9x9 maze accurately. This could be due to a few issues. Our main line sensors were off-center, which meant that our robot left intersectios at an angle. If the robot is off-center and trying to correct its position near an intersection, the robot has a good chance of missing the intersection and possibly making an accidental turn. Another cause of intersection misses may have been from software timing. The robot runs FFT algorithms and reads sensor inputs while moving between intersections, and running a complex algorithm like FFT while on an intersection may have contributed to some misses. Overall, we may have benefitted from more rigourosly testing our robot's intersection detection abilities and making either software or hardware changes to improve accuracy. 
- While we do not complelety consider this a flaw, our robot was not prepared to detect most of the treasures placed in the competition mazes. Our robot was designed to detect tresures on right-hand walls after going foreward, when the robot was aligned with the wall, but competition treasures were mostly placed at dead ends. Detecting a treasure at the end of an alley meant that our robot would have to turn 90 degrees to detect it, which it could not do accurately because of offset line sensors mentioned above. If we had known that treasures would only be placed on corners or alleys only, we could have put the camera on the front of the robot and had more success during the competition.

[Return Home](https://ece3400-team14.github.io/Team-14-Website/)
