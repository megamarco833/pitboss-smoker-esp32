# pitboss-smoker-esp32
hack of pitboss navigator bbq smoker with ESP32 controlling remotely. 
this hack is only for navigator model with rotary switch to control the temperature
the purpose of this hack is to control remoetly the temperature of the pitboss navigator and smoke level

 # what you need:
- ESP32
- tasmota firmware
- nodered
- domoticz
- pitboss navigator 850

![photo_2022-08-10_15-54-18](https://user-images.githubusercontent.com/44502572/183919165-a58cb6dc-dcd4-4ed5-92af-0bf4cb0b3443.jpg)


**thanks to @Vic**  for help on making this happen to modify the circuit and also thanks to tasmota discord channel: [Discord](https://discord.gg/Ks2Kzd4)


1. deassemble the control board and look at the rotary pins and GND pin (look at inage attaced)
![IMG_20211228_093902](https://user-images.githubusercontent.com/44502572/183908095-6dba43e9-775e-44c1-b0a7-5516849fcbde.jpg)

2. create a small PCB with electonic devices as in the picture below.
  please note that R32 is where we can provide vcc and gnd is exposed on pcb
  the idea is that ESP32 with tasmota firmware can simulate using PWM the rotray switch.
  Rotary switch is changing Volage value like in the table below
  ![0unknown](https://user-images.githubusercontent.com/44502572/183912722-62b82c26-4db4-42a6-9c9b-f7fc120a4d24.png)



here you can see the connection

![0temperature](https://user-images.githubusercontent.com/44502572/183915460-1aed9b2c-4c1c-45d2-984b-8aac52bf96bc.png)

with this mode, "I simply" send to pitboos CPU a different value of voltage, because pitboss CPU read the voltage that rotary encoder send out.
so now without manual rotation of rotary encoder, thanks to PWM i send out the volage step that the pitboss CPU expect.
at every voltage step correspond a temperature settings and pitboss CPU regulate the pellet speed consequentially to reach that temperature and then to maintain that temperature constantly
keep in mind that measuring volts between R32 and GND exposed on pitboss PCB
 you will find:
 slector OFF => POSITION1 = 0V => PWM level from ESP32 = 0
 selector AT SMOKE LEVEL => POSITION2 = 0.5v => PWM level from ESP32 = 6
 selector AT 95°c => POSITION3 = 1v  => PWM level from ESP32 = 8
 selector AT 110°c => POSITION4 = 1.5v => PWM level from ESP32 = 11
 selector AT 120°c => POSITION5 = 2v => PWM level from ESP32 = 14
 selector AT 150°c => POSITION5 = 2v => PWM level from ESP32 = 14
 selector AT 175°c => POSITION6 = 2.5v => PWM level from ESP32 = 18
 selector AT 200°c => POSITION7 = 3v => PWM level from ESP32 = 24
 selector AT 220°c => POSITION8 = 3.5v => PWM level from ESP32 = 36
 selector AT 240°c => POSITION9 = 4v => PWM level from ESP32 = 70
 selector AT 260°c => POSITION10 = 4.5v => PWM level from ESP32 = 190
 selector AT MAX°c => POSITION11 = 5v => PWM level from ESP32 = 600
  

  below an image to explan it 
   ![immagine](https://user-images.githubusercontent.com/44502572/185743035-c55ff010-fe17-45e9-ad29-fddadc4197ba.png)


3. for push button that control the smoke here below you can see the connection to ESP32
tasmota will simulate the pression of the button so you can control remotely the smoke level
![0push-button](https://user-images.githubusercontent.com/44502572/183913133-0b1f04f4-e981-45e2-81e5-7b1e761f0d0a.png)

4. below you can fine a rule that is setup on tasmota to publish the value of pwm:

`rule3 ON Analog#A1div10 DO publish BBQ %Value% ENDON ON SYSTEM#BOOT DO Status 10 ENDON `

`rule3 1`

the purpose is to publish the ADC values in a topic named BBQ. in this case every settings of the rotrary switch can be easly recognized on the BBQ topic.
using also this rule we will publish on topic BBQ only if there is a variation of ADC value that conrespond to a temperature step, avoiding to continue to publish every ADC value
as explained rotary encoder send a different value of voltage to pitboss CPU, and so pitboss CPU regulate consequantially the pellet speed to reach and maintain the setted temperature.
so 

5. now if you want to control remotely the BBQ changing the temperature and smoke level it's just a metter to use systems like nodered + domoticz
the idea is to create in domoticz a dummy device that will have multiple selection (selector switch) and so you can address evey level of the domoticz dummy switch to a temperature of pitboss
with nodered you can send the domoticz/out => to tasmota, and tasmota will change the PWM level and BBQ will change the temperature
same story for push botton to control the smoke level
