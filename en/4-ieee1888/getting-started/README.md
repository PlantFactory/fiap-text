## Upload from M2M gateway

Let's create a gateway to upload the temperature acquired by ADT7410 as a sample.

Source code：[temp_upload](https://github.com/PlantFactory/temp_upload)

### Dependent Library

[Arduino fundamentals, using libraries](../../3-arduino-fundamentals/using-libraries/README.md) With reference to the link, please install the library.

* [PFFIAPUploadAgent](https://github.com/PlantFactory/PFFIAPUploadAgent)
* [Time](https://github.com/PaulStoffregen/Time)
* [LocalTimeLib](https://github.com/PlantFactory/LocalTimeLib)
* [SerialCLI](https://github.com/PlantFactory/SerialCLI)
* [NTP](https://github.com/PlantFactory/NTP)
* [ADT74x0](https://github.com/PlantFactory/ADT74x0)


### Initial setting

From line 21 until line 35 is the establishing connection data.

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

* `mac`must specify the MAC adrdress of the M2M gateway.
  * Please note that it's depends on the board.
  * If there's device with same MAC address in same segment, it will be hard work
* When `dhcp` is 'true', IP address is done using dhcp
* `ip` is set as the IP address of GW when `dhcp` is 'false'
* `gw` id set as default gateway address of GW when `dhcp` is 'false'
* `sm` is set as subnet.mask of GW when `dhcp` is 'false'
* `dns_server` is set as DNS server used by GW when `dhcp` is 'false'
* `ntp` specifies the NTP server referred by GW
* `host` specifies the address of the Storage where GW WRITE the data
* `port` specifies the port of the Storage where GW WRITE the data
* `path` specifies the service path of the Storage where GW WRITE the data
* `prefix` represents point ID prefix of the GW
  * It is similar to the dirname command in linux.
  * `http://ujyu.net/sample_gw/temperature`
    * Prefix：`http://ujyu.net/sample_gw/`

Basically, it would not be a problem if you establish `mac`, `host`, `prefix`

### Hardware

Use the circuit created in [Arduino fundamentals ，ADT74x0]() as it is.

### Test

Let's connect the LAN cable after checking the setting, writing and sensor connection. If you are using GUTP's Reference Implementation Storage, you can probably view the measured values by opening the Storage URL in a browser.
