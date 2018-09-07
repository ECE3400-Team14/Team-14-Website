#  Lab 1: Microcontrollers

## Blinking an internal LED:

## Blinking an external LED:

## Reading the value of a potentiometer via the serial port:

## Map the value of the potentiometer to the LED:

## Map the value of the potentiometer to the servo:

## Assembling the robot:
When building our robot testing, we used the following parts:
* Arduino Uno
* 3D-Printed Chassis
* 2x Servos
* 2x 3D-Printed Servo mounts
* Rechargeable 9V battery
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


We decided to mount the battery pack under the chassis using the provided battery mounts. This setup allowed the battery to remain secure during motion while allowing for the battery pack to be easily added or removed. 

<img src="https://user-images.githubusercontent.com/12742304/45185080-f6790e80-b1f6-11e8-92de-32330f5be5b5.jpg" width="400" />

We were concerned that the USB cord would not reach from the Arduino USB port to the battery, so we initially created a customized connector for connecting the USB input of the battery to the Arduino power jack. After testing this, we realized that the Arduino power jack required 9 Volts of power while the battery only provided 5 Volts. We discovered, however, that the provided USB cable was just long enough to reach from the battery to the Arduino USB port, allowing us to power the Arduino from the Battery successfully. 

## Driving our robot autonomously:

```cpp
void loop() {
  forwardAndLeft();
  forwardAndRight();
  forwardAndRight();
  forwardAndRight();
  forwardAndRight();
  forwardAndLeft();
  forwardAndLeft();
  forwardAndLeft();
}
```
