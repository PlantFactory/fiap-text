## How to use Time library

It is a library dealing with time in Arduino. Count the current time using built-in timer. It is based on UNIX time.

First, set the standard time. I count from this time. It is good to use an NTP library etc. separately.
```C++
setTime(0);
```
By the way, when setting the time zone, we recommend setting UTC to the `setTime` function and using LocalTimeLib.

To get the current UNIX time, call the `now ()` function. You can also use the `hour ()` and `minute ()` functions to obtain each numerical value.

* [Time Reference](https://www.pjrc.com/teensy/td_libs_Time.html)
