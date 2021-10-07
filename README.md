# Freya
Freya is robot that aims on cleaning solar panels. This repository is mainly for the electrical and software sections of the project.

## Overview
  Overview of the project
## Componenets & Code
  In this section I will be discussing each compoenets and why we are using them. We will be using Arduino Mega for our project because we will need more pins for controlling Freya
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
Smartphones will be able to connect and communicate with freya using bluetooth communication. We will be using the bluetooth HM05 Module and Dabble Module. The Dabble library has an exisiting app that will able to communicate with freya.
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
one of our goals is to make freya power free, does not need external power to run, thus it will be using solar panels and battery to power itself. The estimated power consumption of the freya will be no more than ```150W``` or less and We will be using two of 14.8v 6000mAh batteries. Regarding generating power We will have 40watts 18v Solar panel.
### Charging caclulations
Our givens:
```
- 14.8v, 12000mAh Lipo battery
- 40 watts Solar panels
```
1.) Divide solar panel wattage by battery voltage to get the output current:
```
40W / 14.8V = 2.7A
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
The power consumption of freya is really hard to estimate due to not enough testing. However the main power consumption will be from the motors based on theoritical calculation and assumption freya can operate for 40-60 minutes (330 meters). Further calculation and testing needed to get the excat number.
