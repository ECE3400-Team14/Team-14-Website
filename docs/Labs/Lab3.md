# Lab 3: System Integration and Radio Communication 

### Purpose: 
The goal of this lab was to integrate the components from previous labs and milestones into our robot and use radio communication between Arduinos to update the [provided GUI](https://github.com/backhous/ece3400-maze-gui) with accurate maze data from the robot. David and Michael worked on integrating robot components, namely the 660 Hz start signal, while Greg and Andrew worked on setting up radio communication and protocols for updating the GUI. All team members worked on combining radio communication with robot right-hand-wall following and robot detection.

## System Integration:

We completed and demonstrated the integration of right-hand wall following and robot detection in [milestone 2](https://ece3400-team14.github.io/Team-14-Website/Milestones/Milestone2.html). In this lab, we integrated the 660 Hz audio start signal into our system from milestone 2.

### Hardware:

We decided to use our combined audio-IR signal circuit from [Lab2](https://ece3400-team14.github.io/Team-14-Website/Labs/Lab2.html): an amplified microphone circuit and phototransistor circuit routed into an analog mux so that both circuits can share an analog port on the Arduino. With our phototransistor circuit already mounted on the robot, we integrate the remaining elements of the circuit onto a perfboard to be mounted on the back side of our robot. This perfboard included our microphone circuit, our microphone and single-rail inverting amplifier from Lab2, and the mux.  

#### Audio-IR Circuit Diagram:
![combinedcircuit2](https://user-images.githubusercontent.com/12742304/46565779-791de800-c8e1-11e8-95dd-8c57641d9373.png)

#### Picture of Mounted Circuit:
<img src="https://user-images.githubusercontent.com/12742304/47275458-aa531500-d57d-11e8-8240-7aa98b1976b9.jpeg" width="600" />


#### Software:

Our software system for start signal detection on the robot performs similarly to the one we used in [Lab2](https://ece3400-team14.github.io/Team-14-Website/Labs/Lab2.html): we run FFT analysis on a free-running ADC input from the microphone until a valid start signal is detected, then we switch the mux input to the IR circuit to perform FFT analysis on the IR signal to detect other robots while our robot is moving. To do this, we loop our `fft_analyze()` function in `setup()`, which continuously runs FFT free-running analysis on the input to pin `A0` until the start signal is detected, setting `has_started` to true:

```cpp
  analogRead(1);//initialize analog (explanation below)
  digitalWrite(fft_mux_pin, LOW);//select audio circuit mux input
  
  //loop FFT analysis on audio signal until positive 660 Hz detection
  while(has_started == false) {
    fft_analyze();
  }
  digitalWrite(fft_mux_pin, HIGH);//After start signal is sent, select IR signal to A0
```
Due to issues with initializing our analog port for FFT analysis, our program occasionally crashed when using analog port `A0` in free-running mode while using `analogRead()` in other parts of the code. We came up with some solutions in [milestone2](https://ece3400-team14.github.io/Team-14-Website/Milestones/Milestone2.html), but we also needed to make sure we ran `analogRead()` before `fft_analyze()` for the free-running ADC settings to enable properly.

One challenged that we faced was making sure that the conditions for a start signal were sufficient that normal speech nearby the robot would cause it to start moving. We distinguish a valid start audio signal from other audio by checking the FFT bin closest to 660 Hz against a minimum threshold. If the amplitude of the frequency signal in this bin remains above the threshold for a certain number of consecutive cycles, we then say the start signal is valid. 

```cpp
  if(!has_started) {
      //Check bin 9 (Bin = Frequency/SampleRate*[# of bins]: 
      // (660 Hz) / (16 MHz / 13 / 128) * 128 ~ 9)
      if(fft_log_out[9] > threshold) {
        start_count++;
        if(start_count == 5) {//when 5 concurrent detections have occurred
          has_started = true;//start the Robot and IR analysis
          fft_detect = true;//indicate valid fft detection
        }
        else  fft_detect = false;//indicate no fft detection
   }
```

Complete FFT code showing this algorithm can be found [here](https://github.com/ECE3400-Team14/3400/blob/master/Lab3_Combined/Signal_Processing.ino)

### Combining Audio and IR Detection with Wall Following:

We tested that our combined Audio-IR FFT detection code worked along with our line following and wall following code. The following video demonstrates that our robot is able to start on an audio signal and switch over to IR detection while moving:

<iframe width="853" height="480" src="https://www.youtube.com/embed/TWhD7SeIBSQ" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

&nbsp

### Data Storage:

Since our Arduino has a very limited memory and the maze has size of 9 by 9, we had to use an efficient way to store our data, otherwise we can possibly exceed our memory limit. If we use int to store every piece of data, which includes x/y coordinates, whether there are walls on east, south, west, and north directions, whether we have a treasure at the intersection and what color it is, that would use at least 8 \* 81 = 648 ints, which has a size of 648 \* 2 = 1296 bytes, if each int is represented by 2 bytes. This is clearly unacceptable.

Therefore, we utilize every bit of an int by bit-masking an int to store information more efficiently and compactly. As we found out, we could fit the data of an intersection completely into an int, which means we only need an int array of size 81 to store the entire maze, which only has a size of 81 \* 2 = 162 bytes, an astonishing improvement from the plan mentioned above. Details of the encoding and parts of the code are shown below.

![format](https://user-images.githubusercontent.com/42748229/47276135-1e43ec00-d583-11e8-921a-59b541f5e384.png)

```cpp
const int rowLength = 2;
const int colLength = 3;
unsigned int mazeData[rowLength * colLength ];//index = x + rowLength*y

void initMaze(){
  for (int xn = 0; xn < rowLength; xn++){
    for (int yn = 0; yn < colLength; yn++){
      int coordinates = 0;
      coordinates = coordinates | (xn | (yn << 4));
      mazeData[xn+rowLength*yn] = coordinates; 
    }
  }
}
void setNorthWall(int x, int y, int valid){
  if(valid) bitSet(mazeData[x+rowLength*y], 8);
  else bitClear(mazeData[x+rowLength*y], 8);
}
...
```

### Radio:

#### Verification of Radio Function:

First we followed the Getting Started sketches to get the radios communicating with each other. After double checking that the radios were hooked up to 3.3V, we ran the sketches using the correct “pipes” numbers. For our lab (Wednesday, Team 14) we used 0x28 and 0x29 for each of the radios. 

Running the base code given to us, we got good transmission between the radios and had good send/receive messages from all the way down a hallway (at least 30 feet with no blocking) which will be plenty for competition day. 

We were now ready to set up the data structure for getting information from the robot GUI to the base station GUI, since all other parts of the pipeline were verified. [See GUI section below]

#### Robot Information Gathering:

At each intersection, our robot detects the walls around that intersection and stores that information into mazeData. That information will be incomplete for the first time we visit that intersection, since right now we are only collecting information about our front wall and right wall using our front wall sensor and right wall sensor. The back wall, which is where the robot comes from, is always false without doubt. We update our mazeData by calling the function updateMaze(). 

```cpp
void updateMaze(){
  int hasFrontWall = readForwardWallSensor();
  int hasRightWall = readRightWallSensor();
  if (orientation == 0){
    setNorthWall(x, y, hasFrontWall);
    setEastWall(x, y, hasRightWall);
  }else if (orientation == 1){
  ...
}
```

The source code for `updateMaze()` can be found [here](https://github.com/ECE3400-Team14/3400/blob/master/Lab3_Combined/Lab3_Combined.ino).

Then, the robot Arduino sends the int that contains all the information of this specific information through our RF transceiver. We modified the example code given to us slightly to do that.

```cpp
...
void sendMaze(){
  // First, stop listening so we can talk. 
  radio.stopListening();
  // Take the time, and send it. This will block until complete 
  bool ok = radio.write( mazeData+x+rowLength*y, sizeof(unsigned int) );
  if (ok) Serial.println("ok...");
  else Serial.print("failed.\n\r");
  ...
}
...
```
The source code for `sendMaze()` can be found [here](https://github.com/ECE3400-Team14/3400/blob/master/Lab3_Combined/Radio.ino).

### GUI:

After installing the GUI, we tested it using 2x3.ino provided in the arduino folder. This program has nothing to do with radio, as it simply prints information to the serial port.

```cpp
...
void loop(){
  Serial.println("reset");
  delay(1000);
  Serial.println("0,0,north=true,west=true");
  ...
}
...
```

After verifying that the GUI opens and refreshes properly on Chrome, we started testing our implementation of encoding and decoding data sent through RF. On the robot Arduino, we manually created an int with value 0b 0000 0101 1001 0100. According to the standard we developed, that means at x = 4 and y = 9, there is a south wall and a north wall. We transmitted that message from the robot Arduino to the base station Arduino, and decoded the message using bitwise AND and shifting. We then print that message to serial to confirm the result. It worked just as we expected.

```cpp
...
int x = data & 0xf;
int y = (data & 0xf0) >> 4;
int n = (data & (1<<8)) >> 8;
int e = (data & (1<<9)) >> 9;
int s = (data & (1<<10)) >> 10;
...
printf("%d",x);
printf(",%d", y);
if (n) Serial.print(",north=true");
if (e) Serial.print(",east=true"); 
...
Serial.print("\n");
```

### Robot-to-GUI integration, full exploration and update on the GUI:

Once each of these pieces was verified, we put together the radio transmission code with our right-wall following algorithm, complete with audio start and IR robot detection into a single working sketch. We also added a few extra lines for the receiver so that if our wall detection is a false positive, which happens way more often than false negatives as we only saw false negatives happened once or twice, it can correct itself later when it crosses that non-existent wall. 

```cpp
...
if (n) Serial.print(",north=true");
else Serial.print(",north=false")
if (e) Serial.print(",east=true"); 
else Serial.print(",east=false")
...
```

We set our robot free in the maze, but the GUI was updating results that we did not anticipate. We were seeing phantom west walls at places where there are no west walls. In fact, there were west walls in every block. After carefully comparing what the serial monitor showed and what the GUI showed, We thought that maybe the GUI wasn’t parsing west=false correctly, so we got rid of that line completely to see what happens. To our surprise, we now saw phantom south walls.It was then clear to us that the GUI wasn’t parsing the last argument correctly. If the last parameter is false, the GUI thinks that it is true. We then solved the issue by printing arguments that are false before printing arguments that are true.


```cpp
...
if (n == 0) Serial.print(",north=false")
if (e == 0)Serial.print(",east=false")
...
if (n) Serial.print(",north=true");
if (e) Serial.print(",east=true"); 
...
```

The GUI then works perfectly fine. When we performed our final test, the wheels are a bit sluggish at times (an unknown outstanding error, split of opinion on the cause) but with a bit of encouragement in the form of a gentle nudge, our robot performed quite well:

<iframe width="853" height="480" src="https://www.youtube.com/embed/3K9Ro9Qo02I" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

Maze data was accurate and the start signal was consistent as ever.

Our Lab 3 source code for the robot can be found [here](https://github.com/ECE3400-Team14/3400/tree/master/Lab3_Combined) and our full base station code can be found [here](https://github.com/ECE3400-Team14/3400/tree/master/Lab3_Base_Station/Base_Station_to_GUI)
