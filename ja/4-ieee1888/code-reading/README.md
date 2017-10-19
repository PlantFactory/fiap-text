## 解説 temp_upload

### 依存ライブラリ

解説は各ライブラリ毎に[Arduinoの基礎](../../3-arduino-fundamentals/README.md)で説明します．

* [PFFIAPUploadAgent](https://github.com/PlantFactory/PFFIAPUploadAgent)
* [Time](https://github.com/PaulStoffregen/Time)
* [LocalTimeLib](https://github.com/PlantFactory/LocalTimeLib)
* [SerialCLI](https://github.com/PlantFactory/SerialCLI)
* [NTP](https://github.com/PlantFactory/NTP)
* [ADT74x0](https://github.com/PlantFactory/ADT74x0)

### ソースコード解説

#### インスタンス宣言

Arduinoは利用できるRAMが2KBほどから8KBほどと，パソコンに比べ少ないです．また，処理能力もあまり高くないので，頻繁にインスタンスを作成，廃棄することはおすすめしません．

そこで，`temp_upload`ではグローバル変数としてインスタンスを生成しています．

SerialCLIの宣言，通信インターフェースはSerialを使います．
```C++
//cli
SerialCLI      commandline(Serial);
```

設定パラメータ類，解説は[M2Mゲートウェイからのアップロード](4-ieee1888/getting-started/README.md)を見てください．
```C++
//  ethernet
MacEntry       mac("MAC", "01:23:45:67:89:AB", "mac address");
//  ip
BoolEntry      dhcp("DHCP", "true", "DHCP enable/disable");
IPAddressEntry ip("IP", "192.168.0.2", "IP address");
IPAddressEntry gw("GW", "192.168.0.1", "default gateway IP address");
IPAddressEntry sm("SM", "255.255.255.0", "subnet mask");
IPAddressEntry dns_server("DNS", "8.8.8.8", "dns server");
//  ntp
StringEntry    ntp("NTP", "ntp.nict.jp", "ntp server");
//  fiap
StringEntry    host("HOST", "fiap-dev.gutp.ic.i.u-tokyo.ac.jp", "host of ieee1888 server end point");
IntegerEntry   port("PORT", "80", "port of ieee1888 server end point");
StringEntry    path("PATH", "/axis2/services/FIAPStorage", "path of ieee1888 server end point");
StringEntry    prefix("PREFIX", "http://taisyo.hongo.wide.ad.jp/MyHome/Node1/", "prefix of point id");
```

デバッグフラグ．１ならばデバッグモードになります．
```C++
//  debug
int debug = 0;
```

NTPとタイムゾーンの宣言．
```C++
//ntp
NTPClient ntpclient;
TimeZone localtimezone = { 9*60*60, 0, "+09:00" };
```

FIAPUploadAgentの宣言とエレメントの設定．FIAPUploadAgentはエレメント構造体の１番目の要素をプリフィックスの末尾に結合して，２番めの要素に設定されている文字列を送信します．送信する文字列のバッファは毎回再生成する必要はないので，グローバルで宣言します．送信しなければならないポイントを増やすには，エレメントを追加してください．
```C++
//fiap
FIAPUploadAgent fiap_upload_agent;
char temperature_str[16];
struct fiap_element fiap_elements [] = {
  { "Temperature", temperature_str, 0, &localtimezone, },
};
```

ADT74x0の宣言．
```C++
//sensor
ADT74x0 tempsensor;
```

デバッグ切り替え関数，SerialCLIから`debug`，`nodebug`受信時に実行されます．
```C++
void enable_debug()
{
  debug = 1;
}

void disable_debug()
{
  debug = 0;
}
```

#### setup関数

setup関数は起動時に１度だけ実行されます．

SerialCLIへ設定項目とデバッグコマンドを登録します．
```C++
void setup()
{
  int ret;
  commandline.add_entry(&mac);

  commandline.add_entry(&dhcp);
  commandline.add_entry(&ip);
  commandline.add_entry(&gw);
  commandline.add_entry(&sm);
  commandline.add_entry(&dns_server);

  commandline.add_entry(&ntp);

  commandline.add_entry(&host);
  commandline.add_entry(&port);
  commandline.add_entry(&path);
  commandline.add_entry(&prefix);

  commandline.add_command("debug", enable_debug);
  commandline.add_command("nodebug", disable_debug);

  commandline.begin(9600, "ADT74x0 Gateway");
```

ここではDHCPを利用する場合と利用しない場合をDHCPフラグで切り分けます．そこから先はArduinoの[Ethernetライブラリ](https://www.arduino.cc/en/Reference/Ethernet)の使い方を参照してください．
```C++
  // ethernet & ip connection
  if(dhcp.get_val() == 1){
    ret = Ethernet.begin(mac.get_val());
    if(ret == 0) {
      restart("Failed to configure Ethernet using DHCP", 10);
    }
  }else{
    Ethernet.begin(mac.get_val(), ip.get_val(), dns_server.get_val(), gw.get_val(), sm.get_val());
  }
```

NTPで時間を取得します．正常に取得できた場合，Timeライブラリに設定します．
```C++
  // fetch time
  uint32_t unix_time;
  ntpclient.begin();
  ret = ntpclient.getTime(ntp.get_val(), &unix_time);
  if(ret < 0){
    restart("Failed to configure time using NTP", 10);
  }
  setTime(unix_time);
```

FIAPUploadAgentの初期化です．
```C++
  // fiap
  fiap_upload_agent.begin(host.get_val(), path.get_val(), port.get_val(), prefix.get_val());
```

センサーの初期化です．
```C++
  // sensor
  Wire.begin();
  tempsensor.begin(0x48);
}
```

### loop関数

loop関数は，無限ループで実行されます．

`epoch`と`old_epoch`は`static`です．通常ローカル変数は関数呼び出し毎にリセットされますが，`static`がついているときは前回の値を記憶します．`commandline.process()`で溜まっているコマンドを処理します．
```C++
void loop()
{
  static unsigned long old_epoch = 0, epoch;

  commandline.process();
```

`epoch`を更新します．
```C++
  epoch = now();
```

DHCP利用時は定期的にアドレス情報の更新が必要です．
```C++
  if(dhcp.get_val() == 1){
    Ethernet.maintain();
  }
```

この条件式により，この`if`は１秒に１回だけ実行されます．
```C++
  if(epoch != old_epoch){
```

ADT74x0から温度を取り込み，送信用バッファーへ書き込みます．温度は`float`なので文字列化する必要があります．普通は`sprintf`を使いますが，Arduinoの`sprintf`は`float`非対応なので，`dtostrf`を使います．`dtostrf`についてはArduinoの基礎，[浮動小数点を文字列にする(dtostrf)](../../3-arduino-fundamentals/dtostrf/README.md)を参照してください．
```C++
    dtostrf(tempsensor.readTemperature(), -1, 2, temperature_str);
    debug_msg(temperature_str);
```

`epoch`には現在の秒が入っています．0から59までの間を取るので，この条件が成り立つのは毎分０秒のときです．つまり，１分に１回アップロードの処理を実行できるわけです．
```C++
    if(epoch % 60 == 0){
      debug_msg("uploading...");
```

アップロード処理です．エレメントとして`fiap_elements`配列に登録したすべてを送信します．`fiap_elements[i].time = epoch;`は書き込む値の時刻を現在の時刻に設定します．`epoch`ではなく`now()`を使うと，秒単位でズレが生じる可能性があるので注意しましょう．ちなみに`sizeof(array)/sizeof(array[0])`はイディオムで，配列の要素数を取得できます．
```C++
      for(int i = 0; i < sizeof(fiap_elements)/sizeof(fiap_elements[0]); i++){
        fiap_elements[i].time = epoch;
      }
      int ret = fiap_upload_agent.post(fiap_elements, sizeof(fiap_elements)/sizeof(fiap_elements[0]));
      if(ret == 0){
        debug_msg("done");
      }else{
        debug_msg("failed");
        Serial.println(ret);
      }
    }
```

最後に`old_epoch`を更新します．
```C++
  }

  old_epoch = epoch;
}
```

### 各種ユーティリティ関数

デバッグメッセージを出力します．
```C++
void debug_msg(String msg)
{
  if(debug == 1){
    Serial.print("[");
    print_time();
    Serial.print("]");
    Serial.println(msg);
  }
}
```

デバッグメッセージで時間を出力する部分．
```C++
void print_time()
{
  char print_time_buf[32];
  TimeElements* tm;
  tm = localtime();
  sprintf(print_time_buf, "%04d/%02d/%02d %02d:%02d:%02d",
      tm->Year + 1970, tm->Month, tm->Day, tm->Hour, tm->Minute, tm->Second);
  Serial.print(print_time_buf);
}
```

リスタート関数，再起動します．
```C++
void restart(String msg, int restart_minutes)
{
  Serial.println(msg);
  Serial.print("This system will restart after ");
  Serial.print(restart_minutes);
  Serial.print("minutes.");

  unsigned int start_ms = millis();
  while(1){
    commandline.process();
    if(millis() - start_ms > restart_minutes*60UL*1000UL){
      commandline.reboot();
    }
  }
}
```
