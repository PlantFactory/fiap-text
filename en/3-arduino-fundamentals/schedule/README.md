## Run every n seconds

In an application that regularly measures with a sensor, you would want to acquire data from the sensor every n seconds. Here, we will explain the scheduling method for executing every n seconds using the Time library.

### Run every second

Arduino loops the `loop` function indefinitely after executing the` setup` function. In general it is not good to stop the `loop` function so we will let it continue looping. That means that n seconds elapsed counts are done without using the `delay` function. The key point to doing this is the Time library. The Time library can get the current number of seconds. １ループ前の秒数と比較して異なれば１秒がカウントアップしたタイミングと言えます．Processing can be executed in one second cycles if processing is done here.

```C++
#include <Time.h>

void loop() {
  static time_t old_epoch = 0, epoch;

  epoch = now();

  if(epoch != old_epoch) {
    // runs at once in every second
  }
}
```

### Run every minute n seconds

Now you can run it once per second, so let's run it every minute n seconds. It can be done simply by executing it only when the current seconds number is 0 second.
```C++
#include <Time.h>

void loop() {
  static time_t old_epoch = 0, epoch;

  epoch = now();

  if(epoch != old_epoch) {
    // runs at once in every second
    if(epoch % 60 == 0) {
      // runs at once in every minute
    }
  }
}
```
However, if this method is used, if the next execution timing is shorter than the processing time, skip it and it will not be executed, so always be concerned about processing time. It is desirable to end within 1 second
