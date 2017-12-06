## FIAPUploadAgent

FIAPUploadAgent is a library that performs WRITE on IEEE 1888 Storage in an environment equivalent to Arduino Ethernet. There is a version of this library called PFFIAPUploadAgent. Its is  forked from the original FIAPUploadAgent, and contains minor bug fixes. This document uses PFFIAPUploadAgent.

`fiap_element` structure is able to handle declaration of multiple point.

It is dependant on [LocalTimeLib](../localtime/README.md) and [Time](../time/README.md) libraries, so please install them.

### 1. Declaration and instantiation of fiap_element

First, we will instantiate it.
```C++
#include <PFFIAPUploadAgent.h>
FIAPUploadAgent fiap_upload_agent;
```

Next, time zone information is needed, so we will declare the local time zone. Regarding this part, please refer to [LocalTimeLib](../localtime/README.md) under arduino fundamentals.
```C++
#include <LocalTimeLib.h>
TimeZone localtime = { 9*60*60, 0, "+09:00" }; // offset, summer time, timezone string
```

Next, we allocate a value string buffer and declare `fiap_element` structure. FIAPUploadAgent writes the string in the value string buffer set in the `fiap_element` structure to the specified point ID. The point ID only defines the postfix. In practice, we will use the point ID combined with the prefix we set later. Since there are often multiple points, it is better to declare it as an array.
```C++
char temperature_str[16];
char voltage_str[16];
struct fiap_element fiap_elements [] = {
  { "Temperature", temperature_str, 0, &localtimezone, },
  { "Voltage", voltage_str, 0, &localtimezone, }, // postfix, buffer address, time = 0, timezone address
};
```

### 2. Initializing and setting the instance

First let's initialize the Ethernet library. In this case, the IP address is set by DHCP. If you want to use other methods or want to know more about the Ethernet library, please refer to the [official reference](https://www.arduino.cc/en/Reference/Ethernet).
```C++
byte mac_address[] = { 0xAA, 0xBB, 0xCC, 0xDD, 0xEE, 0xFF };
Ethernet.begin(mac_address);
```

Next, set up and initialize the instance. Call up the `begin` method for` fiap_upload_agent` declared earlier. The argument sets the prefix of the destination storage's host name, storage service path, port number, and point ID. If you are using Reference Storage, then the path of the Storage service is probably `/ axis2 / services / FIAPStorage` . Do not forget to add a slash at the end of the prefix of the point ID.
```C++
fiap_upload_agent.begin("fiap-dev.gutp.ic.i.u-tokyo.ac.jp", "/axis2/services/FIAPStorage", 80, "http://sample.j.kisarazu.ac.jp/sample-gw/");
```

### 3. Upload

Now, the preparation is complete. Let's actually upload. This part of the code should be able to be handled mostly idiomatically. Current value uploads the contents of the character string buffer with the UNIX time stamp entered for time. This code uploads all `fiap_element` in `fiap_elements` array.
```C++
for(int i = 0; i < sizeof(fiap_elements)/sizeof(fiap_elements[0]); i++) {
  fiap_elements[i].time = now();
}

int ret = fiap_upload_agent.post(fiap_elements, sizeof(fiap_elements)/sizeof(fiap_elements[0]));

if(ret == 0) {
  Serial.println("done");
} else {
  Serial.println("failed");
}
```
