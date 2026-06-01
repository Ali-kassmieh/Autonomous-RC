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
