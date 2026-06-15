# June 14 2026
## Goals
- Power on ESP via RC car
- Move motor with ESP
- Connect servo to ESP in order to turn the car
- Start design on ESP32 mount
## Important note
  - After a week of attempting to desolder throughholes I have chosen to try again at a later date near a professional and buy a new ESP whose headers are not soldered as to not waste valuable time.
## Log 1: Power
### Try 1:
  - I have spliced a female wire to the male wire for both a 5V and GND in order to connect the power from the buck to the ESP, and it worked flawlessly. This allows the code to execute with the robot, eliminating lag.
    - [ESP is powered on by RC car](https://drive.google.com/file/d/1m3gU6e3gT_kMek5OZZbFTPobyGX79Qeu/view?usp=drive_link)
## Log 2: 
### Try 1:
  - Ran code from previous log file without altering it
  - Try failed, the serial port is not being read, maybe the board configuration was done incorrectly
      - (Running motor try 1)[https://drive.google.com/file/d/1vUi3IwTjfsRy6KyLP9qBH4FcRZp4Uwap/view?usp=drive_link]
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
      - (Running motor try 3)[
  - By plugging my phone into my PC, I verify that this is a transfering USB cable
### try 4:
  - Rebotted ESP
  - The code has uploaded and the motor runs, but the wheels won't move
      - (Wheel's wont run}[https://drive.google.com/file/d/1nYilarqoGKA7xZtTdCusuBla2_7aNoTo/view?usp=drive_link]
