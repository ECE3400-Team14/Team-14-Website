# Lab 4: FPGA and Camera

### Purpose:

The goal of this lab was to set up the Camera-FPGA system our robot will use to detect treasures during the competition. This involved configuring the settings our [OV7670 camera](http://web.mit.edu/6.111/www/f2016/tools/OV7670_2006.pdf) camera, storing the camera output data in our [DE0-Nano FPGA](http://www.ti.com/lit/ug/tidu737/tidu737.pdf), and using the FPGA to communicate the color of the camera image to our Arduino. One team, Greg and Michael, used our Arduino to both control the camera via I2C communication and set up a communication protocol for receiving color and shape data about the camera image from the FPGA. The other team, Andrew and David, worked on setting up the FPGA to provide a clock signal to the camera, receive, down-sample, and store the camera output in memory, and process the image in memory to determine the image color. Both teams worked on integrating the two systems together to accurately transmit images from the camera to the FPGA and transmit the color of the image from the FPGA to the Arduino. 

## Team Arduino
### Configuring the Camera

In order to get the camera to send the correct data, we first searched through the datasheet for the OV7670 camera to find the settings we needed. Consulting the lab description we found registers and values for the following settings:

Reset all registers --> COM7 is register 0x12 in hex, and setting it to 0x80 resets all registers on the camera<br/>
Enable scaling --> COM14 at 0x3E set to 0x08<br/>
Use external clock as internal clock --> register 0x11 set to 0xC0<br/>
Pixel and resolution format --> register 0x12 set to 0xC<br/>
Set the gain ceiling to something stable --> register 0x14 set to 0x01<br/>
Set pixel format --> register 0x40 to 0xD0<br/>
Enable a color bar test --> register 0x42 set to 0x8<br/>
Vertical and mirror flip --> register 0x1E set to 0x30<br/>
Other parameters --> We set the camera to RGB444 since 565 wasn't working (register 0x8C set to 0x2)<br/>

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

SDA and SCL from the camera were hooked up to A4 and A5 of the Arduino, respectively. It looked something like this:

and tried to write the registers of the camera, which seemed to come out accurately:
![image](https://user-images.githubusercontent.com/16722348/48296667-af620080-e467-11e8-8426-2aa60d31fb95.png)


It took some significant debugging to make sure that these register values were correct, since communication to the FPGA was wrong and hard to diagnose. After changing many of them around, we settled on the values listed above.

### Setting up Arduino-FPGA Communication
For this lab, we decided to keep it simple and do a straightforward proof-of-concept for the FPGA-->Arduino communication scheme. The FPGA sends high or low signals to a few of its pins based on color and shape, and the Arduino reads individually over those pins to figure out if the camera is seeing a certain shape or color.

```cpp
if (isBlue) {digitalWrite(blueLED, HIGH);}
else {digitalWrite(blueLED, LOW);}
if (isRed) {digitalWrite(redLED, HIGH);}
else {digitalWrite(redLED, LOW);}
if (isTriangle) {digitalWrite(triangleLED, HIGH);}
else {digitalWrite(triangleLED, LOW);}
if (isSquare) {digitalWrite(squareLED, HIGH);}
else {digitalWrite(squareLED, LOW);}
```

The Arduino lights up LEDs based on these read values.

For the actual robot, this commnunication scheme will be done over a [spi or i2c] data scheme so that the Arduino need only read information of a certain size over a given input pin.

## Team FPGA

### Setting Up PLL

To set up our PLL, we followed the instructions given to us exactly. What we ended up with are three clock signals with frequencies 24 MHz, 25 MHz, and 50 MHz. We then probed all three signals using an oscilloscope to confirm that we have the correct frequencies.

### Reading and Writing memory

### The Control Unit and Downsampler

We set up our 'CONTROL_UNIT' Module to read in a stream of pixel data, down-sample the data to and 8-bit format, and store it in memory. 
The module definition is shown below:

```verilog
module CONTROL_UNIT (
  CLK,//the PCLK output from the camera, set using the 24 MHz clock (c0_sig) from the PLL. 
  HREF,//Input HREF from the camera to indicate when row data is being sent
  VSYNC,//Indicates frame reset
  input_data,//8-bit input from the camera[D7-D0]
  output_data,//8-bit output to be sent to memory
  X_ADDR,//pixel x-address where [output_data] should be stored in memory
  Y_ADDR,//pixel y-address where [output_data] should be stored in memory
  w_en//output that enables the M9K block to write [output_data] to the address indicated by [X_Addr] and [Y_Addr]
);
```

Because the camera sends 16 bits of data per pixel (when using RGB565, RGB555, or RGB444, we needed a way to read pixel data over two clock cycles to be down-sampled to 8-bits for storage in memory. Our module alternates between writing data to the 8-bit register `part1` and `part2`, then combines these parts into the 16-bit value `{part1,part2}` to be downsampled and written to memory after both parts have been sent by the camera. 

```verilog
always @ (posedge CLK) begin
  if (HREF)//row data is sent only when HREF is high
  begin
    if (write == 0)//write data to part1
    begin
      part1 <= input_data;
      write <= 1; 
      w_en <= 0;
      X_ADDR <= X_ADDR ;
    end
    else//write data to part2 and store output to memory
    begin
      part2 <= input_data;
      write <= 0;
      w_en <= 1;//enable writing to memory
      X_ADDR <= X_ADDR+1;//update pixel x address
    end
  end
  else//no row data is being sent
  begin
     w_en <= 0;
     write <= 0;
     X_ADDR <= 0;
  end
end

```

We use signals from the camera's `HREF` and `VSYNC` output to determine when to read data from each pixel to store in memory. As shown above, `HREF` is high when row data is being sent, and `HREF` goes low between rows. Thus, we use the negative edge of `HREF` to update the y address `Y_ADDR` of the pixel being read. `VSYNC` goes high to indicate the start of a new frame, so we use the positive edge of `VSYNC` to reset `Y_ADDR`.

<img src="https://user-images.githubusercontent.com/12742304/48309517-004a2580-e54a-11e8-8052-1d2a17e8c3f4.png" width="800" />


```verilog
always @ (posedge VSYNC, negedge HREF) begin
  if (VSYNC) begin
    Y_ADDR <= 0;
  end
  else begin
    Y_ADDR <= Y_ADDR + 1;
  end
end
```
To convert the 16-bit camera data to 8-bits to store the data in memory, we put the data read from the camera, `{part1, part2}` through a downsampler. This downsampler took the most significant bits from each color to construct an 8-bit RGB332 value (3-bits red, 3-bits green, 3-bits blue). 

### OV7670 Color Formats:
<img src="https://user-images.githubusercontent.com/12742304/48309510-e9a3ce80-e549-11e8-94ff-89ef7d3ae721.png" width="800" />

### Our Initial Downsampler:
<img src="https://user-images.githubusercontent.com/12742304/48309525-10620500-e54a-11e8-997e-6b13c83c5f98.png" width="800" />


We started off by writing some test images to memory. We did this by writing sample data from our [Simulator]() to our `CONTROL_UNIT` module to write to each pixel of the 176 x 144 image. Connecting our FPGA to the computer screen via our VGA adaptor, we were able to see the shapes we created, trying out various options:

<img src="https://user-images.githubusercontent.com/12742304/48309614-4b653800-e54c-11e8-9d73-195a590ea2f0.jpg" width="250" />

<img src="https://user-images.githubusercontent.com/12742304/48309590-f1647280-e54b-11e8-84d3-f3e2b2c911a2.jpg" width="250" />

<img src="https://user-images.githubusercontent.com/12742304/48309608-3092c380-e54c-11e8-9479-c64f8968aadb.jpg" width="250" />



### Sampling From the Camera:

#### Arduino-Camera-FPGA Setup:
<img width="500" alt="image" src="https://user-images.githubusercontent.com/12742304/48309624-5f109e80-e54c-11e8-98f9-269bc3d439e9.jpg" />

For a frustratingly long time, we attempted to read in RGB565 data from the camera. We were able to get an image from the camera, but the colors were all jumbled. Not good if your trying to detect certain colors! We thought that this was initially due to the camera sending us the wrong color format, but we found no way on the camera to correct the error. When we tried RGB444, however, we began to see an image with more correct color output. We noticed that the expected order of the bits was swapped (giving us GB, Rx, rather than xR,GB as expected), but this was easily corrected by switching `part1` and `part2` in the control unit. 

Using the camera color bar test, we noticed that most of the colors were close to accurate except the last two:

#### Reference Color Bar
<img src="https://user-images.githubusercontent.com/12742304/48309629-73ed3200-e54c-11e8-8524-0572750f21c2.jpg" width="250" />

### Actual Color Bar:
![img_4446](https://user-images.githubusercontent.com/12742304/48309671-16a5b080-e54d-11e8-9bc1-fcc779138d31.jpg)

Notably, the second-to-last color was orange instead of dark red, and the last color bar was green when it should be black. What the color bar test suggested is that we were receiving excess amounts of green and red in our image. Viewing the camera feed confirmed this suspician, as the entire image was saturated with green. We found that specifically the second-most signifcant green bit (G2) seemed to trigger much more often than it should. Therefore, we removed it from the downsampler. After doing this, we still noticed a lot of red in the image, so we removed the second-most significant bit of red (R1) from the downsampler. The resulting image was dark, but we did start to see colors correctly. 

<img src="https://user-images.githubusercontent.com/12742304/48309540-73ec3280-e54a-11e8-8ac4-1e5eed06d4d4.png" width="1213" />

#### Red Saturation:
![img_4453](https://user-images.githubusercontent.com/12742304/48309637-86676b80-e54c-11e8-83c6-ab8f8885f6f8.jpg)

#### With Modified Downsampler:
![img_4456](https://user-images.githubusercontent.com/12742304/48309641-92ebc400-e54c-11e8-9654-4c7829c1fc8e.jpg)

With the resulting solution, we were able to easily distinguish red on a white background, and somewhat distinguish blue. The blue tresure must be directly illuminated in order to be visible on the camera feed, suggesting that the current setup of the camera might not be sensitive enough to blue. 

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

[code]

### Demonstration of Color Detection from Camera Feed:
[video]


