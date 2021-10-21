# Freya
Freya is robot that aims on cleaning solar panels. This repository is mainly for the electrical and software sections of the project.

## Overview
  Overview of the project
## Componenets & Code
  In this section I will be discussing each compoenets and why we are using them. We will be using Arduino Mega for our project due to large amount of I/O pins required.
### DS3231 Real Time clock
One of the functionality Freya is to clean the solar panel on a schedule (ie. every day at 7:00AM). Even though the arduino has a buit-in timer the DS3231 module provides highly accurate time and also with its 3V batttery the internal clock will keep running even thought the arduino is off(up to a year).

The module will be using Serial 13 pins on the arduino mega for communications.
```c++ 
DS3231  rtc(SDA, SCL);
```
The program will grab time from the module on every loop. The methodd ```updateTime()``` will be called on every loop to keep track of time.
```c++
void updateTime(bool print)
{
  time_0 = rtc.getTime();

  if(print)
  {
    Serial.print(rtc.getDOWStr());
    Serial.print(" ");
  
    Serial.print(rtc.getDateStr());
    Serial.print(" -- ");
    Serial.println(rtc.getTimeStr()); 
  }
}
```
### Bluetooth HM05
Smartphones will be able to connect and communicate with freya using bluetooth communication. We will be using the bluetooth HM05 Module and Dabble Library. The Dabble library has an exisiting app that will able to communicate with freya.
We start by adding the Dabble Library.
```c++
//library for bluetooth communication
#include <Dabble.h>
```
Secondly, In the ```start()``` We will set the baud for terminal and Dabble. (both must share the same baud)
```c++
  Serial.begin(9600); //Commuincation setup for Serial terminal
  Dabble.begin(9600); //Communication setup for Bluetooth module
```
Finally, we will a method called ```loop_BT()``` that will be called in the main loop function to check for any incoming data
```c++
void loop_BT()
{
  Dabble.processInput();
  while (Serial.available() != 0)
  {
    Serialdata = String(Serialdata + char(Serial.read()));
    dataflag = 1;
  }
  if (dataflag == 1)
  {
    Terminal.print(Serialdata);
    Serialdata = "";
    dataflag = 0;
  }
  if(Terminal.available())
  {
    while (Terminal.available() != 0)
    {
      Serial.write(Terminal.read());
    }
    Serial.println();
  }
}
```
### Vibration Sensor (SW-420)
Wind speed in solar farms or in open areas can reach up to 25m/s (90km/h) and any anomalies in the system will cause vibration. Therefore, a braking mechanism is needed to stop freya and hold it in place in case of high vibration.
We will be using SW-420 vibration module that will trigger at a certain threshold. The threshold can be adjusted manully on module using a screwdriver (testing is required)
In terms of coding the module will send a signal of ```1``` or ```0``` if it sense above the threshold.
```c++
void loop_Vibration()
{
  vibrationState = digitalRead(vibration_pin);

  if(vibrationState == 1)
  {
    completeStop();
  }else
  {
    //ignore
  }
}
```
```completeStop()``` will be called when vibration is above the threshold and will trigger braking mechanism and stop the robot.
## Power & Storage unit
one of our goals is to make freya power free, does not need external power to run, thus it will be using solar panels and battery to power itself. For power generating We will be using 2x100 watts @ ~18V solar panel that will be connected in parraller.

### Power Consumption:
  Before jumping into the power generating and storage components lets analyzie our power consumption to better understand our choices.
  
  ----TABLE-----
  
  Using the same motor models for our application will simplify our work in terms of coding, testing, and shipping, hence we are using the 3 same motors in our application. As we can see in the above table, the major power consumption will be from our motors. Although the motors are rated at 240 watts We will be using much less power and we can ignore the microntroller and others power consumption.
  
  lets calculate power consumption for each motor(Higher end)
  ```
  Given: 120 watts (expected power needed) @ 12v for each motor
  ```
  We increase our wattage consumption to prevent undercalculations.
  ```
  120watts * 1.2 = 144 watts
  ```
  Now, we get the current needed:
  ```
  120watts / 12V = 12A
  ```
  Multipling by each motor we have:
  ```
  12A * 3 = 36A
  ```
  
  Based on the calculation above assuming all motors are running at that power consumption we get `36A`. At first this seems a lot of amps in a small device, yes it is, however this not gonna happen. For instance, the brush motors might need 120 watts at the start but after 1-2 seconds the only forces that is slowing it down are air resistance and fricion which the motor has to compensate for that only. 

### Storage Unit
  With the power consumption in mind, We can determine which battery suits our needs. First of foremost, lithium ion batteries are a great choice for moblie robots such as our application. They have high power density, low internal resistance, and lightweight. However due to our enviorments that we will be using it in and power needs the Lithitum ion batteries does not seem a suitable option for our application. For example, the robot will be running under the heat and direct sunlight which will expose the batteries under high heat. Another problem, We will need multiple of them to power the device and they are not very cheap for comparsion a double 6Ah @ 14.8 would cost around 500 AED (~130 USD) however similar specific of lead acid battery would cost 150 AED (~40 USD). Our battery choice provided by company by the name LONG is `WP18-12` with the specific of 18Ah at 12V.
  
 Discharge calculation for the battery:
 ```
 Given: 36A @ 12V
 ```
 Based on the given battery we can calculate the time needed to fully discharge by:
 ```
 18Ah / 36 A = 0.6 hour
 ```
Converting to minutes:
```
60 minutes * 0.6 = 36 mintues
```

  Although batteries dont discharge linearly so we will take `30 minutes` of total discharge time (neglating solar panels) since our device is rated at `11 meter/minute` so it will be able to clean for `330 meters` until fully discharged.
  
### Power Generating (Soloar Panel)
  The provided solar panels are 2x100watts they can output max. current at 5-6A. They solar panels will be connceted in parrellel becuase of high current output(faster charging) and reduance in terms of panel failure.
  Caclulating solar charging at ideal

Caclulating charging time:
```
- 12v, 18000mAh Lead Acid battery
- 2x100 watts Solar panels

- Since solar panels are not ideal we will assume that both are running at 50%:
  
  200 watts * 0.5 = 100 watts
  
- Divide solar panel wattage by battery voltage to get the output current:

  100W / 12V = 8.3A
  
- Multiply the current by the losses due to PWM & MPPT:

  8.3A * 80% * 80% = 5.3A

- Multiply the battery capacity by 1 divided by efficiency (for Lead Acid 85%):

  18Ah * ( 1/ 0.85) = 21.2 Ah
  
- Divide the battery capacity by the current:

  21.2Ah / 5.3A = 6.0 hours


```
Since solar panels are not ideal we will assume that both are running at 50%:
```
200w
```
1.) Divide solar panel wattage by battery voltage to get the output current:
```
100W / 12V = 2.7A
```
2.) Multiply the current by the losses due to PWM & MPPT:
```
2.7A * 80% * 95% = 2.05A
```
3.) Multiply the battery capacity by 1 divided by efficiency (for Lithium 95%):
```
12Ah * ( 1/ 0.95) = 12.63 Ah
```
4.) Divide the battery capacity by the current:
```
12.63Ah / 2.05A = 6.0 hours
```
Therefore, Freya will need at least 7 hours of direct sunlight to fully recharge.
### Power consumption
The power consumption of freya is really hard to estimate due to not enough testing. However the main power consumption will be from the motors based on theoritical calculation and assumption freya can operate for 40-60 minutes (330 meters). Further calculation and testing needed to get the excat number & confirm the duration.
