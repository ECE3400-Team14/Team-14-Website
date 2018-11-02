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
