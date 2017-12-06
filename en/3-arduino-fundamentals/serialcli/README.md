## Serial CLI (serial console)

SerialCLI is a library that provides a command line interface with serial console. It is bothersome to change Arduino's setting by recompiling and rewriting the program. SerialCLI connects to Arduino at the serial terminal and ends with `conf` ->`<entry> = <value> `->`save`! You can save arbitrary parameters. Other than that, you can also define and extend the original command.

The corresponding data types are as follows.
* String
* Boolean
* Integer
* IP Address
* MAC Address

### 1. Instantiation and entry settings

First, let's instantiate the SerialCLI class. Pass the class of the serial interface you want to use as an argument. Usually Arduino such as Uno has only one serial interface, but Arduino with multiple serial interfaces such as Mega can also pass like Serial 2.
```C++
#include <SerialCLI.h>

SerialCLI commandline(Serial);
```

Let's set up an entry next. `Entry (<entry name>, <initial value>, <description of entry>)` respectively.
```C++
MacEntry mac("MAC", "AA:BB:CC:DD:EE:FF", "mac address");
BoolEntry dhcp("DHCP", "true", "DHCP enable/disable");
IPAddressEntry ip("IP", "192.168.13.127", "IP address");
StringEntry host("HOST", "www.kisarazu.ac.jp", "hostname");
```

Lastly is the declaration of the extension command handler. You can call the specified function with any command string. Please define global function without argument and return value.
```C++
void enable_debug()
{
  Serial.println("Debug enabled.");
}

void disable_debug()
{
  Serial.println("Debug disabled.");
}
```

### 2. Entry and command registration

Next we will register entries and commands in the command line interface. To register an entry, pass the instance of the entry as a pointer to the `add_entry` method. To add a command, pass the command string and the handler to call to the `add_command` method.
```C++
commandline.add_entry(&mac);
commandline.add_entry(&dhcp);
commandline.add_entry(&ip);
commandline.add_entry(&host);

commandline.add_command("debug", enable_debug);
commandline.add_command("nodebug", disable_debug);
```

### 3. Start

Advance preparation is completed. After that, specify the baud rate and the message at startup and start it.
```C++
commandline.begin(9600, "SerialCLI Sample");
```

### 4. Event handling

Finally call the `process` function for each loop with the `loop` function to perform processing.
```C++
void loop() {
  commandline.process();
}
```

### 5. Get value

To retrieve the value from the entry, call the `get_val` method.
```C++
Serial.println(host.get_val());
```

### How to use

Such a message is output when connecting with a serial terminal.
```
SerialCLI Sample
SerialCLI Ver.0.01
current setting
    MAC=00:00:00:00:00:00
    DHCP=false
    IP=0.0.0.0
    HOST=
setting description (and default value)
    MAC... mac address(default AA:BB:CC:DD:EE:FF)
    DHCP... DHCP enable/disable(default true)
    IP... IP address(default 192.168.13.127)
    HOST... hostname(default www.kisarazu.ac.jp)
commands
    help, show, conf, exit, load [default], save,
    debug, nodebug,
>
```
All registered entries and commands are displayed. Let's type `show` as a test.
```
>show
    MAC=00:00:00:00:00:00
    DHCP=false
    IP=0.0.0.0
    HOST=
```
Nothing is set. The initial value is used in this state. Let's compete the values. Let's start by entering `conf` and setting mode.
```
>conf
enter conf mode
#
```
The prompt has changed. In this state all entries can be set. To set up an entry, enter `<entry name> = <value>`.
```C++
#HOST=ujyu.net
#
```
Let's confirm.
```
#show
    MAC=00:00:00:00:00:00
    DHCP=false
    IP=0.0.0.0
    HOST=ujyu.net
#
```
Since it is not saved as it is, enter `save` and save it.
```
#save
saving...done.
#
```
The set value is now written to the EEPROM. Restart with `reboot` and finish.

To load the initial value, type `load default` and it is `save`.

Extended commands are invoked by entering the calling string.
```
>debug
debug
Debug enabled.
>
>nodebug
debug
nodebug
Debug disabled.
>
```

### Source code

```C++
#include <SerialCLI.h>

SerialCLI commandline(Serial);

MacEntry mac("MAC", "AA:BB:CC:DD:EE:FF", "mac address");
BoolEntry dhcp("DHCP", "true", "DHCP enable/disable");
IPAddressEntry ip("IP", "192.168.13.127", "IP address");
StringEntry host("HOST", "www.kisarazu.ac.jp", "hostname");

void enable_debug()
{
  Serial.println("Debug enabled.");
}

void disable_debug()
{
  Serial.println("Debug disabled.");
}

void setup() {
  commandline.add_entry(&mac);
  commandline.add_entry(&dhcp);
  commandline.add_entry(&ip);
  commandline.add_entry(&host);

  commandline.add_command("debug", enable_debug);
  commandline.add_command("nodebug", disable_debug);

  commandline.begin(9600, "SerialCLI Sample");

  Serial.println(host.get_val());
}

void loop() {
  commandline.process();
}
```
