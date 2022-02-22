# Characterizing the Cyclone Stunt Car
The purpose of this lab is to characterize the Cyclone stunt car's abilities and characteristics. This will allow me to compare the modified car I make with the remote control car that comes in the package. 

## Measurements
### Dimensions, weight
I chose to not measure the weight of the car because it will change once I add and remove parts; however, the final weight will be important when determining how fast it will drive, as well as how much current the batteries need to supply (a heavier car = more current draw = shorter battery life). The car is approximately 14 cm by 18cm. These values will be useful later, when I will need to map rooms and have an understanding of the robot's profile. The height of the robot won't be useful for mapping,so I didn't measure it. 
### Battery life
I checked the battery at full charge with a multimeter. It's labeled as a 3.7V battery, but was actually at 3.8V. I dropped the battery down to 2.98V, at which point
the motor drivers stopped functioning. The wheels would act as if they were stuck. Starting from 2.98V, the battery was charged back to 3.8V in about eight minutes. 
I'm not sure if it's possible that the battery isn't fully charged once it reaches its peak voltage, because some charges seemed to last longer (10 minutes of driving) than others (3-4 minutes of driving). As far as I know, battery charge is at 100% once the max voltage is reached. This is something I'll have to confirm, because the disparity in battery output is as yet unexplained. 
### Max speed/acceleration
I unfortunately had an issue with bluetooth, so my IMU measurements for acceleration are pending; however, I measured an average max velocity  
as 2.36 m/s. This was calculated by starting the car from a stopped position for 3 meters. The car took 1.27 seconds to drive this distance, but I believe this result could be improved on a grippier floor surface. The video is shown below. 

<iframe width="560" height="315" src="https://youtu.be/B3_lD5pFFHM" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

### Control/tricks
I noticed that the robot handles differently depending on the surface and battery level. On grippy surfaces with high charge, the robot can get incredibly fast. I look forward to measuring the max acceleration of the robot, because it is powerful enough to flip and leave the ground. On less grippy surfaces, the robot is slower due to slippage. Cleaning dust and dirt off the wheels helps it gain traction on smooth floors. I was very happy to note that the robot could turn in place with a high degree of accuracy, as you can see in the following video. Controlling the robot with the remote is difficult, but the actual behavior of the robot is encouraging since it remains in the same position when turning 360 degrees. This will help the robot effectively map the room it is in once the ToF sensors are mounted. 

<iframe width="560" height="315" src="https://youtu.be/ALRNZdU9o0A" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

### Observations
My main observation is that this really is a fast robot. Mapping and localization at this speed is going to be nearly impossible, so I expect to slow the robot down during these events. In addition, at high speed the robot is subject to a lot of slipping and can even leave the ground when the batteries are fresh enough. Controlling the robot is extremely challenging. The remote control only allows for impulses to the motor controller, but these impulses are not fine enough to move the robot smoothly. My motor drivers will need to be able to more precisely control the robot if I want it to not jerk and turn too far. The following video shows how challenging it is to drive the robot around a square (marked out on the floor with blue tape). 

<iframe width="560" height="315" src="https://youtu.be/Kz4Bg6w9VNY" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
