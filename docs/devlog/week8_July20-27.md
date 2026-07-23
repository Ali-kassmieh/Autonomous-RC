# July 21
- My brother visited us for two weeks which caused me to miss week 6 and 7, but now that im back I need to finish this project before August 11
## Goals
- Calculate yaw
- Create a prototype PID system
## Log 1: Issue with BNO08x
- The BNO08X is not working how I need it to, it is not initializing.
- I have narrowed it down to three possibilities: Power is not reaching the board, the SDA and SCL pins have been called incorrectly, or my programming is wrong
### Issue 1: Power
- A multimeter immedietly disproves this, 3.3V is coming out of the microboard
### Issue 2: SDA and SCL
- Upon further research, SDA and SCL pins are very flexible as long as they are called within the Wire.begin(SDA_PIN, SCL_PIN). It would be harder to mess this up than get it correct
### Issue 3: Code
- I have been having a hard time finding a reliable tutorial to program this, but I think I found one: How2Electronics High-Accuracy Pitch, Roll, Yaw with ESP32 & BNO08x IMU.
- My wiring is correct but according to this tutorial my code is vastly overcomplicated, and I have been using the wrong libraries.
- Here are the libraries I should have been using
``` cpp
#include "SparkFun_BNO080_Adruino_Library.h"
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
```
- *Note*: The sparkfun library was downloaded from github and moved to the library folder
- And rather than using the rotational vector equation shown previously, I simply need three lines
```cpp
Serial.println("Started calculating yaw");
    Serial.println("Started calculating yaw");
    //Converts from radiam to degree
    float yaw = bno08x.getYaw() * 180.0f / PI;
    return yaw;
  }
```
- As for initialization, I can now remove the part which initializes the event/report of the rotation vector and rather than check if the I2C has begun, I can check whether data has become avaliable. This changes the code from this
```cpp
void ESC_CONTROLLER::initialize(int ESCPin, int turnServoPin, int SDA, int SCL){
  esc.attach(ESCPin);
  turnServo.attach(turnServoPin);
  ESC_CONTROLLER::initAsNeutral();

  Serial.println("Initializing I2C...");
  Wire.begin(SDA,SCL);

  Serial.println("Initializing bno08x...");
  if(!bno08x.begin_I2C()){
    Serial.println("bno08x failed to initialize");
    while(1){
      delay(10);
    }
  }

  Serial.println("Initializing event");
  if(!bno08x.enableReport(SH2_ROTATION_VECTOR)){
    Serial.println("Event failed to initialize");
    while(1){
      delay(10);
    }
  }
  
  Serial.println("Initialization complete");
}
```
- To this
``` cpp
void ESC_CONTROLLER::initialize(int ESCPin, int turnServoPin, int SDA, int SCL){
  esc.attach(ESCPin);
  turnServo.attach(turnServoPin);
  ESC_CONTROLLER::initAsNeutral();

  Serial.println("Initializing I2C...");
  Wire.begin(SDA,SCL);

  Serial.println("Initializing bno08x...");
  if(!bno08x.dataAvailable()){
    Serial.println("bno08x failed to initialize");
    while(1){
      delay(10);
    }
  }
  Serial.println("Initialization complete");
}
```
## Log 2: Fixing messy code
- The code is a mess so I will seperate the driving logic and the PID/yaw logic into seperate h and cpp files, however everything will share one ino file.
- Rather than follow the code from the tutorial shown before, I am rather going to follow the example code from the sparkfun library itself, that way I have a known example which works
- The issue is it uses A4 and such pins, which is an arduino thing. I do not know if this translates well to ESP32.
### Solution arose
- After so long, maybe an hour or two, I finally got the code to register the existance of the bno08x. My mistake was sending the SDA and SCL pin to the bno08x.begin() command. You are meant to send it to the Wire.begin command, where you then define bno08x.begin(BNO08x_DEFAULT_ADDRESS, Wire).
## Log 3: Calculating yaw
- This was quite simple, as the library I am using (sparkfun bno08x library) has multiple example programs, so I just used that to create two functions
 ```cpp
//Here is where you define the sensor output you want to reveive
//FUNCTION COPIED FROM SparkFun_BNO09x_Arduino_Library
void PID_CALCULATIONS::setReports() {
  Serial.println("Setting desired reports");
  if (bno08x.enableRotationVector() == true) {
    Serial.println(F("Rotation vector enabled"));
    Serial.println(F("Output in form roll, pitch, yaw"));
  } else {
    Serial.println("Could not enable rotation vector");
  }
}
 ```
```cpp
float PID_CALCULATIONS::calculateYaw(){
  if (bno08x.wasReset()) {
    Serial.print("sensor was reset ");
  }
  //Is something happening
  if(bno08x.getSensorEvent() == true){
    //Is it what we want
    if(bno08x.getSensorEventID() == SENSOR_REPORTID_ROTATION_VECTOR){
      float yaw = (bno08x.getYaw()) * 180.0f/PI;// Convert yaw to degree
      Serial.printf("Yaw: %.2f\n", yaw);
      return yaw;
    }
  }
  return NAN;
}
```
- For once it works pretty easily.
# July 22
## Goals
- Make a protottype PID controll
## Log 1: Prototyping
- I'm printing a new control base alongside new support beams
- While I am waiting I might as well make a PID system to test out once it is wired up
### Thing to tackle 1: how to calculate angle
- The header will read from -180 to 180, so I need to adjust the target angle based on its current position. The simplest thing is to make an enum class for the directions right and left then add the target angle if I am moving right and subtract it if I am moving left. Then I will add a do-while statement afterwords so that the target angle does not change
```cpp
if(Direction == Turn::Right){
    targetHeader = PID_CALCULATIONS::calculateYaw() + targetHeader;
  }else if (Direction == Turn::Left){
    targetHeader = PID_CALCULATIONS::calculateYaw() - targetHeader;
  }
do{

}while(error < 3)
```
- Now I will add a PI system. I have chosen to delete D as it is not needed unless there is overshoot or oscilation. The items that are commented out are what is needed in order to add D later on.
```cpp
 //header and time calculations
  float startingHeader = PID_CALCULATIONS::calculateYaw();
  //float currentTime;
  float currentHeader;

  //k goal calculations
  float Kprop, Kint, Kdif = 0;
  float turnError;

  //Drive calculations
  float driveAdjustment;
  float goalMicroseconds;
  
  if(turningDirection == Turn::Right){
    targetHeader = startingHeader + targetHeader;
  }else if (turningDirection == Turn::Left){
    targetHeader = startingHeader - targetHeader;
  }

  do{
//    previous time = currentTime;
//    currentTime = millis();
//    float deltaTime = previousTime - currentTime;
    

    currentHeader = PID_CALCULATIONS::calculateYaw();
    error = targetHeader - currentHeader;
    
    //Wrap around
    while(error > 180) error -= 360;
    while(error < -180) error += 360;

    //Calculates how much the servo needs to shift by in order to acheive needed turn
    Kprop = kp * error;
    integral += error * dt;
    Kint = ki * integral
    turnError = Kprop + Kint;
    
```
- I also want to make it so as the error gets closer to 0, the car slows down. This section of commented code should explain my thought process well
```cpp
    /*
   * This is a bit of a complicated thought process so let me walk you through it.
   * The most I need the speed to vary by is 500 (1500 is stop, 1000 is full reverse 1500 is full forward)
   * Because of this, I need a difference of 180 to lead to a speed variance of 500 and -180 to -500to lead to +/- 180
   * The +/- part will be decided by whether I want to go forward or backwards
   * Therefore I need a variable to multiply this by to change the speed, lets call it driveAdjustment
   * driveAdjustment * 180 = 500, therefore driveAdjustment = 500/180
   * If i want to go in reverse this number is negetive
   */
```
- Implementing this idea we get an issue, the enum for drive is in the ESC files, I need the PID files to also acess them
- **Solution** Make a new class for things both files need to access.
- This file will be enums.h.
```cpp
#ifndef ENUM_H
#define ENUM_H
enum class Direction {
  Forward,
  Backward,
  Neutral
};
enum class Turn {
  Right,
  Left
};
#endif
```
-Now we can implement my idea
```cpp
if (movementDirection == Direction::Forward){
      drivingAdjustment = 500.0/180.0;
    }else if (movementDirection == Direction::Reverse){
      drivingAdjustment = -500.0/180.0;
    }
    goalMicroseconds = neautralMicroseconds + (drivingAdjustment * abs(error));
```
- Now I need to return these values. The issue is that I have two value of different types. How can i return two values?
- My research has shown that I can use **Dot operators** in order to attach multiple values to one variable. In action this looks like this.
- To do this I need to do struct, which is like grouping similar variables into one object
- The two variables I want to use are steeringCorrection and speedControl. I will add this to my return
## Log 3: Final prototype
-After fixes some small mistakes (semi collons missing, deleting any D variables, fixing while loop, etc) This is the final code
### Enums.h
```cpp
#ifndef ENUM_H
#define ENUM_H
enum class Direction {
  Forward,
  Backward,
  Neutral
};
enum class Turn {
  Right,
  Left
};
#endif
```
### PIDCalculations.h
```cpp
#ifndef PID_CALCULATIONS_H
#define PID_CALCULATIONS_H

#include "enums.h"
#include <Math.h>
#include <Wire.h>

#include "SparkFun_BNO08x_Arduino_Library.h"
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SDA_PIN 8
#define SCL_PIN 9

struct PIDOutput{
  float steeringCorrection;
  float speedControl;
};

class PID_CALCULATIONS{
  private:
    //IMU
    BNO08x bno08x;
    
    //K goals
    float Kp = 1.0;
    float Ki = 0.0;
    float Kd = 0.0;

    //Variables which need to be constantly recorded
    float integral = 0.0;
    unsigned long previousTime = 0.0 ;
    float lastError = 0.0;

    //Driving variables
    int neautralMicroseconds = 1500;

  public:
    //BNO08x
    void initBNO08x();
    // Yaw
    void setReports();
    float calculateYaw();
    // PID
    PIDOutput PIDSystem(float targetHeader, Turn turningDirection, Direction movementDirection);
};

#endif
```
### PIDCalculations.cpp
```cpp
#include "PIDCalculations.h"


  
void PID_CALCULATIONS::initBNO08x(){
  delay(10);
  
  Wire.begin(SDA_PIN, SCL_PIN);

  if(bno08x.begin(BNO08x_DEFAULT_ADDRESS, Wire) == false){
    Serial.println("BNO08x failed to initialize");
    while(1);
  }
  Serial.println("BNO08x found!");
}


//Here is where you define the sensor output you want to reveive
//FUNCTION COPIED FROM SparkFun_BNO09x_Arduino_Library
void PID_CALCULATIONS::setReports() {
  Serial.println("Setting desired reports");
  if (bno08x.enableRotationVector() == true) {
    Serial.println(F("Rotation vector enabled"));
    Serial.println(F("Output in form roll, pitch, yaw"));
  } else {
    Serial.println("Could not enable rotation vector");
  }
}


float PID_CALCULATIONS::calculateYaw(){
  if (bno08x.wasReset()) {
    Serial.print("sensor was reset ");
  }
  //Is something happening
  if(bno08x.getSensorEvent() == true){
    //Is it what we want
    if(bno08x.getSensorEventID() == SENSOR_REPORTID_ROTATION_VECTOR){
      float yaw = (bno08x.getYaw()) * 180.0f/PI;// Convert yaw to degree
      Serial.printf("Yaw: %.2f\n", yaw);
      return yaw;
    }
  }
  return NAN;
}

/* PID skeleton
 * error = target - current;
 * P = Kp * error;
 * integral = error * dt //global vaiable
 * I += Ki * integral;
 * D = Kd * (error - lastError) / dt;
 * output = P + I + D;
 */


PIDOutput PID_CALCULATIONS::PIDSystem(float targetHeading, Turn turningDirection, Direction movementDirection){

  //header and time calculations
  float startingHeading = calculateYaw();
  float currentHeading;
  unsigned long currentTime;

  float error;
  float steeringCorrection;
  
  //k goal calculations
  float Kprop = 0;
  float Kint = 0;

  //Drive calculations
  float drivingAdjustment;
  float goalMicroseconds;

  if(turningDirection == Turn::Right){
    targetHeading = startingHeading + targetHeading;
  }
  else if(turningDirection == Turn::Left){
    targetHeading = startingHeading - targetHeading;
  }

  do{

    previousTime = currentTime;
    currentTime = millis();
    float deltaTime = previousTime - currentTime;

    currentHeading = calculateYaw();
    error = targetHeading - currentHeading;

    //Wrap around
    while(error > 180) error -= 360;
    while(error < -180) error += 360;


    //Calculates how much the servo needs to shift by in order to acheive needed turn
    Kprop = Kp * error;

    //Integral accumulates error over time to remove steady-state error
    integral += error * deltaTime;
    Kint = Ki * integral;

    steeringCorrection = Kprop + Kint;



    //Slows down the motor as it reaches target header
    /*
     * This is a bit of a complicated thought process so let me walk you through it.
     * The most I need the speed to vary by is 500 (1500 is stop, 1000 is full reverse, 2000 is full forward)
     * Because of this, I need a difference of 180 to lead to a speed variance of 500 and -180 to -500 to lead to +/- 180
     * The +/- part will be decided by whether I want to go forward or backwards
     * Therefore I need a variable to multiply this by to change the speed, lets call it driveAdjustment
     * driveAdjustment * 180 = 500, therefore driveAdjustment = 500/180
     * If I want to go in reverse this number is negative
     */

    if(movementDirection == Direction::Forward){
      drivingAdjustment = 500.0 / 180.0;
    }
    else if(movementDirection == Direction::Backward){
      drivingAdjustment = -500.0 / 180.0;
    }
    goalMicroseconds = neautralMicroseconds + (drivingAdjustment * abs(error));

  }while(abs(error) > 3);

  //The results of our PID system
  PIDOutput output;

  output.steeringCorrection = steeringCorrection;
  output.speedControl = goalMicroseconds;

  return output;
}

```
# July 23, 2026
## Goals
- Finish printing new power base
- Create improved base stands
- Wire up car
## Log 1: New base
- Now that I have a final product, I will walk through the first three iterations
### Iteration 1
- This iteration has a few flaws, the mounts for each electronic is not the right size and there are no trenches for the soldered pins to go into
- [Design 1](https://drive.google.com/drive/folders/1dt7QzPdBV1O8T7NkzWnFHfQqQZx-trvL?usp=drive_link)
### Iteration 2
- This iteration fixes the issues of the first and would work as a base for my electronics, but it is ineffeicent. There is a lot of wasted space.
- [Design 2](https://drive.google.com/drive/folders/12wDUeO4QkyUliJY3y_FEt3RcoDbyPqOR?usp=drive_link)
### Iteration 3
- This will be the final product, I have removed a lot of unneeded plastic which does two very important things: it saves on plastic and I can print it in one piece
- [Final design](https://drive.google.com/drive/folders/1SjAeHhAOPUheJ63TsBto86ksDxHwo5R8?usp=drive_link)
## Log 2: Base holders
- Now that I have a base, I need a way to hold it above the RC car so it does not fit the servo.
- Luckily this only had two designs
### Design 1
- For this design I planned to replace the mounting brackets for the RC car's body, however I will be the first to say that it sucks. By sliding over an existing screw, it becomes too wide to slide a pin in. It also has too many weak points since it is hollow.
- For my next design I will keep the part which attaches to the electronics base the same, however I am going to add a threaded screw to attach to the RC car itself for a stronger hold
- [Design one](https://drive.google.com/file/d/1O8Iumoc8ymEDaNzHmxilMSTGcEkbcfHc/view?usp=drive_link)
### Design 2
- I added the modification mentioned in the previous design log, however the screw is terrible. The main issue is that the screw is too thin, so I will 3D print just the screw part but make it 3mm in diameter rather than 2
- This has shown me another issue, the overly long screw breaks easily within the screw hole, forcing me to melt it with a soldering iron and pluck it out. However, 3mm did work. For my new design I will shorten the screw's length.
