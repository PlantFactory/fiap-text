## 基本用語

* FETCH
  * データを取得すること
  * Download
* WRITE
  * データを書き込むこと
  * Upload
* TRAP
  * TODO: Under construction
* Storage
  * データを蓄積するサーバー
* Gateway
  * IEEE1888と異なるプロトコルで接続する機器を橋渡しするもの
  * 電力計 <> RS485 <> GW <> IEEE1888 <> Storage
  * 温度センサー <> I2C <> GW <> IEEE1888 <> Storage
* APP
  * IEEE1888を利用してデータを読み書きするもの
  * 例えばビューワーならStorageからFETCHして表示するAPP
  * 分析アプリケーションならStorageからFETCHし，分析，結果をWRITEするAPP
* Point, Point ID
  * 計測点，変数のようなもの
    * ある地点の温度
    * ある地点の電力
  * URIで表す．が，IPアドレスと対応するわけではない．あくまで，ユニークな識別子として用いる．
    * http://ujyu.net/laboratory/temperature
      * ujyu.net->鵜重が管理するもの
      * laboratory->場所
      * temperature->ポイント
      * ujyu.netはドメインを所持していなくても良いが，ユニークであることを保証するなら所持するべき
      * 繰り返すが，識別子です．
