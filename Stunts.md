# Lab 8
Actively being written (1:41 PM 4/22)

The purpose of this lab is to use the control methods used in previous labs to drive a car toward a wall, flip when 0.5m from the wall, and drive back in the direction it came. Unfortunately, my car has been plagued with mechanical issues which has prevented me from completing the stunt. I will still detail my process below as well as how the stunt could be implemented on a working car. 

### Hardware

![IMG_6453](https://user-images.githubusercontent.com/71809396/164754864-a224776a-2f57-4c0e-926b-4fa36a3728e7.jpg)

When I first tried to execute the stunt, my car overshot the 0.5m mark and slammed into the wall. This caused one of the wires that were soldered to the motor controller boards to come loose and short the board. After replacing the board, I decided to use male header pins to better secure the wires to the board. Since the wires themselves had female ends, there was no risk that they would short if a wire came loose. The other pieces of hardware I used were a single forward-facing ToF sensor and an IMU. 

### Control

In previous labs I used PID to control the robot; however, PID responds proportionally to the error between the robot's current position and the setpoint. This meant that the robot would slow down as it approached the 0.5m mark, which is not ideal for a flipping stunt. Instead, I wanted the robot to drive full speed until it reached the setpoint, then drive full speed in the opposite direction. The sudden drastic change should be enough to flip the car, provided the tires can gain enough traction on the floor. 

The simplest way to implement this lab would be to have the car drive forward at full speed toward the wall, then send a command to the robot to change direction; however, this would not require the ToF sensor or IMU and could be accomplished with any regular remote control car. A better alternative would be to use the Kalman filter from lab7 to allow the car to get as close to the 0.5m mark as possible. The Kalman filter is needed because the ToF sensor can update only once every ~40ms, in which time the car can move up to 10cm (the car's max speed, 2.4m/s * 40 ms). By using a Kalman filter we can combine a motion model with the ToF sensor data to more accurately flip. I would use the exact same implementation of the Kalman filter from lab7. My implementation used a global boolean variable "reachedSetPoint" which was set to true as soon as the Kalman filter output reached 0.5m. If reachedSetPoint is false, the car would drive full-speed forward else it would drive full-speed in reverse. The implementation is below. 

<img width="677" alt="Screen Shot 2022-04-22 at 2 10 27 PM" src="https://user-images.githubusercontent.com/71809396/164770938-8c5dd4da-9a3f-4b5b-a655-3dfa5aef65cf.png">

The implementation of kfFunction is shown below. It uses the same Kalman filter as lab7, except it changes the value of reachedSetPoint to true if the filter output is less than 0.5m. 

** place screenshot here
