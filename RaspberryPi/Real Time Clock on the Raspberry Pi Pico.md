# Real Time Clock on the Raspberry Pi Pico

[![Workshopshed](https://community-storage.element14.com/communityserver-components-imagefileviewer/communityserver/components/avatars/00/00/41/85/02/4UFBJXOXOCQX.png-80x80x2.png?_=ljtNyEKtC25b2deookdtzg==)](https://community.element14.com/members/workshopshed)

[Workshopshed](https://community.element14.com/members/workshopshed)

25 Jun 2022

I've been working on a Raspberry Pi Pico project that I wanted to run off batteries. I found a "low power" example in the [pico-extras](https://github.com/raspberrypi/pico-extras) which uses the Real Time Clock to wake the device up after an interval. So I set about porting it to the Arduino platform and had a bunch of issues with it compiling and linking, mostly to do with the real time clock.

I found a library [RP2040_RTC](https://github.com/khoih-prog/RP2040_RTC) by  Khoi Hoang but that didn't seem to want to install and when I manually installed it, it didn't want to compile.

Help was at hand however in the Arduino core discussion forum - [https://github.com/arduino/ArduinoCore-mbed/issues/461](https://github.com/arduino/ArduinoCore-mbed/issues/461) this gave me a few ideas to try.

I eventually stumbled upon the [rtc_api](https://github.com/arduino/mbed-os/blob/extrapatches-6.15.1/targets/TARGET_RASPBERRYPI/TARGET_RP2040/rtc_api.c) part of the code provided with the "Pi Pico Board" in Arduino. It had some calls I could use to initialise the clock and check it was running. I found that it took a few ms to run and could incorrectly report 1970 as the date if you didn't wait for the enabled status to change. Khoi's code also helped me out in recommending Paul Stoffregen's time library to do some of the basic conversions.

This led to the following code:

```
#include "rtc_api.h"
#include <TimeLib.h> //https://github.com/PaulStoffregen/Time

tmElements_t tm;
time_t t;

void setup() {
tm.Year = CalendarYrToTm(2022);
tm.Month = 06;
tm.Day = 25;
tm.Wday = 6; // 0 is Sunday, so 6 is Saturday
tm.Hour = 8;
tm.Minute = 23;
tm.Second = 00;

t = makeTime(tm);
// put your setup code here, to run once:
Serial.begin(9600);
while(!Serial);

sleep_ms(5000);

// Start the RTC
Serial.println("Initialising RTC");
rtc_init();

//Write initial date
rtc_write(t);

while(!rtc_isenabled())
{ Serial.print(".");}
Serial.println("");

Serial.println("RTC Running");
}

void loop() {
t = rtc_read();

Serial.print(year(t));
Serial.print("-");
Serial.print(month(t));
Serial.print("-");
Serial.print(day(t));
Serial.print(" ");
Serial.print(hour(t));
Serial.print(":");
Serial.print(minute(t));
Serial.print(":");
Serial.println(second(t));
Serial.println();
sleep_ms(1000);
}
```
With a bit of tuning I also go the alarm function to work you can see both examples at: https://github.com/Workshopshed/PiPico