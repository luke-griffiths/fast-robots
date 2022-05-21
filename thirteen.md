# Lab 13
## Overview
The goal for this lab is to combine all the things I've learned over the semester into one project. The expectation is that the robot will be able to navigate through the map, hitting several waypoints along the way. This means that I must combine aspects of 
* Motor Control
* PID
* Graph search Algorithms
* Localization
* Mapping
* Control
* Sensor Fusion
* Bluetooth communication

to create a robot that can hit waypoints in a maze. I'll first discuss the different options I had, followed by the implementation I chose. 
## Strategies
### 1: Grid Based Localization and Graph Search (BFS)
My first thought was to use the localization code from lab12 accompanied with a BFS implementation to navigate between waypoints. This meant that I needed to create a new command for the arduino, CHANGE_WAYPOINT. This command would have allowed me to send the next destination to the robot, which would localize to determine its current position, compute an optimal path to the next waypoint using a BFS, and then use city-block movements (90 degree turns) to get from its current position to its destination. CHANGE_WAYPOINT would be need x and y coordinates, as well as an orientation since turning would cost time. The cost for turning would have to be accounted for in the BFS, so that solutions would minimize translations made and turning time. The advantages of this strategy is the robot's ability to "get back on track" if it gets lost by using localization. The disadvantage is that localization would have to work extremely well for this to be feasible. In addition, localizing at every step costs time. 
```
        case CHANGE_WAYPOINT:
            float x, y, theta;
            success = robot_cmd.get_next_value(x);
            if (!success)
                return;
            waypoint[0] = x;
            success = robot_cmd.get_next_value(y);
            if (!success)
                return;
            waypoint[1] = y;
            success = robot_cmd.get_next_value(theta);
            if (!success)
                return;
            waypoint[2] = theta;
            break;
  ```
  ### 2: Open-loop Control, pre-cached translations/rotations
  The second, more simple option would be to use open-loop control such as PID along with a pre-cached set of instructions for the robot to follow to move from waypoint to waypoint. This method requires a lot of faith in the robot's turning reliability, because turning slightly too much or slightly too little could cause the robot to get lost. Since this method doesn't use localization, one bad turn early on could cause the robot to miss all the remaining waypoints. This implementation would require taking various measurements at different points throughout the map, calibrating PID to the lab floor, and using PID on the ToF sensor values and map measurements to move from waypoint to waypoint. The advantage of this method is that it's simpler to implement than localization, and doesn't rely on the cheap IMU giving an accurate spin for localization. The disadvantage is that it trusts that a 45 degree turn is actually a 45 degree turn and not 52 degrees. Inaccurate turns means the robot can get lost, and having a precached sequence of translations and rotations won't help a robot that's lost. 
  

## My Chosen Implementation

My results from lab12 showed that localization was good, but not great. The robot perfectly and reliably localized half of the time. When the environment had few distinct features (such as the upper right corner of the map behind the island), the robot had a tough time figuring out where it was. In addition, the unreliability of the IMU played was a huge factor in deciding whether to use localization. Even with a localization_PID to try to keep the robot spinning consistently at 20 degrees, the wheels got stuck, spun too quickly, or acted erratically too frequently to trust localization. 

Instead, I went with a modified open-loop control method. I placed the robot at positions throughout the map and recorded the ToF readout. This allowed me to generate the sequence of distances to travel below. Note that I also used basic math to compute the angles required to make each turn. Unfortunately, my robot didn't turn consistently enough on its own, so I modified the open-loop control by making the robot ask for permission before executing the next step. This was implemented using a simple AM_I_OK(?) command that was called in two cases:

* After every rotation
* After the robot believed it had reached each waypoint

This robot stopped whenever it completed one of these actions, and the value sent in the AM_I_OK command determined its next course of action. Originally I had the command return a boolean to the robot, allowing it to proceed if AM_I_OK == true and adjust if false, but this would only work if all needed adjustments were equal(I'll talk more about this in the Future Iterations section). Instead, I made the command return a float from 0-10, where a 0 meant no correction was needed and a 10 meant repeat the previous action at full effort value. 

Code samples of each new function I made are below, as well as a map showing the robot's moves.

!!!! Insert Turn function


!!!! Insert travel function


!!!! Insert AM_I_OK command


!!!! Insert loop() sequence of instructions


!!!! Insert map of ToF data

## Results
My robot was able to navigate through the waypoints, although it did make some mistakes. It generally stayed pretty close to the points I tried to hit. I noticed during the runs that the robot had several situations that caused it to underperform.

* turns were rarely satisfactory. The lab floor alternated between dirty and freshly swiffered, which made it difficult to calibrate the PID for turns. This meant that I needed to tell the robot to re-execute turns way more frequently than I would have liked. 
* the ToF sensor was angled slightly downward on the robot, which caused it to stop prematurely on long travels. This was easy to overcome using the AM_I_OK command, but it wasn't ideal.

Below are two runs of the robot. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/S_FE6KnpowE" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/hSddVXIX4f4" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>


## Ways to Improve Results/Future Iterations

For the future I would like to use a hybrid presequenced steps and localization approach. When I originally planned to make the AM_I_OK command a boolean, I thought it would be a good idea to use localization if and only if AM_I_OK returned false. This would mean that localization could be used to correct mistakes that had accumulated, but it would be used sparingly and thus cost less time. This would be implemented by following the sequence of pre-cached translations and rotations UNTIl AM_I_OK returned false, it which point the robot would localize, attempt to return to the nearest waypoint, and then resume its pre-cached instructions. This approach would combine advantages of both methods and hopefully mitigate the disadvantages. 

A very creative approach for the future would be to create a makeshift scanner for localization. A ToF sensor could be mounted on a servo on the top of the robot, allowing very precise ToF measurements while the robot is stopped. This would generate a much better localization result, since the robot wouldn't move from its initial position while localizing. In addition, even cheap feedback servos are more reliable than the gyroscope data on the IMU, so the change in theta for ToF measurements would be more reliable. 
