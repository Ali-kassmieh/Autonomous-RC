# May 28th, 2026:
## Objective: 
  - Educate myself on the innerworkings of RC cars, including how the relay and ESC function in relation to the DC motors and determine how a microcontroller may be connected in order to achomplish autonomy.
## Notes: 
  - While I did not do this on purpose, by buying the VW Type 2 bus model I have effectively given myself ample empty space in order to fit electronic parts.
  - It is crucial to constantly make sure that the electroic parts I add in are able to handle the RC cars battery
  - The ESC controls how much voltage is sent to each motor by choping up constant DC into controlled pulses. This sounds like PWM I am used to in microcontrollers; coupled with the fact that these ESC's are programmable, maybe I can utalize PWM? More research needed
  - Some power from the ESC is diverted into the relay which is connected to a servo. It does this with BEC which to my current understanding is like a resistor but instead of resisting current it brings down the batteries volt into a constant lower volt. **The important thing is that if I chage out the relay and servo to a higher power one I need to reprogram BEC**
  - It seems like the secret to this project is in the relay. Perhaps switching out the relay or hacking into a component may create my autonomous teleop hybrid
## Goals:
  - Look into the posibility of PWN integration
  - Look into how to connect a microcontroller to a relay

# May 31, 2026
## Objective
- Figure out how to replace relay with microcontroller
- Funding
## Notes
### Microcontroller
- My previous thought that ESC work on PWM was correct and therefore rather than interfacing a microcontroller with the relay I will replace the relay
- I will use an ESP 32 for this project as it already has wifi capabilities that will make relaying much easier
- **New Research:** turns out I was thinking incorrectly. The ESC has a BEC which powers the receiver, not the relay. However, if i remove the receiver and connect the ESC's GND and Control wire to the ESP's GND and GPIO wire then seperatly power the ESP, it will work.
### Funding
- I have also realized that there is one constant in life: money. I am without funding except for my part time job at Subway, which as you can imagine is not a lot of money
  - I can try to sell some old workout things I have lying around, though its only a bench and adjustable dumbbells
  - I could get another job but given I leave to college in 3 months I doubt I would get hired
## Takeaways
- Replace receiver by connecting ESP to ESC's control and GND pin
## Future research
- How do I code this

# June 3, 2026
## Objective
- Figure out the correct Pulse Widths for forward and backward
- If time allows, get the servo working
## Notes
- I got the funding and am awaiting my ESP 32, but for now ill use my arduino for basic code
- I got the arduino to connect to the ESC and I got the motor running for a predetermined amount of time
- Movements work via microseconds, and I have found that 1500 microseconds coresponds to neutral

  ### Issue: I cannot seem to get the motor to move backwards
- **debug try 1** Try a bunch of values until it moves
 ```cpp
#include <Servo.h>
Servo esc;

void setup() {
  // put your setup code here, to run once:
  Serial.begin(115200);
  esc.attach(13);
  esc.writeMicroseconds(1500);
  delay(3000);
}

void loop() {
  // put your main code here, to run repeatedly:
  int testTime = 3000;
  Serial.println("began testing");
   for (int i = 0; i < 10; i++){
     int testSpeed = 1000 + (100*i);
     Serial.println(testSpeed);
     esc.writeMicroseconds(testSpeed);
     delay(testTime);
   } 
}
```
- **Findings** The first test works very well, proving that 1000 is full backward, 1500 is neutral, and 2000 is full forward. However, when it loops back only the forward speeds work. This could become problomatic if it wont behave the same every time

- ***Debug try 2:*** Force neutral before restarting loop
```cpp
  #include <Servo.h>
  Servo esc;

  void setup() {
    // put your setup code here, to run once:
    Serial.begin(115200);
    esc.attach(13);
    esc.writeMicroseconds(1500);
    delay(3000);
  }
  
  void loop() {
    // put your main code here, to run repeatedly:
    int testTime = 3000;
    Serial.println("began testing");
     for (int i = 0; i < 11; i++){
        esc.writeMicroseconds(1500);
        delay(1000);
       int testSpeed = 1000 + (100*i);
       Serial.println(testSpeed);
       delay(500);
       esc.writeMicroseconds(testSpeed);
       delay(testTime);
     } 
    }
```
- ***Findings:*** It works now across multiple tests
- ***Video:*** 


