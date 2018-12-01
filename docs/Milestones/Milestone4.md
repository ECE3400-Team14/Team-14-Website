# Milestone 4: Treasure Detection

### Purpose: 

The goal of this milestone was to integrate treasure detection onto our robot. This required our robot to use the [OV7670 Camera](http://web.mit.edu/6.111/www/f2016/tools/OV7670_2006.pdf) to detect red or blue triangles, squares, and diamonds on the walls of the maze. David and Andrew worked on improving our color detection algorithm from [Lab4](../Labs/Lab4.md) to detect shapes and implemented the serial communication protocol to transfer shape information from the FPGA to the robot's Arduino. Gregory and Michael worked on integrating the camera and FPGA circuitry onto the robot. 

## The Shape Detection Algorithm

## Arduino-FPGA Communication

Rather than using a standard serial communication method, such as I2C or UART, we decided to create our own serial communication protocol to communicate shape data from the FPGA and Arduino. This decision was motiviated by a few reasons:
- Many serial protocols requred either too many pins on our Arduino (SPI, for example). The one we created used open pins on our Arduino (one digital pin and one analog pin). 
- Our communication protocol did not require two-way communication: The Arduino is only reading data and the FPGA was only writing data.

### Protocol Overview:
Our serial protocol requires two wires:
- **SIGNAL**: Driven by the Arduino. Sending this wire high requests the next bit from the FPGA.
- **DATA**: Driven by the FPGA. Carries the data stream containing the shape/color data of the image. 

[picture?]

**SIGNAL** is driven at around 500Hz, slow compared to other communication protocols but fast enough for our needs (this is subject to change for the competition). 

Video of isolated camera detection (not on the robot):
<iframe width="901" height="507" src="https://www.youtube.com/embed/z2tLbDWV484" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Hardware: Adding FPGA and Camera to the Robot

We have been having some issues with weight distribution on the robot, and "rocking" forwards and backwards, so we will put the FPGA closer to the rear of the bot to mitigate this issue.

The camera wiring was switched from a breadboard to a protoboard to allow the camera to be attached to the robot in a more custom location. Since the treasures are along walls, and since we rotate at intersections with front walls, we will orient the camera on the side of the robot in order to maximize detection. We move one of our wall sensors backwards to make room before attaching the camera to the underside of our FPGA mount plate on the top of the robot.

The protoboarded mount:
![20181130_183412](https://user-images.githubusercontent.com/16722348/49323505-8d99ed80-f4ea-11e8-938c-679cd398ed61.jpg)
![20181130_183416](https://user-images.githubusercontent.com/16722348/49323506-8d99ed80-f4ea-11e8-8918-41245e0e59ed.jpg)

The wiring:

## Combined System

[TODO?]

