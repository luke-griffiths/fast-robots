# Using ToF and IMU Sensors

## ToF

The first part of the lab involved connecting the Time-of-Flight sensors to the Artemis via I2C. This worked well for one sensor, 
but adding a second sensor didn't quite work. 

<img width="505" alt="Tof Address" src="https://user-images.githubusercontent.com/71809396/153962058-deaaec15-35ab-4c47-bf01-8d0f43eaa6ab.png">

As you can see, a device is detected at 0x29. But this isn't the sensor's address! The address is actually 0x52, but this is right shifted by one bit which becomes 0x29. Next, I daisy-chained two ToF sensors and tried to read their addresses. Below is the output with two sensors. 

<img width="464" alt="Screen Shot 2022-02-14 at 6 14 33 PM" src="https://user-images.githubusercontent.com/71809396/153962771-38c50fff-1054-4eb4-90d1-71ba050fe501.png">

As you can see, when two sensors are connected, a slave is detected at each address possible. Obviously, I don't have 127 slave devices 
connected to my Artemis (only 2). The issue is that both of the two ToF sensors come with an identical address. This causes a bug on the Artemis that displays 127 slave devices. To fix this issue, the XSHUT pin on one sensor needs to be held low to disable the sensor. Then the address of the remaining sensor can be changed to 0x54 by calling
```
setI2CAddress((uint8_t)0x54)
```
Finally the first sensor can be re-enabled (by letting XSHUT be HIGH) and now both sensors are working. 

These ToF sensors have three software-defined modes:
* short < 1.3 meters
* medium < 3m
* long  < 4m

Because the long mode is most impacted by ambient light AND takes longer to range, I plan on using either the short or medium modes for the duration of this course. Realistically, the medium range sensor will be perfect for driving a fast robot, since it will read quickly enough and measure far enough to prevent the car from colliding with walls and other objects. 

To test the efficacy of the ToF sensors, I used a tape measure and measured the sensor distance vs the actual distance. I measured every inch between 1 and 24, then every foot after that. I converted the values to mm in code. 

![IMG_6770](https://user-images.githubusercontent.com/71809396/169631481-6845db20-9d7a-4b19-a831-5273150daeb3.jpg)

![figure_1](https://user-images.githubusercontent.com/71809396/169631491-095b6628-b5e5-462e-a8b4-8d35da8ee409.png)


As you can see, the sensor is extremely accurate up to the range I measured, which was 1.75m. At this distance, the sensor seemed to pick up the slight curve in the table, and distance measurements seemed to max out at around 1.75m. If I untaped the sensor from the table it again read correctly, but since I had to aim it at a small target the values fluctuated too much and I excluded these measurements. 


I used the following code to record the measurement time for the sensor. It records the time when the sensor starts ranging, and again once the sensor has a result back. 
```
 distanceSensor.startRanging(); //Write configuration bytes to initiate measurement
  int curr = millis();
  while (!distanceSensor.checkForDataReady())
  {
    delay(1);
  }
  int post = millis();
  Serial.print(" Wait time was (in ms): ");
  Serial.println(post - curr);
```
When using distanceSensor.setDistanceModeShort(), the average delay was about 49 ms. With distanceSensor.setDistanceModeLong(), it was 94ms.
I noted that the time it took to get a reading depended slightly on the distance to the nearest object, but median number was 94 ms with range 92-96 ms. 


https://user-images.githubusercontent.com/71809396/169632830-6d41ee2a-231a-420a-8e99-b9c309f91c77.mov



Here's a video of both ToF sensors connected to the Artemis with ToF. I didn't notice any variation in accuracy when detecting different materials (glass, metal, cardboard),
so the ToF sensor seems to work equally well when ranging any of these materials. The color/texture inconsequentiality of the ToF sensor is discussed <a href="https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwiLqb-hroD2AhVYkIkEHculACMQFnoECAUQAw&url=https%3A%2F%2Fwww.ti.com%2Flit%2Fwp%2Fsloa190b%2Fsloa190b.pdf&usg=AOvVaw3QxOdhjqmPfI1o3Byg2_Or"> here </a>

<iframe width="560" height="315" src="https://www.youtube.com/embed/0IgsaXmH2HE" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>


## IMU

To setup the IMU, I once again used the I2C code. This time the device was detected, and the address that was preloaded is 0x68.

<img width="458" alt="Screen Shot 2022-02-14 at 6 45 15 PM" src="https://user-images.githubusercontent.com/71809396/153965847-c82f61f9-12fa-48e6-9874-83f1286da9c9.png">

To test the IMU, I ran the provided code after making one change: the AD0_VAL needed to be set to 0, because the last bit of the IMU's I2C address is a 0. 
Note that any odd numbered I2C addresses need AD0_VAL to be 1, and any even addresses need it to be 0. I then tested the IMU's ability to capture pitch. I started in the -90 degree position, then changed to 0 degrees, followed by 90 degrees. For some reason, the IMU registered -90 as a large positive value rather than the -pi/2 radians I was expecting. I don't believe this is an issue with my code, but I need to get access to a second IMU to determine if there's a defect with the one I'm using. 

<img width="435" alt="Screen Shot 2022-02-14 at 11 27 04 PM" src="https://user-images.githubusercontent.com/71809396/153992615-be887daf-1aa5-4f7c-b3d1-2581532bf2c5.png">








