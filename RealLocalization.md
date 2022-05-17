# Lab 12

## Simulation
The first step of this lab is running the simulated localization notebook, which uses Vivek's simulation to show the robot doing localization in the map. 
The screenshot below shows the robot's simulated localization results. The red trace is the odometry model's prediction, the blue trace is the belief using the Bayes filter, and the green is the robot's ground truth. 

<img width="631" alt="Screen Shot 2022-05-03 at 3 36 39 PM" src="https://user-images.githubusercontent.com/71809396/168680471-5c9ebc90-1e50-4b04-b3ea-0daa3599f804.png">

## Reality

Most of the work for this lab was already completed in lab 11. I made a SPIN command for the robot shown below
```
        case SPIN:{
            int numMeasures = 0;
            prevTime = millis() - 1000; //Just so we take a measurement at the zero angle
            while(numMeasures < 20){
                spin();
                if(millis() - prevTime > 1000){
                    global_data_array[numMeasures] = tof();
                    numMeasures += 1;
                    prevTime = millis();
                }
            }
            stop();
            break;}
```
This function takes a measurement once per second, which assumes that the robot's PID is keeping it spinning at 20 degrees per second for a total of 18 measurements. My robot then sends the array to the computer over bluetooth. Below is the code for perform_observation_loop(). Its main purpose is to prompt the robot to spin (by sending the SPIN command), receive the data from the robot, and then process the data into numpy arrays. 
```
    def perform_observation_loop(self, rot_vel=120):
        """Perform the observation loop behavior on the real robot, where the robot does  
        a 360 degree turn in place while collecting equidistant (in the angular space) sensor
        readings, with the first sensor reading taken at the robot's current heading. 
        The number of sensor readings depends on "observations_count"(=18) defined in world.yaml.
        
        Keyword arguments:
            rot_vel -- (Optional) Angular Velocity for loop (degrees/second)
                        Do not remove this parameter from the function definition, even if you don't use it.
        Returns:
            sensor_ranges   -- A column numpy array of the range values (meters)
            sensor_bearings -- A column numpy array of the bearings at which the sensor readings were taken (degrees)
                               The bearing values are not used in the Localization module, so you may return a empty numpy array
        """
       
        #This function assumes that the BLE IS ALREADY CONNECTED! 
        ble.send_command(CMD.SPIN, "") 
        time.sleep(25)
        ble.send_command(CMD.TRANSMIT_DATA, "")
        s = ble.receive_string(ble.uuid['RX_STRING'])
        s = s[1:len(s)-2] 
        fl = []
        for item in s.split(','):
            fl.append(float(item))
        arr = np.array(fl).T
        arr *= 0.001     #convert to meters
        arr2 = np.zeros_like(arr)
        for i in range(18):
            arr2[i] = 20 * i
        print(arr)
        return arr, arr2
```

All that's left is to calibrate my PID to the robot in the lab (since mine is broken), place the robot at each position and run the jupyter notebook to predict where the robot is. This will be done today, 5/17 in lab
