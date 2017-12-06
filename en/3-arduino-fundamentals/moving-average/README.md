## Moving Average

For a detailed explanation of moving average, please refer to this [Wikipedia](https://en.wikipedia.org/wiki/Moving_average) page.

### SMA Library

Simple Moving Average library

* [Arduino SMA](https://github.com/ainehanta/arduino-sma)

Please refer to [using libraries](../using-libraries/README.md) under arduino fundamentals to learn how to install libraries.

### How to use the SMA library

This library allows you to specify the number of data points to use for the moving average. You can pass the  number of data to save to the constructor at the time of instantiation. The default is 10.
```C++
#include <SimpleMovingAverage.h>
SimpleMovingAverage avg; // 10
SimpleMovingAverage avg5(5); // 5
```

Be sure to call the `begin` function before using it.
```C++
avg.begin();
```

After that you can update the value and get the average by calling `update` on the instance.
```C++
avg.update(10); // 10
avg.update(11); // 10.5
avg.update(12); // 11
```

You can use the `average` function if you want to get the current average without updating the value.
