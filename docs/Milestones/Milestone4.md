# Milestone 4: Treasure Detection

### Purpose: 

The goal of this milestone was to integrate treasure detection onto our robot. This required our robot to use the [OV7670 Camera](http://web.mit.edu/6.111/www/f2016/tools/OV7670_2006.pdf) to detect red or blue triangles, squares, and diamonds on the walls of the maze. David and Andrew worked on improving our color detection algorithm from [Lab4](../Labs/Lab4.md) to detect shapes and implemented the serial communication protocol to transfer shape information from the FPGA to the robot's Arduino. Gregory and Michael worked on integrating the camera and FPGA circuitry onto the robot. 

## The Shape Detection Algorithm

In order to detect shapes, we continue implementing our image processor. In [Lab4](../Labs/Lab4.md), we finished the color detection part where the module simply counts the number of red pixels and blue pixels in the frame. We now do something more complicated to recognize shape.

The method we chose to do is what I would consider a shortcut. Instead of using every pixel in the picture and evaluating the entire frame as a whole, we just count the number of "colored" pixels from three rows, i.e., if the entire picture has a majority of red pixels, we count the number of red pixels for those three rows, and if it has a majority of blue pixels, we count the number of blue pixels. At first, we tried to determine the y-coordinates of those rows dynamically on-the-fly. Basically, we would find the `top` and the `bottom` rows of the shape and find three euqually spaced rows within that range. Unfortunately, calculating that was somehow problematic. After estimating the distance between the camera and the treasures, we figured out that the treasure would most likely fill the entire frame and we wouldn't have to determine those three rows dynamically. Therefore, we decided to have three fixed rows instead, namely 48,72,96.

```verilog
```

After counting the number of "colored" pixels for those rows, we store the results and start determining the shape of the treasure. For squares, the three rows would be approximately the same. For diamonds, the second row would be greater than both the first and the third row. For triangles, the third row would be greater than the second row, which would be greater than the first row. Depending on the lighting, our shape detection system is sometimes very accurate.

[insert video]

## Arduino-FPGA Communication

Rather than using a standard serial communication method, such as I2C or UART, we decided to create our own serial communication protocol to communicate shape data from the FPGA and Arduino. This decision was motiviated by a few reasons:
- Many serial protocols requred either too many pins on our Arduino (SPI, for example). The one we created used open pins on our Arduino (one digital pin and one analog pin). 
- Our communication did not require two-way communication: The Arduino is only reading data and the FPGA was only sending data.

### Protocol Overview:
Our serial protocol requires two wires:
- **SIGNAL**: Driven by the Arduino. Driving this wire high requests the next bit from the FPGA.
- **DATA**: Driven by the FPGA. Carries the data stream containing the shape/color data of the image. 

The picture below illustrates a typical transmission. Each packet consists of 16 bits. The first 6 bits are five zeros followed by a one. The purpose of this pilot sequence is time synchronization as five consecutive zeros would never occur. Data is a agreed-upon 4-bit field where 2 bits are dedicated to shape detection and 2 bits to color detection. Data is transmitted twice for error detection. If a transmission error occurs, the Arduino will request another transmission.

![packet](https://user-images.githubusercontent.com/42748229/49323587-b66eb280-f4eb-11e8-9f1a-5c5497f4fae2.png)


**SIGNAL** is driven at around 500Hz, slow compared to other communication protocols but fast enough for our needs (this is subject to change for the competition). That means transmission of a single data packet takes about 0.03s.

Video of isolated camera detection (not on the robot):
<iframe width="901" height="507" src="https://www.youtube.com/embed/z2tLbDWV484" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Hardware: Adding FPGA and Camera to the Robot

We have been having some issues with weight distribution on the robot, and "rocking" forwards and backwards, so we will put the FPGA closer to the rear of the bot to mitigate this issue.

The camera wiring was switched from a breadboard to a protoboard to allow the camera to be attached to the robot in a more custom location. Since the treasures are along walls, and since we rotate at intersections with front walls, we will orient the camera on the side of the robot in order to maximize detection. We move one of our wall sensors backwards to make room before attaching the camera to the underside of our FPGA mount plate on the top of the robot.

The protoboarded mount:
![20181130_183412](https://user-images.githubusercontent.com/16722348/49323505-8d99ed80-f4ea-11e8-938c-679cd398ed61.jpg)
![20181130_183416](https://user-images.githubusercontent.com/16722348/49323506-8d99ed80-f4ea-11e8-8918-41245e0e59ed.jpg)

The wiring:
![circuit](https://user-images.githubusercontent.com/16722348/49323555-0f8a1680-f4eb-11e8-92e3-4cedf973f0b5.png)

## Combined System

[TODO?]

