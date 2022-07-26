# Localization

The purpose of this lab is to implement a Bayes filter for robot localization. The tasks in this lab were to write 5 functions that make up the Bayes filter and then demonstrate via screen capture that the implementation works correctly. 


### compute_control
This function takes two poses and calculates the change between them using a rotation, translation, and then a final rotation. The idea is that any robot movement can be simplified to this sequence of movements. I calculated the translation as the euclidean distance between the x,y coordinates of the initial pose and the x,y coordinates of the current pose.  

```
def compute_control(cur_pose, prev_pose):
    """ Given the current and previous odometry poses, this function extracts
    the control information based on the odometry motion model.

    Args:
        cur_pose  ([Pose]): Current Pose
        prev_pose ([Pose]): Previous Pose 

    Returns:
        [delta_rot_1]: Rotation 1  (degrees)
        [delta_trans]: Translation (meters)
        [delta_rot_2]: Rotation 2  (degrees)
    """
    dx = cur_pose[0] - prev_pose[0]
    dy = cur_pose[1] - prev_pose[1]
    alpha = mapper.normalize_angle(np.degrees(np.arctan2(dy, dx)))
    
    delta_trans = np.sqrt(dx**2 + dy**2)
    delta_rot_1 = mapper.normalize_angle(alpha - prev_pose[2])
    delta_rot_2 = mapper.normalize_angle(cur_pose[2] - alpha)

    return delta_rot_1, delta_trans, delta_rot_2
 ```
 ### odom_motion_model
 This function outputs the probability of a state change given a previous state and a current state. Since the errors in state change are independent for rotation 1, translation, and rotation 2, we can compute the total probability by multiplying these three. 
 ```
 def odom_motion_model(cur_pose, prev_pose, u):
    """ Odometry Motion Model

    Args:
        cur_pose  ([Pose]): Current Pose
        prev_pose ([Pose]): Previous Pose
        (rot1, trans, rot2) (float, float, float): A tuple with control data in the format 
                                                   format (rot1, trans, rot2) with units (degrees, meters, degrees)


    Returns:
        prob [float]: Probability p(x'|x, u)
    """
    u_ex = compute_control(cur_pose, prev_pose)
    p1 = loc.gaussian(mapper.normalize_angle(u_ex[0] - u[0]), 0, loc.odom_rot_sigma)
    p2 = loc.gaussian(mapper.normalize_angle(u_ex[2] - u[2]), 0, loc.odom_rot_sigma)
    p3 = loc.gaussian (u_ex[1] - u[1], 0, loc.odom_trans_sigma)
    return p1 * p2 * p3
 ```
 ### prediction_step
 This function updates the bel_bar probabilities using the odometry motion model. Note that this code has 6 nested for-loops. This is extremely computationally expensive, and gave me a lot of trouble. I ended up restricting the innermost three for-loops by requiring the belief to be above a 0.0001 threshold, thus reducing the number of computations and speeding up this function. For future implementations, I would want to change this to using np functions rather than for-loops, because this would make computation much faster. 
 ```
def prediction_step(cur_odom, prev_odom):
    """ Prediction step of the Bayes Filter.
    Update the probabilities in loc.bel_bar based on loc.bel from the previous time step and the odometry motion model.

    Args:
        cur_odom  ([Pose]): Current Pose
        prev_odom ([Pose]): Previous Pose
    """
    u = compute_control(cur_odom, prev_odom)
   # u[0] = mapper.normalize_angle(u[0])
   # u[2] = mapper.normalize_angle(u[2])

    for i in range(12) :
        for j in range (9) :
            for k in range(18) :
                x_t_prev = mapper.from_map(i, j, k)
                if loc.bel[i][j][k] > 0.0001:
                    for x in range (12) :
                        for y in range(9) :
                            for z in range(18) :
                                x_t = mapper.from_map(x, y, z) 
                                prob = odom_motion_model(x_t, x_t_prev, u)
                                loc.bel_bar[x][y][z] += prob * loc.bel[i][j][k]
    #normalization                        
    loc.bel_bar = loc.bel_bar / np.sum(loc.bel_bar)
```
### sensor_model
This function the simplest of the 5, and simply returns an array of sensor measurements for a particular pose in the map. Note that this means the return is an array of 18 measurements. This function could also be done with a np function, which would improve efficiency; however, I returned to a simpler method when debugging because I wanted to make sure the issue wasn't with sensor_model. 
```
def sensor_model(obs):
    """ This is the equivalent of p(z|x).


    Args:
        obs ([ndarray]): A 1D array consisting of the true observations for a specific robot pose in the map 

    Returns:
        [ndarray]: Returns a 1D array of size 18 (=loc.OBS_PER_CELL) with the likelihoods of each individual sensor measurement
    """
    prob = np.zeros_like(obs)
    for i in range(18):
        prob[i] = loc.gaussian(loc.obs_range_data[i], obs[i], loc.sensor_sigma)
    return prob
```

### update_step
The update step is where I had an issue. Thank you Jonathan for pointing out my error. Because sensor_model returns an array of 18 measurements, I need to take the product of all 18 measurements. This is the probability of a particular measurement given a particular pose (P(Zt | Xt)). I used np.prod() to multiply the array; however, this means I need to scale the belief at the end. 

```
def update_step():
    """ Update step of the Bayes Filter.
    Update the probabilities in loc.bel based on loc.bel_bar and the sensor model.
    """
    for i in range(12):
        for j in range(9):
            for k in range(18):
                loc.bel[i][j][k] = loc.bel_bar[i][j][k] * np.prod(sensor_model(mapper.get_views(i, j, k)))
    loc.bel = loc.bel / np.sum(loc.bel)
```

## Video 
Below you can see that my implementation works. The red line is the robot without running the Bayes filter, the green line is the robot's actual movement, and the blue line is the predicted movement with the Bayes filter. Note how incorrect the red line gets, as well as how well the blue line follows the actual movement of the robot. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/xgGgxwPifBg" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## Data 
Here is a sample of the ground truth poses and the belief of the robot's pose. Clearly the robot is able to accurately determine the tile it is in and rarely makes a mistake. Within the tile there is some difference between the ground truth and the robot's actual position, but it's always very close. The robot is most likely to have an incorrect belief when it's at a tile that is near another tile which would have very similar measurements during a rotation. For example, if the robot was in the middle of a large open space, it would have difficulty differentiating between two adjacent tiles since the 18 measurements during rotation would likely be very similar. When the robot is moving through landscapes that have drastic differences in measurements for adjacent positions (such as near a doorway) the Bayes filter is going to have a much easier time guessing where it is. 

<img width="331" alt="Screen Shot 2022-04-26 at 5 16 20 PM" src="https://user-images.githubusercontent.com/71809396/165394018-44880d46-b157-4e55-93bc-9b11b5847182.png">





