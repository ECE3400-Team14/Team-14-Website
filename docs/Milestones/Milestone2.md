#  Milestone 1

## Overview of Robot Modifications

In this milestone, we aimed to use line tracking (previously developed) to implement a wall-following algorithm 
to navigate an arbitrary set of walls while avoiding other robots. This integrates multiple systems explored in previous labs
into a working system on the robot.

### Robot Maintenance:

In order to have more room on the top of our robot for mounting the new components (wall sensors, IR circuit, and IR hat), we added a second chassis under the wheels of the robot. On this lower chassis, we mounted our line sensors and stored our breadboard circuits while keeping our Arduino on the top level, freeing up space next to the arduino for other components. 

[Picture?]

Additionally, in order to free up analog ports for our wall sensors and IR detection circuit, we routed all of our line sensor outputs to a single analog port through a 4-to-1 mux circuit, with two digital pins to select which line sensor to read from. We did using the first 4 inputs (A0 to A3) on an 8-to-1 MUX, grounding the most significant selector pin. In order to perform accurate readings from each sensor, we had to include a small delay before reading from the analog port after swithcing the MUX input channel selector pins (arount 6 ms). 

![image](https://user-images.githubusercontent.com/42748229/46560456-d43fe280-c8c1-11e8-92ec-740b3bd49977.png)![functiontable](https://user-images.githubusercontent.com/42748229/46560910-63012f00-c8c3-11e8-9337-37a1eb17cdac.png)


#### Example for reading Left Sensor:
```cpp
  //Select input 0 with 00 select input
  digitalWrite(mux0, LOW);
  digitalWrite(mux1, LOW);
  delay(muxReadDelay);// delay for around 6 ms
  int val = analogRead(muxRead);//read the mux output
```

#### Circuit Diagram:

![arduino diagram_m2_8_bb](https://user-images.githubusercontent.com/12742304/46901204-9290d800-ce7d-11e8-908f-b253cb75bee4.png)

### Adding Wall Sensors and IR circuit:

We decided to mount two wall sensors onto our robot: one facing forward and one facing to the right. We decided that these two wall sensors would be effective enough to implement right-hand wall following. Additionally, we added LEDs corresponding to each wall sensor to indicate when a wall sensor detects a wall. (see section __Wall Follow Algorithm__ for more details). 
 
#### Updated Circuit Diagram:
![arduino diagram_m2_10_bb](https://user-images.githubusercontent.com/12742304/46901306-02539280-ce7f-11e8-9437-b0db9fb561f1.png)

To add our IR circuit to the robot...

### Wall Follow Algorithm:

The basic wall following algorithm is a simple set of rules that hold the robot hugging a right-hand wall. If there is a wall on the right
the robot will go straight. If there is a wall on the right, and a wall in front, the robot will turn left. Finally, if there is no wall on the right,
the robot should turn right in order to find the wall again:

```cpp
void wallFollowAndStop(){
  int hasRightWall = readRightWallSensor();
  int hasFrontWall = readForwardWallSensor();
` if (hasRightWall==1&&hasFrontWall==0) {
    forwardAndStop();
   }
  else if (hasRightWall==1&&hasFrontWall==1){
    turnLeft();
    finishTurn();
   }
  else if (hasRightWall==0){
    turnRight();
    finishTurn();
    forwardAndStop();
   }
}
```

Our robot is tuned such that it will only sense a wall at quite close range. This is so that the robot only has to check for a wall at every intersection.
This is why, in the video below, the LED indicators only change their values at intersections. That is where turn decisions are made.

The green light on the right side of the robot indicates that the robot "sees" a wall on the right side.
The yellow light on the front of the robot indicates that the robot"sees" a wall in front of the robot.

<iframe width="560" height="315" src="https://www.youtube.com/embed/LUa8-9yA-2w" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

Notice that this algorithm can successfully navigate any set of walls until hitting an "edge" of our table.

### Robot Avoidance Algorithm:

During the process of integrating our free-running analog sampling and FFT analysis, we encountered the following issues:

* __Memory Shortage:__ We found that integrating our FFT analysis with 256 samples from [Lab 2](https://ece3400-team14.github.io/Team-14-Website/Labs/Lab2.html) caused our program to use 75% of dynamic memory, highly limiting our heap space. In order to avoid any issues when adding additional code in the future, we reduced the memory used by the FFT library by halving the number of samples we analyzed to 128 (`#define FFT_N 128`), reducing our total dynamic memory usage to 47%. While this reduced the number of frequency bins and the accuracy of the FFT output frequencies, we found that we were still getting accurate detection results from our IR circuit. 

* __Combining Free-Running ADC Reading With Analog Read:__ After our initial failed attempts at combining FFT analysis with our wall-following code, we realized we had to make changes to the FFT setup:
  * We made sure not to turn off `timer0`, as it affects timing used when calling `delay()` in other parts of the program. 
  * To avoid conflicts when running `AnalogRead()` between FFT calls, we made sure to run our FFT setup each time before reading from our free-running port devoted to FFT (`A0`). This included setting `ADMUX` to select `A0` and setting our ADC free-running and prescale settings in `ADCSRA`. We also needed to make sure we restored the previous settings of `ADCSRA` before exiting our FFT analysis function. 
  
Our combined code for Milestone 2 can be found [here](https://github.com/ECE3400-Team14/3400/tree/master/Milestone2). 

The way we ran robot avoidance was once every few cycles anytime the robot is moving between intersections:
```cpp
void forwardAndStop(){
    int i = 0;
    while (readLeftmostSensor() == 1 || readRightmostSensor() == 1){
      //Serial.println("BEGIN");
      trackLine();
      //perform FFT every __ number of cycles
      if (i == 5) {
        Serial.println("Running fft");
        fft_analyze();
        while (fft_detect/**/) {
          stopMovement();
          fft_analyze();
        } //Added
        i = 0;
      }
      else { i++; }
    }
  stopMovement();
}
```
This allowed adequate time for the line sensors to keep the robot on track while also maintaining a decent reaction time to a robot stop signal.

### Both Wall and Robot Avoidance:

Combining the two algorithms was then easy because we integrated avoidance into the movement of the robot. Like an obedient pet, our robot correctly executed wall following while ignoring decoy bots and stopping in the prescence of a real robot signal:

<iframe width="560" height="315" src="https://www.youtube.com/embed/Yrjw4R42oCg" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
