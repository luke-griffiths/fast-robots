# Lab 10
The objective for this lab is to setup and use the virtual environment that simulates the robot, its environment, and its odometry and sensor data. 
For the simulator, ground truth is the exact position of the robot in the simulated environment. I will be comparing the robot's sensor data to this ground truth. 

Interaction between me and the simulator is facilitated by the Commander class. There are four functions to interact with the simulator itself:
```
set_vel(linear_vel, angular_vel)	#Set the linear velocity (m/s) and angular velocity (rad/s) of the virtual robot.
get_pose()	#Get the odometry and ground truth poses of the virtual robot as two numpy arrays. The units of each pose are (meters, meters, radians)
get_sensor()	#Get the ToF sensor data (in meters) of the virtual robot as a numpy column array.
reset_sim()	#Reset the virtual robot to its initial pose.
```
In addition, there are several functions in the Commander class to plot points in the plotter.
```
plot_odom(x,y)	#Plot a point (x,y) in the plotter in red. Units are (meters, meters).
plot_gt(x,y)	#Plot a point (x,y) in the plotter in green. Units are (meters, meters).
plot_bel(x,y)	#Plot a point (x,y) in the plotter in blue. Units are (meters, meters).
plot_map()	#Plot the map based on the map lines in world.yaml.
reset_plotter()	#Reset the plots in the plotter.
```

## Open-loop control
For this part of the lab I gave the robot a series of commands to follow to move in a square.
I used a series of alternating set_vel(linear_vel, angular_vel) and sleep functions to allow the robot to execute turns and straight-line drives. 
My open-loop square code was
```
cmdr.reset_sim()
cmdr.set_vel(0.5,0)
await asyncio.sleep(1)
print(cmdr.get_pose())
cmdr.set_vel(0,0.4)
await asyncio.sleep(3.9)
cmdr.set_vel(0.5,0)
await asyncio.sleep(1)
print(cmdr.get_pose())
cmdr.set_vel(0,0.4)
await asyncio.sleep(3.9)
cmdr.set_vel(0.5,0)
await asyncio.sleep(1)
print(cmdr.get_pose())
cmdr.set_vel(0,0.4)
await asyncio.sleep(3.9)
cmdr.set_vel(0.5,0)
await asyncio.sleep(1)
print(cmdr.get_pose())
cmdr.set_vel(0,0.4)
await asyncio.sleep(3.9)
cmdr.set_vel(0,0)
```
Note that set_vel() commands have an infinite duration. This is why I used sleep() to act as a timer before changing the set_vel command. A video of the robot completing this square is below.


https://user-images.githubusercontent.com/71809396/166268199-ffd56e5a-7aed-419e-83b2-542d5bf198db.mov

I wanted to see whether the robot followed the exact same path every time or if there was slight differences due to noise. I ran the open-loop square commands twice and used get_pose() at each point before the robot made a turn. Since get_pose() returns two arrays,
I only used the ground truth array since I would expect the odometry array to be different. Trial #1 resulted in these four sets of measurments
0.45833322	0	          0
0.49246258	0.47494012	1.58666539
0.01728401	0.49003166	3.17999721
-0.00510357	-0.01947328	4.7666626
and trial #2 resulted in these measurements
0.49166653	0	          0
0.51548159	0.46666497	1.57333207
0.01551206	0.50412995	3.16666389
1.34E-03	  -1.31E-02	  4.73E+00

Clearly there is some variation between the ground truth poses across different runs of the robot, which means the simulator introduces a bit of noise just like the real world. This also means that the robot does not travel the same exact square.
I also plotted the ground truth and odometry data for the square loop. The ground truth is in green, and the odometry is in red. Note how inaccurate the odemetry data is. This is the reason that I will use a Bayes filter in Lab 11 to make the odometry data better match the robot's ground truth.

<img width="796" alt="Screen Shot 2022-05-02 at 12 40 10 PM" src="https://user-images.githubusercontent.com/71809396/166288913-5b2e1ae3-5c01-4521-ab46-3383e7cd42fe.png">

## Closed-loop control



