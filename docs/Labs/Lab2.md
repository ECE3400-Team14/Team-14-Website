#  Lab 2: Analog Circuitry and FFT’s

**Purpose:** The goal of this lab was to test and implement the audio and IR signal detection systems to be added onto our robot at a later date. This involved preparing both the analog and digital components necessary for our robot to perform signal analysis. Our first task was to capture an audio signal via an amplified microphone circuit and detect a 660 Hz signal. Our second task was to read a 6.08 kHz IR signal using a photo-diode circuit while ignoring an 18 kHz IR signal. Analysis of these signals were done on an Arduino using Fast Fourier Transform (FFT) analysis through the [Open Music Labs FFT Library](http://wiki.openmusiclabs.com/wiki/ArduinoFFT). 


**Team Assignment:** David and Greg worked on the audio-detection task while Andrew and Michael worked on IR detection. Both groups worked on constructing the combined audio-IR analysis system.

## Audio Team:

2 points: Correct FFT analysis for audio

2 points: Working amplifier (or active filter) circuit for audio

2 points: Distinguish a 660Hz tone from background noise (talking/music)

## IR Team:

2 points: Correct FFT analysis for IR

2 points: Working amplifier (or active filter) circuit for IR

2 points: Distinguish a 6.08kHz IR signal from an IR signal at 18kHz

## Integrated System:

3 points: Demonstration of a single system with nicely merged code that can do both IR and audio
