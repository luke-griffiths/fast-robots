# Lab 9

The purpose of this lab is to implement a very simple PID to control the angular speed of the robot. This will allow me to capture distance measurements as the robot spins in place. 
Arrays of measurement data can then be combined to form a map of the robot's environment. 

## PID
For this lab, I exchanged my PID for a much simpler version. Previously, my implementation was 
```
PID(float setpoint, float currentPoint){
  //calculate the error
  float error = setpoint - currentPoint;
  //calculate terms
  float p = Kp * error;
  integral
  += error * dt:
  float i = Ki * integral;
  float d = Kd * (error - previousError)/dt:
  float output = p + i + d;
  //update previous error
  previousError = error:
  return output; 
}
```
which was then fed into a complicated clamping function to ensure the motor values didn't exceed 255 or fall below 0; however, since I just wanted to keep the robot spinning
in place at 20 degrees per second, I experimented and found that a PWM value of 75/255 was around 20 degrees/sec. I then used the value of 75 as my setpoint.
```
int setpoint = 75;
int simplePID(){
    //aim for 18 measurements per rotation
    int error = 20 - myICM.gyrZ(); //the axis (x,y,z) needs to be changed if the IMU orientation is changed
    int motorVal = Kp * error;

    analogWrite(0,0);
    analogWrite(1,setpoint + motorVal);//right wheel forward
    analogWrite(2,setpoint + motorVal);//left wheel backward
    analogWrite(3,0);

}
```
Below is a video of the car spinning in a circle at the setpoint of 75. It's spinning much faster than 20 degrees per second here because the surface is slippery, 
but you can clearly see that the robot stays roughly in the exact same position while spinning. If I wanted the car to slow down enough to be useful on this surface, I would need to make Kp much largeror adjust the setpoint.

<iframe width="560" height="315" src="https://www.youtube.com/embed/eMrYPoE4qGI" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

I then created an interrupt function so that I would only take a distance measurement every 20 degrees (1 second).
This made it much easier to deal with the data, and ensured that I only collected 18 data points per location. 
```
void sensorInterrupt{
    if (millis() - prevTime > 1000){
        distanceArray[i] = distanceSensor.getDistance();
        prevTime += 1000;
        i +=1;
    }
}
```
## Data Collection
I put the robot at position (5,-3) in the map and recorded its ToF sensor values as it spun in a circle. 
These values were stored in an array which I sent to my computer and processed in python. An example of the ToF readout is below

![image1](https://user-images.githubusercontent.com/71809396/168492131-6f198633-bbee-4500-8d63-1ef69a2b546c.png)

I then collected the sensor values for all five positions. The polar plots are below.

![pos53real](https://user-images.githubusercontent.com/71809396/168492190-528a1374-03b8-46c8-b82c-e1eb9ec8673c.png)

![polar03](https://user-images.githubusercontent.com/71809396/168492205-5befaaf4-feaf-41a9-8cb7-275b4b0aa0aa.png)

![polar00](https://user-images.githubusercontent.com/71809396/168492208-fd267823-6e02-4597-9e20-422ca8b73f88.png)

![polar-3-2](https://user-images.githubusercontent.com/71809396/168492214-db7234d9-a27d-42ac-8ee1-b1313d119bb3.png)

![polar5-3](https://user-images.githubusercontent.com/71809396/168492217-6bfecc5a-32e5-43d1-bbc3-662fd3f56935.png)

## Data processing
The next step in forming a map was to convert the polar coordinates of each ToF measurement into cartesian coordinates. 
To do this, I simply used x = (sensor measurement) * cos(theta) and y = (sensor measurement) * sin(theta). I then added or subtracted the offset of the position the measurements were taken at to get the position in the map. 
I used the following code to perform this sequence of operations:

```
for i in range(18):
    p1x[i] = np.cos(pol[i]) * point1[i] + 5 * 0.3 
    p1y[i] = np.sin(pol[i]) * point1[i] - 3 * 0.3
    p2x[i] = np.cos(pol[i]) * point2[i] + 5 * 0.3
    p2y[i] = np.sin(pol[i]) * point2[i] + 3 * 0.3
    p3x[i] = np.cos(pol[i]) * point3[i]
    p3y[i] = np.sin(pol[i]) * point3[i] + 3 * 0.3
    p4x[i] = np.cos(pol[i]) * point4[i]
    p4y[i] = np.sin(pol[i]) * point4[i]
    p5x[i] = np.cos(pol[i]) * point5[i] -3 * 0.3
    p5y[i] = np.sin(pol[i]) * point5[i] -2 * 0.3
```
As you can see, since point1 was (5,-3), I added 5 * 0.3 to the x coordinate of each distance measurement and subtracted 3 * 0.3 for each y coordinate of the distance measurement (the 0.3 is due to the fact that the positions are measured in feet and the ToF on the python script is using meters, so 0.3 m = 1 ft). 
This process allowed me to compute the map below, where each color refers to a different position's data.

![combined map](https://user-images.githubusercontent.com/71809396/168492463-0d5023df-2423-4dbe-af0b-c43124df1bd2.png)

There is clearly a lot of inaccuracy with the measurements, but you can still get a general idea of the room's layout. After examining this plot, 
I decided to adjust the polar coordinates of some of the positions to make a better map. For example, rather than using the assumed theta array containing [0,20,40,..,340] degree measurements, for the red data points I added 8 degrees to each measurement. I believe this is likely due to slight variation in the robot's initial orientation (I was eyeballing it when I set the robot down at each point. A few degrees of difference at the beginning could result in the shifted data points). This gave me the plot below, which I think looks much better than the original.

![combinedmapadjusted](https://user-images.githubusercontent.com/71809396/168492609-4e4289e0-62db-48e1-a47b-fa91ab8c00b1.png)

Finally, I plotted line boundaries that I thought best matched the data. 

![linesoverlay](https://user-images.githubusercontent.com/71809396/168492638-cd7e56f6-ada7-4caf-8a32-343b6141018f.png)

The coordinates for the environment are below. Since I converted everything in meters and the next lab requires mm, I can simply multiply each value by 1000 since I use np arrays. 
```
linex_wall = [-1.3, -1.3, 2.15, 2.15, -0.5, -0.5, -1.3]
liney_wall = [-0.15, -1.4, -1.4, 1.4, 1.4, -0.15, -0.15]
linex_box1 = [0.5, 1.3, 1.3, 0.5, 0.5]
liney_box1 = [-0.3, -0.3, 0.45, 0.45, -0.3]
linex_box2 = [-0.3, -0.3, 0.75, 0.75, -0.3]
liney_box2 = [-1.4, -0.9, -0.9, -1.4, -1.4]
```
## Improvements
To get a better map, I would need to take many more measurements. I didn't experiment with different time intervals between measurements since I was told 18 would be enough, but doubling the number of measurements seems reasonable and would still be feasible for the ToF sensor. As far as reliability of the measurements, there seems to be some difference across different trials at each position; however, by using the theta adjustment technique I talked about earlier, the data matches much better. This further supports my assumption that differences in trials is caused by slightly different initial orientations, or possibly slightly different behavior of the robot during each trial (such as slipping, getting stuck on something, etc). 







