# Milestone 4: Treasure Detection

### Purpose: 

The goal of this milestone was to integrate treasure detection onto our robot. This required our robot to use the [OV7670 Camera](http://web.mit.edu/6.111/www/f2016/tools/OV7670_2006.pdf) to detect red or blue triangles, squares, and diamonds on the walls of the maze. David and Andrew worked on improving our color detection algorithm from [Lab4](../Labs/Lab4.md) to detect shapes and implemented the serial communication protocol to transfer shape information from the FPGA to the robot's Arduino. Gregory and Michael worked on integrating the camera and FPGA circuitry onto the robot. 

## The Shape Detection Algorithm

In order to detect shapes, we continue implementing our image processor. In [Lab4](../Labs/Lab4.md), we finished the color detection part where the module simply counts the number of red pixels and blue pixels in the frame. We now do something more complicated to recognize shape.

The method we chose to do is what I would consider a shortcut. Instead of using every pixel in the picture, we just count the number of "colored" pixels from three rows, i.e., if the entire picture has a majority of red pixels, we count the number of red pixels for those three rows, and if it has a majority of blue pixels, we count the number of blue pixels. At first, we tried to determine the y-coordinates of those rows on-the-fly. Basically, we found the `top` and the `bottom`

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

## Hardware: Adding FPGA and Camera to the Robot

## Combined System
