# June 14 2026
## Goals
- Power on ESP via RC car
- Move motor with ESP
- Connect servo to ESP in order to turn the car
## Important note
  - After a week of attempting to desolder throughholes I have chosen to try again at a later date near a professional and buy a new ESP whose headers are not soldered as to not waste valuable time.
## Log 1: Power
### Try 1:
  - I have spliced a female wire to the male wire for both a 5V and GND in order to connect the power from the buck to the ESP, and it worked flawlessly. This allows the code to execute with the robot, eliminating lag.
    - [ESP is powered on by RC car](https://drive.google.com/file/d/1m3gU6e3gT_kMek5OZZbFTPobyGX79Qeu/view?usp=drive_link)
## Log 2: Power wheels
### Try 1:
  - Ran code from previous log file without altering it
  - Try failed, the serial port is not being read, maybe the board configuration was done incorrectly
      - [Running motor try 1](https://drive.google.com/file/d/1vUi3IwTjfsRy6KyLP9qBH4FcRZp4Uwap/view?usp=drive_link)
### Try 2:
  - Changed these settings:
    - | Setting          | Value                                |
| ---------------- | ------------------------------------ |
| Board            | ESP32S3 Dev Module                   |
| USB CDC On Boot  | Enabled                              |
| Flash Size       | 16MB                                 |
| PSRAM            | OPI PSRAM (or Enabled)               |
| Upload Speed     | 921600 (or 460800 if unstable)       |
| Partition Scheme | Default 8MB with spiffs (or default) |
| Port             | COM 7                                |
- Try failed, same thing. Maybe this USB C is charge only
- Also I failed to connect data pin, only GND and 5V were attached
### Try 3:
  - Settings from before unchanged, USB-C port changed
  - Try failed, I need to verify that the USB works
      - [Running motor try 3](https://drive.google.com/file/d/1OnLAtSFd7pJiGgBTQFBYWPpZwHUfpsNA/view?usp=drive_link)
  - By plugging my phone into my PC, I verify that this is a transfering USB cable
### Try 4:
  - Rebotted ESP
  - The code has uploaded and the motor runs, but the wheels won't move
      - [Wheel's wont run](https://drive.google.com/file/d/1nYilarqoGKA7xZtTdCusuBla2_7aNoTo/view?usp=drive_link)
### Try 5
  - Opened differential cover, gears were not connected
      - [Bad gear connection](https://drive.google.com/file/d/1GKDmwgQOor4cDD_ReAj1IwnqIgu5BLTh/view?usp=drive_link)
      - [Fixed gear connection](https://drive.google.com/file/d/1POKuk53iQw5wsmc3ZTg9TavO7AL8pmbY/view?usp=drive_link)
      - [Mostly successful run](https://drive.google.com/file/d/1nrzQ8fd0h0CVtC9r4v0YI_7si10cXEF6/view?usp=drive_link)
## Log 3: Servo
### Try 1:
  - Changed code to include servo and set it's position to 90
  - Since my motor is 180, by setting neutral at 90 degrees moving the servo <90 and >90 will turn the rover different directions
    ```cpp
      enum class Turn {
      Left,
      Right
      };
    
      Servo turnServo;
      int turnNeutral = 90;
    
      void Turning(Turn Direction, int turnAngle);
    ```
      - These are the changes to the .h file
    ```cpp
      //Initializes movement as neutral
      void ESC_CONTROLLER::initAsNeutral(){
        esc.writeMicroseconds(neutralMicroseconds);
        turnServo.write(turnNeutral);
        delay(restTime);
      }
      
      //Servo turning logic
      void ESC_CONTROLLER::Turning(Turn direction, int angle){
        turnServo.write(angle);
        Serial.println("Servo angle: ");
        Serial.println(angle);
      }
    ```
      - These are changes to the cpp file
      - Logic has yet to be coded in
    - The servo has moved to its neutral position and the turn stick has been installed correctly
        - [Turning rod centered](https://drive.google.com/file/d/1GI6HSlNoDsrrXENKdQ9Gr1zQJ_ME4F9S/view?usp=drive_link)
# June 15, 2026
## Goals:
  - Complete steering logic
  - Figure out encoders
  - Drive the rover one foot, turn left, drive 6 iches, turn right, then reverse six inches
## Log 1: Complete steering logic
### Try one
  - I will set the code to move the servo at 80 and 100 degrees
    ```cpp
      mainESC.Turning(Turn::Left, 80);
     mainESC.Turning(Turn::Left, 100);
    ```
  - **Notes**: Need to add delay to turning function and break at the end of the main loop
      - [Turning test try 1](https://drive.google.com/file/d/1FKzO4u9u99CTPYs-Lia31mYaci5TXr5X/view?usp=drive_link)
### Try two
  - Code has been updated with the previous notes
  - **Notes**: break statement can't be used so it was removed; video shows that <90 is right and >90 is left
      - [Turning test try 2](https://drive.google.com/file/d/1dSEC8vdrTq2DU7rf9UlD6RBa1D8ENo3t/view?usp=drive_link)
    
### Try three
  - Turn function has updated logic to make it more readable and understandable
    ```cpp
      //Servo turning logic
      void ESC_CONTROLLER::Turning(Turn direction, int angle){
        int turnAngle;
        switch(direction){
          case Turn::Right:
            turnAngle = turnNeutral - angle;
            break;
          case Turn::Left:
            turnAngle = turnNeutral + angle;
            break;
        }
        turnServo.write(turnAngle);
        Serial.print("Turn ");
        Serial.print(direction);
        Serial.print(" at ");
        Serial.print(angle);
        Serial.print(" degrees");
      }
    ```
- Success
  - [Updated turning](https://drive.google.com/file/d/1fbrsKT01M-7IIJK3n-BFmdFYB5H9_Tj1/view?usp=drive_link)
  ## Log 2: Experiment with turning angle
  ### Try 1
    - **Notes**: I assume that the wheels and servo are not 1:1, that is to say turning the servo one degree will not turn the wheels one degree. The only thing to do is experiment
