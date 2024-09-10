# lesliefootswitch
Leslie Dual Motor foot switch speed control

I love Leslies.  I've had dozens of them over the years.  My favs are the 146's and 122's, and the 46w is equally good.  But.... I to control them, I like a momentary on/off foot switch that varies the speed.  And, Ideally, allows the OFF function to stop both motors. Jimm McGriff and other artists used the OFF function to great effect.  So, how do we do this?  I started with a 555 time circuit that "sort of worked", but hook up a highly inductive load like a leslie motor to the relay and all hell broke loose.  Not to mention the CLICK of the relay. How do we get rid of the mechanical relay, add slow/fast AND OFF switching?  Read on.

Several years ago, I purchased a solid state AC relay module that is controlled with 5VDC signal.  Clearly I had something in mind at the time, I just didnt know what until now.  What if I used an ESP8266 and programmed it with ESPHome to enable full control?  OK... But... Inductive interference from thre motors like on the 555 circuit?   I can absolutely mitigate that with debouncing logic on the GPIO's.  OK let's try it!

So, I wired it up, and I noticed immediatly the outputs needed to be pulled up to 5v w/ 100 ohm resistors so the solid state relay would trigger.  When the GPIO is low, at 1.2v 100 ohm resistos will have to sink 0.012A. 0.012A  * 1.2v = 0.012w.  No problem.

ok, lets work on the logic.  Initially I used the "toggle" output function, but then thought what if the 2 outputs (slow motor, fast motor) get out of sync?  they could be both on, or both off.  So, I opted to explicitily set the speeds:
```
    on_release:
       then:
        - if:
            condition:
              switch.is_on: fastmotor        
            then:
              - switch.turn_off: fastmotor            
              - switch.turn_on: slowmotor            
            else:    
              - switch.turn_on: fastmotor            
              - switch.turn_off: slowmotor
```
I also configured on_boot so startup has the slow motor selected
```
 on_boot:
    then:  
      - switch.turn_on: slowmotor
      - switch.turn_off: fastmotor
```
I've always wanted a STOP function.  here it is.  Hold the switch for 2s and it's enabled.
```
script:
  - id: checkforstop
    mode: restart                   
    then:
    - logger.log: "script CheckForStop running"
    - delay: 2s
    - if:
        condition:
          binary_sensor.is_off: footswitch
        then:          
            - logger.log: "stop mode detected"  
            - switch.turn_off: fastmotor            
            - switch.turn_off: slowmotor
```
So, there you have it.  A footswitch enabled 2 speed Leslie motor controller. With stop function.  Open Source.
Enjoy!

