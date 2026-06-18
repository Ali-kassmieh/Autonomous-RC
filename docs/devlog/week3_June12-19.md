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
  - **Notes**: break statement can't be used so it was removed; video shows that >90 is right and <90 is left
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
  - For the saftey of the rover I will cut off steering angle at 30 degrees in both direction
  - I will move the rovers wheels by 5 degrees each time
  - I then sent the video to Gemini AI and it made a chart of each angle.
 
#### right max angle test

| Servo Input Angle | Measured Wheel Angle | Step Delta |
| :---: | :---: | :---: |
| **0°** | **0.0°** | — |
| **5°** | **2.2°** | +2.2° |
| **10°** | **4.6°** | +2.4° |
| **15°** | **7.3°** | +2.7° |
| **20°** | **10.1°** | +2.8° |
| **25°** | **12.8°** | +2.7° |
| **30°** | **15.4°** | +2.6° |

#### left max angle test

| Servo Input Angle | Measured Wheel Angle | Step Delta |
| :---: | :---: | :---: |
| **0°** | **0.0°** | — |
| **5°** | **2.4°** | +2.4° |
| **10°** | **4.9°** | +2.5° |
| **15°** | **7.7°** | +2.8° |
| **20°** | **10.6°** | +2.9° |
| **25°** | **13.3°** | +2.7° |
| **30°** | **16.1°** | +2.8° |

-Taking the Average Rate of Change of each we get this univeral equation

### Universal Steering Conversion Equation

* **For Right Turns** ($\theta_s \ge 0$):  
  $$\theta_w = 0.513 \cdot \theta_s$$

* **For Left Turns** ($\theta_s < 0$):  
  $$\theta_w = 0.537 \cdot \theta_s$$

---

### Variable Definitions

* **$\theta_w$ (Wheel Angle):** The actual geometric steering angle of the physical wheels relative to the straight-ahead centerline ($0^\circ$). A positive result indicates a right-hand turn, while a negative result indicates a left-hand turn.
* **$\theta_s$ (Servo Input Angle):** The commanded rotation angle sent to the servo motor relative to its center position ($0^\circ$). Positive inputs ($>0^\circ$) drive the car to the right, and negative inputs ($<0^\circ$) drive it to the left.

### Variable Definitions

* **$\theta_w$ (Wheel Angle):** The actual geometric steering angle of the physical wheels relative to the straight-ahead centerline ($0^\circ$). A positive result indicates a right-hand turn, while a negative result indicates a left-hand turn.
* **$\theta_s$ (Servo Input Angle):** The commanded rotation angle sent to the servo motor relative to its center position ($0^\circ$). In this system, positive inputs ($>0^\circ$) drive the steering linkage to the right, and negative inputs ($<0^\circ$) drive it to the left.
  ### Issue 1
- I need to actually drive the thing, however my axle from before has fallen off and I need to fix it
- I bought a few o-rings and used them to cover the gap
# June 16, 2026
## Goals:
- Understand PID controls since they are used for most controllers in industrial applications
## Notes on PID control
- In an open loop system, the robot is told to do one thing (ie move forward one meter)
    - However, it cannot adjust itself based on unknowns, such as dirt in the wheels or a faster than usual robot
- PID (Proportional integral derivitive) on the other hand is a closed loop system, it feeds the output back into the input and measures the error between the starting state and goal state
- The goal is to convert this error into commands in a way where the error eventually reaches 0/goal is reached
- With the acronym, the output is the sum of the error term times a constant kp, the error multiplied my a constant Ki then integrated, and the derivitive of constant Kd and the error 
  - These k-terms are called gains and they refer to how strongly the system should react to different forms of errors
      - **Kp * e(t)**: This is the proportional term. It reacts to the current error. For example, if the car is 10° off its desired heading, the controller applies a steering correction proportional to that 10° error.
      - **Ki * ∫e(t)dt**: This is the integral term. It reacts to accumulated error over time. P only reacts to the current issue. However there may be a constant underlying issue that is causing a persistant error. In that case, I will keep note of this constant error and add extra correction until the error is resolved
      - **Kd * de/dt** This is the derivative term. It monitors how quickly the error is changing. If the error is being resolved very quickly, it reduces the correction to avoid overshooting the target. If the error is increasing quickly, it increases the correction to counteract the worsening error. It essentially acts as a shock absorber for the controller.
  - To summarize, P changes the current error, I adjusts for persistant error, and D makes sure we don't bounce back and force around the target
## How to implement this into my code
- I think what I will do is make a PI system for the servo and a D system for the motor. It will turn to its designated header angle and check for any reoccuring errors while turning, however the motor will use a d function of dt to slow down as the error becomes smaller
- The header will be the yaw of the robot and the calculates simple, just P = pi * error; I += ki * error * dt; D = (ki * (error-lastError)/dt)*motorSpeed
# June 18, 2026
## Goals
- Install encoder onto rover
- Test PID system from yesterday
## Log 1: Encoder installation
### Notes
- I am using the AS5600 magnetic encoder. It requires both the sensing chip and magnet to be at most 3mm apart with 1.0mm being reccomended
- The issue is I cannot install the sensor inside the drivetrain as the PCB is too big and there are too many metalic parts in one small area, potentially causing the magnet to slip off.
- I can put it under the dogbone, however that would result in a 10mm space
- I can't glue it on top of the chip as direct magnetic contact can ruin the reading
- The best thing is to 3D model a mount to be put ontop the PCB with a slot in the middle for the magnet. It will be shaped like an I to make room for the header pins
  - **Important measurment**
  - 20mm from hole to hole
  - Mounting hole is 3.6mm
  - Chip is 1.7mm
  - Magnet is 3mm in diameter
### Design 1:
- This is a basic I design with a holder in the middle for the magnet
    - [AS5600 mount try 1](https://drive.google.com/file/d/1nssDZpm3eW2xAHSCkljeuwxwtecOYRUB/view?usp=drive_link)
