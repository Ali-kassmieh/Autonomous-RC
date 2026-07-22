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
