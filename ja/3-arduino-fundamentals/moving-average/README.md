## 移動平均を取る

詳しい移動平均の説明は[Wikipedia](https://ja.wikipedia.org/wiki/%E7%A7%BB%E5%8B%95%E5%B9%B3%E5%9D%87)が詳しいのでそちらを参照してください．

### SMAライブラリ

単純移動平均(Simple Moving Average)を求めるライブラリです．

* [Arduino SMA](https://github.com/ainehanta/arduino-sma)

ライブラリのインストール方法はArduinoの基礎，[ライブラリを使用する](../using-libraries/README.md)を参照してください．

### SMAライブラリの使い方

このライブラリは移動平均を保存する数を指定できます．インスタンス化の際にコンストラクタへ保存数を渡すことができます．デフォルトは10です．
```C++
#include <SimpleMovingAverage.h>
SimpleMovingAverage avg; // 10
SimpleMovingAverage avg5(5); // 5
```

使用前にかならず`begin`関数を呼び出してください．
```C++
avg.begin();
```

あとは，インスタンスに対して`update`を呼び出すことで値の更新と平均値の取得ができます．
```C++
avg.update(10); // 10
avg.update(11); // 10.5
avg.update(12); // 11
```

値の更新を行わずに現在の平均値を取得したい場合は`average`関数が使えます．
