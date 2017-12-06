## How to use LocalTimeLib

LocalTimeLib is a library which manages Arduino's time using UNIX time with time zone. Simply, we can treat this as though we are using the Time library.

In the ino file, define the time zone in the global scope.
```C++
TimeZone localtimezone = {  9*60*60, 0, "+09:00" };
// { Offset from UTC, SummerTime, timezone String }
```
Daylight saving time is not used. Please set the timezone to "timezone String" according to the ISO8601 standard.

When you call up `localtime()`, a pointer to `TimeElements` is returned. Each element within TimeElements stores the time zone information that is applied to it. When displaying the time in the ISO 8601 format, the code is as follows.
```C++
// in global
TimeZone localtimezone = {  9*60*60, 0, "+09:00" };

// in some function
setTime(0);
TimeElements *tm = localtime();
printf("%04d-%02d-%02dT%02d:%02d:%02d%s",tm->Year + 1970, tm->Month, tm->Day, tm->Hour, tm->Minute, tm->Second, localtimezone.iso_string);
// get: 1970-01-01-00:00:00+09:00
```
