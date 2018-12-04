# Elevation's Robot

Tall Boi is outfitted with almost entirely default materials provided by ECE 3400, and is very capable with just those pieces! 

[image of Tall Boi]

You can see where our robot, Tall Boi, gets his name! Three tiers of electrical engineering effectiveness brings his stature to quite a height, and he is sure to raise eyebrows on Robotics Day on December 4th, 2018, when he is put to the test against our competitors.


## Capabilities

Tall Boi can track lines and intersections with great accuracy (if batteries are propery fueled), sense walls on three sides
for maze mapping, detect a 660Hz audio start signal, detect other robots' IR signals, and detect treasure color and location.
In terms of software, [Dijkstra's algorithm probably best explained by David but I'll re read Milestone 3 for a refresher],
performs fast Fourier transform algorithms to detect audio and IR signal frequencies, transmits maze data to a base station over radio frequency communication, and adapts to detected robots in its vicinity.

## Hardware Included

* Larger wheels with rubber band treads as grips.
* A low sitting chassis for consistent line tracking.
* Ball bearings in the rear and just behind the line sensors in the front for stability and smooth traversal.
* Four photosensitive line sensors, three short range IR distance sensors (wall sensing), attached to a MUX for easy analog reading.
* A microphone and attached amplifier circuit.
* IR phototransistor for robot detection (MUX'd with the audio circuit since audio is only required for the starting command).
* An Arduino Genuino Uno for all processing except for treasure detection and for I/O.
* An FPGA and accompanying CMOS OV7670 camera for imaging, image processing, and transmission of shape/color data to the Arduino.
* A radio [brand? make? model?] for base station data transmission.

The base station has its own radio and Arduino Uno for updating the GUI (an interface provided byt he 3400 staff).

Everything is mounted to three main chassis plates, supported by struts, nuts, and bolts, hot glue, tape, and anything else to hold it together (including the sweat and tears of the invested Elevation Team).

## Pre-Competition Activities

## Performance Concerns
