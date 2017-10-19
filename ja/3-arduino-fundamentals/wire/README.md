## I2C(Wire)

I2Cはたった２本の結線で最大１１２ノードが通信できるシリアルバスです．マイコン同士の通信やセンサーからの情報の読み出しに広く利用されます．通信は必ずマスターから開始され，指定されたスレーブのみが返信できる１対nのバスです．ArduinではI2CをWireライブラリから扱うことができます．

### I2Cマスター

自分がI2Cのマスターになり，スレーブのセンサーやマイコンと通信を行います．

必ず使用前に`Wire.begin()`を１回だけ実行して初期化を忘れないでください．

#### 一方的な書き込み

マスターからスレーブへ一方的に書き込みます．書き込みたいデータ量に合わせて`write`メソッドの呼び出し回数を増減してください．
```C++
byte address = 0x48;
Wire.beginTransmission(address);
Wire.write(0xAA);
Wire.write(0xBB);
Wire.endTransmission();
```

#### 一方的な読み込み

スレーブから一方的にデータを読み込みます．読み込みたいバイト数を最初にスレーブへ送信します．その後，スレーブからデータが送信されますので`Wire.read()`で受け取ります．
```C++
byte address = 0x48;
Wire.requestFrom(address, 2); // request 2byte

while(Wire.available()) {
  byte val = Wire.read();
}
```

#### 書き込んでから読み込み

マスターからデータを書き込んでから読み込みます．レジスタ番号を指定して読み込む場合などに使われます．
```C++
byte address = 0x48;
Wire.beginTransmission(address);
Wire.write(0xAA);
Wire.endTransmission(false);

Wire.requestFrom(address, 1, true);
byte val = Wire.read();
```

###　もっと詳しい情報

もっとわかりやすく書いてある記事

* [Arduinoを30分でI2C通信する - Qiita](https://qiita.com/MergeCells/items/20c3c1a0adfb222a19cd)
