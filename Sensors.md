# Using ToF and IMU Sensors

## ToF

The first part of the lab involved connecting the Time-of-Flight sensors to the Artemis via I2C. This worked well for one sensor, 
but adding a second sensor didn't quite work. 
<img width="505" alt="Tof Address" src="https://user-images.githubusercontent.com/71809396/153962058-deaaec15-35ab-4c47-bf01-8d0f43eaa6ab.png">
Above is the serial monitor output with one sensor connected. Below is the output with two sensors. 
<img width="464" alt="Screen Shot 2022-02-14 at 6 14 33 PM" src="https://user-images.githubusercontent.com/71809396/153962771-38c50fff-1054-4eb4-90d1-71ba050fe501.png">
As you can see, when two sensors are connected, a slave is detected at each address possible. Obviously, I don't have 127 slave devices 
connected to my Artemis (only 2), so there's an issue with the provided I2C code that needs to be addressed for future labs. 

To test the efficacy of the ToF sensors, I used a tape measure and measured the sensor distance vs the actual distance. Unfortunately, the only tape measure
I could find used Imperial units, so my measurements are in inches instead of cm. 
![ToF Sensor vs Actual Distance](https://user-images.githubusercontent.com/71809396/153963669-bb330f50-f868-41de-a886-3a806ead2cc7.png)
As you can see, the sensor is extremely accurate up to the range I measured, which was 60 inches. I stopped at this distance because the table I was measuring on
was only five feet long; however, its precision throughout the entire measured range makes me confident that the ToF sensors can accurately calculate distances 
greater than 60 inches. You can see the speed at which the sensor updates in the video below. It's pretty responsive to change in distance, as you can hopefully
see in the video. 
*insert youtube link to Single Tof*

Here's a video of both ToF sensors connected to the Artemis with ToF. I didn't notice any variation in accuracy when detecting different materials (glass, metal, cardboard),
so the ToF sensor seems to work equally well when ranging any of these materials. The color/texture inconsequentiality of the ToF sensor is discussed <a href="https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwiLqb-hroD2AhVYkIkEHculACMQFnoECAUQAw&url=https%3A%2F%2Fwww.ti.com%2Flit%2Fwp%2Fsloa190b%2Fsloa190b.pdf&usg=AOvVaw3QxOdhjqmPfI1o3Byg2_Or"> here </a>

*insert second youtube link*

## IMU









