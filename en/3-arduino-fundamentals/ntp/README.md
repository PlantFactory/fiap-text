## How to use NTP

Get the time with NTP. For this to work, the Ethernet library must be usable. First let's instantiate.
```C++
#include <NTP.h>

NTPClient ntpclient;
```

Please check the [official reference](https://www.arduino.cc/en/Reference/Ethernet) regarding the initialization and usage of the Ethernet library. In addition, the code to set the IP address with commonly used DHCP is as follows.
```C++
#include <Ethernet.h>

byte mac[] = {
  0x00, 0xAA, 0xBB, 0xCC, 0xDE, 0x02
};

Ethernet.begin(mac);
```

Communicate by NTP and get time information.
```C++
uint32_t unix_time = 0;

ntpclient.begin();
ntpclient.getTime("ntp.nict.jp", &unix_time);
```
If the return value of `getTime` function is negative, it means the acquisition has failed, so let's retry. If you are successful, all that's left to do is to call the `setTime` function with the Time library.
