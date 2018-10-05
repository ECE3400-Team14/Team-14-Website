#  Lab 2: Analog Circuitry and FFT’s

**Purpose:** The goal of this lab was to test and implement the audio and IR signal detection systems to be added onto our robot at a later date. This involved preparing both the analog and digital components necessary for our robot to perform signal analysis. Our first task was to capture an audio signal via an amplified microphone circuit and detect a 660 Hz signal. Our second task was to read a 6.08 kHz IR signal using a phototransistor circuit while ignoring an 18 kHz IR signal. Analysis of these signals were done on an Arduino using Fast Fourier Transform (FFT) analysis through the [Open Music Labs FFT Library](http://wiki.openmusiclabs.com/wiki/ArduinoFFT). 

**Team Assignment:** David and Greg worked on the audio-detection task while Andrew and Michael worked on IR detection. Both groups worked on constructing the combined audio-IR analysis system.

## Audio Team:

### Setting Up FFT Analysis for Audio

We started by checking that we could detect a 660 Hz signal from a signal generator using the FFT library. In order to detect the signal, we used the example file `fft_adc_serial`, found in the FFT/examples folder downloaded with the library. We made some small additions to the code:

We changed the prescale factor of the Arduino Analog-to-Digital Converter (ADC) to the highest value possible (128). This is because the audio signal we are trying to detect is at a low frequency, and does not require a high sample rate. The ADC sample rate is the Arduino clock speed (16 MHz) divided by the prescale factor and the number of cycles to process an input (13): 16 MHz / 128 / 13 ~ 9600 Hz sample rate. The maximum frequency that can be samples at 9600 Hz is half the sampling rate, 4800 Hz, more than enough for sampling 660 Hz. 

```cpp
void setup() {
  Serial.begin(9600); // use the serial port
  TIMSK0 = 0; // turn off timer0 for lower jitter
  ADCSRA = 0xe7; //CHANGED THIS LINE (for prescale factor of 128)
  ADMUX = 0x40; // use adc0
  DIDR0 = 0x01; // turn off the digital input for adc0
}
```

At this sampling rate, we needed to determine which bin to find the amplitude of a 660 Hz signal calculated by the FFT. As the FFT converts a signal in the time domain to one in the frequency domain, the output of the FFT is organized into bins corresponding to the signal amplitude at different frequencies. With 127 evenly-spaced frequency bins available and a maximum frequency of 4800 Hz in the last bin, 660 Hz would be in the bin (660/4800)*127 ~ 17. 

Testing this in practice with a 660 Hz signal from a signal generator, we found that the highest-amplitude bin was indeed pin 17:

![signal gen no amp2](https://user-images.githubusercontent.com/12742304/46562847-284fc480-c8cc-11e8-8610-b3972fd84115.png)

We noticed, however, that there were a lot of harmonics present in the signal, despite the input being a sine wave. We expected that this may have been caused by low output impedance of the signal generator when input into the Arduino analog port. Such a problem was solved later when we added an amplifier with higher output impedance. 

### Building the Amplifier:

After constructing the simple microphone circuit from lab and monitoring the output on the oscilloscope, we noticed that the output signal had a low amplitude (around 50 mV) even for loud noise. Therefore, we decided to build an amplifier for our microphone circuit. We got the amplifier to work was setting it up with a single rail and DC biasing our input signal. We initially tried to build a non-inverting amplifier with an op-amp, but we had trouble getting the DC biasing to work at the positive terminal along with the AC signal. We eventually switched over to an inverting amplifier resembling the one constructed by [Team Alpha last year](https://cei-lab.github.io/ECE3400-2017-teamAlpha/lab2.html). Including a capacitor between VCC and ground reduced some noice in the circuit caused by the amplified AC signal. 

#### Image of Circuit: 

![img_4307](https://user-images.githubusercontent.com/12742304/46564783-9a79d680-c8d7-11e8-9ab5-7763a0f102cf.jpg)

#### Circuit Diagram:

![microphoneampcircuit](https://user-images.githubusercontent.com/12742304/46564808-b41b1e00-c8d7-11e8-990c-038a51144e3a.png)

We had to adjust the DC biasing of the positive terminal in order to make sure that a large enough input would not hit the rails of the circuit. We initially assumed that the DC bias needed to be exactly between VCC and ground (2.5V) but we found that our signal was clipping at a lower voltage than expected from above. We tried lowering the voltage divider ratio at the positive terminal (R5/(R4 + R5) so the DC bias was around 1.8 Volts, and this gave us the largest voltage swing:

#### The maximum voltage swing through the amplifier (from signal generator):

![signal gen](https://user-images.githubusercontent.com/12742304/46564901-563b0600-c8d8-11e8-84cc-009f24f3f423.png)

#### A 660 Hz tone played in close proximity to the microphone. 

![microphone signal](https://user-images.githubusercontent.com/12742304/46564881-27bd2b00-c8d8-11e8-8a75-79862429c925.png)

2 points: Distinguish a 660Hz tone from background noise (talking/music)

## IR Team:

For our IR circuit, we started with the circuit that only consists of the phototransistor and a 1.8kΩ resistor, as provided in the lab handout:

<img src="https://cei-lab.github.io/ece3400-2018/images/lab2_phototransistor_schem.png"/>

And it looks like this on the breadboard:

<img src="https://user-images.githubusercontent.com/42748229/46548398-fa06c080-c89c-11e8-80ef-c81fa1885d85.png" width="720" height="540"/>

We started testing this circuit to determine if we actually need an amplifier or an active filter. First, we connect the oscilloscope to see if the circuit works, and it worked like a charm. Here's the waveform detected by the oscilloscope from the IR hat that emits a 6.08kHz IR signal. As we can see, the frequency we pick up is not a perfect 6.08kHz. Instead, it is about 6.47kHz.

<img src="https://user-images.githubusercontent.com/42748229/46548656-b2346900-c89d-11e8-9019-e88dd62f9795.jpeg" width="720" height="540"/>

After seeing the result, we were satisfied with the strength of the signal as this circuit actually acts as an amplifier. The current generated by the light affects the base region and this is amplified by the current gain of the transistor in the normal way. We also decided that the filtering can be done digitally as it would be more effective and clean.

By modifying the sample code given by the FFT library and plotting the output, we can clearly distinguish the robot signal and the decoy signal. But first, we had to make a minor and yet important change to our code. In the sample code, ADCSRA was set up to be 0xe5, where the last last digit corresponds to the prescale factor as shown in the table below.

<img src="https://user-images.githubusercontent.com/42748229/46558523-60024080-c8bb-11e8-8624-8f1513b950d4.png"/>

The theoretical sampling frequency is calculated as follows. The arduino ADC clock runs at 16MHz. If the prescale factor is 64, i.e. ADCSRA set to 0xe6, the clock then runs at 16/64 = 0.25MHz. Each ADC conversion takes 13 cycles, which means the theoretical sampling frequency is 0.25MHz/13 = 19.2kHz. In order to eliminate aliasing for our 18kHz signal, the sampling rate needs to be at least 36kHz, which means a prescale factor of 32 should suffice. However, the decoy signal is not a pure sinusodal wave, which means it has peaks not only at its first harmonic frequency, but also at its overtones. The amplitudes at the overtone frequencies compared to the amplitude at the fundemental frequency is small and yet cannot be ignored. By setting the sampling frequency to 76.8kHz, theoretically only frequencies over 38.4kHz would cause aliasing, i.e. the 3rd harmonic. In addition, only be peaks after the 4th harmonic frequency might accumulate near 6kHz. Since the amplitudes at overones decays linearly, the accumulated result does not matter if we choose our threshold carefully. This way, we most likely will not confuse the 18kHz signal with the 6kHz signal. 

```cpp
ADCSRA = 0xe4; // prescale factor of 16
```
With a sampling frequency of 76.8kHz and 128 bins, the 6kHz signal, which is actually 6.47kHz, should have a peak at bin 21, as 6.47kHz is between 76.8/2/128\*21 = 6.3kHz and 76.8/2/128\*22 = 6.6kHz. Similarly, the 18kHz decoy signal should have a peak near bin 62. As we plot the arrays obtained through FFT, we see exactly that.

FFT from IR Hat:

<img src="https://user-images.githubusercontent.com/42748229/46559383-3565b700-c8be-11e8-998c-e61b1a442d93.png"/>

FFT from Decoy:

<img src="https://user-images.githubusercontent.com/42748229/46559389-3bf42e80-c8be-11e8-90a9-d87d710551df.png"/>

By setting an appropriate threshold for bin 21, we can detect the IR hat from about 50cm away and ignore the decoy. We connected an external LED to indicate such detection.

```cpp
if (fft_log_out[21] > 60) digitalWrite(LED_pin, HIGH);
else digitalWrite(LED_pin, LOW);
```

[insert videos of IR hat and decoy]

## Integrated System:

To have an integrated system where both the audio and IR system share the same FFT analysis system on the Arduino, we added a mux in order to route both signals through a single analog port. The mux is an analog 8-to-1 mux with 3-bit selection inputs, which we used as a 2-1 mux by grounding two of the select pins and selecting between two inputs with one pin. Pin diagram and function table shown below.

![image](https://user-images.githubusercontent.com/42748229/46560456-d43fe280-c8c1-11e8-92ec-740b3bd49977.png)![functiontable](https://user-images.githubusercontent.com/42748229/46560910-63012f00-c8c3-11e8-9337-37a1eb17cdac.png)

Since we only have two signals going into the mux, we decided to ground S1 and S2, and only control S0 so that we select from A1 and A0. Pin 6, which is enable, also needs to be grounded, since it's active-low.

[integrated system circuit diagram?]

When our system first starts running, it only receives signal from the microphone and waits for the 660Hz signal. Once it detects a tone at that frequency, it flips S0 so now it only receives signal from the phototransistor. From that point on, it ignores the decoy signal at about 18kHz and reacts once it detects the IR hat signal from other robots at about 6kHz.

[insert code segmants, video, etc.]
