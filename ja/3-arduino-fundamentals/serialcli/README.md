## SerialCLI(シリアルコンソール)

SerialCLIはシリアルコンソールでコマンドライン・インタフェースを提供するライブラリです．Arduinoの設定値をプログラムの再コンパイル，再書き込みで変更するのはめんどくさいですよね．SerialCLIを使えばシリアルターミナルでArduinoに接続して`conf`→`<entry>=<value>`→`save`で終わり！任意のパラメーターを保存できます．それ以外にもオリジナルのコマンドを定義して拡張することもできます．

対応しているデータ型は以下のとおりです．
* String
* Boolean
* Integer
* IP Address
* MAC Address

### 1. インスタンス化とエントリーの設定

まずSerialCLIクラスをインスタンス化しましょう．引数として使用したいシリアルインターフェースのクラスを渡してください．普通UnoなどのArduinoはシリアルインターフェースは１つしか持ちませんが，Megaなど複数のシリアルインターフェースをもつArduinoではSerial2のように渡すこともできます．
```C++
#include <SerialCLI.h>

SerialCLI commandline(Serial);
```

続いて，エントリーを設定しましょう．それぞれ`Entry(<エントリー名>, <初期値>, <エントリーの説明>)`です．
```C++
MacEntry mac("MAC", "AA:BB:CC:DD:EE:FF", "mac address");
BoolEntry dhcp("DHCP", "true", "DHCP enable/disable");
IPAddressEntry ip("IP", "192.168.13.127", "IP address");
StringEntry host("HOST", "www.kisarazu.ac.jp", "hostname");
```

最後は拡張コマンドのハンドラの宣言です．任意のコマンド文字列で指定の関数を呼び出すことができます．引数，戻り値なしのグローバル関数を定義してください．
```C++
void enable_debug()
{
  Serial.println("Debug enabled.");
}

void disable_debug()
{
  Serial.println("Debug disabled.");
}
```

### 2. エントリーとコマンドの登録

次はコマンドライン・インタフェースにエントリーとコマンドを登録します．エントリーの登録には`add_entry`メソッドにエントリーのインスタンスをポインタで渡します．コマンドの追加には`add_command`メソッドにコマンド文字列と呼び出すハンドラを渡します．
```C++
commandline.add_entry(&mac);
commandline.add_entry(&dhcp);
commandline.add_entry(&ip);
commandline.add_entry(&host);

commandline.add_command("debug", enable_debug);
commandline.add_command("nodebug", disable_debug);
```

### 3. 開始

事前準備は完了しました．あとはボーレートと起動時のメッセージを指定して起動します．
```C++
commandline.begin(9600, "SerialCLI Sample");
```

### 4. イベントの処理

最後に`loop`関数でループ毎に`process`関数を呼び出して処理を必ず行ってください．
```C++
void loop() {
  commandline.process();
}
```

### 5. 値の取得

値をエントリーから取り出すには`get_val`メソッドを呼び出します．
```C++
Serial.println(host.get_val());
```

### 使い方

シリアルターミナルで接続するとこのようなメッセージが出力されます．
```
SerialCLI Sample
SerialCLI Ver.0.01
current setting
    MAC=00:00:00:00:00:00
    DHCP=false
    IP=0.0.0.0
    HOST=
setting description (and default value)
    MAC... mac address(default AA:BB:CC:DD:EE:FF)
    DHCP... DHCP enable/disable(default true)
    IP... IP address(default 192.168.13.127)
    HOST... hostname(default www.kisarazu.ac.jp)
commands
    help, show, conf, exit, load [default], save,
    debug, nodebug,
>
```
すべての登録したエントリーとコマンドが表示されていると思います．試しに`show`と入力してみましょう．
```
>show
    MAC=00:00:00:00:00:00
    DHCP=false
    IP=0.0.0.0
    HOST=
```
なにも設定されていないと思います．この状態では初期値が使われます．では，値を競ってしてみましょう．まず`conf`と入力し設定モードへ入りましょう．
```
>conf
enter conf mode
#
```
プロンプトが変化したと思います．この状態ですべてのエントリーの設定ができます．エントリーを設定するには`<エントリー名>=<値>`と入力します．
```C++
#HOST=ujyu.net
#
```
では，確認しましょう．
```
#show
    MAC=00:00:00:00:00:00
    DHCP=false
    IP=0.0.0.0
    HOST=ujyu.net
#
```
設定されていると思います．このままでは保存されていませんので`save`と入力して保存しましょう．
```
#save
saving...done.
#
```
これで設定値がEEPROMに書き込まれました．あとは`reboot`で再起動して終了です．

初期値をロードしたい場合は`load default`と入力し`save`です．

拡張コマンドは呼び出し文字列を入力すれば呼び出されます．
```
>debug
debug
Debug enabled.
>
>nodebug
debug
nodebug
Debug disabled.
>
```

### ソースコード

```C++
#include <SerialCLI.h>

SerialCLI commandline(Serial);

MacEntry mac("MAC", "AA:BB:CC:DD:EE:FF", "mac address");
BoolEntry dhcp("DHCP", "true", "DHCP enable/disable");
IPAddressEntry ip("IP", "192.168.13.127", "IP address");
StringEntry host("HOST", "www.kisarazu.ac.jp", "hostname");

void enable_debug()
{
  Serial.println("Debug enabled.");
}

void disable_debug()
{
  Serial.println("Debug disabled.");
}

void setup() {
  commandline.add_entry(&mac);
  commandline.add_entry(&dhcp);
  commandline.add_entry(&ip);
  commandline.add_entry(&host);

  commandline.add_command("debug", enable_debug);
  commandline.add_command("nodebug", disable_debug);

  commandline.begin(9600, "SerialCLI Sample");

  Serial.println(host.get_val());
}

void loop() {
  commandline.process();
}
```
