## FIAPUploadAgent

FIAPUploadAgentはArduino Ethernet相当の環境でIEEE1888 Storageに対してWRITEを行うライブラリです．オリジナルのFIAPUploadAgentからフォークし，細かいバグ修正を行ったバージョンとしてPFFIAPUploadAgentがあります．この資料ではPFFIAPUploadAgentを使用します．

複数のポイントを`fiap_element`構造体で宣言することにより扱うことができます．

[LocalTimeLib](../localtime/README.md)と[Time](../time/README.md)に依存しているのでインストールしてください．

### 1. fiap_elementの宣言とインスタンス化

まず，インスタンス化を行います．
```C++
#include <PFFIAPUploadAgent.h>
FIAPUploadAgent fiap_upload_agent;
```

次にタイムゾーン情報が必要になるので，ローカルタイムゾーンを宣言します．この部分に関してはArduinoの基礎，[LocalTimeLib](../localtime/README.md)を参照してください．
```C++
#include <LocalTimeLib.h>
TimeZone localtime = { 9*60*60, 0, "+09:00" }; // offset, summer time, timezone string
```

次に，値文字列バッファの確保と`fiap_element`構造体の宣言を行います．FIAPUploadAgentは`fiap_element`構造体に設定された値文字列バッファの文字列を指定されたポイントIDに書き込みます．なおポイントIDはポストフィックスのみを定義します．実際には，あとで設定するプレフィックスと結合されたポイントIDを使用します．ポイントは複数個存在することが多いので，配列で宣言すると良いでしょう．
```C++
char temperature_str[16];
char voltage_str[16];
struct fiap_element fiap_elements [] = {
  { "Temperature", temperature_str, 0, &localtimezone, },
  { "Voltage", voltage_str, 0, &localtimezone, }, // postfix, buffer address, time = 0, timezone address
};
```

### 2. インスタンスの初期化と設定

まず，Ethernetライブラリを初期化しましょう．この場合はDHCPでIPアドレスを設定しますが他の方法を取りたい場合やEthernetライブラリについて深く知りたい場合は[公式リファレンス](https://www.arduino.cc/en/Reference/Ethernet)を参照してください．
```C++
byte mac_address[] = { 0xAA, 0xBB, 0xCC, 0xDD, 0xEE, 0xFF };
Ethernet.begin(mac_address);
```

続いてインスタンスの設定と初期化を行います．先程宣言した`fiap_upload_agent`に対して`begin`メソッドを呼び出します．引数は送信先のStorageのホスト名，Storageサービスのパス，ポート番号，そしてポイントIDのプレフィックスを設定します．StorageサービスのパスはリファレンスStorageを利用している場合はおそらく`/axis2/services/FIAPStorage`でしょう．ポイントIDのプレフィックス末尾にスラッシュを付けることを忘れないようにしましょう．
```C++
fiap_upload_agent.begin("fiap-dev.gutp.ic.i.u-tokyo.ac.jp", "/axis2/services/FIAPStorage", 80, "http://sample.j.kisarazu.ac.jp/sample-gw/");
```

### 3. アップロード

これで準備は完了です．実際にアップロードしてみましょう．ほぼこの部分のコードはイディオム的な扱いで良いと思います．現在の値文字列バッファの内容をtimeに入力されているUNIX時のタイムスタンプでアップロードします．このコードで`fiap_elements`配列内すべての`fiap_element`をアップロードします．
```C++
for(int i = 0; i < sizeof(fiap_elements)/sizeof(fiap_elements[0]); i++) {
  fiap_elements[i].time = now();
}

int ret = fiap_upload_agent.post(fiap_elements, sizeof(fiap_elements)/sizeof(fiap_elements[0]));

if(ret == 0) {
  Serial.println("done");
} else {
  Serial.println("failed");
}
```
