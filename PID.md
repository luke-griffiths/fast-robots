# PID Control
The purpose of this lab is to implement and tune a PID controller for the car that is capable of stopping the car 300mm from an object in its path. 
The simplest part of this lab was the actual PID implementation. The bulk of time spent in this lab was either developing the bluetooth commands to send data between the computer and the car or analyzing the car sensor data to tune the PID constants. 
## Bluetooth Commands
I implemented several new commands to streamline the process of sending data between the computer and Artemis. I first made the PID constants Ki, Kp, and Kd global variables. I also created global data arrays so I could transmit sensor data from test runs to the computer to analyze. I then added several command functions including SET_VEL, CUT_POWER, TRANSMIT_DATA, UPDATE_PID_CONSTANTS, and TRANSMIT_CURRENT_PID_CONSTANTS. I'll only include one example, but these commands are all similar. 

<img width="499" alt="Screen Shot 2022-03-15 at 12 20 52 AM" src="https://user-images.githubusercontent.com/71809396/158305873-711b8085-1a28-494c-95bc-82f4564383b7.png">

## PID Implementation

My PID implementation is simple, and takes up just a few lines. 

<img width="499" alt="Screen Shot 2022-03-15 at 12 20 52 AM" src="https://user-images.githubusercontent.com/71809396/158306243-502b2304-dbd8-466b-b469-56f5719f6cf6.png">

However, since I want the car to stop 300mm from the wall, I need the PID output to activate different pins on the Artemis depending on whether the car is less than or greater than 300mm from the wall. Since my implementation gives negative PID output values for sensor measurements that exceed the setpoint, I write signals to pins 1 and 3 when negative and 0 and 2 when positive. 

<img width="719" alt="Screen Shot 2022-03-15 at 12 28 26 AM" src="https://user-images.githubusercontent.com/71809396/158306654-f0476b97-629c-4807-b7d1-200fac04c7e6.png">

To clamp the values, I use the above conditional statements to ensure that values are never written to the motors that go out of range. minVal is set to 25 (the deadband) and maxVal is set to 255. 

One issue I had was the car slamming into the wall. To prevent this, I weighted positive PID output values (meaning the car is less than 300mm from the wall) heavier. I simply added a scale factor to positive PID outputs to increase the values written to the motors if the car was close to the wall. This helped prevent the car from slamming into the wall when traveling toward it at a high speed. 

I'll now discuss PID in general and my rationale for the values I chose. 

### P
The proportional term is simply a scaled version of the current error. This means that the greater the error, the greater the P response (in linear fashion). A plot of Kp versus distance to the wall would look something like this:

<img width="375" alt="Screen Shot 2022-03-15 at 12 58 43 AM" src="https://user-images.githubusercontent.com/71809396/158309955-e27ac941-6bba-421d-a0f1-8d81b47acc4f.png">

Note that the region where x < 300 is significantly more sloped. In this example the scaling factor for positive P values is 3, and Kp is set to 0.2. This Kp term is what I ended up settling on for my actual car, though different floor conditions may cause me to adjust this later. When Kp is too high, the system oscillates too much. The car continues to overshoot the setpoint and exhibits underdamped motion; however, if Kp is too low, the car simply takes too long to get to the setpoint. This results in a sluggish car that slows down as it rolls to the 300mm point, and is not ideal for what should be a fast car. A video of both of these cases is in the following section. 

### I
I decided against using an integral term in my PID controller, because even extremely low integral term values caused the PID output to dramatically shoot up and oversaturate the motors by writing the max value to them. In addition, the integral term is somewhat dangerous in a system like this one because if the robot gets stuck on something or is held back momentarily, integrator windup occurs and the car will overshoot the setpoint. Thus I decided to make my PID controller a PD controller. 

### D
The derivative term is a measure of how fast the error is changing with respect to time. The approximation which is used in PID is the difference between the previous error and the current error divided by the time since the last measurement was taken. This means that a quick change in error (from me suddenly moving the car further away from the setpoint) will cause a large valued derivative term. I settled on a value of 0.75 after testing. 

The total output of my PD controller is the sum of the proportional and derivative terms. I didn't need to worry about derivative kick for the D term because the nature of the car means it's okay if one or two cycles after a change in setpoint saturate the PID output and write high values to the motors. This is because I'm sampling frequently, and the motors aren't being asked to do anything too precise. If the system was different and a derivative kick could cause a current surge to the motors or something similar, I'd need to worry about this, but for these simple motors it isn't an issue. As for my sampling rate, I'm updating PID at a rate that is nearly two times faster than my sensing distance (once every ~11ms).The range of values was 7-13ms, but it's pointless to try to update PID faster since it already exceeds the maximum frequency at which the ToF sensor can update, which is once per 20ms. The sensor is updating once approximately every 50ms, but I might choose to make this faster at the expense of measurement accuracy later on.   

## Tuning PID
Here is a video of the car driving before modifying the PID to treat positive values more significantly than negative values. As you can see, the car slams into the wall before backing up to the correct distance. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/XpTaJQuNRc" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

Next, I modified the car's PID to weight positive values (closer to the wall than 300mm) more heavily so it would avoid hitting the wall. Here are some clips. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/VXeETTDI_TM" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

Finally, here is a video of Kp being too high followed by a clip where Kp is too low. As you can see, when Kp is too high, the system starts to oscillate. If I continued to increase Kp, it could eventually become unstable. When Kp is too low, the car is very sluggish when moving toward the set point. The max speed I could drive the car at the wall (without crashing) was estimated to be about 1.5m/s, which is much slower than the 2.4 m/s max speed I measured in an earlier lab; however, when I modified the setpoint to be 400mm from the wall, I was able to drive much closer to 2.4 m/s and not crash. The speed tests were done in my apartment where the floor is slippery and there isn't much room to accelerate, so these values can hopefully be improved with a sticky mat in the lab. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/Ct2ve_t8yLo" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>


