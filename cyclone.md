# Characterizing the Cyclone Stunt Car
*intro about why I'm doing this

## Measurements
### Dimensions, weight
I chose to not measure the weight of the car, despite the fact that it will be important when determining how fast it will drive later, 
as well as how much current the batteries need to supply (a heavier car = more current draw = shorter battery life). The car is approximately 14 cm by 18cm. 
These values will be useful later, when I will need to map rooms and have an understanding of the robot's profile. The height of the robot won't be useful for mapping,
so I didn't measure it. 
### Battery life
I checked the battery at full charge with a multimeter. It's labeled as a 3.7V battery, but was actually at 3.8V. I dropped the battery down to 2.98V, at which point
the motor drivers stopped functioning. The wheels would act as if they were stuck. Starting from 2.98V, the battery was charged back to 3.8V in about eight minutes. 
I'm not sure if it's possible that the battery isn't fully charged once it reaches its peak voltage, because some charges seemed to last longer (10 minutes of driving) 
than others (3-4 minutes of driving). 
### Max speed/acceleration
I unfortunately had an issue with bluetooth, so my IMU measurements for acceleration are pending; however, I measured an average velocity from stopped position 
as 2.36 m/s. The car took 1.27 seconds to drive three meters. The video is shown below. Clearly, the robot is a fast robot. 

### Control/tricks
Different surfaces, turning incrementally difficult, spinning in place is good. 
