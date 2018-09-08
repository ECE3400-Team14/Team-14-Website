#  Lab 1: Microcontrollers

**Purpose:** The goal of this lab was to become familiar with the Arduino Uno platform and the Arduino IDE. This page describes how to connect various hardware components, such as LEDs, potentiometers, and servos, to the Arduino, and we show how to write code to control these devices through the Arduino's analog and digital ports. The final part of this lab details the construction and testing of our initial robot design. 

The Arduino IDE was downloaded from [here](https://www.arduino.cc/en/Main/Software). We used the [Arduino reference page](https://www.arduino.cc/reference/en/) for learning the various commands available for programming our Arduino.  

## Blinking an Internal LED:
In order to familiarize ourselves with basic Arduino code, we used one of the examples already provided with the Arduino IDE. The example used in order to blink the LED is called Blink, and can be accessed within the IDE by selecting File -> Examples -> Basics -> Blink. 

### Code Segment 1:
```cpp
void setup() {
  // initialize digital pin LED_BUILTIN as an output.
  pinMode(LED_BUILTIN, OUTPUT);
}
```

As seen in **Code Segment 1** above, we are setting the built-in LED port as an output, which will allow us to toggle it off and on. 

### Code Segment 2:
```cpp
void loop() {
  digitalWrite(LED_BUILTIN, HIGH); // turn the LED on with HIGH voltage
  delay(1000); // wait for a second
  digitalWrite(LED_BUILTIN, LOW); // turn the LED off with LOW voltage
  delay(1000); // wait for a second
}

```

**Code Segment 2** is the main function/body of the Blink program. The `digitalWrite()` function takes a pin name and a value as an input. The HIGH and LOW values have already been predefined in the Arduino IDE and correspond to setting the output LED on/off. The `delay()` function here takes an input in ms and allows us to actually see the LED toggle between states.  

### Video Demonsration:
<iframe width="560" height="315" src="https://www.youtube.com/embed/JMLbzeyAlCI" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe> 

&nbsp;


## Blinking an External LED:
The code used to blink the external LED is very similar to the example Blink program used above. Instead of using the `LED_BUILTIN` as the output, we used an I/O pin to toggle between HIGH and LOW values. 

### Code:
```cpp
void setup() {
  // initialize digital pin 6 as an output.
  pinMode(6, OUTPUT);
}
// the loop function runs over and over again forever
void loop() {
  digitalWrite(6, HIGH);   // turn the LED on (HIGH is the voltage level)
  delay(1000);             // wait for a second
  digitalWrite(6, LOW);    // turn the LED off by making the voltage LOW
  delay(1000);             // wait for a second
}

```

This was all the code needed to make an external LED blink. On the hardware side, we still had to connect the external LED to the Arduino. In order to do this, we simply have to wire the previously defined output pin in series with a 330 ohm resistor, to prevent the LED pin from taking too much current and blowing out, and then grounding the LED with the already defined ground pin on the Arduino. 

### Circuit Diagram:
![led only](https://user-images.githubusercontent.com/16722348/45246918-0a457300-b2d2-11e8-9483-712681f4a5cc.png)

### Video Demonstration:
<iframe width="560" height="315" src="https://www.youtube.com/embed/G_6xJG0BkIg" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe> 

&nbsp;

## Reading the Value of a Potentiometer via the Serial Port: 

A potentiometer is essentially a voltage divider, with an adjustable ratio of input to output voltage. By hooking up one end to HIGH (5V) and the other end to ground, we can adjust the output voltage of the potentiometer between those two values. We connected the analog pin to the potentiometer through a 330 ohm resistor as a precaution to ensure the analog port did not take too much current.

### Circuit Diagram:

![potentiometer](https://user-images.githubusercontent.com/16722348/45246920-0a457300-b2d2-11e8-8558-bc733f779b9c.png)

Using the `analogRead()` in-built function on the Arduino, we can utilize an analog to digital converter in the microcontroller to read this adjustable voltage as a number between 0 and 1023. A value of zero corresponds to the potentiometer’s minimum, and 1023 to its maximum. Using a print statement, this values can be seen clearly.

### Code:
```cpp
int A_PINNAME = A0;//analog pin number

void setup() {
  // initialize serial port
  Serial.begin(9600);
}

void loop() {
  //Read analog pin value connected to potentiometer
  int value = analogRead(A_PINNAME);

  //print potentiometer value to serial port
  Serial.println(value);
}

```
### Video Demonstration:
<iframe width="560" height="315" src="https://www.youtube.com/embed/t6pg28G1tBA" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe> 

&nbsp;

## Mapping the Value of the Potentiometer to the LED:

It happens that the Arduino can output a pulse-width-modulated square waveform in order to turn on and off an LED rapidly. This function is `analogWrite()`. The percentage of the period of this oscillation that the signal reads HIGH corresponds to how long during each period that the LED is on. By decreasing this percentage, over the same period, the LED is on for a shorter amount of total time. Since this frequency is still well above the threshold of human persistence of vison, the LED simply appears dim.

After reading in an analog value from a potentiometer as a number between 0 and 1023, it can be linearly mapped to any other range of values. The `analogWrite()` function takes values of 0 to 255, so we map to those values. Now, by adjusting the potentiometer, and therefore the value fed to `analogWrite()`, the LED dims or brightens accordingly.

### Code:
```cpp
int PINNAME = 10;//digital PWM pin # connected to LED
int A_PINNAME = A0;//Analog pin # connected to potentiometer

void setup() {
  // initialize digital pin PINNAME as an output for PWM.
  pinMode(PINNAME, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  //Read analog pin value connected to potentiometer
  int value = analogRead(A_PINNAME);

  //Map analog pin value to between 0 and 255 for PWM output
  int mapValue = map(value, 0, 1023, 0, 255);

  //write value to PWM output pin  PINNAME
  analogWrite(PINNAME, mapValue);
  //display PINNAME value in serial port
  Serial.println(mapValue);
}
```
### Circuit Diagram:

![pot-and-led](https://user-images.githubusercontent.com/16722348/45246919-0a457300-b2d2-11e8-9115-843a7708181d.png)

### Video Demonsration:
<iframe width="560" height="315" src="https://www.youtube.com/embed/-2hUMhN6iGc" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>  

&nbsp;

## Mapping the Value of the Potentiometer to the Servo:

Similar to the LED dimming, the potentiometer read value is mapped to a range of numbers, this time between 0 and 180. The continuously rotating servos will rotate one way at full speed at 180, and in the opposite direction at full speed at 0, where 90 corresponds to a stationary servo. Therefore in the middle of the potentiometer range, we saw the servo stop moving entirely, and at the maximum and minimum the servo turned quite fast!

### Code:
```cpp
Servo myservo; // create servo object to control a servo

int PINNAME = 10;//digital PWM pin # connected to servo
int A_PINNAME = A0;//Analog pin # connected to potentiometer

void setup() {
  // initialize digital pin PINNAME as an output for PWM.
  pinMode(PINNAME, OUTPUT);
  Serial.begin(9600);
  //connect digital pin PINNAME to servo object
  myservo.attach(PINNAME);
}

void loop() {
  //Read analog pin value connected to potentiometer
  int value = analogRead(A_PINNAME);
  
  //Map analog pin value to between 0 and 180 for servo control
  int mapValue = map(value, 0, 1023, 0, 180);
 
  //write value to PWM output pin PINNAME
  myservo.write(mapValue);  
  //display PINNAME value in serial port
  Serial.println(mapValue);
}
```

### Circuit Diagram:
We connected the servo to the Arduino, powering it through the Arduino 5V pin and connecting the servo signal pin to a digital pin through a 330 ohm resistor as a precaution to ensure the pin does not draw too much current. 

![servo](https://user-images.githubusercontent.com/16722348/45246921-0a457300-b2d2-11e8-862a-ae9048efdca7.png)

### Video Demonsration:
<iframe width="560" height="315" src="https://www.youtube.com/embed/JtpTWwkTesI" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe> 

&nbsp;

## Assembling the Robot:
When building our robot testing, we used the following parts:
* Arduino Uno
* 3D-Printed Chassis
* 2x Servos
* 2x 3D-Printed Servo mounts
* Rechargeable 5V battery
* USB Cable
* 2x 3D-Printed battery mounts
* Ball bearing + screw mount
* 3D-printed ball bearing extension mount
* Small Breadboard
* Tape
* Screws, Bolts, & Standoffs
* Wires

First, we mounted the Arduino Uno to the top-center of our chassis using plastic mounting screws and standoffs: 


<img src="https://user-images.githubusercontent.com/12742304/45184889-51f6cc80-b1f6-11e8-9b3a-dd86a88d3815.jpg" width="400" />


Next, we attached wheels to our servos and attached the servos to the servo mounts using bolts. We then connected the servo mounts to the underside of the chassis.

<img src="https://user-images.githubusercontent.com/12742304/45184918-65a23300-b1f6-11e8-90df-89f9d2c8d415.jpg" width="300" /> <img src="https://user-images.githubusercontent.com/12742304/45184936-7783d600-b1f6-11e8-8db8-5fa590b3e585.png" width="300" />



Next, we mounted a 3D-printed ball bearing extension to the back (rounded side) of the chassis in order to provide stabilization for the robot. We spend time trying out various extension mounts available in the lab in order to find one that was close to the height of the chassis with the wheels attached. 

<img src="https://user-images.githubusercontent.com/12742304/45184955-8d919680-b1f6-11e8-8160-354b3899fe99.jpg" width="400" />

After this, we attached the battery pack and breadboard to the chassis. We first tried taping the breadboard to the bottom of the chassis, but this approach was unsuccessful since the tape was not strong enough to hold up the breadboard. It was finally decided that we tape the breadboard to the top of the chassis for now, and later create custom parts to more securely mount the breadboard on the chassis.


<img src="https://user-images.githubusercontent.com/12742304/45185048-e3663e80-b1f6-11e8-9781-4d4908149fd9.jpg" width="400" />

### Circuit Diagram:

<img src="https://user-images.githubusercontent.com/12742304/45248389-1800f580-b2de-11e8-971d-0d76bd3425da.png" width="400" />


We decided to mount the battery pack under the chassis using the provided battery mounts. This setup allowed the battery to remain secure during motion while allowing for the battery pack to be easily added or removed. 

<img src="https://user-images.githubusercontent.com/12742304/45185080-f6790e80-b1f6-11e8-92de-32330f5be5b5.jpg" width="400" />

We were concerned that the USB cord would not reach from the Arduino USB port to the battery, so we initially created a customized connector for connecting the USB input of the battery to the Arduino power jack. After testing this, we realized that the Arduino power jack required 9 Volts of power while the battery only provided 5 Volts. We discovered, however, that the provided USB cable was just long enough to reach from the battery to the Arduino USB port, allowing us to power the Arduino from the Battery successfully. 



## Driving our Robot Autonomously:
Having two servos connected to our Arduino board, we can simply move our robot using a few lines of code. If we were to make our robot go forward, we would have the two servos move in the same direction. If we were to make it turn, we would have the two servos move in the opposite direction. Wrapping these basic intructions for movement into functions, we have something like this:

### Code Segment 1:
```cpp
#include <Servo.h>
Servo left;
Servo right;
void setup() {
  left.attach(3);
  right.attach(5);
}
void forward(){
  left.write(180);
  right.write(0);
  }
void turnLeft(){
  left.write(0);
  right.write(0);
  }
void stopMovement(){
  left.write(90);
  right.write(90);
  }
...
```

After this, we coded some random pattern with random delays in between random movements. Then, we tested it on the floor as we didn't want our robot to fall off the table.
<iframe width="560" height="315" src="https://www.youtube.com/embed/cHkXJKhpaUw" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

Since we made sure each function works, we moved on to having the robot move in a specific pattern, like a square or a figure-8. At this point, we had yet to install line sensors for our robot, thus the only way for it to move in a specific pattern was to hard code the delays between movements into the program. After testing different delay time, we achieved an accuracy that we were satisfied with.

### Code Segment 2:
```cpp
void forwardOneBlock(){
  forward();
  delay(1700);
  stopMovement();
  }
void left90(){
  turnLeft();
  delay(670);
  stopMovement();
  }
void right90(){
  turnRight();
  delay(670);
  stopMovement();
  }
```
The only thing left now is to combine these functions to create said patterns. For a figure-8, it only takes a few lines.

### Code Segment 3:
```cpp
void loop() {
  forwardOneBlock();
  right90();
  forwardOneBlock();
  left90();
  forwardOneBlock();
  left90();
  forwardOneBlock();
  left90();
  forwardOneBlock();
  left90();
  forwardOneBlock();
  right90();
  forwardOneBlock();
  right90();
  forwardOneBlock();
}
```

### Video Demonstration of Figure-8 Motion
<iframe width="560" height="315" src="https://www.youtube.com/embed/mq7fXP7EtU8" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
