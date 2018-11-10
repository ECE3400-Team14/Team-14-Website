# Lab 4: FPGA and Camera

### Purpose:

The goal of this lab was to set up the Camera-FPGA system our robot will use to detect treasures during the competition. This involved configuring the settings our [OV7670 camera](https://www.voti.nl/docs/OV7670.pdf) camera, storing the camera output data in our [DE0-Nano FPGA](http://www.ti.com/lit/ug/tidu737/tidu737.pdf), and using the FPGA to communicate the color of the camera image to our Arduino. One team, Greg and Michael, used our Arduino to both control the camera via I2C communication and set up a communication protocol for receiving color and shape data about the camera image from the FPGA. The other team, Andrew and David, worked on setting up the FPGA to provide a clock signal to the camera, receive, down-sample, and store the camera output in memory, and process the image in memory to determine the image color. Both teams worked on integrating the two systems together to accurately transmit images from the camera to the FPGA and transmit the color of the image from the FPGA to the Arduino. 

## Team Arduino
### Configuring the Camera
2 points: Arduino-Camera communication (writing the correct registers)

In order to get the camera to send the correct data, we first searched through the datasheet for the OV7670 camera to find the settings we needed. Consulting the lab description we found registers and values for the following settings:

Reset all registers --> COM7 is register 0x12 in hex, and setting it to 0x80 resets all registers on the camera
Enable scaling --> COM14 at 0x3E set to 0x08
Use external clock as internal clock --> register 0x11 set to 0xC0
Pixel and resolution format --> register 0x12 set to 0xC
Set the gain ceiling to something stable --> register 0x14 set to 0x01
Set pixel format --> register 0x40 to 0xD0
Enable a color bar test --> register 0x42 set to 0x8
Vertical and mirror flip --> register 0x1E set to 0x30
Other parameters --> We set the camera to RGB444 since 565 wasn't working (register 0x8C set to 0x2)

Once we decided the values to start with, we wrote code for the OV7670_SETUP file which first reads and prints the values stored at the register addresses, writes the values that we want, and then reads and prints them again to check for good setup. 

```cpp
  read_key_registers();
  OV7670_write_register(0x12, 0x80);//reset all registers COM7
  OV7670_write_register(0x0C, 0x8); //COM3 enable scaling
  OV7670_write_register(0x11, 0xC0);//use external clock
  OV7670_write_register(0x12, 0xE);//set camera pixel format and enable color bar test with 0xE disable with 0xC
  OV7670_write_register(0x14, 0x01);//automated gain ceiling of 2x
  OV7670_write_register(0x40, 0xD0);//COM15 set for RGB 565 11010000 (208) D0
  OV7670_write_register(0x1E, 0x30); //mirror and flip
  OV7670_write_register(0x8C, 0x2); //RGB444
  OV7670_write_register(0x42, 0x8);//COM17 enable DSP color bar
  read_key_registers();
  set_color_matrix();
```
Then we set up the circuit as shown in the Lab description,
![image](https://user-images.githubusercontent.com/16722348/48296590-8d1bb300-e466-11e8-8ca3-37659c2458d5.png)
(from https://cei-lab.github.io/ece3400-2018/lab4.html)

SIOD and SIOC from the camera were hooked up to A4 and A5 of the Arduino, respectively. It looked something like this:

and tried to write the registers of the camera, which seemed to come out accurately:
![image](https://user-images.githubusercontent.com/16722348/48296667-af620080-e467-11e8-8426-2aa60d31fb95.png)


It took some significant debugging to make sure that these register values were correct, since communication to the FPGA was wrong and hard to diagnose. After changing many of them around, we settled on the values listed above.

### Setting up Arduino-FPGA Communication
For this lab, we decided to keep it simple and do a straightforward proof-of-concept for the FPGA-->Arduino communication scheme. The FPGA sends high or low signals to a few of its pins based on color and shape, and the Arduino reads individually over those pins to figure out if the camera is seeing a certain shape or color.

```cpp
//TODO insert code block for communication scheme
```

The Arduino lights up LEDs based on these read values.

For the actual robot, this commnuication scheme will be done over a [spi or i2c] data scheme so that the Arduino need only read information of a certain size over a given input pin.

## Team FPGA

### Setting Up PLL

### Reading and Writing memory

We started off by writing some test images to memory. We did this by writing our `CONTROL_UNIT` module to iterate through each pixel of the 176 x 144 image and output a color based on the coordinates of the pixel (noted by `X_ADDR` and `Y_ADDR`). Connecting our FPGA to the computer screen via our FPGA adaptor, we were able to see the shapes we created, trying out various options:

[images here]

### Color Detection

Next, we wrote our image processing module to detect the color of the image. Our algorithm was as follows:
* For each pixel in the image, check if the pixel is mostly red or mostly blue. Mostly red means the most significant bit of the red part is 1 and the two most significant bits of the green and blue parts are 0. Mostly blue means the most significant bit of the blue part is 1 and two most significant parts of the red and green parts are 0. 

#### Analyzing Each Pixel in the Image:
```verilog
always @ (posedge CLK) begin
	if (VGA_PIXEL_X == 0 && VGA_PIXEL_Y == 0) begin
		blue <= 0;
		red <= 0;
	end
	else begin
		if (VGA_PIXEL_X < 176 && VGA_PIXEL_Y < 144) begin
			if (PIXEL_IN[7] == 1 && PIXEL_IN[1:0] == 0 && PIXEL_IN[4:3] == 0) begin
				red <= red + 1;
			end
			else begin
				red <= red;
			end
			if (PIXEL_IN[7:6] == 0 && PIXEL_IN[1] == 1 && PIXEL_IN[4:3] == 0) begin
				blue <= blue + 1;
			end
			else begin
				blue <= blue;
			end
		end
	end
end
```
#### Calculating the Total Color of the Image

```verilog
always @ (*) begin
	if (red > 12672) begin
		RESULT <= 8'b00000001;
	end
	else begin
		if (blue > 12672) begin
			RESULT <= 8'b00000010;
		end
		else begin
			RESULT <= 8'b0;
		end
	end
end
```
We connected the output of color detection to the LEDs on the FPGA. The left 4 LEDs light up when the image is red and the 4 right LEDs light up when the image is blue.

##### Detecting Red:
<img src="https://user-images.githubusercontent.com/12742304/47939808-02afde00-debf-11e8-87ac-3e35e1386200.jpg" width="400" />

##### Detecting Blue:
<img src="https://user-images.githubusercontent.com/12742304/47939837-11969080-debf-11e8-80ca-b91544b9a397.jpg" width="400" />

##### Detecting No Color (White):
<img src="https://user-images.githubusercontent.com/12742304/47939863-24a96080-debf-11e8-97ff-fe6eae21846d.jpg" width="400" />

##### Detecting No Color (Purple):
<img src="https://user-images.githubusercontent.com/12742304/47939878-2d019b80-debf-11e8-95f4-0800ee1c90cb.jpg" width="400" />

### Down-sampler and Camera Simulation
2 points: Displaying the contents of an M9K block on the screen

## Integrating the Camera, Arduino, and FPGA
3 points: Camera-FPGA communication (downsampling and storing in the M9K buffer)
2 points: Displaying the test image from the camera on the screen
3 points: Color detection with the FPGA, camera, and Arduino
