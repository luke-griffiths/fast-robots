# Lab 7
**being written right now, 4/24 2:03p
## Overview
The purpose of this lab is to implement a Kalman filter that takes data from a ToF sensor and combines it with a motion model to calculate a more accurate measurement of my car's position as it drives toward a wall. 
## Data collection
I first drove the car toward a wall using a fixed PWM value of 80 (out of 255 max) because I wanted the car to be fast enough to reach steady state before hitting the wall, but not so fast that it would smash itself to pieces. Note that since the motor response is not identical, I actually used 80 PWM for the left motor and 85 PWM for the right so the car could travel in a straight line. That doesn't have any effect on the Kalman filter result, so it isn't a problem. I created two arrays of 50 elements each, one to store the current time (by calling the millis() function) and one to store the current ToF sensor output. 
