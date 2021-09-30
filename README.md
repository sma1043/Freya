# Freya
Freya is robot that aims on cleaning solar panels. This repository is mainly for the electrical and software section of the project.

## Overview
  Overview of the project
## Componenets
  In this section I will be discussing each compoenets and why we are using them.
### DS3231 Real Time clock
  One of the functionality Freya is to clean the solar panel on a schedule (ie. every day at 7:00AM). Even though the arduino has a buit-in timer the DS3231 module provides highly accurate time and also with its 3V batttery the internal clock will keep running even thought the arduino is off(up to a year).
  
```c++ DS3231  rtc(SDA, SCL);```
