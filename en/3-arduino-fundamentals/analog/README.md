## Analog output
### Analog input

Analong data such brightness or temperature is read from the input voltage. For microcontrollers such as Arduino, use AD (analog / digital) converter. The AD converter compares the input voltage with a reference voltage and outputs the ratio from the reference voltage. In the AD converter, the number of convertible bits is fixed. Writing the pseudocode AD converter looks like this.
```C++
const uint16_t maxValue = 1023; // 10bit -> 0 ~ 1023
float inputVoltage = 3.1415;
float referenceVoltage = 5.001;

int digitalValue = inputVoltage / referenceVoltage * 1023; // 643
```
By the way, it is easy to return this digital value to the voltage again.
```C++
const uint16_t maxValue = 1023; // 10bit -> 0 ~ 1023
float referenceVoltage = 5.001;
int digitalValue = 643;

float analogVoltage = 643 / maxValue * referenceVoltage; // 3.1433460411
```

To do an analog input, simply input a voltage from Arduino's A0 to A5 pin and read it with the `analogRead` function.
```C++
uint16_t val = analogRead(A0);
```

### Analog output

In reverse, converting a digital value to analog output is done using a DA(digital / analog) converter. There are many ways of doing this, for the Arduino, PWM(Pulse Width Modulation) can be used. PWM expresses the desired voltage by switching between ON・OFF at a very high speed in a constant cycle. In the long term, if ON time is longer within each cycle, the apparent output voltage will be higher. Likewise, if the OFF time is longer within each cycle, the apparent output voltage will be lower. This ratio of ON time to OFF time is called the duty ratio. Arduino's PWM uses an 8 bit DA converter. However, only pins number 3, 5, 6, 9, 10 and 11 can output PWM. Now, let's set the pin and duty ratio to the `analogWrite` function. The duty ratio at 0 represents 0[V], and at 255 is the power supply voltage.
```C++
analogWrite(3, 255);
```

Strictly speaking, PWM is a digital signal. Devices such as lights may experience high frequency flickering. To bring this closer to the analog value, please incorporate a smoothing circuit with a capacitor and a resistor.
