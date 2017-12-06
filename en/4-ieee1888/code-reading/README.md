## Explanation of temp_upload

### Dependent Library

Explanation for each library is in [Arduino fundamentals](../../3-arduino-fundamentals/README.md)

* [PFFIAPUploadAgent](https://github.com/PlantFactory/PFFIAPUploadAgent)
* [Time](https://github.com/PaulStoffregen/Time)
* [LocalTimeLib](https://github.com/PlantFactory/LocalTimeLib)
* [SerialCLI](https://github.com/PlantFactory/SerialCLI)
* [NTP](https://github.com/PlantFactory/NTP)
* [ADT74x0](https://github.com/PlantFactory/ADT74x0)

### Source code description

#### Instance declaration

Arduino has RAM that is capable about 2KB to 8KB，which is less than a laptop. Also, as the processing power is not so high, we do not recommend creating or discarding instances frequently.
Therefore, `temp_upload` creates an instance as a global variable．

SerialCLI declaration, communication interface uses Serial.
```C++
//cli
SerialCLI      commandline(Serial);
```

See [Upload from M2M Gateway](4-ieee1888/getting-started/README.md)for configuration parameters and explaination.
```C++
//  ethernet
MacEntry       mac("MAC", "01:23:45:67:89:AB", "mac address");
//  ip
BoolEntry      dhcp("DHCP", "true", "DHCP enable/disable");
IPAddressEntry ip("IP", "192.168.0.2", "IP address");
IPAddressEntry gw("GW", "192.168.0.1", "default gateway IP address");
IPAddressEntry sm("SM", "255.255.255.0", "subnet mask");
IPAddressEntry dns_server("DNS", "8.8.8.8", "dns server");
//  ntp
StringEntry    ntp("NTP", "ntp.nict.jp", "ntp server");
//  fiap
StringEntry    host("HOST", "fiap-dev.gutp.ic.i.u-tokyo.ac.jp", "host of ieee1888 server end point");
IntegerEntry   port("PORT", "80", "port of ieee1888 server end point");
StringEntry    path("PATH", "/axis2/services/FIAPStorage", "path of ieee1888 server end point");
StringEntry    prefix("PREFIX", "http://taisyo.hongo.wide.ad.jp/MyHome/Node1/", "prefix of point id");
```

Debug flag. If it is 1, it will be in debug mode.
```C++
//  debug
int debug = 0;
```

NTP and time zone declaration.
```C++
//ntp
NTPClient ntpclient;
TimeZone localtimezone = { 9*60*60, 0, "+09:00" };
```

Declaration of FIAPUploadAgent and setting of element.
FIAPUploadAgent combines the first element of the element structure at the end of the prefix, send the character string set for the second element. Since it is not necessary to regenerate the buffer of the character string to be sent every time, declare it globally. To increase the number of points you need to send, add an element.
```C++
//fiap
FIAPUploadAgent fiap_upload_agent;
char temperature_str[16];
struct fiap_element fiap_elements [] = {
  { "Temperature", temperature_str, 0, &localtimezone, },
};
```

Declaration of ADT74x0
```C++
//sensor
ADT74x0 tempsensor;
```

It is executed when debug switching function, upon reception of `debug`, `nodebug` from SerialCLI.
```C++
void enable_debug()
{
  debug = 1;
}

void disable_debug()
{
  debug = 0;
}
```

#### setup function

setup function runs only once at startup.

Register setting items and debug commands to Serial CLI.
```C++
void setup()
{
  int ret;
  commandline.add_entry(&mac);

  commandline.add_entry(&dhcp);
  commandline.add_entry(&ip);
  commandline.add_entry(&gw);
  commandline.add_entry(&sm);
  commandline.add_entry(&dns_server);

  commandline.add_entry(&ntp);

  commandline.add_entry(&host);
  commandline.add_entry(&port);
  commandline.add_entry(&path);
  commandline.add_entry(&prefix);

  commandline.add_command("debug", enable_debug);
  commandline.add_command("nodebug", disable_debug);

  commandline.begin(9600, "ADT74x0 Gateway");
```

In this case, we will use DHCP flag to distinguish whether DHCP is used or not. From that point forward, please refer to how to use Arduino's [Ethernet library](https://www.arduino.cc/en/Reference/Ethernet).
```C++
  // ethernet & ip connection
  if(dhcp.get_val() == 1){
    ret = Ethernet.begin(mac.get_val());
    if(ret == 0) {
      restart("Failed to configure Ethernet using DHCP", 10);
    }
  }else{
    Ethernet.begin(mac.get_val(), ip.get_val(), dns_server.get_val(), gw.get_val(), sm.get_val());
  }
```

Acquire the time with NTP. After acquiring, set it to the Time library．
```C++
  // fetch time
  uint32_t unix_time;
  ntpclient.begin();
  ret = ntpclient.getTime(ntp.get_val(), &unix_time);
  if(ret < 0){
    restart("Failed to configure time using NTP", 10);
  }
  setTime(unix_time);
```

FIAPUploadAgent Initialization
```C++
  // fiap
  fiap_upload_agent.begin(host.get_val(), path.get_val(), port.get_val(), prefix.get_val());
```

Sensor Initialization
```C++
  // sensor
  Wire.begin();
  tempsensor.begin(0x48);
}
```

### loop function

loop function is executed in an infinite loop.

`epoch` and `old_epoch` are `static`. Normally, local variables are reset for each function call，but if `static` is attached, the previous value is memorized.`commandline.process ()` will process the stored commands.
```C++
void loop()
{
  static unsigned long old_epoch = 0, epoch;

  commandline.process();
```

Update `epoch`.
```C++
  epoch = now();
```

When using DHCP, it is necessary to update the address information on a regular basis.
```C++
  if(dhcp.get_val() == 1){
    Ethernet.maintain();
  }
```

With this conditional expression, this `if` is executed only once per second.
```C++
  if(epoch != old_epoch){
```

Temperature is fetched from ADT74x0 and written to transmission buffer. Since the temperature is `float`, it must be string.Usually we use `sprintf`, but Arduino`sprintf` is not compatible with `float`, so use`dtostrf`.For `dtostrf` refer to Arduino fundamentals，[Make float decimal point to string (dtostrf)](../../3-arduino-fundamentals/dtostrf/README.md)．
    dtostrf(tempsensor.readTemperature(), -1, 2, temperature_str);
    debug_msg(temperature_str);
```

`epoch` contains the current second. Since it takes between 0 and 59, this condition holds at 0 seconds per minute. In other words, upload processing can be executed once a minute.
```C++
    if(epoch % 60 == 0){
      debug_msg("uploading...");
```

It is an upload process. Send everything registered in the `fiap_elements` array as an element.`fiap_elements [i] .time = epoch;` sets the time of the value to be written to the current time.Note that using `now ()` instead of `epoch` may cause a deviation occurs in seconds.By the way, `sizeof (array) / sizeof (array [0])` is an idiom that can get the number of elements of an array.
```C++
      for(int i = 0; i < sizeof(fiap_elements)/sizeof(fiap_elements[0]); i++){
        fiap_elements[i].time = epoch;
      }
      int ret = fiap_upload_agent.post(fiap_elements, sizeof(fiap_elements)/sizeof(fiap_elements[0]));
      if(ret == 0){
        debug_msg("done");
      }else{
        debug_msg("failed");
        Serial.println(ret);
      }
    }
```

Lastly update `old_epoch`.
```C++
  }

  old_epoch = epoch;
}
```

### Various utility functions

Output debug messages.
```C++
void debug_msg(String msg)
{
  if(debug == 1){
    Serial.print("[");
    print_time();
    Serial.print("]");
    Serial.println(msg);
  }
}
```

Output time with debug messages.
```C++
void print_time()
{
  char print_time_buf[32];
  TimeElements* tm;
  tm = localtime();
  sprintf(print_time_buf, "%04d/%02d/%02d %02d:%02d:%02d",
      tm->Year + 1970, tm->Month, tm->Day, tm->Hour, tm->Minute, tm->Second);
  Serial.print(print_time_buf);
}
```

Restart function.
```C++
void restart(String msg, int restart_minutes)
{
  Serial.println(msg);
  Serial.print("This system will restart after ");
  Serial.print(restart_minutes);
  Serial.print("minutes.");

  unsigned int start_ms = millis();
  while(1){
    commandline.process();
    if(millis() - start_ms > restart_minutes*60UL*1000UL){
      commandline.reboot();
    }
  }
}
```
