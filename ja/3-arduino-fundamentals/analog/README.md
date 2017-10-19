## アナログ入出力

### アナログ入力

明るさや温度などのアナログ情報を電圧から読み取ります．ArduinoなどのマイコンではAD(アナログ・デジタル)変換器を使います．AD変換器は入力電圧を基準電圧と比較して基準電圧からの比を出力します．AD変換器は変換可能なビット数が決まっています．Arduinoの変換器は10bitです．AD変換器を擬似コードで書くとこのようになります．
```C++
const uint16_t maxValue = 1023; // 10bit -> 0 ~ 1023
float inputVoltage = 3.1415;
float referenceVoltage = 5.001;

int digitalValue = inputVoltage / referenceVoltage * 1023; // 643
```
ちなみにこのデジタル値を再び電圧に戻すのも簡単です．
```C++
const uint16_t maxValue = 1023; // 10bit -> 0 ~ 1023
float referenceVoltage = 5.001;
int digitalValue = 643;

float analogVoltage = 643 / maxValue * referenceVoltage; // 3.1433460411
```

実際にアナログ入力をするにはArduinoのA0からA5ピンに電圧を入力して`analogRead`関数で読み込むだけです．
```C++
uint16_t val = analogRead(A0);
```

### アナログ出力

逆に，デジタル値からアナログ電圧で出力するものをDA変換器と言います．方式はたくさんありますが，ArduinoではPWM(Pulse Width Modulation)が利用できます．PWMは一定の周期でかつ高速にデジタル信号のON・OFFを切り替えることによって電圧を表現します．長期的に見ればONしている時間のほうが多ければ電圧が高く，OFFしている時間のほうが短ければ電圧は低く見えます．このONとOFFの時間の比をデューティー比と言います．ArduinoのPWMは8bitで指定できます．ただしPWMが出力できるのは3,5,6,9,10,11の6ピンだけです．`analogWrite`関数にピンとデューティー比を渡しましょう．デューティー比は0で0[V]，255で電源電圧になります．
```C++
analogWrite(3, 255);
```

PWMは厳密に言えばデジタル信号です．光などの高速な媒体ではチラツキが問題になることがあります．これをよりアナログ値に近づけるにはコンデンサや抵抗で平滑回路を組み込んでください．
