## I2C(Wire)

I2C is a serial bus that can communicate up to 112 nodes with just 2 wires. It is widely used for communications between microcomputers and reading of information from sensors. Communication is always a one-to-many bus that starts from the master and only the designated slave can reply. Arduino can handle I2C from the Wire library.

### I2C master

Become the master of I2C and communicate with slave sensors and microcomputers.

Be sure to execute `Wire.begin()` only once before using and do not forget to initialize it.

#### Unilateral write

Write unilaterally from the master to the slave. Increase / decrease the number of calls to the `write` method according to the amount of data you want to write.
```C++
byte address = 0x48;
Wire.beginTransmission(address);
Wire.write(0xAA);
Wire.write(0xBB);
Wire.endTransmission();
```

#### Unilateral reading

Data is unilaterally read from the slave. The number of bytes to be read is first sent to the slave. After that, data will be sent from the slave, so we will receive it with `Wire.read ()`.
```C++
byte address = 0x48;
Wire.requestFrom(address, 2); // request 2byte

while(Wire.available()) {
  byte val = Wire.read();
}
```

#### Read after writing
Read data from the master after writing data. It is used when specifying register number and reading.
```C++
byte address = 0x48;
Wire.beginTransmission(address);
Wire.write(0xAA);
Wire.endTransmission(false);

Wire.requestFrom(address, 1, true);
byte val = Wire.read();
```

###　More information

More descriptive articles

* [Communicate Arduino in I2C in 30 minutes - Qiita](https://qiita.com/MergeCells/items/20c3c1a0adfb222a19cd)
