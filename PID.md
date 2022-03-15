# PID Control
The purpose of this lab is to implement and tune a PID controller for the car that is capable of stopping the car 300mm from an object in its path. 
The simplest part of this lab was the actual PID implementation. The bulk of time spent in this lab was either developing the bluetooth commands to send data between
the computer and the car or analyzing the car sensor data to tune the PID constants. 
## Bluetooth Commands


## PID Implementation


## Tuning PID
Here is a video of the car driving before modifying the PID to treat positive values more significantly than negative values. As you can see, the car slams into the wall before backing up to the correct distance. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/XpTaJQuNRc" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

Next, I modified the car's PID to weight positive values (closer to the wall than 300mm) more heavily so it would avoid hitting the wall. Here are some clips. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/VXeETTDI_TM" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

Finally, here is a video of Kp being too high followed by a clip where Kp is too low. As you can see, when Kp is too high, the system starts to oscillate. If I continued to increase Kp, it could eventually become unstable. When Kp is too low, the car is very sluggish when moving toward the set point. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/Ct2ve_t8yLo" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>


