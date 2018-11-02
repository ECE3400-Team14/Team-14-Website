# Lab 4: FPGA and Camera

### Purpose:

The goal of this lab was to set up the Camera-FPGA system our robot will use to detect treasures during the competition. This involved configuring the settings our [OV7670 camera](https://www.voti.nl/docs/OV7670.pdf) camera, storing the camera output data in our [DE0-Nano FPGA](http://www.ti.com/lit/ug/tidu737/tidu737.pdf), and using the FPGA to communicate the color of the camera image to our Arduino. One team, Greg and Michael, used our Arduino to both control the camera via I2C communication and set up a communication protocol for receiving color and shape data about the camera image from the FPGA. The other team, Andrew and David, worked on setting up the FPGA to provide a clock signal to the camera, receive, down-sample, and store the camera output in memory, and process the image in memeory to determine the image color. Both teams worked on integrating the two systems together to accurately transmit images from the camera to the FPGA and transmit the color of the image from the FPGA to the Arduino. 

## Team Arduino
### Configuring the Camera
2 points: Arduino-Camera communication (writing the correct registers)

### Setting up Arduino-FPGA Communication
3 points: Arduino-FPGA communication (communicating treasure/no treasure + shape and color)

## Team FPGA

### Setting Up PLL

### Down-sampler and Color Detection

### Camera Simulation
2 points: Displaying the contents of an M9K block on the screen

## Integrating the Camera, Arduino, and FPGA
3 points: Camera-FPGA communication (downsampling and storing in the M9K buffer)
2 points: Displaying the test image from the camera on the screen
3 points: Color detection with the FPGA, camera, and Arduino
