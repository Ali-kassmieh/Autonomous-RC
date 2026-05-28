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
