## LocalTimeLibの使い方

LocalTimeLibはArduinoでUNIX時をタイムゾーン付きで管理するライブラリです．基本的にはTimeライブラリを使用している前提で処理を行います．

inoファイルのグローバルスコープにタイムゾーンを定義してください．
```C++
TimeZone localtimezone = {  9*60*60, 0, "+09:00" };
// { Offset from UTC, SummerTime, ISO String }
```
サマータイムは非対応ですが，データとしては確保してあります． ISO StringにはISO8601に従って表記を行う場合のタイムゾーン文字列を定義してください．

`localtime()`を呼び出すと`TimeElements`のポインタが帰ってきます．TimeElements内の各要素にタイムゾーンを適用した時刻情報が保存されています．ISO8601形式で時刻を表示する場合，次のようなコードになります．
```C++
// in global
TimeZone localtimezone = {  9*60*60, 0, "+09:00" };

// in some function
setTime(0);
TimeElements *tm = localtime();
printf("%04d-%02d-%02dT%02d:%02d:%02d%s",tm->Year + 1970, tm->Month, tm->Day, tm->Hour, tm->Minute, tm->Second, localtimezone.iso_string);
// get: 1970-01-01-00:00:00+09:00
```
