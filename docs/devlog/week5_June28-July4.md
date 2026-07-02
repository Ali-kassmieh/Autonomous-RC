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

# July 2, 2026
## Goals
- Print electronics base
- Fix the yaw reading
## Log 1: Debug the code
