# Freya
Freya is robot that aims on cleaning solar panels. This repository is mainly for the electrical and software section of the project.

## Overview
  Overview of the project
## Componenets & Code
  In this section I will be discussing each compoenets and why we are using them. We will be using Arduino Mega for our project
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
