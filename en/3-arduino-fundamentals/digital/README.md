## Digital input / output

Digital input / output is the most fundamental operation when using Arduino. On the input pin, HIGH or LOW is determined by the pin voltage. For output, any pin can be set to output HIGH (power supply voltage) or LOW (0 V).

### Digital output

We will digitally output from arduino to any pin. In order to digitally output from a pin, it is necessary to set the pin to output mode. By the way, analog pins (A0 to A5) can also be used as digital output pins. Depending on the circumstances, if you do not need to change it ,we recommend setting it in the `setup` function.
```C++
pinMode(13, OUTPUT); // pin13を出力にする
pinMode(A0, OUTPUT); // pinA0を出力にする
```

All that's left to do now is digital output. Please ensure that you do not do output to a pin that has been set up as an input pin as this will have a different meaning, and will not work as an output.
```C++
digitalWrite(13, HIGH); // 5V is output from pin 13
digitalWrite(13, LOW);  // 0V is output from pin 13
```

### Digital input

Digital input can be performed in ways such as pushing a button. If the voltage at the pin is around 5V, the input will be HIGH, and if the voltage is near 0V, the input will be LOW. Firstly, set the pin to input mode as shown below.
```C++
pinMode(13, INPUT);
```

Let's get the input value.
```C++
uint8_t ret;
ret = digitalRead(13);
```

When reading a device such as a button that requires a pull up resistor, the  internal pull up resistor can be used. It is possible to further reduce the number of external parts used, even if by only 1 resistor. Just set the mode of the pin you want to pull up to `INPUT_PULLUP`.
```C++
pinMode(13, INPUT_PULLUP);
digitalRead(13);
```
