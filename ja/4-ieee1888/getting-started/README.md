## M2Mゲートウェイからアップロード

サンプルとしてADT7410で取得した温度をアップロードするゲートウェイを作成しましょう．

ソースコード：[temp_upload](https://github.com/PlantFactory/temp_upload)

### 依存ライブラリ

[Arduinoの基礎，ライブラリの使用](../../3-arduino-fundamentals/using-libraries/README.md)を参考に以下のライブラリをインストールしてください．

* [PFFIAPUploadAgent](https://github.com/PlantFactory/PFFIAPUploadAgent)
* [Time](https://github.com/PaulStoffregen/Time)
* [LocalTimeLib](https://github.com/PlantFactory/LocalTimeLib)
* [SerialCLI](https://github.com/PlantFactory/SerialCLI)
* [NTP](https://github.com/PlantFactory/NTP)
* [ADT74x0](https://github.com/PlantFactory/ADT74x0)


### 初期設定

21行目から35行目にかけて，設定項目があります．

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

* `mac`はM2MゲートウェイのMACアドレスを指定してください．
  * ボードによって異なりますから注意してください．
  * おなじセグメントにおなじMACアドレスのデバイスが存在すると，大変なことになります．
* `dhcp`はtrueのときIPアドレスの設定をdhcpで行います．
* `ip`は`dhcp`がfalseのとき，GWのIPアドレスとして設定されます．
* `gw`は`dhcp`がfalseのとき，GWのデフォルトゲートウェイアドレスとして設定されます．
* `sm`は`dhcp`がfalseのとき，GWのサブネット・マスクとして設定されます．
* `dns_server`は`dhcp`がfalseのとき，GWが利用するDNSサーバーとして設定されます．
* `ntp`はGWが参照するNTPサーバーを指定します．
* `host`はGWがデータをWRITEするStorageのアドレスを指定します．
* `port`はGWがデータをWRITEするStorageのポートを指定します．
* `path`はGWがデータをWRITEするStorageのサービスのパスを指定します．
* `prefix`はGWのポイントIDのプレフィックスを表現します．
  * linuxでいうdirnameコマンドの結果を想像するとわかりやすいかもしれません．
  * `http://ujyu.net/sample_gw/temperature`
    * プレフィックス：`http://ujyu.net/sample_gw/`

基本的には`mac`，`host`，`prefix`を設定すれば問題ないでしょう．

### ハードウェア

[Arduinoの基礎，ADT74x0]()で作成した回路がそのまま使えます．

### テスト

それでは設定，書き込みとセンサーの接続を確認したら，LANケーブルを接続しましょう．おそらく，GUTPのリファレンス実装Storageを利用している場合はStorageのURLをブラウザで開くと，計測値が閲覧できるでしょう．
