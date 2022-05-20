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

I used Kirsten's robot for this lab, since my motors are broken. I placed the robot at each of the positions and got the localization results shown below. 

### Real (-3,-2)
Result (-3,-2). 

<img width="792" alt="Screen Shot 2022-05-19 at 2 43 17 PM" src="https://user-images.githubusercontent.com/71809396/169465609-7e8cceaa-0ac9-4efd-b6b5-872ca4964014.png">

![-3-2](https://user-images.githubusercontent.com/71809396/169466970-2ded40da-0bb3-4573-9026-b71f98725ad9.png)

Clearly this result shows that that localization is rotation invariant. My first measurement was taken with the car facing the front of the room rather than the 0 degree direction, however, the localization code still works great. 

### Real (0,3)
Result (0,3). 

<img width="787" alt="Screen Shot 2022-05-19 at 2 41 26 PM" src="https://user-images.githubusercontent.com/71809396/169467519-0857f686-73ac-4e8a-a226-2d708e4dfdc6.png">

![03](https://user-images.githubusercontent.com/71809396/169468101-fc578fef-42e3-4dac-847b-586e2c8d23c1.png)

Once again, this result correctly predicted the exact tile that the car was in. You can also tell that the car was again pointed forward rather than at the 0 degree point (to the right). So far so good!

### Real (5,3)
Result (6,4). This result is a bit off from the actual value. I noticed that my PID was having difficulty keeping the car at a steady 20 degrees/second at this spot on the floor. Since the robot seemed to be getting stuck, I tried to increase Kp to account for this. This made my car react too aggressively and the localization results came out even worse, so I dropped back down to my previous Kp value. Unfortunately, this meant that my car undershot the 360 degree circle, and actually went about 330. This means that several measurements got clustered together, and the robot got slightly confused as to where it was. 

<img width="790" alt="Screen Shot 2022-05-19 at 2 50 02 PM" src="https://user-images.githubusercontent.com/71809396/169469899-e22c5656-88c7-42e5-b5ae-da7097186631.png">

![53](https://user-images.githubusercontent.com/71809396/169469912-05e65439-e3c1-45dd-a071-5be57ff2d794.png)

### Real (5,-3)
Result (6,-3). This result is also slightly different from the expected value. Once again, my robot didn't behave smoothly at this point. This means localization might be difficult in lab13, since I would need to consistently turn 360 degrees no matter what point the robot is at in the map. 

<img width="792" alt="Screen Shot 2022-05-19 at 2 46 44 PM" src="https://user-images.githubusercontent.com/71809396/169470016-28daff87-e1ae-4800-998b-32e5bc1d5323.png">

![5-3](https://user-images.githubusercontent.com/71809396/169471798-34523387-8b57-4054-96a9-cee32129565a.png)


The robot localized slightly better in positions (-3,-2) and (0,3) than the other two positions. For the (5,3) and (5,-3) positions, I think it's more difficult for the robot to determine its position since the environments are similar when in these two positions. Just look at the polar plots for these positions, and rotate one to be on top of the other. The difference between the two is very difficult to identify, which is why the robot has trouble in these spots (it also didn't help that there was underrotation when localizing at these points). A room with more distinct features/landmarks will likely give more accurate localization results.


Here's a quick video of my robot performing real localization. It's not in the map because lab hours are packed and the map is in high demand, but you can see that it works well. Ignore my reflection in the computer screen. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/o97fTZqnQxY" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
