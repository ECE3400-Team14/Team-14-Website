# Lab 3: System Integration and Radio Communication 

### Purpose: 

Efficient data scheme to store all maze information on an Arduino

Sending maze information wirelessly between Arduino’s

Updating the GUI from the base station

Updating the GUI from a virtual robot on a separate Arduino which is wirelessly connected to the base station

## System Integration:

We completed and demonstrated the integration of right-hand wall following and robot detection in [milestone 2](https://ece3400-team14.github.io/Team-14-Website/Milestones/Milestone2.html). In this lab, we integrated the 660 Hz audio start signal into our system from milestone 2.

### Hardware:

We decided to use our combined audio-IR signal circuit from [Lab2](https://ece3400-team14.github.io/Team-14-Website/Labs/Lab2.html): an amplified microphone circuit and phototransistor circuit routed into an analog mux so that both circuits can share an analog port on the Arduino. With our phototransistor circuit already mounted on the robot, we integrate the remaning elements of the circuit onto a perfboard to be mounted on the back side of our robot. This perfboard included our microphone circuit, our microphone and single-rail inverting amplifier from Lab2, and the mux.  

#### Audio-IR Circuit Diagram:
![combinedcircuit2](https://user-images.githubusercontent.com/12742304/46565779-791de800-c8e1-11e8-95dd-8c57641d9373.png)

#### Picture of Mounted Circuit:
[TODO: picture of mounted circuit]

#### Software:

Our software system for start signal detection on the robot performs similarly to the one we used in [Lab2](https://ece3400-team14.github.io/Team-14-Website/Labs/Lab2.html): we run FFT analysis on a free-running ADC input from the microphone until a valid start signal is detected, then we switch the mux input to the IR circuit to perform FFT analysis on the IR signal to detect other robots while our robot is moving. To do this, we loop our `fft_analyze()` function in `setup()`, which continously runs FFT free-running analysis on the input to pin `A0` until the start signal is detected, setting `has_started` to true:

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

One challenged that we faced was making sure that the conditions for a start signal were sufficient that normal speech nearby the robot would cause it to start moving. We distinguish a valid start audio signal from other audio by checking the FFT bin closest to 660 Hz against a minimum threshold. If the amplitude of the frequency signal in this bin remains above the threshold for a certain number of consectutive cycles, we then say the start signal is valid. 

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
### Radio Verification

### Robot starting on a 660Hz tone
[TODO: Demo Video]

### Robot starting on a 660Hz tone and exploring the entire maze
[TODO: Demo Video]

### Robot that explores, and stops if it sees another robot, but ignores decoys
[TODO: Demo Video]

### Robot-to-GUI integration, full exploration and update on the GUI
[TODO: Demo Video]
