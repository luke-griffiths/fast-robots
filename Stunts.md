# Lab 8

The purpose of this lab is to use the control methods used in previous labs to drive a car toward a wall, flip when 0.5m from the wall, and drive back in the direction it came. Unfortunately, my car has been plagued with mechanical issues which has prevented me from completing the stunt. I will still detail how the stunt could be implemented on a working car, as well as my thorough testing process. 

### Hardware

![IMG_6453](https://user-images.githubusercontent.com/71809396/164754864-a224776a-2f57-4c0e-926b-4fa36a3728e7.jpg)

When I first tried to execute the stunt, my car overshot the 0.5m mark and slammed into the wall. This caused one of the wires that were soldered to the motor controller boards to come loose. I decided to use male header pins to better secure the wires to the board. Since the wires themselves had female ends, there was no risk that they would short if a wire came loose. I also covered the back side of each motor driver and the Artemis with electrical tape to prevent shorts. The other pieces of hardware I used were a single forward-facing ToF sensor and an IMU. 

### Control

In previous labs I used PID to control the robot; however, PID responds proportionally to the error between the robot's current position and the setpoint. This meant that the robot would slow down as it approached the 0.5m mark, which is not ideal for a flipping stunt. Instead, I wanted the robot to drive full speed until it reached the setpoint, then drive full speed in the opposite direction. The sudden drastic change should be enough to flip the car, provided the tires can gain enough traction on the floor. 

The simplest way to implement this lab would be to have the car drive forward at full speed toward the wall, then send a command to the robot to change direction; however, this would not require the ToF sensor or IMU and could be accomplished with any regular remote control car. A better alternative would be to use the Kalman filter from lab7 to allow the car to get as close to the 0.5m mark as possible. The Kalman filter is needed because the ToF sensor can update only once every ~40ms, in which time the car can move up to 10cm (the car's max speed, 2.4m/s * 40 ms). By using a Kalman filter we can combine a motion model with the ToF sensor data to more accurately flip. I would use the exact same implementation of the Kalman filter from lab7. My implementation used a global boolean variable "reachedSetPoint" which was set to true as soon as the Kalman filter output reached 0.5m. If reachedSetPoint is false, the car would drive full-speed forward else it would drive full-speed in reverse. The implementation is below. 

<img width="677" alt="Screen Shot 2022-04-22 at 2 10 27 PM" src="https://user-images.githubusercontent.com/71809396/164770938-8c5dd4da-9a3f-4b5b-a655-3dfa5aef65cf.png">

The implementation of kfFunction is shown below. It uses the same Kalman filter as lab7, except it changes the value of reachedSetPoint to true if the filter output is less than 0.5m. 

<img width="416" alt="Screen Shot 2022-04-25 at 12 22 18 AM" src="https://user-images.githubusercontent.com/71809396/165020573-208fe402-1c40-4d5d-8873-422262b475d0.png">

I believe this implementation should result in a successful flip. Unfortunately, my car would not allow me to test out the stunt. 
## The Issue
I started to notice strange behavior when doing trial runs of the stunt. The motors made a high-pitched whine, and the car seemed to start and stop as if there was a loose power connection. In addition, one motor seemed to work better than the other, allowing the robot to spin in place sometimes; however, both motors were somewhat unreliable. I used a multimeter to check the battery and the wires connecting the battery to the motor controllers. Both were ok. I then looked at my motor controllers themselves. I wasn't too happy with the soldering job, so I decided to get new motor controllers and resolder them more carefully. This ensured that if I had somehow shorted a motor controller, it would be fixed. This also didn't solve the issue. I then started a more thorough debugging process.
### Checking for shorts
I used a multimeter to check for shorts among all the pins on the Artemis and both motor controllers. I found no shorts. You can see in the video my momentary confusion when the multimeter beeped on the connections between AISEN/BISEN and GND, but after consulting with the datasheet I found that these are intended to be connected. The video only shows me checking for any shorts to GND, I did the same for all pins and for the Artemis as well. 

* insert video of short checking here

### Checking the PWM signals
I wanted to make sure the PWM signals being given to the motor controller were accurate and reliable. I connected each pin (A0-A3) to the oscope and checked them at 25% duty cycle and 75% duty cycle. I found no difference among different pins, and both duty cycles looked as I'd expect them to. Here's a video of the oscope output for one of the pins at 25% duty cycle. 

* insert video of 25% duty cycle

### Ensuring motor controllers output correct voltage
I then decided to check the DC voltage output by the motor controllers themselves. I connected the motor controllers to a power supply, set it to 3.7V (about what the battery voltage is) and checked the voltages at both 50% duty cycle and 100% duty cycle. I got the following values:
 * MC#1: 50%-> 2.01V :: 100%-> 3.67V
 * MC#2: 50%->2.00V :: 100%-> 3.67V
These values looked pretty reasonable to me, since doubling the duty cycle nearly doubled the output voltage. In addition, the numbers are consistent across motor controllers. I then removed the power supply and connected the robot to the battery. I performed the same test, and got nearly identical numbers with the battery as I had with the power supply. This made me pretty confident in my motor controllers themselves, since nothing so far had indicated that they were broken. 


* insert image of power supply setup

My final check to make sure the motor controllers weren't the issue was to use another brushed DC motor. I bought a hobby DC motor and connected it to one of the motor controllers and tested various duty cycles on it. As you can see in the video below, the hobby motor works perfectly when powered by the motor controllers and the Artemis. The tape on the end of the motor is just to make it clear that it's moving. Both motor controllers were connected to the new motor, and both worked reliably. 

* insert video of motor with tape

### My diagnosis
I believe the issue is with the brushed motors on the robot, though I'm still not sure what the issue is or how it may have been caused. I've done a lot of testing with brushless motors but those techniques do not generally apply to brushed motors. I used a multimeter to check the resistance between the leads of the motors as well as the connectivity. Neither revealed a difference between the robot motors and my working hobby motor. At this point, I've run out of things to check. I'm very confident that my motor controllers and Artemis are working as intended. I am less confident in the motors themselves and the mechanical aspects/gears within the car itself. 

## Stunts...ish
Here is the only short clip I could get of a trial run for the car. This was when I was adjusting my code because it wasn't behaving properly. I later found it was a hardware issue and not a software issue.

* insert video of flip

I found that giving the car a push usually helped motor #1 spin somewhat reliably. I implemented a simple interrupt routine to change the direction of the car's travel every ~2.2 seconds. You can see the result of that below. With only one motor working, it made the car do donuts. This was the best open-loop "stunt" I could complete with my broken motors. 

* insert spin video

## Conclusion
I'm disappointed that I couldn't get the car to perform a stunt; however, I'm hopeful that I can still get a reasonable grade for this lab due to the amount of time, effort, and hardware debugging I've put in. This lab has been a hardware headache, but I think spending hours resoldering leads and debugging code/electrical signals is an important aspect of this class. After all, the goal of this class isn't to make a robot do a flip. Rather, the goal is to learn techniques for robotic control while gaining valuable problem-solving/hardware debugging skills.

