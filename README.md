Leslie Dual Motor foot switch speed control

![image](https://github.com/user-attachments/assets/01bef3c2-a474-4144-9130-0a6561636c59)


<img width="200" alt="image" src="https://github.com/user-attachments/assets/5ffa6505-e3ad-4f38-842d-050278efdd07"> <img  width="300" alt="image" src="https://upload.wikimedia.org/wikipedia/commons/f/f8/Lesliebox_Animation.gif?20070728184739">

I love Leslies.  I've had dozens of them over the years.  My favs are the 146's and 122's, and the 46w is equally good.  But....  to control them, I like a momentary on/off foot switch that varies the speed and also stops the motors completely - like on My Hammond XK5 and Leslie 3300.  I know, I know.  Clonewheel + new Leslie.  The sound is close, but not ideal.  So I need to add another REAL leslie to the picture.  And, Ideally, one that allows the OFF function to stop both motors. Jimmy McGriff and other artists used the OFF function on the Leslie to great effect.  So, how do we do this?  I started experimenting with a 555 timer circuit that "sort of worked", but when I hooked up a highly inductive load like a leslie motor to the miniture relay, all hell broke loose. I couldnt mitigate the interference.   Not to mention the loud CLICK of the typical relay. 

so, How do we get rid of the mechanical relay for totally silent speed switching, and add reliable slow/fast AND OFF switching?  Read on.

Several years ago, I purchased a solid state AC relay module that is controlled with 5VDC signal.  Clearly I had something in mind for my Lslies at the time, I just didnt know what until now.  What if I used a programmable micro controller like a cheap, readily available ESP8266 and then programmed it with ESPHome to enable full control of the motors?  OK... But... What about Inductive interference from the motors like on the 555 circuit?   I can absolutely mitigate that with debouncing logic on the GPIO's through clever programming.  OK let's try it!

So, I wired it up, added a 3.3v to 5v level shifter to the outputs, and I noticed immediatly that even w/ the level shifter, the outputs both needed to be pulled up to 5v w/ 100 ohm resistors so the solid state relay would trigger when the GPIO's go high.  When the GPIO is low, at 1.2v 100 ohm 0.25w resistors will have to sink 0.012A. 0.012A  * 1.2v = 0.012w.  No problem.

UPDATE: I just got the updated version of the SS relay board and it now has transistor drivers on it.  it works like a champ, directly connected to the GPIO outs of the 8266.

![image](https://github.com/user-attachments/assets/2d34f49e-0abb-4718-b9a5-f9e02b148c6b)


ok, lets work on the logic.  Initially I used the "toggle" output function to control the switching of the GPIO's, but then thought what if the 2 outputs (slow motor, fast motor) get out of sync?  they could be both on, or both off. not ideal.  So, I opted to explicitily set the speeds when the foot switch is activated:
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
I also configured on_boot so startup has the slow motor selected:
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
several other mods in the config file mitigate having no Wifi and setting startup speeds upon boot, etc.  
Good thing I've got loads of ESPHome experience w/ my Pellet stove! 
So, there you have it.  

A footswitch enabled 2 speed Solid State Leslie motor controller. With stop function.  Open Source.
Enjoy!

Link to the SainSmart 5V 2-Channel Solid State Relay Board controlled with 5v
https://www.amazon.com/dp/B0079WI2ZC?ref=fed_asin_title

UPDATE: these relay modules only work with SOME leslies that use DC motors. Most others (147,122,etc) require the non-zero cross version because those motors are INDUCTIVE MOTORS. The SSR part numberfor those ends in PL (202-PL).  The board logic works, all you have to do is replace the 202P relays with the 202PL relays.  They are the same footprint so they are direct replacements. 

![image](https://github.com/user-attachments/assets/dd387b88-90e4-41a6-ae7a-23c24b7739c1)

video of Motor in action

https://youtube.com/shorts/yKDL8vKTTh8?feature=share

<img width="100" alt="image" src="https://github.com/user-attachments/assets/d404a2ec-5a46-47b8-8777-807fee2db2e2">


Pics of completed unit
![image](https://github.com/user-attachments/assets/28257a7e-4166-48bb-aa41-3e095643bd33)



