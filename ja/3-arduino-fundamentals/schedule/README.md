## n秒ごとに実行する

よくセンサーで計測をするアプリケーションでn秒周期でセンサーからデータを取得したい，といった要望があると思います．ここではTimeライブラリを使ったn秒ごとに実行するスケジューリングの手法を説明します．

### 1秒ごとに実行する

Arduinoは`setup`関数を実行した後，`loop`関数を無限ループします．一般的に`loop`関数をストップすることはよろしくないので回し続けます．ということは，n秒経過カウントは`delay`関数を使わずに行います．そこでキーポイントとなるのがTimeライブラリです．Timeライブラリでは現在の秒数を取得できます．１ループ前の秒数と比較して異なれば１秒がカウントアップしたタイミングと言えます．ここで処理を行えば１秒周期で処理を実行できます．

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

### 毎分n秒に実行する

１秒に１回実行できるようになったので，今度は毎分n秒になった時に実行するようにしましょう．単純に現在の秒数が0秒である時にのみ実行するようにすれば実現できますね．
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
ただ，この方法だと次の実行タイミングが処理時間より短い場合，スキップされ実行されませんので処理時間には常に気を配りましょう．１秒以内に終了することが望ましいですが…
