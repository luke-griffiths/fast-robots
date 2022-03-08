# Motor Control
For this lab, the objective was to wire up the motor drivers and demonstrate open-loop control. The motor driver used for this lab (and for all subsequent labs) 
is the DRV8833 Dual Motor Driver. This driver has an operating voltage of 2.7-10.8V, so when testing on the power supply I ran the motors at 6V. 
Because the RC car's motors require a lot of current, I doubled up the motor drivers and made each motor driver chip power one motor. This allowed the max current to 
be 2.4A instead of 1.2A; however, the higher current means that I needed to be careful about the wire gauge. The jumper wires that were fine to use for the IMU and ToF
sensors can't handle more than ~1A, so I used higher gauge wire for the motor driver output and power(I just recently watched someone learn this lesson and blow a jumper wire,
so I was sure not to make the same mistake). For the signal wires coming from the Artemis, I used normal jumper wires since the current is guaranteed to be low. 

![IMG_6222](https://user-images.githubusercontent.com/71809396/157168254-4570de25-3a12-4178-8536-3954c25624fe.jpg)

I plan on keeping both batteries in the battery compartment of the car, with the Artemis, IMU, and motor drivers where they are in the image above. The ToF sensors will 
be mounted on the front and right sides of the car.

## PWM Duty Cycle
I ran the motors at a 10% duty cycle and a 20% duty cycle. Because the wheels were suspended in the air and had little resistance, they were scarily fast 
so I didn't exceed 20%. 
