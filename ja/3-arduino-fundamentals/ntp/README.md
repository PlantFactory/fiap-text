## NTPの使い方

NTPで時刻を取得します．Ethernetライブラリが使用できることが前提となります．まずインスタンス化しましょう．
```C++
#include <NTP.h>

NTPClient ntpclient;
```

Ethernetライブラリの初期化や使い方に関しては[公式リファレンス](https://www.arduino.cc/en/Reference/Ethernet)を確認してください．また，一般的なDHCPでIPアドレスを設定するコードは次のとおりです．
```C++
#include <Ethernet.h>

byte mac[] = {
  0x00, 0xAA, 0xBB, 0xCC, 0xDE, 0x02
};

Ethernet.begin(mac);
```

実際にNTPで通信し，時刻情報を取得します．
```C++
uint32_t unix_time = 0;

ntpclient.begin();
ntpclient.getTime("ntp.nict.jp", &unix_time);
```
`getTime`関数の戻り値が負であれば取得に失敗していますので，リトライしてみましょう．成功してるならば，あとはTimeライブラリなりの`setTime`関数を呼び出すだけです．
