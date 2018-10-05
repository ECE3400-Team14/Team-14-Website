#  Lab 2: Analog Circuitry and FFT’s

**Purpose:**

**Team Assignment:** David and Greg worked on audio while Andrew and Michael worked on IR.

## Audio Team:

2 points: Correct FFT analysis for audio

2 points: Working amplifier (or active filter) circuit for audio

2 points: Distinguish a 660Hz tone from background noise (talking/music)

## IR Team:

For our IR circuit, we started with the circuit that only consists of a phototransistor and a 1.8kΩ resistor, as provided in the lab handout:

![circuit](https://cei-lab.github.io/ece3400-2018/images/lab2_phototransistor_schem.png)

And it looks like this:

![circuit](https://user-images.githubusercontent.com/42748229/46548398-fa06c080-c89c-11e8-80ef-c81fa1885d85.png)

We started testing this circuit to determine if it needs an amplifier or an active filter. First, we connect this circuit to the oscilloscope to see if the circuit actually works, and it worked like a charm. Here are the waveforms detected by the oscilloscope from the IR hat that emits a 6kHz IR signal and the decoy that emits a 18kHz IR signal, respectively. 

![6kHz](https://user-images.githubusercontent.com/42748229/46548656-b2346900-c89d-11e8-9019-e88dd62f9795.jpeg)
![18kHz](https://user-images.githubusercontent.com/42748229/46548669-ba8ca400-c89d-11e8-93c9-38204cf12884.jpeg)





2 points: Correct FFT analysis for IR

2 points: Working amplifier (or active filter) circuit for IR

2 points: Distinguish a 6.08kHz IR signal from an IR signal at 18kHz

## Integrated System:

3 points: Demonstration of a single system with nicely merged code that can do both IR and audio
