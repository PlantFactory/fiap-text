## Convert a floating point into a character string (dtostrf)

Generally,  `sprintf` is used to convert numbers such as floating points (float, double) to strings. However, as `printf` functions are large in scale, it is often unsuitable for microcontrollers such as Arduino. In fact, using `sprintf` on Arduino, `%f` will return `?`.
```C++
char str[10];
sprintf(str, "%f", 10.8f);
// str == "?"
```

Instead, `dtostrf` provides an avr-libcfunction.
```C++
char str[10];
sprintf(10.8f, -1, 2, str);
// str == "10.80"
```
The parameters are, `dtostrf(<number>, <right align>, <number of decimal places>, <string name>)`. Character string that are set to right align will be aligned to the right, and filled in with spaces to fulfill the total number of characters defined in the `right align` parameter. Numbers, periods and symbols all are counted as characters. `number of decimal places` is the number of digits to display after the decimal point. If right alignment is unnecessary, then set it as -1. This will make the string left aligned.
