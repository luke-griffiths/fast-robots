# Localization
The purpose of this lab is to implement a Bayes filter for robot localization. The tasks in this lab were to write 4-5 functions that make up the Bayes filter and then demonstrate via screen capture that the implementation works correctly. 
### compute_control
This function takes two poses and calculates the change between them using a rotation, translation, and then a final rotation. 

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
 This function outputs the probability of a state change given a previous state and a current state. 
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
 This function updates the probabilities from the previous time step using the odometry motion model. 
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
                for x in range (12) :
                    for y in range(9) :
                        for z in range(18) :
                            x_t = mapper.from_map(x, y, z) 
                            prob = odom_motion_model(x_t, x, u)
                            loc.bel_bar[x][y][z] += prob * loc.bel[i][j][k]
    #normalization                        
    loc.bel_bar = loc.bel_bar / np.sum(loc.bel_bar)
```
### sensor_model
This function is implemented incorrectly. I originally returned one single measurement, not an array of 18. This needs to be updated !!!
```
def sensor_model(obs):
    """ This is the equivalent of p(z|x).


    Args:
        obs ([ndarray]): A 1D array consisting of the true observations for a specific robot pose in the map 

    Returns:
        [ndarray]: Returns a 1D array of size 18 (=loc.OBS_PER_CELL) with the likelihoods of each individual sensor measurement
    """
    s = mapper.get_views(get_pose())
    prob = np.zeros_like(obs)
    prob[:] = loc.gaussian(obs[:] - s[:], 0, loc.sensor_sigma)
    return prob
```

### update_step
This function simply performs the update of beliefs using the sensor model and bel_bar. 

```
def update_step():
    """ Update step of the Bayes Filter.
    Update the probabilities in loc.bel based on loc.bel_bar and the sensor model.
    """
    for i in range(12):
        for j in range(9):
            for k in range(18):
                loc.bel[i][j][k] = loc.bel_bar[i][j][k] * sensor_model(mapper.get_views(i, j, k))
    loc.bel = loc.bel / np.sum(loc.bel)
```



