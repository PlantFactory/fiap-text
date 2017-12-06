## How to use serial communication (Serial)

This library may be the most helpful in using Arduino. It communicates with hosts and other devices using built-in UART. Some chips may have multiple serials. In that case, the asterisk of `Serial *` will change to interface number.

### Initialization

Let's initialize first. You can set the baud rate and parity here, please refer to the [official reference](https://www.arduino.cc/en/Serial/Begin) for details.
```C++
Serial.begin(9600); // Initialize serial to 9600bps
```

### Communication

After that is just communication.
```C++
Serial.print("AAA"); // No line break
Serial.println("BBB"); // Line break
// output> AAABBB
```

### Reduce RAM capacity

The AVR adopted by Arduino is the Harvard architecture, and the program ROM and RAM have independent buses. Since this value can not be accessed directly from the program ROM, it is necessary to temporarily copy it to RAM using a dedicated library. For such use, character strings normally present in the code are expanded into the RAM by the activation routine at startup. In other words, as the number of strings increases, RAM is exhausted even if it is not used.

Therefore, as mentioned above, you can use a library that copies data from program ROM to RAM, but it is troublesome to say clearly. That's why `Serial` has a `_F () `macro. If you use `Serial` directly as a string literal, rewrite and put the string in the program ROM as follows.
```C++
Serial.println("Hello"); // placed on RAM
Serial.println(_F("Hello")); // placed on ROM
```
