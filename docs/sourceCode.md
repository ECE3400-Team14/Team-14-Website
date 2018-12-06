# Source Code

You can find our team's complete robot github repository [here](https://github.com/ECE3400-Team14/3400), with the code used in each lab and milestone as well as the final competition. 

For detailed comments and descriptions of the main code on the competition robot: see the [Competition Robot](https://github.com/ECE3400-Team14/3400/tree/master/CompetitionRobot) repository:

- [CompletionRobot.ino](https://github.com/ECE3400-Team14/3400/blob/master/CompetitionRobot/CompetitionRobot.ino): The main `setup()` and `loop()` functions, as well as global fields used.
- [Dijkstra.ino](https://github.com/ECE3400-Team14/3400/blob/master/CompetitionRobot/Dijkstra.ino): Contains the logic for performing Dijkstra's algorithm to search for unexplored squares (main logic is in `dijkstra()`).
- [Maze_Data.ino](https://github.com/ECE3400-Team14/3400/blob/master/CompetitionRobot/Maze_Data.ino): Functions for reading and writing maze information to memory.
- [Movement.ino](https://github.com/ECE3400-Team14/3400/blob/master/CompetitionRobot/Movement.ino): Functions for moving the robot using line following and intersection detection. 
- [OV7670_SETUP.ino](https://github.com/ECE3400-Team14/3400/blob/master/CompetitionRobot/OV7670_SETUP.ino): Code that uses I2C to set ups the OV 7670 Camera.
- [Radio.ino](https://github.com/ECE3400-Team14/3400/blob/master/CompetitionRobot/Radio.ino): Code for setting up the Nordic nRF24L01+ radio and sending data from the robot's memory to the base station.
- [Search_Maze.ino](https://github.com/ECE3400-Team14/3400/blob/master/CompetitionRobot/Search_Maze.ino): Functions that help the robot make decision when searching for and moving to unexplored squares.
- [Sensors.ino](https://github.com/ECE3400-Team14/3400/blob/master/CompetitionRobot/Sensors.ino): Functions for reading from the line sensors and wall sensors on the robot. 
- [Shape_Detect.ino(https://github.com/ECE3400-Team14/3400/blob/master/CompetitionRobot/Shape_Detect.ino): Functions for receiving treasure detection information from the FPGA using our custom serial protocol. 
- [Signal_Processing.ino](https://github.com/ECE3400-Team14/3400/blob/master/CompetitionRobot/Signal_Processing.ino): Runs FFT on the audio/IR analog signal. 

The code for our base station can be found [here](https://github.com/ECE3400-Team14/3400/tree/master/Lab3_Base_Station/Base_Station_to_GUI).

[Return Home](./index.md)
