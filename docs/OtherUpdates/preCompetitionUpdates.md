# Pre-Competition Updates

## Integrating shape detection onto the robot:

After finishing [milestone 4](../Milestones/Milestone4.md), we wanted to focus our efforts on getting shape detection onto our robot. This involved constructing an entirely new level of our robot to contain the [DE0-Nano FPGA](http://www.ti.com/lit/ug/tidu737/tidu737.pdf), battery, and camera circuitry so we could mount the [OV-7670 camera](http://web.mit.edu/6.111/www/f2016/tools/OV7670_2006.pdf) on the robot. We though it best to put the camera/FPGA on its own power source to avoid interference with other working systems on the robot. In order to pick up treasures on walls of the maze, we mounted our camera on the right side of the robot, in line with our intersection-detection line sensors. We printed a custom camera mount, but had to adjust it and mount it with hot glue to get the camera at the right angle. We also realized our treasure detection was inconsistent under different lighting conditions, so we added an LED above the camera to slightly improve lighting conditions on treasures in front of the camera.

<img width="646" alt="robot" src="https://user-images.githubusercontent.com/12742304/49557189-d845ac00-f8d4-11e8-818d-6e9d9c53fe43.jpeg">

## Videos Demonstrating Treasure Detection
Our robot was quite accurate at distinguising the color of treasures, and the numbers of missed treasures and false positives were small. Shape detection worked occasionally but was mostly inaccurate. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/HzJs7IEtQ9A" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

&nbsp;

<iframe width="560" height="315" src="https://www.youtube.com/embed/Vtmck79LdVs" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

&nbsp;

<iframe width="560" height="315" src="https://www.youtube.com/embed/Cj8_2oRjutA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

&nbsp;

<iframe width="560" height="315" src="https://www.youtube.com/embed/A5Kt-bS-FXg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Robot Avoidance Updates:

We wanted to update and test our IR detection and robot avoidance algorithm before the competition. We had our robot turn aroun 180 degrees when it reads a high enough IR signal at 6000 Hz, and it returns to the previous intersection it was at. The robot then finds the path to the next nearest unexplored square, avoiding the square where it just detected a robot. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/qCYuCaHZuYs" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

&nbsp;

[return home](../index.md)
