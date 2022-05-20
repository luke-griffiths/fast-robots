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
  The second, more simple option would be to use open-loop control such as PID along with a pre-cached 
  

## My Chosen Implementation

## Results

## Ways to Improve Results/Future Iterations
