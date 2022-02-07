# Setting Up Bluetooth 
To build the next part of the robot, I needed to enable communication between my computer and the robot itself. This was achieved via a bluetooth connection 
between the Artemis board and the built-in bluetooth on my Mac. For the Artemis, bluetooth was enabled with the ArduinoBLE (Bluetooth Low Energy) library. 
For the Mac, commands were run in python via a Jupyter notebook. Although the ArduinoBLE limits the maximum message size to 255 bytes, for the purposes of ECE 4960, 
the max will be 150 bytes. 

## Task 1: Sending Three Float Values
The first task was to write the Arduino code for a SEND_THREE_FLOATS command. This would enable the Artemis to receive three float values (separated by |). 
To ensure the Artemis received the three values, all three were printed to the serial monitor. 
<img width="491" alt="Screen Shot 2022-02-06 at 2 42 30 PM" src="https://user-images.githubusercontent.com/71809396/152830337-7488a7f7-a518-450b-a4a9-3a97e2102844.png">

As you can see from the code for SEND_THREE_FLOATS, the command lies within an Arduino switch statement. The switch case is determined by the command type sent from the 
base computer (my Mac). Some possible commands are SEND_TWO_INTS, SEND_THREE_FLOATS, PING, and ECHO. 

## Task 2: Echoing Data Sent from the Mac
The purpose of this task is to use the ECHO command on the Mac, which sends a string to the Artemis. The Artemis receives the data, adds " :)" to it, then sends it back to the computer. The code and a demonstration are below. As you can see, the Mac sent "Hi" to the Artemis, which sent back "Hi :)".


<img width="587" alt="Screen Shot 2022-02-07 at 4 49 12 PM" src="https://user-images.githubusercontent.com/71809396/152879046-9a25e09a-471c-4a63-a5b9-c038b9be6a4b.png">


![ezgif com-gif-maker](https://user-images.githubusercontent.com/71809396/152879233-cbd8099b-a65d-44ea-9b68-a9867e3791f4.gif)


## Task 3: Python Notification Handler
<img width="746" alt="Screen Shot 2022-02-07 at 11 18 37 AM" src="https://user-images.githubusercontent.com/71809396/152832859-a76ae3f2-34ed-4a38-8e1d-3974f03ddfe3.png">
This task required code in the Jupyter Notebook, not for the Artemis. I created a global variable called float_value in the Notebook, which was updated in a function
I defined called notification_handler. This function took a UUID and an array of bytes as paramaters. For now, the only notifications it handles are of type float, so 
the UUID isn't used in my code; however, if I want to be able to handle other data types in the future, the UUID will come into play. Notification_handler converts the 
array of bytes into a float and updates the global float_value to this value. To test my function (and ensure the global variable was updated) I used a while loop that 
printed the value of the global variable every second. 

![ezgif com-gif-maker](https://user-images.githubusercontent.com/71809396/152833559-abe95d20-1e3c-49f3-b749-d36cbd01a82a.gif)

## Task 4: String vs Float
Sending less (more compact) data is more efficient than sending longer sequences of bits. We want our robot to transfer data to the computer as efficiently as 
possibly. When choosing between sending a float from the Artemis to the Mac as a float or instead sending it as a string and then converting the string to
a float, it would only make sense to send the float value as a string if the string would be less than 4 characters. This is because an ASCII character is 1 byte, 
and a float is 4 bytes. So if the data we want to send is small enough to be encoded in three or fewer characters, I should send it as a string and then have 
the Mac convert the string to a float. If not, just send the float as is. 
