# July 1, 2026
## Update
- Got busy fixing up our yard but I'm back with two goals
## Goals
- Fix electronic base
- Get a yaw reading
## Goal 1: Fix electronics base
### Log 1: Fix electronic base
- The first iteration has many issues
  - The casing for the terminal block is 0.5 centimeters too long, however the screw holes are fine
  - For all the soldered parts I failed to add a trench for the soldered metal to go in
  - The casing for the buck voltage regulator is 2mm too thin
  - The ESP32's casing is 2mm too wide
- There was one sucess
  - The header fit perfectly
## Goal two: Get yaw reading:
### Log 1: First attempt
```cpp
float getYaw() {
    if (bno08x.getSensorEvent(&event)) {
        if (event.sensorId == SH2_ROTATION_VECTOR) {
          
            float qi = event.un.rotationVector.i;
            float qj = event.un.rotationVector.j;
            float qk = event.un.rotationVector.k;
            float qr = event.un.rotationVector.real;

            float yaw = atan2(
                2.0f * (qr * qk + qi * qj),
                1.0f - 2.0f * (qj * qj + qk * qk)
            );

            yaw *= 180.0f / PI;
            if (yaw < 0)
                yaw += 360.0f;

            return yaw;
        }
    }

    return 0.0f;
}
```
- This is the code/equation I got from the ardunio forum utilizing the equation to calculate yaw using rotational vecotrs.
- I chose this equation because it fits cleanly into using the bno08x IMU's roational vectors.
#### Issues:
- When I try to print the yaw, a lot of weird text about my esp prints into the serial monitor
- Upon further research, this happens whenever my ESP is stuck in reboot mode
- Before changing code I should alter some settings
### Log 2 & 3: Alter settings
- There are three issues that could have caused this
    - USB cord is too long/isn;t transfering data well
    - Using QIO instead of DIO flash mode
    - The ESP is not in download mode
- Given I have been using this cable for a while I doubt thats the issue, so it must be the second or third issue
#### Issues:
- It was not the second issue
- I will add debugging to my begin function since I beleive that is where this is coming from
- I also renamed the function to initialize in order to avoid confusion with Serial.begin().
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
```
# July 3, 2026
## Goals
- Fix the yaw reading
## Log 1: Debug the code
- After a lot of research and asking questions I have fixed my code. To begin, the println function wasn't the thing that was broken, the entire upload mechanic was broken. This is because of two reason: PSRAM and USB CDC
    - PSRAM means Psuedo Static RAM. It is basically temporary RAM put in reserve incase the existing RAM is overfilled. My microcontroller ends in N16R8. This means that there are 16 MB of flash and 8MB of PSRAm. My setting were set to 4MB flash and ) PSRAM
    - USB CDC is how the microcontroller gets data uploaded via USB. It was disabled so no data was being uploaded
## Log 2: Get YAW to work
- Running the new initialization function returns this
```plaintext
Setup begun
Initializing I2C...
Initializing bno08x...
I2C address not found
bno08x failed to initialize
```
- Looking at the bno08x, the ground pin may not be soldered correctly
- Yea I fried it. Next time I need to be careful
- I have to wait until sunday for a new board, in the meantime
## New goals
- Figure out encoder
### Log 1: New method
- The magnetic encoder I have is not very good for this project because I need the chip the remain hovering only 1-3mm from the magnet which is rotating on the wheel, however the chip cannot move.
- My first idea was simply a casing fitted around the wheel so that the microcontroller can be stationary while hovering over the magnet
    - This would require the wires be directly soldered onto the board rather than connected to the header pins. This is not the worst solution
- From my research I have discoverd that last year the TN Tech Slow-Mho designed their own ground vehicle in which they utilized GPS and motor controllers
    - While this is a much better method, there are a few issues
        - GPS's do not work well indoors
        - I am too poor
- The casing idea will have to work
### Try 1
