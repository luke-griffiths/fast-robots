### A Brief Introduction to the Artemis Nano Board
This simple lab involved downloading the Arduino IDE and uploading a few pre-written test scripts to the Artemis Nano board. 
# Blinking the Onboard LED
The first test script simply blinked the onboard LED. You'll notice in the video that the LED remains on for three seconds, then turns off for one. The default code for the blink script has the LED turn on and off for one second each; however, I noticed that the Artemis's LED was blinking at this rate before I had even uploaded the blink script to the board. It's likely that the Artemis board was shipped with the blink code pre-uploaded, so I changed the blink script to be three seconds on, one second off to ensure the script worked properly. 

https://user-images.githubusercontent.com/71809396/151715278-307cd7ea-559f-42ed-a364-c2c899c01adc.mov

# Mic Check
The next test script for the Artemis board was Arduino's MicrophoneOutput script. This script prints out the detected frequency coming from the microphone to the serial monitor. You can see the result of whistling on the microphone's detected frequency below. 

<img width="322" alt="Screen Shot 2022-01-30 at 2 41 35 PM" src="https://user-images.githubusercontent.com/71809396/151715231-605a94f9-0277-4367-b3f1-0d84b33736d1.png">

# Printing to the Serial Monitor
The Serial test script ensures the serial monitor works, and that messages can be sent and received between my computer and the Artemis. This is a simple UART script, which you can see at work in the video below. 

https://user-images.githubusercontent.com/71809396/151715232-2427c568-e7c8-4458-b76b-cd248d59813a.mov

# Internal Measurements
The final test script is the analogRead script. This script reads the values of   
  - the internal die temperature
  - the internal VCC voltage
  - the internal VSS voltage
and prints their values to the serial monitor. 

<img width="910" alt="Screen Shot 2022-01-30 at 2 44 34 PM" src="https://user-images.githubusercontent.com/71809396/151715230-f8ebf86d-8960-476a-9f5d-ffb338d9f99c.png">

